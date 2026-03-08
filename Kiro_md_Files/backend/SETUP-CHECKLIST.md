# Setup Checklist - Test Case Generator Agent

## Prerequisites
- [ ] AWS Account with Bedrock access
- [ ] IAM permissions for Bedrock Agent operations
- [ ] Backend server running

## AWS Bedrock Setup

### 1. Create Agent
- [ ] Log in to AWS Console
- [ ] Navigate to Amazon Bedrock → Agents
- [ ] Click "Create Agent"
- [ ] Name: `Test-Case-Generator`
- [ ] Description: `Generates 7-10 hidden test cases for coding challenges`
- [ ] Select model: Claude 3.5 Sonnet (recommended) or Claude 3 Haiku

### 2. Configure Agent Instructions
- [ ] Copy content from `knowlegde-base/test-case-generation-rules.txt`
- [ ] Paste into agent instructions field
- [ ] Save agent configuration

### 3. Create Knowledge Base (Optional)
- [ ] Click "Add knowledge base"
- [ ] Create new knowledge base
- [ ] Upload `test-case-generation-rules.txt`
- [ ] Associate with agent

### 4. Create Alias
- [ ] Go to agent "Aliases" tab
- [ ] Click "Create alias"
- [ ] Name: `production` or `v1`
- [ ] Associate with latest version
- [ ] Save alias

### 5. Get Credentials
- [ ] Copy Agent ID (format: `XXXXXXXXXX`)
- [ ] Copy Alias ID (format: `XXXXXXXXXX`)

## Backend Configuration

### 6. Update Environment Variables
- [ ] Open `Hackveda/backend/.env`
- [ ] Find these lines:
  ```env
  TEST_CASE_GENERATOR_AGENT_ID=YOUR_AGENT_ID_HERE
  TEST_CASE_GENERATOR_AGENT_ALIAS=YOUR_AGENT_ALIAS_HERE
  ```
- [ ] Replace with your actual Agent ID and Alias ID
- [ ] Save file

### 7. Restart Backend
- [ ] Stop backend server (Ctrl+C)
- [ ] Start backend: `npm start` or `node server.js`
- [ ] Verify no errors in startup logs

## Testing

### 8. Test Coding Challenge Generation
- [ ] Open frontend application
- [ ] Navigate to a roadmap
- [ ] Select a topic with coding requirement
- [ ] Click "Generate Challenge" or similar button
- [ ] Wait for generation to complete

### 9. Verify Backend Logs
Check for these log messages:
- [ ] `📚 User has X previous coding problems`
- [ ] `✅ Coding Agent Response: {...}`
- [ ] `✅ Test Case Generator Response: X hidden test cases`
- [ ] `📊 Total test cases: X (3 visible + Y hidden)`

### 10. Verify Frontend Display
- [ ] Problem statement displays correctly
- [ ] 3 visible test cases shown in description
- [ ] Each visible test case has explanation
- [ ] Run button works (executes only visible test cases)
- [ ] Submit button works (executes all test cases)

### 11. Test Run Button
- [ ] Click "Run" button
- [ ] Should execute only 3 visible test cases
- [ ] Should show detailed input/output for each
- [ ] Should display pass/fail status

### 12. Test Submit Button
- [ ] Click "Submit" button
- [ ] Should execute all test cases (visible + hidden)
- [ ] Should show total pass/fail count
- [ ] Should calculate score
- [ ] Should support multiple submissions
- [ ] Should calculate average score

## Troubleshooting

### If Agent Not Found Error
- [ ] Verify Agent ID is correct
- [ ] Verify Alias ID is correct
- [ ] Check AWS region matches `.env` (ap-south-1)
- [ ] Verify IAM permissions

### If Invalid JSON Response
- [ ] Review agent instructions in AWS console
- [ ] Ensure knowledge base is attached
- [ ] Try with simpler problem first
- [ ] Check model selection (Claude 3.5 Sonnet recommended)

### If Too Few Test Cases
- [ ] Check agent logs in CloudWatch
- [ ] Verify agent instructions are complete
- [ ] Try adjusting model temperature
- [ ] Review test case generation prompt

### If Graceful Degradation Occurs
If you see: `⚠️  Proceeding with only visible test cases`
- [ ] Test case generator agent failed
- [ ] System continues with 3 visible test cases only
- [ ] Check agent configuration
- [ ] Review CloudWatch logs for errors

## Success Criteria

✅ All checkboxes above are checked
✅ Backend logs show successful agent calls
✅ Frontend displays 3 visible test cases
✅ Run button executes only visible test cases
✅ Submit button executes all test cases
✅ Multiple submissions work correctly
✅ Average score calculation works

## Next Steps After Setup

1. Monitor agent performance over time
2. Collect user feedback on test case quality
3. Adjust agent instructions if needed
4. Consider adding more examples to knowledge base
5. Optimize for cost if needed (use Claude 3 Haiku)

## Support Resources

- `TWO-AGENT-TEST-CASE-SYSTEM.md` - Architecture overview
- `TEST-CASE-GENERATOR-SETUP.md` - Detailed setup guide
- `IMPLEMENTATION-COMPLETE.md` - Implementation summary
- AWS Bedrock Documentation
- Backend logs for debugging

## Estimated Time
- AWS Setup: 10-15 minutes
- Backend Configuration: 2-3 minutes
- Testing: 5-10 minutes
- **Total: ~20-30 minutes**

---

**Note**: If you encounter any issues not covered in this checklist, refer to the troubleshooting section in `TEST-CASE-GENERATOR-SETUP.md` or check the backend logs for detailed error messages.
