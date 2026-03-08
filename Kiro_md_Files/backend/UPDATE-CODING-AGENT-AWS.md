# Update Coding Agent in AWS Bedrock

## Problem
The coding agent is still generating incomplete responses and not following the new 3-test-case rule because the AWS Bedrock agent hasn't been updated with the new instructions.

## Solution: Update the Coding Agent

### Step 1: Go to AWS Bedrock Console
1. Log in to AWS Console
2. Navigate to Amazon Bedrock → Agents
3. Find agent with ID: `BDZFCSFZJF`

### Step 2: Update Agent Instructions
Click "Edit" on the agent and update the instructions to match `CODING-AGENT-INSTRUCTION.txt`:

**Key changes to emphasize:**
```
IMPORTANT: Include EXACTLY 3 visible test cases with explanations. 
Hidden test cases will be generated separately by another agent.
```

### Step 3: Update Knowledge Base
If you're using a knowledge base:
1. Go to the knowledge base attached to the coding agent
2. Update or replace the `coding-challenge-rules.txt` file
3. Sync the knowledge base

### Step 4: Create New Version
1. After making changes, create a new version of the agent
2. Update the alias to point to the new version
3. Or create a new alias

### Step 5: Test
Restart your backend and test again. You should see:
```
✅ Coding Agent Response: {...} (with 3 test cases)
✅ Test Case Generator Response: 7 hidden test cases
📊 Total test cases: 10 (3 visible + 7 hidden)
```

## Alternative: Check Agent Alias

Your current setup:
- Agent ID: `BDZFCSFZJF`
- Alias ID: `SCZHLNY6G7`

Make sure the alias points to the latest version of the agent with updated instructions.

## Verification Checklist

After updating:
- [ ] Agent instructions mention "EXACTLY 3 visible test cases"
- [ ] Agent instructions mention "Hidden test cases will be generated separately"
- [ ] Knowledge base has updated `coding-challenge-rules.txt`
- [ ] New version created
- [ ] Alias updated to new version
- [ ] Backend restarted
- [ ] Test challenge generation works

## Expected Behavior

**Before Fix:**
- Coding agent generates incomplete JSON
- No test cases or placeholder test cases
- Test case generator never called
- Error: "Failed to prepare coding challenge"

**After Fix:**
- Coding agent generates complete problem + 3 visible test cases
- Test case generator generates 7-10 hidden test cases
- Total 10-13 test cases merged
- Challenge saved successfully
