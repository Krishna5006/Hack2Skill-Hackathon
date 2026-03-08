# IAM Permissions Fix for Test Case Generator Agent

## Problem
The Test Case Generator Agent fails with error:
```
❌ Coding agent invocation failed: Failed to retrieve resource because it doesn't exist.
```

This error occurs when trying to invoke agent `RTGOKPWEMA` with alias `K02LY9NQYE`.

## Root Causes

### 1. IAM Permissions Missing
The IAM user/role running the backend doesn't have permission to invoke the Test Case Generator Agent.

### 2. Agent Doesn't Exist
The agent ID or alias might be incorrect, or the agent was deleted.

### 3. Region Mismatch
The agent exists in a different AWS region than configured in `.env`.

## Solution Steps

### Step 1: Verify Agent Exists

1. Log in to AWS Console
2. Navigate to **Amazon Bedrock** → **Agents**
3. Ensure region is set to **ap-south-1** (Mumbai) - matches your `.env`
4. Look for agent with ID: `RTGOKPWEMA`

**If agent doesn't exist:**
- You need to create it following `TEST-CASE-GENERATOR-SETUP.md`
- Or update `.env` with the correct agent ID

**If agent exists:**
- Verify the alias ID is `K02LY9NQYE`
- If alias is different, update `.env` with correct alias

### Step 2: Add IAM Permissions

You need to add permissions for the IAM user/role that runs your backend.

#### Option A: Using AWS Console (Recommended)

1. Go to **IAM** → **Users** (or **Roles** if using EC2/Lambda)
2. Find the user/role that your backend uses
3. Click **Add permissions** → **Attach policies directly**
4. Click **Create policy**
5. Use JSON editor and paste:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "InvokeTestCaseGeneratorAgent",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeAgent"
            ],
            "Resource": [
                "arn:aws:bedrock:ap-south-1:*:agent/RTGOKPWEMA",
                "arn:aws:bedrock:ap-south-1:*:agent-alias/RTGOKPWEMA/K02LY9NQYE"
            ]
        },
        {
            "Sid": "InvokeBedrockModels",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "arn:aws:bedrock:ap-south-1::foundation-model/*"
        }
    ]
}
```

6. Name the policy: `TestCaseGeneratorAgentAccess`
7. Create policy
8. Attach it to your IAM user/role

#### Option B: Using AWS CLI

```bash
# Create the policy
aws iam create-policy \
  --policy-name TestCaseGeneratorAgentAccess \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["bedrock:InvokeAgent"],
            "Resource": [
                "arn:aws:bedrock:ap-south-1:*:agent/RTGOKPWEMA",
                "arn:aws:bedrock:ap-south-1:*:agent-alias/RTGOKPWEMA/K02LY9NQYE"
            ]
        }
    ]
}'

# Attach to your IAM user (replace YOUR_USERNAME)
aws iam attach-user-policy \
  --user-name YOUR_USERNAME \
  --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/TestCaseGeneratorAgentAccess
```

### Step 3: Update Existing Bedrock Policy (If You Have One)

If you already have a Bedrock policy attached, you can update it instead:

1. Go to **IAM** → **Policies**
2. Find your existing Bedrock policy (might be named `BedrockAgentAccess` or similar)
3. Click **Edit policy** → **JSON**
4. Add the Test Case Generator Agent to the Resource array:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeAgent"
            ],
            "Resource": [
                "arn:aws:bedrock:ap-south-1:*:agent/SFOFRVB0QS",
                "arn:aws:bedrock:ap-south-1:*:agent/AA7UFR1MZO",
                "arn:aws:bedrock:ap-south-1:*:agent/ZJM4T6UZE1",
                "arn:aws:bedrock:ap-south-1:*:agent/3GE5HYWL4P",
                "arn:aws:bedrock:ap-south-1:*:agent/BDZFCSFZJF",
                "arn:aws:bedrock:ap-south-1:*:agent/RTGOKPWEMA",
                "arn:aws:bedrock:ap-south-1:*:agent/7X8DN7HOOI",
                "arn:aws:bedrock:ap-south-1:*:agent-alias/*/*"
            ]
        }
    ]
}
```

5. Save changes

### Step 4: Verify AWS Credentials

Ensure your backend is using the correct AWS credentials:

1. Check if you're using environment variables:
   ```bash
   echo $AWS_ACCESS_KEY_ID
   echo $AWS_SECRET_ACCESS_KEY
   ```

2. Or check AWS credentials file:
   ```bash
   cat ~/.aws/credentials
   ```

3. Verify the credentials belong to the user/role you just updated

### Step 5: Restart Backend

After updating IAM permissions:

```bash
cd Hackveda/backend
# Kill existing process
# Then restart
node server.js
```

### Step 6: Test the Fix

1. Request a coding challenge from the frontend
2. Check backend logs for:
   ```
   ✅ Coding Agent Response: {...}
   ✅ Test Case Generator Response: 7 hidden test cases
   📊 Total test cases: 10 (3 visible + 7 hidden)
   ```

## Verification Checklist

- [ ] Agent `RTGOKPWEMA` exists in AWS Bedrock (ap-south-1 region)
- [ ] Alias `K02LY9NQYE` is correct
- [ ] IAM policy includes `bedrock:InvokeAgent` permission
- [ ] IAM policy Resource includes agent ARN
- [ ] Backend is using correct AWS credentials
- [ ] Backend has been restarted
- [ ] Test case generation works end-to-end

## Alternative: Use Wildcard Permissions (Development Only)

For development/testing, you can use broader permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeAgent",
                "bedrock:InvokeModel"
            ],
            "Resource": "*"
        }
    ]
}
```

⚠️ **Warning**: Don't use wildcard permissions in production!

## Troubleshooting

### Still Getting "Resource doesn't exist" Error

1. **Double-check agent ID and alias**:
   ```bash
   # List all agents in your region
   aws bedrock-agent list-agents --region ap-south-1
   
   # Get specific agent details
   aws bedrock-agent get-agent --agent-id RTGOKPWEMA --region ap-south-1
   
   # List agent aliases
   aws bedrock-agent list-agent-aliases --agent-id RTGOKPWEMA --region ap-south-1
   ```

2. **Verify region**: Ensure agent is in `ap-south-1` (Mumbai)

3. **Check agent status**: Agent must be in "PREPARED" or "READY" state

4. **Test with AWS CLI**:
   ```bash
   aws bedrock-agent-runtime invoke-agent \
     --agent-id RTGOKPWEMA \
     --agent-alias-id K02LY9NQYE \
     --session-id test-123 \
     --input-text "Generate test cases" \
     --region ap-south-1 \
     response.txt
   ```

### Permission Denied Error

If you get "Access Denied" instead:
- IAM permissions are not yet propagated (wait 1-2 minutes)
- Wrong IAM user/role is being used
- Policy is attached but not active

### Agent Works But Returns Invalid JSON

This is a different issue - see `FIX-TOKEN-LIMIT-ISSUE.md` for:
- Increasing output token limits
- Adjusting agent instructions
- Improving JSON parsing

## Next Steps

Once permissions are fixed:

1. ✅ Test with a simple coding challenge
2. ✅ Verify 10-13 total test cases are generated
3. ✅ Check test case quality and diversity
4. ✅ Monitor for any other errors

## Additional Resources

- [AWS Bedrock Agent Permissions](https://docs.aws.amazon.com/bedrock/latest/userguide/security-iam.html)
- [IAM Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
- `TEST-CASE-GENERATOR-SETUP.md` - Agent creation guide
- `FIX-TOKEN-LIMIT-ISSUE.md` - Token limit solutions
