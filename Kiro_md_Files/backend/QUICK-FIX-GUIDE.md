# Quick Fix Guide - Increase Coding Agent Token Limit

## Problem
Coding Agent response is truncated, causing JSON parsing errors.

## Solution
Increase output token limit to 4000 in AWS Bedrock Console.

## Step-by-Step Instructions

### 1. Access AWS Console
- Go to: https://console.aws.amazon.com/
- Sign in with your credentials
- Region: **ap-south-1** (Mumbai)

### 2. Navigate to Bedrock Agents
- Search for "Bedrock" in the top search bar
- Click "Amazon Bedrock"
- In left sidebar, click "Agents"

### 3. Find Coding Agent
- Look for agent with ID: **BDZFCSFZJF**
- Or search for name: "Coding Agent" or similar
- Click on the agent name

### 4. Edit Agent Configuration
- Click "Edit" button (top right)
- Scroll to find one of these sections:
  - "Inference configuration"
  - "Model configuration"
  - "Advanced settings"

### 5. Update Token Limit
Look for one of these settings:
- "Max tokens"
- "Maximum length"
- "Max tokens to sample"
- "Output token limit"

**Change the value to: 4000**

### 6. Save Changes
- Click "Save" or "Update"
- AWS will create a new version of the agent

### 7. Update Alias (IMPORTANT!)
- Go to "Aliases" tab
- Find the alias: **JSTC6CIBFU** (or the one you're using)
- Click "Edit"
- Update to point to the new version you just created
- Save

### 8. Verify Changes
Wait 1-2 minutes for changes to propagate, then run:

```bash
cd Hackveda/backend
node test-two-agent-flow.js
```

Expected output:
```
✅ Coding Agent Response: {...}
✅ Test Case Generator Response: 7 hidden test cases
📊 Total test cases: 10 (3 visible + 7 hidden)
✅ TWO-AGENT FLOW TEST PASSED!
```

## Visual Guide

```
AWS Console
    ↓
Amazon Bedrock
    ↓
Agents (left sidebar)
    ↓
Find: BDZFCSFZJF
    ↓
Click: Edit
    ↓
Find: Inference Configuration
    ↓
Update: Max tokens = 4000
    ↓
Save
    ↓
Aliases tab
    ↓
Edit alias: JSTC6CIBFU
    ↓
Point to new version
    ↓
Save
    ↓
Done!
```

## Troubleshooting

### Can't Find "Max Tokens" Setting
- Try "Advanced settings" or "Model parameters"
- Look for "Temperature", "Top P" - token limit is usually nearby
- Different models may have different UI layouts

### Changes Not Taking Effect
- Verify alias points to new version (not old version)
- Wait 2-3 minutes for propagation
- Restart your backend server
- Clear any caches

### Still Getting Truncated Responses
- Verify the change was saved (check agent version)
- Try increasing to 5000 tokens
- Check CloudWatch logs for actual token usage
- Consider using a different model

## Alternative: Quick Test Without Full Fix

If you want to test the system works before fixing token limit, you can temporarily modify the Coding Agent instructions to generate shorter content:

1. Edit agent instructions in AWS Console
2. Add at top: "KEEP ALL RESPONSES UNDER 1000 TOKENS"
3. Add: "Use brief explanations, short examples"
4. Save and test

This will reduce quality but prove the system works.

## After Fix is Complete

1. ✅ Run test script: `node test-two-agent-flow.js`
2. ✅ Start backend: `node server.js`
3. ✅ Test in frontend: Request a coding challenge
4. ✅ Verify: Problem displays with 3 sample test cases
5. ✅ Test: "Run" button works
6. ✅ Test: "Submit" button works with all test cases

## Expected Results

### Before Fix
```
❌ Response truncated at ~1900 characters
❌ JSON parsing fails
❌ Error: "Failed to invoke coding agent"
```

### After Fix
```
✅ Complete JSON response (~2500-3500 characters)
✅ Problem statement with 3 examples
✅ 3 visible test cases with explanations
✅ 7-10 hidden test cases added
✅ Total: 10-13 test cases
✅ Challenge displays in frontend
```

## Time Estimate
- Finding the setting: 2-3 minutes
- Making the change: 1 minute
- Saving and updating alias: 2 minutes
- Testing: 3-5 minutes
- **Total: ~10 minutes**

## Need Help?
- See `FIX-TOKEN-LIMIT-ISSUE.md` for detailed explanation
- See `FINAL-DIAGNOSIS.md` for complete test results
- Check AWS Bedrock documentation for your specific model
