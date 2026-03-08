# LeetCode-Style Layout Redesign

## Current Issues

1. **Code editor and Submit button not visible** - Cut off at bottom
2. **Layout is Chatbot-left, Problem+Code-right** - Not intuitive
3. **No Run button** - Users can't test code without submitting
4. **Problem description takes too much space** - Reduces code editor visibility

## Proposed LeetCode-Style Layout

```
┌─────────────────────────────────────────────────────────────┐
│                         Header                               │
├──────────────────────────┬──────────────────────────────────┤
│                          │                                   │
│   Problem Description    │      Code Editor                  │
│   (Collapsible Sections) │      (Large textarea)             │
│                          │                                   │
│   - Description          │      [Run] [Submit]               │
│   - Input Format         │                                   │
│   - Constraints          │      ┌─────────────────────────┐ │
│   - Output Format        │      │  Output / Feedback      │ │
│   - Sample Cases         │      │  (Tabs)                 │ │
│   - Hint                 │      └─────────────────────────┘ │
│                          │                                   │
│                          │      ┌─────────────────────────┐ │
│                          │      │  AI Chatbot             │ │
│                          │      │  (Collapsible)          │ │
│                          │      └─────────────────────────┘ │
└──────────────────────────┴──────────────────────────────────┘
```

## Key Changes

### 1. Layout Structure
- **Left Panel (40%):** Problem description with collapsible sections
- **Right Panel (60%):** Code editor + Output + Chatbot

### 2. New Features
- **Run Button:** Test code without submitting (doesn't lock the problem)
- **Collapsible Chatbot:** Minimize to give more space to code
- **Resizable Panels:** Drag to adjust problem/code split
- **Sticky Header:** Problem title always visible

### 3. Improved UX
- **Larger code editor:** More vertical space
- **Tabbed output:** Output, Test Results, Feedback in tabs
- **Compact problem sections:** Collapsible accordion style
- **Better button visibility:** Always visible at top of code section

## Implementation Plan

### Phase 1: Layout Restructure
1. Change main layout from `[Chatbot | Problem+Code]` to `[Problem | Code+Output+Chat]`
2. Make problem panel scrollable independently
3. Make code panel with fixed header for buttons

### Phase 2: Add Run Functionality
1. Create new backend endpoint `/learning/run-code` (doesn't save submission)
2. Add Run button next to Submit button
3. Show test case results without locking problem

### Phase 3: UI Improvements
1. Make chatbot collapsible/expandable
2. Add panel resizing
3. Improve button styling and visibility
4. Add loading states

### Phase 4: Polish
1. Add keyboard shortcuts (Ctrl+Enter to run, Ctrl+Shift+Enter to submit)
2. Add code formatting button
3. Add reset code button
4. Improve mobile responsiveness

## Files to Modify

### Frontend:
1. `CodingPage.jsx` - Complete layout restructure
2. `CodingPage.css` - New styles for LeetCode layout
3. `problemStatementParser.js` - Already done ✓

### Backend:
1. `learningController.js` - Add `runCode` endpoint
2. `judgeService.js` - Reuse for running code

## Benefits

1. **Better visibility:** Code editor and buttons always visible
2. **More intuitive:** Matches LeetCode/HackerRank UX that users know
3. **More flexible:** Run code multiple times before submitting
4. **Better learning:** Test different approaches without penalty
5. **Cleaner UI:** Organized sections, less clutter

## Next Steps

1. Approve this design
2. Implement Phase 1 (layout restructure)
3. Implement Phase 2 (Run button)
4. Test and refine
5. Deploy

## Estimated Time

- Phase 1: 2-3 hours (major restructure)
- Phase 2: 1 hour (backend + frontend)
- Phase 3: 1-2 hours (UI polish)
- Phase 4: 1 hour (final touches)

**Total: 5-7 hours of development**

This is a significant undertaking but will greatly improve the user experience!
