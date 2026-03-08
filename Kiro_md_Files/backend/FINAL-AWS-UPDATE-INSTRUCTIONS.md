# Final AWS Update Instructions - Complete Guide

**Date:** March 6, 2026  
**Status:** Ready for AWS Updates  
**Priority:** High

---

## What Needs to Be Updated

You have TWO things to update in AWS Bedrock:

1. **Tutor Agent Instructions** - Updated to include quiz generation rules
2. **Knowledge Base File** - Updated teaching-rules.txt with code snippet rules

---

## Update 1: Tutor Agent Instructions

### Location
AWS Console → Bedrock → Agents → Tutor Agent → Edit → Instructions

### What Changed
The Tutor Agent now handles BOTH:
- Theory content generation (with ---THEORY--- markers)
- Quiz generation (with Q1:, CODE:, ANSWER: format)

### New Instructions Include
- Theory content generation format (structured text with markers)
- Quiz generation format (Q/A format with CODE: field)
- CODE SNIPPET RULES (critical for quiz fix)
- Good and bad examples for both formats

### Copy From
`Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt` - Section 1

### Instructions Preview
```
You are the Tutor Agent for an educational platform.

Your responsibilities:
- Generate theory content for learning
- Generate quiz questions for assessment
- Explain concepts clearly and step by step

===========================================
THEORY CONTENT GENERATION
===========================================
[structured text format with markers]

===========================================
QUIZ GENERATION
===========================================
[Q/A format with CODE: field]

CODE SNIPPET RULES (CRITICAL):
- If question asks about code, MUST provide actual code
- If cannot provide code, DO NOT create code-based questions
- Code snippets should be 1-5 lines maximum
...
```

---

## Update 2: Knowledge Base File

### Location
1. AWS Console → S3 → Your Knowledge Base Bucket
2. Upload: `Hackveda/knowlegde-base/teaching-rules.txt`

### What Changed
- Added CODE SNIPPET RULES section
- Added GOOD and BAD examples for quizzes
- Clarified when to use CODE: field
- Added warning about not creating code questions without code

### KB Instruction (200 chars)
```
Use these rules for teaching, theory content, and quiz generation. Follow structured text format with markers. For quizzes with code, provide actual code snippets.
```

### Copy From
- File: `Hackveda/knowlegde-base/teaching-rules.txt`
- KB Instruction: `Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt` - Section 2

---

## Step-by-Step AWS Update Process

### Step 1: Update Tutor Agent Instructions (5 min)

1. Open AWS Bedrock Console
2. Navigate to: **Agents** (left sidebar)
3. Find and click: **Tutor Agent**
4. Click: **Edit** button
5. Find: **Instructions** field (large text area)
6. Open: `Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt`
7. Copy: Everything between `---START COPY---` and `---END COPY---` in Section 1
8. Paste: Into the Instructions field (replace all existing text)
9. Click: **Save**

### Step 2: Upload Knowledge Base File (3 min)

1. Open AWS S3 Console
2. Navigate to: Your knowledge base bucket
3. Find: `teaching-rules.txt` file
4. Click: **Upload** button
5. Select: `Hackveda/knowlegde-base/teaching-rules.txt` from your local machine
6. Check: **Replace existing file** option
7. Click: **Upload**
8. Wait: Until upload completes

### Step 3: Update KB Instruction (2 min)

1. Open AWS Bedrock Console
2. Navigate to: **Knowledge bases** (left sidebar)
3. Find: Your knowledge base
4. Click: **Edit**
5. Find: **Instruction** field (200 character limit)
6. Open: `Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt`
7. Copy: Text from Section 2 (between ---START COPY--- and ---END COPY---)
8. Paste: Into Instruction field
9. Verify: Character count is under 200
10. Click: **Save**

### Step 4: Sync Knowledge Base (2 min)

1. Stay in: AWS Bedrock Console → Knowledge bases
2. Select: Your knowledge base
3. Click: **Sync** button (top right)
4. Wait: Until sync status shows "Completed" (1-5 minutes)
5. Verify: Sync completed successfully

### Step 5: Prepare Agent (2 min)

1. Navigate back to: AWS Bedrock Console → Agents
2. Select: **Tutor Agent**
3. Click: **Prepare** button (top right)
4. Wait: Until preparation completes
5. Verify: New version created

### Step 6: Verify Alias (1 min)

1. Stay in: Tutor Agent page
2. Click: **Aliases** tab
3. Check: Your alias points to latest version or "DRAFT"
4. If needed: Update alias to point to latest version

---

## Verification Checklist

After completing all steps:

### AWS Console Verification
- [ ] Tutor Agent instructions updated and saved
- [ ] teaching-rules.txt uploaded to S3
- [ ] Knowledge base instruction updated
- [ ] Knowledge base sync completed successfully
- [ ] Agent prepared (new version created)
- [ ] Agent alias points to correct version

### Backend Verification
- [ ] Restart backend server: `cd Hackveda/backend && npm start`
- [ ] Check logs for startup errors
- [ ] Verify environment variables are set

### Functional Testing
- [ ] Generate theory content for a subtopic
- [ ] Check logs for `✅ Parsed structured text response`
- [ ] Verify content displays correctly in frontend
- [ ] Generate quizzes for a subtopic
- [ ] Check that code snippets are provided when appropriate
- [ ] Verify code blocks display in quiz questions
- [ ] Test quiz without code snippets (should work normally)

---

## Expected Results

### Theory Content Generation
✅ Agent returns structured text with markers  
✅ Backend parses successfully  
✅ Content displays with proper formatting  
✅ Code examples render in code blocks  
✅ No JSON parse errors  

### Quiz Generation
✅ Agent provides code snippets for code-based questions  
✅ Agent avoids code-based questions if can't provide code  
✅ Code snippets are simple and relevant (1-5 lines)  
✅ Quiz questions display correctly  
✅ Code blocks render with dark background  
✅ Questions without code display normally  

---

## Troubleshooting

### Issue: Agent still returns old format

**Solution:**
1. Verify agent instructions were saved
2. Click "Prepare" button to create new version
3. Ensure alias points to latest version
4. Restart backend server

### Issue: Code snippets still missing

**Solution:**
1. Check knowledge base was synced
2. Verify teaching-rules.txt was uploaded
3. Check KB instruction was updated
4. Try regenerating quizzes for a new subtopic

### Issue: Agent creates code questions without code

**Solution:**
1. Verify agent instructions include CODE SNIPPET RULES
2. Check knowledge base has BAD example
3. Agent may need more examples - add to knowledge base
4. Try with different subtopics

---

## Files Reference

### For Copy/Paste
- `Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt` - All copy/paste text

### For Upload
- `Hackveda/knowlegde-base/teaching-rules.txt` - Upload to S3

### For Reference
- `Hackveda/backend/TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt` - Full agent instructions
- `Hackveda/backend/QUIZ-CODE-SNIPPET-FIX-COMPLETE.md` - Quiz fix documentation
- `Hackveda/backend/STRUCTURED-TEXT-FORMAT-STATUS.md` - Theory fix documentation

---

## Summary of Changes

### Agent Instructions
- **Before:** Only theory content generation
- **After:** Theory content + quiz generation with code snippet rules

### Knowledge Base
- **Before:** Basic teaching rules
- **After:** Teaching rules + quiz rules + code snippet rules + examples

### KB Instruction
- **Before:** Generic teaching rules reference
- **After:** Specific mention of quizzes and code snippets

---

## Time Estimate

- Update agent instructions: 5 minutes
- Upload KB file: 3 minutes
- Update KB instruction: 2 minutes
- Sync KB: 2 minutes (+ 1-5 min wait)
- Prepare agent: 2 minutes
- Verify: 1 minute
- **Total: ~15-20 minutes**

---

## Next Steps After AWS Updates

1. Restart backend server
2. Test theory content generation
3. Test quiz generation
4. Verify code snippets display
5. Monitor logs for any errors
6. Test with multiple subtopics
7. Verify both code and non-code quizzes work

---

**Status:** Ready for AWS updates  
**Risk Level:** Low (changes are additive, fallback available)  
**Rollback:** Revert agent instructions and KB file if needed

---

## Quick Reference

**Agent Instructions:** Section 1 in AWS-COPY-PASTE-INSTRUCTIONS.txt  
**KB File:** Hackveda/knowlegde-base/teaching-rules.txt  
**KB Instruction:** Section 2 in AWS-COPY-PASTE-INSTRUCTIONS.txt  

**AWS Console Links:**
- Agents: AWS Console → Bedrock → Agents
- Knowledge Bases: AWS Console → Bedrock → Knowledge bases
- S3: AWS Console → S3

---

Good luck with the updates! The changes are straightforward and should take about 15-20 minutes to complete.
