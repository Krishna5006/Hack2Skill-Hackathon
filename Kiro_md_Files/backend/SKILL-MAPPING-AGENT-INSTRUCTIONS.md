# Skill Mapping Agent - AWS Instructions

## Agent Configuration

**Agent ID**: 3GE5HYWL4P
**Agent Alias**: ZYG8JOISVP
**Region**: ap-south-1

---

## Agent Instructions (Main Configuration)

### Current (Good)
```
Your responsibility:
- Map raw or non-standard skill descriptions to canonical skill IDs from the Knowledge Base.

Rules:
- Use ONLY canonical skill IDs from the Knowledge Base.
- Output ONLY a JSON array of strings.
- Do NOT explain.
- Do NOT invent new skills.
- Deduplicate results.

If no match exists, return an empty array.
```

### Enhanced (Recommended)
```
You are a Skill Mapping Agent.

Your responsibility:
- Map raw or non-standard skill descriptions to canonical skill IDs from the Knowledge Base.

CRITICAL RULES:
- Use ONLY canonical skill IDs from the Knowledge Base (canonical_skills.json)
- Canonical skill IDs use underscores (e.g., "dynamic_programming", "linked_lists")
- NEVER return skills with spaces (e.g., "image processing" is INVALID)
- Output ONLY a JSON array of strings
- Do NOT explain
- Do NOT invent new skills
- Deduplicate results
- If no canonical match exists, return empty array []

Examples:
Input: ["image processing", "matrix operations"]
Output: ["arrays"]

Input: ["networking concepts", "TCP/IP"]
Output: ["computer_networks"]

Input: ["nested conditionals", "logical flow"]
Output: ["arrays"]

Input: ["maintainability", "code quality"]
Output: []

CRITICAL: Only return skill IDs that exist in canonical_skills.json. Never return vague or invented skills.
```

---

## Knowledge Base Instructions

### Current (Good)
```
Use this KB to map user input topics to canonical skill IDs. Return ONLY canonical skill IDs (snake_case). No explanations. No extra text. Output must be valid JSON array.
```

### Enhanced (Recommended)
```
Use this knowledge base to map user input topics to canonical skill IDs from canonical_skills.json.

STRICT RULES:
- Return ONLY canonical skill IDs in snake_case format (e.g., "dynamic_programming")
- NEVER return skills with spaces (e.g., "image processing" is INVALID)
- NEVER invent new skill names
- If no match in canonical_skills.json, return empty array []
- No explanations, no extra text
- Output must be valid JSON array

The knowledge base contains:
1. canonical_skills.json - Complete list of valid skill IDs
2. skill-mapping-rules.txt - Mapping rules and examples

Only return skill IDs that exist in canonical_skills.json.
```

---

## Short Versions (if character limit)

### Agent Instruction (200 chars)
```
Map raw skills to canonical IDs from KB. Use ONLY canonical IDs (snake_case). NEVER return skills with spaces. Output JSON array only. No explanations. If no match, return [].
```
Character count: 182 ✓

### KB Instruction (200 chars)
```
Map to canonical skill IDs from canonical_skills.json. Return ONLY snake_case IDs. NEVER return skills with spaces. Output valid JSON array. No explanations.
```
Character count: 165 ✓

---

## Setup Steps

### 1. Upload Knowledge Base Files to S3

```bash
# Upload skill mapping rules
aws s3 cp Hackveda/knowlegde-base/skill-mapping-rules.txt \
  s3://YOUR-BUCKET-NAME/skill-mapping-rules.txt --force

# Upload canonical skills JSON
aws s3 cp Hackveda/knowlegde-base/canonical_skills.json \
  s3://YOUR-BUCKET-NAME/canonical_skills.json --force
```

### 2. Update Agent in AWS Console

1. **Navigate to Agent**:
   - AWS Bedrock Console → Agents
   - Find agent ID: `3GE5HYWL4P`
   - Click on agent name

2. **Update Agent Instructions**:
   - Click "Edit" in Agent Instructions section
   - Replace with **Enhanced** version above
   - Click "Save"

3. **Verify Knowledge Base**:
   - Scroll to "Knowledge bases" section
   - Ensure both files are attached:
     - `skill-mapping-rules.txt`
     - `canonical_skills.json`
   - If not attached, add them

4. **Update KB Instructions**:
   - Click on the knowledge base
   - Click "Edit"
   - Update instructions with **Enhanced** version above
   - Click "Save"

5. **Sync Knowledge Base**:
   - In Knowledge bases section
   - Click "Sync"
   - Wait for sync to complete (1-2 minutes)

6. **Prepare Agent**:
   - Go to "Aliases" tab
   - Find alias: `ZYG8JOISVP`
   - Click "Prepare"
   - Wait for preparation (30-60 seconds)

### 3. Test in AWS Console

Click "Test" button and try:

**Test 1 - Vague Skills (Should Map to Canonical)**:
```
Input: ["image processing", "matrix operations"]
Expected: ["arrays"]
```

**Test 2 - Networking (Should Map)**:
```
Input: ["networking concepts", "TCP/IP"]
Expected: ["computer_networks"]
```

**Test 3 - Unrelated (Should Return Empty)**:
```
Input: ["maintainability", "code quality"]
Expected: []
```

**Test 4 - Already Canonical (Should Return As-Is)**:
```
Input: ["arrays", "hashing", "dynamic_programming"]
Expected: ["arrays", "hashing", "dynamic_programming"]
```

---

## Validation Checklist

Before considering setup complete:

- [ ] Agent instructions updated with enhanced version
- [ ] Both KB files uploaded to S3:
  - [ ] skill-mapping-rules.txt
  - [ ] canonical_skills.json
- [ ] Knowledge base attached to agent
- [ ] KB instructions updated with enhanced version
- [ ] Knowledge base synced successfully
- [ ] Agent prepared with latest version
- [ ] Alias `ZYG8JOISVP` points to prepared version
- [ ] Test in AWS console shows canonical skills only
- [ ] Test shows no skills with spaces
- [ ] Test shows empty array for unrelated skills

---

## Expected Behavior

### Good Responses ✅

**Input**: `["image processing", "matrix operations"]`
**Output**: `["arrays"]`

**Input**: `["networking concepts", "TCP/IP"]`
**Output**: `["computer_networks"]`

**Input**: `["nested conditionals", "logical flow"]`
**Output**: `["arrays"]`

**Input**: `["recursion basics", "base case"]`
**Output**: `["recursion"]`

**Input**: `["hash maps", "dictionary"]`
**Output**: `["hashing"]`

**Input**: `["maintainability", "code quality"]`
**Output**: `[]`

### Bad Responses ❌

**Input**: `["image processing"]`
**Output**: `["image processing"]` ❌ (should be `["arrays"]`)

**Input**: `["networking"]`
**Output**: `["networking"]` ❌ (should be `["computer_networks"]`)

**Input**: `["matrix operations"]`
**Output**: `["matrix operations"]` ❌ (should be `["arrays"]`)

---

## Canonical Skills Reference

All returned skills MUST be from this list (32 total):

**Data Structures & Algorithms** (25):
- arrays
- strings
- linked_lists
- stacks
- queues
- hashing
- recursion
- backtracking
- binary_search
- sorting
- trees
- binary_search_trees
- heaps_priority_queue
- trie
- graphs
- shortest_path_algorithms
- minimum_spanning_tree
- topological_sort
- union_find
- dynamic_programming
- greedy_algorithms
- bit_manipulation
- math_algorithms
- sliding_window
- complexity_analysis

**CS Fundamentals** (1):
- object_oriented_programming

**Systems** (6):
- operating_systems
- database_management_systems
- computer_networks
- cloud_computing
- devops
- system_design

---

## Troubleshooting

### Issue: Agent returns vague skills with spaces

**Cause**: Agent instructions not updated or KB not synced

**Fix**:
1. Update agent instructions with enhanced version
2. Sync knowledge base
3. Prepare agent
4. Test again

### Issue: Agent returns empty array for everything

**Cause**: KB files not attached or not synced

**Fix**:
1. Verify both files are in S3
2. Attach KB to agent if not attached
3. Sync KB
4. Prepare agent

### Issue: Agent invents new skills

**Cause**: Instructions not strict enough

**Fix**:
1. Use enhanced agent instructions
2. Emphasize "ONLY from canonical_skills.json"
3. Sync and prepare

---

## Backend Validation

The backend now validates and filters skills:

```javascript
// Filter out any non-canonical skills (vague ones)
const validSkills = result.filter(skill => {
  // Canonical skills use underscore format
  // Vague skills use spaces
  const isCanonical = !skill.includes(' ') && skill.length > 0;
  if (!isCanonical) {
    console.log(`⚠️  Filtering out non-canonical skill: "${skill}"`);
  }
  return isCanonical;
});
```

This provides a safety net even if the agent returns vague skills.

---

## Summary

**Current Instructions**: Good foundation
**Enhanced Instructions**: More explicit about canonical-only
**Key Addition**: Examples showing vague → canonical mapping
**Safety Net**: Backend validation filters non-canonical skills

Use the **Enhanced** versions for best results!

---

## Quick Copy-Paste

### Agent Instruction (Enhanced)
```
You are a Skill Mapping Agent.

Your responsibility:
- Map raw or non-standard skill descriptions to canonical skill IDs from the Knowledge Base.

CRITICAL RULES:
- Use ONLY canonical skill IDs from the Knowledge Base (canonical_skills.json)
- Canonical skill IDs use underscores (e.g., "dynamic_programming", "linked_lists")
- NEVER return skills with spaces (e.g., "image processing" is INVALID)
- Output ONLY a JSON array of strings
- Do NOT explain
- Do NOT invent new skills
- Deduplicate results
- If no canonical match exists, return empty array []

Examples:
Input: ["image processing", "matrix operations"]
Output: ["arrays"]

Input: ["networking concepts", "TCP/IP"]
Output: ["computer_networks"]

Input: ["nested conditionals", "logical flow"]
Output: ["arrays"]

Input: ["maintainability", "code quality"]
Output: []

CRITICAL: Only return skill IDs that exist in canonical_skills.json. Never return vague or invented skills.
```

### KB Instruction (Enhanced)
```
Use this knowledge base to map user input topics to canonical skill IDs from canonical_skills.json.

STRICT RULES:
- Return ONLY canonical skill IDs in snake_case format (e.g., "dynamic_programming")
- NEVER return skills with spaces (e.g., "image processing" is INVALID)
- NEVER invent new skill names
- If no match in canonical_skills.json, return empty array []
- No explanations, no extra text
- Output must be valid JSON array

The knowledge base contains:
1. canonical_skills.json - Complete list of valid skill IDs
2. skill-mapping-rules.txt - Mapping rules and examples

Only return skill IDs that exist in canonical_skills.json.
```
