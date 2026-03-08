# HackVeda AI - Quick Setup Guide

This is a condensed setup guide. For full documentation, see [README.md](README.md).

## 🚀 Quick Setup (5 Steps)

### Step 1: AWS Prerequisites

1. **Create DynamoDB Table**
   - Table name: `hackveda-users`
   - Partition key: `email` (String)

2. **Create S3 Bucket for Knowledge Base**
   ```bash
   aws s3 mb s3://your-kb-bucket-name
   aws s3 sync knowlegde-base/ s3://your-kb-bucket-name/knowledge-base/
   ```

3. **Create Knowledge Base in Bedrock**
   - Go to AWS Bedrock → Knowledge bases
   - Create new KB pointing to your S3 bucket
   - Note the Knowledge Base ID

### Step 2: Create 8 Bedrock Agents

Use instructions from `Instructions_for_agents/` folder:

| Agent | Instruction File | KB Files Needed |
|-------|-----------------|-----------------|
| Tutor | `TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt` | `teaching-rules.txt` |
| Curriculum | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `canonical_skills.json` |
| Evaluation | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `evaluation-policy.txt` |
| Skill Mapping | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `skill-mapping-rules.txt` |
| Coding | `CODING-AGENT-INSTRUCTION-FINAL.txt` | `coding-challenge-rules-compact.txt` |
| Test Generator | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `test-case-generation-rules.txt` |
| Chatbot | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `chatbot-assistant-rules.txt` |
| Learning Assistant | `AWS-COPY-PASTE-INSTRUCTIONS.txt` | `learning-assistant-rules.txt` |

For each agent:
- Create in AWS Bedrock Console
- Attach relevant KB files
- Create an alias
- Note Agent ID and Alias ID

### Step 3: Configure Environment Files

**4 files to configure:**

#### 1. `backend/.env`
```env
PORT=5012
AWS_REGION=ap-south-1
DYNAMODB_USERS_TABLE=hackveda-users

TUTOR_AGENT_ID=xxx
TUTOR_AGENT_ALIAS=xxx
CURRICULUM_AGENT_ID=xxx
CURRICULUM_AGENT_ALIAS=xxx
EVALUATION_AGENT_ID=xxx
EVALUATION_AGENT_ALIAS=xxx
SKILL_MAPPING_AGENT_ID=xxx
SKILL_MAPPING_AGENT_ALIAS=xxx
CODING_AGENT_ID=xxx
CODING_AGENT_ALIAS=xxx
TEST_CASE_GENERATOR_AGENT_ID=xxx
TEST_CASE_GENERATOR_AGENT_ALIAS=xxx
CHATBOT_AGENT_ID=xxx
CHATBOT_AGENT_ALIAS=xxx
LEARNING_ASSISTANT_AGENT_ID=xxx
LEARNING_ASSISTANT_AGENT_ALIAS=xxx

JWT_SECRET=your-secret-key
JUDGE0_URL=http://your-judge0-ip:2358
```

#### 2. `chatbotAWS/.env`
```env
AWS_REGION=ap-south-1
BEDROCK_AGENT_ID=your-chatbot-agent-id
BEDROCK_AGENT_ALIAS=your-chatbot-alias
PORT=5005
```

#### 3. `HackathonPDF/backend/.env`
```env
AWS_REGION=ap-south-1
BEDROCK_AGENT_ID=your-learning-assistant-id
BEDROCK_AGENT_ALIAS=your-learning-assistant-alias
BEDROCK_KNOWLEDGE_BASE_ID=your-kb-id
S3_BUCKET_NAME=your-pdf-bucket
PORT=5006
```

#### 4. `frontend/.env`
```env
VITE_API_URL=http://localhost:5012/api
VITE_CHATBOT_URL=http://localhost:5005/chatbot
VITE_PDF_URL=http://localhost:5006/pdf
```

### Step 4: Install Dependencies

```bash
# Backend
cd backend && npm install

# Chatbot
cd ../chatbotAWS && npm install

# PDF Service
cd ../HackathonPDF/backend && npm install

# Frontend
cd ../../frontend && npm install
```

### Step 5: Start All Services

Open 4 terminals:

```bash
# Terminal 1 - Backend
cd backend && npm start

# Terminal 2 - Chatbot
cd chatbotAWS && node backend.js

# Terminal 3 - PDF Service
cd HackathonPDF/backend && npm run dev

# Terminal 4 - Frontend
cd frontend && npm run dev
```

Access: http://localhost:5173

---

## 🔍 Verification Checklist

- [ ] All 4 `.env` files configured
- [ ] DynamoDB table exists
- [ ] S3 bucket created with KB files
- [ ] Knowledge Base synced in AWS
- [ ] 8 Bedrock agents created with aliases
- [ ] AWS CLI configured (`aws sts get-caller-identity`)
- [ ] All dependencies installed
- [ ] All 4 services running
- [ ] Can access frontend at localhost:5173
- [ ] Can create account and login

---

## 🆘 Quick Troubleshooting

**Services won't start?**
- Check `.env` files exist and are configured
- Verify AWS credentials: `aws sts get-caller-identity`
- Check ports aren't already in use

**Agents not responding?**
- Verify Agent IDs and Aliases in `.env`
- Check agent status in AWS Bedrock Console
- Ensure IAM permissions for Bedrock

**Database errors?**
- Verify DynamoDB table name matches `.env`
- Check IAM permissions for DynamoDB
- Ensure table exists in correct region

---

## 📚 Additional Resources

- Full documentation: [README.md](README.md)
- Agent instructions: `Instructions_for_agents/`
- Knowledge base files: `knowlegde-base/`
- Detailed guides: `Kiro_md_Files/`

---

**Need help?** Check the main README.md for detailed troubleshooting and documentation.
