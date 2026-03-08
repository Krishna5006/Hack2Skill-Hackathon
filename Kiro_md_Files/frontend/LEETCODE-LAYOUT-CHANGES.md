# LeetCode Layout - Exact Changes

## Changes to Apply

### 1. State Already Added ✅
```javascript
const [chatbotMinimized, setChatbotMinimized] = useState(false);
```

### 2. Main Content Structure Change

**REPLACE THIS ENTIRE SECTION:**

From line ~418 (after `<div className="coding-main-content">`) 
To line ~680 (before closing `</div>` of coding-main-content)

**WITH THIS NEW STRUCTURE:**

```jsx
      {/* Main Content - NEW LEETCODE LAYOUT */}
      <div className="coding-main-content">
        {/* LEFT PANEL - Problem Description */}
        <div className="left-panel">
          <div className="problem-header">
            <div className="problem-header-content">
              <h1 className="problem-title">{subtopicTitle}</h1>
              <span className="badge badge-medium">Challenge</span>
            </div>
          </div>
          
          <div className="problem-description-scroll">
            {parsedProblem && (parsedProblem.description || parsedProblem.inputFormat || parsedProblem.samples.length > 0) ? (
              <>
                {/* Description Section */}
                {parsedProblem.description && (
                  <div className="problem-section">
                    <h3 className="section-title">Description</h3>
                    <div className="section-content">
                      <p>{parsedProblem.description}</p>
                    </div>
                  </div>
                )}

                {/* Input Format Section */}
                {parsedProblem.inputFormat && (
                  <div className="problem-section">
                    <h3 className="section-title">Input Format</h3>
                    <div className="section-content">
                      <pre className="format-text">{parsedProblem.inputFormat}</pre>
                    </div>
                  </div>
                )}

                {/* Constraints Section */}
                {parsedProblem.constraints && (
                  <div className="problem-section">
                    <h3 className="section-title">Constraints</h3>
                    <div className="section-content">
                      <pre className="format-text">{parsedProblem.constraints}</pre>
                    </div>
                  </div>
                )}

                {/* Output Format Section */}
                {parsedProblem.outputFormat && (
                  <div className="problem-section">
                    <h3 className="section-title">Output Format</h3>
                    <div className="section-content">
                      <pre className="format-text">{parsedProblem.outputFormat}</pre>
                    </div>
                  </div>
                )}

                {/* Sample Test Cases */}
                {parsedProblem.samples && parsedProblem.samples.length > 0 && (
                  <div className="problem-section">
                    <h3 className="section-title">Sample Test Cases</h3>
                    {parsedProblem.samples.map((sample, idx) => (
                      <div key={idx} className="sample-case">
                        <div className="sample-header">Sample {sample.index}</div>
                        <div className="sample-io">
                          <div className="sample-input">
                            <div className="io-label">Input:</div>
                            <pre className="io-content">{sample.input}</pre>
                          </div>
                          <div className="sample-output">
                            <div className="io-label">Output:</div>
                            <pre className="io-content">{sample.output}</pre>
                          </div>
                        </div>
                        {sample.explanation && (
                          <div className="sample-explanation">
                            <div className="io-label">Explanation:</div>
                            <p>{sample.explanation}</p>
                          </div>
                        )}
                      </div>
                    ))}
                  </div>
                )}

                {/* Hint Section */}
                {subtopic?.codingChallenge?.hint && (
                  <div className="problem-section">
                    <h3 className="section-title">💡 Hint</h3>
                    <div className="section-content hint-content">
                      <p>{subtopic.codingChallenge.hint}</p>
                    </div>
                  </div>
                )}
              </>
            ) : (
              <div className="problem-section">
                <div className="section-content">
                  <pre style={{ whiteSpace: 'pre-wrap', lineHeight: '1.6' }}>
                    {subtopic?.codingChallenge?.problemStatement || 'Loading problem...'}
                  </pre>
                </div>
              </div>
            )}
          </div>
        </div>

        {/* RIGHT PANEL - Code + Output + Chatbot */}
        <div className="right-panel">
          {/* Code Section */}
          <div className="code-section">
            <div className="code-header">
              <div className="code-header-left">
                <span className="code-label">Your Solution</span>
              </div>
              <div className="code-header-right">
                <div className="font-size-menu">
                  <button
                    className="btn-icon"
                    onClick={() => setShowFontMenu(!showFontMenu)}
                    title="Font size"
                  >
                    ⋮
                  </button>
                  {showFontMenu && (
                    <div className="font-size-dropdown">
                      {[12, 14, 16, 18, 20].map(size => (
                        <div
                          key={size}
                          className={`font-size-option ${fontSize === size ? 'active' : ''}`}
                          onClick={() => {
                            setFontSize(size);
                            setShowFontMenu(false);
                          }}
                        >
                          {size}px
                        </div>
                      ))}
                    </div>
                  )}
                </div>
                <button 
                  className="btn btn-run" 
                  onClick={runCode}
                  disabled={running || locked}
                  title="Run code to see output (doesn't submit)"
                >
                  {running ? '⏳ Running...' : '▶ Run'}
                </button>
                <button 
                  className="btn btn-success" 
                  onClick={submitCode}
                  disabled={submitting || locked}
                  title="Submit code for evaluation"
                >
                  {submitting ? '⏳ Submitting...' : locked ? '✅ Submitted' : '✓ Submit'}
                </button>
              </div>
            </div>
            <div className="code-editor">
              <textarea
                className="code-textarea"
                value={code}
                onChange={(e) => setCode(e.target.value)}
                style={{ fontSize: `${fontSize}px` }}
                spellCheck="false"
                disabled={locked}
              />
            </div>
          </div>

          {/* Output Section */}
          <div 
            className="resize-handle" 
            onMouseDown={handleMouseDown}
            title="Drag to resize"
          >
            <div className="resize-handle-line"></div>
          </div>
          <div className="output-section" style={{ height: `${outputHeight}px` }}>
            <div className="output-header">
              <div className="output-tabs">
                <button
                  className={`output-tab ${activeTab === 'output' ? 'active' : ''}`}
                  onClick={() => setActiveTab('output')}
                >
                  Output
                </button>
                <button
                  className={`output-tab ${activeTab === 'feedback' ? 'active' : ''}`}
                  onClick={() => setActiveTab('feedback')}
                >
                  Feedback
                </button>
              </div>
            </div>

            <div className="output-content">
              {activeTab === 'output' ? (
                <div className="output-display">
                  <pre className={`output-text ${!output ? 'output-placeholder' : ''}`}>
                    {output || 'Click "Run" or "Submit" to see results'}
                  </pre>
                  {locked && score !== null && (
                    <div className="output-actions">
                      <div className="score-badge">
                        Score: {score}/60
                      </div>
                      <button className="btn btn-primary" onClick={onFinish}>
                        Finish Subtopic →
                      </button>
                    </div>
                  )}
                </div>
              ) : (
                <div className="feedback-content">
                  {score !== null && (
                    <div className="score-display">
                      <h3>Score: {score}/60</h3>
                    </div>
                  )}
                  {feedback && (
                    <div className="feedback-text">
                      <h4>Evaluator Feedback:</h4>
                      <p>{feedback}</p>
                    </div>
                  )}
                  {!feedback && !locked && (
                    <p className="output-placeholder">Submit your code to see feedback</p>
                  )}
                </div>
              )}
            </div>
          </div>

          {/* Chatbot Section - NEW LOCATION (Bottom Right) */}
          <div className={`chatbot-section ${chatbotMinimized ? 'minimized' : ''}`}>
            <div className="chatbot-header">
              <div className="chatbot-title">
                <span>🤖</span>
                <span>AI Assistant</span>
              </div>
              <button 
                className="minimize-btn"
                onClick={() => setChatbotMinimized(!chatbotMinimized)}
                title={chatbotMinimized ? "Expand chatbot" : "Minimize chatbot"}
              >
                {chatbotMinimized ? '▲' : '▼'}
              </button>
            </div>
            
            {!chatbotMinimized && (
              <>
                <div className="chatbot-messages" ref={chatMessagesRef}>
                  {messages.map((message, idx) => (
                    <div key={idx} className={`message ${message.type}`}>
                      <div className="message-avatar">
                        {message.type === 'user' ? '👤' : '🤖'}
                      </div>
                      <div className="message-content">
                        {formatMessageContent(message.text)}
                      </div>
                    </div>
                  ))}
                </div>
                <div className="chatbot-input-container">
                  <div className="chatbot-input-wrapper">
                    <input
                      type="text"
                      className="chatbot-input"
                      placeholder="Ask for hints..."
                      value={chatInput}
                      onChange={(e) => setChatInput(e.target.value)}
                      onKeyPress={handleKeyPress}
                    />
                    <button
                      className={`mic-btn ${isListening ? 'listening' : ''}`}
                      onClick={startListening}
                      title={isListening ? 'Stop listening' : 'Speak to AI'}
                    >
                      🎤
                    </button>
                    <button className="send-btn" onClick={handleSendMessage}>
                      Send
                    </button>
                  </div>
                </div>
              </>
            )}
          </div>
        </div>
      </div>
```

This is the complete new structure. Should I proceed with applying this change?
