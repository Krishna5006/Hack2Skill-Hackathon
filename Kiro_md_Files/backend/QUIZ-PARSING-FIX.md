# Quiz Parsing Fix

## Problem
Quiz generation was failing with error: "No valid quiz questions generated"

The tutor agent was returning quiz questions in the correct format, but the parser was failing to extract them because:
1. Questions weren't always separated by blank lines
2. The parser was splitting by double newlines, which didn't work when questions were consecutive
3. No clear format specification in the knowledge base for the agent to follow

## Root Cause
The original parser in `parseQuizText.js` used:
```javascript
const blocks = text.split(/\n\s*\n/)  // Split by blank lines
```

This failed when the agent returned questions without blank line separators, like:
```
Q1: Question text?
A) Option A
...
ANSWER: B
EXPLANATION: ...
SKILLS: ...
Q2: Next question?  // ← No blank line before this
A) Option A
...
```

## Solution

### 1. Improved Quiz Parser
Rewrote `parseQuizText.js` to use regex pattern matching instead of blank line splitting:

```javascript
const questionPattern = /Q\d+:/g;
const matches = [...text.matchAll(questionPattern)];
```

This finds all questions by their `Q<number>:` pattern, then extracts the content between each question marker.

### Key Improvements:
- Works with or without blank lines between questions
- More robust regex patterns for finding components:
  * Question: `/^Q\d+:/`
  * Answer: `/^ANSWER:\s*[A-D]$/i` (case-insensitive)
  * Options: `/^[A-D]\)/` (with closing parenthesis)
- Better error messages showing what's missing in malformed blocks
- Handles extra whitespace gracefully
- Continues parsing even if some questions are malformed

### 2. Updated Knowledge Base
Added quiz format specification to `teaching-rules.txt`:

```
Quiz Generation Format (MANDATORY):
When generating quizzes, you MUST follow this EXACT format:

Q1: <question text>
A) <option A>
B) <option B>
C) <option C>
D) <option D>
ANSWER: <A|B|C|D>
EXPLANATION: <single sentence>
SKILLS: <comma separated skills>
```

This gives the tutor agent clear instructions on the expected format.

## Testing
Created comprehensive test suite in `src/utils/__tests__/parseQuizText.test.js`:

✅ Parses correctly formatted quiz text
✅ Parses quiz text without blank lines between questions
✅ Handles lowercase answer letters
✅ Skips malformed questions and continues parsing
✅ Throws error when no valid questions found
✅ Handles questions with extra whitespace
✅ Handles missing skills line
✅ Handles missing explanation line

All 21 tests pass (8 new quiz parsing tests + 13 existing tests).

## Example: Before vs After

### Before (Failed)
```
⚠️ Skipping malformed quiz block:
Q1: Which of the following problems is most suitable for a recursive solution?
A) Finding the maximum value in an array
B) Calculating the factorial of a number
C) Sorting an array using bubble sort
D) Performing a binary search on a sorted array
⚠️ Skipping malformed quiz block:
ANSWER: B
EXPLANATION: ...
❌ Quiz generation error: No valid quiz questions generated
```

### After (Success)
```
✅ Parsed 4 quiz questions successfully
```

## Files Modified
1. `src/utils/parseQuizText.js` - Rewrote parser with regex pattern matching
2. `src/utils/__tests__/parseQuizText.test.js` - Added comprehensive test suite
3. `knowlegde-base/teaching-rules.txt` - Added quiz format specification

## Impact
- Quiz generation now works reliably regardless of formatting variations
- Better error messages help debug issues when they occur
- Agent has clear format specification to follow
- More robust parsing handles edge cases gracefully
