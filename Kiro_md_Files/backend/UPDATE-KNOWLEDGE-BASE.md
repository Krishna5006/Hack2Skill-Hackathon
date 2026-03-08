# How to Update AWS Bedrock Agent Knowledge Base

## Overview
When you update knowledge base files (like `coding-challenge-rules.txt`), you need to upload them to S3 and sync with the Bedrock Agent.

## Steps to Update

### Option 1: AWS Console (Easiest)

1. **Go to AWS S3 Console:**
   - Navigate to: https://console.aws.amazon.com/s3/
   - Region: ap-south-1 (Mumbai)

2. **Find your Knowledge Base bucket:**
   - Look for a bucket with a name like: `bedrock-agent-kb-*` or similar
   - Or check your Bedrock Agent configuration for the S3 bucket name

3. **Upload the updated file:**
   - Click on the bucket
   - Navigate to the folder containing `coding-challenge-rules.txt`
   - Click "Upload"
   - Select the updated `Hackveda/knowlegde-base/coding-challenge-rules.txt`
   - Click "Upload"

4. **Sync the Knowledge Base:**
   - Go to AWS Bedrock Console: https://console.aws.amazon.com/bedrock/
   - Click "Agents" in the left sidebar
   - Find your Coding Agent (ID: BDZFCSFZJF)
   - Click on the agent
   - Go to "Knowledge bases" tab
   - Click "Sync" button
   - Wait for sync to complete (usually 1-2 minutes)

### Option 2: AWS CLI

```bash
# 1. Upload to S3 (replace BUCKET_NAME with your actual bucket)
aws s3 cp Hackveda/knowlegde-base/coding-challenge-rules.txt \
  s3://YOUR_BUCKET_NAME/coding-challenge-rules.txt \
  --region ap-south-1

# 2. Sync Knowledge Base (replace KB_ID with your knowledge base ID)
aws bedrock-agent start-ingestion-job \
  --knowledge-base-id YOUR_KB_ID \
  --data-source-id YOUR_DATA_SOURCE_ID \
  --region ap-south-1
```

### Option 3: Using AWS SDK (Node.js)

```javascript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { BedrockAgentClient, StartIngestionJobCommand } from "@aws-sdk/client-bedrock-agent";
import fs from "fs";

// Upload to S3
const s3Client = new S3Client({ region: "ap-south-1" });
const fileContent = fs.readFileSync("Hackveda/knowlegde-base/coding-challenge-rules.txt");

await s3Client.send(new PutObjectCommand({
  Bucket: "YOUR_BUCKET_NAME",
  Key: "coding-challenge-rules.txt",
  Body: fileContent
}));

// Sync Knowledge Base
const bedrockClient = new BedrockAgentClient({ region: "ap-south-1" });
await bedrockClient.send(new StartIngestionJobCommand({
  knowledgeBaseId: "YOUR_KB_ID",
  dataSourceId: "YOUR_DATA_SOURCE_ID"
}));
```

## Finding Your Knowledge Base Details

### Method 1: AWS Console
1. Go to Bedrock Console
2. Click "Agents" → Find "Coding Agent"
3. Click "Knowledge bases" tab
4. Note the Knowledge Base ID and Data Source ID
5. Click on the Knowledge Base name
6. Check the "Data source" section for S3 bucket details

### Method 2: AWS CLI
```bash
# List agents
aws bedrock-agent list-agents --region ap-south-1

# Get agent details
aws bedrock-agent get-agent \
  --agent-id BDZFCSFZJF \
  --region ap-south-1

# List knowledge bases for agent
aws bedrock-agent list-agent-knowledge-bases \
  --agent-id BDZFCSFZJF \
  --agent-version DRAFT \
  --region ap-south-1
```

## Verification

After updating and syncing:

1. **Test the agent:**
   - Generate a new coding challenge
   - Check if quotes are handled correctly
   - Verify JSON parsing succeeds

2. **Check logs:**
   - Look for "First parse failed" messages (should be rare)
   - Verify problem statements don't have unescaped quotes

3. **Monitor for a few challenges:**
   - Generate 3-5 challenges
   - Ensure all parse successfully
   - Check problem statement formatting

## Troubleshooting

### Sync fails:
- Check S3 bucket permissions
- Verify file was uploaded correctly
- Check Bedrock Agent has access to S3 bucket

### Agent still generates bad JSON:
- Wait 5-10 minutes after sync (caching)
- Try creating a new agent session
- Check if file was uploaded to correct location

### Can't find Knowledge Base:
- Check Bedrock Console → Agents → Coding Agent
- Look for "Knowledge bases" tab
- If no knowledge base attached, you may need to create one

## Important Notes

1. **Sync is required:** Just uploading to S3 is not enough - you must sync the knowledge base
2. **Caching:** Changes may take a few minutes to propagate
3. **Version:** Make sure you're syncing the correct agent version (DRAFT vs LIVE)
4. **Backup:** Keep a backup of the old file before updating

## Current Agent Details

- **Agent ID:** BDZFCSFZJF
- **Agent Alias:** MNAD2YJYBP
- **Region:** ap-south-1
- **Knowledge Base File:** `coding-challenge-rules.txt`

## Files to Update

When you modify these files, follow the update process:
- `Hackveda/knowlegde-base/coding-challenge-rules.txt` → Coding Agent KB
- `Hackveda/knowlegde-base/chatbot-assistant-rules.txt` → Chatbot Agent KB
- `Hackveda/knowlegde-base/skill-mapping-rules.txt` → Skill Mapping Agent KB
