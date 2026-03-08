# EC2 Instance Launch Guide

## Step 1: Go to EC2 Console

Open: https://ap-south-1.console.aws.amazon.com/ec2/home?region=ap-south-1

## Step 2: Launch Instance

Click **"Launch Instance"** button (orange button at top right)

## Step 3: Configure Instance

### Name and Tags
```
Name: hackveda-app-server
```

### Application and OS Images (AMI)
- **Quick Start**: Amazon Linux
- **Amazon Machine Image (AMI)**: Amazon Linux 2023 AMI
- **Architecture**: 64-bit (x86)

### Instance Type
- **Instance type**: t3.medium
  - 2 vCPU
  - 4 GiB Memory
  - Cost: ~$0.0416/hour (~$30/month)

### Key Pair (login)
- If you have an existing key pair: Select it
- If not, click **"Create new key pair"**:
  - Name: `hackveda-key`
  - Key pair type: RSA
  - Private key file format: .pem (for Mac/Linux) or .ppk (for Windows PuTTY)
  - Click **"Create key pair"**
  - **IMPORTANT**: Save the downloaded file securely!

### Network Settings
Click **"Edit"** and configure:

**VPC**: Default VPC (or your existing VPC)

**Subnet**: No preference (or choose one)

**Auto-assign public IP**: Enable

**Firewall (security groups)**: Create security group
- **Security group name**: hackveda-sg
- **Description**: Security group for HackVeda application

**Inbound Security Group Rules** - Add these rules:

1. **SSH** (for connecting to server)
   - Type: SSH
   - Protocol: TCP
   - Port: 22
   - Source: My IP (automatically detects your IP)

2. **HTTP** (for web traffic)
   - Type: HTTP
   - Protocol: TCP
   - Port: 80
   - Source: Anywhere (0.0.0.0/0)

3. **HTTPS** (for secure web traffic)
   - Type: HTTPS
   - Protocol: TCP
   - Port: 443
   - Source: Anywhere (0.0.0.0/0)

4. **Custom TCP - Main Backend**
   - Type: Custom TCP
   - Protocol: TCP
   - Port: 5012
   - Source: Anywhere (0.0.0.0/0)

5. **Custom TCP - Chatbot Backend**
   - Type: Custom TCP
   - Protocol: TCP
   - Port: 5005
   - Source: Anywhere (0.0.0.0/0)

6. **Custom TCP - PDF Backend**
   - Type: Custom TCP
   - Protocol: TCP
   - Port: 5006
   - Source: Anywhere (0.0.0.0/0)

### Configure Storage
- **Size**: 30 GiB
- **Volume type**: gp3 (General Purpose SSD)
- **Delete on termination**: Yes (checked)

### Advanced Details
Leave as default (no changes needed)

## Step 4: Launch

1. Review your configuration in the **Summary** panel on the right
2. Click **"Launch instance"** (orange button)
3. Wait for the instance to launch (takes 1-2 minutes)

## Step 5: Get Instance Details

1. Click **"View all instances"**
2. Find your instance: `hackveda-app-server`
3. Wait until **Instance state** shows "Running" (green)
4. Note down these details:
   - **Instance ID**: i-xxxxxxxxxxxxxxxxx
   - **Public IPv4 address**: xx.xx.xx.xx (this is your server IP)
   - **Public IPv4 DNS**: ec2-xx-xx-xx-xx.ap-south-1.compute.amazonaws.com

## Step 6: Test SSH Connection

### On Windows (using Git Bash or PowerShell):
```bash
ssh -i path/to/hackveda-key.pem ec2-user@YOUR_PUBLIC_IP
```

### On Mac/Linux:
```bash
# First, set correct permissions on key file
chmod 400 path/to/hackveda-key.pem

# Then connect
ssh -i path/to/hackveda-key.pem ec2-user@YOUR_PUBLIC_IP
```

If you see a warning about authenticity, type `yes` and press Enter.

You should see:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'

[ec2-user@ip-xxx-xx-xx-xx ~]$
```

**Success!** You're now connected to your EC2 instance.

## Troubleshooting

### "Permission denied (publickey)"
- Check key file permissions: `chmod 400 hackveda-key.pem`
- Verify you're using the correct key file
- Verify you're using `ec2-user` as username (not `root` or `ubuntu`)

### "Connection timed out"
- Check Security Group allows SSH (port 22) from your IP
- Verify instance is running
- Check your internet connection

### "Host key verification failed"
- Remove old host key: `ssh-keygen -R YOUR_PUBLIC_IP`
- Try connecting again

## Next Steps

Once connected, proceed to **Step 3** in `DEPLOYMENT-QUICK-START.md` to install dependencies.

---

**Your EC2 Instance Details:**
- Instance ID: ________________
- Public IP: ________________
- Key file location: ________________

Save these details for future reference!
