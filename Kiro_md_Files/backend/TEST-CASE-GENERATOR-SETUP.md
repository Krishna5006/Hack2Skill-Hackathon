# Test Case Generator Agent Setup Guide

## Quick Setup Instructions

### Step 1: Create the Agent in AWS Bedrock

1. Log in to AWS Console
2. Navigate to Amazon Bedrock
3. Go to "Agents" section
4. Click "Create Agent"

### Step 2: Configure Agent Settings

**Agent Name**: `Test-Case-Generator`

**Agent Description**: 
```
Generates 7-10 hidden test cases for coding challenges. 
Focuses on edge cases, boundary conditions, and diverse test scenarios.
```

**Model Selection**: Choose one of:
- Claude 3.5 Sonnet (recommended for best quality)
- Claude 3 Haiku (faster, lower cost)
- Amazon Nova Pro (good balance)

**Instructions**: Copy and paste from `knowlegde-base/test-case-generation-rules.txt`

### Step 3: Create Knowledge Base (Optional but Recommended)

1. In the agent configuration, go to "Knowledge bases"
2. Click "Add knowledge base"
3. Create new knowledge base:
   - Name: `test-case-generation-kb`
   - Data source: Upload `test-case-generation-rules.txt`
4. Associate with agent

### Step 4: Create Agent Alias

1. After creating the agent, go to "Aliases"
2. Click "Create alias"
3. Name: `production` or `v1`
4. Associate with the latest version

### Step 5: Get Credentials

After creating the agent and alias:

1. Copy the **Agent ID** (format: `XXXXXXXXXX`)
2. Copy the **Alias ID** (format: `XXXXXXXXXX`)

### Step 6: Update Environment Variables

Edit `Hackveda/backend/.env`:

```env
# Test Case Generator Agent
TEST_CASE_GENERATOR_AGENT_ID=YOUR_AGENT_ID_HERE
TEST_CASE_GENERATOR_AGENT_ALIAS=YOUR_AGENT_ALIAS_HERE
```

Replace `YOUR_AGENT_ID_HERE` and `YOUR_AGENT_ALIAS_HERE` with the values from Step 5.

### Step 7: Test the Agent

1. Restart your backend server
2. Request a coding challenge from the frontend
3. Check the backend logs for:
   ```
   ✅ Coding Agent Response: {...}
   ✅ Test Case Generator Response: 7 hidden test cases
   📊 Total test cases: 10 (3 visible + 7 hidden)
   ```

## Troubleshooting

### Agent Not Found Error
- Verify Agent ID and Alias ID are correct
- Ensure agent is in the same AWS region as configured in `.env` (ap-south-1)
- Check IAM permissions for Bedrock access

### Invalid JSON Response
- Review agent instructions in AWS console
- Ensure knowledge base is properly attached
- Try regenerating with a simpler problem first

### Too Few Test Cases
- Agent should generate 7-10 test cases
- If consistently generating fewer, adjust instructions
- Check model temperature settings (should be around 0.7)

### Test Cases Have Placeholders
- Agent instructions emphasize "NO placeholders"
- May need to add more examples to knowledge base
- Try different model (Claude 3.5 Sonnet is most reliable)

## Cost Optimization

- Use Claude 3 Haiku for lower costs (still good quality)
- Test case generation is typically < 500 tokens output
- Expected cost: ~$0.001-0.005 per challenge generation

## Monitoring

Monitor agent performance:
1. AWS CloudWatch logs for agent invocations
2. Backend console logs for response validation
3. User feedback on test case quality

## Agent Prompt Template

The agent receives prompts in this format:

```
Generate 7-10 hidden test cases for this coding problem:

PROBLEM STATEMENT:
[Full problem description]

PROGRAMMING LANGUAGE: [C++/Python/Java/etc]

EXISTING VISIBLE TEST CASES (for reference):
Test 1:
Input: 5
Output: 120

Test 2:
Input: 3
Output: 6

Test 3:
Input: 0
Output: 1

REQUIREMENTS:
1. Generate 7-10 additional test cases
2. Cover edge cases, boundary conditions, and complex scenarios
3. Ensure diversity in test inputs
4. NO explanations needed (these are hidden test cases)
5. Return ONLY valid JSON with testCases array

Generate the hidden test cases now.
```

## Expected Response Format

```json
{
  "testCases": [
    {"input": "1", "expectedOutput": "1"},
    {"input": "10", "expectedOutput": "3628800"},
    {"input": "7", "expectedOutput": "5040"},
    {"input": "15", "expectedOutput": "1307674368000"},
    {"input": "2", "expectedOutput": "2"},
    {"input": "12", "expectedOutput": "479001600"},
    {"input": "8", "expectedOutput": "40320"}
  ]
}
```

## Next Steps

After setup is complete:
1. Test with various problem types
2. Monitor test case quality
3. Adjust agent instructions if needed
4. Consider adding more examples to knowledge base
