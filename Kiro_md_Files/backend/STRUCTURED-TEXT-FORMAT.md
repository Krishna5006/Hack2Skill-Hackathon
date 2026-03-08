# Structured Text Format for Content Generation

## Problem Solved
The Tutor Agent was unable to consistently return valid JSON with properly escaped newlines. This caused parsing errors and required complex fallback logic.

## New Solution: Structured Text Format

Instead of asking the agent to return JSON, we now use a **structured text format with clear markers** that's much easier for the agent to follow.

### Format Structure

```
---THEORY---
[Markdown content with actual newlines]

## Overview
...

## Key Concepts
...

---YOUTUBE---
[YouTube search query]

---SKILLS---
[Comma-separated skills]

---ISCODING---
[true or false]
```

### Example Agent Response

```
---THEORY---
## Overview
If statements allow programs to make decisions based on conditions.

## Key Concepts
- Conditional execution: code runs only when condition is true
- Boolean expressions: conditions that evaluate to true/false
- Code blocks: indented code that runs conditionally

## Explanation
An if statement evaluates a condition and executes code only if the condition is true.

## Example
```python
age = 18
if age >= 18:
    print('Adult')
```
This checks if age is 18 or more.

## Important Points
- Conditions must be boolean
- Indentation matters
- Use comparison operators

---YOUTUBE---
if statements programming tutorial

---SKILLS---
conditionals, control_flow, boolean_logic

---ISCODING---
true
```

## Backend Parsing

The backend extracts each section using regex markers:

```javascript
const theoryMatch = agentResponse.match(/---THEORY---([\s\S]*?)---YOUTUBE---/);
const youtubeMatch = agentResponse.match(/---YOUTUBE---([\s\S]*?)---SKILLS---/);
const skillsMatch = agentResponse.match(/---SKILLS---([\s\S]*?)---ISCODING---/);
const isCodingMatch = agentResponse.match(/---ISCODING---([\s\S]*?)$/);
```

Then constructs the database object:

```javascript
{
  theory: theoryMatch[1].trim(),
  youtubeQuery: youtubeMatch[1].trim(),
  skills: skillsMatch[1].trim().split(',').map(s => s.trim()),
  isCoding: isCodingMatch[1].trim().toLowerCase() === 'true'
}
```

## Benefits

✅ **No JSON escaping needed** - Agent can use actual newlines
✅ **Clear section markers** - Easy for agent to follow
✅ **Simpler parsing** - Regex-based extraction is straightforward
✅ **More reliable** - No JSON syntax errors
✅ **Better content** - Agent focuses on content, not JSON formatting
✅ **Fallback support** - Can extract partial content if needed

## Fallback Logic

If the structured format parsing fails, the backend attempts fallback extraction:

1. Find `---THEORY---` marker and extract until next marker
2. Find `---YOUTUBE---` marker and extract until next marker
3. Find `---SKILLS---` marker and extract until next marker
4. Find `---ISCODING---` marker and extract remaining text
5. Use defaults if any section is missing

## Agent Instructions

The agent prompt now includes:

```
CRITICAL: Return your response in this EXACT format with these markers:

---THEORY---
[Your markdown content here - use actual newlines, not \n]

---YOUTUBE---
[YouTube search query - one line only]

---SKILLS---
[Comma-separated skills]

---ISCODING---
[true or false - one word only]
```

## Database Storage

The parsed content is stored in the same database format as before:

```javascript
subtopic.content = {
  theory: parsed.theory,        // Markdown string with actual newlines
  youtubeQuery: parsed.youtubeQuery,  // String
  skills: parsed.skills || []   // Array of strings
};

subtopic.isCoding = parsed.isCoding;  // Boolean
```

## Testing

After implementing this change:

1. Restart backend server
2. Generate content for a new subtopic
3. Look for log: `✅ Parsed structured text response`
4. Verify content displays correctly in frontend

## Expected Logs

Success:
```
📝 Tutor Agent Response: ---THEORY---...
✅ Parsed structured text response
   Theory length: 1234 chars
   YouTube query: if statements tutorial
   Skills: conditionals, control_flow
   isCoding: true
```

Fallback:
```
📝 Tutor Agent Response: ---THEORY---...
❌ Failed to parse structured text: Could not find all required sections
🔧 Attempting fallback extraction...
✅ Fallback extraction successful
```

## Files Modified

- `Hackveda/backend/src/controllers/learningController.js` - Updated prompt and parsing logic
- `Hackveda/backend/STRUCTURED-TEXT-FORMAT.md` - This documentation

---

**Status:** ✅ Implemented
**Date:** March 6, 2026
**Priority:** High (fixes JSON parsing issues)
