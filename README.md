# 🚀 TypeScript Backend - AWS EC2 Deployment Guide

> Complete step-by-step guide to deploy a TypeScript Node.js backend to AWS EC2 with Redis, Nginx, SSL, and PM2.

---

## 📋 Pre-Deployment Checklist

Before starting, make sure you have:

- [ ] AWS IAM credentials with EC2 access
- [ ] MongoDB Atlas production database ready
- [ ] All production credentials (Firebase, payment gateway, etc.)
- [ ] Code pushed to GitHub `main` branch
- [ ] `npm run build` works locally without errors
- [ ] Domain name purchased and accessible

---

## 📝 Replace These Placeholders

Throughout this guide, replace these with your actual values:

| Placeholder | Example | Your Value |
|-------------|---------|------------|
| `YOUR_PROJECT_NAME` | loopcall-backend | ___________ |
| `YOUR_DOMAIN` | api.example.com | ___________ |
| `YOUR_PUBLIC_IP` | 13.235.xxx.xxx | ___________ |
| `YOUR_GITHUB_URL` | https://github.com/user/repo.git | ___________ |
| `YOUR_PORT` | 8000 | ___________ |

---

## Part 1: Create EC2 Instance

### Step 1.1: Login to AWS Console

1. Go to [AWS Console](https://console.aws.amazon.com/)
2. Login with your IAM credentials
3. Select your preferred region (e.g., Mumbai `ap-south-1`)

### Step 1.2: Launch EC2 Instance

1. Go to **EC2 Dashboard** → Click **Launch Instance**

2. **Name your instance:**
   ```
   YOUR_PROJECT_NAME
   ```

3. **Choose AMI (Amazon Machine Image):**
   - Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type**
   - Architecture: **64-bit (x86)**

4. **Choose Instance Type:**
   - Select **t2.micro** (Free Tier) or **t2.small** for more traffic

5. **Create Key Pair:**
   - Click **Create new key pair**
   - Name: `YOUR_PROJECT_NAME-key`
   - Type: **RSA**
   - Format: **.pem**
   - Click **Create key pair** - **⚠️ SAVE THIS FILE SECURELY!**

6. **Network Settings:**
   - Click **Edit**
   - Keep default VPC
   - **Auto-assign public IP:** Enable
   - **Create security group** with name: `YOUR_PROJECT_NAME-sg`

7. **Configure Security Group Rules:**

   | Type | Port Range | Source | Description |
   |------|------------|--------|-------------|
   | SSH | 22 | My IP | SSH access |
   | HTTP | 80 | 0.0.0.0/0, ::/0 | Web traffic |
   | HTTPS | 443 | 0.0.0.0/0, ::/0 | Secure web traffic |
   | Custom TCP | YOUR_PORT | 0.0.0.0/0, ::/0 | Node.js app (temporary) |

8. **Configure Storage:**
   - Keep default **8 GB** or increase to **20 GB** if needed

9. Click **Launch Instance**

### Step 1.3: Wait for Instance to Start

1. Go to **EC2 Dashboard** → **Instances**
2. Wait until **Instance State** shows **Running** (green)
3. Wait until **Status checks** shows **2/2 checks passed**
4. **Copy the Public IPv4 address** - you'll need this!

---

## Part 2: Connect to EC2 via SSH

### Step 2.1: Open PowerShell

1. Open **PowerShell** (search in Start menu)
2. Navigate to folder where `.pem` file is saved:
   ```bash
   cd Downloads
   ```

### Step 2.2: Connect via SSH

```bash
ssh -i "YOUR_PROJECT_NAME-key.pem" ubuntu@YOUR_PUBLIC_IP
```

**First time connecting?** Type `yes` when asked about fingerprint.

✅ You should now see: `ubuntu@ip-xxx-xxx-xxx-xxx:~$`

---

## Part 3: Install Required Software

### Step 3.1: Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3.2: Install Node.js 20.x

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify installation:
```bash
node -v    # Should show v20.x.x
npm -v     # Should show 10.x.x
```

### Step 3.3: Install Git

```bash
sudo apt install git -y
git --version
```

### Step 3.4: Install Redis (if your project uses Redis)

```bash
sudo apt install redis-server -y
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

Verify Redis:
```bash
redis-cli ping    # Should return: PONG
```

### Step 3.5: Install PM2 (Process Manager)

```bash
sudo npm install -g pm2
pm2 --version
```

### Step 3.6: Install Nginx (Web Server)

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Verify Nginx:
```bash
sudo systemctl status nginx    # Should show: active (running)
```

---

## Part 4: Clone and Setup Application

### Step 4.1: Generate GitHub Token (for private repos)

1. Go to: **https://github.com/settings/tokens**
2. Click **Generate new token (classic)**
3. Name: `EC2-YOUR_PROJECT_NAME`
4. Expiration: **No expiration**
5. Check: ✅ `repo`
6. Click **Generate token** and **COPY IT!**

### Step 4.2: Clone Repository

```bash
cd ~
git clone https://YOUR_TOKEN@github.com/USERNAME/REPO_NAME.git
cd REPO_NAME
```

### Step 4.3: Install Dependencies

```bash
npm install
```

### Step 4.4: Create Environment File

```bash
nano .env
```

Paste your production environment variables, then save:
- Press `Ctrl + X`
- Press `Y`
- Press `Enter`

Verify `.env` file:
```bash
cat .env
```

### Step 4.5: Build TypeScript

```bash
npm run build
```

Verify build:
```bash
ls dist/    # Should show server.js and other files
```

---

## Part 5: Start Application with PM2

### Step 5.1: Start the Application

```bash
pm2 start dist/server.js --name "YOUR_PROJECT_NAME"
```

### Step 5.2: Verify Application is Running

```bash
pm2 status
```

### Step 5.3: Check Logs

```bash
pm2 logs YOUR_PROJECT_NAME
```

### Step 5.4: Configure PM2 to Auto-Start on Reboot

```bash
pm2 startup
```

Copy and run the command it shows, then:
```bash
pm2 save
```

### Step 5.5: Test Application

```bash
curl http://localhost:YOUR_PORT/health
```

---

## Part 6: Configure Nginx as Reverse Proxy

### Step 6.1: Edit Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/default
```

**Delete everything** and paste this:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name YOUR_DOMAIN;

    location / {
        proxy_pass http://localhost:YOUR_PORT;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # WebSocket support for Socket.IO
        proxy_read_timeout 86400;
    }
}
```

Save and exit.

### Step 6.2: Test Nginx Configuration

```bash
sudo nginx -t
```

### Step 6.3: Restart Nginx

```bash
sudo systemctl restart nginx
```

### Step 6.4: Test via Public IP

Open browser: `http://YOUR_PUBLIC_IP/health`

---

## Part 7: Configure Domain DNS

### Step 7.1: Get Your EC2 Public IP

```bash
curl ifconfig.me
```

### Step 7.2: Add DNS Records

In your domain registrar, add:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | api | YOUR_PUBLIC_IP | 300 |

For root domain and www:
| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | YOUR_PUBLIC_IP | 300 |
| A | www | YOUR_PUBLIC_IP | 300 |

### Step 7.3: Test DNS

Wait 5-30 minutes, then:
```bash
curl http://YOUR_DOMAIN/health
```

---

## Part 8: Install SSL Certificate (HTTPS)

### Step 8.1: Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Step 8.2: Get SSL Certificate

```bash
sudo certbot --nginx -d YOUR_DOMAIN
```

**When prompted:**
1. Enter email address
2. Agree to terms: `Y`
3. Share email with EFF: `N`
4. Redirect HTTP to HTTPS: `2`

### Step 8.3: Verify SSL Auto-Renewal

```bash
sudo certbot renew --dry-run
```

### Step 8.4: Test HTTPS

Open browser: `https://YOUR_DOMAIN/health`

---

## Part 9: MongoDB Atlas - Whitelist EC2 IP

1. Go to [MongoDB Atlas](https://cloud.mongodb.com/)
2. Go to your cluster → **Network Access**
3. Click **Add IP Address**
4. Add: `YOUR_PUBLIC_IP/32`
5. Description: `EC2 Production Server`

---

## 📋 Useful PM2 Commands

| Command | Description |
|---------|-------------|
| `pm2 status` | Check application status |
| `pm2 logs YOUR_PROJECT_NAME` | View real-time logs |
| `pm2 logs YOUR_PROJECT_NAME --lines 100` | View last 100 logs |
| `pm2 restart YOUR_PROJECT_NAME` | Restart application |
| `pm2 stop YOUR_PROJECT_NAME` | Stop application |
| `pm2 delete YOUR_PROJECT_NAME` | Remove from PM2 |
| `pm2 monit` | Real-time monitoring |

---

## 🔄 How to Deploy Updates

```bash
cd ~/REPO_NAME
git pull origin main
npm install
npm run build
pm2 restart YOUR_PROJECT_NAME
```

---

## ⚠️ Troubleshooting

### Application not starting?
```bash
pm2 logs YOUR_PROJECT_NAME --lines 50
```

### Nginx errors?
```bash
sudo nginx -t
sudo tail -f /var/log/nginx/error.log
```

### Redis not working?
```bash
sudo systemctl status redis-server
redis-cli ping
```

### MongoDB connection failed?
- Check if EC2 IP is whitelisted in MongoDB Atlas
- Verify connection string in `.env`

### Port already in use?
```bash
sudo lsof -i :YOUR_PORT
pm2 delete all
pm2 start dist/server.js --name "YOUR_PROJECT_NAME"
```

### Certbot "already running" error?
```bash
sudo pkill certbot
sudo certbot --nginx -d YOUR_DOMAIN
```

---

## ✅ Deployment Complete!

Your backend is now:
- ✅ Running on EC2 with PM2
- ✅ Behind Nginx reverse proxy
- ✅ Secured with SSL (HTTPS)
- ✅ Auto-restarts on crash/reboot

---

## 👤 Author

**Sameen K A** - CTO at Veevity Technologies
