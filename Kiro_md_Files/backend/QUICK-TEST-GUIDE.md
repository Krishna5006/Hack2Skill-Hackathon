# Quick Test Guide - DynamoDB Migration

## 🚀 Quick Start (5 minutes)

### Step 1: Test DynamoDB Connection
```bash
cd Hackveda/backend
node test-dynamodb-connection.js
```

**Expected Output:**
```
🧪 Testing DynamoDB Connection...
✅ User created: { userId: '...', name: 'Test User', email: '...' }
✅ User found by ID: { userId: '...', name: 'Test User' }
✅ User found by email: { userId: '...', email: '...' }
✅ User updated: { userId: '...', name: 'Updated Test User' }
✅ Instance saved with skills
✅ Verified skills: [...]
✅ All tests passed! DynamoDB connection is working.
```

### Step 2: Start the Server
```bash
npm start
```

**Expected Output:**
```
✅ Using DynamoDB for data storage
📊 Table: hackveda-users
🌍 Region: ap-south-1
Backend running on port 5012
```

### Step 3: Test Registration (New Terminal)
```bash
curl -X POST http://localhost:5012/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"test123"}'
```

**Expected Response:**
```json
{
  "message": "User registered successfully",
  "userId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Step 4: Test Login
```bash
curl -X POST http://localhost:5012/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123"}'
```

**Expected Response:**
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Test User"
}
```

## ❌ Troubleshooting

### Error: "ResourceNotFoundException"
**Problem:** DynamoDB table doesn't exist

**Solution:**
1. Go to AWS Console → DynamoDB
2. Create table `hackveda-users`
3. Partition key: `userId` (String)
4. Create GSI: `email-index` with partition key `email` (String)

### Error: "AccessDeniedException"
**Problem:** No AWS credentials or insufficient permissions

**Solution for Local Development:**
```bash
# Configure AWS credentials
aws configure
# Enter: Access Key ID, Secret Access Key, Region (ap-south-1)
```

**Solution for EC2:**
1. Create IAM role with DynamoDB permissions
2. Attach role to EC2 instance
3. No credentials needed in .env

### Error: "Cannot find module"
**Problem:** Dependencies not installed

**Solution:**
```bash
npm install
```

### Server starts but endpoints fail
**Problem:** Wrong table name or region

**Solution:** Check `.env` file:
```env
DYNAMODB_USERS_TABLE=hackveda-users
AWS_REGION=ap-south-1
```

## 🔍 Verify DynamoDB Table

### Using AWS CLI
```bash
# Describe table
aws dynamodb describe-table --table-name hackveda-users

# List items (first 10)
aws dynamodb scan --table-name hackveda-users --max-items 10

# Get specific user by userId
aws dynamodb get-item \
  --table-name hackveda-users \
  --key '{"userId":{"S":"your-user-id-here"}}'
```

### Using AWS Console
1. Go to AWS Console → DynamoDB
2. Click on `hackveda-users` table
3. Click "Explore table items"
4. View all users

## 📊 Test All Endpoints

### 1. Register
```bash
curl -X POST http://localhost:5012/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@test.com","password":"pass123"}'
```

### 2. Login
```bash
curl -X POST http://localhost:5012/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@test.com","password":"pass123"}'
```

Save the token from response!

### 3. Create Roadmap (use token from login)
```bash
curl -X POST http://localhost:5012/api/learning/roadmap \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{"subject":"JavaScript","level":"beginner","goals":"Learn basics"}'
```

### 4. Get Roadmaps
```bash
curl -X GET http://localhost:5012/api/learning/roadmaps \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### 5. Get Skills
```bash
curl -X GET http://localhost:5012/api/profile/skills \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

## ✅ Success Indicators

- [ ] Connection test passes
- [ ] Server starts without errors
- [ ] Registration creates user in DynamoDB
- [ ] Login returns JWT token with userId
- [ ] Roadmap creation works
- [ ] Skills are tracked correctly
- [ ] No MongoDB connection errors

## 🎯 Next Steps

1. **Test with Frontend**: Start frontend and test full flow
2. **Deploy to EC2**: Follow deployment guide
3. **Monitor Costs**: Check DynamoDB usage in AWS Console
4. **Remove MongoDB**: Uninstall mongoose if everything works

## 📞 Quick Commands Reference

```bash
# Test connection
node test-dynamodb-connection.js

# Start server
npm start

# Run tests
npm test

# Check AWS credentials
aws sts get-caller-identity

# Describe DynamoDB table
aws dynamodb describe-table --table-name hackveda-users

# View server logs (if using PM2)
pm2 logs hackveda-backend
```

---

**Everything working?** Great! You're ready to deploy to EC2. 🚀
