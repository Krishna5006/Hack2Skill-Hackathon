# Chatbot Setup Guide - Quick Reference

## Current Status
✅ Agent alias fixed in `.env` file
✅ Knowledge base file created
✅ Backend code working correctly
⏳ Knowledge base needs to be uploaded to AWS

## What's Been Fixed

### 1. Agent Alias Correction
**File**: `Hackveda/backend/.env`

**Before**:
```env
CHATBOT_AGENT_ALIAS=GJ7YMCGPKE  # Wrong - this was Coding Agent's alias
```

**After**:
```env
CHATBOT_AGENT_ALIAS=3KNCV8OBXZ  # Correct - Chatbot Agent's alias
```

### 2. Knowledge Base Created
**File**: `Hackveda/knowlegde-base/chatbot-assistant-rules.txt`

This file contains comprehensive instructions for the chatbot:
- Never provide complete code solutions
- Give strategic hints and guiding questions
- Analyze code complexity when requested
- Help with debugging without giving away fixes
- Explain concepts clearly
- Be encouraging and supportive

## Steps to Complete Setup

### Step 1: Upload Knowledge Base to S3

```bash
# Navigate to the knowledge base directory
cd Hackveda/knowlegde-base

# Upload to your S3 bucket (replace with your bucket name)
# Use --force to overwrite if it already exists
aws s3 cp chatbot-assistant-rules.txt s3://YOUR-BUCKET-NAME/chatbot-assistant-rules.txt --force

# Verify upload
aws s3 ls s3://YOUR-BUCKET-NAME/
```

**Important**: The knowledge base has been completely rewritten with STRICT no-code policy. The chatbot will NEVER provide code solutions now.

**Note**: Use the same S3 bucket you used for the coding agent knowledge base.

### Step 2: Attach Knowledge Base to Agent in AWS Console

1. **Open AWS Bedrock Console**
   - Go to: https://console.aws.amazon.com/bedrock/
   - Region: ap-south-1 (Mumbai)

2. **Navigate to Agents**
   - Click "Agents" in the left sidebar
   - Find agent with ID: `OKPUEODEZC`
   - Click on the agent name

3. **Add Knowledge Base**
   - Scroll to "Knowledge bases" section
   - Click "Add knowledge base"
   - Select "Create new knowledge base" or use existing one
   - Choose the S3 bucket and file path
   - Name it: `chatbot-assistant-kb`

4. **Set Knowledge Base Instruction**
   In the instruction field, enter:
   ```
   Follow the rules and guidelines in this knowledge base to assist students with coding problems. Never provide complete solutions. Guide students to discover answers themselves through hints and questions.
   ```

5. **Sync Knowledge Base**
   - Click "Sync" button
   - Wait for sync to complete (usually 1-2 minutes)

6. **Prepare Agent**
   - Go to agent aliases
   - Select alias: `3KNCV8OBXZ`
   - Click "Prepare"
   - Wait for preparation to complete

### Step 3: Test the Chatbot

1. **Test in AWS Console First**
   - In the agent page, click "Test"
   - Try these test queries:
     - "Give me a hint for this problem"
     - "What's the solution?" (should refuse)
     - "Explain dynamic programming"
     - "What's the time complexity of my code?"

2. **Test in Your Application**
   - Start your backend server
   - Navigate to a coding challenge
   - Open the chatbot panel
   - Try the same test queries

## Expected Behavior

### ✅ Good Responses (After Setup)

**User**: "Give me a hint"
```
Bot: "Great question! For this problem, think about:
1. What data structure would let you look up values quickly?
2. How can you track what you've already seen?
3. Consider using a hash map to store frequencies.

Try implementing that approach and let me know how it goes!"
```

**User**: "Give me the solution"
```
Bot: "I can't give you the complete solution, but I can help you work through it! Let's start with understanding the problem. What approach are you thinking of trying?"
```

**User**: "What's the time complexity of my code?"
```
Bot: "Let me analyze your code:

Your nested loops create O(n²) time complexity because:
- The outer loop runs n times
- For each iteration, the inner loop also runs n times
- Total operations: n × n = n²

Space complexity is O(1) since you're only using a few variables.

Can you think of a way to reduce this to O(n) by avoiding the nested loop?"
```

### ❌ Bad Responses (Before Setup)

**User**: "Give me a hint"
```
Bot: "I can help you with:
- Give me a hint
- Show me a test case
- What's the time complexity?"
```

This is the fallback response, indicating the agent isn't working properly.

## Troubleshooting

### Error: "Failed to retrieve resource"
**Cause**: Wrong agent alias or agent not prepared
**Fix**: 
1. Verify `CHATBOT_AGENT_ALIAS=3KNCV8OBXZ` in `.env`
2. Restart backend server
3. Ensure agent is prepared in AWS console

### Error: "Chatbot failed"
**Cause**: AWS credentials or network issue
**Fix**:
1. Check AWS credentials are configured
2. Verify region is `ap-south-1`
3. Check network connectivity to AWS

### Chatbot gives generic responses
**Cause**: Knowledge base not attached or not synced
**Fix**:
1. Verify knowledge base is attached to agent
2. Sync the knowledge base
3. Prepare the agent again

### Chatbot gives code solutions
**Cause**: Knowledge base instruction not set correctly
**Fix**:
1. Update knowledge base instruction
2. Emphasize "Never provide complete solutions"
3. Sync and prepare agent again

## Verification Checklist

Before considering setup complete, verify:

- [ ] `.env` has correct `CHATBOT_AGENT_ALIAS=3KNCV8OBXZ`
- [ ] Knowledge base file uploaded to S3
- [ ] Knowledge base attached to agent in AWS
- [ ] Knowledge base instruction set correctly
- [ ] Knowledge base synced successfully
- [ ] Agent prepared with latest version
- [ ] Agent alias `3KNCV8OBXZ` points to prepared version
- [ ] Test in AWS console shows intelligent responses
- [ ] Test in application shows intelligent responses
- [ ] Chatbot refuses to give complete solutions
- [ ] Chatbot provides helpful hints
- [ ] Chatbot analyzes code complexity when asked

## Quick Test Commands

### Test Backend Endpoint
```bash
curl -X POST http://localhost:5012/api/learning/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "message": "Give me a hint",
    "sessionId": "test-session",
    "problemContext": {
      "title": "Two Sum",
      "description": "Find two numbers that add up to target",
      "hint": "Use a hash map"
    }
  }'
```

### Check Backend Logs
```bash
# In backend directory
npm start

# Watch for these logs:
# ✅ Chatbot response: <intelligent response>
# ❌ Chatbot error: <error message>
```

## Files Reference

### Modified Files
1. `Hackveda/backend/.env` - Fixed agent alias
2. `Hackveda/backend/src/agents/chatbotAgent.js` - Already working
3. `Hackveda/backend/src/controllers/learningController.js` - Already working
4. `Hackveda/frontend/src/pages/CodingPage.jsx` - Already working

### New Files
1. `Hackveda/knowlegde-base/chatbot-assistant-rules.txt` - Knowledge base
2. `Hackveda/backend/CHATBOT-INTELLIGENCE-FIX.md` - Detailed documentation
3. `Hackveda/backend/CHATBOT-SETUP-GUIDE.md` - This file

## Support

If you encounter issues:
1. Check backend console logs for errors
2. Verify AWS Bedrock agent status in console
3. Test agent directly in AWS console first
4. Ensure knowledge base is synced and agent is prepared
5. Restart backend server after any `.env` changes

## Summary

The chatbot code is ready and working. You just need to:
1. Upload the knowledge base file to S3
2. Attach it to the agent in AWS console
3. Sync and prepare the agent

Once these steps are complete, the chatbot will provide intelligent, context-aware assistance without giving away solutions!
