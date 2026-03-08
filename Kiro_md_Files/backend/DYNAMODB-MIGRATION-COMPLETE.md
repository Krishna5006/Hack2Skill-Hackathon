# DynamoDB Migration Complete ✅

## Summary
Successfully migrated from MongoDB (Mongoose) to AWS DynamoDB for user data storage.

## Changes Made

### 1. Installed Dependencies
```bash
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb uuid
```

### 2. Created DynamoDB Configuration
- **File**: `src/config/dynamodb.js`
- Sets up DynamoDB client with AWS SDK v3
- Uses IAM role credentials when running on EC2
- Falls back to environment credentials for local development

### 3. Created DynamoDB User Model
- **File**: `src/models/UserDynamoDB.js`
- Replaces Mongoose User model
- Key changes:
  - Uses `userId` (UUID) instead of MongoDB `_id`
  - Implements static methods: `create()`, `findById()`, `findByEmail()`, `update()`
  - Implements instance methods: `save()`, `comparePassword()`
  - Uses GSI `email-index` for email lookups

### 4. Updated Controllers
- **authController.js**: Changed `user._id` to `user.userId`
- **profileController.js**: Removed `.select()` method (not needed in DynamoDB)
- **learningController.js**: 
  - Updated import to use UserDynamoDB
  - Removed all `.select()` calls
  - Changed `user._id` to `user.userId`

### 5. Updated Services
- **skillService.js**: Updated import to use UserDynamoDB

### 6. Updated Server Configuration
- **server.js**: Removed MongoDB connection, added DynamoDB logging
- **.env**: Added DynamoDB configuration, commented out MongoDB URI

### 7. Created Tests
- **File**: `src/models/__tests__/UserDynamoDB.test.js`
- Tests for create, findById, findByEmail, update, and save operations

## DynamoDB Table Structure

### Table: `hackveda-users`
- **Partition Key**: `userId` (String)
- **GSI**: `email-index`
  - Partition Key: `email` (String)
- **Billing Mode**: On-demand

### User Schema
```javascript
{
  userId: "uuid-v4",
  name: "string",
  email: "string",
  password: "hashed-string",
  globalSkills: [
    {
      skill: "string",
      confidence: number,
      quizAttempts: number,
      codeAttempts: number,
      lastUpdated: "ISO-date",
      confidenceHistory: [
        { value: number, date: "ISO-date" }
      ]
    }
  ],
  roadmaps: [
    {
      subject: "string",
      level: "string",
      goals: "string",
      topics: [...]
    }
  ],
  createdAt: "ISO-date",
  updatedAt: "ISO-date"
}
```

## Environment Variables

### Required in .env:
```env
DYNAMODB_USERS_TABLE=hackveda-users
AWS_REGION=ap-south-1
```

### For Local Development:
If running locally (not on EC2), you'll need AWS credentials:
```env
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
```

### For EC2 Deployment:
- Attach IAM role to EC2 instance with DynamoDB permissions
- No credentials needed in .env (automatic via IAM role)

## IAM Permissions Required

The EC2 instance or local AWS credentials need these DynamoDB permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:UpdateItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-south-1:*:table/hackveda-users",
        "arn:aws:dynamodb:ap-south-1:*:table/hackveda-users/index/email-index"
      ]
    }
  ]
}
```

## Testing the Migration

### 1. Start the Server
```bash
cd Hackveda/backend
npm start
```

You should see:
```
✅ Using DynamoDB for data storage
📊 Table: hackveda-users
🌍 Region: ap-south-1
Backend running on port 5012
```

### 2. Test User Registration
```bash
curl -X POST http://localhost:5012/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "password": "password123"
  }'
```

Expected response:
```json
{
  "message": "User registered successfully",
  "userId": "uuid-here"
}
```

### 3. Test User Login
```bash
curl -X POST http://localhost:5012/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

Expected response:
```json
{
  "message": "Login successful",
  "token": "jwt-token-here",
  "userId": "uuid-here",
  "name": "Test User"
}
```

### 4. Run Tests
```bash
npm test
```

## Migration Checklist

- [x] Install AWS SDK packages
- [x] Create DynamoDB client configuration
- [x] Create UserDynamoDB model
- [x] Update authController
- [x] Update profileController
- [x] Update learningController
- [x] Update skillService
- [x] Update server.js
- [x] Update .env configuration
- [x] Create test file
- [ ] Test user registration endpoint
- [ ] Test user login endpoint
- [ ] Test roadmap creation
- [ ] Test content generation
- [ ] Test quiz submission
- [ ] Test code submission
- [ ] Deploy to EC2 with IAM role

## Key Differences: MongoDB vs DynamoDB

| Feature | MongoDB (Mongoose) | DynamoDB |
|---------|-------------------|----------|
| Primary Key | `_id` (ObjectId) | `userId` (UUID) |
| Email Lookup | Direct query | GSI query |
| Field Selection | `.select("field")` | Returns all fields |
| Connection | Requires connection | No connection needed |
| Schema | Mongoose schema | JavaScript class |
| Updates | `findByIdAndUpdate()` | `UpdateCommand` |
| Credentials | Connection string | IAM role or env vars |

## Rollback Plan

If you need to rollback to MongoDB:

1. Restore `.env`:
   ```env
   MONGO_URI=mongodb://127.0.0.1:27017/ai_mentor
   ```

2. Restore `server.js`:
   ```javascript
   import connectDB from "./src/config/db.js";
   await connectDB();
   ```

3. Update all imports:
   ```javascript
   import User from "../models/User.js";
   ```

4. Change `userId` back to `_id` in controllers

## Next Steps

1. **Test all endpoints** to ensure functionality
2. **Deploy to EC2** with IAM role attached
3. **Monitor DynamoDB** usage and costs
4. **Remove MongoDB** dependencies if everything works:
   ```bash
   npm uninstall mongoose mongodb
   ```
5. **Delete old User.js** model file after confirming migration success

## Cost Optimization

DynamoDB on-demand pricing:
- **Write**: $1.25 per million write requests
- **Read**: $0.25 per million read requests
- **Storage**: $0.25 per GB-month

For a demo/prototype with ~1000 users:
- Estimated cost: **$0.50 - $2.00/month**
- Much cheaper than running MongoDB on EC2

## Support

If you encounter issues:
1. Check CloudWatch logs for DynamoDB errors
2. Verify IAM permissions
3. Confirm table name and region in .env
4. Test AWS credentials with AWS CLI:
   ```bash
   aws dynamodb describe-table --table-name hackveda-users
   ```
