# 🚀 Pareng Boyong - Quick Start for New VPS

**Deploy Pareng Boyong to your own VPS in ~15 minutes**

---

## 📋 Prerequisites

Before you start, ensure your VPS has:

- ✅ Ubuntu 22.04+ or Debian 11+
- ✅ Root/sudo access
- ✅ Minimum 2GB RAM, 10GB disk space
- ✅ Domain name (optional but recommended)
- ✅ Public IP address

---

## 🔧 Quick Install (Docker Method)

### Step 1: Update System & Install Docker

```bash
sudo apt-get update && sudo apt-get upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Enable Docker service
sudo systemctl enable docker
sudo systemctl start docker
```

### Step 2: Clone Repository

```bash
cd /opt
sudo git clone https://github.com/innovatehubph/pareng-boyong.git
cd pareng-boyong
```

### Step 3: Create Data Directory

```bash
sudo mkdir -p /srv/pareng-boyong-data
sudo chmod 777 /srv/pareng-boyong-data
```

### Step 4: Pull Docker Image

```bash
docker pull agent0ai/agent-zero:latest
```

### Step 5: Run Container

```bash
docker run -d \
  --name pareng-boyong \
  --restart unless-stopped \
  -p 5000:80 \
  -v /srv/pareng-boyong-data:/a0 \
  agent0ai/agent-zero:latest
```

### Step 6: Verify Installation

```bash
# Check container is running
docker ps | grep pareng-boyong

# Check logs
docker logs -f pareng-boyong

# Test endpoint
curl http://localhost:5000/api/chat/health
```

**You should see:** `{"status": "ok"}`

---

## 🌐 Expose to Internet (Nginx)

### Install Nginx

```bash
sudo apt-get install -y nginx
sudo systemctl enable nginx
```

### Create Nginx Config

Replace `your-domain.com` with your actual domain:

```bash
sudo nano /etc/nginx/sites-available/default
```

Paste this configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Test & Start Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔐 Add HTTPS (Let's Encrypt)

### Install Certbot

```bash
sudo apt-get install -y certbot python3-certbot-nginx
```

### Get Free SSL Certificate

```bash
sudo certbot --nginx -d your-domain.com
```

The certificate will auto-renew. Nginx config is automatically updated.

---

## 🔑 Configure Authentication

### Set Environment Variables

```bash
nano /srv/pareng-boyong-data/.env
```

Add:

```env
# Basic Authentication
BASIC_AUTH_USERNAME=admin
BASIC_AUTH_PASSWORD=changeme123

# CORS (replace with your domain)
ALLOWED_ORIGINS=https://your-domain.com

# Optional: API Key
API_KEY=your-secure-api-key
```

---

## 🧪 Test Your Installation

### Access Web Interface

1. Go to `https://your-domain.com` in browser
2. Login with credentials from `.env` file
3. Start chatting with Pareng Boyong!

### Test via API

```bash
curl -X POST https://your-domain.com/api_message \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Hello, Pareng Boyong!",
    "user": "Boss Marc"
  }'
```

---

## 📊 Monitor Your Instance

### Check Logs

```bash
# Real-time logs
docker logs -f pareng-boyong

# Last 100 lines
docker logs --tail 100 pareng-boyong
```

### Check Resource Usage

```bash
docker stats pareng-boyong
```

### Check Disk Space

```bash
df -h /srv/pareng-boyong-data/
du -sh /srv/pareng-boyong-data/
```

---

## 🆘 Troubleshooting

### Container won't start

```bash
# Remove and recreate
docker stop pareng-boyong
docker rm pareng-boyong
docker run -d --name pareng-boyong --restart unless-stopped \
  -p 5000:80 \
  -v /srv/pareng-boyong-data:/a0 \
  agent0ai/agent-zero:latest

# Check logs
docker logs pareng-boyong
```

### Website not accessible

```bash
# Check Nginx
sudo nginx -t
sudo systemctl status nginx

# Check firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Test locally
curl http://localhost:5000/api/chat/health
```

### Out of disk space

```bash
# Clean Docker
docker system prune -a

# Check data directory
du -sh /srv/pareng-boyong-data/

# Remove old logs if needed
sudo find /srv/pareng-boyong-data -name "*.log" -mtime +30 -delete
```

---

## 📝 Common Commands

```bash
# Restart container
docker restart pareng-boyong

# Stop container
docker stop pareng-boyong

# Start container
docker start pareng-boyong

# View all container info
docker inspect pareng-boyong

# Execute command in container
docker exec pareng-boyong ls -la /a0

# Backup data
tar -czf ~/backup-$(date +%Y%m%d).tar.gz /srv/pareng-boyong-data/
```

---

## 🎯 Next Steps

1. ✅ Container running
2. ✅ Exposed via Nginx
3. ✅ SSL certificate active
4. ✅ Authentication enabled
5. **→** Read [LINUX_DEPLOYMENT.md](LINUX_DEPLOYMENT.md) for advanced config
6. **→** Check [CLAUDE.md](CLAUDE.md) for adding custom tools
7. **→** Explore [TOOL_CREATION_QUICKSTART.md](TOOL_CREATION_QUICKSTART.md) for extensions

---

## 📞 Support

For issues, check:
- [LINUX_DEPLOYMENT.md](LINUX_DEPLOYMENT.md) - Full setup guide
- [README.md](README.md) - Overview
- Docker logs: `docker logs pareng-boyong`

---

**Deployed successfully?** 🎉
Update your DNS to point to your VPS IP, and you're live!
