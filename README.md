# BankAI — AI-Powered Bank Statement Analyzer

A production-oriented full-stack web application that transforms unstructured bank statements (PDF, JPG, JPEG, PNG) into verified, structured financial intelligence, automated anomaly detection, AI-powered conversational assistance, and downloadable CSV/ZIP exports.

---

## 🌟 Key Capabilities

1. **Multi-Format Document Pipeline**:
   - Accepts **PDF, JPG, JPEG, and PNG** bank statements (up to 15MB).
   - High-performance text extraction using **PDF.js** for native digital PDFs.
   - Intelligent OCR using **Tesseract.js** paired with **Sharp** image preprocessing (adaptive thresholding, binarization, auto-rotation, and noise reduction) for scanned receipts and photos.

2. **AI & Deterministic Structured Extraction**:
   - Structured JSON schema validation using **Zod**.
   - Extracts complete account details: Bank Name, Account Holder, Masked Account Number (`XXXXXXXX1234`), IFSC Code, Account Type, Branch, Statement Dates, Balances, and Currency.
   - Extracts complete transaction records: Date, Value Date, Description, Reference/UTR, Debit Amount, Credit Amount, Running Balance, and Type.
   - Zero-hallucination policy: Missing or ambiguous values are flagged as `null` with preserved OCR source evidence.

3. **Mathematical Consistency & Balance Verification**:
   - Deterministic verification engine:
     $$\text{Calculated Closing} = \text{Opening Balance} + \sum \text{Credits} - \sum \text{Debits}$$
   - Validates running balances across each row and flags discrepancies before persistence.

4. **20-Category Financial Classification**:
   - Categorizes each transaction with confidence scores and explainable rationale:
     `UPI`, `NEFT`, `RTGS`, `IMPS`, `ATM Withdrawal`, `Cash Deposit`, `Cash Withdrawal`, `Bank Transfer`, `Cheque`, `Debit Card`, `Credit Card`, `Salary`, `Interest`, `Bank Charges`, `Shopping`, `Food`, `Travel`, `Bills`, `Other`, `Unknown`.

5. **Explainable Anomaly Detection**:
   - Identifies statistical spending outliers (> 2.5 std deviations from mean debit).
   - Detects potential duplicate charges (identical date, amount, and narration).
   - Flags sudden transaction velocity bursts and large round-sum transfers with human-readable explanations.

6. **AI Conversational Assistant (Grounded RAG)**:
   - Natural language chatbot powered by Google Gemini and backed by authoritative MongoDB indexes.
   - Can answer questions like:
     - *"What is my total spending?"*
     - *"Show my UPI transactions"*
     - *"How much money did I spend on shopping?"*
     - *"What is my highest transaction?"*
     - *"Show transactions above ₹10,000"*
     - *"Find unusual transactions"*

7. **Fintech Dashboard & Analytics**:
   - Real-time KPI stat cards (Total Balance, Income, Spending, Velocity).
   - Interactive Recharts visualizations: Category Spending Donut Chart, Income vs. Spending Area Chart, Weekly Spending Volatility Bar Chart, and Monthly Summary.
   - Executive AI Insights Card: Financial Health Score (0-100), key observations, spending habits, and actionable savings advice.

8. **RFC 4180 CSV & ZIP Exports**:
   - Direct download of `account_data.csv`.
   - Direct download of `transactions.csv` (with preserved precision, UTRs, and anomaly flags).
   - Full bundle download as `bank_statement_export.zip` (includes summary JSON).

9. **Security & Privacy**:
   - JWT Bearer Authentication with 7-day expiry.
   - Bcrypt password hashing (salt rounds: 10).
   - 256-bit masked account representation in UI and exports.
   - User-scoped multi-tenant database isolation.
   - Opt-in AI processing consent toggle.

---

## 🏗️ Project Architecture

```
Bank OCR/
├── backend/
│   ├── config/
│   │   ├── env.js                     # Environment configuration
│   │   └── db.js                      # MongoDB connection + in-process MongoMemoryServer fallback
│   ├── controllers/
│   │   ├── authController.js          # Authentication (register, login, me, consent)
│   │   ├── documentController.js      # Statement upload, listing, reprocessing, delete
│   │   ├── transactionController.js   # Transactions listing, search, category update
│   │   ├── analyticsController.js     # Deterministic metrics & AI insights
│   │   ├── chatController.js          # RAG query processing & chat history
│   │   └── exportController.js        # CSV & ZIP streaming downloads
│   ├── middleware/
│   │   ├── authMiddleware.js          # JWT authentication guard
│   │   ├── uploadMiddleware.js        # Multer file validation (PDF, JPG, JPEG, PNG, 15MB)
│   │   ├── errorMiddleware.js         # Global Zod/Multer/Mongoose error handler
│   │   └── validationMiddleware.js    # Schema validation helper
│   ├── models/
│   │   ├── User.js                    # User account schema with bcrypt
│   │   ├── Document.js                # Document tracking and OCR metadata
│   │   ├── Account.js                 # Extracted bank account entity
│   │   ├── Transaction.js             # Categorized transactions with anomaly flags
│   │   ├── Analytics.js               # Cached metrics, timelines & AI narrative
│   │   └── ChatMessage.js             # Grounded chat message history
│   ├── routes/                        # Express API route declarations
│   ├── services/
│   │   ├── ocrService.js              # Tesseract.js worker execution
│   │   ├── pdfService.js              # PDF.js text extraction with resilient stream recovery
│   │   ├── imageService.js            # Sharp preprocessing (grayscale, normalize, threshold)
│   │   ├── aiExtractionService.js     # Gemini API structured JSON parser & deterministic fallback
│   │   ├── transactionService.js      # Full pipeline orchestration
│   │   ├── classificationService.js   # 20 standard transaction category rules
│   │   ├── analyticsService.js        # Deterministic math engine + AI insights
│   │   ├── anomalyDetectionService.js # Statistical and duplicate anomaly engine
│   │   ├── chatbotService.js          # MongoDB context retriever & prompt grounding
│   │   └── csvService.js              # RFC 4180 CSV generator and ZIP archiver
│   ├── utils/                         # Account masking, math validation & response helpers
│   ├── samples/                       # Sample PDF and PNG statements for testing
│   ├── scripts/                       # Automated pipeline and image OCR test scripts
│   └── server.js                      # Express server entry point
│
├── frontend/
│   ├── src/
│   │   ├── components/                # Navbar, Sidebar, StatCard, FileUploadZone, TransactionTable,
│   │   │                              # Charts (Spending, Cashflow, Weekly, Monthly), ChatWidget, ExportModal
│   │   ├── context/                   # AuthContext, DocumentContext
│   │   ├── layouts/                   # AppLayout, AuthLayout
│   │   ├── pages/                     # Dashboard, Upload, Transactions, Analytics, Chat, Settings, Login, Register
│   │   ├── services/                  # Axios API clients
│   │   └── utils/                     # Formatters, constants, category colors
│   ├── vite.config.js
│   └── tailwind.config.js
└── README.md
```

---

## 🚀 Quick Start Guide

### 1. Prerequisites
- **Node.js** v18+ (tested on Node v24)
- **npm** v9+
- *(Optional)* Local MongoDB instance or MongoDB Atlas. If none is running, the backend **automatically initializes an embedded MongoMemoryServer** so that the application runs with zero configuration!
- *(Optional)* Google Gemini API Key. If not supplied, the application automatically uses its high-accuracy deterministic banking parser!

### 2. Environment Setup

Create `backend/.env`:
```env
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb://127.0.0.1:27017/bank_ocr
JWT_SECRET=your_production_secret_key_here
JWT_EXPIRES_IN=7d
GEMINI_API_KEY=your_gemini_api_key_here
MAX_FILE_SIZE_MB=15
CORS_ORIGIN=http://localhost:5173
```

### 3. Installation

Install backend dependencies:
```bash
cd backend
npm install
```

Install frontend dependencies:
```bash
cd ../frontend
npm install
npm run build
```

### 4. Running the Application

#### Option A: Unified Full-Stack Server (Recommended)
You can run the full application (frontend + backend + auto-embedded database) from port 5000:
```bash
cd backend
npm start
```
Then open your browser at **`http://localhost:5000`**.

#### Option B: Independent Vite Dev Server
Run backend:
```bash
cd backend
npm run dev
```
In another terminal, run frontend:
```bash
cd frontend
npm run dev
```
Then open your browser at **`http://localhost:5173`**.

---

## 🔑 Demo Account Credentials

A verified demo account is automatically seeded on first launch:
- **Email**: `fintech_demo@bankai.org`
- **Password**: `DemoFintech2026!`

*(You can also click the **"Auto-fill Demo Account"** button on the sign-in screen to instantly fill these credentials).*

---

## 📡 REST API Reference

| Method | Endpoint | Description | Auth Required |
|---|---|---|---|
| `POST` | `/api/auth/register` | Register new user | No |
| `POST` | `/api/auth/login` | Authenticate user & get JWT | No |
| `GET` | `/api/auth/me` | Current user profile | Bearer Token |
| `PATCH`| `/api/auth/consent` | Update AI processing consent | Bearer Token |
| `POST` | `/api/documents/upload` | Upload PDF/JPG/PNG statement | Bearer Token |
| `GET` | `/api/documents` | List uploaded statements | Bearer Token |
| `GET` | `/api/documents/:id` | Get document metadata & account | Bearer Token |
| `POST` | `/api/documents/:id/reprocess` | Re-run OCR & parsing | Bearer Token |
| `DELETE`| `/api/documents/:id` | Delete document & all txns | Bearer Token |
| `GET` | `/api/transactions` | Query & filter transactions | Bearer Token |
| `PATCH`| `/api/transactions/:id/category` | Manually reclassify category | Bearer Token |
| `GET` | `/api/analytics/:documentId` | Statement financial metrics & AI insights | Bearer Token |
| `POST` | `/api/analytics/:documentId/refresh-insights` | Refresh Gemini AI insights | Bearer Token |
| `POST` | `/api/chat/query` | Grounded RAG query to assistant | Bearer Token |
| `GET` | `/api/chat/history/:documentId` | Statement chat history | Bearer Token |
| `DELETE`| `/api/chat/history/:documentId` | Clear statement chat history | Bearer Token |
| `GET` | `/api/export/:documentId/account-csv` | Download `account_data.csv` | Bearer Token |
| `GET` | `/api/export/:documentId/transactions-csv` | Download `transactions.csv` | Bearer Token |
| `GET` | `/api/export/:documentId/zip` | Download complete `export.zip` | Bearer Token |

---

## 🧪 Automated Verification Suite

Run the full end-to-end pipeline test:
```bash
cd backend
node scripts/testPipeline.js
```

Run the Image OCR test:
```bash
cd backend
node scripts/testImageOcr.js
```

Probe live HTTP endpoints:
```bash
cd backend
node scripts/probeServer.js
```

---

## 🛡️ Financial Integrity & Privacy Safeguards

- **No Financial Hallucinations**: Strict prompt guidelines instruct the LLM never to invent figures. When an amount or reference is unreadable, it is assigned `null`.
- **Mathematical Balance Cross-Check**: Computed credits and debits must reconcile against the opening and closing balance:
  $$\Delta = |\text{Calculated Closing} - \text{Stated Closing}| \le 0.05$$
- **Account Number Masking**: Account numbers are presented as `XXXXXXXX9012` to prevent disclosure.
- **Tenant Isolation**: All transaction, document, and analytics queries strictly scope to `req.user._id`.

