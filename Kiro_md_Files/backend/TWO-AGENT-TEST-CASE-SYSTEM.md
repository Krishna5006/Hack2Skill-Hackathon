# Two-Agent Test Case Generation System

## Overview
This document describes the implementation of a two-agent system for generating coding challenges with 10-15 test cases, solving the token limit issue that occurred when a single agent tried to generate everything.

## Problem
The original coding agent was hitting token limits when trying to generate:
- Problem statement
- Starter code
- Hint
- 10-15 test cases with explanations

This resulted in incomplete JSON responses and parsing errors.

## Solution: Two-Agent Architecture

### Agent 1: Coding Agent
**Purpose**: Generate the core problem and visible test cases

**Responsibilities**:
- Generate problem statement (LeetCode format)
- Generate starter code
- Generate hint
- Generate EXACTLY 3 visible test cases with explanations
- Identify skills tested

**Output**: JSON with problem details + 3 visible test cases

**Agent ID**: `CODING_AGENT_ID` (already configured)

### Agent 2: Test Case Generator Agent
**Purpose**: Generate hidden test cases only

**Responsibilities**:
- Generate 7-10 hidden test cases
- Cover edge cases and boundary conditions
- NO explanations (saves tokens)
- Compact format

**Output**: JSON array of 7-10 test cases

**Agent ID**: `TEST_CASE_GENERATOR_AGENT_ID` (needs to be configured)

## Implementation Details

### Files Created/Modified

1. **New Agent**: `src/agents/testCaseGeneratorAgent.js`
   - Calls the test case generator agent
   - Validates response structure
   - Returns array of hidden test cases

2. **New Knowledge Base**: `knowlegde-base/test-case-generation-rules.txt`
   - Instructions for test case generator agent
   - Emphasizes compact format (no explanations)
   - Provides examples

3. **Updated Controller**: `src/controllers/learningController.js`
   - Modified `generateCodingChallenge` function
   - Now calls both agents sequentially
   - Merges visible + hidden test cases
   - Better error handling

4. **Updated Instructions**: `CODING-AGENT-INSTRUCTION.txt`
   - Clarified that only 3 visible test cases are needed
   - Removed references to generating 10-15 test cases

5. **Updated Environment**: `.env`
   - Added `TEST_CASE_GENERATOR_AGENT_ID`
   - Added `TEST_CASE_GENERATOR_AGENT_ALIAS`

## Workflow

```
User requests coding challenge
         ↓
Step 1: Get problem history
         ↓
Step 2: Call Coding Agent
         → Generate problem statement
         → Generate starter code
         → Generate 3 visible test cases with explanations
         ↓
Step 3: Validate coding agent response
         ↓
Step 4: Call Test Case Generator Agent
         → Generate 7-10 hidden test cases
         → No explanations needed
         ↓
Step 5: Merge test cases
         → 3 visible + 7-10 hidden = 10-13 total
         ↓
Step 6: Save to database
         ↓
Return complete challenge to frontend
```

## Test Case Structure

### Visible Test Cases (3)
```json
{
  "input": "5",
  "expectedOutput": "120",
  "isVisible": true,
  "explanation": "Factorial of 5 is 5*4*3*2*1 = 120"
}
```

### Hidden Test Cases (7-10)
```json
{
  "input": "10",
  "expectedOutput": "3628800",
  "isVisible": false
}
```

## Benefits

1. **Solves Token Limit Issue**: Each agent has a focused task with smaller output
2. **Better Separation of Concerns**: Problem generation vs test case generation
3. **More Reliable**: Less likely to produce incomplete JSON
4. **Scalable**: Can easily adjust number of hidden test cases
5. **Graceful Degradation**: If test case generator fails, still have 3 visible test cases

## Configuration Required

### AWS Bedrock Agent Setup

You need to create a new Bedrock Agent for test case generation:

1. Go to AWS Bedrock Console
2. Create new agent: "Test Case Generator"
3. Upload knowledge base: `test-case-generation-rules.txt`
4. Configure agent with appropriate model
5. Get Agent ID and Alias ID
6. Update `.env` file:
   ```
   TEST_CASE_GENERATOR_AGENT_ID=YOUR_AGENT_ID
   TEST_CASE_GENERATOR_AGENT_ALIAS=YOUR_AGENT_ALIAS
   ```

## Testing

To test the implementation:

1. Ensure both agents are configured in AWS
2. Update `.env` with test case generator credentials
3. Request a coding challenge from the frontend
4. Verify:
   - Problem statement is generated correctly
   - Exactly 3 visible test cases appear in description
   - Run button executes only visible test cases
   - Submit button executes all test cases (visible + hidden)
   - Total test cases = 10-13

## Error Handling

- If coding agent fails: Return error to user
- If test case generator fails: Proceed with only 3 visible test cases (graceful degradation)
- Validation at each step ensures data integrity

## Future Improvements

1. Cache test cases to avoid regeneration
2. Allow users to specify difficulty for test cases
3. Add test case diversity metrics
4. Support custom test case formats for different problem types
