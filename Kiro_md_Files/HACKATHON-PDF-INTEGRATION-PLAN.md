# HackathonPDF Integration Plan

## Overview
Integrate the HackathonPDF frontend into the main frontend application as a new page accessible from the dashboard.

## Integration Steps

### 1. Create PDF Learning Page
- **File**: `frontend/src/pages/PDFLearningPage.jsx`
- Convert TypeScript components to JavaScript/JSX
- Integrate with existing routing

### 2. Copy Required Components
Copy and convert these components from `HackathonPDF/frontend/src/components/`:
- `Header.tsx` → `frontend/src/components/pdf/PDFHeader.jsx`
- `FileUpload.tsx` → `frontend/src/components/pdf/FileUpload.jsx`
- `IntentSelector.tsx` → `frontend/src/components/pdf/IntentSelector.jsx`
- `LoadingSpinner.tsx` → `frontend/src/components/pdf/LoadingSpinner.jsx`
- `ErrorBanner.tsx` → `frontend/src/components/pdf/ErrorBanner.jsx`
- `OutputCard.tsx` → `frontend/src/components/pdf/OutputCard.jsx`

### 3. Create API Hook
- **File**: `frontend/src/hooks/usePDFLearning.js`
- API endpoint: `http://localhost:5006/api/learn` (HackathonPDF backend)
- Methods: submit, reset
- State: loading, success, error

### 4. Add CSS Styles
Copy styles from `HackathonPDF/frontend/src/index.css` to:
- `frontend/src/pages/PDFLearningPage.css`

Key styles needed:
- Glass morphism effects
- Gradient backgrounds
- Animated orbs
- Glow buttons

### 5. Update Routing
Add route in `frontend/src/App.jsx`:
```jsx
import PDFLearningPage from './pages/PDFLearningPage';

// Add route
<Route path="/pdf-learning" element={<PDFLearningPage />} />
```

### 6. Add Dashboard Button
Update dashboard to include "PDF Learning" button:
- Icon: 📄 or 📚
- Label: "Learn from PDF"
- Route: `/pdf-learning`

### 7. Configure API Proxy
Update `frontend/vite.config.js` or `frontend/package.json` proxy:
```json
{
  "proxy": {
    "/api": {
      "target": "http://localhost:5006",
      "changeOrigin": true
    }
  }
}
```

## Backend Configuration
The HackathonPDF backend runs separately on port 5006:
```bash
cd HackathonPDF/backend
npm run dev
```

Ensure `.env` is configured with:
- AWS credentials
- Bedrock Agent ID
- S3 bucket name

## Features to Integrate
1. **PDF Upload** - Drag & drop or click to upload
2. **Intent Selection** - Teach, Plan, Quiz, Summary
3. **Custom Prompts** - User can customize learning request
4. **AI-Generated Content** - Display formatted markdown output
5. **Source References** - Show relevant PDF chunks used
6. **Metadata Display** - Processing time, tokens, model version

## API Endpoint
```
POST http://localhost:5006/api/learn
Content-Type: multipart/form-data

Fields:
- file: PDF file
- prompt: Learning request
- intent: teach | plan | quiz | summary
- userId: (optional)
- sessionId: (optional)
```

## Next Steps
1. Create the PDF learning page component
2. Copy and adapt all sub-components
3. Add routing and dashboard integration
4. Test with HackathonPDF backend running
5. Style to match existing UI theme

## Notes
- Keep HackathonPDF backend separate (port 5006)
- Main backend remains on port 5012
- Frontend will proxy API calls to port 5006 for PDF learning features
- All AWS/Bedrock logic stays in HackathonPDF backend
