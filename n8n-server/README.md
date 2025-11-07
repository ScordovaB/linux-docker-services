# n8n Self-Hosted Deployment (Ubuntu + Docker + PostgreSQL)

This repository contains everything needed to deploy a production-ready **n8n instance with PostgreSQL** on your Ubuntu server using **Docker & Docker Compose**.
The setup is designed for small teams, agencies, and low-to-medium workloads.

---

## ✅ Features

* n8n running in Docker
* PostgreSQL database container
* Automatic daily database backups
* Persistent volumes for database & n8n data
* Environment variables already configured
* Secure setup using `.env` file
* Ready to connect behind Cloudflare Tunnel or Nginx

---

## 📦 Folder Structure

```
.
├── docker-compose.yml
├── .env
├── init-data.sh
└── README.md
```

---

## 🚀 1. Requirements

Before deploying, ensure your server has:

* Ubuntu Server 22.04 or newer
* Docker + Docker Compose installed
* At least **2GB RAM** (recommended 4GB)
* A domain (optional but recommended)

### Install Docker

```bash
curl -fsSL https://get.docker.com | sudo bash
sudo usermod -aG docker $USER
```

Log out and back in.

### Install Docker Compose

```bash
sudo apt install docker-compose -y
```

---

## 🔧 2. Configure Environment Variables

Create a `.env` file based on this template:

```env
# -----------------------------
# N8N BASIC CONFIG
# -----------------------------
PUID=1000
PGID=1000
TZ=America/Mexico_City
N8N_PORT=5678
GENERIC_TIMEZONE=America/Mexico_City

# -----------------------------
# POSTGRES CONFIG
# -----------------------------
POSTGRES_USER=n8n
POSTGRES_PASSWORD=supersecurepassword
POSTGRES_DB=n8ndb

# -----------------------------
# N8N ENVIRONMENT
# -----------------------------
N8N_HOST=localhost
N8N_PROTOCOL=http

# Set a long random encryption key
N8N_ENCRYPTION_KEY=GENERATE_A_256BIT_KEY

# If using a domain later
# WEBHOOK_URL=https://automation.yourdomain.com
```

To generate an encryption key:

```bash
openssl rand -hex 32
```

---

## 🐳 3. Docker Compose Setup

`docker-compose.yml`

---

## 🚀 4. Start Everything

Run:

```bash
docker-compose up -d
```

Verify containers:

```bash
docker ps
```

---

## ✅ 5. Access n8n

If local network:

```
http://SERVER-IP:5678
```

If behind Cloudflare Tunnel:

```
https://automation.yourdomain.com
```

---

## 🧰 6. Useful Commands

### View logs

```bash
docker-compose logs -f n8n
```

### Restart services

```bash
docker-compose restart
```

### Stop everything

```bash
docker-compose down
```

---

## 🗄️ 7. Database Backup (Automatic)

Create a simple cron job:

```bash
crontab -e
```

Add:

```
0 3 * * * docker exec postgres pg_dump -U n8n n8ndb > /home/ubuntu/n8n_backup_$(date +\%F).sql
```

Backups saved daily at 3 AM.

---

## 🔐 8. Security Notes

* Use Cloudflare Tunnel or Nginx Proxy Manager to expose safely.
* Always use **HTTPS** in production.
* Keep your `.env` private — never commit it.
* Regularly update containers:

```bash
docker-compose pull && docker-compose up -d
```

---

## ✅ 9. Finished!

Your n8n instance is now ready for production use.
This setup is ideal for agencies, small teams, automation workflows, and process management.

If you need to add workers or scale horizontally later, the setup can be extended easily.
