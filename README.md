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
- OpenVPN & WireGuard
- High Availability Architecture

---

# 🏗️ AWS VPC Architecture

```text
                              INTERNET
                                  │
                                  ▼
                           Route53 DNS
                                  │
                                  ▼
                         AWS Load Balancer
                              (ALB/NLB)
                                  │
                ┌────────────────────────────────┐
                │           AWS VPC              │
                │         10.0.0.0/16            │
                │                                │
                │  ┌──────────────────────────┐  │
                │  │      Public Subnet       │  │
                │  │       10.0.1.0/24        │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │  Pritunl EC2     │    │  │
                │  │  │ OpenVPN/WireGuard│    │  │
                │  │  └──────────────────┘    │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │   NAT Gateway    │    │  │
                │  │  └──────────────────┘    │  │
                │  └──────────────────────────┘  │
                │                                │
                │  ┌──────────────────────────┐  │
                │  │      Private Subnet      │  │
                │  │       10.0.2.0/24        │  │
                │  │                          │  │
                │  │  ┌──────────────────┐    │  │
                │  │  │ MongoDB Server   │    │  │
                │  │  │ Replica Set      │    │  │
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
| NAT Gateway | Internet Access for Private Subnets |
| CloudWatch | Monitoring |
| S3 | Backup Storage |
| ALB/NLB | High Availability |
| IAM Roles | Secure AWS Access |

---

# 🔐 Port Usage Documentation

| Port | Protocol | Purpose | Access |
|---|---|---|---|
| 22 | TCP | SSH Access | Admin IP Only |
| 80 | TCP | SSL Validation (Let's Encrypt) | Public |
| 443 | TCP | Pritunl Web UI/API | Public |
| 1194 | UDP | OpenVPN Traffic | Public |
| 51820 | UDP | WireGuard VPN | Public |
| 27017 | TCP | MongoDB | Private Only |
| 53 | TCP/UDP | DNS Resolution | Internal |
| 123 | UDP | NTP Time Sync | Internal |

---

# 🌐 Step 1 — Create AWS VPC

## Create VPC

| Setting | Value |
|---|---|
| Name | pritunl-vpc |
| CIDR | 10.0.0.0/16 |

Enable:
- DNS Resolution
- DNS Hostnames

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

## Public Route Table

Add Route:

| Destination | Target |
|---|---|
| 0.0.0.0/0 | Internet Gateway |

Associate:
- Public Subnet

---

## Private Route Table

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
- Allow internet access from private subnet resources

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
| 500+ | m5.large |

## Network Settings

| Setting | Value |
|---|---|
| VPC | pritunl-vpc |
| Subnet | public-subnet |
| Auto Assign Public IP | Enabled |

---

# 🔐 Step 7 — Configure Security Groups

## Pritunl Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | Your Public IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |
| OpenVPN | UDP | 1194 | 0.0.0.0/0 |
| WireGuard | UDP | 51820 | 0.0.0.0/0 |

---

## MongoDB Security Group

Allow only:

| Type | Protocol | Port | Source |
|---|---|---|---|
| MongoDB | TCP | 27017 | Pritunl Security Group |

---

# 🗄️ Step 8 — Launch MongoDB EC2

## MongoDB Instance Configuration

| Setting | Value |
|---|---|
| Subnet | private-subnet |
| Public IP | Disabled |
| Security Group | mongodb-sg |
| Storage | gp3 SSD |
| Backup | Enabled |

---

# ⚙️ Step 9 — Install MongoDB

```bash
sudo apt update

sudo apt install gnupg curl -y

curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg

echo "deb [ arch=amd64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt update

sudo apt install mongodb-org -y
```

Start MongoDB:

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

Verify:

```bash
sudo systemctl status mongod
```

---

# 🔧 Step 10 — Secure MongoDB

Edit:

```bash
sudo nano /etc/mongod.conf
```

Update:

```yaml
security:
  authorization: enabled

net:
  bindIp: 10.0.2.10
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

echo "deb [signed-by=/usr/share/keyrings/pritunl.gpg] https://repo.pritunl.com/stable/apt jammy main" | \
sudo tee /etc/apt/sources.list.d/pritunl.list

sudo apt update

sudo apt install pritunl wireguard wireguard-tools -y
```

Enable Services:

```bash
sudo systemctl enable pritunl
sudo systemctl start pritunl
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
  "mongodb_uri": "mongodb://pritunlUser:StrongPassword@10.0.2.10:27017/pritunl"
}
```

Restart:

```bash
sudo systemctl restart pritunl
```

---

# 🌍 Step 13 — Configure Route53 DNS

Create Record:

```text
vpn.yourdomain.com → Load Balancer DNS / Elastic IP
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

# 🔥 Step 17 — Configure NAT Rules

```bash
sudo iptables -t nat -A POSTROUTING \
-s 10.10.0.0/16 -o eth0 -j MASQUERADE
```

Persist Rules:

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

## Recommended Stack

| Tool | Purpose |
|---|---|
| CloudWatch | EC2 Monitoring |
| Datadog | VPN Monitoring |
| Grafana | Dashboard |
| Loki | Centralized Logs |
| Prometheus | Metrics |

---

# 💾 Step 20 — Backup Strategy

## MongoDB Backup

```bash
mongodump --gzip --archive=/backup/pritunl.gz
```

## Upload to S3

```bash
aws s3 cp /backup/pritunl.gz s3://your-backup-bucket/
```

## Cron Job

```bash
0 2 * * * /usr/bin/mongodump --gzip --archive=/backup/pritunl.gz
```

---

# 🏢 High Availability Architecture

## Recommended Production Setup

- 2+ Pritunl Nodes
- MongoDB Replica Set
- AWS ALB/NLB
- Route53 Health Checks
- Multi-AZ Deployment
- Automated Snapshots

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
- Enable Automatic Security Updates
- Restrict MongoDB to Private Network Only

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
- [x] CloudTrail Enabled
- [x] GuardDuty Enabled

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
