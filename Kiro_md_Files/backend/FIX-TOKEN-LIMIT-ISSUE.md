# Fix Token Limit Issue - Coding Agent

## Problem
The coding agent is returning incomplete/truncated JSON responses because it's hitting the output token limit. The response gets cut off mid-generation, resulting in invalid JSON.

## Symptoms
- Logs show: `Unterminated string in JSON`
- Response ends abruptly (e.g., `**Sample Output 1:*` instead of complete explanation)
- Parsing attempts fail even after cleanup

## Root Cause
The coding agent needs to generate:
1. Problem statement (500-800 tokens)
2. Starter code (200-400 tokens)
3. Sample inputs/outputs with explanations (300-500 tokens)
4. 3 test cases with explanations (200-300 tokens)
5. Hint and skills (50-100 tokens)

**Total: ~1500-2000 tokens minimum**

Default agent token limits are often 1000-1500 tokens, which is insufficient.

## Solution: Increase Output Token Limit in AWS

### Step 1: Access Agent Configuration
1. Log in to AWS Console
2. Navigate to Amazon Bedrock → Agents
3. Find agent: `BDZFCSFZJF` (Coding Agent)
4. Click "Edit"

### Step 2: Update Inference Configuration
Look for one of these sections:
- "Inference configuration"
- "Model configuration"  
- "Advanced settings"

### Step 3: Increase Token Limit
Find the setting for:
- "Max tokens" or
- "Maximum length" or
- "Max tokens to sample"

**Change it to: 3000-4000 tokens**

Recommended values by model:
- Claude 3.5 Sonnet: 4000 tokens
- Claude 3 Haiku: 3000 tokens
- Amazon Nova Pro: 3500 tokens

### Step 4: Save and Deploy
1. Save the configuration
2. Create a new version of the agent
3. Update the alias to point to the new version
4. Restart your backend server

## Alternative Solution: Simplify Output

If you can't increase the token limit, you can simplify the agent instructions to generate shorter content:

1. Shorter problem descriptions
2. Fewer sample inputs/outputs (just 2 instead of 3)
3. Briefer explanations
4. Simpler starter code

However, this reduces the quality of the generated problems.

## Verification

After increasing the token limit, you should see:
```
✅ Coding Agent Response: {...} (complete JSON with all fields)
✅ Test Case Generator Response: 7 hidden test cases
📊 Total test cases: 10 (3 visible + 7 hidden)
```

No more truncation errors!

## Additional Notes

- The test case generator agent also needs sufficient tokens (2000-2500 recommended)
- Monitor CloudWatch logs to see actual token usage
- If problems persist, consider using a model with larger context windows
