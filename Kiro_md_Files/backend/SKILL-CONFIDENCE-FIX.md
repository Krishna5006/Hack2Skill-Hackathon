# Skill Confidence Calculation Fix

## Problem
All skills were showing 100% confidence even when users made mistakes. The confidence should decrease when users perform poorly, but it was only increasing.

## Root Cause
The original algorithm in `skillService.js` was:
```javascript
skill.confidence = Math.min(100, Math.round(skill.confidence + score * NEW_WEIGHT));
```

This ALWAYS added to confidence, never subtracted. The `Math.min(100, ...)` only capped the maximum but never allowed confidence to decrease.

## Solution
Implemented a **weighted moving average** algorithm:

```javascript
newConfidence = oldConfidence * OLD_WEIGHT + score * NEW_WEIGHT
```

### Weights
- **Quiz attempts**: `NEW_WEIGHT = 0.4` (40% weight to new score, 60% to existing)
- **Coding attempts**: `NEW_WEIGHT = 0.3` (30% weight to new score, 70% to existing)

Coding has less weight because coding challenges are harder and should have less impact per attempt.

## How It Works

### Example 1: High Score Increases Confidence
- Current confidence: 60
- New quiz score: 90
- Calculation: `60 * 0.6 + 90 * 0.4 = 36 + 36 = 72`
- Result: Confidence increases from 60 → 72 ✅

### Example 2: Low Score Decreases Confidence
- Current confidence: 80
- New quiz score: 20
- Calculation: `80 * 0.6 + 20 * 0.4 = 48 + 8 = 56`
- Result: Confidence decreases from 80 → 56 ✅

### Example 3: Coding Has Less Impact
- Current confidence: 70
- New coding score: 40
- Calculation: `70 * 0.7 + 40 * 0.3 = 49 + 12 = 61`
- Result: Confidence decreases from 70 → 61 (less impact than quiz)

## Additional Features

### Confidence History Tracking
The fix also adds confidence history tracking:
- Stores up to 20 historical confidence values
- Each entry includes the confidence value and timestamp
- Useful for visualizing skill progress over time

### Bounds Checking
Confidence is always kept within 0-100 range:
```javascript
skill.confidence = Math.max(0, Math.min(100, newConfidence));
```

## Testing
Created comprehensive unit tests in `src/services/__tests__/skillService.test.js`:
- ✅ High score increases confidence
- ✅ Low score decreases confidence
- ✅ Coding has less impact than quiz
- ✅ Medium scores slightly adjust confidence
- ✅ Perfect scores increase confidence appropriately
- ✅ Zero scores significantly decrease confidence
- ✅ Bounds are respected (0-100)

All tests pass successfully.

## Files Modified
1. `src/services/skillService.js` - Updated confidence calculation algorithm
2. `src/services/__tests__/skillService.test.js` - Added comprehensive tests

## Impact
Users will now see realistic skill confidence levels that:
- Increase when they perform well
- Decrease when they make mistakes
- Reflect their actual mastery of each skill
- Provide accurate feedback on learning progress
