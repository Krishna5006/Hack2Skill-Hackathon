# Quick Action Guide - Fix Coding Challenge Issues

## 🚀 Immediate Actions Required

### Step 1: Restart Backend (2 minutes)
```bash
cd Hackveda\backend
# Press Ctrl+C to stop current server
node server.js
```

**Why**: 
- User model schema was updated to include `isVisible` and `explanation` fields
- Controller logic updated to automatically set `isVisible` flags

---

### Step 2: Update AWS Coding Agent Instructions (5 minutes)

1. Go to: https://console.aws.amazon.com/bedrock/
2. Navigate to: **Agents** → Find agent with ID `RTGOKPWEMA`
3. Click **Edit** → **Agent Instructions**
4. **Replace entire content** with the text from:
   ```
   Hackveda/backend/CODING-AGENT-INSTRUCTION-FINAL.txt
   ```
5. Click **Save and exit**
6. Click **Prepare** (wait for preparation to complete)
7. Go to **Aliases** → Select your alias → **Update alias** → Select new version
8. Click **Update alias**

**Why**: 
- Prevents agent from returning explanatory text instead of JSON
- Simplifies agent instructions by removing "visible/hidden" terminology
- Agent now just generates 3 test cases with explanations (backend sets visibility)

---

### Step 3: Update Test Case Generator Knowledge Base (5 minutes)

1. Go to: https://console.aws.amazon.com/bedrock/
2. Navigate to: **Knowledge bases** → Find the knowledge base used by Test Case Generator
3. Click **Data sources** → Select your data source
4. **Upload/Update** the file:
   ```
   Hackveda/knowlegde-base/test-case-generation-rules.txt
   ```
5. Click **Sync** (wait for sync to complete - usually 1-2 minutes)

**Why**: 
- Ensures agent properly escapes newlines in JSON output
- Simplifies agent instructions by removing "hidden" terminology
- Agent now just generates 7-10 test cases without explanations (backend sets visibility)

---

### Step 4: Test the Fix (3 minutes)

```bash
cd Hackveda\backend
node test-two-agent-flow.js
```

**Expected Output**:
```
✅ Coding Agent Response: {...}
✅ Successfully parsed 9 hidden test cases
✅ Test Case Generator Response: 9 hidden test cases
📊 Total test cases: 12 (3 visible + 9 hidden)
```

**If you see errors**:
- ❌ "Agent returned explanatory text" → Coding Agent instructions not updated
- ❌ "Bad control character" → Knowledge base not synced
- ❌ "No visible test cases found" → Backend not restarted

---

### Step 5: Test from Frontend (2 minutes)

1. Open browser → Login to application
2. Navigate to any roadmap → Select a coding subtopic
3. Click **"Generate Coding Challenge"** button
4. Verify:
   - ✅ Problem statement appears with examples
   - ✅ Code editor shows starter code
   - ✅ "Run" button works (tests 3 visible cases)
   - ✅ "Submit" button works (tests all cases)

---

## 🔍 What Was Fixed

### Issue 1: "No visible test cases found"
**Root Cause**: Database schema missing `isVisible` field  
**Fix**: Updated `User.js` model to include `isVisible: Boolean` and `explanation: String`

### Issue 2: "Bad control character in string literal"
**Root Cause**: Test Case Generator returning actual newlines instead of `\n`  
**Fix**: Added automatic newline fixing in `testCaseGeneratorAgent.js`

### Issue 3: "The search results provided guidelines..."
**Root Cause**: Coding Agent returning explanatory text instead of JSON  
**Fix**: Added early detection and rejection in `codingAgent.js`

### Issue 4: Confusing "visible/hidden" terminology in agent instructions
**Root Cause**: Agents were told about implementation details they don't need to know  
**Fix**: 
- Removed "visible/hidden" from all agent instructions
- Coding Agent now just generates 3 test cases with explanations
- Test Case Generator now just generates 7-10 test cases without explanations
- Backend controller automatically sets `isVisible=true` for first 3, `isVisible=false` for rest

---

## 📋 Verification Checklist

After completing all steps, verify:

- [ ] Backend server restarted successfully
- [ ] Test script passes without errors
- [ ] Frontend shows problem statement with examples
- [ ] "Run" button executes 3 visible test cases
- [ ] "Submit" button executes all test cases (10-15 total)
- [ ] No console errors about JSON parsing
- [ ] No "No visible test cases found" errors

---

## 🆘 Troubleshooting

### Backend won't start
```bash
# Check if port is in use
netstat -ano | findstr :5000
# Kill the process if needed
taskkill /PID <process_id> /F
```

### Agent still returns explanatory text
- Verify you updated the **correct agent** (ID: RTGOKPWEMA)
- Ensure you clicked **"Prepare"** after saving instructions
- Ensure you **updated the alias** to use the new version
- Wait 1-2 minutes for changes to propagate

### Test cases still have newline errors
- Verify knowledge base **sync completed** (check status in AWS console)
- Try syncing again if it failed
- Check that you uploaded the **correct file** (test-case-generation-rules.txt)

### Frontend still shows "No visible test cases found"
- Verify backend was **restarted** after schema change
- Check MongoDB to ensure new fields exist:
  ```javascript
  db.users.findOne({}, {"roadmaps.topics.subtopics.codingChallenge.testCases": 1})
  ```
- If old data exists, you may need to regenerate coding challenges

---

## 📞 Need Help?

If issues persist after following all steps:

1. Check backend logs for detailed error messages
2. Check browser console for frontend errors
3. Verify AWS agent versions and aliases are correct
4. Ensure MongoDB connection is working
5. Try generating a new coding challenge (not using cached one)

---

## 📚 Related Documentation

- Full details: `Hackveda/backend/CRITICAL-FIXES-APPLIED.md`
- Simplification details: `Hackveda/backend/AGENT-INSTRUCTION-SIMPLIFICATION.md`
- Compact format: `Hackveda/backend/COMPACT-FORMAT-COMPLETE.md`
- Agent instructions: `Hackveda/backend/CODING-AGENT-INSTRUCTION-FINAL.txt`
- Test case rules: `Hackveda/knowlegde-base/test-case-generation-rules.txt`
