# AWS Deployment Guide - Complete Project

## 🚨 CRITICAL SECURITY ISSUES TO FIX FIRST

### Exposed AWS Credentials
Your AWS access keys are hardcoded in multiple files and exposed in this conversation:
- `chatbotAWS/backend.js` - Hardcoded credentials
- `HackathonPDF/backend/.env` - Credentials in .env file
- `rishu.119484@stu.upes.ac.in_accessKeys.csv` - Credentials file in root

**⚠️ IMMEDIATE ACTION REQUIRED:**
1. **Rotate these credentials immediately** in AWS IAM Console
2. **Never commit credentials to code** - use environment variables
3. **Delete the CSV file** from your repository

---

## Project Architecture Overview

Your application consists of 4 separate services:

1. **Main Backend** (port 5012) - Express.js with DynamoDB
2. **AI Chatbot Backend** (port 5005) - Bedrock Agent for Dashboard/Sandbox
3. **PDF Learning Backend** (port 5006) - Bedrock Agent for PDF features
4. **Frontend** (port 5010) - React/Vite application

**External Services:**
- DynamoDB (already on AWS)
- Judge0 (already on EC2: 13.234.32.75:2358)
- AWS Bedrock Agents (already configured)

---

## Deployment Strategy

### Option 1: AWS Elastic Beanstalk (Recommended - Easiest)
- Deploy each backend as separate Elastic Beanstalk application
- Frontend on S3 + CloudFront
- Automatic scaling and load balancing
- Easy environment variable management

### Option 2: AWS EC2 (More Control)
- Single EC2 instance running all backends with PM2
- Frontend on S3 + CloudFront
- Manual scaling and management
- Lower cost for small projects

### Option 3: AWS ECS/Fargate (Production-Grade)
- Containerized deployment with Docker
- Auto-scaling and high availability
- More complex setup
- Higher cost

**We'll use Option 1 (Elastic Beanstalk) for balance of ease and scalability.**

---

## Pre-Deployment Checklist

### 1. Fix Security Issues

#### A. Create .env file for chatbotAWS
```bash
cd chatbotAWS
```

Create `.env` file:
```env
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your-new-key-here
AWS_SECRET_ACCESS_KEY=your-new-secret-here
BEDROCK_AGENT_ID=OKPUEODEZC
BEDROCK_AGENT_ALIAS=3KNCV8OBXZ
PORT=5005
```

#### B. Update chatbotAWS/backend.js to use environment variables
```javascript
import dotenv from 'dotenv';
dotenv.config();

const REGION = process.env.AWS_REGION || "ap-south-1";
const AWS_ACCESS_KEY = process.env.AWS_ACCESS_KEY_ID;
const AWS_SECRET_KEY = process.env.AWS_SECRET_ACCESS_KEY;
const AGENT_ID = process.env.BEDROCK_AGENT_ID;
const AGENT_ALIAS = process.env.BEDROCK_AGENT_ALIAS;
const PORT = process.env.PORT || 5005;
```

#### C. Install dotenv in chatbotAWS
```bash
cd chatbotAWS
npm install dotenv
```

#### D. Update .gitignore files
Add to all `.gitignore` files:
```
.env
.env.local
.env.production
*.csv
```

#### E. Rotate AWS Credentials
1. Go to AWS IAM Console
2. Delete the exposed access key: `AKIAZ36GRW2VTHXIKWPI`
3. Create new access key
4. Update all `.env` files with new credentials
5. Delete `rishu.119484@stu.upes.ac.in_accessKeys.csv`

---

## Deployment Steps

### Phase 1: Prepare Backends for Deployment

#### 1.1 Main Backend Preparation

```bash
cd backend
```

Create `package.json` scripts for production:
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

Create `.ebignore` file (similar to .gitignore):
```
node_modules/
.env.local
*.md
test-*.js
*.log
.DS_Store
```

#### 1.2 ChatbotAWS Backend Preparation

```bash
cd chatbotAWS
```

Update `package.json`:
```json
{
  "name": "chatbot-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node backend.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "@aws-sdk/client-bedrock-agent-runtime": "^3.x.x",
    "dotenv": "^16.0.3"
  }
}
```

#### 1.3 HackathonPDF Backend Preparation

```bash
cd HackathonPDF/backend
```

Ensure `package.json` has start script:
```json
{
  "scripts": {
    "start": "node src/index.ts",
    "dev": "nodemon src/index.ts"
  }
}
```

---

### Phase 2: Deploy Backends to Elastic Beanstalk

#### 2.1 Install AWS EB CLI

```bash
pip install awsebcli
```

Or on Windows:
```bash
pip install awsebcli --user
```

#### 2.2 Configure AWS Credentials

```bash
aws configure
```

Enter:
- AWS Access Key ID: [Your NEW rotated key]
- AWS Secret Access Key: [Your NEW rotated secret]
- Default region: ap-south-1
- Default output format: json

#### 2.3 Deploy Main Backend

```bash
cd backend
eb init -p node.js-18 hackveda-backend --region ap-south-1
eb create hackveda-backend-env
```

Set environment variables:
```bash
eb setenv \
  PORT=5012 \
  DYNAMODB_USERS_TABLE=hackveda-users \
  AWS_REGION=ap-south-1 \
  TUTOR_AGENT_ID=SFOFRVB0QS \
  TUTOR_AGENT_ALIAS=B7QTD3ULOF \
  CURRICULUM_AGENT_ID=AA7UFR1MZO \
  CURRICULUM_AGENT_ALIAS=IVVZBZ2IQP \
  EVALUATION_AGENT_ID=ZJM4T6UZE1 \
  EVALUATION_AGENT_ALIAS=XNMJR5BSON \
  SKILL_MAPPING_AGENT_ID=3GE5HYWL4P \
  SKILL_MAPPING_AGENT_ALIAS=ZU5SYG5BP7 \
  CODING_AGENT_ID=BDZFCSFZJF \
  CODING_AGENT_ALIAS=VG8YGSHO1J \
  TEST_CASE_GENERATOR_AGENT_ID=RTGOKPWEMA \
  TEST_CASE_GENERATOR_AGENT_ALIAS=BIO17IXNOJ \
  CHATBOT_AGENT_ID=VLZSQPKDUV \
  CHATBOT_AGENT_ALIAS=7X8DN7HOOI \
  LEARNING_ASSISTANT_AGENT_ID=LNM6FK35II \
  LEARNING_ASSISTANT_AGENT_ALIAS=BIFYDVMJ6N \
  JWT_SECRET=my_super_secret_key_123 \
  JUDGE0_URL=http://13.234.32.75:2358
```

#### 2.4 Deploy Chatbot Backend

```bash
cd chatbotAWS
eb init -p node.js-18 hackveda-chatbot --region ap-south-1
eb create hackveda-chatbot-env
```

Set environment variables:
```bash
eb setenv \
  PORT=5005 \
  AWS_REGION=ap-south-1 \
  BEDROCK_AGENT_ID=OKPUEODEZC \
  BEDROCK_AGENT_ALIAS=3KNCV8OBXZ \
  AWS_ACCESS_KEY_ID=[your-new-key] \
  AWS_SECRET_ACCESS_KEY=[your-new-secret]
```

#### 2.5 Deploy PDF Learning Backend

```bash
cd HackathonPDF/backend
eb init -p node.js-18 hackveda-pdf --region ap-south-1
eb create hackveda-pdf-env
```

Set environment variables:
```bash
eb setenv \
  PORT=5006 \
  AWS_REGION=ap-south-1 \
  AWS_ACCESS_KEY_ID=[your-new-key] \
  AWS_SECRET_ACCESS_KEY=[your-new-secret] \
  BEDROCK_AGENT_ID=KGBXWL8FHG \
  BEDROCK_AGENT_ALIAS_ID=[your-alias] \
  S3_BUCKET_NAME=[your-bucket]
```

---

### Phase 3: Deploy Frontend to S3 + CloudFront

#### 3.1 Update Frontend API URLs

Create `frontend/.env.production`:
```env
VITE_API_URL=https://hackveda-backend-env.ap-south-1.elasticbeanstalk.com
VITE_CHATBOT_API_URL=https://hackveda-chatbot-env.ap-south-1.elasticbeanstalk.com
VITE_PDF_API_URL=https://hackveda-pdf-env.ap-south-1.elasticbeanstalk.com
```

Update `frontend/src/api/api.js`:
```javascript
const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:5012';
```

Update `frontend/src/pages/Sandbox.jsx` and `AIMentorDrawer.jsx`:
```javascript
const CHAT_API = import.meta.env.VITE_CHATBOT_API_URL + '/chat' || 'http://localhost:5005/chat';
```

Update `frontend/src/hooks/usePDFLearning.js`:
```javascript
const API_BASE = import.meta.env.VITE_PDF_API_URL + '/api' || '/api';
```

#### 3.2 Build Frontend

```bash
cd frontend
npm run build
```

#### 3.3 Create S3 Bucket for Frontend

```bash
aws s3 mb s3://hackveda-frontend --region ap-south-1
```

Enable static website hosting:
```bash
aws s3 website s3://hackveda-frontend --index-document index.html --error-document index.html
```

#### 3.4 Upload Build to S3

```bash
cd frontend
aws s3 sync dist/ s3://hackveda-frontend --delete
```

#### 3.5 Make Bucket Public

Create bucket policy (`bucket-policy.json`):
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
aws s3api put-bucket-policy --bucket hackveda-frontend --policy file://bucket-policy.json
```

#### 3.6 Create CloudFront Distribution (Optional but Recommended)

1. Go to AWS CloudFront Console
2. Create Distribution
3. Origin Domain: hackveda-frontend.s3-website.ap-south-1.amazonaws.com
4. Viewer Protocol Policy: Redirect HTTP to HTTPS
5. Default Root Object: index.html
6. Create Distribution
7. Note the CloudFront URL (e.g., d1234567890.cloudfront.net)

---

### Phase 4: Configure CORS

Update all backend CORS configurations to allow your frontend domain:

**backend/server.js:**
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'https://hackveda-frontend.s3-website.ap-south-1.amazonaws.com',
    'https://d1234567890.cloudfront.net' // Your CloudFront URL
  ],
  credentials: true
}));
```

**chatbotAWS/backend.js:**
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'https://hackveda-frontend.s3-website.ap-south-1.amazonaws.com',
    'https://d1234567890.cloudfront.net'
  ]
}));
```

**HackathonPDF/backend:** (similar update)

Redeploy backends:
```bash
cd backend && eb deploy
cd chatbotAWS && eb deploy
cd HackathonPDF/backend && eb deploy
```

---

### Phase 5: Configure IAM Roles (Better than Access Keys)

#### 5.1 Create IAM Role for Elastic Beanstalk

1. Go to IAM Console → Roles → Create Role
2. Select: AWS Service → Elastic Beanstalk
3. Attach policies:
   - `AmazonDynamoDBFullAccess`
   - `AmazonBedrockFullAccess`
   - `AmazonS3FullAccess`
4. Name: `hackveda-eb-role`
5. Create Role

#### 5.2 Attach Role to Elastic Beanstalk Environments

```bash
eb config
```

Add under `aws:autoscaling:launchconfiguration`:
```yaml
IamInstanceProfile: hackveda-eb-role
```

Save and deploy:
```bash
eb deploy
```

#### 5.3 Remove Hardcoded Credentials

Once IAM roles are attached, remove these from environment variables:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

The AWS SDK will automatically use the IAM role credentials.

---

## Cost Estimation (Monthly)

### Elastic Beanstalk (3 environments):
- t3.micro instances (3x): ~$25/month
- Load balancers (3x): ~$50/month
- **Subtotal: ~$75/month**

### S3 + CloudFront:
- S3 storage (1GB): ~$0.02/month
- CloudFront (10GB transfer): ~$1/month
- **Subtotal: ~$1/month**

### Existing Services:
- DynamoDB: Variable (pay per request)
- Judge0 EC2: Already running
- Bedrock Agents: Pay per invocation

**Total Estimated Cost: ~$76-100/month**

---

## Alternative: Single EC2 Deployment (Lower Cost)

If budget is tight, deploy all backends on a single EC2 instance:

### 1. Launch EC2 Instance
- Instance type: t3.medium (2 vCPU, 4GB RAM)
- AMI: Amazon Linux 2023
- Security Group: Allow ports 80, 443, 5005, 5006, 5012
- Cost: ~$30/month

### 2. Install Dependencies
```bash
# Connect to EC2
ssh -i your-key.pem ec2-user@your-ec2-ip

# Install Node.js
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo yum install -y nginx
```

### 3. Deploy Backends
```bash
# Clone your repository
git clone your-repo-url
cd your-repo

# Setup main backend
cd backend
npm install
pm2 start server.js --name hackveda-backend

# Setup chatbot backend
cd ../chatbotAWS
npm install
pm2 start backend.js --name hackveda-chatbot

# Setup PDF backend
cd ../HackathonPDF/backend
npm install
pm2 start src/index.ts --name hackveda-pdf

# Save PM2 configuration
pm2 save
pm2 startup
```

### 4. Configure Nginx as Reverse Proxy

Create `/etc/nginx/conf.d/hackveda.conf`:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    # Main backend
    location /api/ {
        proxy_pass http://localhost:5012/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Chatbot backend
    location /chatbot/ {
        proxy_pass http://localhost:5005/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # PDF backend
    location /pdf/ {
        proxy_pass http://localhost:5006/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Restart Nginx:
```bash
sudo systemctl restart nginx
sudo systemctl enable nginx
```

### 5. Deploy Frontend to S3 (same as above)

**Single EC2 Cost: ~$30-35/month** (much cheaper!)

---

## Post-Deployment Checklist

- [ ] All AWS credentials rotated
- [ ] No credentials in code (using .env files)
- [ ] .env files in .gitignore
- [ ] All backends deployed and running
- [ ] Frontend deployed to S3
- [ ] CloudFront distribution created (optional)
- [ ] CORS configured correctly
- [ ] IAM roles attached (no hardcoded keys)
- [ ] Environment variables set in EB
- [ ] DynamoDB accessible from backends
- [ ] Judge0 accessible from main backend
- [ ] Bedrock Agents accessible
- [ ] SSL certificate configured (optional)
- [ ] Domain name configured (optional)
- [ ] Monitoring and logging enabled

---

## Monitoring and Maintenance

### CloudWatch Logs
All Elastic Beanstalk logs are automatically sent to CloudWatch.

View logs:
```bash
eb logs
```

### Health Monitoring
```bash
eb health
```

### Scaling
```bash
eb scale 2  # Scale to 2 instances
```

### Updates
```bash
# Update code
git pull
eb deploy
```

---

## Troubleshooting

### Backend not starting
- Check logs: `eb logs`
- Verify environment variables: `eb printenv`
- Check IAM role permissions

### CORS errors
- Update CORS configuration in backends
- Redeploy: `eb deploy`

### DynamoDB access denied
- Attach `AmazonDynamoDBFullAccess` to IAM role
- Verify region matches

### Bedrock Agent errors
- Verify agent IDs and aliases
- Check IAM permissions for Bedrock
- Ensure region is correct

---

## Security Best Practices

1. **Never commit credentials** - Use .env files and .gitignore
2. **Use IAM roles** instead of access keys when possible
3. **Rotate credentials regularly** (every 90 days)
4. **Enable MFA** on AWS account
5. **Use HTTPS** for all production traffic
6. **Implement rate limiting** on APIs
7. **Enable CloudWatch alarms** for unusual activity
8. **Regular security audits** with AWS Trusted Advisor

---

## Next Steps

1. Fix security issues (rotate credentials)
2. Choose deployment strategy (Elastic Beanstalk or EC2)
3. Deploy backends
4. Deploy frontend
5. Configure domain name (optional)
6. Set up SSL certificate (optional)
7. Enable monitoring and alerts
8. Test thoroughly
9. Go live!

---

## Support Resources

- AWS Elastic Beanstalk Docs: https://docs.aws.amazon.com/elasticbeanstalk/
- AWS S3 Static Website: https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html
- AWS CloudFront: https://docs.aws.amazon.com/cloudfront/
- PM2 Documentation: https://pm2.keymetrics.io/docs/usage/quick-start/

---

**Good luck with your deployment! 🚀**
