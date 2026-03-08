# Project Cleanup Guide - Unnecessary Files & Folders

## ⚠️ CRITICAL - DO NOT DELETE
These are essential for running the project:

### Essential Folders
- `backend/` - Main backend server (port 5012)
- `frontend/` - Main frontend application (port 5010)
- `HackathonPDF/` - PDF Learning backend (port 5006)
- `knowlegde-base/` - AWS Agent knowledge base files
- `.kiro/` - Kiro IDE configuration and specs

### Essential Files in Root
- `AWS-AGENT-INSTRUCTION-UPDATED.txt` - Latest AWS agent instructions
- `JUDGE0-EC2-SETUP-GUIDE.md` - Judge0 setup documentation

---

## ✅ SAFE TO DELETE - Documentation & Temporary Files

### Root Level - Documentation Files (Safe to Delete)
These are just documentation/summary files created during development:

```
CHATBOT-FIX-COMPLETE-SUMMARY.md
FINAL-SUMMARY.md
HACKATHON-PDF-INTEGRATION-PLAN.md
PDF-INTEGRATION-COMPLETE.md
PDF-INTEGRATION-REMAINING-FILES.md
```

### Root Level - Temporary/System Files (Safe to Delete)
```
.DS_Store (Mac system file)
rishu.119484@stu.upes.ac.in_accessKeys.csv (AWS credentials - should be deleted for security!)
```

---

## ✅ SAFE TO DELETE - Backend Documentation Files

All these `.md` and `.txt` files in `backend/` are just documentation:

```
backend/AGENT-INSTRUCTION-SIMPLIFICATION.md
backend/AGENT-JSON-FORMAT-FIX.md
backend/AWS-AGENT-UPDATE-GUIDE.md
backend/AWS-CHATBOT-INSTRUCTIONS.md
backend/AWS-COPY-PASTE-INSTRUCTIONS.txt
backend/AWS-SKILL-TRACKING-SETUP.md
backend/CHATBOT-AND-FORMAT-FIX.md
backend/CHATBOT-INTELLIGENCE-FIX.md
backend/CHATBOT-NO-CODE-POLICY.md
backend/CHATBOT-SETUP-GUIDE.md
backend/CODING-AGENT-INSTRUCTION-FINAL.txt
backend/CODING-AGENT-INSTRUCTION-STRICT-JSON.txt
backend/CODING-AGENT-INSTRUCTION.txt
backend/CODING-AGENT-KB-INSTRUCTION.txt
backend/CODING-CHALLENGE-FIX.md
backend/COMPACT-FORMAT-COMPLETE.md
backend/COMPLETE-FIXES-SUMMARY.md
backend/CONTENT-GENERATION-JSON-FIX-SUMMARY.md
backend/COPY-PASTE-INSTRUCTIONS.txt
backend/CRITICAL-FIXES-APPLIED.md
backend/CURRENT-STATUS-AND-NEXT-STEPS.md
backend/DYNAMODB-MIGRATION-COMPLETE.md
backend/DYNAMODB-WORKING.md
backend/ENHANCED-CONTENT-GENERATION.md
backend/FINAL-AWS-UPDATE-INSTRUCTIONS.md
backend/FINAL-DIAGNOSIS.md
backend/FINAL-SUMMARY.md
backend/FIX-TOKEN-LIMIT-ISSUE.md
backend/IAM-PERMISSIONS-FIX.md
backend/IMPLEMENTATION-COMPLETE.md
backend/JSON-ESCAPING-FIX.md
backend/JSON-FIX-FINAL.md
backend/JSON-PARSING-FIX.md
backend/JSON-PARSING-IMPROVED.md
backend/LEARNING-ASSISTANT-CHATBOT-SETUP.md
backend/PROBLEM-VARIETY-FIX.md
backend/QUICK-ACTION-GUIDE.md
backend/QUICK-FIX-GUIDE.md
backend/QUICK-START-STRUCTURED-FORMAT.md
backend/QUICK-TEST-GUIDE.md
backend/QUIZ-CODE-SNIPPET-FIX-COMPLETE.md
backend/QUIZ-CODE-SNIPPET-FIX.md
backend/QUIZ-PARSING-FIX.md
backend/ROADMAP-CREATION-FIXES.md
backend/SETUP-CHECKLIST.md
backend/SHORT-INSTRUCTIONS.txt
backend/SKILL-CONFIDENCE-FIX.md
backend/SKILL-LEVEL-CATEGORIZATION.md
backend/SKILL-MAPPING-AGENT-INSTRUCTIONS.md
backend/SKILL-MAPPING-FIX.md
backend/SKILL-STORAGE-COMPARISON.md
backend/SKILL-TRACKING-FIX.md
backend/SMART-PREREQUISITE-RESOLUTION-COMPLETE.md
backend/STRUCTURED-TEXT-FORMAT-STATUS.md
backend/STRUCTURED-TEXT-FORMAT.md
backend/SUBMIT-BUTTON-NAN-FIX.md
backend/TEST-CASE-GENERATOR-SETUP.md
backend/TEST-CASE-SYSTEM-COMPLETE.md
backend/TUTOR-AGENT-INSTRUCTIONS-UPDATED.txt
backend/TWO-AGENT-TEST-CASE-SYSTEM.md
backend/UPDATE-CODING-AGENT-AWS.md
backend/UPDATE-KNOWLEDGE-BASE.md
```

### Backend Test Files (Safe to Delete if not testing)
```
backend/test-api.js
backend/test-dynamodb-connection.js
backend/test-full-flow.js
backend/test-judge0.js
backend/test-register.js
backend/test-request.json
backend/test-two-agent-flow.js
backend/verify-agent-setup.js
```

### Backend System Files (Safe to Delete)
```
backend/.DS_Store
backend/start-judge0.sh (Judge0 is on separate EC2 now)
```

---

## ✅ SAFE TO DELETE - Frontend Documentation Files

All these `.md` files in `frontend/` are just documentation:

```
frontend/COMPLETE-UI-ENHANCEMENTS.md
frontend/CONFIDENCE-DISPLAY-FIX.md
frontend/CREATE-BUTTON-LAG-FIX.md
frontend/DASHBOARD-UI-FIX.md
frontend/DRAWER-CHATBOT-COMPLETE.md
frontend/GRID-DEBUG.md
frontend/LANGUAGE-SELECTION-FEATURE.md
frontend/LEETCODE-EXACT-LAYOUT-COMPLETE.md
frontend/LEETCODE-LAYOUT-CHANGES.md
frontend/LEETCODE-LAYOUT-COMPLETE.md
frontend/LEETCODE-LAYOUT-IMPLEMENTATION.md
frontend/LEETCODE-LAYOUT-REDESIGN.md
frontend/PROBLEM-DISPLAY-FIX.md
frontend/PROBLEM-STATEMENT-PARSING.md
frontend/RUN-BUTTON-FIX.md
frontend/TESTCASE-FEATURE-COMPLETE.md
frontend/UI-ENHANCEMENTS.md
```

### Frontend System Files (Safe to Delete)
```
frontend/.DS_Store
```

---

## ❓ OPTIONAL TO DELETE - chatbotAWS Folder

⚠️ **UPDATE: DO NOT DELETE `chatbotAWS/backend.js`** - This is actively used!

The `chatbotAWS/` folder contains the AI chatbot backend that powers:
- **Dashboard AI Mentor** (AIMentorDrawer)
- **Sandbox AI Assistant**

### Keep These Files:
```
chatbotAWS/backend.js (ESSENTIAL - runs on port 5005)
chatbotAWS/package.json
chatbotAWS/package-lock.json
chatbotAWS/node_modules/
```

### Safe to Delete from chatbotAWS:
```
chatbotAWS/CODE_FORMATTING_GUIDE.md (documentation)
chatbotAWS/tmpclaude-801a-cwd (temp file)
chatbotAWS/tmpclaude-9889-cwd (temp file)
chatbotAWS/frontend/ (separate frontend - not used, main app has its own)
```

**Note:** The chatbot backend runs on port 5005 and is called by:
- `frontend/src/pages/AIMentorDrawer.jsx`
- `frontend/src/pages/Sandbox.jsx`

---

## 🔒 SECURITY WARNING - DELETE IMMEDIATELY

```
rishu.119484@stu.upes.ac.in_accessKeys.csv
```

This contains AWS access keys and should be deleted immediately! Never commit credentials to version control.

---

## ⚠️ DO NOT DELETE - Essential chatbotAWS Files

```
chatbotAWS/backend.js (AI chatbot backend - port 5005)
chatbotAWS/package.json
chatbotAWS/package-lock.json
chatbotAWS/node_modules/ (dependencies)
```

This backend powers:
- Dashboard AI Mentor
- Sandbox AI Assistant

---

## ⚠️ DO NOT DELETE - Essential Backend Files

```
backend/.env (contains environment variables)
backend/server.js (main server file)
backend/package.json
backend/package-lock.json
backend/jest.config.js
backend/src/ (all source code)
backend/node_modules/ (dependencies)
backend/.github/ (GitHub workflows if any)
```

---

## ⚠️ DO NOT DELETE - Essential Frontend Files

```
frontend/package.json
frontend/package-lock.json
frontend/vite.config.js
frontend/eslint.config.js
frontend/index.html
frontend/.gitignore
frontend/README.md
frontend/src/ (all source code)
frontend/public/ (static assets)
frontend/node_modules/ (dependencies)
```

---

## ⚠️ DO NOT DELETE - Essential HackathonPDF Files

```
HackathonPDF/backend/ (PDF learning backend - port 5006)
HackathonPDF/README.md
```

**Safe to Delete:**
```
HackathonPDF/frontend/ (already integrated into main frontend)
```

Note: The HackathonPDF frontend components were fully integrated into `frontend/src/pages/PDFLearningPage.jsx` and `frontend/src/components/pdf/`. The original frontend is no longer needed.

---

## ⚠️ DO NOT DELETE - Knowledge Base Files

```
knowlegde-base/canonical_skills.json
knowlegde-base/chatbot-assistant-rules.txt
knowlegde-base/coding-challenge-rules-compact.txt
knowlegde-base/difficulty-adjustment.txt
knowlegde-base/evaluation-policy.txt
knowlegde-base/learning-assistant-rules.txt
knowlegde-base/skill-mapping-rules.txt
knowlegde-base/teaching-rules.txt
knowlegde-base/test-case-generation-rules.txt
```

These are used by AWS Bedrock Agents.

---

## Cleanup Commands

### Delete All Documentation Files (Safe)
```bash
# Root level docs
rm CHATBOT-FIX-COMPLETE-SUMMARY.md
rm FINAL-SUMMARY.md
rm HACKATHON-PDF-INTEGRATION-PLAN.md
rm PDF-INTEGRATION-COMPLETE.md
rm PDF-INTEGRATION-REMAINING-FILES.md

# Backend docs
rm backend/*.md
rm backend/*.txt
rm backend/test-*.js
rm backend/verify-agent-setup.js
rm backend/start-judge0.sh

# Frontend docs
rm frontend/*.md
```

### Delete System Files (Safe)
```bash
rm .DS_Store
rm backend/.DS_Store
rm frontend/.DS_Store
rm chatbotAWS/.DS_Store
```

### Delete AWS Credentials (CRITICAL - Do this first!)
```bash
rm rishu.119484@stu.upes.ac.in_accessKeys.csv
```

### Delete chatbotAWS Documentation (Safe)
```bash
rm chatbotAWS/CODE_FORMATTING_GUIDE.md
rm chatbotAWS/tmpclaude-*
rm -rf chatbotAWS/frontend/
```

Note: Keep `chatbotAWS/backend.js` - it's essential for AI chatbot functionality!

### Delete HackathonPDF Frontend (Safe)
```bash
rm -rf HackathonPDF/frontend/
```

Note: The frontend was fully integrated into main `frontend/` folder. Only the backend (port 5006) is needed.

---

## Summary

### Total Deletable Files:
- **Root**: 5 documentation files + 2 system files
- **Backend**: ~60 documentation files + 7 test files + 1 system file
- **Frontend**: ~15 documentation files + 1 system file
- **chatbotAWS**: 1 documentation file + 2 temp files + frontend folder
- **HackathonPDF**: frontend folder (already integrated into main app)

### Estimated Space Saved:
- Documentation files: ~5-10 MB
- chatbotAWS/frontend: ~50-100 MB
- HackathonPDF/frontend: ~100-200 MB
- System files: <1 MB
- **Total: ~200-350 MB**

### After Cleanup, Essential Structure:
```
project/
├── .kiro/                    # Kiro IDE config
├── .vscode/                  # VS Code settings
├── backend/                  # Main backend (port 5012)
│   ├── src/
│   ├── node_modules/
│   ├── .env
│   ├── server.js
│   └── package.json
├── chatbotAWS/              # AI Chatbot backend (port 5005)
│   ├── backend.js
│   ├── node_modules/
│   └── package.json
├── frontend/                 # Main frontend (port 5010)
│   ├── src/
│   ├── public/
│   ├── node_modules/
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── HackathonPDF/            # PDF Learning backend (port 5006)
│   ├── backend/
│   └── README.md
├── knowlegde-base/          # AWS Agent knowledge base
└── AWS-AGENT-INSTRUCTION-UPDATED.txt
```

---

## Recommendation

1. **Delete immediately**: AWS credentials file (`rishu.119484@stu.upes.ac.in_accessKeys.csv`)
2. **Safe to delete**: All documentation `.md` and `.txt` files
3. **Safe to delete**: All test files (if not actively testing)
4. **Safe to delete**: All `.DS_Store` files
5. **Safe to delete**: `chatbotAWS/frontend/` folder and temp files
6. **Safe to delete**: `HackathonPDF/frontend/` folder (already integrated)
7. **Keep**: `chatbotAWS/backend.js` - Essential for AI chatbot (Dashboard & Sandbox)
8. **Keep**: `HackathonPDF/backend/` - Essential for PDF Learning feature
9. **Keep**: All source code, config files, and dependencies

## Running Servers After Cleanup

Your project requires 4 servers to run:

1. **Main Backend** (port 5012)
   ```bash
   cd backend
   node server.js
   ```

2. **AI Chatbot Backend** (port 5005)
   ```bash
   cd chatbotAWS
   node backend.js
   ```

3. **PDF Learning Backend** (port 5006)
   ```bash
   cd HackathonPDF/backend
   npm run dev
   ```

4. **Frontend** (port 5010)
   ```bash
   cd frontend
   npm run dev
   ```

This will clean up your project significantly while keeping all essential functionality intact.
