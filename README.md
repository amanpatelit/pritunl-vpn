# ☁️ Pritunl VPN Production Setup on Ubuntu 22.04

<p align="center">
  <img src="https://pritunl.com/img/logo.png" width="180" alt="Pritunl VPN">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ubuntu-22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white">
  <img src="https://img.shields.io/badge/Pritunl-VPN-blue?style=for-the-badge">
  <img src="https://img.shields.io/badge/WireGuard-Supported-success?style=for-the-badge">
  <img src="https://img.shields.io/badge/OpenVPN-Supported-orange?style=for-the-badge">
  <img src="https://img.shields.io/badge/AWS-Production-yellow?style=for-the-badge&logo=amazonaws">
</p>

---

# 📌 Overview

This repository provides a complete guide for deploying a **Production-Ready Pritunl VPN Server** on **Ubuntu 22.04 LTS** using:

- ✅ Pritunl VPN
- ✅ MongoDB
- ✅ OpenVPN
- ✅ WireGuard
- ✅ Nginx Reverse Proxy
- ✅ Let's Encrypt SSL
- ✅ UFW Firewall
- ✅ Fail2Ban
- ✅ Production Security Hardening

---

# 🏗️ Architecture

```text
                    Internet
                        │
                ┌───────▼───────┐
                │   Cloudflare  │
                └───────┬───────┘
                        │
                ┌───────▼───────┐
                │   AWS EC2     │
                │ Ubuntu 22.04  │
                └───────┬───────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
 ┌───────▼────────┐         ┌──────────▼─────────┐
 │    Nginx       │         │      MongoDB       │
 │ Reverse Proxy  │         │    Database        │
 └───────┬────────┘         └──────────┬─────────┘
         │                              │
         └──────────────┬──────────────┘
                        │
                ┌───────▼───────┐
                │    Pritunl    │
                │ VPN Server    │
                └───────┬───────┘
                        │
          ┌─────────────┴─────────────┐
          │                           │
   ┌──────▼──────┐            ┌──────▼──────┐
   │  OpenVPN    │            │ WireGuard   │
   └─────────────┘            └─────────────┘
```

---

# 🚀 Features

- Multi-Protocol VPN
- Web-Based Dashboard
- User Management
- WireGuard Support
- OpenVPN Support
- Multi-Factor Authentication
- Enterprise Ready
- SSL Security
- Production Hardened
- AWS Compatible
- Cloudflare Compatible

---

# 🖥️ Server Requirements

| Resource | Recommended |
|----------|-------------|
| OS | Ubuntu 22.04 LTS |
| CPU | 2 vCPU |
| RAM | 4 GB |
| Storage | 20 GB SSD |
| Public IP | Required |
| Domain | Recommended |

---

# 🔥 Required Firewall Ports

| Service | Protocol | Port |
|----------|----------|------|
| SSH | TCP | 22 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| OpenVPN | UDP | 1194 |
| WireGuard | UDP | 51820 |

---

# 📦 Installation

---

# 1️⃣ Update Server

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

# 2️⃣ Install Dependencies

```bash
sudo apt install curl wget gnupg2 unzip nginx ufw -y
```

---

# 3️⃣ Install MongoDB

## Add MongoDB GPG Key

```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
```

---

## Add MongoDB Repository

```bash
echo "deb [ arch=amd64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

---

## Install MongoDB

```bash
sudo apt update
sudo apt install mongodb-org -y
```

---

## Start MongoDB

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

---

## Verify MongoDB

```bash
sudo systemctl status mongod
```

---

# 4️⃣ Install Pritunl

## Add Pritunl GPG Key

```bash
curl https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | \
gpg --dearmor | sudo tee /usr/share/keyrings/pritunl.gpg > /dev/null
```

---

## Add Pritunl Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/pritunl.gpg] https://repo.pritunl.com/stable/apt jammy main" | \
sudo tee /etc/apt/sources.list.d/pritunl.list
```

---

## Install Pritunl

```bash
sudo apt update
sudo apt install pritunl -y
```

---

# 5️⃣ Enable IP Forwarding

Edit sysctl configuration:

```bash
sudo nano /etc/sysctl.conf
```

Uncomment:

```bash
net.ipv4.ip_forward=1
```

Apply changes:

```bash
sudo sysctl -p
```

---

# 6️⃣ Configure Firewall

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 1194/udp
sudo ufw allow 51820/udp

sudo ufw enable
```

Verify firewall:

```bash
sudo ufw status
```

---

# 7️⃣ Start Pritunl

```bash
sudo systemctl enable pritunl
sudo systemctl start pritunl
```

Check service:

```bash
sudo systemctl status pritunl
```

---

# 8️⃣ Get Setup Key

```bash
sudo pritunl setup-key
```

Example:

```bash
setup-key: xxxxxxxxxxxxxxxxx
```

---

# 9️⃣ Get Default Admin Password

```bash
sudo pritunl default-password
```

Example:

```bash
Administrator password: password123
```

---

# 🔟 Access Pritunl Dashboard

Open in browser:

```text
https://YOUR_SERVER_IP
```

OR

```text
https://vpn.yourdomain.com
```

---

# 🌐 Configure SSL using Nginx

---

## Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

---

## Create Nginx Config

```bash
sudo nano /etc/nginx/sites-available/pritunl
```

Add:

```nginx
server {
    listen 80;
    server_name vpn.yourdomain.com;

    location / {
        proxy_pass https://127.0.0.1:9700;
        proxy_ssl_verify off;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## Enable Configuration

```bash
sudo ln -s /etc/nginx/sites-available/pritunl /etc/nginx/sites-enabled/

sudo nginx -t
sudo systemctl restart nginx
```

---

## Generate SSL Certificate

```bash
sudo certbot --nginx -d vpn.yourdomain.com
```

---

# 🔐 Production Security Hardening

---

# Disable Root Login

```bash
sudo nano /etc/ssh/sshd_config
```

Update:

```bash
PermitRootLogin no
PasswordAuthentication no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

# Install Fail2Ban

```bash
sudo apt install fail2ban -y

sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

# Enable Automatic Security Updates

```bash
sudo apt install unattended-upgrades -y

sudo dpkg-reconfigure unattended-upgrades
```

---

# 👥 Create VPN Users

Inside Pritunl Dashboard:

1. Create Organization
2. Create Users
3. Create VPN Server
4. Attach Organization
5. Start Server

---

# 📊 Useful Commands

---

## Restart Pritunl

```bash
sudo systemctl restart pritunl
```

---

## Check Pritunl Logs

```bash
sudo journalctl -u pritunl -f
```

---

## Check MongoDB Logs

```bash
sudo journalctl -u mongod -f
```

---

## Restart MongoDB

```bash
sudo systemctl restart mongod
```

---

# ☁️ AWS Security Group Rules

| Type | Protocol | Port |
|------|----------|------|
| SSH | TCP | 22 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| OpenVPN | UDP | 1194 |
| WireGuard | UDP | 51820 |

---

# 📈 Recommended Production Improvements

- AWS ALB / NLB
- Cloudflare Proxy
- S3 Backup Storage
- Monitoring using:
  - Datadog
  - Prometheus
  - Grafana
- MFA Authentication
- External MongoDB Replica Set
- Infrastructure as Code using Terraform
- Ansible Automation
- Centralized Logging

---

# 📚 Official Documentation

- https://pritunl.com/
- https://docs.pritunl.com/
- https://www.mongodb.com/docs/
- https://www.wireguard.com/

---

# 🤝 Contributing

Contributions are welcome.

Feel free to open issues or submit pull requests.

---

# 📄 License

MIT License

---

# ⭐ Support

If you found this repository useful:

- ⭐ Star this repository
- 🍴 Fork this repository
- 🛠️ Contribute improvements

---

# 👨‍💻 Author

## Aman Patel

DevOps Engineer | Cloud | Kubernetes | AWS | DevSecOps
---
