# 🚀 Pritunl VPN Production Setup on Ubuntu 22.04

![Pritunl VPN Production Setup](./assets/pritunl-production-setup.png)

> Production-Ready Pritunl VPN Deployment using Ubuntu 22.04, OpenVPN, WireGuard & MongoDB

---

# 📌 Overview

This guide explains how to deploy **Pritunl VPN** securely on **Ubuntu 22.04** for production environments.

Features included:

- OpenVPN + WireGuard
- MongoDB 7
- SSL/TLS Encryption
- UFW Firewall
- NAT Configuration
- IP Forwarding
- Fail2Ban Protection
- Automated Backups
- Monitoring Ready
- Production Security Hardening

---

# 🖥️ Server Requirements

| Users | Recommended Instance |
|---|---|
| 50–100 | 2 vCPU / 4GB RAM |
| 100–500 | 4 vCPU / 8GB RAM |
| 500+ | 8 vCPU / 16GB RAM |

---

# 🌐 Required Ports

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH Access |
| 80 | TCP | SSL Validation |
| 443 | TCP | Pritunl Web UI/API |
| 1194 | UDP | OpenVPN |
| 51820 | UDP | WireGuard |
| 27017 | TCP | MongoDB (Private Only) |

---

# 🔐 Security Recommendations

## Recommended

- Use SSH Key Authentication
- Disable Root Login
- Enable MFA
- Install Fail2Ban
- Use SSL Certificates
- Restrict MongoDB Access
- Enable Automatic Updates

---

# 📦 Step 1 — Update Ubuntu

```bash
sudo apt update -y
sudo apt upgrade -y
```

---

# 📦 Step 2 — Install Required Packages

```bash
sudo apt install -y curl gnupg wget unzip ca-certificates software-properties-common
```

---

# 🗄️ Step 3 — Install MongoDB 7

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

Expected:

```text
active (running)
```

---

# 🔒 Step 4 — Secure MongoDB

Edit configuration:

```bash
sudo nano /etc/mongod.conf
```

Update:

```yaml
security:
  authorization: enabled

net:
  bindIp: 127.0.0.1
```

Restart MongoDB:

```bash
sudo systemctl restart mongod
```

---

# 🚀 Step 5 — Install Pritunl

## Add Pritunl GPG Key

```bash
curl -fsSL https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/pritunl.gpg
```

---

## Add Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/pritunl.gpg] https://repo.pritunl.com/stable/apt jammy main" | \
sudo tee /etc/apt/sources.list.d/pritunl.list
```

---

## Install WireGuard

```bash
sudo apt install wireguard wireguard-tools -y
```

---

## Install Pritunl

```bash
sudo apt update

sudo apt install pritunl -y
```

---

## Start Pritunl

```bash
sudo systemctl enable pritunl
sudo systemctl start pritunl
```

---

# 🌐 Step 6 — Configure Firewall

## Allow Required Ports

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 1194/udp
sudo ufw allow 51820/udp
```

Enable Firewall:

```bash
sudo ufw --force enable
```

Check Status:

```bash
sudo ufw status
```

---

# 🌐 Step 7 — Enable IP Forwarding

Edit:

```bash
sudo nano /etc/sysctl.conf
```

Enable:

```bash
net.ipv4.ip_forward=1
```

Apply:

```bash
sudo sysctl -p
```

---

# 🔥 Step 8 — Configure NAT

Find Interface:

```bash
ip route | grep default
```

Example Interface:

```text
eth0
```

Add NAT Rule:

```bash
sudo iptables -t nat -A POSTROUTING \
-s 10.10.0.0/16 -o eth0 -j MASQUERADE
```

Install Persistence:

```bash
sudo apt install iptables-persistent -y
```

Save Rules:

```bash
sudo netfilter-persistent save
```

---

# 🔒 Step 9 — Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

Enable Service:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

# 🌍 Step 10 — Configure Domain

Create DNS Record:

```text
vpn.yourdomain.com → SERVER_PUBLIC_IP
```

---

# 🔐 Step 11 — Configure SSL

## Install Certbot

```bash
sudo apt install certbot -y
```

---

## Generate SSL Certificate

```bash
sudo certbot certonly --standalone \
-d vpn.yourdomain.com
```

---

# 🛡️ Step 12 — Configure SSL in Pritunl

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

# 🌐 Step 13 — Access Pritunl Dashboard

Open:

```text
https://vpn.yourdomain.com
```

or

```text
https://SERVER_PUBLIC_IP
```

---

## Get Setup Key

```bash
sudo pritunl setup-key
```

---

## Get Default Password

```bash
sudo pritunl default-password
```

---

# 👥 Step 14 — Create VPN Server

From Dashboard:

1. Create Organization
2. Create User
3. Create Server
4. Attach Organization
5. Start Server

---

# ⚙️ Recommended VPN Settings

| Setting | Value |
|---|---|
| Protocol | UDP |
| Cipher | AES-256-GCM |
| Hash | SHA256 |
| DNS | 1.1.1.1 |
| WireGuard | Enabled |

---

# 📊 Monitoring & Logging

## Recommended Tools

| Tool | Purpose |
|---|---|
| Datadog | Monitoring |
| CloudWatch | Metrics |
| Grafana | Dashboard |
| Loki | Logs |

---

## Important Logs

```bash
journalctl -u pritunl -f
```

```bash
journalctl -u mongod -f
```

---

# 💾 Backup Strategy

## MongoDB Backup

```bash
mongodump --gzip --archive=/backup/pritunl.gz
```

---

## Restore Backup

```bash
mongorestore --gzip --archive=/backup/pritunl.gz
```

---

# 🔄 SSL Auto Renewal

Open Crontab:

```bash
sudo crontab -e
```

Add:

```bash
0 3 * * * certbot renew --quiet && systemctl restart pritunl
```

---

# 🔒 Security Hardening

## Recommended

- Disable Password SSH
- Disable Root Login
- Restrict SSH Access
- Enable MFA
- Enable Automatic Updates
- Restrict MongoDB to Localhost
- Use Private Networking
- Regular Backups

---

# 🚨 Common Issues

| Issue | Solution |
|---|---|
| Cannot Access UI | Check Firewall |
| MongoDB Failed | Check Service Status |
| SSL Error | Verify Domain DNS |
| VPN Connected but No Internet | Check NAT & IP Forwarding |
| WireGuard Not Working | Open UDP 51820 |

---

# ✅ Production Checklist

- [x] SSL Enabled
- [x] Firewall Configured
- [x] WireGuard Enabled
- [x] OpenVPN Enabled
- [x] Fail2Ban Installed
- [x] IP Forwarding Enabled
- [x] NAT Configured
- [x] MongoDB Secured
- [x] MFA Enabled
- [x] Monitoring Enabled

---

# 📚 Official Documentation

- https://docs.pritunl.com/
- https://github.com/pritunl/pritunl
- https://www.mongodb.com/docs/

---

# 👨‍💻 Author

**Aman Patel**  
DevOps Engineer | AWS | Kubernetes | DevSecOps

---
