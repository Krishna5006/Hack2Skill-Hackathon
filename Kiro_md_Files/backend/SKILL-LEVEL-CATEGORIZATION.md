# Skill Level Categorization Feature

## Overview

Skills are now categorized by difficulty level (Beginner, Intermediate, Advanced) based on the roadmap level where they were learned. This allows tracking skill progression across different difficulty levels.

## Problem Statement

**Before**: All skills were stored in a flat structure without level information.
- User learns "arrays" in Beginner roadmap → stored as "arrays"
- User learns "arrays" in Advanced roadmap → updates same "arrays" entry
- No way to track progression across difficulty levels

**After**: Skills are categorized by level.
- User learns "arrays" in Beginner roadmap → stored as "arrays [Beginner]"
- User learns "arrays" in Advanced roadmap → stored as "arrays [Advanced]"
- Can track separate confidence for same skill at different levels

## Implementation

### 1. MongoDB Schema Update

**File**: `Hackveda/backend/src/models/User.js`

**Added `level` field to globalSkills**:
```javascript
globalSkills: [{
  skill: String,
  confidence: { type: Number, default: 0 },
  level: { 
    type: String, 
    enum: ['Beginner', 'Intermediate', 'Advanced'],
    default: 'Beginner'
  },
  quizAttempts: { type: Number, default: 0 },
  codeAttempts: { type: Number, default: 0 },
  lastUpdated: { type: Date, default: Date.now },
  confidenceHistory: [{
    value: Number,
    date: Date
  }]
}]
```

**Key Points**:
- `level` is an enum with 3 values: Beginner, Intermediate, Advanced
- Defaults to 'Beginner' for backward compatibility
- Same skill can exist multiple times with different levels

### 2. Skill Service Update

**File**: `Hackveda/backend/src/services/skillService.js`

**Added `level` parameter**:
```javascript
export const updateUserSkills = async ({
  userId,
  skills,
  score,
  isCoding = false,
  level = 'Beginner' // NEW parameter
}) => {
  // ...
  
  // Find existing skill with same ID AND level
  let skill = user.globalSkills.find(s => s.skill === skillId && s.level === level);
  
  if (!skill) {
    // Create new skill with level
    user.globalSkills.push({
      skill: skillId,
      level: level, // Store the difficulty level
      confidence: Math.round(avgScore),
      // ...
    });
  }
  // ...
}
```

**Key Changes**:
- Added `level` parameter with default 'Beginner'
- Find skill by BOTH `skill` AND `level`
- Store level when creating new skill
- Log includes level information

### 3. Controller Updates

**File**: `Hackveda/backend/src/controllers/learningController.js`

#### submitQuiz Update:
```javascript
if (canonicalSkills.length > 0) {
  const roadmap = user.roadmaps[roadmapIndex];
  await updateUserSkills({
    userId,
    skills: canonicalSkills,
    score: normalizedScore,
    isCoding: false,
    level: roadmap.level || 'Beginner' // Pass roadmap level
  });
  
  console.log(`✅ Updated skills for quiz at ${roadmap.level} level: ${canonicalSkills.join(', ')}`);
}
```

#### submitCode Update:
```javascript
if (canonicalSkills.length > 0) {
  const roadmap = user.roadmaps[roadmapIndex];
  await updateUserSkills({
    userId: req.userId,
    skills: canonicalSkills,
    score,
    isCoding: true,
    level: roadmap.level || 'Beginner' // Pass roadmap level
  });
  
  console.log(`✅ Updated skills for coding at ${roadmap.level} level: ${canonicalSkills.join(', ')}`);
}
```

**Key Changes**:
- Extract roadmap level before calling updateUserSkills
- Pass level to updateUserSkills
- Updated console logs to show level

## How It Works

### Example Scenario

**User Journey**:
1. User creates "Beginner" roadmap for "arrays"
2. Completes quiz on arrays → Score: 80
3. Creates "Advanced" roadmap for "arrays"
4. Completes quiz on arrays → Score: 90

**Database State**:
```javascript
globalSkills: [
  {
    skill: "arrays",
    level: "Beginner",
    confidence: 80,
    quizAttempts: 1,
    codeAttempts: 0
  },
  {
    skill: "arrays",
    level: "Advanced",
    confidence: 90,
    quizAttempts: 1,
    codeAttempts: 0
  }
]
```

### Skill Lookup Logic

**Before** (without level):
```javascript
let skill = user.globalSkills.find(s => s.skill === skillId);
// Returns first match, regardless of level
```

**After** (with level):
```javascript
let skill = user.globalSkills.find(s => s.skill === skillId && s.level === level);
// Returns match for specific skill AND level
```

## Benefits

### 1. Track Skill Progression
- See how confidence improves from Beginner → Intermediate → Advanced
- Identify skills that need more practice at higher levels

### 2. Accurate Skill Assessment
- Beginner-level confidence doesn't get mixed with Advanced-level
- Each level has independent confidence tracking

### 3. Better Learning Insights
- User can see: "I'm 90% confident in arrays at Beginner level"
- User can see: "I'm 60% confident in arrays at Advanced level"
- Clear progression path

### 4. Personalized Recommendations
- Can recommend Advanced topics for skills with high Beginner confidence
- Can suggest revisiting Beginner topics if Advanced confidence is low

## Data Structure Examples

### Single Skill, Multiple Levels
```javascript
globalSkills: [
  {
    skill: "recursion",
    level: "Beginner",
    confidence: 85,
    quizAttempts: 2,
    codeAttempts: 1
  },
  {
    skill: "recursion",
    level: "Intermediate",
    confidence: 70,
    quizAttempts: 1,
    codeAttempts: 2
  },
  {
    skill: "recursion",
    level: "Advanced",
    confidence: 55,
    quizAttempts: 1,
    codeAttempts: 1
  }
]
```

### Multiple Skills, Same Level
```javascript
globalSkills: [
  {
    skill: "arrays",
    level: "Beginner",
    confidence: 80
  },
  {
    skill: "linked_lists",
    level: "Beginner",
    confidence: 75
  },
  {
    skill: "stacks",
    level: "Beginner",
    confidence: 90
  }
]
```

## Console Logs

### Quiz Submission
```
📊 Raw skills from quizzes: arrays, loops
🔄 Mapping skills: arrays, loops
📥 Skill mapping agent response: ["arrays","loops"]
✅ Mapped to canonical skills: arrays, loops
✅ Created new skill: arrays [Beginner] with confidence 80 (quiz)
✅ Created new skill: loops [Beginner] with confidence 80 (quiz)
💾 Saved 2 unique skills to globalSkills at Beginner level
✅ Updated skills for quiz at Beginner level: arrays, loops
```

### Coding Submission
```
📊 Raw skills from coding challenge: arrays, two_pointer
🔄 Mapping skills: arrays, two_pointer
✅ Mapped to canonical skills: arrays, two_pointer
✅ Updated skill: arrays [Advanced] confidence 85 (coding)
✅ Created new skill: two_pointer [Advanced] with confidence 75 (coding)
💾 Saved 2 unique skills to globalSkills at Advanced level
✅ Updated skills for coding at Advanced level: arrays, two_pointer
```

## API Response Example

### GET /profile/my-skills

**Response**:
```json
{
  "skills": [
    {
      "skill": "arrays",
      "level": "Beginner",
      "confidence": 80,
      "quizAttempts": 2,
      "codeAttempts": 1,
      "lastUpdated": "2024-01-15T10:30:00Z"
    },
    {
      "skill": "arrays",
      "level": "Advanced",
      "confidence": 65,
      "quizAttempts": 1,
      "codeAttempts": 2,
      "lastUpdated": "2024-01-16T14:20:00Z"
    },
    {
      "skill": "recursion",
      "level": "Intermediate",
      "confidence": 70,
      "quizAttempts": 1,
      "codeAttempts": 1,
      "lastUpdated": "2024-01-14T09:15:00Z"
    }
  ]
}
```

## Migration Notes

### Backward Compatibility
- Existing skills without `level` field will default to 'Beginner'
- No data migration needed - MongoDB handles missing fields gracefully
- Old skills will work normally with default level

### For Existing Users
- Skills earned before this update: level = 'Beginner' (default)
- Skills earned after this update: level = roadmap level
- No data loss or corruption

## Testing Checklist

### Test Case 1: New Skill at Beginner Level
- [ ] Create Beginner roadmap
- [ ] Complete quiz
- [ ] Verify skill stored with level='Beginner'
- [ ] Check console logs show "[Beginner]"

### Test Case 2: Same Skill at Different Levels
- [ ] Create Beginner roadmap for "arrays"
- [ ] Complete quiz → confidence 80
- [ ] Create Advanced roadmap for "arrays"
- [ ] Complete quiz → confidence 90
- [ ] Verify TWO separate entries in globalSkills
- [ ] Verify one has level='Beginner', other has level='Advanced'

### Test Case 3: Skill Update at Same Level
- [ ] Create Beginner roadmap
- [ ] Complete first quiz → confidence 70
- [ ] Complete second quiz → confidence updated (not new entry)
- [ ] Verify only ONE entry for that skill+level combination

### Test Case 4: Coding vs Quiz at Same Level
- [ ] Create Intermediate roadmap
- [ ] Complete quiz → confidence 75
- [ ] Complete coding → confidence updated with 5x weight
- [ ] Verify same skill entry updated (not new entry)

## Future Enhancements

### 1. Skill Progression Visualization
Show user's journey:
```
arrays:
  Beginner: ████████░░ 80%
  Intermediate: ██████░░░░ 60%
  Advanced: ███░░░░░░░ 30%
```

### 2. Level-Up Recommendations
```
"You've mastered 'arrays' at Beginner level (85% confidence)!
Ready to try Intermediate level?"
```

### 3. Skill Gap Analysis
```
"You're strong in 'arrays' at Beginner (90%) but struggling at Advanced (45%).
Consider reviewing intermediate concepts first."
```

### 4. Aggregate Statistics
```
Beginner Skills: 15 (avg confidence: 82%)
Intermediate Skills: 8 (avg confidence: 68%)
Advanced Skills: 3 (avg confidence: 55%)
```

## Files Modified

1. **`Hackveda/backend/src/models/User.js`**
   - Added `level` field to globalSkills schema

2. **`Hackveda/backend/src/services/skillService.js`**
   - Added `level` parameter to updateUserSkills
   - Updated skill lookup to match by skill AND level
   - Updated logging to include level

3. **`Hackveda/backend/src/controllers/learningController.js`**
   - Updated submitQuiz to pass roadmap level
   - Updated submitCode to pass roadmap level
   - Updated console logs to show level

## Summary

✅ Skills now categorized by difficulty level
✅ Same skill can exist at multiple levels
✅ Independent confidence tracking per level
✅ Backward compatible with existing data
✅ Enhanced logging shows level information
✅ Ready for skill progression features

This feature enables better tracking of user learning progression across difficulty levels!
