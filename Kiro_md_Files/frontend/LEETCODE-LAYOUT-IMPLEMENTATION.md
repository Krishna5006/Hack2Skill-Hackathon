# LeetCode-Style Layout Implementation Guide

## Current Status
✅ Run button added
✅ Visibility fixed
⏳ Layout restructure needed

## Target Layout

```
┌──────────────────────────────────────────────────────────────┐
│                         Header                                │
├─────────────────────────┬────────────────────────────────────┤
│                         │                                     │
│   Problem Description   │      Code Editor                    │
│   (Left 40%)            │      (Right 60%)                    │
│                         │                                     │
│   Scrollable sections:  │      [Run] [Submit] [Font]          │
│   - Description         │                                     │
│   - Input Format        │      ┌───────────────────────────┐ │
│   - Constraints         │      │                           │ │
│   - Output Format       │      │   Code Textarea           │ │
│   - Sample Cases        │      │   (Large)                 │ │
│   - Hint                │      │                           │ │
│                         │      └───────────────────────────┘ │
│                         │                                     │
│                         │      ┌───────────────────────────┐ │
│                         │      │  Output (Resizable)       │ │
│                         │      │  [Output | Feedback]      │ │
│                         │      └───────────────────────────┘ │
│                         │                                     │
│                         │      ┌───────────────────────────┐ │
│                         │      │  AI Chatbot               │ │
│                         │      │  [Minimize ▼]             │ │
│                         │      │  Messages...              │ │
│                         │      │  [Input] [Send]           │ │
│                         │      └───────────────────────────┘ │
└─────────────────────────┴────────────────────────────────────┘
```

## Key Changes Needed

### 1. JSX Structure Change

**Current:**
```jsx
<div className="coding-main-content">
  <div className="chatbot-panel">...</div>  {/* LEFT */}
  <div className="right-content">
    <div className="problem">...</div>
    <div className="code">...</div>
    <div className="output">...</div>
  </div>
</div>
```

**New:**
```jsx
<div className="coding-main-content">
  <div className="left-panel">  {/* Problem */}
    <div className="problem-description">...</div>
  </div>
  <div className="right-panel">  {/* Code + Output + Chat */}
    <div className="code-section">...</div>
    <div className="output-section">...</div>
    <div className="chatbot-section">...</div>
  </div>
</div>
```

### 2. CSS Changes

**Main Content Grid:**
```css
.coding-main-content {
  display: grid;
  grid-template-columns: 40% 60%;  /* Problem left, Code right */
  gap: 0;
  flex: 1;
  overflow: hidden;
}
```

**Left Panel (Problem):**
```css
.left-panel {
  background: var(--bg-secondary);
  border-right: 1px solid var(--border);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}
```

**Right Panel (Code + Output + Chat):**
```css
.right-panel {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
```

**Chatbot Section (Bottom Right):**
```css
.chatbot-section {
  background: var(--bg-secondary);
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  max-height: 300px;
  min-height: 50px;
}

.chatbot-section.minimized {
  max-height: 50px;
}
```

### 3. Component State Updates

Add:
```javascript
const [showChatbot, setShowChatbot] = useState(true);
const [chatbotMinimized, setChatbotMinimized] = useState(false);
```

### 4. Chatbot Minimize Feature

Add toggle button:
```jsx
<div className="chatbot-header">
  <div className="chatbot-title">
    <span>🤖 AI Assistant</span>
  </div>
  <button 
    className="minimize-btn"
    onClick={() => setChatbotMinimized(!chatbotMinimized)}
  >
    {chatbotMinimized ? '▲' : '▼'}
  </button>
</div>
```

## Implementation Steps

### Step 1: Update JSX Structure (30 min)
1. Move chatbot from left to bottom-right
2. Move problem description to left panel
3. Keep code editor in right panel
4. Add chatbot minimize functionality

### Step 2: Update CSS (20 min)
1. Change grid layout to 40/60 split
2. Style left panel (problem)
3. Style right panel (code + output + chat)
4. Add chatbot minimize styles
5. Add responsive breakpoints

### Step 3: Test & Polish (10 min)
1. Test all functionality
2. Verify resizing works
3. Check theme switching
4. Test on different screen sizes

## Detailed Code Changes

### File: `CodingPage.jsx`

**Replace main content div:**

```jsx
{/* Main Content - NEW LAYOUT */}
<div className="coding-main-content">
  {/* LEFT PANEL - Problem Description */}
  <div className="left-panel">
    <div className="problem-header">
      <h1 className="problem-title">{subtopicTitle}</h1>
      <span className="badge badge-medium">Challenge</span>
    </div>
    
    <div className="problem-description-scroll">
      {/* All problem sections here */}
      {parsedProblem && (
        <>
          {parsedProblem.description && (
            <div className="problem-section">
              <h3>Description</h3>
              <p>{parsedProblem.description}</p>
            </div>
          )}
          {/* ... other sections ... */}
        </>
      )}
    </div>
  </div>

  {/* RIGHT PANEL - Code + Output + Chat */}
  <div className="right-panel">
    {/* Code Editor */}
    <div className="code-section">
      <div className="code-header">
        <span>Your Solution</span>
        <div className="code-actions">
          <button className="btn btn-run" onClick={runCode}>▶ Run</button>
          <button className="btn btn-success" onClick={submitCode}>✓ Submit</button>
        </div>
      </div>
      <textarea className="code-textarea" value={code} onChange={(e) => setCode(e.target.value)} />
    </div>

    {/* Output Section */}
    <div className="output-section" style={{ height: `${outputHeight}px` }}>
      {/* Output tabs and content */}
    </div>

    {/* Chatbot Section */}
    <div className={`chatbot-section ${chatbotMinimized ? 'minimized' : ''}`}>
      <div className="chatbot-header">
        <span>🤖 AI Assistant</span>
        <button onClick={() => setChatbotMinimized(!chatbotMinimized)}>
          {chatbotMinimized ? '▲' : '▼'}
        </button>
      </div>
      {!chatbotMinimized && (
        <>
          <div className="chatbot-messages" ref={chatMessagesRef}>
            {/* Messages */}
          </div>
          <div className="chatbot-input">
            {/* Input */}
          </div>
        </>
      )}
    </div>
  </div>
</div>
```

### File: `CodingPage.css`

**Add new layout styles:**

```css
/* NEW LAYOUT - LeetCode Style */
.coding-main-content {
  display: grid;
  grid-template-columns: 40% 60%;
  flex: 1;
  overflow: hidden;
}

.left-panel {
  background: var(--bg-secondary);
  border-right: 1px solid var(--border);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.problem-header {
  padding: 1.5rem;
  border-bottom: 1px solid var(--border);
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  background: var(--bg-secondary);
  z-index: 10;
}

.problem-description-scroll {
  padding: 1.5rem;
  overflow-y: auto;
}

.right-panel {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.chatbot-section {
  background: var(--bg-secondary);
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  height: 300px;
  flex-shrink: 0;
}

.chatbot-section.minimized {
  height: 50px;
  overflow: hidden;
}

.chatbot-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid var(--border);
  background: var(--bg-tertiary);
}

.minimize-btn {
  background: transparent;
  border: none;
  color: var(--text-primary);
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0.25rem 0.5rem;
}

.minimize-btn:hover {
  color: var(--accent);
}

/* Responsive */
@media (max-width: 1024px) {
  .coding-main-content {
    grid-template-columns: 35% 65%;
  }
}

@media (max-width: 768px) {
  .coding-main-content {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
  }
  
  .left-panel {
    max-height: 300px;
    border-right: none;
    border-bottom: 1px solid var(--border);
  }
}
```

## Benefits of New Layout

1. ✅ **More intuitive** - Matches LeetCode/HackerRank
2. ✅ **Better space usage** - Problem and code side-by-side
3. ✅ **Chatbot accessible** - Always available at bottom
4. ✅ **Minimizable chat** - More space when needed
5. ✅ **Professional look** - Industry-standard layout

## Time Estimate

- JSX restructure: 30 minutes
- CSS updates: 20 minutes
- Testing: 10 minutes
- **Total: ~1 hour**

## Next Steps

1. Backup current CodingPage.jsx
2. Implement JSX changes
3. Update CSS
4. Test thoroughly
5. Deploy

Would you like me to implement this now?
