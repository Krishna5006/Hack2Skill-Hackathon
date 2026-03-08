# Quick Start: Structured Text Format

**Time Required:** 15-20 minutes  
**Difficulty:** Easy (copy/paste operations)

---

## What You Need to Do

The backend code is ready. You just need to update AWS Bedrock Console.

---

## Step-by-Step Instructions

### 1. Update Agent Instructions (5 min)

1. Open: [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/)
2. Click: **Agents** (left sidebar)
3. Find: **Tutor Agent**
4. Click: **Edit**
5. Find: **Instructions** field
6. Open: `Hackveda/backend/AWS-COPY-PASTE-INSTRUCTIONS.txt`
7. Copy: Section 1 (between ---START COPY--- and ---END COPY---)
8. Paste: Into Instructions field (replace all existing text)
9. Click: **Save**

### 2. Upload Knowledge Base File (3 min)

1. Open: [AWS S3 Console](https://console.aws.amazon.com/s3/)
2. Find: Your knowledge base bucket
3. Locate: `teaching-rules.txt` file
4. Click: **Upload**
5. Select: `Hackveda/knowlegde-base/teaching-rules.txt`
6. Check: **Replace existing file**
7. Click: **Upload**

### 3. Sync Knowledge Base (2 min)

1. Open: [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/)
2. Click: **Knowledge bases** (left sidebar)
3. Find: Your knowledge base
4. Click: **Sync**
5. Wait: Until status shows "Completed" (1-5 minutes)

### 4. Prepare Agent (2 min)

1. Open: [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/)
2. Click: **Agents** (left sidebar)
3. Find: **Tutor Agent**
4. Click: **Prepare** (top right button)
5. Wait: Until preparation completes

### 5. Test (5 min)

1. Restart backend:
   ```bash
   cd Hackveda/backend
   npm start
   ```

2. Open frontend and generate content for a subtopic

3. Check backend logs for:
   ```
   ✅ Parsed structured text response
   ```

4. Verify content displays correctly in frontend

---

## Success Indicators

✅ No JSON parse errors in logs  
✅ Content displays with proper formatting  
✅ Code examples render correctly  
✅ No visible `\n` or `\\n` in text  

---

## If Something Goes Wrong

1. Check: Agent instructions were saved
2. Check: Knowledge base sync completed
3. Check: Agent was prepared
4. Check: Backend server was restarted
5. Read: `AWS-AGENT-UPDATE-GUIDE.md` for detailed troubleshooting

---

## Files You Need

- `AWS-COPY-PASTE-INSTRUCTIONS.txt` - Copy/paste text for AWS
- `teaching-rules.txt` - Upload to S3
- `AWS-AGENT-UPDATE-GUIDE.md` - Detailed guide
- `STRUCTURED-TEXT-FORMAT-STATUS.md` - Full status report

---

## That's It!

After these 5 steps, your content generation will use the new structured text format and JSON parsing errors will be eliminated.
