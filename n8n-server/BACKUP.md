✅ **This includes BOTH:**

* A full **Postgres dump** (`.sql`)
* A full **n8n data folder backup** (`.tar.gz`)

✅ **These backups can be dropped into a brand-new server and restored 1:1.**
✅ **Cron job will run every night at 3:00 AM automatically.**

Below is the complete setup.

---

# ✅ 1. Create backup folder on your Ubuntu server

```bash
sudo mkdir -p /opt/backups/n8n
sudo chmod 755 /opt/backups/n8n
```

---

# ✅ 2. Create the backup script

**Create file:**

```
sudo nano /opt/backups/backup_n8n.sh
```

**Paste this:**

```bash
#!/bin/bash

# Directories
BACKUP_DIR="/opt/backups/n8n"
DATE=$(date +"%Y-%m-%d_%H-%M")
POSTGRES_CONTAINER="yourfoldername_postgres_1" # update this below
N8N_VOLUME="n8n_storage"

# 1. Postgres backup
echo "Creating Postgres backup..."
docker exec $POSTGRES_CONTAINER pg_dump -U ${POSTGRES_NON_ROOT_USER} ${POSTGRES_DB} > $BACKUP_DIR/db_$DATE.sql

# 2. n8n data backup
echo "Creating n8n volume backup..."
docker run --rm -v ${N8N_VOLUME}:/volume -v $BACKUP_DIR:/backup alpine \
  sh -c "cd /volume && tar -czf /backup/n8n_data_$DATE.tar.gz ."

# 3. Clean backups older than 30 days
find $BACKUP_DIR -type f -mtime +30 -delete

echo "Backup completed: $DATE"
```

✅ This script will create two files every night:

```
db_2025-01-02_03-00.sql
n8n_data_2025-01-02_03-00.tar.gz
```

These allow **full recreate on any machine**.

---

# ✅ 3. Make script executable

```bash
sudo chmod +x /opt/backups/backup_n8n.sh
```

---

# ✅ 4. Find your Postgres container name

Run:

```bash
docker ps
```

Find your Postgres service, for example:

```
myproject_postgres_1
```

Put that name inside the script here:

```
POSTGRES_CONTAINER="myproject_postgres_1"
```

---

# ✅ 5. Add cron job to run at 3 AM

Open your crontab:

```bash
sudo crontab -e
```

Add this line:

```
0 3 * * * /opt/backups/backup_n8n.sh >> /opt/backups/n8n/cron.log 2>&1
```

✅ Runs every day at 3:00 AM
✅ Logs output so you can debug if needed

---

# ✅ 6. (Optional) Upload backups to cloud automatically

I can help you integrate:

✅ AWS S3
✅ Google Cloud Storage
✅ Backblaze B2 (very cheap!)
✅ Dropbox
✅ Mega
✅ Google Drive
✅ rclone (supports 60+ storage providers)

---

# ✅ 7. Restore Instructions (Safe & Simple)

On a new server, you do:

### Restore Postgres:

```bash
cat db_2025-01-02_03-00.sql | docker exec -i myproject_postgres_1 psql -U myuser mydb
```

### Restore n8n data:

```bash
docker run --rm -v n8n_storage:/volume -v $(pwd):/backup alpine \
  sh -c "rm -rf /volume/* && tar -xzf /backup/n8n_data_2025-01-02_03-00.tar.gz -C /volume"
```

Boom — **your entire n8n is restored perfectly**.

---