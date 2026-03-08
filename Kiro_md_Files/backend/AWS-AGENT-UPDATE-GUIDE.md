# AWS Bedrock Agent Update Guide - Structured Text Format

## Current Status
✅ Backend code updated with structured text format parsing
✅ Knowledge base file (`teaching-rules.txt`) updated with new format rules
⏳ AWS Bedrock Agent needs to be updated with new instructions
⏳ Knowledge base needs to be synced in AWS

## What Changed

### Old Format (JSON - FAILED)
The agent was asked to return JSON with escaped newlines (`\n`), but consistently failed to escape properly, causing parse errors.

### New Format (Structured Text - SUCCESS)
The agent now returns plain text with clear section markers:
```
---THEORY---
[markdown with actual newlines]

---YOUTUBE---
[search query]

---SKILLS---
[comma-separated skills]

---ISCODING---
[true or false]
```

## AWS Bedrock Updates Required

### Step 1: Update Tutor Agent Instructions

1. **Open AWS Bedrock Console**
   - Navigate to: AWS Console → Bedrock → Agents
   - Find and select: **Tutor Agent**

2. **Update Agent Instructions**
   - Click "Edit" on the agent
   - Replace the current instructions with the following:

```
You are the Tutor Agent for an educational platform.

CRITICAL: When generating content, return in this EXACT format with markers:

---THEORY---
[Your markdown content - use actual newlines]

## Overview
[Introduction]

## Key Concepts
- Concept 1
- Concept 2

## Explanation
[Detailed content with examples]

## Example
```
code here
```

## Important Points
- Point 1
- Point 2

---YOUTUBE---
[YouTube search query]

---SKILLS---
[Comma-separated skills]

---ISCODING---
[true or false]

Your responsibilities:
- Explain concepts clearly
- Generate focused content (250-350 words)
- Set isCoding=true for DSA topics (arrays, loops, algorithms, data structures)
- Set isCoding=false for pure theory (history, comparisons, overviews)
- Use actual newlines in markdown (not \n)
- Follow the marker format exactly

Coding Decision Rules:
- Set isCoding=true for: implementing algorithms, building data structures, writing functions, coding exercises, practical programming tasks
- Set isCoding=false for: history, comparisons, overviews, definitions, introductions without hands-on implementation
- When in doubt for programming/DSA topics, prefer isCoding=true

Content Requirements:
- Keep content concise (250-350 words)
- Include 1-2 code examples maximum
- Use markdown formatting (##, -, ```)
- Each section must be on its own line after the marker
```

3. **Save the agent instructions**

### Step 2: Update Knowledge Base

1. **Upload Updated File to S3**
   - Navigate to: AWS Console → S3
   - Find the bucket used for the Tutor Agent knowledge base
   - Locate the file: `teaching-rules.txt`
   - Upload the updated version from: `Hackveda/knowlegde-base/teaching-rules.txt`
   - **IMPORTANT**: Overwrite the existing file

2. **Sync Knowledge Base**
   - Navigate to: AWS Console → Bedrock → Knowledge bases
   - Find the knowledge base associated with the Tutor Agent
   - Click "Sync" button
   - Wait for sync to complete (usually 1-5 minutes)
   - Verify sync status shows "Completed"

### Step 3: Prepare/Build the Agent

1. **Prepare the Agent**
   - Navigate back to: AWS Console → Bedrock → Agents → Tutor Agent
   - Click "Prepare" button (top right)
   - Wait for preparation to complete
   - This creates a new version with the updated instructions

2. **Update Agent Alias (if needed)**
   - Check if your agent alias points to "DRAFT" or a specific version
   - If using a specific version, create a new alias or update existing one
   - Point the alias to the latest prepared version

### Step 4: Verify Environment Variables

Ensure your `.env` file has the correct agent configuration:

```env
# Tutor Agent (for content generation)
TUTOR_AGENT_ID=<your-tutor-agent-id>
TUTOR_AGENT_ALIAS=<your-tutor-agent-alias>

# Learning Assistant Agent (for chatbot)
LEARNING_ASSISTANT_AGENT_ID=<your-learning-assistant-agent-id>
LEARNING_ASSISTANT_AGENT_ALIAS=<your-learning-assistant-agent-alias>
```

## Testing the Implementation

### Step 1: Restart Backend Server

```bash
cd Hackveda/backend
npm start
```

### Step 2: Test Content Generation

1. **Create a new roadmap** or use an existing one
2. **Navigate to a topic** and generate subtopics
3. **Click on a subtopic** to generate content
4. **Watch the backend logs** for:

**Success indicators:**
```
📝 Tutor Agent Response: ---THEORY---...
✅ Parsed structured text response
   Theory length: 1234 chars
   YouTube query: if statements tutorial
   Skills: conditionals, control_flow
   isCoding: true
```

**Fallback indicators (still works but agent didn't follow format perfectly):**
```
📝 Tutor Agent Response: ---THEORY---...
❌ Failed to parse structured text: Could not find all required sections
🔧 Attempting fallback extraction...
✅ Fallback extraction successful
```

**Failure indicators (needs investigation):**
```
❌ Failed to parse structured text: Could not find all required sections
❌ Fallback extraction failed: Could not extract theory content
```

### Step 3: Verify Frontend Display

1. **Check theory content** displays correctly with:
   - Proper markdown formatting
   - Code examples render in code blocks
   - Sections are properly separated
   - No `\n` or `\\n` visible in text

2. **Check YouTube query** is generated
3. **Check coding challenge** appears if `isCoding=true`

## Troubleshooting

### Issue: Agent still returns JSON format

**Cause:** Agent instructions not updated or agent not prepared

**Solution:**
1. Verify agent instructions were saved
2. Click "Prepare" button to create new version
3. Ensure alias points to latest version
4. Restart backend server

### Issue: Agent returns text but parsing fails

**Cause:** Agent not following marker format exactly

**Solution:**
1. Check agent response in logs
2. Verify markers are present: `---THEORY---`, `---YOUTUBE---`, etc.
3. Check knowledge base was synced
4. Try regenerating content for a different subtopic

### Issue: Content is truncated or incomplete

**Cause:** Agent hitting token limit

**Solution:**
1. Content is already limited to 250-350 words
2. Check if subtopic title is too complex
3. Try simpler subtopic titles
4. Check agent model settings (token limit)

### Issue: Fallback extraction always triggers

**Cause:** Agent not following exact marker format

**Solution:**
1. This is acceptable - fallback still works
2. Review agent instructions for clarity
3. Add more examples to knowledge base
4. Consider adjusting marker format if needed

## Expected Behavior After Update

### Content Generation Flow

1. **User clicks on subtopic**
2. **Backend sends prompt** with structured format instructions
3. **Agent returns text** with markers:
   ```
   ---THEORY---
   [markdown content]
   ---YOUTUBE---
   [search query]
   ---SKILLS---
   [skills]
   ---ISCODING---
   [true/false]
   ```
4. **Backend parses** using regex markers
5. **Backend stores** in database:
   ```javascript
   {
     theory: "markdown string",
     youtubeQuery: "search query",
     skills: ["skill1", "skill2"],
     isCoding: true
   }
   ```
6. **Frontend displays** formatted content

### Success Metrics

✅ No JSON parse errors in logs
✅ Content displays with proper formatting
✅ Code examples render correctly
✅ No visible `\n` or `\\n` in text
✅ YouTube queries are relevant
✅ Skills are extracted correctly
✅ isCoding flag is accurate

## Rollback Plan

If the new format causes issues:

1. **Revert agent instructions** to previous version
2. **Revert knowledge base file** in S3
3. **Sync knowledge base** again
4. **Prepare agent** with old instructions
5. **Revert backend code** to JSON parsing (git revert)

## Files Modified

- ✅ `Hackveda/backend/src/controllers/learningController.js` - Updated parsing logic
- ✅ `Hackveda/knowlegde-base/teaching-rules.txt` - Updated format rules
- ✅ `Hackveda/backend/STRUCTURED-TEXT-FORMAT.md` - Documentation
- ✅ `Hackveda/backend/AWS-AGENT-UPDATE-GUIDE.md` - This guide

## Next Steps

1. ☐ Update Tutor Agent instructions in AWS Bedrock Console
2. ☐ Upload updated `teaching-rules.txt` to S3
3. ☐ Sync knowledge base in AWS Bedrock
4. ☐ Prepare/Build the agent
5. ☐ Restart backend server
6. ☐ Test content generation
7. ☐ Verify frontend display
8. ☐ Monitor logs for success/failure patterns

---

**Date:** March 6, 2026
**Status:** Ready for AWS updates
**Priority:** High - Fixes critical JSON parsing issues
