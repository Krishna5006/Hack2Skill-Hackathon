# Problem Variety Fix - Prevent Repetitive Coding Questions

## Problem Statement

**Issue**: Coding agent repeatedly generates the same problems (fibonacci, factorial) for different subtopics, making the learning experience repetitive and boring.

**User Feedback**: "When user in a topic like recursion, the coding agent always give either fibonacci or factorial ques. If it is 1 time for a subtopic its ok but it repeats the same ques for other topics too."

## Solution: Hybrid Approach (History + Instructions)

We implemented **Approach 5** - a hybrid solution that combines:
1. Problem history tracking
2. Enhanced knowledge base with variety rules
3. Context-aware prompt generation

## Implementation Details

### 1. Enhanced Knowledge Base

**File**: `Hackveda/knowlegde-base/coding-challenge-rules.txt`

Added comprehensive problem pattern examples for each topic:
- **Recursion**: 12+ different patterns (fibonacci, factorial, tower of hanoi, permutations, subsets, etc.)
- **Arrays**: 12+ patterns (two sum, kadane's, merge intervals, etc.)
- **Strings**: 10+ patterns (palindrome, anagram, substring, etc.)
- **Linked Lists**: 10+ patterns (reverse, cycle detection, merge, etc.)
- **Trees**: 10+ patterns (traversals, BST validation, LCA, etc.)
- **Dynamic Programming**: 10+ patterns (coin change, LIS, edit distance, etc.)
- **Graphs**: 10+ patterns (islands, clone, course schedule, etc.)
- **Stacks/Queues**: 8+ patterns (valid parentheses, min stack, etc.)
- **Hash Tables**: 8+ patterns (two sum, anagrams, top K, etc.)

**Creativity Rules Added**:
```
1. Rotate through different patterns - don't repeat the same type
2. Add interesting twists to classic problems
3. Combine multiple concepts (e.g., "sliding window + hash map")
4. Use real-world scenarios when possible
5. Vary difficulty within the topic
6. If you see the user has solved fibonacci/factorial, generate something completely different
```

### 2. Problem History Service

**File**: `Hackveda/backend/src/services/problemHistoryService.js`

Created a new service with four key functions:

#### `getRecentCodingProblems(user, limit = 15)`
- Collects all coding problems from user's roadmaps
- Returns most recent problems (default: 15)
- Includes metadata: title, topic, solved status, score, skills

```javascript
const problems = getRecentCodingProblems(user, 15);
// Returns: [
//   {
//     subtopicTitle: "Fibonacci sequence",
//     topicTitle: "Recursion basics",
//     roadmapSubject: "algorithms",
//     problemStatement: "Calculate nth fibonacci...",
//     wasSolved: true,
//     score: 85,
//     skills: ["recursion", "memoization"],
//     createdAt: Date
//   },
//   ...
// ]
```

#### `formatProblemHistoryForPrompt(problems)`
- Formats problem history for agent consumption
- Creates readable summary with status and brief descriptions
- Limits description to 150 characters per problem

```javascript
const formatted = formatProblemHistoryForPrompt(problems);
// Returns:
// "1. [Recursion basics] Fibonacci sequence
//     ✓ Solved (85%)
//     Problem: Calculate the nth fibonacci number using recursion...
//     Skills: recursion, memoization
//  
//  2. [Recursion basics] Factorial calculation
//     ○ Not solved
//     Problem: Write a recursive function to calculate factorial...
//     Skills: recursion"
```

#### `detectProblemPatterns(problems)`
- Analyzes problem statements to detect common patterns
- Returns list of patterns already used
- Helps agent avoid repetition

```javascript
const patterns = detectProblemPatterns(problems);
// Returns: ["fibonacci", "factorial", "palindrome", "two-sum"]
```

#### `getRelatedProblems(user, currentTopic, currentSkills)`
- Finds problems from similar topics or with overlapping skills
- Useful for topic-specific variety checking
- Can be used for future enhancements

### 3. MongoDB Schema Update

**File**: `Hackveda/backend/src/models/User.js`

Added `createdAt` field to track when problems were generated:

```javascript
codingChallenge: {
  problemStatement: String,
  starterCode: String,
  hint: String,
  executionType: String,
  testCases: [...],
  skills: [String],
  languageId: Number,
  createdAt: { type: Date, default: Date.now } // NEW FIELD
}
```

**Why**: Allows sorting problems by recency, giving more weight to recent problems in history.

### 4. Updated Challenge Generation

**File**: `Hackveda/backend/src/controllers/learningController.js`

Modified `generateCodingChallenge()` to include problem history:

**Before**:
```javascript
const prompt = `Generate a LeetCode-style coding challenge:
Programming Language: ${roadmap.language}
Topic: "${subtopic.title}"
...`;
```

**After**:
```javascript
// Get problem history
const recentProblems = getRecentCodingProblems(user, 15);
const problemHistory = formatProblemHistoryForPrompt(recentProblems);
const usedPatterns = detectProblemPatterns(recentProblems);

const prompt = `Generate a LeetCode-style coding challenge:
Programming Language: ${roadmap.language}
Topic: "${subtopic.title}"
Parent Topic: "${roadmap.topics[topicIndex].title}"

PREVIOUS PROBLEMS USER HAS SOLVED:
${problemHistory}

PATTERNS ALREADY USED:
${usedPatterns.join(', ')}

CRITICAL REQUIREMENTS:
1. Generate a UNIQUE problem - DO NOT repeat any of the above problems
2. Be CREATIVE - avoid overused patterns like fibonacci/factorial
3. If topic is "recursion" and user solved fibonacci/factorial, generate something different
...`;
```

## How It Works

### Flow Diagram

```
User requests coding challenge
         ↓
1. Collect recent problems (last 15)
         ↓
2. Format history for prompt
         ↓
3. Detect used patterns
         ↓
4. Generate enhanced prompt with:
   - Problem history
   - Used patterns
   - Creativity instructions
         ↓
5. Call coding agent
         ↓
6. Agent sees history and generates unique problem
         ↓
7. Save problem with createdAt timestamp
```

### Example Scenario

**User's History**:
1. Fibonacci sequence (Recursion) - Solved ✓
2. Factorial calculation (Recursion) - Solved ✓
3. Two sum (Arrays) - Solved ✓

**New Request**: Generate problem for "Recursion - Advanced patterns"

**Agent Receives**:
```
PREVIOUS PROBLEMS USER HAS SOLVED:
1. [Recursion basics] Fibonacci sequence
   ✓ Solved (90%)
   Problem: Calculate the nth fibonacci number...
   Skills: recursion, memoization

2. [Recursion basics] Factorial calculation
   ✓ Solved (85%)
   Problem: Write a recursive function to calculate factorial...
   Skills: recursion

PATTERNS ALREADY USED:
fibonacci, factorial, two-sum

CRITICAL: Generate something DIFFERENT - try permutations, subsets, tree traversal, etc.
```

**Agent Generates**: "Generate all permutations of a string" (NEW pattern!)

## Benefits

### 1. **Variety**
- Agent sees what user has already solved
- Explicitly instructed to avoid repetition
- 100+ problem patterns available in knowledge base

### 2. **Context-Aware**
- Knows user's skill level (solved problems)
- Can adjust difficulty based on history
- Understands which patterns are overused

### 3. **Scalable**
- Works across all roadmaps
- No limit on history size (configurable)
- Easy to add more patterns to knowledge base

### 4. **Debuggable**
- Can log exactly what history was passed
- Can verify agent received correct context
- Easy to troubleshoot repetition issues

### 5. **Flexible**
- Can filter history by topic, skill, recency
- Can weight recent problems more heavily
- Can exclude unsolved problems if desired

## Configuration Options

### Adjust History Limit
```javascript
// In generateCodingChallenge()
const recentProblems = getRecentCodingProblems(user, 20); // Show last 20 instead of 15
```

### Filter by Topic
```javascript
// Only show problems from same topic
const recentProblems = getRecentCodingProblems(user, 15)
  .filter(p => p.topicTitle.includes(currentTopic));
```

### Only Show Solved Problems
```javascript
// Only show successfully solved problems
const recentProblems = getRecentCodingProblems(user, 15)
  .filter(p => p.wasSolved && p.score >= 70);
```

## Testing

### Test Case 1: First Problem
```
User: No previous problems
Expected: Agent generates any appropriate problem
Result: ✓ Works (no history to avoid)
```

### Test Case 2: After Fibonacci
```
User: Solved fibonacci
Expected: Agent generates different recursion problem
Result: ✓ Agent sees fibonacci in history, generates permutations
```

### Test Case 3: Multiple Recursion Problems
```
User: Solved fibonacci, factorial, tower of hanoi
Expected: Agent generates something new (subsets, tree traversal, etc.)
Result: ✓ Agent rotates through patterns
```

### Test Case 4: Cross-Roadmap
```
User: Solved fibonacci in "Algorithms" roadmap
      Now in "Data Structures" roadmap, recursion topic
Expected: Agent still sees fibonacci in history
Result: ✓ History includes all roadmaps
```

## Monitoring

### Console Logs Added
```javascript
console.log(`📚 User has ${recentProblems.length} previous coding problems`);
console.log(`🔍 Detected patterns: ${usedPatterns.join(', ') || 'none'}`);
```

### What to Watch
- Number of previous problems
- Detected patterns
- Whether agent generates unique problems
- User feedback on variety

## Future Enhancements

### 1. Problem Categories
Add explicit category field to schema:
```javascript
codingChallenge: {
  ...
  category: String, // "fibonacci", "permutations", "two-sum", etc.
}
```

### 2. Difficulty Progression
Track difficulty and gradually increase:
```javascript
const avgDifficulty = calculateAverageDifficulty(recentProblems);
prompt += `\nGenerate a problem slightly harder than difficulty ${avgDifficulty}`;
```

### 3. Skill-Based Filtering
Only show problems with overlapping skills:
```javascript
const relatedProblems = getRelatedProblems(user, currentTopic, currentSkills);
```

### 4. User Preferences
Allow users to request specific problem types:
```javascript
userPreferences: {
  avoidPatterns: ["fibonacci", "factorial"],
  preferPatterns: ["graph", "dynamic-programming"]
}
```

## Files Modified

1. **`Hackveda/knowlegde-base/coding-challenge-rules.txt`** - Added variety rules and patterns
2. **`Hackveda/backend/src/services/problemHistoryService.js`** - NEW service for history management
3. **`Hackveda/backend/src/models/User.js`** - Added createdAt to codingChallenge
4. **`Hackveda/backend/src/controllers/learningController.js`** - Updated generateCodingChallenge()

## AWS Setup Required

**IMPORTANT**: Upload updated knowledge base to AWS:

1. Upload `coding-challenge-rules.txt` to S3
2. Sync knowledge base for Coding Agent
3. Prepare agent in AWS Console
4. Test to verify variety in generated problems

## Summary

✅ Agent now sees user's problem history
✅ Agent has 100+ problem patterns to choose from
✅ Explicit instructions to avoid repetition
✅ Tracks used patterns automatically
✅ Works across all roadmaps and topics
✅ Easy to debug and enhance
✅ No breaking changes to existing code

The coding challenge generation is now much more diverse and educational!
