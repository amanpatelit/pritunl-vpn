# 🚀 Pritunl VPN Production Setup Guide on AWS/Linux

![Pritunl VPN Production Setup](./assets/pritunl-production-setup.png)

> Secure • Scalable • High Availability • Production Ready

---

# 📌 Overview

This repository contains a complete production-grade setup guide for deploying **Pritunl VPN** on **AWS/Linux** using:

- Ubuntu 22.04 LTS
- MongoDB 7
- AWS EC2
- OpenVPN + WireGuard
- SSL with Let's Encrypt
- High Availability Architecture
- Monitoring & Backup Strategy

---

# 🏗️ Architecture

```text
Users
   │
   ▼
Internet
   │
   ▼
AWS Load Balancer (ALB/NLB)
   │
   ├──────────────┐
   ▼              ▼
Pritunl Node 1   Pritunl Node 2
   │              │
   └──────┬───────┘
          ▼
 MongoDB Replica Set
```

---

# 📋 Recommended Stack

| Component | Recommendation |
|---|---|
| OS | Ubuntu 22.04 LTS |
| VPN | Pritunl |
| Database | MongoDB 7 |
| Cloud | AWS EC2 |
| SSL | Let's Encrypt |
| Reverse Proxy | Nginx (Optional) |
| Authentication | Google OAuth / Okta / Azure AD |
| Monitoring | Datadog / CloudWatch |
| Backup | MongoDB Backup to S3 |

---

# 🔐 AWS Security Group Rules

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH |
| 80 | TCP | SSL Validation |
| 443 | TCP | Pritunl UI/API |
| 1194 | UDP | OpenVPN |
| 51820 | UDP | WireGuard |

---

# ⚙️ Server Requirements

| Users | AWS Instance |
|---|---|
| 50–100 | t3.medium |
| 100–500 | t3.large |
| 500+ | m5.large |

---

# 🛠️ Installation Steps

# 1️⃣ Install MongoDB

## Add MongoDB Repository

```bash
sudo apt update

sudo apt install gnupg curl -y

curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg

echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt update

sudo apt install mongodb-org -y
```

## Start MongoDB

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
sudo systemctl status mongod
```

---

# 2️⃣ Install Pritunl

## Add Repository

```bash
curl -fsSL https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/pritunl.gpg

echo "deb [signed-by=/usr/share/keyrings/pritunl.gpg] \
https://repo.pritunl.com/stable/apt jammy main" | \
sudo tee /etc/apt/sources.list.d/pritunl.list
```

## Install Pritunl

```bash
sudo apt update
sudo apt install pritunl -y
```

## Start Pritunl

```bash
sudo systemctl enable pritunl
sudo systemctl start pritunl
```

---

# 3️⃣ Configure SSL (Production Mandatory)

## Install Certbot

```bash
sudo apt install certbot -y
```

## Generate SSL Certificate

```bash
sudo systemctl stop pritunl

sudo certbot certonly --standalone \
-d vpn.yourdomain.com
```

Certificate Paths:

```bash
/etc/letsencrypt/live/vpn.yourdomain.com/fullchain.pem
/etc/letsencrypt/live/vpn.yourdomain.com/privkey.pem
```

---

# 4️⃣ Configure Pritunl SSL

Edit:

```bash
sudo nano /etc/pritunl.conf
```

Add:

```json
{
  "ssl": true,
  "ssl_cert": "/etc/letsencrypt/live/vpn.yourdomain.com/fullchain.pem",
  "ssl_key": "/etc/letsencrypt/live/vpn.yourdomain.com/privkey.pem"
}
```

Restart:

```bash
sudo systemctl restart pritunl
```

---

# 5️⃣ Access Web UI

Open:

```text
https://vpn.yourdomain.com
```

Get setup key:

```bash
sudo pritunl setup-key
```

Get default password:

```bash
sudo pritunl default-password
```

---

# 🌐 Enable IP Forwarding

Edit:

```bash
sudo nano /etc/sysctl.conf
```

Uncomment:

```bash
net.ipv4.ip_forward=1
```

Apply:

```bash
sudo sysctl -p
```

---

# 🔥 Configure Firewall & NAT

## UFW Rules

```bash
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw allow 1194/udp
sudo ufw allow 51820/udp

sudo ufw enable
```

## NAT Configuration

```bash
sudo iptables -t nat -A POSTROUTING \
-s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

Persist:

```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

---

# 🔒 Security Hardening

## Recommended Best Practices

- Disable password SSH login
- Use SSH Keys only
- Enable MFA in Pritunl
- Install Fail2Ban
- Restrict Admin UI access
- Enable automatic updates
- Centralize logs

## Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

---

# 🏢 High Availability Setup

## Recommended Architecture

- 2+ Pritunl Nodes
- MongoDB Replica Set
- AWS ALB/NLB
- Route53 Failover

---

# 💾 Backup Strategy

## MongoDB Backup

```bash
mongodump --out /backup/mongo
```

## Cron Job

```bash
0 2 * * * mongodump --out /backup/mongo-$(date +\%F)
```

Store backups in:

- Amazon S3
- Glacier

---

# 📊 Monitoring & Logging

| Tool | Purpose |
|---|---|
| Datadog | Infra Monitoring |
| CloudWatch | AWS Metrics |
| Grafana | Visualization |
| Loki | Logs |
| Prometheus | Metrics |

## Important Logs

```bash
journalctl -u pritunl -f
```

```bash
journalctl -u mongod -f
```

---

# 🔑 Authentication Integrations

Supported Providers:

- Google OAuth
- Azure AD
- Okta
- LDAP
- SAML
- OneLogin

---

# 💻 VPN Client Support

Supported Clients:

- OpenVPN
- WireGuard
- Pritunl Client

Official Website:

https://pritunl.com/

---

# 🚨 Common Issues

| Issue | Fix |
|---|---|
| TLS Handshake Failed | Open UDP Ports |
| No Internet Access | Fix NAT/IP Forwarding |
| Slow VPN | Adjust MTU |
| SSL Invalid | Renew Certbot Certificate |
| MongoDB Failed | Check Firewall/Bind IP |

---

# 🔄 Auto Renew SSL

Edit Crontab:

```bash
sudo crontab -e
```

Add:

```bash
0 3 * * * certbot renew --quiet && systemctl restart pritunl
```

---

# ✅ Production Checklist

- [x] SSL Enabled
- [x] MFA Enabled
- [x] MongoDB Replica Set
- [x] Monitoring Configured
- [x] Automated Backups
- [x] Fail2Ban Installed
- [x] SSH Key Authentication
- [x] WireGuard Enabled
- [x] Security Groups Restricted

---

# 📚 Official Documentation

- https://docs.pritunl.com/
- https://github.com/pritunl/pritunl
- https://www.mongodb.com/docs/

---

# 🤝 Contributing

Feel free to contribute by creating pull requests, improving documentation, or suggesting enhancements.

---

# 📄 License

MIT License

---

# 👨‍💻 Author

**Aman Patel**  
DevOps Engineer | Cloud & Kubernetes Specialist

🌐 Website: https://amanops.com

---
