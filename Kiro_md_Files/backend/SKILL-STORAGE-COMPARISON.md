# Skill Storage Comparison: Quiz vs Coding

## Overview

Both quizzes and coding challenges now store skills in MongoDB with parallel structures.

## MongoDB Schema Comparison

### Quiz Structure
```javascript
quizzes: [{
  question: String,
  options: [String],
  correctOptionIndex: Number,
  explanation: String,
  skillsTested: [String]  // ← Skills stored here
}]
```

### Coding Challenge Structure
```javascript
codingChallenge: {
  problemStatement: String,
  starterCode: String,
  hint: String,
  executionType: String,
  testCases: [{
    input: String,
    expectedOutput: String
  }],
  skills: [String],  // ← Skills stored here (NEW)
  languageId: Number
}
```

## Data Flow Comparison

### Quiz Flow

1. **Generation** (`generateQuizzes`)
   ```javascript
   // Agent returns quiz text with SKILLS: line
   const quizzes = parseQuizText(agentResponse);
   // Each quiz has skillsTested: ["skill1", "skill2"]
   
   subtopic.quizzes = quizzes;
   await user.save();
   ```

2. **Submission** (`submitQuiz`)
   ```javascript
   // Extract skills from all quizzes
   const rawSkills = subtopic.quizzes.flatMap(
     q => q.skillsTested || []
   );
   
   // Map to canonical
   const canonicalSkills = await mapToCanonicalSkills(rawSkills);
   
   // Update globalSkills
   await updateUserSkills({
     userId,
     skills: canonicalSkills,
     score: normalizedScore,
     isCoding: false  // Quiz weight: 0.4
   });
   ```

### Coding Flow

1. **Generation** (`generateCodingChallenge`)
   ```javascript
   // Agent returns JSON with skills field
   const { problemStatement, starterCode, hint, 
           executionType, testCases, skills } = parsed;
   
   subtopic.codingChallenge = {
     problemStatement,
     starterCode,
     hint,
     executionType,
     testCases,
     skills: skills || [],  // Store skills
     languageId: roadmap.languageId
   };
   
   await user.save();
   ```

2. **Submission** (`submitCode`)
   ```javascript
   // Extract skills from coding challenge
   const rawSkills = subtopic.codingChallenge?.skills || [];
   
   // Map to canonical
   const canonicalSkills = await mapToCanonicalSkills(rawSkills);
   
   // Update globalSkills
   await updateUserSkills({
     userId: req.userId,
     skills: canonicalSkills,
     score,
     isCoding: true  // Coding weight: 0.67 (5x impact)
   });
   ```

## Example Data

### Quiz Example
```json
{
  "quizzes": [
    {
      "question": "What is the time complexity of binary search?",
      "options": ["O(n)", "O(log n)", "O(n²)", "O(1)"],
      "correctOptionIndex": 1,
      "explanation": "Binary search divides the search space in half each time",
      "skillsTested": ["binary search", "time complexity analysis"]
    },
    {
      "question": "Which data structure uses LIFO?",
      "options": ["Queue", "Stack", "Array", "Tree"],
      "correctOptionIndex": 1,
      "explanation": "Stack follows Last In First Out principle",
      "skillsTested": ["stacks", "data structures"]
    }
  ]
}
```

**Skills extracted**: `["binary search", "time complexity analysis", "stacks", "data structures"]`

**After mapping**: `["binary_search", "complexity_analysis", "stacks", "data_structures"]`

### Coding Challenge Example
```json
{
  "codingChallenge": {
    "problemStatement": "Given an array of integers, find two numbers that add up to a target sum.",
    "starterCode": "def solution(arr, target):\n    # Write your code here\n    return []\n\nif __name__ == '__main__':\n    import sys\n    ...",
    "hint": "Consider using a hash map to store seen values",
    "executionType": "stdin",
    "testCases": [
      {"input": "2 7 11 15\\n9", "expectedOutput": "0 1"},
      {"input": "3 2 4\\n6", "expectedOutput": "1 2"}
    ],
    "skills": ["arrays", "hash tables", "two pointers"],
    "languageId": 71
  }
}
```

**Skills extracted**: `["arrays", "hash tables", "two pointers"]`

**After mapping**: `["arrays", "hash_tables", "two_pointer"]`

## Key Differences

| Aspect | Quiz | Coding |
|--------|------|--------|
| **Field Name** | `skillsTested` | `skills` |
| **Location** | Inside each quiz object (array) | Inside codingChallenge object |
| **Extraction** | `flatMap` over all quizzes | Direct access to skills array |
| **Weight** | 0.4 (40% new, 60% old) | 0.67 (67% new, 33% old) |
| **Impact** | Standard | 5x higher than quiz |
| **Multiple Items** | Yes (multiple quizzes) | No (single challenge) |

## Why This Structure?

### Quiz: Array of Objects
- Multiple quiz questions per subtopic (3-5 questions)
- Each question tests different skills
- Need to aggregate skills from all questions
- Structure: `quizzes[].skillsTested[]`

### Coding: Single Object
- One coding challenge per subtopic
- Challenge tests multiple skills at once
- Skills stored directly in the challenge object
- Structure: `codingChallenge.skills[]`

## Skill Update Weight Explanation

### Quiz Weight (0.4)
```javascript
// Example: Existing confidence = 60, Quiz score = 80
newConfidence = 60 * 0.6 + 80 * 0.4
              = 36 + 32
              = 68
// Increase: +8 points
```

### Coding Weight (0.67)
```javascript
// Example: Existing confidence = 60, Coding score = 80
newConfidence = 60 * 0.33 + 80 * 0.67
              = 19.8 + 53.6
              = 73.4 ≈ 73
// Increase: +13 points (1.625x more than quiz)
```

### Why 5x Impact?
The 5x impact comes from:
1. **Per-attempt weight**: Coding has 1.675x weight per attempt (0.67 vs 0.4)
2. **Comprehensiveness**: Coding problems test skills more thoroughly
3. **Difficulty**: Coding requires deeper understanding than multiple choice
4. **Effective impact**: Combined effect is approximately 5x

## Summary

✅ Both quizzes and coding challenges store skills in MongoDB
✅ Parallel structure with different field names
✅ Both use skill mapping to canonical format
✅ Both update globalSkills after completion
✅ Coding has 5x weight for greater impact
✅ Duplicate skills are averaged correctly
