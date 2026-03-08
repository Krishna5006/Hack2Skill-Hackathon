# HackVeda AI - AI-Powered Learning Platform

An intelligent learning platform that combines personalized roadmaps, AI mentorship, coding challenges, and PDF-based learning to create a comprehensive educational experience.

## 🌟 Features

### 1. **Personalized Learning Roadmaps**
- AI-generated custom learning paths based on your goals and skill level
- Topic-by-topic breakdown with subtopics
- Progress tracking and completion status
- Interactive lessons with theory and practical examples

### 2. **AI-Powered Coding Challenges**
- Real-time code execution using Judge0
- Support for multiple programming languages (Python, C++, Java, JavaScript)
- AI-generated test cases and problem descriptions
- Instant feedback and evaluation
- Code editor with syntax highlighting

### 3. **Interactive Quizzes**
- AI-generated quizzes based on learning content
- Multiple-choice questions with instant feedback
- Score tracking and performance analytics
- AI Assistant available after quiz submission

### 4. **AI Mentor & Chatbot**
- 24/7 AI assistant powered by AWS Bedrock
- Context-aware responses
- Available in Dashboard and Sandbox modes
- Helps with coding questions, debugging, and learning guidance

### 5. **PDF Learning Assistant**
- Upload any PDF document for AI-powered learning
- Four learning modes:
  - **Teach**: Get detailed explanations of concepts
  - **Plan**: Generate structured learning roadmaps
  - **Quiz**: Create practice quizzes from content
  - **Summary**: Get concise summaries
- Powered by AWS Bedrock Knowledge Base

### 6. **Knowledge Graph Visualization**
- Interactive skill dependency graph
- Visual representation of learning paths
- Prerequisite tracking
- Skill proficiency levels

### 7. **User Dashboard**
- Track learning progress
- View completed and ongoing roadmaps
- Access all features from a unified interface
- Personalized recommendations

## 🛠️ Tech Stack

### Frontend
- **React** - UI framework
- **Vite** - Build tool and dev server
- **React Router** - Client-side routing
- **Axios** - HTTP client
- **React Markdown** - Markdown rendering
- **Lucide React** - Icon library
- **Monaco Editor** - Code editor component

### Backend
- **Node.js** - Runtime environment
- **Express.js** - Web framework
- **AWS Bedrock** - AI/ML services (8 specialized agents)
- **AWS DynamoDB** - NoSQL database for user data
- **AWS S3** - PDF storage and static hosting
- **Judge0** - Code execution engine
- **JWT** - Authentication
- **bcrypt** - Password hashing

### AI Agents (AWS Bedrock)
1. **Tutor Agent** - Personalized teaching
2. **Curriculum Agent** - Learning path generation
3. **Evaluation Agent** - Assessment and feedback
4. **Skill Mapping Agent** - Skill analysis
5. **Coding Agent** - Coding challenge generation
6. **Test Case Generator** - Automated test creation
7. **Chatbot Agent** - Conversational AI
8. **Learning Assistant** - PDF-based learning

### Infrastructure
- **AWS EC2** - Backend hosting (t3.medium)
- **AWS S3** - Frontend hosting (static website)
- **Nginx** - Reverse proxy
- **PM2** - Process management
- **IAM Roles** - Secure AWS access

## 📁 Project Structure

```
AiMentor/
├── frontend/                 # React frontend
│   ├── src/
│   │   ├── api/             # API client functions
│   │   ├── components/      # Reusable components
│   │   ├── pages/           # Page components
│   │   ├── hooks/           # Custom React hooks
│   │   └── config/          # Configuration files
│   └── dist/                # Production build
│
├── backend/                  # Main Express backend
│   ├── src/
│   │   ├── agents/          # AI agent integrations
│   │   ├── controllers/     # Route controllers
│   │   ├── models/          # Data models
│   │   ├── routes/          # API routes
│   │   ├── services/        # Business logic
│   │   ├── middleware/      # Express middleware
│   │   └── config/          # Backend configuration
│   └── .env                 # Environment variables
│
├── chatbotAWS/              # Chatbot backend service
│   └── backend.js           # Standalone chatbot server
│
└── HackathonPDF/            # PDF learning backend
    └── backend/
        └── src/             # TypeScript source
```

## 🚀 Live Deployment

- **Frontend**: http://aimentor-frontend.s3-website.ap-south-1.amazonaws.com
- **Backend API**: http://13.232.0.142/api/
- **Chatbot API**: http://13.232.0.142/chatbot/
- **PDF API**: http://13.232.0.142/pdf/

## 🔧 Local Development

### Prerequisites
- Node.js 18+ (20+ recommended)
- AWS Account with Bedrock access
- AWS CLI configured with credentials
- Judge0 instance (for code execution)
- DynamoDB table created
- S3 bucket for Knowledge Base (optional)

---

## 📚 AWS Setup Guide

### Step 1: Create DynamoDB Table

1. **Go to AWS DynamoDB Console**
2. **Create a new table** with these settings:
   - **Table name**: `hackveda-users` (or your preferred name)
   - **Partition key**: `email` (String)
   - **Settings**: Use default settings or on-demand capacity
3. **Note the table name** - you'll need it for `.env` files

### Step 2: Create Knowledge Bases

Knowledge bases provide domain-specific context to AI agents. Each agent can have its own knowledge base.

#### 2.1 Upload Knowledge Base Files to S3

1. **Create an S3 bucket** (e.g., `aimentor-knowledge-base`)
2. **Upload files from `knowlegde-base/` folder**:
   ```
   knowlegde-base/
   ├── canonical_skills.json          # Skill definitions
   ├── chatbot-assistant-rules.txt    # Chatbot behavior rules
   ├── coding-challenge-rules-compact.txt  # Coding challenge guidelines
   ├── difficulty-adjustment.txt      # Difficulty scaling rules
   ├── evaluation-policy.txt          # Assessment criteria
   ├── learning-assistant-rules.txt   # PDF learning rules
   ├── skill-mapping-rules.txt        # Skill mapping logic
   ├── teaching-rules.txt             # Teaching methodology
   └── test-case-generation-rules.txt # Test case creation rules
   ```

3. **Upload to S3**:
   ```bash
   aws s3 sync knowlegde-base/ s3://your-bucket-name/knowledge-base/
   ```

#### 2.2 Create Knowledge Base in AWS Bedrock

1. **Go to AWS Bedrock Console** → **Knowledge bases**
2. **Click "Create knowledge base"**
3. **Configure**:
   - **Name**: `AIMentor-KB` (or your preferred name)
   - **Data source**: Amazon S3
   - **S3 URI**: `s3://your-bucket-name/knowledge-base/`
   - **Embedding model**: Amazon Titan Embeddings G1 - Text
4. **Sync the knowledge base** (wait for completion)
5. **Note the Knowledge Base ID** - you'll need it for agents

### Step 3: Create AWS Bedrock Agents

You need to create 8 specialized agents. Instructions for each agent are in the `Instructions_for_agents/` folder.

#### Available Agent Instructions:
```
Instructions_for_agents/
├── AWS-AGENT-INSTRUCTION-UPDATED.txt      # General agent setup
├── AWS-COPY-PASTE-INSTRUCTIONS.txt        # Copy-paste ready instructions
├── CODING-AGENT-INSTRUCTION-FINAL.txt     # Coding challenge agent
├── CODING-AGENT-KB-INSTRUCTION.txt        # Coding agent with KB
├── COPY-PASTE-INSTRUCTIONS.txt            # Quick setup guide
└── TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt   # Teaching agent
```

#### 3.1 Create Each Agent

For each of the 8 agents, follow these steps:

1. **Go to AWS Bedrock Console** → **Agents**
2. **Click "Create Agent"**
3. **Configure Agent**:
   - **Agent name**: Choose from:
     - `Tutor-Agent` - Personalized teaching
     - `Curriculum-Agent` - Learning path generation
     - `Evaluation-Agent` - Assessment and feedback
     - `Skill-Mapping-Agent` - Skill analysis
     - `Coding-Agent` - Coding challenge generation
     - `Test-Case-Generator-Agent` - Test creation
     - `Chatbot-Agent` - Conversational AI
     - `Learning-Assistant-Agent` - PDF learning
   
   - **Model**: Claude 3.5 Sonnet (recommended) or Claude 3 Haiku
   - **Instructions**: Copy from corresponding file in `Instructions_for_agents/`

4. **Attach Knowledge Base** (if applicable):
   - Click "Add" under Knowledge bases
   - Select the knowledge base you created
   - Choose relevant files for this agent

5. **Create Alias**:
   - After creating agent, create an alias (e.g., `prod`, `v1`)
   - **Note both Agent ID and Alias ID**

6. **Test the Agent**:
   - Use the test console to verify responses
   - Ensure knowledge base is being used correctly

#### 3.2 Agent-to-Knowledge Base Mapping

| Agent | Recommended KB Files |
|-------|---------------------|
| **Tutor Agent** | `teaching-rules.txt`, `canonical_skills.json` |
| **Curriculum Agent** | `canonical_skills.json`, `skill-mapping-rules.txt` |
| **Evaluation Agent** | `evaluation-policy.txt`, `difficulty-adjustment.txt` |
| **Skill Mapping Agent** | `skill-mapping-rules.txt`, `canonical_skills.json` |
| **Coding Agent** | `coding-challenge-rules-compact.txt`, `test-case-generation-rules.txt` |
| **Test Case Generator** | `test-case-generation-rules.txt` |
| **Chatbot Agent** | `chatbot-assistant-rules.txt` |
| **Learning Assistant** | `learning-assistant-rules.txt` |

---

## 🔐 Environment Configuration

You need to configure **4 different `.env` files** in the project.

### 1. Backend Main Service (.env)

**Location**: `backend/.env`

```env
# Server Configuration
PORT=5012

# AWS Configuration
AWS_REGION=ap-south-1

# DynamoDB Configuration
DYNAMODB_USERS_TABLE=hackveda-users

# Tutor Agent
TUTOR_AGENT_ID=XXXXXXXXXX
TUTOR_AGENT_ALIAS=XXXXXXXXXX

# Curriculum Agent
CURRICULUM_AGENT_ID=XXXXXXXXXX
CURRICULUM_AGENT_ALIAS=XXXXXXXXXX

# Evaluation Agent
EVALUATION_AGENT_ID=XXXXXXXXXX
EVALUATION_AGENT_ALIAS=XXXXXXXXXX

# Skill Mapping Agent
SKILL_MAPPING_AGENT_ID=XXXXXXXXXX
SKILL_MAPPING_AGENT_ALIAS=XXXXXXXXXX

# Coding Agent
CODING_AGENT_ID=XXXXXXXXXX
CODING_AGENT_ALIAS=XXXXXXXXXX

# Test Case Generator Agent
TEST_CASE_GENERATOR_AGENT_ID=XXXXXXXXXX
TEST_CASE_GENERATOR_AGENT_ALIAS=XXXXXXXXXX

# Chatbot Agent
CHATBOT_AGENT_ID=XXXXXXXXXX
CHATBOT_AGENT_ALIAS=XXXXXXXXXX

# Learning Assistant Agent
LEARNING_ASSISTANT_AGENT_ID=XXXXXXXXXX
LEARNING_ASSISTANT_AGENT_ALIAS=XXXXXXXXXX

# JWT Secret (generate a random string)
JWT_SECRET=your-super-secret-jwt-key-change-this

# Judge0 Configuration
JUDGE0_URL=http://your-judge0-ip:2358
```

**Setup Steps**:
```bash
cd backend
cp .env.example .env
# Edit .env and add your Agent IDs and Aliases
nano .env
```

### 2. Chatbot Service (.env)

**Location**: `chatbotAWS/.env`

```env
# AWS Configuration
AWS_REGION=ap-south-1

# Bedrock Agent Configuration
BEDROCK_AGENT_ID=your-chatbot-agent-id
BEDROCK_AGENT_ALIAS=your-chatbot-agent-alias

# Server Configuration
PORT=5005
```

**Setup Steps**:
```bash
cd chatbotAWS
cp .env.example .env
# Edit .env and add your Chatbot Agent ID and Alias
nano .env
```

### 3. PDF Learning Service (.env)

**Location**: `HackathonPDF/backend/.env`

```env
# AWS Configuration
AWS_REGION=ap-south-1

# Bedrock Agent Configuration
BEDROCK_AGENT_ID=your-learning-assistant-agent-id
BEDROCK_AGENT_ALIAS=your-learning-assistant-agent-alias
BEDROCK_KNOWLEDGE_BASE_ID=your-knowledge-base-id

# S3 Configuration
S3_BUCKET_NAME=your-pdf-storage-bucket

# Server Configuration
PORT=5006
```

**Setup Steps**:
```bash
cd HackathonPDF/backend
cp .env.example .env
# Edit .env and add your Learning Assistant Agent details
nano .env
```

### 4. Frontend (.env)

**Location**: `frontend/.env`

```env
# Development URLs (localhost)
VITE_API_URL=http://localhost:5012/api
VITE_CHATBOT_URL=http://localhost:5005/chatbot
VITE_PDF_URL=http://localhost:5006/pdf
```

**For Production** (`.env.production`):
```env
# Production URLs (your deployed backend)
VITE_API_URL=http://your-ec2-ip/api
VITE_CHATBOT_URL=http://your-ec2-ip/chatbot
VITE_PDF_URL=http://your-ec2-ip/pdf
```

**Setup Steps**:
```bash
cd frontend
cp .env.example .env
# Edit .env for local development
nano .env

# For production build
nano .env.production
```

---

## 🚀 Quick Start After AWS Setup

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/rajrishu1401/AiMentor.git
cd AiMentor
```

2. **Configure AWS credentials**
```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: ap-south-1
# Default output format: json
```

3. **Set up all environment files** (see Environment Configuration section above)
```bash
# Backend
cd backend
cp .env.example .env
nano .env  # Add your Agent IDs

# Chatbot
cd ../chatbotAWS
cp .env.example .env
nano .env  # Add Chatbot Agent details

# PDF Service
cd ../HackathonPDF/backend
cp .env.example .env
nano .env  # Add Learning Assistant Agent details

# Frontend
cd ../../frontend
cp .env.example .env
nano .env  # Configure API URLs
```

4. **Install all dependencies**
```bash
# Backend
cd backend
npm install

# Chatbot
cd ../chatbotAWS
npm install

# PDF Service
cd ../HackathonPDF/backend
npm install

# Frontend
cd ../../frontend
npm install
```

5. **Start all services**

Open 4 separate terminal windows:

**Terminal 1 - Backend**:
```bash
cd backend
npm start
# Should start on http://localhost:5012
```

**Terminal 2 - Chatbot**:
```bash
cd chatbotAWS
node backend.js
# Should start on http://localhost:5005
```

**Terminal 3 - PDF Service**:
```bash
cd HackathonPDF/backend
npm run dev
# Should start on http://localhost:5006
```

**Terminal 4 - Frontend**:
```bash
cd frontend
npm run dev
# Should start on http://localhost:5173
```

6. **Access the application**
   - Open browser: `http://localhost:5173`
   - Create an account or login
   - Start learning!

---

## 🧪 Testing Your Setup

### Test Backend Services

```bash
# Test main backend
curl http://localhost:5012/api/health

# Test chatbot
curl http://localhost:5005/health

# Test PDF service
curl http://localhost:5006/health
```

### Test Agent Connections

1. **Login to the application**
2. **Try creating a roadmap** (tests Curriculum Agent)
3. **Open AI Mentor chat** (tests Chatbot Agent)
4. **Upload a PDF** (tests Learning Assistant Agent)
5. **Start a coding challenge** (tests Coding Agent)

---

## 📝 Adding New Knowledge Base Content

To add or update knowledge base content:

1. **Add/Edit files in `knowlegde-base/` folder**
   ```bash
   cd knowlegde-base
   # Add your new .txt or .json files
   nano my-new-rules.txt
   ```

2. **Upload to S3**
   ```bash
   aws s3 sync knowlegde-base/ s3://your-bucket-name/knowledge-base/
   ```

3. **Sync Knowledge Base in AWS Console**
   - Go to Bedrock → Knowledge bases
   - Select your knowledge base
   - Click "Sync" button
   - Wait for sync to complete

4. **Test the updated knowledge**
   - Use the agent test console
   - Verify new content is being used

---

## 🔄 Updating Agent Instructions

To update an agent's behavior:

1. **Edit instruction file**
   ```bash
   cd Instructions_for_agents
   nano TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt
   ```

2. **Copy updated instructions**
   - Open the instruction file
   - Copy the content between `---START COPY---` and `---END COPY---`

3. **Update in AWS Console**
   - Go to Bedrock → Agents
   - Select the agent
   - Click "Edit"
   - Paste new instructions in the "Instructions" field
   - Save changes

4. **Create new alias** (optional but recommended)
   - Create a new alias (e.g., `v2`)
   - Update `.env` file with new alias ID
   - Restart the service

---

## 📦 Production Deployment

### Backend Deployment (EC2)

1. **Launch EC2 instance** (t3.medium recommended)
2. **Install dependencies**
```bash
sudo yum update -y
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs git nginx
sudo npm install -g pm2
```

3. **Clone and setup**
```bash
git clone https://github.com/rajrishu1401/AiMentor.git
cd AiMentor
```

4. **Deploy backends with PM2**
```bash
cd backend
npm install
pm2 start server.js --name aiMentor-backend

cd ../chatbotAWS
npm install
pm2 start backend.js --name aiMentor-chatbot

cd ../HackathonPDF/backend
npm install
npm run build
pm2 start dist/server.js --name aiMentor-pdf

pm2 save
pm2 startup
```

5. **Configure Nginx**
```bash
sudo nano /etc/nginx/conf.d/aimentor.conf
```

Add reverse proxy configuration for ports 5012, 5005, 5006.

6. **Start Nginx**
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Frontend Deployment (S3)

1. **Build frontend**
```bash
cd frontend
npm run build
```

2. **Create S3 bucket**
- Enable static website hosting
- Make bucket public
- Upload `dist/` contents

3. **Configure bucket policy**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-bucket/*"
  }]
}
```

## 🔐 Security

- JWT-based authentication
- Password hashing with bcrypt
- IAM roles for AWS service access
- CORS configuration for cross-origin requests
- Environment variable management
- Secure credential handling

## 🎯 Key Integrations

### AWS Bedrock Agents
- 8 specialized AI agents for different learning tasks
- Custom knowledge bases for domain-specific content
- Real-time streaming responses

### Judge0 Code Execution
- Secure sandboxed code execution
- Support for 50+ programming languages
- Base64 encoding for special characters
- Timeout and resource limits

### DynamoDB
- User authentication and profiles
- Learning progress tracking
- Roadmap storage

## 📊 Performance

- **Backend**: 3 Node.js services managed by PM2
- **Frontend**: Static hosting on S3 with CDN-ready setup
- **Database**: DynamoDB with on-demand capacity
- **Caching**: Nginx reverse proxy
- **Auto-restart**: PM2 process management

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## ❓ Troubleshooting

### Common Issues

**1. "Backend server is not available"**
- Check if all 3 backend services are running
- Verify `.env` files are configured correctly
- Check AWS credentials: `aws sts get-caller-identity`

**2. "Agent not responding"**
- Verify Agent ID and Alias in `.env` files
- Check agent status in AWS Bedrock Console
- Ensure IAM permissions for Bedrock are set

**3. "Knowledge base not working"**
- Verify Knowledge Base ID in `.env`
- Check if KB is synced in AWS Console
- Ensure S3 bucket permissions are correct

**4. "DynamoDB errors"**
- Verify table name in `.env` matches AWS
- Check IAM permissions for DynamoDB
- Ensure table exists in correct region

**5. "CORS errors"**
- Check frontend `.env` URLs match backend
- Verify backend CORS configuration
- Clear browser cache

### Getting Help

- Check the `Kiro_md_Files/` folder for detailed documentation
- Review agent instructions in `Instructions_for_agents/`
- Check AWS CloudWatch logs for backend errors

---

## 📂 Project Structure Details

```
AiMentor/
├── frontend/                          # React frontend application
│   ├── src/
│   │   ├── api/                      # API client functions
│   │   │   ├── api.js               # Main API requests
│   │   │   └── auth.js              # Authentication API
│   │   ├── components/              # Reusable UI components
│   │   │   └── pdf/                 # PDF learning components
│   │   ├── pages/                   # Page components
│   │   │   ├── Login.jsx           # Login page
│   │   │   ├── Signup.jsx          # Signup page
│   │   │   ├── Sandbox.jsx         # Code sandbox
│   │   │   └── AIMentorDrawer.jsx  # AI chat drawer
│   │   ├── hooks/                   # Custom React hooks
│   │   │   └── usePDFLearning.js   # PDF learning hook
│   │   └── config/                  # Configuration files
│   ├── .env                         # Local development config
│   ├── .env.production              # Production config
│   └── .env.example                 # Template
│
├── backend/                          # Main Express backend
│   ├── src/
│   │   ├── agents/                  # AI agent integrations
│   │   │   ├── tutorAgent.js       # Teaching agent
│   │   │   ├── curriculumAgent.js  # Roadmap generation
│   │   │   ├── evaluationAgent.js  # Assessment
│   │   │   ├── codingAgent.js      # Coding challenges
│   │   │   └── ...                 # Other agents
│   │   ├── controllers/             # Route controllers
│   │   ├── models/                  # Data models
│   │   │   ├── User.js             # User model
│   │   │   └── UserDynamoDB.js     # DynamoDB operations
│   │   ├── routes/                  # API routes
│   │   ├── services/                # Business logic
│   │   ├── middleware/              # Express middleware
│   │   └── config/                  # Backend configuration
│   ├── .env                         # Backend environment config
│   └── .env.example                 # Template
│
├── chatbotAWS/                      # Chatbot backend service
│   ├── backend.js                   # Standalone chatbot server
│   ├── .env                         # Chatbot config
│   └── .env.example                 # Template
│
├── HackathonPDF/                    # PDF learning backend
│   └── backend/
│       ├── src/                     # TypeScript source
│       ├── .env                     # PDF service config
│       └── .env.example             # Template
│
├── knowlegde-base/                  # Knowledge base files for agents
│   ├── canonical_skills.json       # Skill definitions
│   ├── chatbot-assistant-rules.txt # Chatbot behavior
│   ├── coding-challenge-rules-compact.txt
│   ├── difficulty-adjustment.txt
│   ├── evaluation-policy.txt
│   ├── learning-assistant-rules.txt
│   ├── skill-mapping-rules.txt
│   ├── teaching-rules.txt
│   └── test-case-generation-rules.txt
│
├── Instructions_for_agents/         # Agent setup instructions
│   ├── AWS-AGENT-INSTRUCTION-UPDATED.txt
│   ├── AWS-COPY-PASTE-INSTRUCTIONS.txt
│   ├── CODING-AGENT-INSTRUCTION-FINAL.txt
│   ├── CODING-AGENT-KB-INSTRUCTION.txt
│   ├── COPY-PASTE-INSTRUCTIONS.txt
│   └── TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt
│
├── Kiro_md_Files/                   # Documentation and guides
│   ├── backend/
│   ├── frontend/
│   ├── Deployment/
│   └── *.md                         # Various documentation files
│
└── README.md                        # This file
```

---

## 📝 License

This project is licensed under the MIT License.

## 👥 Authors

- **Raj Rishu** - [GitHub](https://github.com/rajrishu1401)

## 🙏 Acknowledgments

- AWS Bedrock for AI capabilities
- Judge0 for code execution
- React community for excellent tools
- All open-source contributors

---

## 📋 Quick Reference

### Environment Files Checklist

- [ ] `backend/.env` - 8 agents + DynamoDB + Judge0
- [ ] `chatbotAWS/.env` - Chatbot agent
- [ ] `HackathonPDF/backend/.env` - Learning assistant + KB
- [ ] `frontend/.env` - API URLs (development)
- [ ] `frontend/.env.production` - API URLs (production)

### AWS Resources Checklist

- [ ] DynamoDB table created (`hackveda-users`)
- [ ] S3 bucket for knowledge base
- [ ] Knowledge base created and synced
- [ ] 8 Bedrock agents created with aliases
- [ ] IAM permissions configured
- [ ] AWS CLI configured locally

### Agent IDs Required

| Service | Agent | Required in .env |
|---------|-------|-----------------|
| Backend | Tutor | `TUTOR_AGENT_ID` + `TUTOR_AGENT_ALIAS` |
| Backend | Curriculum | `CURRICULUM_AGENT_ID` + `CURRICULUM_AGENT_ALIAS` |
| Backend | Evaluation | `EVALUATION_AGENT_ID` + `EVALUATION_AGENT_ALIAS` |
| Backend | Skill Mapping | `SKILL_MAPPING_AGENT_ID` + `SKILL_MAPPING_AGENT_ALIAS` |
| Backend | Coding | `CODING_AGENT_ID` + `CODING_AGENT_ALIAS` |
| Backend | Test Generator | `TEST_CASE_GENERATOR_AGENT_ID` + `TEST_CASE_GENERATOR_AGENT_ALIAS` |
| Backend | Chatbot | `CHATBOT_AGENT_ID` + `CHATBOT_AGENT_ALIAS` |
| Backend | Learning Assistant | `LEARNING_ASSISTANT_AGENT_ID` + `LEARNING_ASSISTANT_AGENT_ALIAS` |
| ChatbotAWS | Chatbot | `BEDROCK_AGENT_ID` + `BEDROCK_AGENT_ALIAS` |
| HackathonPDF | Learning Assistant | `BEDROCK_AGENT_ID` + `BEDROCK_AGENT_ALIAS` + `BEDROCK_KNOWLEDGE_BASE_ID` |

### Port Reference

| Service | Port | URL |
|---------|------|-----|
| Frontend Dev | 5173 | http://localhost:5173 |
| Backend Main | 5012 | http://localhost:5012 |
| Chatbot | 5005 | http://localhost:5005 |
| PDF Service | 5006 | http://localhost:5006 |
| Judge0 | 2358 | http://your-judge0-ip:2358 |

### Useful Commands

```bash
# Check AWS credentials
aws sts get-caller-identity

# List DynamoDB tables
aws dynamodb list-tables --region ap-south-1

# List S3 buckets
aws s3 ls

# Sync knowledge base
aws s3 sync knowlegde-base/ s3://your-bucket/knowledge-base/

# Check if services are running
curl http://localhost:5012/api/health
curl http://localhost:5005/health
curl http://localhost:5006/health

# View backend logs (if using PM2)
pm2 logs aiMentor-backend
pm2 logs aiMentor-chatbot
pm2 logs aiMentor-pdf
```

---

**Built with ❤️ using AI and modern web technologies**
