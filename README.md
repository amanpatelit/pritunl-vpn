# ☁️ Pritunl VPN Production Setup on AWS VPC

![Pritunl VPN Production Setup](./assets/pritunl-production-setup.png)

> Production-Ready VPN Infrastructure using AWS VPC, EC2, MongoDB, OpenVPN & WireGuard

---

# 📌 Overview

This guide explains how to deploy **Pritunl VPN** securely inside an **AWS VPC** with:

- Public & Private Subnets
- Internet Gateway
- NAT Gateway
- EC2 Instances
- Security Groups
- Route Tables
- MongoDB
- SSL Certificates
- High Availability Architecture

---

# 🏗️ AWS VPC Architecture

```text
                              INTERNET
                                  │
                                  ▼
                         Internet Gateway
                                  │
                ┌────────────────────────────────┐
                │           AWS VPC              │
                │        10.0.0.0/16             │
                │                                │
                │  ┌──────────────────────────┐  │
                │  │      Public Subnet       │  │
                │  │      10.0.1.0/24         │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │  Pritunl EC2     │    │  │
                │  │  │ OpenVPN/WG VPN   │    │  │
                │  │  └──────────────────┘    │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │   NAT Gateway    │    │  │
                │  │  └──────────────────┘    │  │
                │  └──────────────────────────┘  │
                │                                │
                │  ┌──────────────────────────┐  │
                │  │      Private Subnet      │  │
                │  │      10.0.2.0/24         │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │ MongoDB Server   │    │  │
                │  │  └──────────────────┘    │  │
                │  └──────────────────────────┘  │
                └────────────────────────────────┘
```

---

# 📋 Recommended AWS Components

| AWS Service | Purpose |
|---|---|
| VPC | Network Isolation |
| EC2 | VPN Server |
| Security Groups | Firewall |
| Route53 | DNS |
| ACM / Let's Encrypt | SSL |
| NAT Gateway | Private Internet Access |
| CloudWatch | Monitoring |
| S3 | Backup Storage |
| ALB/NLB | High Availability |

---

# 🌐 Step 1 — Create AWS VPC

## Create VPC

| Setting | Value |
|---|---|
| Name | pritunl-vpc |
| CIDR | 10.0.0.0/16 |

---

# 🌍 Step 2 — Create Subnets

## Public Subnet

| Setting | Value |
|---|---|
| Name | public-subnet |
| CIDR | 10.0.1.0/24 |
| AZ | ap-south-1a |

## Private Subnet

| Setting | Value |
|---|---|
| Name | private-subnet |
| CIDR | 10.0.2.0/24 |
| AZ | ap-south-1a |

---

# 🚪 Step 3 — Create Internet Gateway

## Create IGW

```text
Name: pritunl-igw
```

Attach to:

```text
pritunl-vpc
```

---

# 🛣️ Step 4 — Configure Route Tables

# Public Route Table

Add Route:

| Destination | Target |
|---|---|
| 0.0.0.0/0 | Internet Gateway |

Associate:
- Public Subnet

---

# Private Route Table

Add Route:

| Destination | Target |
|---|---|
| 0.0.0.0/0 | NAT Gateway |

Associate:
- Private Subnet

---

# 🔥 Step 5 — Create NAT Gateway

Deploy NAT Gateway inside:

```text
Public Subnet
```

Allocate:
- Elastic IP

Purpose:
- Allow private subnet internet access

---

# 🖥️ Step 6 — Launch EC2 for Pritunl

## Recommended AMI

```text
Ubuntu 22.04 LTS
```

## Instance Type

| Users | Instance |
|---|---|
| 50–100 | t3.medium |
| 100–500 | t3.large |

## Network Settings

| Setting | Value |
|---|---|
| VPC | pritunl-vpc |
| Subnet | public-subnet |
| Auto Assign Public IP | Enabled |

---

# 🔐 Step 7 — Configure Security Groups

# Pritunl Security Group

| Port | Protocol | Source |
|---|---|---|
| 22 | TCP | Your IP |
| 80 | TCP | 0.0.0.0/0 |
| 443 | TCP | 0.0.0.0/0 |
| 1194 | UDP | 0.0.0.0/0 |
| 51820 | UDP | 0.0.0.0/0 |

---

# 🗄️ Step 8 — Launch MongoDB EC2

## MongoDB Instance

| Setting | Value |
|---|---|
| Subnet | private-subnet |
| Public IP | Disabled |
| Security Group | mongodb-sg |

---

# MongoDB Security Group

Allow only:

| Port | Protocol | Source |
|---|---|---|
| 27017 | TCP | Pritunl Security Group |

---

# ⚙️ Step 9 — Install MongoDB

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

Start MongoDB:

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

---

# 🔧 Step 10 — Configure MongoDB Bind IP

Edit:

```bash
sudo nano /etc/mongod.conf
```

Update:

```yaml
bindIp: 0.0.0.0
```

Restart:

```bash
sudo systemctl restart mongod
```

---

# 🚀 Step 11 — Install Pritunl

```bash
curl -fsSL https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/pritunl.gpg

echo "deb [signed-by=/usr/share/keyrings/pritunl.gpg] \
https://repo.pritunl.com/stable/apt jammy main" | \
sudo tee /etc/apt/sources.list.d/pritunl.list

sudo apt update

sudo apt install pritunl -y
```

---

# 🔗 Step 12 — Connect Pritunl to MongoDB

Edit:

```bash
sudo nano /etc/pritunl.conf
```

Example:

```json
{
  "mongodb_uri": "mongodb://10.0.2.10:27017/pritunl"
}
```

Restart:

```bash
sudo systemctl restart pritunl
```

---

# 🌍 Step 13 — Configure DNS

Create Route53 Record:

```text
vpn.yourdomain.com → Elastic IP
```

---

# 🔒 Step 14 — Configure SSL

Install Certbot:

```bash
sudo apt install certbot -y
```

Generate SSL:

```bash
sudo certbot certonly --standalone \
-d vpn.yourdomain.com
```

---

# 🧩 Step 15 — Configure Pritunl SSL

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

# 🌐 Step 16 — Enable IP Forwarding

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

# 🔥 Step 17 — Configure NAT Rules

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

# 🌍 Step 18 — Access Pritunl Dashboard

Open:

```text
https://vpn.yourdomain.com
```

Get Setup Key:

```bash
sudo pritunl setup-key
```

Get Default Password:

```bash
sudo pritunl default-password
```

---

# 📊 Step 19 — Monitoring

## Recommended

| Tool | Purpose |
|---|---|
| CloudWatch | EC2 Monitoring |
| Datadog | VPN Monitoring |
| Grafana | Dashboard |
| Loki | Logs |

---

# 💾 Step 20 — Backup Strategy

## MongoDB Backup

```bash
mongodump --out /backup/mongo
```

## Upload to S3

```bash
aws s3 cp /backup s3://your-backup-bucket/ --recursive
```

---

# 🏢 High Availability Architecture

## Recommended Setup

- 2+ Pritunl Nodes
- MongoDB Replica Set
- AWS ALB/NLB
- Route53 Failover
- Multi-AZ Deployment

---

# 🔒 Security Hardening

## Recommended

- Disable Password SSH
- Enable MFA
- Install Fail2Ban
- Restrict Admin Access
- Use IAM Roles
- Enable CloudTrail
- Enable GuardDuty

---

# ✅ Production Checklist

- [x] VPC Configured
- [x] Public/Private Subnets
- [x] NAT Gateway Configured
- [x] SSL Enabled
- [x] WireGuard Enabled
- [x] MongoDB Secured
- [x] Automated Backups
- [x] Monitoring Enabled
- [x] MFA Enabled
- [x] Security Groups Restricted

---

# 📚 Official Documentation

- https://docs.pritunl.com/
- https://github.com/pritunl/pritunl
- https://docs.aws.amazon.com/vpc/
- https://www.mongodb.com/docs/

---

# 👨‍💻 Author

**Aman Patel**  
DevOps Engineer | Kubernetes | AWS | DevSecOps

---
