# AWS Setup for Skill Tracking Fix

## Quick Setup Instructions

### Step 1: Upload Knowledge Base to S3

1. Open AWS S3 Console
2. Navigate to your knowledge base bucket
3. Upload the updated file:
   - **File**: `Hackveda/knowlegde-base/coding-challenge-rules.txt`
   - **Location**: Same folder as existing coding challenge KB

### Step 2: Sync Knowledge Base

1. Open AWS Bedrock Console
2. Go to "Knowledge bases"
3. Find the Coding Agent knowledge base
4. Click "Sync" button
5. Wait for sync to complete (usually 1-2 minutes)

### Step 3: Prepare Agent

1. Go to "Agents" in Bedrock Console
2. Find Coding Agent:
   - **Agent ID**: BDZFCSFZJF
   - **Alias**: MNAD2YJYBP
3. Click "Prepare" button
4. Wait for preparation to complete

### Step 4: Test

Run a test to verify the agent now returns skills:

```bash
# Test prompt
Generate a LeetCode-style coding challenge:

Programming Language: Python
Topic: "Array manipulation"

Requirements:
- Return valid JSON with all required fields including skills
- Include 2-4 test cases
```

Expected response should include:
```json
{
  "problemStatement": "...",
  "starterCode": "...",
  "hint": "...",
  "executionType": "stdin",
  "testCases": [...],
  "skills": ["arrays", "iteration"]
}
```

## Verification

After AWS setup, test the complete flow:

1. **Create a roadmap** with a coding topic
2. **Generate subtopics**
3. **Generate coding challenge** - should include skills
4. **Submit code** - should update globalSkills
5. **Check `/profile/my-skills`** - should show the skills

## Console Logs to Watch

When testing, watch for these console logs:

```
📊 Raw skills from coding challenge: arrays, iteration
🔄 Mapping skills: arrays, iteration
📥 Skill mapping agent response: ["arrays","loops"]
✅ Mapped to canonical skills: arrays, loops
✅ Updated 2 skills for coding: arrays, loops
✅ Created new skill: arrays with confidence 75 (coding)
✅ Created new skill: loops with confidence 75 (coding)
💾 Saved 2 unique skills to globalSkills
```

## Troubleshooting

### Issue: Agent doesn't return skills field
- **Solution**: Make sure knowledge base is synced and agent is prepared
- **Check**: View agent's knowledge base in AWS Console

### Issue: Skills field is empty array
- **Solution**: Agent may need more specific prompt or examples
- **Check**: Test with different topics to see if some work

### Issue: Skills not appearing in globalSkills
- **Solution**: Check console logs for mapping errors
- **Check**: Verify skill mapping agent is working correctly

## Related Files

- Knowledge Base: `Hackveda/knowlegde-base/coding-challenge-rules.txt`
- Controller: `Hackveda/backend/src/controllers/learningController.js`
- Skill Service: `Hackveda/backend/src/services/skillService.js`
- Skill Mapping: `Hackveda/backend/src/services/skillMappingService.js`

## Summary

After completing these steps:
- Coding challenges will include skills field
- Skills will be stored in codingChallenge.skills
- Skills will be mapped to canonical format
- Skills will be saved to globalSkills with 5x weight
- Duplicate skills will be averaged correctly
