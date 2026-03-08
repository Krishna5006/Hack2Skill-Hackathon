# JSON Escaping Fix for Coding Challenge Generation

## Problem
The coding agent was generating JSON with unescaped quotes inside string values, causing JSON parsing errors:

```
Error: Unexpected token ',', ...", "Yellow", or "Gree"... is not valid JSON
```

Example problematic output:
```json
{
  "problemStatement": "Output should be \"Red\", \"Yellow\", or \"Green\"",
  ...
}
```

The quotes around "Red", "Yellow", and "Green" were not escaped, breaking the JSON structure.

## Root Cause
The agent was using quotes in the problem statement's Output Format section like:
- `Output should be "Red", "Yellow", or "Green"`

These unescaped quotes inside the JSON string value caused parsing failures.

## Solution

### 1. Updated Knowledge Base (`coding-challenge-rules.txt`)

Added explicit instructions to avoid unescaped quotes:

**New Rules:**
- DO NOT use quotes like: `"Red", "Yellow", "Green"` in problem statements
- INSTEAD use: `Red, Yellow, or Green` (no quotes)
- OR use backticks: `` `Red`, `Yellow`, `Green` ``
- Added examples of correct and wrong formats
- Added final checklist before returning JSON

**Key Addition:**
```
CRITICAL: When describing output format in problem statements:
- DO NOT use quotes like: "Red", "Yellow", "Green" 
- INSTEAD use: Red, Yellow, or Green
- OR use backticks: `Red`, `Yellow`, `Green`
- This prevents JSON parsing errors!
```

### 2. Improved Backend Quote Fixing (`codingAgent.js`)

Enhanced the JSON parsing error recovery with multiple strategies:

**Strategy 1: Pattern-based fixing**
- Detects and removes quotes in common patterns like `"Red", "Yellow", or "Green"`
- Replaces with: `Red, Yellow, or Green`

**Strategy 2: Boundary-based escaping**
- Escapes quotes that appear in the middle of string values
- Preserves quotes at string boundaries

**Strategy 3: Character-by-character analysis**
- Last resort fallback
- Analyzes each character to determine if quote is a delimiter or content
- Escapes content quotes, preserves delimiter quotes

**Improved Logging:**
- Shows problematic JSON snippet (first 500 chars)
- Logs each recovery attempt
- Helps debug future issues

## Changes Made

### Files Modified
1. `Hackveda/knowlegde-base/coding-challenge-rules.txt`
   - Added JSON escaping rules with examples
   - Added quote usage guidelines for problem statements
   - Added final checklist

2. `Hackveda/backend/src/agents/codingAgent.js`
   - Enhanced quote-fixing logic with 3-tier strategy
   - Added better error logging
   - Improved pattern matching for common issues

## Testing Recommendations

1. **Test with various output formats:**
   - String outputs with quotes
   - Numeric outputs
   - Boolean outputs
   - Array/list outputs

2. **Test problem statement variations:**
   - With backticks: `` `Red`, `Yellow` ``
   - Without quotes: `Red, Yellow`
   - With escaped quotes: `\"Red\", \"Yellow\"`

3. **Monitor logs for:**
   - First parse failures (should be rare now)
   - Second parse failures (should be very rare)
   - Third parse failures (should never happen)

## Prevention

The knowledge base now explicitly instructs the agent to:
1. Avoid using quotes in problem descriptions
2. Use backticks for code/literal values
3. Use plain text for word lists
4. Follow the final checklist before returning JSON

## Next Steps

1. **Upload updated knowledge base to S3:**
   - File: `coding-challenge-rules.txt`
   - Bucket: Your AWS S3 bucket
   - Sync with Bedrock Agent knowledge base

2. **Monitor agent responses:**
   - Check logs for parse errors
   - Verify quote handling works correctly
   - Collect examples of successful generations

3. **If issues persist:**
   - Review agent logs for patterns
   - Update knowledge base with more specific examples
   - Consider adding post-processing rules

## Example Correct Formats

### Good - No quotes:
```
Output Format: A single line containing Red, Yellow, or Green
```

### Good - Backticks:
```
Output Format: A single line containing `Red`, `Yellow`, or `Green`
```

### Bad - Unescaped quotes:
```
Output Format: A single line containing "Red", "Yellow", or "Green"
```

## Impact
- Prevents JSON parsing errors in coding challenge generation
- Improves reliability of the coding agent
- Provides better error recovery when issues occur
- Makes problem statements cleaner and more readable
