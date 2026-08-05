# VentureIQ — Faculty Presentation Guide
### AI-Powered Startup Intelligence & Entrepreneurial Decision Support Platform
### Team: Parshva Shah, Jinang Shah, Jay Raval | LJ University

---

## TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Technology Stack](#3-technology-stack)
4. [System Architecture](#4-system-architecture)
5. [Project Folder Structure — File by File](#5-project-folder-structure--file-by-file)
6. [Backend — Django Apps Explained](#6-backend--django-apps-explained)
7. [Frontend — Pages Explained](#7-frontend--pages-explained)
8. [Database Design](#8-database-design)
9. [AI / ML Pipeline — 18 Stages](#9-ai--ml-pipeline--18-stages)
10. [Security Implementation](#10-security-implementation)
11. [API Endpoints Reference](#11-api-endpoints-reference)
12. [Deployment Architecture](#12-deployment-architecture)
13. [Faculty Questions & Answers](#13-faculty-questions--answers)

---

## 1. PROJECT OVERVIEW

**VentureIQ** is a full-stack, AI/ML-powered web application that evaluates startup ideas automatically. A founder types a startup idea description, clicks submit, and within minutes receives a complete intelligence report including:

- Innovation score with 5 sub-dimensions
- Market size and competition analysis
- Risk assessment across 5 categories
- SWOT analysis
- Competitor mapping
- Customer personas
- Business Model Canvas
- Team recommendations
- Investor readiness score
- 4-phase development roadmap
- AI mentor chatbot (powered by Google Gemini)
- Downloadable PDF report

**Who uses it?**
- Founders / Students — submit ideas, get analysis
- Admins — manage platform, configure ML weights, view audit logs

**Deployed on:**
- Frontend → Vercel (React SPA)
- Backend API → Render (Django + Gunicorn)
- Database → Render managed PostgreSQL
- Cache + Queue broker → Render managed Redis

---

## 2. PROBLEM STATEMENT

Traditional startup idea evaluation requires:
- Hiring consultants (expensive)
- Weeks of manual research
- Subjective human judgment

VentureIQ solves this by providing an automated, objective, ML-powered evaluation in minutes — accessible to any student or early-stage founder.

---

## 3. TECHNOLOGY STACK

### Backend (Python)
| Technology | Version | Why We Used It |
|-----------|---------|----------------|
| Python | 3.11+ | Language of choice for ML/AI integration |
| Django | 4.2.7 | Mature web framework with ORM and admin panel |
| Django REST Framework | 3.14.0 | RESTful API with serializers and viewsets |
| djangorestframework-simplejwt | 5.3.1 | JWT token-based authentication |
| Celery | 5.3.6 | Async task queue for the 18-stage pipeline |
| Redis | 7.0 | Message broker for Celery + result caching |
| PostgreSQL | 15 | Primary relational database |
| scikit-learn | 1.3.2 | ML models — GradientBoostingRegressor |
| spaCy | 3.7.2 | NLP — tokenization, lemmatization, POS tagging |
| NLTK | 3.8.1 | Text preprocessing |
| pandas / numpy | 2.1.4 / 1.26.2 | Data manipulation and feature engineering |
| WeasyPrint | 60.2 | PDF report generation from HTML templates |
| Google Gemini 2.0 Flash API | — | AI Mentor chatbot responses |
| Gunicorn | 21.2.0 | Production WSGI server |
| WhiteNoise | 6.6.0 | Serve static files in production |

### Frontend (JavaScript)
| Technology | Version | Why We Used It |
|-----------|---------|----------------|
| React | 18.3.1 | Component-based UI with hooks |
| Vite | 6.3.5 | Fast dev server and build tool |
| Tailwind CSS | 3.4.17 | Utility-first CSS — faster styling, responsive |
| React Router | 6.30.1 | Client-side routing (SPA navigation) |
| TanStack React Query | 5.83.0 | Server state caching and data fetching |
| Axios | 1.9.0 | HTTP client for API calls |
| Recharts | 2.15.3 | Data visualization charts |
| Headless UI | 2.2.4 | Accessible UI components (modals, dropdowns) |

---

## 4. SYSTEM ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────────┐
│                 FRONTEND  (React 18 + Vite + Tailwind)               │
│  User opens browser → Vercel delivers the React SPA                  │
│  React Router handles page navigation without page reload            │
│  Axios sends HTTP requests to Django backend                         │
│  React Query caches responses and manages loading/error states       │
└──────────────────────────┬──────────────────────────────────────────┘
                           │  REST API calls (JSON over HTTPS)
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│              BACKEND  (Django 4.2 + DRF on Render)                   │
│  JWT Auth → RBAC Middleware → API Views → Serializers → ORM → DB    │
│  13 Django Apps handle different domains                             │
└────────────┬────────────────────────────────────┬───────────────────┘
             │  Celery task dispatch               │  DB queries
             ▼                                    ▼
┌────────────────────────┐             ┌──────────────────────────────┐
│  Redis (Broker+Cache)  │             │  PostgreSQL Database (38+ tables)│
│  Task queue for AI     │             │  Users, Ideas, Scores, Chat, │
│  analysis pipeline     │             │  Reports, Analytics, Audit   │
└────────────┬───────────┘             └──────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CELERY WORKER (AI/ML Engine)                     │
│  Stage 1: Load Idea → Stage 2: NLP → Stage 3: Feature Extract →     │
│  Stage 4: Industry Classify → Stage 5: Competitor Search →          │
│  Stage 6-10: ML Scoring → Stage 11-15: Content Generation →         │
│  Stage 16-18: Save → Status Update → Notify Founder                 │
└─────────────────────────────────────────────────────────────────────┘
```

**Key Design Decisions:**
- Celery separates the heavy AI/ML work from the web request cycle (non-blocking)
- Redis is dual-purpose: Celery broker AND application cache
- PostgreSQL uses UUID primary keys for security (no guessable integer IDs)
- Soft-delete pattern: records are never physically deleted

---

## 5. PROJECT FOLDER STRUCTURE — FILE BY FILE

```
VentureIQ/
├── backend/                      ← Django project root
│   ├── manage.py                 ← Django CLI entry point (run server, migrations)
│   ├── requirements.txt          ← All Python dependencies with pinned versions
│   ├── Procfile                  ← Tells Render/Heroku how to start the app
│   ├── render.yaml               ← Infrastructure-as-code for Render deployment
│   ├── pytest.ini                ← Test configuration (pytest settings)
│   ├── .env                      ← Local environment variables (never committed)
│   ├── .env.example              ← Template showing required env variable names
│   │
│   ├── ventureiq/                ← Core Django project package
│   │   ├── settings/
│   │   │   ├── base.py           ← Shared settings (DB, DRF, JWT, Celery, etc.)
│   │   │   ├── dev.py            ← Development overrides (DEBUG=True, SQLite option)
│   │   │   └── prod.py           ← Production overrides (HTTPS, HSTS, S3, etc.)
│   │   ├── urls.py               ← Root URL router (registers all 13 app URL modules)
│   │   ├── celery.py             ← Celery app setup and task autodiscovery
│   │   ├── wsgi.py               ← WSGI entry for Gunicorn (production)
│   │   └── asgi.py               ← ASGI entry (future WebSocket support)
│   │
│   ├── accounts/                 ← User auth, JWT, RBAC
│   │   ├── models.py             ← CustomUser, RefreshToken, LoginAttempt, AuditLog
│   │   ├── views.py              ← Register, Login, Logout, Profile, ChangePassword
│   │   ├── serializers.py        ← Input/output validation for auth endpoints
│   │   ├── permissions.py        ← Custom DRF permission classes (IsAdmin, IsFounder)
│   │   ├── middleware.py         ← RateLimitMiddleware, LoginAttemptMiddleware, AuditLogMiddleware
│   │   ├── managers.py           ← CustomUserManager (email-based create_user)
│   │   ├── tokens.py             ← Password reset token generator (SHA-256)
│   │   └── urls.py               ← Auth URL patterns
│   │
│   ├── ideas/                    ← Startup idea CRUD
│   │   ├── models.py             ← Idea model (UUID PK, soft-delete, status machine)
│   │   ├── views.py              ← List, Create, Detail, Update, Delete, TriggerAnalysis
│   │   ├── serializers.py        ← IdeaSerializer with validation
│   │   └── urls.py               ← Idea URL patterns
│   │
│   ├── analysis/                 ← Pipeline models + Celery tasks
│   │   ├── models.py             ← NLPOutput, InnovationScore, MarketIntelligence,
│   │   │                           RiskAssessment, SWOT, Recommendation, Persona,
│   │   │                           Canvas, TeamRecommendation, InvestorReadiness, Roadmap
│   │   ├── tasks.py              ← 18-stage Celery pipeline (run_analysis_pipeline)
│   │   ├── views.py              ← AnalysisDetailView (returns all analysis results)
│   │   └── urls.py               ← /api/v1/analysis/<idea_id>/
│   │
│   ├── nlp_engine/               ← NLP processing
│   │   └── pipeline.py           ← Tokenize → lemmatize → TF-IDF → keywords → classify
│   │
│   ├── scoring/                  ← ML scoring modules
│   │   ├── innovation.py         ← InnovationScoreEngine (5 sub-scores)
│   │   ├── market.py             ← MarketIntelligenceModule
│   │   ├── risk.py               ← RiskAssessmentEngine
│   │   └── investor.py           ← InvestorReadinessModule
│   │
│   ├── ml_models/                ← Trained model files
│   │   ├── novelty_model.pkl
│   │   ├── practicality_model.pkl
│   │   ├── scalability_model.pkl
│   │   ├── business_value_model.pkl
│   │   ├── tech_adoption_model.pkl
│   │   ├── market_size_model.pkl
│   │   ├── growth_rate_model.pkl
│   │   ├── competition_model.pkl
│   │   ├── audience_fit_model.pkl
│   │   ├── timing_model.pkl
│   │   ├── investor_readiness_model.pkl
│   │   └── overall_risk_model.pkl
│   │
│   ├── generators/               ← Content generation modules
│   │   ├── swot.py               ← SWOT generator (rule-based + ML)
│   │   ├── recommendations.py    ← Revenue model, marketing, improvement tips
│   │   ├── personas.py           ← Customer persona generator
│   │   ├── canvas.py             ← Business Model Canvas (9 blocks)
│   │   ├── team.py               ← Team role recommender
│   │   └── roadmap.py            ← 4-phase roadmap generator
│   │
│   ├── competitor/               ← Competitor analysis
│   │   ├── models.py             ← CompetitorProfile, CompetitorResult
│   │   ├── analyzer.py           ← TF-IDF cosine similarity matching
│   │   └── fixtures/competitors.json ← 50+ competitor profiles dataset
│   │
│   ├── mentor_chat/              ← AI mentor chatbot
│   │   ├── models.py             ← ChatSession (UUID), ChatMessage
│   │   ├── views.py              ← SessionListCreate, MessageListCreate (calls Gemini)
│   │   └── gemini_client.py      ← Google Gemini API wrapper
│   │
│   ├── reports/                  ← PDF generation
│   │   ├── models.py             ← Report (status, file path)
│   │   ├── tasks.py              ← Celery task: generate PDF via WeasyPrint
│   │   └── templates/            ← HTML templates for PDF
│   │
│   ├── analytics/                ← Platform analytics
│   │   └── views.py              ← Aggregated stats (total ideas, score averages, trends)
│   │
│   ├── admin_panel/              ← Admin governance APIs
│   │   └── views.py              ← Users, Categories, ScoreWeights, AuditLog, ML Models
│   │
│   └── notifications/            ← User notification system
│       ├── models.py             ← Notification (user, message, is_read, type)
│       └── views.py              ← List, mark-read endpoints
```

---

### Frontend Folder Structure

```
frontend/
├── index.html                    ← Single HTML file — React mounts here
├── vite.config.js                ← Build config, dev proxy to Django backend
├── tailwind.config.js            ← Tailwind theme customization
├── postcss.config.js             ← CSS processing plugins
├── vercel.json                   ← Vercel SPA routing config (all routes → index.html)
├── package.json                  ← npm dependencies and scripts
│
└── src/
    ├── main.jsx                  ← App entry point (ReactDOM.render, providers)
    ├── App.jsx                   ← Router setup, protected route wrappers
    ├── index.css                 ← Global styles + Tailwind base imports
    │
    ├── api/                      ← Axios API client modules
    │   ├── axiosInstance.js      ← Base Axios config (base URL, interceptors, JWT attach)
    │   ├── authApi.js            ← Login, register, logout, profile calls
    │   ├── ideasApi.js           ← CRUD + trigger analysis
    │   ├── analysisApi.js        ← Fetch analysis results
    │   ├── mentorApi.js          ← Chat session + message calls
    │   └── reportsApi.js         ← Generate + download PDF
    │
    ├── context/                  ← React Context providers
    │   └── AuthContext.jsx       ← Current user state, login/logout helpers
    │
    ├── hooks/                    ← Custom React hooks
    │   ├── useAuth.js            ← Access auth context
    │   └── useIdeas.js           ← React Query hooks for idea fetching
    │
    ├── components/               ← Reusable UI components
    │   ├── Navbar.jsx
    │   ├── Sidebar.jsx
    │   ├── ProtectedRoute.jsx    ← Redirect to login if not authenticated
    │   ├── LoadingSpinner.jsx
    │   ├── ScoreCard.jsx
    │   ├── ChartComponents/      ← Recharts wrappers (radar, bar, funnel, etc.)
    │   └── NotificationPanel.jsx
    │
    ├── pages/                    ← One file per route/screen
    │   ├── LandingPage.jsx       ← Public homepage
    │   ├── LoginPage.jsx
    │   ├── RegisterPage.jsx
    │   ├── ForgotPasswordPage.jsx
    │   ├── ResetPasswordPage.jsx
    │   ├── DashboardPage.jsx
    │   ├── IdeaSubmissionPage.jsx
    │   ├── IdeaListPage.jsx
    │   ├── IdeaDetailPage.jsx
    │   ├── AnalysisTabsPage.jsx  ← 13-tab analysis view
    │   ├── ComparisonPage.jsx
    │   ├── MentorChatPage.jsx
    │   ├── ReportsPage.jsx
    │   ├── AnalyticsDashboardPage.jsx
    │   ├── ProfilePage.jsx
    │   ├── SubmissionHistoryPage.jsx
    │   ├── PitchDeckEditorPage.jsx
    │   └── AdminPanelPage.jsx
    │
    └── utils/                    ← Helper utilities
        ├── formatters.js         ← Number, date, score formatting
        └── validators.js         ← Frontend input validation helpers
```

---

## 6. BACKEND — DJANGO APPS EXPLAINED

### App 1: `accounts/` — Authentication & User Management

**Purpose:** Handles everything related to users — registration, login, JWT tokens, passwords, roles, and audit logging.

**Key files:**
- `models.py` — Defines 5 models:
  - `CustomUser` — UUID primary key, email-based login (not username), role field (founder/admin/mentor/student/incubation), soft-delete flags
  - `RefreshToken` — stores SHA-256 hash of refresh tokens (never the raw token) for server-side revocation
  - `LoginAttempt` — logs every login try per IP for brute-force protection
  - `PasswordResetToken` — one-time token for password reset (SHA-256 hash stored)
  - `AuditLog` — immutable record of every auth event and admin action

- `views.py` — 8 API views:
  - `RegisterView` — creates user, returns access + refresh JWT
  - `LoginView` — verifies password, returns tokens
  - `LogoutView` — revokes refresh token
  - `TokenRefreshView` — issues new token pair
  - `ForgotPasswordView` — sends reset email (always returns 200 to prevent email enumeration)
  - `ResetPasswordView` — validates token, sets new password
  - `ProfileView` — GET/PATCH current user profile
  - `ChangePasswordView` — requires current password before changing

- `middleware.py` — 3 middleware classes:
  - `RateLimitMiddleware` — throttles too-frequent requests
  - `LoginAttemptMiddleware` — blocks IPs after 5 failed login attempts
  - `AuditLogMiddleware` — automatically logs every request/response

- `permissions.py` — Custom DRF permission classes that check user role (IsAdmin, IsFounder)

**JWT Flow:**
```
Register/Login → { access_token (60 min), refresh_token (7 days) }
Every API call → Authorization: Bearer <access_token>
Token expires → POST /token/refresh/ with refresh_token → new access_token
Logout → refresh_token is blacklisted/revoked
```

---

### App 2: `ideas/` — Startup Idea Management

**Purpose:** Core entity — the startup idea a founder submits.

**Key model fields (`Idea`):**
- `id` — UUID (not integer), prevents ID enumeration attacks
- `founder` — FK to CustomUser
- `startup_name`, `description`, `target_audience`, `tagline`
- `industry` — FK to StartupCategory
- `budget_range`, `business_model`, `development_stage`, `team_size`
- `analysis_status` — state machine: `pending → processing → classified → market_analyzed → scored → complete → failed`
- `is_deleted` — soft-delete flag
- `domain_data` — JSON field for industry-specific extra inputs

**Status Machine:**
```
Founder submits idea
       ↓
   [PENDING]
       ↓  (Celery task starts)
  [PROCESSING]
       ↓  (NLP done)
  [CLASSIFIED]
       ↓  (Market analysis done)
[MARKET_ANALYZED]
       ↓  (ML scoring done)
   [SCORED]
       ↓  (All stages complete)
  [COMPLETE]  ←→  [FAILED] (on error)
```

---

### App 3: `analysis/` — Pipeline Models & Celery Tasks

**Purpose:** Stores all AI analysis outputs and orchestrates the 18-stage pipeline.

**`tasks.py` — Celery task `run_analysis_pipeline(idea_id)`:**
- This is the heart of the system
- Decorated with `@shared_task(bind=True, max_retries=3)`
- Uses exponential backoff: retries after 10s, 20s, 40s on failure
- Calls NLP engine, ML scoring, all generators in sequence
- Saves each output to corresponding model
- Updates `Idea.analysis_status` at each stage

**Models in analysis app (12 output tables):**
- `NLPOutput` — keywords, tokens, industry classification
- `InnovationScore` — 5 sub-scores + composite score
- `MarketIntelligence` — market_size, growth_rate, competition, audience_fit, timing
- `RiskAssessment` — 5 risk dimensions + mitigation advice
- `SWOTAnalysis` — strengths/weaknesses/opportunities/threats (JSON arrays)
- `Recommendation` — revenue models, marketing tactics, improvements
- `CustomerPersona` — 3 personas with demographics
- `BusinessModelCanvas` — 9 blocks (key partners, activities, value prop, etc.)
- `TeamRecommendation` — required roles
- `InvestorReadiness` — score + missing requirements
- `RoadmapPlan` — 4 development phases with milestones

---

### App 4: `nlp_engine/` — Natural Language Processing

**Purpose:** Processes raw text of the startup description.

**Pipeline steps:**
1. Text cleaning (lowercase, remove special chars)
2. Tokenization (split into words)
3. Lemmatization with spaCy (runs → run, better → good)
4. POS tagging (noun, verb, adjective detection)
5. Stop-word removal
6. TF-IDF vectorization (Term Frequency-Inverse Document Frequency)
7. Keyword extraction (top-N most important terms)
8. Industry classification (maps keywords to 16 industry categories)
9. Technology label detection (AI, blockchain, IoT, etc.)
10. Quality check (warns if description < 50 words)

---

### App 5: `scoring/` — ML Scoring Engines

**Purpose:** Generates numeric scores using trained GradientBoosting ML models.

**Four scoring modules:**

1. **InnovationScoreEngine**
   - Inputs: NLP features (keyword count, TF-IDF scores, industry encoding, development stage, team size)
   - Outputs: novelty (0-100), practicality (0-100), scalability (0-100), business_value (0-100), tech_adoption (0-100)
   - Composite score = weighted average of 5 sub-scores

2. **MarketIntelligenceModule**
   - Outputs: market_size (0-100), growth_rate (0-100), competition_level (low/medium/high), audience_fit (0-100), market_timing (0-100)

3. **RiskAssessmentEngine**
   - Outputs: financial_risk, technical_risk, operational_risk, market_risk, legal_risk (each 0-100)
   - Generates mitigation suggestions for each high-risk area

4. **InvestorReadinessModule**
   - Outputs: investor_readiness_score (0-100)
   - Lists missing requirements (e.g., "No prototype mentioned", "Team size too small")

**Fallback:** If `.pkl` model files are missing/corrupt, the system falls back to rule-based heuristic scoring automatically.

---

### App 6: `ml_models/` — Trained ML Model Files

**Purpose:** Stores 12 serialized (`.pkl`) GradientBoosting models.

**Algorithm:** GradientBoostingRegressor from scikit-learn

**Why GradientBoosting?**
- Handles non-linear relationships in startup data
- Robust to outliers compared to linear regression
- Ensemble method (combines many weak decision trees) = better accuracy
- Built-in feature importance for explainability

**Training process:**
- Training dataset: synthetic startup evaluation data
- Features extracted: keyword_count, description_length, tfidf_score, industry_code, development_stage_code, team_size, business_model_code
- Cross-validation used to prevent overfitting
- Models serialized with `pickle`/`joblib` for reuse without retraining

**12 trained models:**
`novelty`, `practicality`, `scalability`, `business_value`, `tech_adoption`, `market_size`, `growth_rate`, `competition`, `audience_fit`, `timing`, `investor_readiness`, `overall_risk`

---

### App 7: `generators/` — Content Generation

**Purpose:** Generates human-readable content (not just numbers).

| Module | What it generates |
|--------|------------------|
| `swot.py` | 4 SWOT lists based on scores + industry |
| `recommendations.py` | Revenue model suggestions, marketing channels, improvement tips |
| `personas.py` | 3 customer personas (name, age, job, pain points, goals) |
| `canvas.py` | 9-block Business Model Canvas |
| `team.py` | Required team roles (CTO, Marketing Lead, etc.) |
| `roadmap.py` | 4-phase plan (Phase 1: Validation, Phase 2: MVP, Phase 3: Growth, Phase 4: Scale) |

---

### App 8: `competitor/` — Competitor Analysis

**Purpose:** Finds similar existing startups/companies from a curated dataset.

**How it works (TF-IDF Cosine Similarity):**
1. Load `competitors.json` (50+ competitor profiles, each with description text)
2. Build a TF-IDF matrix of all competitor descriptions
3. Convert the user's idea description to a TF-IDF vector
4. Calculate cosine similarity between the idea and each competitor
5. Return top-N most similar competitors with similarity scores
6. Classify competition level: Low (<0.3), Medium (0.3-0.6), High (>0.6)

---

### App 9: `mentor_chat/` — AI Mentor Chatbot

**Purpose:** Conversational AI that answers questions about the startup idea.

**How it works:**
1. Founder opens chat on their analyzed idea's page
2. System creates a `ChatSession` (UUID-based)
3. On each message, the backend:
   - Loads last 10 messages as conversation history
   - Injects full startup analysis data as context
   - Calls Google Gemini 2.0 Flash API
   - Saves AI response as a `ChatMessage`
4. If Gemini API fails → falls back to rule-based responses

**Context injected to Gemini:**
- Startup name and description
- Innovation scores, market scores, risk scores
- SWOT analysis
- Competitor results
- Investor readiness score

---

### App 10: `reports/` — PDF Generation

**Purpose:** Creates a downloadable PDF with the full analysis.

**Process:**
1. Celery task triggered by founder clicking "Generate Report"
2. Django renders an HTML template with all analysis data
3. WeasyPrint converts HTML → PDF (supports CSS, charts)
4. PDF saved to `media/reports/` folder
5. Download URL returned to frontend

---

### App 11: `analytics/` — Platform Analytics

**Purpose:** Admin dashboard showing platform-wide statistics.

**Data provided:**
- Total ideas submitted, total users, analyses completed
- Industry distribution (FinTech 30%, HealthTech 20%, etc.)
- Monthly activity trends
- Average scores by dimension
- Top ideas leaderboard
- Risk distribution (Low/Medium/High)

---

### App 12: `admin_panel/` — Administrative APIs

**Purpose:** Backend endpoints exclusively for admin users.

**Endpoints:**
- User management (list all users, toggle active/inactive)
- Category management (add/edit startup industries)
- Score weight configuration (change ML score weights)
- ML model metadata (view model info)
- Audit log viewer
- System notifications management

---

### App 13: `notifications/` — User Notifications

**Purpose:** In-app notification system for founders.

**Triggers:**
- Analysis complete → "Your idea analysis is ready!"
- Analysis failed → "Analysis failed, please retry"
- Admin messages

---

## 7. FRONTEND — PAGES EXPLAINED

### Public Pages (No Login Required)

| Page | File | What it does |
|------|------|-------------|
| Landing Page | `LandingPage.jsx` | Hero section, 55 startup idea explorer, features, how-it-works, testimonials, FAQ, government schemes section, team section, footer |
| Login | `LoginPage.jsx` | Split-screen dark design. JWT tokens stored in memory/localStorage on success |
| Register | `RegisterPage.jsx` | Sign up with full_name, email, password. Password strength indicator shows in real time |
| Forgot Password | `ForgotPasswordPage.jsx` | Sends reset email via backend (always shows "check email" regardless of whether email exists) |
| Reset Password | `ResetPasswordPage.jsx` | User lands here from email link, enters new password |

### Authenticated Pages (Founder)

| Page | File | What it does |
|------|------|-------------|
| Dashboard | `DashboardPage.jsx` | Summary stats (ideas submitted, analyzed, avg score), recent ideas, quick action buttons |
| Submit Idea | `IdeaSubmissionPage.jsx` | Multi-step form: Step 1 (basic info), Step 2 (business details), Step 3 (review & submit). Triggers analysis pipeline on submit |
| Idea List | `IdeaListPage.jsx` | All submitted ideas with status badges (pending/processing/complete/failed), search, filter |
| Idea Detail | `IdeaDetailPage.jsx` | Shows idea info + link to analysis tabs |
| Analysis Tabs | `AnalysisTabsPage.jsx` | **13-tab view** covering all analysis dimensions (see below) |
| Comparison | `ComparisonPage.jsx` | Select 2-3 ideas, side-by-side comparison table + radar chart |
| Reports | `ReportsPage.jsx` | Generate and download PDF report for an idea |
| AI Mentor | `MentorChatPage.jsx` | Chat interface. User types a question, Gemini AI answers in context of their startup |
| Analytics | `AnalyticsDashboardPage.jsx` | 15+ chart types showing platform insights and their own performance data |
| Profile | `ProfilePage.jsx` | Edit name, avatar, bio, theme preference, change password |
| Submission History | `SubmissionHistoryPage.jsx` | Timeline view of all past idea submissions |
| Pitch Deck Editor | `PitchDeckEditorPage.jsx` | Auto-generates a pitch deck from analysis data |

### Admin Pages

| Page | File | What it does |
|------|------|-------------|
| Admin Panel | `AdminPanelPage.jsx` | 6-tab admin view: Users, Categories, Score Weights, Audit Log, ML Models, Notifications |

### Analysis Tabs Page — 13 Tabs Detail

| Tab | Content |
|-----|---------|
| 1. Innovation | Radar chart of 5 sub-scores, composite score card |
| 2. Market | Market size, growth rate, competition gauge, timing score |
| 3. Risk | 5 risk bars, risk band (Low/Medium/High), mitigation tips |
| 4. SWOT | 4-quadrant SWOT board |
| 5. Recommendations | Revenue models, marketing channels, improvement suggestions |
| 6. Personas | 3 customer persona cards (avatar, demographics, pain points, goals) |
| 7. Canvas | 9-block Business Model Canvas grid |
| 8. Competitors | Competitor cards with similarity %, links |
| 9. Roadmap | 4-phase timeline with milestones |
| 10. Team | Required roles table with skills needed |
| 11. Investor Readiness | Score gauge + checklist of what's missing |
| 12. Govt. Schemes | Relevant government startup schemes by industry |
| 13. Charts | 8 advanced charts (candlestick, bubble, radar, waterfall, heatmap, gauges, sparklines, polar wheel) |

---

## 8. DATABASE DESIGN

**Database:** PostgreSQL 15
**Total Tables:** 38+
**Primary Keys:** UUID for user-facing entities (prevents enumeration), BigAutoField for internal lookup tables

### Core Table Relationships

```
accounts_user (UUID PK)
    │
    ├── ideas_idea (UUID PK, FK → accounts_user)
    │       │
    │       ├── analysis_nlpoutput (FK → ideas_idea)
    │       ├── analysis_innovationscore (FK → ideas_idea)
    │       ├── analysis_marketintelligence (FK → ideas_idea)
    │       ├── analysis_riskassessment (FK → ideas_idea)
    │       ├── analysis_swotanalysis (FK → ideas_idea)
    │       ├── analysis_recommendation (FK → ideas_idea)
    │       ├── analysis_customerpersona (FK → ideas_idea)
    │       ├── analysis_businessmodelcanvas (FK → ideas_idea)
    │       ├── analysis_teamrecommendation (FK → ideas_idea)
    │       ├── analysis_investorreadiness (FK → ideas_idea)
    │       ├── analysis_roadmapplan (FK → ideas_idea)
    │       └── competitor_competitorresult (FK → ideas_idea)
    │
    ├── mentor_chat_session (UUID PK, FK → accounts_user)
    │       └── mentor_chat_message (FK → session)
    │
    ├── reports_report (FK → accounts_user, FK → ideas_idea)
    ├── notifications_notification (FK → accounts_user)
    └── accounts_auditlog (FK → accounts_user, nullable)

ideas_startupcategory (separate lookup table)
    └── ideas_idea.industry_id → ideas_startupcategory
```

### Design Decisions

1. **UUID primary keys** — Cannot guess other users' idea IDs via sequential integers
2. **Soft delete** — `is_deleted=True` instead of DELETE; data preserved for audit
3. **JSON fields** — `domain_data`, `swot` items stored as JSON for flexibility
4. **Indexed columns** — `founder_id`, `analysis_status`, `industry_id`, `email`, `role` all have indexes for fast queries
5. **Separate analysis tables** — Each analysis dimension is its own table (not one giant JSON blob) → better query performance, easier to update individual dimensions
6. **`django_migrations` table** — Django tracks applied migrations; never manually alter DB schema

### Sample SQL Queries

```sql
-- Count ideas by analysis status
SELECT analysis_status, COUNT(*) 
FROM ideas_idea 
WHERE is_deleted = FALSE 
GROUP BY analysis_status;

-- Top 5 ideas by innovation score
SELECT i.startup_name, s.composite_score 
FROM ideas_idea i 
JOIN analysis_innovationscore s ON s.idea_id = i.id 
ORDER BY s.composite_score DESC 
LIMIT 5;

-- Industry distribution
SELECT sc.name, COUNT(i.id) as total
FROM ideas_idea i
JOIN ideas_startupcategory sc ON i.industry_id = sc.id
WHERE i.is_deleted = FALSE
GROUP BY sc.name
ORDER BY total DESC;

-- Average risk scores
SELECT 
    AVG(financial_risk_score) as avg_financial,
    AVG(technical_risk_score) as avg_technical,
    AVG(overall_risk_score) as avg_overall
FROM analysis_riskassessment;
```

---

## 9. AI / ML PIPELINE — 18 STAGES

When a founder submits an idea and clicks "Analyze", the following happens:

**Step 1:** Django view receives the request, sets `idea.analysis_status = "processing"`, and dispatches the Celery task `run_analysis_pipeline(idea_id)` asynchronously.

**Step 2:** The web response returns immediately to the user — they don't wait. The pipeline runs in the background.

### The 18 Stages (inside Celery worker):

| # | Stage | Module | Output Stored In |
|---|-------|--------|-----------------|
| 1 | Data Ingestion | `analysis/tasks.py` | — |
| 2 | NLP Preprocessing | `nlp_engine/pipeline.py` | — |
| 3 | Feature Extraction | `nlp_engine/pipeline.py` | `analysis_nlpoutput` |
| 4 | Industry Classification | `nlp_engine/pipeline.py` | `analysis_nlpoutput.industry` |
| 5 | Competitor Search | `competitor/analyzer.py` | `competitor_competitorresult` |
| 6 | Innovation Scoring | `scoring/innovation.py` | `analysis_innovationscore` |
| 7 | Market Intelligence | `scoring/market.py` | `analysis_marketintelligence` |
| 8 | Risk Assessment | `scoring/risk.py` | `analysis_riskassessment` |
| 9 | SWOT Generation | `generators/swot.py` | `analysis_swotanalysis` |
| 10 | Recommendations | `generators/recommendations.py` | `analysis_recommendation` |
| 11 | Customer Personas | `generators/personas.py` | `analysis_customerpersona` |
| 12 | Business Model Canvas | `generators/canvas.py` | `analysis_businessmodelcanvas` |
| 13 | Team Recommendation | `generators/team.py` | `analysis_teamrecommendation` |
| 14 | Investor Readiness | `scoring/investor.py` | `analysis_investorreadiness` |
| 15 | Roadmap Generation | `generators/roadmap.py` | `analysis_roadmapplan` |
| 16 | Persist to Database | `analysis/tasks.py` | All above tables |
| 17 | Status Update | `analysis/tasks.py` | `ideas_idea.analysis_status = "complete"` |
| 18 | Notify Founder | `notifications/` | `notifications_notification` |

### Error Handling in Pipeline

```python
@shared_task(bind=True, max_retries=3)
def run_analysis_pipeline(self, idea_id):
    try:
        # ... run stages ...
    except Exception as exc:
        # Exponential backoff: retry after 10s, 20s, 40s
        raise self.retry(exc=exc, countdown=10 * (2 ** self.request.retries))
    # After 3 failures:
    # - Set idea.analysis_status = "failed"
    # - Create AuditLog entry
    # - Send failure notification to founder
```

---

## 10. SECURITY IMPLEMENTATION

### 1. Authentication — JWT (JSON Web Tokens)
- Access token: valid for **60 minutes**
- Refresh token: valid for **7 days**
- Stored in browser (localStorage or HTTP-only cookie)
- Every API request must include: `Authorization: Bearer <access_token>`
- Logout revokes the refresh token server-side (stored hash in `accounts_refreshtoken`)
- Token rotation: each refresh issues a new refresh token and blacklists the old one

### 2. Password Security
- Django's `AbstractBaseUser` never stores plaintext passwords
- Password is hashed with **bcrypt** (work factor ≥ 12)
- Password reset uses SHA-256 hashed one-time tokens (raw token never stored in DB)
- Forgot password endpoint always returns 200 (prevents knowing if email is registered)

### 3. Role-Based Access Control (RBAC)
- 5 roles: student, founder, mentor, incubation, admin
- Custom DRF permission class `IsAdmin` on all admin endpoints
- Custom `IsFounder` on idea and analysis endpoints
- `ProtectedRoute` in React prevents accessing pages without valid token

### 4. Brute-Force Protection
- `LoginAttemptMiddleware` blocks an IP after **5 failed login attempts**
- All attempts logged in `accounts_loginattempt` table

### 5. SQL Injection Prevention
- Django ORM uses **parameterized queries** exclusively — no raw SQL with user input

### 6. XSS Prevention
- DRF returns JSON, not HTML — eliminates most XSS vectors
- React DOM automatically escapes rendered values

### 7. CORS Policy
- `django-cors-headers` configured with explicit origin allowlist
- Only `localhost:5173` (dev) and Vercel URL (prod) can make cross-origin requests

### 8. HTTPS
- Enforced in production via Render's TLS termination
- `SECURE_SSL_REDIRECT = True` in `prod.py`
- HSTS headers enabled

### 9. Soft Delete
- Users and ideas are never physically deleted
- `is_deleted=True` flag preserves data for audit purposes
- Queries always filter `is_deleted=False`

### 10. Audit Logging
- `AuditLogMiddleware` records every request with user ID, action, IP address
- AuditLog table is append-only (no update/delete permissions)

---

## 11. API ENDPOINTS REFERENCE

**Base URL:** `http://localhost:8000/api/v1/` (dev) or `https://ventureiq-api.onrender.com/api/v1/` (prod)

**Response Format — All endpoints return:**
```json
{
  "status": "success",
  "data": { ... },
  "errors": []
}
```

### Authentication Endpoints

| Method | URL | Auth Required | Description |
|--------|-----|--------------|-------------|
| POST | `/auth/register/` | No | Register new user, returns JWT tokens |
| POST | `/auth/login/` | No | Login, returns JWT tokens |
| POST | `/auth/logout/` | Yes | Revoke refresh token |
| POST | `/auth/token/refresh/` | Yes | Get new access token |
| GET | `/auth/profile/` | Yes | Get current user profile |
| PATCH | `/auth/profile/` | Yes | Update profile fields |
| POST | `/auth/password/forgot/` | No | Send password reset email |
| POST | `/auth/password/reset/` | No | Set new password with reset token |
| POST | `/auth/password/change/` | Yes | Change password (requires current) |

### Ideas Endpoints

| Method | URL | Auth Required | Description |
|--------|-----|--------------|-------------|
| GET | `/ideas/` | Yes | List founder's own ideas |
| POST | `/ideas/` | Yes | Submit new idea + trigger analysis |
| GET | `/ideas/<uuid>/` | Yes | Get idea detail |
| PUT | `/ideas/<uuid>/` | Yes | Update idea |
| DELETE | `/ideas/<uuid>/` | Yes | Soft-delete idea |
| GET | `/ideas/<uuid>/analysis/` | Yes | Get complete analysis results |

### Mentor Chat Endpoints

| Method | URL | Auth Required | Description |
|--------|-----|--------------|-------------|
| GET | `/mentor/sessions/` | Yes | List chat sessions |
| POST | `/mentor/sessions/` | Yes | Create new session for an idea |
| GET | `/mentor/sessions/<id>/messages/` | Yes | Get message history |
| POST | `/mentor/sessions/<id>/messages/` | Yes | Send message, get AI reply |

### Reports Endpoints

| Method | URL | Auth Required | Description |
|--------|-----|--------------|-------------|
| GET | `/reports/` | Yes | List generated reports |
| POST | `/reports/generate/` | Yes | Trigger PDF generation |
| GET | `/reports/<id>/download/` | Yes | Download PDF file |

### Analytics & Admin

| Method | URL | Auth Required | Description |
|--------|-----|--------------|-------------|
| GET | `/analytics/` | Yes (any) | Platform analytics data |
| GET | `/comparisons/compare/` | Yes | Compare 2-3 ideas |
| GET | `/admin-panel/users/` | Admin only | List all users |
| GET | `/admin-panel/audit-log/` | Admin only | Security audit trail |
| GET/PUT | `/admin-panel/score-weights/` | Admin only | ML score weights config |

---

## 12. DEPLOYMENT ARCHITECTURE

```
Developer pushes code to GitHub
        │
        ▼
┌───────────────────┐          ┌──────────────────────────────┐
│    Vercel          │          │    Render                     │
│  (Frontend)        │          │  (Backend)                    │
│                   │          │                              │
│  npm run build    │          │  ventureiq-api               │
│  React → static  │          │  (Django + Gunicorn)          │
│  files on CDN     │          │                              │
│                   │          │  ventureiq-worker            │
│  vercel.json:     │          │  (Celery worker)             │
│  all routes →     │          │                              │
│  index.html       │          │  ventureiq-beat              │
│  (SPA routing)    │          │  (Celery scheduler)          │
└───────────────────┘          │                              │
                               │  ventureiq-db                │
                               │  (PostgreSQL 15)             │
                               │                              │
                               │  ventureiq-redis             │
                               │  (Redis 7)                   │
                               └──────────────────────────────┘
```

**`Procfile` content:**
```
web: gunicorn ventureiq.wsgi:application --bind 0.0.0.0:$PORT
worker: celery -A ventureiq worker -l info -Q analysis,reports
beat: celery -A ventureiq beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

**`render.yaml`** declares all services, environment variables references, and resource tiers as code — one file deploys the entire infrastructure.

**`vercel.json`** rewrites all URL patterns to `index.html` so React Router handles navigation without 404 errors on page refresh.

### Performance Targets
- API response time: < 2 seconds at P95 under 100 concurrent users
- Full analysis pipeline: completes within 120 seconds
- Uptime target: 99.5% monthly

---

---

## 13. FACULTY QUESTIONS & ANSWERS

> This section covers the most likely questions a faculty examiner will ask, file by file and concept by concept.

---

### SECTION A — General Project Questions

**Q1. What is VentureIQ and what problem does it solve?**

VentureIQ is an AI/ML-powered web platform that evaluates startup ideas automatically. The problem is that early-stage founders (especially students) cannot afford consultants to validate their ideas. VentureIQ automates this using NLP and machine learning, giving them a full startup intelligence report in minutes instead of weeks.

---

**Q2. What is the overall architecture of your project?**

It follows a 3-tier architecture:
- **Presentation Layer** — React 18 SPA hosted on Vercel
- **Application Layer** — Django REST Framework on Render, handling business logic and API
- **Data Layer** — PostgreSQL for persistent data, Redis as message broker and cache

There is also a 4th component: the Celery worker which processes the AI/ML pipeline asynchronously so the main web server stays responsive.

---

**Q3. Why did you choose Django over Node.js or Flask?**

- Django has a built-in ORM, admin panel, migration system, and authentication framework — which saved development time
- It integrates naturally with Python's ML libraries (scikit-learn, spaCy, pandas)
- Django REST Framework provides serializers, authentication, and pagination out of the box
- Flask would require building everything from scratch; Node.js would need a separate Python service for ML

---

**Q4. Why did you use React for the frontend?**

- Component-based architecture enables code reuse (e.g., ScoreCard component used in 4 different pages)
- React's virtual DOM efficiently updates only changed parts of the UI
- Hooks and React Query simplify state management for async API calls
- Large ecosystem of libraries (Recharts for charts, Headless UI for accessible components)

---

**Q5. How does a user flow through the system from start to finish?**

1. User registers → JWT tokens issued
2. User fills in startup idea submission form (multi-step)
3. Form submits → Django API creates Idea record → dispatches Celery task
4. User sees "Processing" status; they can continue using the app
5. Background pipeline runs 18 stages → saves results to DB
6. Status updates to "Complete", notification sent
7. User opens Analysis tabs to see all 13 dimensions of results
8. User can chat with AI Mentor, compare ideas, download PDF report

---

### SECTION B — Backend & Django Questions

**Q6. Explain the `models.py` file in the `ideas` app.**

The `Idea` model is the core entity. Key design decisions:
- **UUID primary key** instead of integer — prevents guessing other users' idea IDs
- **ForeignKey to CustomUser** — each idea belongs to one founder
- **`analysis_status` field** — a state machine (pending → processing → complete/failed) that tracks pipeline progress
- **Soft delete** via `is_deleted` boolean — data is never permanently lost
- **`domain_data` JSONField** — stores industry-specific extra fields without schema migrations for each industry

---

**Q7. What is the purpose of `serializers.py` in Django REST Framework?**

Serializers do two things:
1. **Deserialization (Input validation):** When a POST request comes in with JSON, the serializer validates the data (required fields, data types, length limits, custom rules) before it reaches the view
2. **Serialization (Output formatting):** When the view returns data, the serializer converts the Django model instance into JSON

For example, `IdeaSerializer` validates that `startup_name` is present and under 200 characters, and that the `industry` ID actually exists in the database.

---

**Q8. What is Celery and why is it needed?**

Celery is an asynchronous task queue. It's needed because the AI/ML analysis pipeline takes 30-120 seconds to complete. Without Celery:
- The user would have to wait 2 minutes for the HTTP response — terrible UX
- The web server would time out

With Celery:
- Django dispatches the task to a Redis queue and returns a response immediately
- A separate Celery worker process picks up the task and runs it in the background
- User sees "Processing" status, then gets notified when done

---

**Q9. What is JWT authentication and how is it implemented here?**

JWT (JSON Web Token) is a stateless authentication mechanism. Instead of storing sessions in a database, the server issues a signed token that proves the user's identity.

**Implementation:**
- `djangorestframework-simplejwt` library handles token generation
- Access token lives 60 minutes; refresh token lives 7 days
- Every API request must include `Authorization: Bearer <token>` header
- DRF's `JWTAuthentication` class automatically verifies the token
- On logout, the refresh token's SHA-256 hash is stored in `accounts_refreshtoken.is_revoked=True`
- Token rotation: using the refresh token issues a new pair and blacklists the old refresh token

---

**Q10. Explain the middleware stack in `settings/base.py`.**

Middleware runs in order for every request/response:
1. `SecurityMiddleware` — adds security headers (HSTS, X-Content-Type-Options)
2. `CorsMiddleware` — checks if the request origin is in the allowed list
3. `RateLimitMiddleware` — throttles requests to prevent abuse
4. `LoginAttemptMiddleware` — blocks IPs that have failed login 5+ times
5. `SessionMiddleware` → `CommonMiddleware` → `CsrfViewMiddleware` — standard Django
6. `AuthenticationMiddleware` — identifies the current user from the JWT token
7. `AuditLogMiddleware` — runs last so it can log the response status code too

---

**Q11. What is the `CustomUser` model and why not use Django's built-in User?**

Django's built-in User uses **username** as the login identifier. We want **email** as the login field. Once you start a project, you cannot change this without resetting migrations. So we defined `CustomUser` early, extending `AbstractBaseUser`, with:
- `email` as `USERNAME_FIELD`
- UUID primary key instead of integer
- Role field (founder/admin/mentor/student/incubation)
- Soft-delete fields
- Theme preference

---

**Q12. How does the password reset flow work?**

1. User submits email to `POST /auth/password/forgot/`
2. Backend finds the user (if they exist) — if not, silently does nothing
3. `make_reset_token(user)` generates a cryptographically random token
4. SHA-256 hash of the token is stored in `accounts_passwordresettoken`
5. Raw token is embedded in the reset URL emailed to the user
6. User clicks link → frontend sends token + new password to `POST /auth/password/reset/`
7. Backend hashes the received token and looks it up in the DB
8. If found and not used/expired → sets new password, marks token as `used=True`

Security note: The endpoint always returns HTTP 200 whether the email exists or not (prevents attackers from enumerating registered emails).

---

**Q13. How do you prevent SQL injection?**

Django's ORM uses parameterized queries exclusively. For example:
```python
# This is SAFE — Django parameterizes email value
user = CustomUser.objects.get(email=request.data['email'])
# SQL generated: SELECT * FROM accounts_user WHERE email = %s  [value bound separately]
```
Raw SQL with `cursor.execute()` is never used with user input in this project.

---

### SECTION C — Machine Learning Questions

**Q14. What ML algorithm did you use and why?**

**GradientBoostingRegressor** from scikit-learn.

Why:
- Handles non-linear relationships in startup evaluation data
- Ensemble method (combines hundreds of weak decision trees) → higher accuracy
- More robust to outliers than linear regression
- Has built-in feature importance — we can explain which features influenced the score
- Better than Random Forest for structured tabular data with this size

---

**Q15. What features are fed into the ML models?**

Features extracted from NLP preprocessing:
- `keyword_count` — number of unique meaningful keywords
- `description_length` — word count of the description
- `tfidf_score` — Term Frequency-Inverse Document Frequency relevance score
- `industry_code` — label-encoded industry category
- `development_stage_code` — encoded (idea=0, validation=1, prototype=2, mvp=3, growth=4, scaling=5)
- `team_size` — number of team members
- `business_model_code` — encoded (SaaS, marketplace, subscription, etc.)

---

**Q16. What is TF-IDF and how is it used here?**

TF-IDF stands for **Term Frequency-Inverse Document Frequency**. It measures how important a word is to a document relative to a collection of documents.

- **TF (Term Frequency):** How often a word appears in this specific description
- **IDF (Inverse Document Frequency):** How rare the word is across all startup descriptions (rare = more meaningful)

In VentureIQ, TF-IDF is used in two ways:
1. **Feature extraction** — top TF-IDF score for a description is fed as an ML feature
2. **Competitor matching** — each competitor's description is vectorized with TF-IDF; cosine similarity against the idea's vector finds the most similar competitors

---

**Q17. What is Cosine Similarity and how does competitor matching work?**

Cosine similarity measures the angle between two vectors. If two startup descriptions have similar vocabulary, their TF-IDF vectors point in similar directions → high cosine similarity (0 to 1).

Steps:
1. Load 50+ competitor profiles from `competitors.json`
2. Build a TF-IDF matrix for all competitor descriptions
3. Convert the user's idea to a TF-IDF vector using the same vocabulary
4. Compute cosine similarity: `similarity = (A · B) / (|A| × |B|)`
5. Sort by similarity, return top-N competitors
6. Classify competition: >0.6 = High, 0.3-0.6 = Medium, <0.3 = Low

---

**Q18. What is the fallback if ML models are not loaded?**

The scoring modules check if `.pkl` files exist and are loadable. If not (e.g., files corrupted, server cold start), they fall back to **rule-based heuristic scoring**:
- Long descriptions with many keywords → higher novelty score
- Later development stage → higher practicality
- Larger team size → lower team risk
- Industry encoding → preset base market scores

This ensures the pipeline never completely fails even without ML models.

---

**Q19. How were the ML models trained?**

1. Created a synthetic dataset of startup evaluations with feature columns and target scores
2. Split 80/20 train/test
3. Trained one `GradientBoostingRegressor` per scoring dimension (12 models total)
4. Used cross-validation (5-fold) to tune hyperparameters
5. Evaluated with Mean Absolute Error (MAE) and R² score
6. Serialized trained models with `joblib.dump()` to `.pkl` files
7. At inference time, `joblib.load()` restores the model and calls `.predict(features)`

---

### SECTION D — Frontend Questions

**Q20. What is React and how does routing work?**

React is a JavaScript library for building UIs with reusable components. It uses a Virtual DOM — changes are computed in memory first, then only the actual differences are applied to the real DOM (efficient updates).

**React Router v6** handles navigation:
- `<BrowserRouter>` wraps the app
- `<Routes>` + `<Route path="..." element={...}>` define the mapping
- Clicking a link does NOT reload the page — React swaps the component
- `<ProtectedRoute>` is a wrapper that redirects to `/login` if no valid JWT token exists

---

**Q21. What is React Query and why use it over plain useState?**

React Query (`@tanstack/react-query`) manages **server state** — data that lives on the server and is fetched asynchronously. Without it, you'd need:
```javascript
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
// manually call useEffect, handle loading, error, refetch...
```
React Query replaces all that with:
```javascript
const { data, isLoading, error } = useQuery(['ideas'], fetchIdeas);
```
It also provides automatic **caching** (don't re-fetch if data is fresh), **background refetching**, and **invalidation** (mark data stale after a mutation).

---

**Q22. How does the frontend handle authentication?**

1. On login, backend returns `{ access, refresh, user }`
2. `AuthContext.jsx` stores the access token in memory and user object in state
3. Axios interceptor in `axiosInstance.js` automatically attaches `Authorization: Bearer <token>` to every request
4. When a 401 response is received, the interceptor silently calls `/token/refresh/` to get a new access token
5. On logout, tokens are cleared from memory and the user is redirected to login

---

**Q23. Why was Tailwind CSS chosen over plain CSS or Bootstrap?**

- **Utility-first** — style directly in JSX with class names like `bg-gray-900 text-white rounded-xl`
- No context switching between `.css` files and `.jsx` files
- Responsive design built-in: `md:flex lg:grid`
- Dark mode built-in: `dark:bg-gray-800`
- No unused CSS in production (PurgeCSS removes unused classes during build)
- Bootstrap has opinionated pre-built components that are harder to customize for a premium UI

---

### SECTION E — System Design & Advanced Questions

**Q24. How does your system handle failure in the analysis pipeline?**

The Celery task uses:
```python
@shared_task(bind=True, max_retries=3)
def run_analysis_pipeline(self, idea_id):
    try:
        # pipeline stages...
    except Exception as exc:
        raise self.retry(exc=exc, countdown=10 * (2 ** self.request.retries))
        # Retry after: 10s, 20s, 40s
```
After 3 failures:
- `idea.analysis_status = "failed"`
- An AuditLog entry is created with the error details
- A failure notification is sent to the founder

---

**Q25. How does your app scale?**

The architecture is **stateless** — the Django web server doesn't store any session data locally. This means:
- Multiple Django instances can run behind a load balancer (horizontal scaling)
- Redis handles shared state (cached results, Celery broker)
- PostgreSQL handles persistent state
- Render allows increasing the instance count with one configuration change

Celery workers can also be scaled independently of web workers — during peak analysis load, add more Celery worker instances.

---

**Q26. What is soft delete and why use it?**

Instead of `DELETE FROM ideas_idea WHERE id = ?`, we set `is_deleted = True`.

Benefits:
- Data is preserved for auditing and compliance
- Accidental deletes can be recovered
- Foreign key integrity maintained (no orphan records)
- Audit log always has full history

All queries filter with `.filter(is_deleted=False)` so deleted records are invisible to users.

---

**Q27. How does the notification system work?**

1. When the Celery pipeline completes (or fails), it creates a `Notification` record in the database with the `founder` FK and message
2. When the frontend loads, it fetches `GET /api/v1/notifications/` via React Query
3. The `NotificationPanel` component shows a badge count with unread notifications
4. Clicking a notification marks it as `is_read = True`

Currently it's a polling mechanism — React Query re-fetches on window focus. WebSocket real-time push is a future enhancement.

---

**Q28. What testing strategy did you use?**

- **Backend:** `pytest` with `pytest-django` plugin
  - Unit tests for models, serializers, scoring modules
  - Integration tests for API endpoints (using Django test client)
  - Property-based testing with `hypothesis` for edge cases in scoring logic
  - Test coverage tracked with `pytest-cov`
  - `factory-boy` for generating realistic test data without fixtures

- **Frontend:** No automated tests (manual testing via browser dev tools and Postman for API)

---

**Q29. What is the purpose of `render.yaml`?**

It's **Infrastructure-as-Code** — a single YAML file that declares all Render cloud resources:
- Web service (Django + Gunicorn)
- Worker service (Celery)
- Beat service (Celery scheduler)
- PostgreSQL database
- Redis instance

Instead of clicking through Render's UI for every deployment, everything is version-controlled and reproducible.

---

**Q30. What is `vercel.json` and why is it needed?**

For a React SPA, all routes like `/dashboard`, `/ideas/123` are handled by React Router — not by a web server. When a user directly types `ventureiq.vercel.app/dashboard` in the browser, Vercel would return 404 because that file doesn't exist.

`vercel.json` has a rewrite rule:
```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```
This tells Vercel: for any URL, serve `index.html`. React Router then reads the URL and renders the correct page.

---

**Q31. What is the `.env` file and why is it never committed to Git?**

`.env` contains sensitive configuration:
- `SECRET_KEY` — Django's cryptographic signing key
- `DB_PASSWORD` — Database password
- `GEMINI_API_KEY` — Third-party API key (paid service)
- `JWT_SECRET` — JWT signing secret

If these were committed to GitHub, anyone could:
- Forge authentication tokens
- Access/destroy the database
- Run up API costs on Gemini

`.gitignore` excludes `.env`. `.env.example` (no real values) is committed as a template.

---

**Q32. What is Gunicorn and why is it used in production instead of Django's built-in server?**

Django's `python manage.py runserver` is a single-threaded development server — one request at a time. It's also not hardened for production.

**Gunicorn** (Green Unicorn) is a production WSGI server that:
- Spawns multiple worker processes to handle concurrent requests
- Handles worker crashes gracefully (restarts failed workers)
- Integrates with Nginx/load balancers
- Is used by virtually all production Python deployments

---

**Q33. Why are there separate settings files (base.py, dev.py, prod.py)?**

**DRY principle + Security separation:**
- `base.py` — settings shared in all environments (installed apps, DRF config, JWT config)
- `dev.py` — development-specific: `DEBUG=True`, console email backend, relaxed CORS
- `prod.py` — production-specific: `DEBUG=False`, HTTPS enforcement, HSTS, strict CORS, S3 media storage

`DJANGO_SETTINGS_MODULE` env variable selects which file to use:
- `ventureiq.settings.dev` locally
- `ventureiq.settings.prod` on Render

---

**Q34. What is the Business Model Canvas and how is it generated?**

The Business Model Canvas is a strategic framework with 9 blocks:
1. Key Partners, 2. Key Activities, 3. Key Resources
4. Value Propositions, 5. Customer Relationships, 6. Channels
7. Customer Segments, 8. Cost Structure, 9. Revenue Streams

In VentureIQ, `generators/canvas.py` uses the idea's industry, business model type, NLP keywords, and scoring results to populate each block with relevant text. For example, a FinTech SaaS startup would get "Banks and NBFCs" as a Key Partner suggestion.

---

**Q35. What are the most important non-functional requirements of this project?**

| Requirement | Implementation |
|------------|---------------|
| Performance: API < 2s | Connection pooling (`CONN_MAX_AGE`), indexed DB columns, React Query caching |
| Security: No plaintext passwords | bcrypt hashing via Django's AbstractBaseUser |
| Security: No SQL injection | Django ORM parameterized queries only |
| Availability: 99.5% uptime | Deployed on Render with health checks, Gunicorn multi-worker |
| Scalability | Stateless Django + Celery + Redis = horizontal scale |
| Data Integrity | UUID PKs, soft-delete, FK constraints, CHECK constraints on score columns |
| Auditability | AuditLog table records every auth event and admin action |

---

### SECTION F — Quick-Fire Technical Questions

**Q36. What is the difference between `access_token` and `refresh_token`?**
- Access token: Short-lived (60 min), sent with every API request in Authorization header
- Refresh token: Long-lived (7 days), only used to get a new access token when it expires

**Q37. What does `on_delete=models.CASCADE` mean?**
When the parent record (e.g., User) is deleted, all related child records (e.g., their Ideas) are automatically deleted too.

**Q38. What is `related_name` in a ForeignKey?**
It creates a reverse accessor. `founder = ForeignKey(User, related_name="ideas")` lets you write `user.ideas.all()` instead of `Idea.objects.filter(founder=user)`.

**Q39. What is the difference between `blank=True` and `null=True`?**
- `null=True` — allows NULL value in the database column
- `blank=True` — allows empty string in form/serializer validation
- For CharField use `blank=True` only; for numeric/date fields use `null=True`

**Q40. What is a UUID and why use it as a primary key?**
UUID (Universally Unique Identifier) is a 128-bit randomly generated ID (e.g., `550e8400-e29b-41d4-a716-446655440000`). Unlike auto-increment integers (1, 2, 3...), UUIDs cannot be guessed, preventing IDOR (Insecure Direct Object Reference) attacks.

**Q41. What is React Query's `useQuery` vs `useMutation`?**
- `useQuery` — for GET requests (reading data), with automatic caching and background refetching
- `useMutation` — for POST/PUT/DELETE requests (writing data), with `onSuccess` callback to invalidate cached queries

**Q42. What is spaCy used for specifically?**
spaCy performs linguistic analysis:
- Tokenization: "AI startup for healthcare" → ["AI", "startup", "for", "healthcare"]
- Lemmatization: "running" → "run", "startups" → "startup"
- POS tagging: "AI" → NOUN, "powerful" → ADJ
- Named entity recognition: detecting proper nouns, organizations

**Q43. What is WeasyPrint?**
WeasyPrint converts HTML+CSS to PDF. The reports module renders a Django HTML template with all analysis data, then WeasyPrint generates a professional PDF document from it.

**Q44. How many database tables does VentureIQ have?**
38+ tables, including all Django internal tables (`django_migrations`, `auth_permission`, etc.) and the 13+ custom app tables.

**Q45. What is `DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"` in settings?**
It sets the default primary key type for all models that don't specify one. `BigAutoField` is a 64-bit integer auto-increment — supports up to 9.2 quintillion rows before overflow.

---

*End of Faculty Presentation Guide*

*Project: VentureIQ — AI-Powered Startup Intelligence Platform*
*Team: Parshva Shah | Jinang Shah | Jay Raval*
*Institution: LJ University*
*Academic Year: 2025–2026*
