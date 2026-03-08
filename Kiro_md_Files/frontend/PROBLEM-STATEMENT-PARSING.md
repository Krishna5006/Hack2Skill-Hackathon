# Problem Statement Parsing and Structured Display

## Problem
Problem statements were displayed as plain text paragraphs, making them hard to read and understand. The backend was generating markdown-formatted statements with headers and code blocks, but the frontend wasn't parsing or displaying them properly.

## Solution
Created a parser utility and updated the UI to display problem statements in a structured, LeetCode-style format with clear sections.

## Changes Made

### 1. Created Problem Statement Parser (`frontend/src/utils/problemStatementParser.js`)
- Parses markdown-formatted problem statements
- Extracts sections: Description, Input Format, Constraints, Output Format, Sample Test Cases, Explanations
- Handles code blocks (```) and markdown headers (**)
- Returns structured data object

### 2. Updated CodingPage Component (`frontend/src/pages/CodingPage.jsx`)
- Imported parser utility
- Added `parsedProblem` state to store parsed sections
- Parse problem statement on load
- Replaced plain text display with structured sections

### 3. Added Structured Display Styling (`frontend/src/pages/CodingPage.css`)
- Section titles with accent underline
- Formatted code blocks for Input/Output/Constraints
- Sample test cases in grid layout (side-by-side Input/Output)
- Explanation sections with proper spacing
- Hint section with special styling (border-left accent)
- Responsive design (stacks on mobile)

## Features

### Structured Sections
1. **Description** - Clear problem explanation
2. **Input Format** - Formatted with monospace font
3. **Constraints** - Listed clearly with monospace font
4. **Output Format** - Expected output description
5. **Sample Test Cases** - Multiple samples with:
   - Input (code block)
   - Output (code block)
   - Explanation (text)
6. **Hint** - Special highlighted section with 💡 icon

### Visual Design
- LeetCode-style layout
- Clear section headers with accent color
- Code blocks with proper formatting
- Side-by-side Input/Output display
- Collapsible problem description (toggle button)
- Scrollable when content is long
- Dark/Light theme support

### Responsive
- Desktop: Side-by-side Input/Output
- Mobile: Stacked layout
- Adjustable max-height with scroll

## Backend Format Expected
The parser expects problem statements in this format:

```
**Problem Title**
[Brief description]

**Problem Description:**
[Detailed explanation]

**Input Format:**
- First line: [description]
- Second line: [description]

**Constraints:**
- [constraint 1]
- [constraint 2]

**Output Format:**
[Expected output format]

**Sample Input 0:**
```
[actual input]
```

**Sample Output 0:**
```
[actual output]
```

**Explanation 0:**
[Explanation text]
```

## Testing
- Tested with existing problem statements
- Handles missing sections gracefully
- Works with multiple sample test cases
- Responsive on different screen sizes
- Theme switching works correctly

## Files Modified
1. `Hackveda/frontend/src/utils/problemStatementParser.js` (NEW)
2. `Hackveda/frontend/src/pages/CodingPage.jsx`
3. `Hackveda/frontend/src/pages/CodingPage.css`

## Result
Problem statements are now displayed in a clean, structured format similar to LeetCode, making them much easier to read and understand. Users can clearly see:
- What the problem is asking
- Input/Output formats
- Constraints
- Sample test cases with explanations
- Helpful hints

The collapsible design allows users to hide the description when they want to focus on coding.
