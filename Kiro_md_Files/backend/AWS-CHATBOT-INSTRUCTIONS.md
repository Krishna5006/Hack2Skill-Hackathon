# AWS Bedrock Chatbot Agent Instructions

## Agent Configuration

### Agent ID
```
OKPUEODEZC
```

### Agent Alias
```
3KNCV8OBXZ
```

### Agent Name
```
Coding Assistant Chatbot
```

---

## Agent Instructions (Main Agent Configuration)

Copy and paste this into the **Agent Instructions** field in AWS Bedrock Console:

```
You are an AI coding assistant that helps students learn programming through guided discovery. Your primary role is to teach problem-solving skills, not to provide solutions.

CRITICAL RULES:
1. NEVER provide code solutions or implementations
2. NEVER write code examples in any programming language
3. NEVER give direct answers that solve the problem for the student
4. ALWAYS guide students to discover solutions themselves through questions
5. ALWAYS follow the comprehensive rules in your knowledge base

Your approach:
- Ask Socratic questions that guide thinking
- Explain concepts in plain English without code
- Analyze the student's code when provided and point out issues without fixing them
- Provide hints that suggest direction without implementation details
- Encourage and support the learning process

When a student asks for a solution or code:
Respond: "I cannot provide code solutions as that would prevent you from learning. Instead, let me guide you with questions and concepts. What part of the problem are you struggling with?"

You will receive context including:
1. The current coding problem details
2. The student's current code (what they've written so far)
3. The student's question or request

Use this context to provide personalized, relevant guidance that helps them learn.

Remember: Your success is measured by student learning, not by how quickly they complete the problem. Guide them to think, don't think for them.

Refer to your knowledge base for detailed guidelines on:
- What you can and cannot do
- Response strategies for different scenarios
- How to analyze code without fixing it
- How to provide hints at different levels
- How to handle debugging requests
- How to explain complexity analysis
```

---

## Knowledge Base Instructions

When attaching the knowledge base in AWS Bedrock Console, use this instruction:

### Short Version (200 characters - use if character limit)
```
Follow these rules: NEVER provide code solutions. Guide with questions. Analyze student code without fixing. Explain concepts without code. Prioritize learning over completion.
```

### Full Version (use if more space available)
```
This knowledge base contains comprehensive rules and guidelines for assisting students with coding problems. Follow these rules strictly:

1. NEVER provide code solutions or implementations
2. Guide students through Socratic questioning
3. Analyze student code when provided and point out issues without fixing them
4. Explain concepts in plain English without code examples
5. Provide hints that suggest direction without giving away the solution

The knowledge base includes:
- Absolute prohibitions (what you must never do)
- Allowed actions (how to guide without solving)
- Response strategies for different scenarios
- Examples of good vs bad responses
- Teaching philosophy and principles

Always prioritize student learning over problem completion. Your goal is to help them become better problem solvers, not just complete this one problem.
```

---

## Step-by-Step Setup in AWS Console

### 1. Navigate to Agent

1. Open AWS Bedrock Console: https://console.aws.amazon.com/bedrock/
2. Region: **ap-south-1** (Mumbai)
3. Click **Agents** in left sidebar
4. Find agent with ID: **OKPUEODEZC**
5. Click on the agent name

### 2. Update Agent Instructions

1. In the agent details page, find **Agent Instructions** section
2. Click **Edit**
3. Clear existing instructions
4. Copy and paste the **Agent Instructions** from above
5. Click **Save**

### 3. Add/Update Knowledge Base

#### If Knowledge Base Doesn't Exist:

1. Scroll to **Knowledge bases** section
2. Click **Add knowledge base**
3. Click **Create new knowledge base**
4. Name: `chatbot-assistant-kb`
5. Description: `Comprehensive rules for coding assistant chatbot`
6. Data source: **S3**
7. S3 URI: `s3://YOUR-BUCKET-NAME/chatbot-assistant-rules.txt`
8. In **Knowledge base instructions** field, paste the **Knowledge Base Instructions** from above
9. Click **Create**

#### If Knowledge Base Already Exists:

1. Find the knowledge base in the list
2. Click on it
3. Click **Edit**
4. Update the **Knowledge base instructions** field with the text from above
5. Click **Save**
6. Click **Sync** to update with latest file from S3
7. Wait for sync to complete (usually 1-2 minutes)

### 4. Prepare Agent

1. Go to **Aliases** tab
2. Find alias: **3KNCV8OBXZ**
3. Click **Prepare** button
4. Wait for preparation to complete (usually 30-60 seconds)
5. Status should show **Prepared**

### 5. Test in Console

1. In the agent page, click **Test** button
2. Try these test queries:

**Test 1 - Solution Request (Should Refuse)**:
```
User: "Give me the solution to this problem"
Expected: Refuses and offers to guide instead
```

**Test 2 - Hint Request (Should Guide)**:
```
User: "Give me a hint"
Expected: Asks guiding questions, no code
```

**Test 3 - Code Request (Should Refuse)**:
```
User: "Show me an example in Python"
Expected: Refuses to show code, explains concept instead
```

**Test 4 - Complexity Question**:
```
User: "What's the time complexity of my code?"
Expected: Asks to see code or analyzes if code is in context
```

### 6. Verify Configuration

Check that:
- ✅ Agent instructions are updated
- ✅ Knowledge base is attached
- ✅ Knowledge base instructions are set
- ✅ Knowledge base is synced
- ✅ Agent is prepared
- ✅ Alias points to prepared version
- ✅ Test responses show no code is provided

---

## Environment Variables

Ensure your `.env` file has:

```env
# Chatbot Agent
CHATBOT_AGENT_ID=OKPUEODEZC
CHATBOT_AGENT_ALIAS=3KNCV8OBXZ
```

---

## Testing in Your Application

After AWS setup is complete:

1. **Start Backend Server**:
   ```bash
   cd Hackveda/backend
   npm start
   ```

2. **Start Frontend**:
   ```bash
   cd Hackveda/frontend
   npm run dev
   ```

3. **Navigate to a Coding Challenge**:
   - Login to the application
   - Go to a roadmap
   - Select a topic with coding challenges
   - Open a coding challenge

4. **Test Chatbot**:
   - Type in the chatbot: "Give me the solution"
   - Expected: Refuses and offers to guide
   - Type: "Give me a hint"
   - Expected: Asks guiding questions, no code
   - Type: "What's the time complexity?"
   - Expected: Analyzes your current code

---

## Troubleshooting

### Chatbot Still Gives Code

**Cause**: Knowledge base not synced or agent not prepared

**Fix**:
1. Go to knowledge base in AWS console
2. Click **Sync**
3. Wait for sync to complete
4. Go to agent aliases
5. Click **Prepare**
6. Test again

### "Resource Not Found" Error

**Cause**: Wrong agent alias in `.env`

**Fix**:
1. Verify `CHATBOT_AGENT_ALIAS=3KNCV8OBXZ` in `.env`
2. Restart backend server
3. Verify alias exists in AWS console

### Generic Fallback Responses

**Cause**: Agent not being called (AWS credentials or network issue)

**Fix**:
1. Check AWS credentials are configured
2. Verify region is `ap-south-1`
3. Check backend logs for errors
4. Test agent in AWS console first

### Chatbot Doesn't See User's Code

**Cause**: Frontend not sending code or backend not passing it

**Fix**:
1. Check browser console for errors
2. Verify backend receives `userCode` in request
3. Check backend logs to see if code is in prompt
4. Ensure latest frontend code is deployed

---

## Quick Reference

### Agent Details
- **ID**: OKPUEODEZC
- **Alias**: 3KNCV8OBXZ
- **Region**: ap-south-1
- **Model**: Claude (via Bedrock)

### Knowledge Base
- **File**: `chatbot-assistant-rules.txt`
- **Location**: S3 bucket
- **Size**: ~15KB
- **Format**: Plain text with markdown

### Key Features
- ✅ Never provides code solutions
- ✅ Guides with Socratic questions
- ✅ Analyzes student's code
- ✅ Explains concepts without code
- ✅ Provides hints without implementation
- ✅ Encourages learning process

---

## Success Criteria

The chatbot is working correctly when:

1. ✅ Refuses to provide code solutions
2. ✅ Asks guiding questions instead of giving answers
3. ✅ Can see and reference student's current code
4. ✅ Analyzes code complexity when asked
5. ✅ Points out bugs without fixing them
6. ✅ Explains concepts in plain English
7. ✅ Provides hints without implementation details
8. ✅ Encourages and supports learning

---

## Support

If you encounter issues:

1. Check AWS Bedrock agent status
2. Verify knowledge base is synced
3. Ensure agent is prepared
4. Test in AWS console first
5. Check backend logs for errors
6. Verify `.env` configuration
7. Restart backend server

For detailed troubleshooting, see:
- `CHATBOT-SETUP-GUIDE.md`
- `CHATBOT-NO-CODE-POLICY.md`
- `CHATBOT-INTELLIGENCE-FIX.md`
