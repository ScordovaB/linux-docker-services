✅ Creates nightly backups of

* PostgreSQL (`.sql`)
* n8n data folder (`.tar.gz`)

✅ Uploads those backups to

* ✅ **Cloudflare R2**
* ✅ **AWS S3**

✅ Keeps local copies (optional)

✅ Automatically cleans up older local backups

✅ Uses **rclone** for all cloud uploads (recommended by Cloudflare & AWS)

You will get:

✅ Installation
✅ Config
✅ Backup script
✅ Cron job
✅ Restore guide

---

# ✅ Step 1 — Install rclone (required)

Run:

```bash
curl https://rclone.org/install.sh | sudo bash
```

Confirm:

```bash
rclone version
```

---

# ✅ Step 2 — Configure Cloudflare R2

Run:

```bash
rclone config
```

Select:

```
n) New remote
name> cloudflare_r2
```

Choose *S3 compatible backend*:

```
Storage> 4  (Amazon S3)
```

Then:

| Option              | Value                                                |
| ------------------- | ---------------------------------------------------- |
| provider            | `Cloudflare`                                         |
| env_auth            | `false`                                              |
| access_key_id       | your R2 access key                                   |
| secret_access_key   | your R2 secret key                                   |
| region              | `auto`                                               |
| endpoint            | `https://<YOUR_ACCOUNT_ID>.r2.cloudflarestorage.com` |
| location_constraint | `auto`                                               |
| acl                 | `private`                                            |

When asked **"Edit advanced config?"**, choose:

```
n
```

When asked **"Use this remote?"**, choose:

```
y
```

✅ Your R2 remote is now ready: `cloudflare_r2:yourbucketname`

---

# ✅ Step 3 — Configure AWS S3

Run again:

```bash
rclone config
```

Create a second remote:

```
name> aws_s3
Storage> 4 (Amazon S3)
```

Enter:

| Option            | Value                      |
| ----------------- | -------------------------- |
| provider          | `AWS`                      |
| env_auth          | `false`                    |
| access_key_id     | your AWS key               |
| secret_access_key | your AWS secret            |
| region            | us-east-1 (or your region) |
| endpoint          | *leave blank*              |
| acl               | private                    |

✅ Your S3 remote is now: `aws_s3:yourbucketname`

---

# ✅ Step 4 — Create backup folder

```bash
sudo mkdir -p /opt/backups/n8n
sudo chmod 755 /opt/backups/n8n
```

---

# ✅ Step 5 — Create the full backup script with cloud uploads

Create:

```bash
sudo nano /opt/backups/backup_n8n_cloud.sh
```

Paste this:

```bash
#!/bin/bash

### CONFIG ###
BACKUP_DIR="/opt/backups/n8n"
DATE=$(date +"%Y-%m-%d_%H-%M")
POSTGRES_CONTAINER="myproject_postgres_1"   # <- UPDATE THIS
N8N_VOLUME="n8n_storage"

CF_R2_REMOTE="cloudflare_r2:myr2bucket/backups/n8n"   # <- UPDATE THIS
AWS_S3_REMOTE="aws_s3:mys3bucket/backups/n8n"         # <- UPDATE THIS

### POSTGRES BACKUP ###
echo "[*] Creating Postgres backup..."
docker exec $POSTGRES_CONTAINER pg_dump -U ${POSTGRES_NON_ROOT_USER} ${POSTGRES_DB} > \
  $BACKUP_DIR/db_$DATE.sql

### N8N VOLUME BACKUP ###
echo "[*] Creating n8n data backup..."
docker run --rm -v ${N8N_VOLUME}:/volume -v $BACKUP_DIR:/backup alpine \
  sh -c "cd /volume && tar -czf /backup/n8n_data_$DATE.tar.gz ."

### UPLOAD TO CLOUDFLARE R2 ###
echo "[*] Uploading to Cloudflare R2..."
rclone copy $BACKUP_DIR/db_$DATE.sql $CF_R2_REMOTE
rclone copy $BACKUP_DIR/n8n_data_$DATE.tar.gz $CF_R2_REMOTE

### UPLOAD TO AWS S3 ###
echo "[*] Uploading to AWS S3..."
rclone copy $BACKUP_DIR/db_$DATE.sql $AWS_S3_REMOTE
rclone copy $BACKUP_DIR/n8n_data_$DATE.tar.gz $AWS_S3_REMOTE

### LOCAL CLEANUP (older than 30 days) ###
echo "[*] Cleaning old local backups..."
find $BACKUP_DIR -type f -mtime +30 -delete

echo "[✔] Backup completed: $DATE"
```

---

# ✅ Step 6 — Make the script executable

```bash
sudo chmod +x /opt/backups/backup_n8n_cloud.sh
```

---

# ✅ Step 7 — Add cron job (3 AM nightly)

```bash
sudo crontab -e
```

Add:

```
0 3 * * * /opt/backups/backup_n8n_cloud.sh >> /opt/backups/n8n/cron.log 2>&1
```

✅ Backups run every night at 3:00 AM
✅ Logs go to `/opt/backups/n8n/cron.log`

---

# ✅ Step 8 — Test the backup manually

Run once:

```bash
sudo /opt/backups/backup_n8n_cloud.sh
```

Check files:

```
ls /opt/backups/n8n
```

Check uploads:

```
rclone ls cloudflare_r2:myr2bucket/backups/n8n
rclone ls aws_s3:mys3bucket/backups/n8n
```

✅ If you see files, everything works.

---

# ✅ Restore Instructions (Cloudflare or S3)

Download backups:

```bash
rclone copy cloudflare_r2:myr2bucket/backups/n8n /opt/backups/restore
```

OR

```bash
rclone copy aws_s3:mys3bucket/backups/n8n /opt/backups/restore
```

### Restore Postgres

```bash
cat db_2025-01-02_03-00.sql | docker exec -i myproject_postgres_1 psql -U myuser mydb
```

### Restore n8n data

```bash
docker run --rm -v n8n_storage:/volume -v /opt/backups/restore:/backup alpine \
  sh -c "rm -rf /volume/* && tar -xzf /backup/n8n_data_2025-01-02_03-00.tar.gz -C /volume"
```

✅ Fully restored system.

---