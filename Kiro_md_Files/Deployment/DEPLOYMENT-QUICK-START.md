# AWS Deployment - Quick Start Guide

Since you've already configured AWS CLI (`aws configure`), deployment is straightforward!

---

## Current Setup ✅

- **AWS CLI configured** - Credentials in `~/.aws/credentials`
- **DynamoDB** - Already on AWS (hackveda-users table)
- **Judge0** - Already on EC2 (13.234.32.75:2358)
- **Bedrock Agents** - Already configured
- **No hardcoded credentials** - Using AWS CLI credentials

---

## Deployment Options

### Option 1: Single EC2 Instance (Recommended - Simple & Cost-Effective)
**Cost:** ~$30-40/month

Deploy all 3 backends on one EC2 instance with PM2, frontend on S3.

### Option 2: Elastic Beanstalk (More Scalable)
**Cost:** ~$75-100/month

Separate Elastic Beanstalk environments for each backend, auto-scaling.

---

## Option 1: Single EC2 Deployment (Recommended)

### Step 1: Launch EC2 Instance

1. **Go to EC2 Console:**
   - https://console.aws.amazon.com/ec2/

2. **Launch Instance:**
   - Name: `hackveda-app-server`
   - AMI: Amazon Linux 2023
   - Instance type: `t3.medium` (2 vCPU, 4GB RAM)
   - Key pair: Create new or use existing
   - Security Group: Create new with these rules:
     - SSH (22) - Your IP
     - HTTP (80) - Anywhere
     - HTTPS (443) - Anywhere
     - Custom TCP (5005) - Anywhere (chatbot)
     - Custom TCP (5006) - Anywhere (PDF)
     - Custom TCP (5012) - Anywhere (main backend)

3. **Launch Instance**

### Step 2: Connect to EC2

```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

### Step 3: Install Dependencies

```bash
# Update system
sudo yum update -y

# Install Node.js 18
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install Git
sudo yum install -y git

# Install PM2 (process manager)
sudo npm install -g pm2

# Install Nginx (reverse proxy)
sudo yum install -y nginx
```

### Step 4: Configure AWS CLI on EC2

```bash
aws configure
```

Enter your AWS credentials (same as your local PC):
- AWS Access Key ID: [your-key]
- AWS Secret Access Key: [your-secret]
- Default region: ap-south-1
- Default output format: json

### Step 5: Clone Your Repository

```bash
cd /home/ec2-user
git clone your-repository-url hackveda
cd hackveda
```

### Step 6: Setup Main Backend

```bash
cd backend
npm install
pm2 start server.js --name hackveda-backend
pm2 save
```

### Step 7: Setup Chatbot Backend

```bash
cd ../chatbotAWS
npm install
pm2 start backend.js --name hackveda-chatbot
pm2 save
```

### Step 8: Setup PDF Backend

```bash
cd ../HackathonPDF/backend
npm install
pm2 start src/index.ts --name hackveda-pdf
pm2 save
```

### Step 9: Configure PM2 to Start on Boot

```bash
pm2 startup
# Copy and run the command it outputs
pm2 save
```

### Step 10: Verify All Services Running

```bash
pm2 status
```

Should show:
```
┌─────┬──────────────────────┬─────────┬─────────┐
│ id  │ name                 │ status  │ restart │
├─────┼──────────────────────┼─────────┼─────────┤
│ 0   │ hackveda-backend     │ online  │ 0       │
│ 1   │ hackveda-chatbot     │ online  │ 0       │
│ 2   │ hackveda-pdf         │ online  │ 0       │
└─────┴──────────────────────┴─────────┴─────────┘
```

### Step 11: Configure Nginx as Reverse Proxy

Create `/etc/nginx/conf.d/hackveda.conf`:

```bash
sudo nano /etc/nginx/conf.d/hackveda.conf
```

Paste this configuration:

```nginx
server {
    listen 80;
    server_name your-ec2-public-ip;  # Replace with your EC2 IP or domain

    # Main Backend API
    location /api/ {
        proxy_pass http://localhost:5012/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    # Chatbot Backend
    location /chatbot/ {
        proxy_pass http://localhost:5005/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    # PDF Backend
    location /pdf/ {
        proxy_pass http://localhost:5006/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and exit (Ctrl+X, Y, Enter)

### Step 12: Start Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### Step 13: Test Backend APIs

```bash
# Test main backend
curl http://localhost:5012/

# Test chatbot backend
curl http://localhost:5005/

# Test PDF backend
curl http://localhost:5006/
```

All should return success responses.

---

## Deploy Frontend to S3

### Step 1: Update Frontend API URLs

On your local PC, create `frontend/.env.production`:

```env
VITE_API_URL=http://your-ec2-public-ip/api
VITE_CHATBOT_API_URL=http://your-ec2-public-ip/chatbot
VITE_PDF_API_URL=http://your-ec2-public-ip/pdf
```

### Step 2: Update API Base URLs in Code

**frontend/src/api/api.js:**
```javascript
const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:5012';

export async function apiRequest(endpoint, method = "GET", body = null) {
  const url = `${API_BASE_URL}${endpoint}`;
  // ... rest of code
}
```

**frontend/src/pages/Sandbox.jsx:**
```javascript
const CHAT_API = import.meta.env.VITE_CHATBOT_API_URL 
  ? `${import.meta.env.VITE_CHATBOT_API_URL}/chat`
  : 'http://localhost:5005/chat';
```

**frontend/src/pages/AIMentorDrawer.jsx:**
```javascript
const CHAT_API = import.meta.env.VITE_CHATBOT_API_URL 
  ? `${import.meta.env.VITE_CHATBOT_API_URL}/chat`
  : 'http://localhost:5005/chat';
```

**frontend/src/hooks/usePDFLearning.js:**
```javascript
const API_BASE = import.meta.env.VITE_PDF_API_URL 
  ? `${import.meta.env.VITE_PDF_API_URL}/api`
  : '/api';
```

### Step 3: Build Frontend

```bash
cd frontend
npm run build
```

### Step 4: Create S3 Bucket

```bash
aws s3 mb s3://hackveda-frontend --region ap-south-1
```

### Step 5: Enable Static Website Hosting

```bash
aws s3 website s3://hackveda-frontend \
  --index-document index.html \
  --error-document index.html
```

### Step 6: Upload Build to S3

```bash
cd frontend
aws s3 sync dist/ s3://hackveda-frontend --delete
```

### Step 7: Make Bucket Public

Create `bucket-policy.json`:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::hackveda-frontend/*"
    }
  ]
}
```

Apply policy:
```bash
aws s3api put-bucket-policy \
  --bucket hackveda-frontend \
  --policy file://bucket-policy.json
```

### Step 8: Get S3 Website URL

```bash
echo "http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com"
```

Visit this URL - your app should be live!

---

## Update Backend CORS

Update CORS in all backends to allow your S3 frontend:

**backend/server.js:**
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com',
    'http://your-ec2-public-ip'
  ],
  credentials: true
}));
```

**chatbotAWS/backend.js:**
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com',
    'http://your-ec2-public-ip'
  ]
}));
```

**HackathonPDF/backend:** (similar update)

Restart backends on EC2:
```bash
pm2 restart all
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│  Frontend (S3 Static Website)                   │
│  http://hackveda-frontend.s3-website...         │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│  EC2 Instance (t3.medium)                       │
│  ┌───────────────────────────────────────────┐  │
│  │  Nginx (Reverse Proxy)                    │  │
│  │  Port 80                                  │  │
│  └───────────┬───────────────────────────────┘  │
│              │                                   │
│  ┌───────────┼───────────────────────────────┐  │
│  │  PM2 Process Manager                      │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  Main Backend (port 5012)           │  │  │
│  │  │  - DynamoDB, Bedrock Agents         │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  Chatbot Backend (port 5005)        │  │  │
│  │  │  - Bedrock Agent for AI Mentor      │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  PDF Backend (port 5006)            │  │  │
│  │  │  - Bedrock Agent for PDF Learning   │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│  External AWS Services                          │
│  - DynamoDB (hackveda-users)                    │
│  - Bedrock Agents (multiple)                    │
│  - Judge0 EC2 (13.234.32.75:2358)              │
└─────────────────────────────────────────────────┘
```

---

## Cost Breakdown

### EC2 Instance (t3.medium)
- Instance: ~$30/month
- Storage (30GB): ~$3/month
- Data transfer: ~$2/month
- **Subtotal: ~$35/month**

### S3 + Data Transfer
- S3 storage (1GB): ~$0.02/month
- S3 requests: ~$0.01/month
- Data transfer: ~$1/month
- **Subtotal: ~$1/month**

### Existing Services (Already Running)
- DynamoDB: Pay per request
- Judge0 EC2: Already running
- Bedrock Agents: Pay per invocation

**Total: ~$36/month** (very affordable!)

---

## Monitoring & Maintenance

### View Logs
```bash
# PM2 logs
pm2 logs

# Specific service logs
pm2 logs hackveda-backend
pm2 logs hackveda-chatbot
pm2 logs hackveda-pdf

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

### Restart Services
```bash
# Restart all
pm2 restart all

# Restart specific service
pm2 restart hackveda-backend
```

### Update Code
```bash
cd /home/ec2-user/hackveda
git pull
cd backend && npm install && pm2 restart hackveda-backend
cd ../chatbotAWS && npm install && pm2 restart hackveda-chatbot
cd ../HackathonPDF/backend && npm install && pm2 restart hackveda-pdf
```

### Update Frontend
```bash
# On your local PC
cd frontend
npm run build
aws s3 sync dist/ s3://hackveda-frontend --delete
```

---

## Troubleshooting

### Backend not accessible
```bash
# Check if services are running
pm2 status

# Check logs
pm2 logs

# Restart services
pm2 restart all
```

### CORS errors
- Update CORS configuration in backends
- Restart: `pm2 restart all`

### DynamoDB access denied
- Verify AWS CLI configured: `aws configure list`
- Test DynamoDB access: `aws dynamodb list-tables`

### Frontend not loading
- Check S3 bucket policy is public
- Verify API URLs in `.env.production`
- Check browser console for errors

---

## Security Checklist

- [ ] EC2 Security Group configured (only necessary ports open)
- [ ] SSH access restricted to your IP
- [ ] AWS credentials configured via `aws configure` (not hardcoded)
- [ ] `.env` files not committed to Git
- [ ] S3 bucket policy allows public read only
- [ ] Nginx configured as reverse proxy
- [ ] PM2 configured to restart on boot
- [ ] Regular backups of DynamoDB table
- [ ] CloudWatch monitoring enabled (optional)

---

## Next Steps

1. Complete EC2 setup and deploy backends
2. Deploy frontend to S3
3. Test all functionality
4. (Optional) Add custom domain with Route 53
5. (Optional) Add SSL certificate with ACM + CloudFront
6. (Optional) Set up CloudWatch alarms for monitoring

---

**Your app will be live at:**
- Frontend: `http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com`
- Backend API: `http://your-ec2-ip/api`
- Chatbot API: `http://your-ec2-ip/chatbot`
- PDF API: `http://your-ec2-ip/pdf`

**Good luck with deployment! 🚀**
