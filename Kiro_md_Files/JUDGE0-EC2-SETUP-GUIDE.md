# Judge0 EC2 Setup Guide (t3.medium)

This guide will help you set up Judge0 on an AWS EC2 t3.medium instance.

## Prerequisites
- AWS Account
- Basic knowledge of EC2 and SSH

---

## Step 1: Launch EC2 Instance

### 1.1 Create EC2 Instance
1. Go to AWS Console → EC2 → Launch Instance
2. **Name**: `judge0-server`
3. **AMI**: Ubuntu Server 22.04 LTS (Free tier eligible)
4. **Instance Type**: t3.medium (2 vCPU, 4 GB RAM)
5. **Key Pair**: Create new or use existing (download .pem file)
6. **Network Settings**:
   - Allow SSH (port 22) from your IP
   - Allow Custom TCP (port 2358) from anywhere (0.0.0.0/0) - for Judge0 API
   - Allow HTTP (port 80) - optional
   - Allow HTTPS (port 443) - optional

### 1.2 Security Group Configuration
Create/Edit security group with these inbound rules:

| Type       | Protocol | Port Range | Source          | Description        |
|------------|----------|------------|-----------------|--------------------|
| SSH        | TCP      | 22         | Your IP         | SSH access         |
| Custom TCP | TCP      | 2358       | 0.0.0.0/0       | Judge0 API         |
| HTTP       | TCP      | 80         | 0.0.0.0/0       | Optional           |
| HTTPS      | TCP      | 443        | 0.0.0.0/0       | Optional           |

### 1.3 Storage
- **Root Volume**: 30 GB gp3 (recommended for Judge0)

---

## Step 2: Connect to EC2 Instance

### Windows (PowerShell/CMD):
```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

### Mac/Linux:
```bash
chmod 400 your-key.pem
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

---

## Step 3: Install Docker and Docker Compose

### 3.1 Update System
```bash
sudo apt update
sudo apt upgrade -y
```

### 3.2 Install Docker
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify Docker installation
docker --version
```

### 3.3 Install Docker Compose
```bash
# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### 3.4 Logout and Login Again
```bash
exit
# Then SSH back in to apply docker group changes
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

---

## Step 4: Install Judge0

### 4.1 Create Judge0 Directory
```bash
mkdir -p ~/judge0
cd ~/judge0
```

### 4.2 Create docker-compose.yml
```bash
nano docker-compose.yml
```

Paste this configuration:

```yaml
version: '3.7'

services:
  judge0-server:
    image: judge0/judge0:1.13.0
    volumes:
      - ./judge0-data:/var/local/lib/isolate
    ports:
      - "2358:2358"
    privileged: true
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - POSTGRES_HOST=postgres
      - POSTGRES_PORT=5432
      - POSTGRES_DB=judge0
      - POSTGRES_USER=judge0
      - POSTGRES_PASSWORD=YourSecurePassword123!
      - ENABLE_WAIT_RESULT=true
      - ENABLE_COMPILER_OPTIONS=true
      - ALLOWED_LANGUAGES_FOR_COMPILE_OPTIONS=44,45,46,47,48,49,50,51,52,53,54
    depends_on:
      - postgres
      - redis
    restart: always

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_DB=judge0
      - POSTGRES_USER=judge0
      - POSTGRES_PASSWORD=YourSecurePassword123!
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: always

  redis:
    image: redis:6
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    restart: always

volumes:
  postgres-data:
  redis-data:
```

**Save and exit**: Press `Ctrl+X`, then `Y`, then `Enter`

### 4.3 Start Judge0
```bash
# Start all services
docker-compose up -d

# Check if containers are running
docker-compose ps

# View logs
docker-compose logs -f judge0-server
```

Wait about 30-60 seconds for all services to start.

### 4.4 Verify Judge0 is Running
```bash
# Test from EC2 instance
curl http://localhost:2358/about

# Should return Judge0 version info
```

---

## Step 5: Test Judge0 from Outside

### 5.1 Get Your EC2 Public IP
```bash
curl http://checkip.amazonaws.com
```

Or find it in AWS Console → EC2 → Instances → Your Instance → Public IPv4 address

### 5.2 Test from Your Local Machine
```bash
# Test about endpoint
curl http://YOUR-EC2-PUBLIC-IP:2358/about

# Test languages endpoint
curl http://YOUR-EC2-PUBLIC-IP:2358/languages

# Test submission
curl -X POST http://YOUR-EC2-PUBLIC-IP:2358/submissions \
  -H "Content-Type: application/json" \
  -d '{
    "source_code": "print(42)",
    "language_id": 71,
    "stdin": ""
  }'
```

---

## Step 6: Configure Your Backend

### 6.1 Update .env file
In your backend `.env` file, update:

```env
JUDGE0_URL=http://YOUR-EC2-PUBLIC-IP:2358
```

Replace `YOUR-EC2-PUBLIC-IP` with your actual EC2 public IP.

### 6.2 Restart Your Backend
```bash
# Stop your backend
# Update .env
# Start your backend again
```

---

## Step 7: Monitor and Maintain

### 7.1 Useful Docker Commands
```bash
# View logs
docker-compose logs -f judge0-server

# Restart services
docker-compose restart

# Stop services
docker-compose down

# Start services
docker-compose up -d

# Check resource usage
docker stats
```

### 7.2 Check System Resources
```bash
# Check disk space
df -h

# Check memory
free -h

# Check CPU
top
```

---

## Step 8: Optional - Set Up Domain Name

### 8.1 If You Have a Domain
1. Go to your domain registrar (e.g., Route 53, GoDaddy)
2. Create an A record pointing to your EC2 public IP
   - Example: `judge0.yourdomain.com` → `YOUR-EC2-PUBLIC-IP`

### 8.2 Update Backend .env
```env
JUDGE0_URL=http://judge0.yourdomain.com:2358
```

---

## Troubleshooting

### Judge0 Not Starting
```bash
# Check logs
docker-compose logs judge0-server

# Check if ports are in use
sudo netstat -tulpn | grep 2358

# Restart services
docker-compose down
docker-compose up -d
```

### Connection Refused
1. Check security group allows port 2358
2. Check if Judge0 is running: `docker-compose ps`
3. Check EC2 instance firewall: `sudo ufw status`
4. If UFW is active, allow port: `sudo ufw allow 2358`

### Out of Memory
```bash
# Check memory usage
free -h

# If needed, upgrade to t3.large or add swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Disk Space Issues
```bash
# Clean up Docker
docker system prune -a

# Check disk usage
df -h
```

---

## Security Best Practices

### 1. Use Elastic IP (Recommended)
- Allocate an Elastic IP in AWS Console
- Associate it with your EC2 instance
- This prevents IP changes on instance restart

### 2. Enable HTTPS (Production)
```bash
# Install Nginx
sudo apt install nginx -y

# Install Certbot for SSL
sudo apt install certbot python3-certbot-nginx -y

# Configure Nginx as reverse proxy
sudo nano /etc/nginx/sites-available/judge0

# Add this configuration:
server {
    listen 80;
    server_name judge0.yourdomain.com;

    location / {
        proxy_pass http://localhost:2358;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Enable site
sudo ln -s /etc/nginx/sites-available/judge0 /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Get SSL certificate
sudo certbot --nginx -d judge0.yourdomain.com
```

### 3. Restrict Access (Optional)
If you only want your backend EC2 to access Judge0:
1. In Security Group, change port 2358 source from `0.0.0.0/0` to your backend EC2's security group or IP

---

## Cost Estimation

**t3.medium EC2 Instance:**
- On-Demand: ~$0.0416/hour = ~$30/month
- Reserved (1 year): ~$20/month
- Spot Instance: ~$12-15/month (can be terminated)

**Additional Costs:**
- EBS Storage (30 GB): ~$3/month
- Data Transfer: Varies based on usage
- Elastic IP: Free if attached to running instance

**Total Estimated Cost: $33-35/month**

---

## Next Steps

1. ✅ Launch EC2 t3.medium instance
2. ✅ Install Docker and Docker Compose
3. ✅ Deploy Judge0 using docker-compose
4. ✅ Test Judge0 from local machine
5. ✅ Update backend .env with Judge0 URL
6. ✅ Test code execution from your application
7. 🔒 (Optional) Set up domain and HTTPS
8. 📊 (Optional) Set up monitoring with CloudWatch

---

## Support

If you encounter issues:
1. Check Judge0 logs: `docker-compose logs judge0-server`
2. Check Judge0 documentation: https://github.com/judge0/judge0
3. Verify security group settings
4. Ensure EC2 instance has enough resources

---

## Quick Reference

**EC2 Instance Details:**
- Type: t3.medium
- vCPU: 2
- RAM: 4 GB
- Storage: 30 GB
- OS: Ubuntu 22.04 LTS

**Judge0 Endpoints:**
- About: `http://YOUR-IP:2358/about`
- Languages: `http://YOUR-IP:2358/languages`
- Submissions: `http://YOUR-IP:2358/submissions`

**Important Files:**
- Docker Compose: `~/judge0/docker-compose.yml`
- Judge0 Data: `~/judge0/judge0-data/`
- Logs: `docker-compose logs judge0-server`
