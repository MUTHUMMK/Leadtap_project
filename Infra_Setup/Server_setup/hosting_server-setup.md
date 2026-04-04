# Linux Server Setup & Deployment Guide (DevSecOps)

This document describes step-by-step setup of Ubuntu 22.04 LTS server, hardening, deployment, and Nginx reverse proxy configuration.

---

# 1 Create Linux Server (Ubuntu 22.04 LTS)

##  Login & Update System

```bash
sudo apt update && sudo apt upgrade -y
```

---

#  2 Create DevOps User

```bash
sudo useradd devops
sudo usermod -aG sudo devops
sudo passwd devops
```

---

##  User Types

-  Human User:
  - SSH key login
  - sudo with password

-  Automation User:
  - Passwordless sudo (restricted usage)

---

# 3 SSH Hardening

```bash
sudo nano /etc/ssh/sshd_config
```

## Configuration

```
Port 222
PermitRootLogin no
PasswordAuthentication no
AllowUsers devops
```

---

##  SSH Key Setup

```bash
sudo mkdir -p /home/devops/.ssh
sudo cp ~/.ssh/authorized_keys /home/devops/.ssh/
```

## Permissions

```bash
sudo chown -R devops:devops /home/devops
sudo chmod 700 /home/devops/.ssh
sudo chmod 600 /home/devops/.ssh/authorized_keys
sudo systemctl restart ssh
```

---

# 4 Configure UFW Firewall

```bash
sudo apt install ufw -y
```
## Enable Firewall

```bash
sudo ufw enable
sudo ufw status
```

## Allow Ports

```bash
sudo ufw allow 222/tcp # custom SSH port
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3000/tcp # App running port
```

---

## Security Enhancements

```bash
sudo ufw limit 222/tcp
```

---

# 5 Install Node.js (NVM)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

## Load NVM

```bash
sudo cp /etc/skel/.bashrc /home/devops/

sudo nano ~/.bashrc
==>
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

source ~/.bashrc
```

## Install Node.js 18

```bash
nvm install 18
nvm use 18
nvm alias default 18
```

## Verify

```bash
node -v
npm -v
```

---

# 6 Application Deployment

## Clone Project

```bash
git clone https://github.com/demo.git
cd app
```

## Install & Build

```bash
npm install
npm run build
npm start
```

---

##  PM2 Process Manager

```bash
npm install pm2 -g
pm2 start npm --name app -- start
pm2 status
```

## Auto Restart on Boot

```bash
pm2 startup
pm2 save
```

---

## Check Application

```bash
curl http://localhost:3000
```

---

# 7 Nginx Reverse Proxy

## Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

---

## Configure Site

```bash
sudo nano /etc/nginx/sites-available/app.conf
```

## Config

```nginx
#  Rate Limiting Zone (add in http block in nginx.conf)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=5r/s;

server {
    listen 80;
    listen [::]:80;

    server_name YourDomain;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Rate Limiting (protect API abuse / DDoS)
    location / {
        limit_req zone=api_limit burst=10 nodelay;

        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    #  Block hidden & sensitive files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    #  Block specific bad IPs
    # deny 192.168.1.100;
    # deny 203.0.113.10;

    #  allow only specific IP range
    # allow 1.2.3.4;
    # deny all;
}
```

---

## Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

# 8 SSL Configuration (HTTPS)

## Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

## Enable SSL

```bash
sudo certbot --nginx -d YourDomain
```

---

## Verify HTTPS

```bash
curl -I https://YourDomain
```

---

#  System Flow

User → HTTPS (443)
   ↓
Nginx Reverse Proxy
   ↓
Node.js App (PM2 on 3000)

---

# Final Result

✔ Secure SSH login  
✔ Firewall enabled  
✔ Node.js runtime ready  
✔ App deployed with PM2  
✔ Nginx reverse proxy  
✔ HTTPS enabled (SSL)  

---


