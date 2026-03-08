# HackVeda AWS Deployment Checklist

Follow these steps in order to deploy your application to AWS.

---

## Phase 1: Pre-Deployment (On Your PC)

### ☐ 1. Security Cleanup
```bash
# Delete exposed credentials file
rm rishu.119484@stu.upes.ac.in_accessKeys.csv

# Verify no credentials in code
grep -r "AKIA" . --exclude-dir=node_modules
```

### ☐ 2. Test Locally
```bash
# Test main backend
cd backend
node server.js
# Should start on port 5012

# Test chatbot backend
cd ../chatbotAWS
npm install dotenv
node backend.js
# Should start on port 5005

# Test PDF backend
cd ../HackathonPDF/backend
npm run dev
# Should start on port 5006

# Test frontend
cd ../../frontend
npm run dev
# Should start on port 5010
```

### ☐ 3. Commit and Push Code
```bash
git add .
git commit -m "Prepare for AWS deployment"
git push origin main
```

---

## Phase 2: Launch EC2 Instance

### ☐ 4. Launch EC2 Instance
Follow: `ec2-launch-guide.md`

**Configuration:**
- Name: hackveda-app-server
- AMI: Amazon Linux 2023
- Instance type: t3.medium
- Key pair: Create/select key pair
- Security group: Allow ports 22, 80, 443, 5005, 5006, 5012
- Storage: 30 GiB

**Save these details:**
- Instance ID: ________________
- Public IP: ________________
- Key file: ________________

### ☐ 5. Connect to EC2
```bash
ssh -i path/to/your-key.pem ec2-user@YOUR_PUBLIC_IP
```

---

## Phase 3: Setup EC2 Instance

### ☐ 6. Run Setup Script
```bash
# Download setup script
curl -O https://raw.githubusercontent.com/your-repo/ec2-setup-script.sh

# Or copy-paste the script content and save as ec2-setup-script.sh
nano ec2-setup-script.sh
# Paste content, Ctrl+X, Y, Enter

# Make executable
chmod +x ec2-setup-script.sh

# Run script
./ec2-setup-script.sh
```

**What it does:**
- Updates system
- Installs Node.js 18
- Installs Git
- Installs PM2
- Installs Nginx
- Configures AWS CLI

### ☐ 7. Clone Your Repository
```bash
cd /home/ec2-user
git clone YOUR_REPOSITORY_URL hackveda
cd hackveda
```

---

## Phase 4: Deploy Backends

### ☐ 8. Run Backend Deployment Script
```bash
# Make script executable
chmod +x deploy-backends.sh

# Run script
./deploy-backends.sh
```

**What it does:**
- Installs dependencies for all backends
- Starts all backends with PM2
- Configures PM2 to start on boot
- Tests all endpoints

### ☐ 9. Verify Backends Running
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

### ☐ 10. Test Backend Endpoints
```bash
curl http://localhost:5012/
curl http://localhost:5005/
curl http://localhost:5006/
```

All should return success responses.

---

## Phase 5: Configure Nginx

### ☐ 11. Run Nginx Configuration Script
```bash
# Make script executable
chmod +x nginx-config.sh

# Run script
./nginx-config.sh
```

**What it does:**
- Creates Nginx configuration
- Sets up reverse proxy for all backends
- Starts and enables Nginx
- Shows your public URLs

### ☐ 12. Test Nginx Endpoints
```bash
# Get your public IP
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)

# Test endpoints
curl http://$PUBLIC_IP/health
curl http://$PUBLIC_IP/api/
curl http://$PUBLIC_IP/chatbot/
curl http://$PUBLIC_IP/pdf/
```

---

## Phase 6: Update Backend CORS (On Your PC)

### ☐ 13. Update CORS in Main Backend
Edit `backend/server.js`:
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'http://YOUR_EC2_PUBLIC_IP',
    'http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com'
  ],
  credentials: true
}));
```

### ☐ 14. Update CORS in Chatbot Backend
Edit `chatbotAWS/backend.js`:
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'http://YOUR_EC2_PUBLIC_IP',
    'http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com'
  ]
}));
```

### ☐ 15. Update CORS in PDF Backend
Edit `HackathonPDF/backend/src/index.ts` (or wherever CORS is configured):
```javascript
app.use(cors({
  origin: [
    'http://localhost:5010',
    'http://YOUR_EC2_PUBLIC_IP',
    'http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com'
  ]
}));
```

### ☐ 16. Push CORS Updates
```bash
git add .
git commit -m "Update CORS for production"
git push origin main
```

### ☐ 17. Pull Updates on EC2
```bash
# On EC2
cd /home/ec2-user/hackveda
git pull

# Restart backends
pm2 restart all
```

---

## Phase 7: Deploy Frontend to S3

### ☐ 18. Create Frontend Production Config (On Your PC)
Create `frontend/.env.production`:
```env
VITE_API_URL=http://YOUR_EC2_PUBLIC_IP/api
VITE_CHATBOT_API_URL=http://YOUR_EC2_PUBLIC_IP/chatbot
VITE_PDF_API_URL=http://YOUR_EC2_PUBLIC_IP/pdf
```

### ☐ 19. Update Frontend API URLs
See `DEPLOYMENT-QUICK-START.md` Step 2 for detailed code changes.

### ☐ 20. Build Frontend
```bash
cd frontend
npm run build
```

### ☐ 21. Create S3 Bucket
```bash
aws s3 mb s3://hackveda-frontend --region ap-south-1
```

### ☐ 22. Enable Static Website Hosting
```bash
aws s3 website s3://hackveda-frontend \
  --index-document index.html \
  --error-document index.html
```

### ☐ 23. Upload Build to S3
```bash
cd frontend
aws s3 sync dist/ s3://hackveda-frontend --delete
```

### ☐ 24. Make Bucket Public
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

### ☐ 25. Get S3 Website URL
```bash
echo "http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com"
```

---

## Phase 8: Testing

### ☐ 26. Test Frontend
Open: `http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com`

### ☐ 27. Test All Features
- [ ] Login/Signup
- [ ] Create Roadmap
- [ ] View Topics
- [ ] Take Quiz
- [ ] Coding Challenge
- [ ] AI Mentor (Dashboard)
- [ ] Sandbox AI Assistant
- [ ] PDF Learning

### ☐ 28. Check Browser Console
- No CORS errors
- No 404 errors
- All API calls successful

### ☐ 29. Check Backend Logs
```bash
# On EC2
pm2 logs
```

---

## Phase 9: Monitoring & Maintenance

### ☐ 30. Set Up Monitoring
```bash
# View PM2 dashboard
pm2 monit

# View logs
pm2 logs

# View Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

### ☐ 31. Create Backup Script (Optional)
```bash
# Backup DynamoDB table
aws dynamodb create-backup \
  --table-name hackveda-users \
  --backup-name hackveda-users-backup-$(date +%Y%m%d)
```

---

## Deployment Complete! 🎉

### Your Application URLs:
- **Frontend**: http://hackveda-frontend.s3-website.ap-south-1.amazonaws.com
- **Backend API**: http://YOUR_EC2_IP/api
- **Chatbot API**: http://YOUR_EC2_IP/chatbot
- **PDF API**: http://YOUR_EC2_IP/pdf

### Useful Commands:

**On EC2:**
```bash
# View service status
pm2 status

# View logs
pm2 logs

# Restart all services
pm2 restart all

# Stop all services
pm2 stop all

# Update code
cd /home/ec2-user/hackveda
git pull
pm2 restart all
```

**On Your PC:**
```bash
# Update frontend
cd frontend
npm run build
aws s3 sync dist/ s3://hackveda-frontend --delete
```

---

## Troubleshooting

### Backend not accessible
```bash
pm2 logs
pm2 restart all
```

### CORS errors
- Update CORS configuration
- Restart backends: `pm2 restart all`

### Frontend not loading
- Check S3 bucket policy
- Verify API URLs in .env.production
- Check browser console

### DynamoDB errors
- Verify AWS CLI configured: `aws configure list`
- Test access: `aws dynamodb list-tables`

---

## Cost Monitoring

Check your AWS costs:
- https://console.aws.amazon.com/billing/

Expected monthly cost: ~$36
- EC2 t3.medium: ~$30
- S3 storage: ~$1
- Data transfer: ~$2-5
- DynamoDB: Pay per request
- Bedrock: Pay per invocation

---

## Next Steps (Optional)

- [ ] Add custom domain with Route 53
- [ ] Add SSL certificate with ACM
- [ ] Set up CloudFront for frontend
- [ ] Enable CloudWatch monitoring
- [ ] Set up automated backups
- [ ] Configure auto-scaling (if needed)

---

**Congratulations! Your application is now live on AWS! 🚀**
