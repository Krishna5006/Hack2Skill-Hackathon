# Learning Assistant Chatbot - Implementation Guide

## Overview
Added a context-aware AI chatbot for theory (LessonPage) and quiz (QuizPage) pages. The chatbot helps students understand concepts, clarify doubts, and get explanations for quiz questions.

## Features

### Theory Page (LessonPage)
- AI assistant button in header
- Chatbot drawer on the right side (30% width)
- Context includes:
  - Subtopic title
  - Theory content (truncated to 1000 chars)
  - Current page type ('theory')
- Students can ask questions about the theory content
- AI provides additional examples and explanations

### Quiz Page (QuizPage)
- AI assistant button in header
- Chatbot drawer on the right side (30% width)
- Context includes:
  - Subtopic title
  - All quiz questions with their numbers (Q1, Q2, etc.)
  - Current page type ('quiz')
- Students can ask about specific questions using question numbers
- AI provides hints before submission, explanations after submission

## Implementation Details

### Backend

#### 1. New Agent: Learning Assistant
**File:** `Hackveda/backend/src/agents/learningAssistantAgent.js`

- Separate agent from the coding chatbot
- Uses different AWS Bedrock agent ID and alias
- Maintains conversation memory (last 8 messages)
- Accepts learning context (theory, quizzes, page type)

#### 2. Knowledge Base
**File:** `Hackveda/knowlegde-base/learning-assistant-rules.txt`

Rules for the Learning Assistant:
- Help students understand theory and quiz questions
- Provide hints rather than direct answers (before quiz submission)
- Explain concepts with examples and analogies
- Reference question numbers when discussing quiz questions
- Be supportive and encouraging

#### 3. Controller Endpoint
**File:** `Hackveda/backend/src/controllers/learningController.js`

New function: `chatWithLearningAssistant`
- Endpoint: `POST /learning/learning-chat`
- Required parameters:
  - `message`: User's question
  - `sessionId`: Unique session identifier
  - `learningContext`: Object with subtopicTitle, currentPage, theory/quizzes

#### 4. Route
**File:** `Hackveda/backend/src/routes/learningRoutes.js`

Added route: `router.post("/learning-chat", protect, chatWithLearningAssistant);`

### Frontend

#### 1. LessonPage Updates
**File:** `Hackveda/frontend/src/pages/LessonPage.jsx`

Changes:
- Added chatbot state (messages, input, showChatbot)
- Added AI Assistant button in header
- Added chatbot drawer component
- Implemented message sending with context
- Added message formatting (supports code blocks)

**File:** `Hackveda/frontend/src/pages/LessonPage.css`

Added styles:
- Chatbot drawer (30% width, sticky)
- Message bubbles (user/bot)
- Input area with send button
- Responsive layout (mobile: stacked)

#### 2. QuizPage Updates
**File:** `Hackveda/frontend/src/pages/QuizPage.jsx`

Changes:
- Fixed duplicate title rendering (removed second h1)
- Added chatbot state (messages, input, showChatbot)
- Added AI Assistant button in header
- Added chatbot drawer component
- Implemented message sending with quiz context
- Added message formatting (supports code blocks)

**File:** `Hackveda/frontend/src/pages/QuizPage-Enhanced.css`

Added styles:
- Chatbot drawer (30% width, sticky)
- Message bubbles (user/bot)
- Input area with send button
- Responsive layout (mobile: stacked)

## AWS Setup Required

### 1. Create Learning Assistant Agent in AWS Bedrock

1. Go to AWS Bedrock Console
2. Create a new agent: "Learning Assistant Agent"
3. Set agent instructions:

```
You are a Learning Assistant AI for an educational platform.

Your role:
- Help students understand theory concepts and explanations
- Clarify quiz questions and explain why answers are correct/incorrect
- Provide additional examples and analogies
- Break down complex topics into simpler terms
- Encourage critical thinking and deeper understanding

Context Awareness:
- You have access to the current subtopic title
- You have access to the theory content (when on theory page)
- You have access to quiz questions with their serial numbers (when on quiz page)
- Use this context to provide relevant, specific answers

Rules for Theory Page:
- Explain concepts in simple, clear language
- Provide additional examples beyond what's in the theory
- Use analogies to make complex topics relatable
- Encourage students to ask follow-up questions
- Do NOT just repeat the theory verbatim - add value with new perspectives
- If asked about a specific section, reference it clearly

Rules for Quiz Page:
- When students ask about a specific question, use the question number (Q1, Q2, etc.)
- Explain WHY an answer is correct, not just WHAT the answer is
- If a student is stuck, provide hints rather than direct answers
- After quiz submission, explain incorrect answers thoroughly
- Help students understand the underlying concepts, not just memorize answers
- Reference the theory content when explaining quiz concepts

Communication Style:
- Be friendly, supportive, and encouraging
- Use clear, concise language
- Break down complex explanations into steps
- Use bullet points and formatting for clarity
- Celebrate student progress and understanding
- Be patient with repeated questions

What NOT to do:
- Do NOT provide direct answers before quiz submission
- Do NOT be condescending or dismissive
- Do NOT use overly technical jargon without explanation
- Do NOT give up on explaining - try different approaches
- Do NOT discuss topics completely unrelated to the current lesson
```

4. Create knowledge base with `learning-assistant-rules.txt`
5. Sync knowledge base to S3
6. Prepare/Build the agent
7. Create agent alias (e.g., "production")
8. Note the Agent ID and Alias ID

### 2. Update Environment Variables

Add to `Hackveda/backend/.env`:

```
LEARNING_ASSISTANT_AGENT_ID=your_agent_id_here
LEARNING_ASSISTANT_AGENT_ALIAS=your_agent_alias_id_here
```

### 3. Knowledge Base Setup

1. Upload `learning-assistant-rules.txt` to S3 bucket
2. Create knowledge base in AWS Bedrock
3. Link knowledge base to Learning Assistant Agent
4. Sync knowledge base

Knowledge base instruction (less than 200 chars):
```
Use these rules to help students understand theory and quiz questions. Provide hints before quiz submission, detailed explanations after.
```

## Testing

### Theory Page Testing
1. Navigate to any subtopic
2. Click "🤖 AI Assistant" button
3. Ask questions like:
   - "Can you explain this concept in simpler terms?"
   - "Give me another example"
   - "What's the difference between X and Y?"
4. Verify chatbot provides helpful explanations

### Quiz Page Testing
1. Navigate to quiz page
2. Click "🤖 AI Assistant" button
3. Before submission, ask:
   - "Give me a hint for Q1"
   - "What concept does Q2 test?"
4. After submission, ask:
   - "Why was my answer to Q3 wrong?"
   - "Explain the correct answer for Q2"
5. Verify chatbot provides appropriate responses

## Features

✅ Context-aware chatbot for theory and quiz pages
✅ Separate agent from coding chatbot
✅ Conversation memory (last 8 messages)
✅ Code block formatting in messages
✅ Responsive design (mobile-friendly)
✅ Fixed duplicate title in QuizPage
✅ Sticky chatbot drawer
✅ Smooth animations

## Files Modified

### Backend
- `Hackveda/backend/src/agents/learningAssistantAgent.js` (NEW)
- `Hackveda/knowlegde-base/learning-assistant-rules.txt` (NEW)
- `Hackveda/backend/src/controllers/learningController.js` (MODIFIED)
- `Hackveda/backend/src/routes/learningRoutes.js` (MODIFIED)

### Frontend
- `Hackveda/frontend/src/pages/LessonPage.jsx` (MODIFIED)
- `Hackveda/frontend/src/pages/LessonPage.css` (MODIFIED)
- `Hackveda/frontend/src/pages/QuizPage.jsx` (MODIFIED)
- `Hackveda/frontend/src/pages/QuizPage-Enhanced.css` (MODIFIED)

## Next Steps

1. Create Learning Assistant Agent in AWS Bedrock
2. Add environment variables to `.env`
3. Upload knowledge base to S3
4. Test on theory and quiz pages
5. Monitor chatbot responses and refine agent instructions if needed

---

**Status:** ✅ Implementation Complete
**Date:** March 6, 2026
**Priority:** High (enhances learning experience)
