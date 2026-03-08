# Remaining Files to Create for PDF Learning Integration

## Status
✅ Created:
- `frontend/src/pages/PDFLearningPage.jsx`
- `frontend/src/pages/PDFLearningPage.css`
- `frontend/src/components/pdf/PDFHeader.jsx`

## Still Need to Create:

### 1. PDF Components CSS
**File**: `frontend/src/components/pdf/PDFComponents.css`
- Styles for all PDF components
- Header, file upload, intent selector, loading, error, output card styles

### 2. File Upload Component
**File**: `frontend/src/components/pdf/FileUpload.jsx`
- Drag & drop functionality
- File validation (PDF only, max 50MB)
- File display with size and remove button

### 3. Intent Selector Component
**File**: `frontend/src/components/pdf/IntentSelector.jsx`
- 4 intent buttons: Teach, Plan, Quiz, Summary
- Each with icon, description, and default prompt

### 4. Loading Spinner Component
**File**: `frontend/src/components/pdf/LoadingSpinner.jsx`
- Step-by-step loading animation
- Shows: Extract → Chunk → Retrieve → Invoke → Generate

### 5. Error Banner Component
**File**: `frontend/src/components/pdf/ErrorBanner.jsx`
- Display API errors
- Dismissable
- Shows error code, message, and optional details

### 6. Output Card Component
**File**: `frontend/src/components/pdf/OutputCard.jsx`
- Display generated content (markdown)
- Show metadata (chunks, tokens, time, model)
- Copy to clipboard button
- Show/hide source references
- "New Query" button to reset

### 7. PDF Learning Hook
**File**: `frontend/src/hooks/usePDFLearning.js`
- API integration with HackathonPDF backend (port 5006)
- State management: idle, loading, success, error
- Methods: submit(file, prompt, intent), reset()

### 8. Update App.jsx Routing
Add route:
```jsx
import PDFLearningPage from './pages/PDFLearningPage';

// In routes:
<Route path="/pdf-learning" element={<PDFLearningPage />} />
```

### 9. Update Dashboard
Add button to access PDF Learning:
```jsx
<button onClick={() => navigate('/pdf-learning')}>
  📄 Learn from PDF
</button>
```

### 10. Configure Vite Proxy (if needed)
**File**: `frontend/vite.config.js`
Add proxy for HackathonPDF backend:
```js
export default defineConfig({
  server: {
    proxy: {
      '/api/learn': {
        target: 'http://localhost:5006',
        changeOrigin: true
      }
    }
  }
})
```

## Quick Implementation Command
Would you like me to:
1. Create all remaining files now (will take multiple responses)
2. Create them in batches (components first, then hook, then routing)
3. Provide you with the complete code for each file to copy manually

## Testing Steps
1. Start HackathonPDF backend: `cd HackathonPDF/backend && npm run dev`
2. Start main frontend: `cd frontend && npm run dev`
3. Navigate to `/pdf-learning` route
4. Upload a PDF and test all 4 intents

## Notes
- HackathonPDF backend must be running on port 5006
- Main backend runs on port 5012
- Frontend will need to proxy API calls to port 5006 for PDF learning
- All AWS/Bedrock configuration stays in HackathonPDF backend
