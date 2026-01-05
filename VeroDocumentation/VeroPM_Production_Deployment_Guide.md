# VeroPM Production Deployment Guide

**Platform:** VeroPM Enterprise Project Management System  
**Version:** 1.0  
**Last Updated:** January 2026  
**Document Owner:** Hassan (Product Owner & Technical Lead)

---

## Table of Contents

1. [Overview](#overview)
2. [Option 1: Azure Cloud Deployment](#option-1-azure-cloud-deployment)
3. [Option 2: Dedicated Physical Server](#option-2-dedicated-physical-server)
4. [Software Installation Guide](#software-installation-guide)
5. [Network Configuration](#network-configuration)
6. [Deployment Architecture](#deployment-architecture)
7. [Cost Analysis](#cost-analysis)
8. [Recommendations](#recommendations)

---

## Overview

This document provides comprehensive deployment specifications for VeroPM platform in production environment. The platform consists of:

- **Backend:** .NET Core 8.0 API
- **Frontend:** ReactJS 19 SPA
- **Primary Database:** MS SQL Server 2022
- **Graph Database:** Neo4j 5.x Community Edition (GraphRAG)
- **Real-time:** SignalR for collaboration features
- **Caching:** Redis for session management
- **AI Orchestration:** AIBOL with multi-LLM integration

### Key Technical Stack
- ReactJS 19
- .NET Core 8.0 / C# 12
- MS SQL Server 2022
- Neo4j Community Edition
- SignalR
- Shadcn/Tailwind UI

---

## Option 1: Azure Cloud Deployment

### Azure Services Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Azure Front Door                      │
│                  (Global Load Balancer)                  │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────┴──────────┐
         │                      │
    ┌────▼─────┐         ┌─────▼────┐
    │ Frontend │         │ Backend  │
    │ App Svc  │         │ App Svc  │
    │ (React)  │         │ (.NET)   │
    └──────────┘         └────┬─────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
         │ Azure   │    │ Neo4j   │    │ Redis   │
         │ SQL DB  │    │ on VM   │    │ Cache   │
         └─────────┘    └─────────┘    └─────────┘
                              │
                         ┌────▼────┐
                         │  Blob   │
                         │ Storage │
                         └─────────┘
```

### 1. App Services

#### Backend API (.NET Core 8.0)
```yaml
Service: Azure App Service
Plan: Premium P2V3
Specs:
  vCPUs: 4 cores
  RAM: 16 GB
  Storage: 250 GB
  OS: Linux (recommended for cost) or Windows
  Runtime: .NET 8.0
  Region: Central US / West Europe (choose based on users)
Features:
  - Auto-scaling enabled
  - Deployment slots (staging + production)
  - Application Insights integration
  - Managed SSL certificates
Estimated Cost: $292/month
```

#### Frontend (ReactJS 19)
```yaml
Service: Azure App Service
Plan: Premium P1V3
Specs:
  vCPUs: 2 cores
  RAM: 8 GB
  Storage: 100 GB
  Runtime: Node.js 20.x LTS
  Build: Static files (npm run build)
Features:
  - CDN integration
  - Custom domain support
  - Auto-scaling
  - Deployment slots
Estimated Cost: $146/month
```

### 2. Database Services

#### Azure SQL Database
```yaml
Service: Azure SQL Database
Tier: Standard S3 (minimum) / Premium P2 (recommended)
Specs:
  DTUs: 100 (Standard) / vCore: 4-8 (Premium)
  Storage: 500 GB (auto-expand enabled)
  Version: SQL Server 2022 compatible
  Backup: Geo-redundant, 35-day retention
Features:
  - Point-in-time restore
  - Automated backups
  - Advanced threat protection
  - Query performance insights
  - Automatic tuning
Networking:
  - Private endpoint (VNet integration)
  - Firewall rules for App Services
Estimated Cost: $300/month (S3)
```

### 3. Neo4j Deployment on Azure VM

#### Virtual Machine Specifications
```yaml
Service: Azure Virtual Machine
VM Size: Standard_D4s_v3
Specs:
  vCPUs: 4 cores
  RAM: 16 GB (minimum for production GraphRAG)
  Storage: 512 GB Premium SSD (P20)
  OS: Ubuntu 22.04 LTS
  Neo4j: Community Edition 5.x
Configuration:
  Disk Type: Premium SSD (better IOPS)
  Network: Accelerated networking enabled
  Availability: Availability Zone deployment
Security:
  - Network Security Group (NSG)
  - Private IP only (no public access)
  - VNet integration with App Services
Neo4j Memory Settings:
  heap.initial: 8GB
  heap.max: 8GB
  pagecache: 4GB
Estimated Cost: $175/month
```

#### Neo4j Installation Script (Ubuntu)
```bash
#!/bin/bash
# Neo4j Community Edition Installation

# Add Neo4j repository
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable latest' | sudo tee /etc/apt/sources.list.d/neo4j.list

# Update and install
sudo apt-get update
sudo apt-get install -y neo4j=1:5.15.0

# Configure memory settings
sudo tee -a /etc/neo4j/neo4j.conf << EOF
server.memory.heap.initial_size=8g
server.memory.heap.max_size=8g
server.memory.pagecache.size=4g
server.default_listen_address=0.0.0.0
dbms.connector.bolt.listen_address=:7687
dbms.connector.http.listen_address=:7474
EOF

# Start Neo4j
sudo systemctl enable neo4j
sudo systemctl start neo4j
```

### 4. Storage Services

#### Azure Blob Storage
```yaml
Service: Azure Blob Storage
Account Type: StorageV2 (general purpose v2)
Performance: Standard
Replication: RA-GRS (Read-Access Geo-Redundant)
Capacity: 1 TB initial (expandable)
Access Tiers:
  - Hot: Frequently accessed files
  - Cool: Backups and archives
Features:
  - Lifecycle management
  - Soft delete (30 days)
  - Versioning enabled
  - Private endpoint access
Use Cases:
  - User uploads (documents, images)
  - File attachments
  - Database backups
  - Application logs
Estimated Cost: $20/month (1TB Hot)
```

### 5. Cache & Session Management

#### Azure Cache for Redis
```yaml
Service: Azure Cache for Redis
Tier: Standard C2
Specs:
  Memory: 2.5 GB
  SLA: 99.9%
  Persistence: RDB snapshots
Features:
  - SSL/TLS encryption
  - Virtual network integration
  - Geo-replication (optional)
Use Cases:
  - SignalR backplane (real-time chat)
  - Session state management
  - Application caching
  - Rate limiting
Estimated Cost: $125/month
```

### 6. Security & Networking

#### Azure Application Gateway (WAF v2)
```yaml
Service: Application Gateway
Tier: WAF V2 (Web Application Firewall)
Capacity: 2 instances (minimum)
Features:
  - SSL/TLS termination
  - URL-based routing
  - DDoS protection
  - OWASP top 10 protection
  - Custom rules
  - HTTP/2 support
SSL Certificates:
  - Azure-managed certificates
  - Custom domain support
Estimated Cost: $300/month
```

#### Azure Key Vault
```yaml
Service: Azure Key Vault
Tier: Standard
Use Cases:
  - Connection strings
  - API keys (OpenAI, Anthropic, Google)
  - SSL certificates
  - Database credentials
  - JWT secrets
Features:
  - RBAC integration
  - Audit logging
  - Soft delete
  - Purge protection
Estimated Cost: $5/month
```

### 7. Monitoring & DevOps

#### Application Insights
```yaml
Service: Application Insights
Data Retention: 90 days
Features:
  - Application performance monitoring
  - Live metrics
  - Failure tracking
  - Custom events and metrics
  - Distributed tracing
Alerts:
  - Response time > 2s
  - Error rate > 1%
  - CPU usage > 80%
  - Memory usage > 85%
Estimated Cost: $50/month (5GB data)
```

#### Azure DevOps / GitHub Actions
```yaml
CI/CD Pipeline:
  - Automated testing
  - Code quality checks
  - Security scanning
  - Automated deployment
Environments:
  - Development
  - Staging
  - Production
Estimated Cost: Free (GitHub Actions) / $6/user (Azure DevOps)
```

### Azure Total Monthly Cost Estimate

| Service | Tier/Size | Monthly Cost |
|---------|-----------|--------------|
| Backend App Service | P2V3 | $292 |
| Frontend App Service | P1V3 | $146 |
| Azure SQL Database | S3/P2 | $300 |
| Neo4j VM | D4s_v3 | $175 |
| Blob Storage | 1TB Hot | $20 |
| Redis Cache | C2 | $125 |
| Application Gateway | WAF V2 | $300 |
| Key Vault | Standard | $5 |
| Application Insights | 5GB | $50 |
| **TOTAL** | | **~$1,413/month** |

*Costs exclude data transfer and bandwidth charges (~$50-100/month expected)*

---

## Option 2: Dedicated Physical Server

### Server Specifications

#### Option A: All-in-One Server (Minimum Production Specs)

```yaml
Server Type: Dell PowerEdge R450 / HP ProLiant DL360 Gen10+
CPU: Intel Xeon E-2388G (8 cores @ 3.2GHz) or AMD EPYC 7443P (24 cores)
RAM: 64 GB DDR4 ECC (4x 16GB modules)
Storage Configuration:
  OS & Applications:
    - 2x 500GB NVMe SSD (RAID 1 mirror)
    - Mount: /opt/veropm
  Databases (SQL Server + Neo4j):
    - 2x 2TB SATA SSD (RAID 1 mirror)
    - Mount: /data
Network:
  - 1 Gbps dedicated port
  - 10TB monthly bandwidth
Power:
  - Redundant PSU (2x 800W)
  - UPS backup (optional but recommended)
RAID Controller: Hardware RAID with battery backup
Monthly Cost: $300-400/month (rental/colocation)
```

#### Option B: High-Performance Server (Recommended)

```yaml
Server Type: Dell PowerEdge R650 / HP ProLiant DL380 Gen10+
CPU: Intel Xeon Gold 6338 (32 cores @ 2.0GHz) or AMD EPYC 7543 (32 cores @ 2.8GHz)
RAM: 128 GB DDR4 ECC (8x 16GB modules)
Storage Configuration:
  OS & Applications:
    - 2x 1TB NVMe SSD (RAID 1 mirror)
    - Mount: /opt/veropm
  Databases:
    - 4x 2TB SSD (RAID 10 for performance + redundancy)
    - Mount: /data
  Backups:
    - 2x 4TB HDD (RAID 1 mirror)
    - Mount: /backups
Network:
  - 10 Gbps dedicated port
  - Unlimited bandwidth or 50TB/month
Power:
  - Redundant PSU (2x 1200W)
  - UPS with 2-hour runtime
  - Generator backup (datacenter facility)
RAID Controller: Hardware RAID with 2GB cache, BBU
Remote Management: iDRAC / iLO for remote access
Monthly Cost: $500-700/month (rental/colocation)
```

### Storage Layout Best Practices

```
/
├── /boot              # 1 GB - Boot partition
├── /                  # 100 GB - Root filesystem
├── /opt/veropm         # NVMe RAID 1
│   ├── /backend       # .NET application
│   ├── /frontend      # React build files
│   ├── /uploads       # User file uploads (symlink to /data/uploads)
│   └── /logs          # Application logs
├── /data              # SSD RAID 1/10
│   ├── /mssql         # SQL Server data files
│   ├── /neo4j         # Neo4j graph database
│   ├── /redis         # Redis persistence
│   └── /uploads       # User uploads (actual location)
└── /backups           # HDD RAID 1
    ├── /daily         # Daily automated backups
    ├── /weekly        # Weekly backups
    └── /monthly       # Monthly archival backups
```

---

## Software Installation Guide

### Operating System Selection

**Recommended:** Ubuntu Server 22.04 LTS (64-bit)

**Reasons:**
- Long-term support (until 2027)
- Stable and mature
- Excellent Docker support
- Large community
- Free (no licensing costs)
- Better performance for .NET Core
- Native support for all required software

**Alternative:** CentOS Stream 9 / Rocky Linux 9

### Base System Setup

```bash
#!/bin/bash
# Initial server setup

# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Install essential tools
sudo apt-get install -y \
    curl \
    wget \
    git \
    vim \
    htop \
    net-tools \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release

# Set timezone
sudo timedatectl set-timezone UTC

# Configure swap (16GB recommended)
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 1. .NET Core 8.0 Runtime & SDK

```bash
#!/bin/bash
# Install .NET 8.0

# Add Microsoft package repository
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

# Install .NET SDK and Runtime
sudo apt-get update
sudo apt-get install -y dotnet-sdk-8.0 dotnet-runtime-8.0 aspnetcore-runtime-8.0

# Verify installation
dotnet --version
# Expected output: 8.0.x
```

**Version Required:** .NET 8.0.1 or later  
**Components:**
- ASP.NET Core Runtime 8.0
- .NET Runtime 8.0
- .NET SDK 8.0 (for builds)

### 2. Node.js & npm

```bash
#!/bin/bash
# Install Node.js 20.x LTS

# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js and npm
sudo apt-get install -y nodejs

# Verify installation
node --version  # Expected: v20.11.x or later
npm --version   # Expected: 10.x or later

# Install global packages
sudo npm install -g pm2 yarn
```

**Version Required:** Node.js 20.11+ LTS  
**Global Packages:**
- PM2 (process manager)
- Yarn (package manager, optional)

### 3. Nginx (Reverse Proxy & Web Server)

```bash
#!/bin/bash
# Install Nginx

sudo apt-get install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify installation
nginx -v
# Expected: nginx/1.24.x or later
```

**Version Required:** 1.24.x or later  
**Purpose:**
- Reverse proxy for .NET backend
- Static file serving for React frontend
- SSL/TLS termination
- Load balancing (future scaling)
- Gzip compression
- Rate limiting

**Configuration File Locations:**
- Main config: `/etc/nginx/nginx.conf`
- Site configs: `/etc/nginx/sites-available/`
- Enabled sites: `/etc/nginx/sites-enabled/`

### 4. MS SQL Server 2022

```bash
#!/bin/bash
# Install Microsoft SQL Server 2022 on Ubuntu

# Add Microsoft SQL Server repository
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/22.04/mssql-server-2022.list)"

# Install SQL Server
sudo apt-get update
sudo apt-get install -y mssql-server

# Run SQL Server setup
sudo /opt/mssql/bin/mssql-conf setup
# Choose: 2) Developer (free) or 3) Standard
# Set SA password (strong password required)

# Enable and start service
sudo systemctl enable mssql-server
sudo systemctl start mssql-server

# Install SQL Server command-line tools
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list | sudo tee /etc/apt/sources.list.d/msprod.list
sudo apt-get update
sudo apt-get install -y mssql-tools unixodbc-dev

# Add tools to PATH
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc

# Verify installation
sqlcmd -S localhost -U SA -P 'YourStrongPassword'
```

**Edition Options:**
- Developer Edition (free, full features, not for production)
- Standard Edition (production licensed)
- Express Edition (free, limited to 10GB database size)

**Recommended: Standard Edition for production**

**Memory Configuration:**
```sql
-- Set maximum server memory (32GB example)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'max server memory', 32768;
RECONFIGURE;

-- Set cost threshold for parallelism
EXEC sp_configure 'cost threshold for parallelism', 50;
RECONFIGURE;

-- Set max degree of parallelism (number of cores)
EXEC sp_configure 'max degree of parallelism', 8;
RECONFIGURE;
```

**Configuration File:** `/var/opt/mssql/mssql.conf`

**Data Directory:** `/var/opt/mssql/data/`  
**Log Directory:** `/var/opt/mssql/log/`  
**Backup Directory:** `/var/opt/mssql/backup/`

### 5. Neo4j Community Edition

```bash
#!/bin/bash
# Install Neo4j Community Edition 5.x

# Add Neo4j repository
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable latest' | sudo tee /etc/apt/sources.list.d/neo4j.list

# Update and install Neo4j
sudo apt-get update
sudo apt-get install -y neo4j=1:5.15.0

# Configure Neo4j memory settings
sudo tee -a /etc/neo4j/neo4j.conf << 'EOF'
# Memory Settings
server.memory.heap.initial_size=8g
server.memory.heap.max_size=8g
server.memory.pagecache.size=4g

# Network Settings
server.default_listen_address=0.0.0.0
dbms.connector.bolt.listen_address=:7687
dbms.connector.http.listen_address=:7474

# Security (change default password on first login)
dbms.security.auth_enabled=true

# Performance
dbms.memory.transaction.total.max=2g
EOF

# Start and enable Neo4j
sudo systemctl enable neo4j
sudo systemctl start neo4j

# Wait for Neo4j to start
sleep 10

# Check status
sudo systemctl status neo4j

# Access Neo4j Browser: http://your-server-ip:7474
# Default credentials: neo4j/neo4j (must change on first login)
```

**Version Required:** Neo4j 5.15+ Community Edition  
**Ports:**
- 7474: HTTP (Neo4j Browser)
- 7687: Bolt protocol (application connection)

**Memory Allocation (for 16GB RAM dedicated to Neo4j):**
- Heap: 8GB (initial and max)
- Page cache: 4GB
- Transaction memory: 2GB
- OS reserve: 2GB

**Data Directory:** `/var/lib/neo4j/data/`  
**Config File:** `/etc/neo4j/neo4j.conf`  
**Logs:** `/var/log/neo4j/`

### 6. Redis

```bash
#!/bin/bash
# Install Redis

sudo apt-get install -y redis-server

# Configure Redis for production
sudo tee -a /etc/redis/redis.conf << 'EOF'
# Memory
maxmemory 4gb
maxmemory-policy allkeys-lru

# Persistence (RDB + AOF)
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec

# Security
requirepass YourStrongRedisPassword
bind 127.0.0.1 ::1

# Performance
tcp-backlog 511
timeout 300
EOF

# Restart Redis
sudo systemctl restart redis-server
sudo systemctl enable redis-server

# Verify
redis-cli ping
# Expected: PONG
```

**Version Required:** 7.0+  
**Memory Allocation:** 4 GB  
**Port:** 6379 (localhost only)

**Use Cases:**
- SignalR backplane (real-time collaboration)
- Session state storage
- Application caching
- Rate limiting

**Config File:** `/etc/redis/redis.conf`  
**Data Directory:** `/var/lib/redis/`

### 7. Docker & Docker Compose (Optional but Recommended)

```bash
#!/bin/bash
# Install Docker

# Add Docker repository
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add current user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose plugin
sudo apt-get install -y docker-compose-plugin

# Verify installation
docker --version
docker compose version

# Test Docker
docker run hello-world
```

**Version Required:**
- Docker Engine: 24.x or later
- Docker Compose: 2.x or later

**Purpose:**
- Containerization of application components
- Easier deployment and rollback
- Environment consistency
- Simplified scaling

### 8. SSL/TLS Certificates (Let's Encrypt)

```bash
#!/bin/bash
# Install Certbot for free SSL certificates

sudo apt-get install -y certbot python3-certbot-nginx

# Obtain SSL certificate (replace with your domain)
sudo certbot --nginx -d veropm.com -d www.veropm.com

# Auto-renewal (Certbot sets this up automatically)
# Test renewal
sudo certbot renew --dry-run
```

**Certificate Renewal:** Automatic (cron job created by Certbot)  
**Validity:** 90 days (auto-renews at 60 days)

### 9. Security Tools

#### Fail2Ban (Brute-force Protection)
```bash
#!/bin/bash
# Install and configure Fail2Ban

sudo apt-get install -y fail2ban

# Configure SSH protection
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
EOF

# Start and enable Fail2Ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

#### UFW Firewall
```bash
#!/bin/bash
# Configure UFW firewall

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (change port if using non-standard)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IPs only (example for database ports)
# sudo ufw allow from 10.0.0.0/8 to any port 1433 proto tcp  # SQL Server
# sudo ufw allow from 10.0.0.0/8 to any port 7687 proto tcp  # Neo4j

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

### 10. Process Manager (PM2)

```bash
#!/bin/bash
# PM2 is already installed with Node.js

# PM2 ecosystem file for .NET application
cat > /opt/veropm/ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'veropm-backend',
    script: 'dotnet',
    args: 'VeroPM.API.dll',
    cwd: '/opt/veropm/backend/publish',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '2G',
    env: {
      ASPNETCORE_ENVIRONMENT: 'Production',
      ASPNETCORE_URLS: 'http://localhost:5000'
    }
  }]
};
EOF

# Start application with PM2
cd /opt/veropm
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Set PM2 to start on boot
pm2 startup
# Follow the command output to enable startup
```

### 11. Monitoring Tools (Optional)

#### Prometheus + Grafana Stack
```bash
#!/bin/bash
# Deploy monitoring stack using Docker Compose

mkdir -p /opt/monitoring
cd /opt/monitoring

cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
EOF

# Start monitoring stack
docker compose up -d
```

**Access:**
- Prometheus: http://server-ip:9090
- Grafana: http://server-ip:3000 (admin/admin)
- Node Exporter: http://server-ip:9100

---

## Network Configuration

### Port Requirements & Security

```yaml
External (Public Internet):
  - 80/tcp: HTTP (redirect to HTTPS)
  - 443/tcp: HTTPS (Nginx reverse proxy)
  - 22/tcp: SSH (key-based authentication only)

Internal (Localhost/Private Network):
  - 5000/tcp: Backend API (.NET Core)
  - 1433/tcp: SQL Server (localhost or VPN only)
  - 7687/tcp: Neo4j Bolt (localhost or VPN only)
  - 7474/tcp: Neo4j Browser (localhost or VPN only)
  - 6379/tcp: Redis (localhost only)

Monitoring (Optional, VPN access):
  - 9090/tcp: Prometheus
  - 3000/tcp: Grafana
  - 9100/tcp: Node Exporter
```

### Nginx Configuration

#### Main Configuration (`/etc/nginx/nginx.conf`)
```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 4096;
    use epoll;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Buffer Settings
    client_max_body_size 100M;
    client_body_buffer_size 128k;
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript 
               application/json application/javascript application/xml+rss;
    
    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    # Include site configurations
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

#### VeroPM Site Configuration (`/etc/nginx/sites-available/veropm`)
```nginx
# Rate limiting
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

# Backend upstream
upstream backend {
    server localhost:5000;
    keepalive 32;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name veropm.com www.veropm.com;
    
    return 301 https://$server_name$request_uri;
}

# Main HTTPS server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name veropm.com www.veropm.com;
    
    # SSL Certificate (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/veropm.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/veropm.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Frontend (React)
    location / {
        root /opt/veropm/frontend/build;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
    
    # Backend API
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # SignalR WebSocket
    location /hubs/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        
        # WebSocket timeout
        proxy_read_timeout 3600s;
    }
    
    # Health check endpoint
    location /health {
        proxy_pass http://backend;
        access_log off;
    }
}
```

**Enable site:**
```bash
sudo ln -s /etc/nginx/sites-available/veropm /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Deployment Architecture

### Directory Structure

```
/opt/veropm/
├── backend/
│   ├── publish/                    # Published .NET application
│   │   ├── VeroPM.API.dll
│   │   ├── appsettings.json
│   │   ├── appsettings.Production.json
│   │   └── wwwroot/
│   ├── logs/                       # Application logs
│   │   ├── app-20260105.log
│   │   └── errors-20260105.log
│   └── scripts/
│       ├── deploy.sh
│       └── rollback.sh
├── frontend/
│   ├── build/                      # React production build
│   │   ├── index.html
│   │   ├── static/
│   │   │   ├── css/
│   │   │   ├── js/
│   │   │   └── media/
│   │   └── assets/
│   └── source/                     # Source code (optional)
├── uploads/                        # Symlink to /data/uploads
├── scripts/
│   ├── backup.sh
│   ├── restore.sh
│   ├── health-check.sh
│   └── maintenance.sh
└── ecosystem.config.js             # PM2 configuration

/data/
├── mssql/                          # SQL Server database files
│   ├── VeroPM_Data.mdf
│   ├── VeroPM_Log.ldf
│   └── tempdb/
├── neo4j/                          # Neo4j graph database
│   ├── databases/
│   ├── transactions/
│   └── logs/
├── redis/                          # Redis persistence
│   └── dump.rdb
└── uploads/                        # User uploaded files
    ├── documents/
    ├── images/
    └── attachments/

/backups/
├── daily/
│   ├── mssql-20260105.bak
│   ├── neo4j-20260105.tar.gz
│   └── uploads-20260105.tar.gz
├── weekly/
└── monthly/

/var/log/
├── nginx/
│   ├── access.log
│   └── error.log
├── veropm/
│   ├── backend.log
│   └── frontend.log
└── supervisor/
```

### Application Configuration

#### Backend (`appsettings.Production.json`)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=VeroPM;User Id=veropm_user;Password=StrongPassword;TrustServerCertificate=True;",
    "Neo4jConnection": "bolt://localhost:7687",
    "RedisConnection": "localhost:6379,password=RedisPassword"
  },
  "Neo4j": {
    "User": "neo4j",
    "Password": "Neo4jPassword",
    "Database": "veropm"
  },
  "Redis": {
    "Configuration": "localhost:6379,password=RedisPassword",
    "InstanceName": "VeroPM_"
  },
  "AIBOL": {
    "AzureOpenAI": {
      "Endpoint": "https://your-resource.openai.azure.com/",
      "ApiKey": "your-api-key",
      "DeploymentName": "gpt-4o"
    },
    "Anthropic": {
      "ApiKey": "your-anthropic-key"
    },
    "Google": {
      "ApiKey": "your-google-key"
    }
  },
  "JWT": {
    "Secret": "your-jwt-secret-key-minimum-32-characters",
    "Issuer": "VeroPM",
    "Audience": "VeroPMUsers",
    "ExpiryMinutes": 1440
  },
  "FileStorage": {
    "MaxFileSize": 104857600,
    "AllowedExtensions": [".pdf", ".docx", ".xlsx", ".png", ".jpg"],
    "StoragePath": "/data/uploads"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    },
    "File": {
      "Path": "/opt/veropm/backend/logs/app-.log",
      "RollingInterval": "Day"
    }
  }
}
```

#### Frontend Environment (`.env.production`)
```bash
REACT_APP_API_URL=https://veropm.com/api
REACT_APP_SIGNALR_URL=https://veropm.com/hubs
REACT_APP_ENVIRONMENT=production
REACT_APP_VERSION=1.0.0
```

### Deployment Scripts

#### Backend Deployment (`/opt/veropm/scripts/deploy-backend.sh`)
```bash
#!/bin/bash
# VeroPM Backend Deployment Script

set -e

APP_DIR="/opt/veropm/backend"
PUBLISH_DIR="$APP_DIR/publish"
BACKUP_DIR="$APP_DIR/backup-$(date +%Y%m%d-%H%M%S)"
PM2_APP_NAME="veropm-backend"

echo "🚀 Starting VeroPM Backend Deployment..."

# Stop the application
echo "⏸️  Stopping application..."
pm2 stop $PM2_APP_NAME || true

# Backup current version
if [ -d "$PUBLISH_DIR" ]; then
    echo "📦 Backing up current version..."
    mv $PUBLISH_DIR $BACKUP_DIR
fi

# Create publish directory
mkdir -p $PUBLISH_DIR

# Copy new build (assumes build artifact is in /tmp/publish)
echo "📂 Copying new build..."
cp -r /tmp/publish/* $PUBLISH_DIR/

# Set permissions
echo "🔒 Setting permissions..."
sudo chown -R www-data:www-data $PUBLISH_DIR
chmod -R 755 $PUBLISH_DIR

# Run database migrations
echo "🗄️  Running database migrations..."
cd $PUBLISH_DIR
dotnet VeroPM.API.dll --migrate || true

# Start the application
echo "▶️  Starting application..."
pm2 start $PM2_APP_NAME

# Wait for health check
echo "🏥 Checking application health..."
sleep 5
curl -f http://localhost:5000/health || {
    echo "❌ Health check failed! Rolling back..."
    pm2 stop $PM2_APP_NAME
    rm -rf $PUBLISH_DIR
    mv $BACKUP_DIR $PUBLISH_DIR
    pm2 start $PM2_APP_NAME
    exit 1
}

echo "✅ Deployment successful!"
pm2 save
```

#### Frontend Deployment (`/opt/veropm/scripts/deploy-frontend.sh`)
```bash
#!/bin/bash
# VeroPM Frontend Deployment Script

set -e

FRONTEND_DIR="/opt/veropm/frontend"
BUILD_DIR="$FRONTEND_DIR/build"
BACKUP_DIR="$FRONTEND_DIR/backup-$(date +%Y%m%d-%H%M%S)"

echo "🚀 Starting VeroPM Frontend Deployment..."

# Backup current version
if [ -d "$BUILD_DIR" ]; then
    echo "📦 Backing up current version..."
    mv $BUILD_DIR $BACKUP_DIR
fi

# Create build directory
mkdir -p $BUILD_DIR

# Copy new build (assumes build artifact is in /tmp/frontend-build)
echo "📂 Copying new build..."
cp -r /tmp/frontend-build/* $BUILD_DIR/

# Set permissions
echo "🔒 Setting permissions..."
sudo chown -R www-data:www-data $BUILD_DIR
chmod -R 755 $BUILD_DIR

# Reload Nginx
echo "🔄 Reloading Nginx..."
sudo nginx -t && sudo systemctl reload nginx

echo "✅ Frontend deployment successful!"
```

#### Automated Backup Script (`/opt/veropm/scripts/backup.sh`)
```bash
#!/bin/bash
# VeroPM Automated Backup Script

BACKUP_ROOT="/backups"
DATE=$(date +%Y%m%d-%H%M%S)
RETENTION_DAYS=30

echo "🔄 Starting backup process..."

# SQL Server backup
echo "📊 Backing up SQL Server..."
sqlcmd -S localhost -U SA -P "$SQL_PASSWORD" -Q "BACKUP DATABASE VeroPM TO DISK = '/backups/daily/veropm-mssql-$DATE.bak' WITH COMPRESSION"

# Neo4j backup
echo "🕸️  Backing up Neo4j..."
sudo systemctl stop neo4j
tar -czf $BACKUP_ROOT/daily/veropm-neo4j-$DATE.tar.gz /var/lib/neo4j/data
sudo systemctl start neo4j

# User uploads backup
echo "📁 Backing up uploads..."
tar -czf $BACKUP_ROOT/daily/veropm-uploads-$DATE.tar.gz /data/uploads

# Application configuration backup
echo "⚙️  Backing up configuration..."
tar -czf $BACKUP_ROOT/daily/veropm-config-$DATE.tar.gz \
    /opt/veropm/backend/publish/appsettings.Production.json \
    /etc/nginx/sites-available/veropm \
    /opt/veropm/ecosystem.config.js

# Delete old backups
echo "🗑️  Cleaning old backups..."
find $BACKUP_ROOT/daily/ -type f -mtime +$RETENTION_DAYS -delete

echo "✅ Backup completed successfully!"
```

**Add to crontab:**
```bash
# Run daily at 2 AM
0 2 * * * /opt/veropm/scripts/backup.sh >> /var/log/veropm/backup.log 2>&1
```

---

## Cost Analysis

### Azure Monthly Costs (Detailed Breakdown)

| Service | SKU/Tier | Specs | Monthly Cost | Annual Cost |
|---------|----------|-------|--------------|-------------|
| **App Service (Backend)** | Premium P2V3 | 4 vCPUs, 16GB RAM | $292 | $3,504 |
| **App Service (Frontend)** | Premium P1V3 | 2 vCPUs, 8GB RAM | $146 | $1,752 |
| **Azure SQL Database** | Standard S3 | 100 DTUs, 500GB | $300 | $3,600 |
| **VM for Neo4j** | D4s_v3 | 4 vCPUs, 16GB RAM, 512GB SSD | $175 | $2,100 |
| **Blob Storage** | Hot Tier | 1TB storage | $20 | $240 |
| **Redis Cache** | Standard C2 | 2.5GB memory | $125 | $1,500 |
| **Application Gateway** | WAF V2 | 2 instances | $300 | $3,600 |
| **Key Vault** | Standard | Secrets storage | $5 | $60 |
| **Application Insights** | Standard | 5GB data/month | $50 | $600 |
| **Bandwidth** | Outbound | ~500GB/month | $50 | $600 |
| **Backup Storage** | LRS | 500GB snapshots | $25 | $300 |
| **Azure DevOps** | Basic | 5 users | $30 | $360 |
| **TOTAL** | | | **$1,518/month** | **$18,216/year** |

**Notes:**
- Costs based on East US region (may vary by region)
- Excludes support plans ($100-1000/month)
- Assumes moderate traffic (~100k requests/day)

### Dedicated Server Monthly Costs

#### Option A: All-in-One Server
| Item | Specs | Monthly Cost | Annual Cost |
|------|-------|--------------|-------------|
| **Server Rental/Colo** | Xeon E-2388G, 64GB RAM, RAID | $350 | $4,200 |
| **Bandwidth** | 1Gbps, 10TB | $100 | $1,200 |
| **Backup Storage** | 500GB offsite | $30 | $360 |
| **SSL Certificate** | Let's Encrypt | $0 | $0 |
| **Monitoring** | Self-hosted | $0 | $0 |
| **TOTAL** | | **$480/month** | **$5,760/year** |

#### Option B: High-Performance Server
| Item | Specs | Monthly Cost | Annual Cost |
|------|-------|--------------|-------------|
| **Server Rental/Colo** | Xeon Gold 6338, 128GB RAM, RAID 10 | $600 | $7,200 |
| **Bandwidth** | 10Gbps, Unlimited | $200 | $2,400 |
| **Backup Storage** | 1TB offsite + local | $50 | $600 |
| **SSL Certificate** | Let's Encrypt | $0 | $0 |
| **Monitoring** | Self-hosted | $0 | $0 |
| **TOTAL** | | **$850/month** | **$10,200/year** |

### Cost Comparison Summary

| Metric | Azure | Dedicated (Min) | Dedicated (High) |
|--------|-------|-----------------|------------------|
| **Monthly Cost** | $1,518 | $480 | $850 |
| **Annual Cost** | $18,216 | $5,760 | $10,200 |
| **3-Year Total** | $54,648 | $17,280 | $30,600 |
| **Monthly Savings vs Azure** | - | **$1,038 (68%)** | **$668 (44%)** |
| **Annual Savings vs Azure** | - | **$12,456** | **$8,016** |
| **3-Year Savings vs Azure** | - | **$37,368** | **$24,048** |

### Hidden Costs Comparison

| Cost Category | Azure | Dedicated Server |
|---------------|-------|------------------|
| **Initial Setup** | $0 (managed) | $500-1000 (one-time) |
| **DevOps Time** | Lower (PaaS) | Higher (IaaS management) |
| **Scaling Complexity** | Very Easy | Manual configuration |
| **Disaster Recovery** | Built-in | Manual setup required |
| **Security Updates** | Automatic (PaaS) | Manual (OS + software) |
| **Support** | $100-1000/month | Self-managed or $50-200/month |

### Break-Even Analysis

**Dedicated server breaks even after:**
- **High-Performance Option:** 2 months of operation
- **Minimum Spec Option:** 1.5 months of operation

**ROI over 3 years:**
- High-Performance: **$24,048 savings** (44% cost reduction)
- Minimum Spec: **$37,368 savings** (68% cost reduction)

---

## Recommendations

### For GITEX Exhibition & Initial Launch (0-6 months)

**Recommended: Azure Cloud**

**Reasons:**
1. **Speed to Market**
   - Deploy in hours, not days
   - No hardware procurement delays
   - Pre-configured services

2. **Professional Image**
   - Enterprise-grade infrastructure
   - 99.9%+ SLA guarantees
   - Microsoft brand association

3. **GITEX Demo Requirements**
   - Guaranteed uptime during exhibition
   - Global accessibility for remote demos
   - Built-in DDoS protection

4. **Flexibility**
   - Scale up/down based on demo traffic
   - Add features without infrastructure changes
   - Easy A/B testing

5. **Focus on Product**
   - Less DevOps overhead
   - More time for feature development
   - Automated backups and monitoring

**Suggested Azure Configuration for Launch:**
```yaml
Minimal Launch Configuration (Cost: ~$800/month):
  - App Service: P1V3 for both backend and frontend
  - Azure SQL: S2 (50 DTUs)
  - Neo4j: D2s_v3 VM (2 vCPUs, 8GB)
  - Redis: C1 (1GB)
  - Blob Storage: 250GB
  - No Application Gateway (use App Service SSL)
  
Scale to Full Production After Launch (Cost: $1,518/month)
```

### For Long-Term Production (6+ months)

**Recommended: Migrate to Dedicated Server**

**Reasons:**
1. **Cost Savings**
   - Save $12,000-37,000 over 3 years
   - Predictable monthly costs
   - No surprise charges

2. **Performance**
   - Dedicated resources (no noisy neighbors)
   - Custom tuning for VeroPM workload
   - Lower latency for database operations

3. **Data Control**
   - Complete data sovereignty
   - No vendor lock-in
   - Custom security policies

4. **GraphRAG Optimization**
   - Neo4j Community unlimited database size
   - Custom kernel tuning for graph operations
   - Direct NVMe storage access

**Migration Strategy:**

**Phase 1: Preparation (Month 4-5)**
- Procure and configure dedicated server
- Set up parallel environment
- Test deployment scripts
- Configure monitoring and backups

**Phase 2: Testing (Month 5-6)**
- Deploy to dedicated server
- Run parallel systems
- Performance testing
- Load testing with real data

**Phase 3: Migration (Month 6-7)**
- DNS cutover during low-traffic window
- Monitor for 2 weeks
- Decommission Azure after stability confirmed
- Keep Azure for disaster recovery (1 month)

### Hybrid Approach (Best of Both Worlds)

**For Maximum Reliability:**

```yaml
Primary: Dedicated Server
  - Main production workload
  - Cost: $850/month

Backup: Azure (Minimal Config)
  - Hot standby for disaster recovery
  - Automated failover capability
  - Cost: $400/month
  
Total: $1,250/month
Still saves $268/month vs full Azure
Provides enterprise-level redundancy
```

### Decision Matrix

| Criteria | Azure | Dedicated | Hybrid |
|----------|-------|-----------|--------|
| **Initial Cost** | ✅ Low | ⚠️ Medium | ⚠️ Medium |
| **Long-term Cost** | ❌ High | ✅ Low | ✅ Medium |
| **Time to Deploy** | ✅ Fast (hours) | ⚠️ Slow (days/weeks) | ⚠️ Medium |
| **Scalability** | ✅ Instant | ⚠️ Manual | ✅ Good |
| **Performance** | ✅ Good | ✅ Excellent | ✅ Excellent |
| **Control** | ⚠️ Limited | ✅ Full | ✅ Full |
| **Maintenance** | ✅ Managed | ❌ Self-managed | ⚠️ Mixed |
| **Disaster Recovery** | ✅ Built-in | ❌ Manual | ✅ Excellent |

### Final Recommendation

**Strategy: Phased Approach**

```
┌─────────────────────────────────────────────────────────┐
│ Month 0-6: Azure Cloud (Launch & Validation)           │
│ - Fast deployment for GITEX                            │
│ - Prove product-market fit                             │
│ - Gather performance metrics                           │
│ Cost: ~$1,500/month                                     │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Month 6-12: Migrate to Dedicated Server               │
│ - Reduce costs by 44-68%                               │
│ - Optimize for known workload                          │
│ - Keep Azure as backup (optional)                      │
│ Cost: $850/month (or $1,250 with backup)               │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Month 12+: Scale Dedicated Infrastructure             │
│ - Add servers as customer base grows                   │
│ - Consider multi-region deployment                     │
│ - Implement CDN for global performance                 │
│ Cost: Scales with revenue                              │
└─────────────────────────────────────────────────────────┘
```

**This approach:**
- Minimizes initial risk and complexity
- Maximizes flexibility during launch phase
- Optimizes costs for long-term sustainability
- Provides clear migration path
- Allows focus on product, not infrastructure

---

## Next Steps

Would you like me to create:

1. **Detailed Azure deployment guide** with ARM templates and scripts?
2. **Docker Compose configuration** for easier dedicated server deployment?
3. **CI/CD pipeline setup** (GitHub Actions or Azure DevOps)?
4. **Complete Nginx configurations** with all security headers?
5. **Database migration scripts** for production setup?
6. **Monitoring and alerting setup** (Prometheus/Grafana configs)?
7. **Disaster recovery runbook** with step-by-step procedures?

Let me know which documentation would be most valuable for your immediate needs!
