# VentureIQ — Complete File Guide

> This document explains every important file in the project, what it does, and how it connects to the rest of the system. Use this as a reference when faculty ask about any specific file.

---

## Table of Contents

1. [Project Structure Overview](#1-project-structure-overview)
2. [Backend — Configuration Files](#2-backend--configuration-files)
3. [Backend — Django Settings](#3-backend--django-settings)
4. [Backend — Accounts App](#4-backend--accounts-app)
5. [Backend — Ideas App](#5-backend--ideas-app)
6. [Backend — Analysis App](#6-backend--analysis-app)
7. [Backend — Other Apps](#7-backend--other-apps)
   - [7a. ml_models Folder (Detailed)](#7a-backend--ml_models-folder-detailed)
8. [Frontend — Entry Point & Config](#8-frontend--entry-point--config)
9. [Frontend — Pages](#9-frontend--pages)
10. [Frontend — Analysis Sub-tabs](#10-frontend--analysis-sub-tabs)
11. [Frontend — Admin Sub-pages](#11-frontend--admin-sub-pages)
12. [Frontend — Components](#12-frontend--components)

---

## 1. Project Structure Overview

```
VentureIQ/
├── backend/                  ← Django REST API (Python)
│   ├── ventureiq/            ← Django project config (settings, urls)
│   ├── accounts/             ← User auth & profiles
│   ├── ideas/                ← Idea submission & management
│   ├── analysis/             ← AI analysis results (all sub-models)
│   ├── scoring/              ← Innovation scoring weights
│   ├── mentor_chat/          ← AI mentor chat
│   ├── reports/              ← PDF reports & pitch decks
│   ├── analytics/            ← Platform statistics
│   ├── notifications/        ← User notifications
│   ├── admin_panel/          ← Admin management APIs
│   ├── nlp_engine/           ← NLP processing logic
│   ├── ml_pipeline/          ← ML training pipeline
│   ├── generators/           ← Content generators
│   ├── competitor/           ← Competitor analysis
│   ├── manage.py             ← Django CLI entry point
│   ├── requirements.txt      ← Python dependencies
│   └── .env                  ← Secret environment variables
│
├── frontend/                 ← React + Vite (JavaScript)
│   ├── src/
│   │   ├── main.jsx          ← App entry point
│   │   ├── App.jsx           ← Route definitions
│   │   ├── pages/            ← One file per page/screen
│   │   ├── components/       ← Shared UI components
│   │   ├── context/          ← Global state (auth)
│   │   ├── api/              ← API call functions
│   │   ├── hooks/            ← Custom React hooks
│   │   └── utils/            ← Utility functions
│   ├── package.json          ← JS dependencies
│   └── vite.config.js        ← Build tool config
│
└── PROJECT_DOCUMENTATION.md  ← High-level project docs
```

---

## 2. Backend — Configuration Files


### `backend/manage.py`

**What it is:** Django's command-line utility — the main tool used to control the backend.

**What it does:**
- Sets `DJANGO_SETTINGS_MODULE=ventureiq.settings.dev` so Django loads the dev settings
- Delegates all commands to Django's management system

**Common commands:**
```bash
python manage.py runserver        # Start development server on localhost:8000
python manage.py migrate          # Apply database migrations (create/update tables)
python manage.py createsuperuser  # Create an admin account
python manage.py makemigrations   # Generate new migration files after model changes
```

---

### `backend/requirements.txt`

**What it is:** A list of all Python libraries the project needs, with pinned (exact) versions for reproducibility.

**Key dependencies:**

| Library | Version | Purpose |
|---------|---------|---------|
| `Django` | 4.2.7 | Core web framework |
| `djangorestframework` | 3.14.0 | Builds the REST API |
| `djangorestframework-simplejwt` | 5.3.1 | JWT authentication tokens |
| `django-cors-headers` | 4.3.1 | Allows frontend to call the API |
| `psycopg2-binary` | 2.9.9 | PostgreSQL database driver |
| `celery` | 5.3.6 | Background task queue (runs analysis async) |
| `redis` | 5.0.1 | Message broker for Celery |
| `spacy` | 3.7.2 | Natural language processing |
| `scikit-learn` | 1.3.2 | Machine learning models |
| `pandas` / `numpy` | 2.1.4 / 1.26.2 | Data manipulation |
| `weasyprint` | 60.2 | PDF report generation |
| `bleach` | 6.1.0 | Input sanitization / XSS prevention |
| `gunicorn` | 21.2.0 | Production WSGI server |

**Install all dependencies:**
```bash
pip install -r requirements.txt
```

---

### `backend/.env`

**What it is:** A local environment variables file. Contains all secrets and configuration values. **Never committed to git.**

**Key variables:**

| Variable | Purpose |
|----------|---------|
| `SECRET_KEY` | Django's cryptographic signing key |
| `DEBUG` | `True` in dev, `False` in production |
| `DB_NAME, DB_USER, DB_PASSWORD, DB_HOST, DB_PORT` | PostgreSQL connection details |
| `GEMINI_API_KEY` | Google Gemini AI API key for analysis |
| `GROQ_API_KEY` | Groq (Llama 3) API key for mentor chat |
| `JWT_SECRET` | Signing key for JWT tokens |
| `REDIS_URL` | Celery/Redis broker connection |
| `CORS_ALLOWED_ORIGINS` | Frontend URLs allowed to call the API |

> **Note:** The `.env.example` file shows the same keys with placeholder values — safe to share or commit.

---

### `backend/Procfile`

**What it is:** Tells Render (cloud hosting) how to start the app in production.

```
web: gunicorn ventureiq.wsgi --log-file -
```

---

### `backend/render.yaml`

**What it is:** Infrastructure-as-code for deploying to Render. Defines the web service, database, and environment variables needed for production deployment.

---

## 3. Backend — Django Settings


### `backend/ventureiq/settings/base.py`

**What it is:** The core Django configuration shared across all environments (dev and production).

**Key sections:**

**INSTALLED_APPS** — All Django apps that are loaded:
- Django built-ins: admin, auth, sessions, staticfiles
- Third-party: rest_framework, simplejwt, corsheaders
- Local apps: accounts, ideas, analysis, scoring, mentor_chat, reports, analytics, notifications, ml_pipeline, admin_panel

**MIDDLEWARE** — The request/response processing pipeline (in order):
1. `SecurityMiddleware` — adds HTTPS/security headers
2. `CorsMiddleware` — handles cross-origin requests from the frontend
3. `RateLimitMiddleware` — throttles excessive API calls
4. `LoginAttemptMiddleware` — blocks IPs after 5 failed login attempts
5. Standard Django middleware (sessions, CSRF, auth, etc.)
6. `AuditLogMiddleware` — records all actions for compliance

**DATABASES** — PostgreSQL connection using `.env` values:
```python
ENGINE = "django.db.backends.postgresql"
NAME   = DB_NAME     # ventureiq
USER   = DB_USER     # postgres
PORT   = DB_PORT     # 5432
```

**REST_FRAMEWORK** — API defaults:
- Authentication: JWT (Bearer token)
- Default permission: IsAuthenticated (login required)
- Pagination: 20 items per page
- Response format: JSON only

**SIMPLE_JWT** — Token settings:
- Access token expires: **60 minutes**
- Refresh token expires: **7 days**
- Token rotation: enabled (new refresh token issued on each refresh)

**CELERY** — Async task queue:
- Broker: Redis
- Task routing: analysis tasks → `analysis` queue, reports → `reports` queue

---

### `backend/ventureiq/settings/dev.py`

**What it is:** Development-specific overrides. Sets `DEBUG=True`, allows all hosts, uses console email backend (prints emails to terminal instead of sending them).

---

### `backend/ventureiq/settings/prod.py`

**What it is:** Production overrides. Sets `DEBUG=False`, reads `DATABASE_URL` from Render, enables WhiteNoise for static files, uses real SMTP email.

---

### `backend/ventureiq/urls.py`

**What it is:** The main URL router — maps URL paths to Django apps.

**All API endpoints:**

| URL Prefix | App Mounted | Purpose |
|------------|-------------|---------|
| `/admin/` | Django admin | Built-in superuser interface |
| `/api/v1/auth/` | accounts | Login, register, password reset |
| `/api/v1/ideas/` | ideas + competitor | Submit and manage ideas |
| `/api/v1/analysis/` | analysis | AI analysis results |
| `/api/v1/reports/` | reports | PDF reports |
| `/api/v1/comparisons/` | analysis | Side-by-side idea comparison |
| `/api/v1/mentor/` | mentor_chat | AI chat sessions |
| `/api/v1/analytics/` | analytics | Platform statistics |
| `/api/v1/notifications/` | notifications | User notifications |
| `/api/v1/admin-panel/` | admin_panel | Admin management |

---

### `backend/ventureiq/celery.py`

**What it is:** Celery app configuration. Configures the async task worker that runs the AI analysis pipeline in the background so API responses don't block.

---

## 4. Backend — Accounts App


### `backend/accounts/models.py`

**What it is:** Defines all database tables for users and authentication.

**Models:**

#### `CustomUser`
Replaces Django's default User model.
- **Primary key:** UUID (not auto-increment integer)
- **Login field:** Email (not username)
- **Roles:** `student`, `founder`, `mentor`, `incubation`, `admin`
- **Soft-delete:** `is_deleted` + `deleted_at` flags (records are hidden, not truly deleted)
- **Extra fields:** `full_name`, `avatar_url`, `bio`, `theme` (light/dark)

#### `RefreshToken`
Stores SHA-256 hash of issued JWT refresh tokens.
- Enables server-side revocation (logout from any device)
- The raw token is never stored — only its hash

#### `LoginAttempt`
Records every login attempt (success or failure) per IP address.
- Used by `LoginAttemptMiddleware` to block after 5 consecutive failures
- Prevents brute-force password attacks

#### `PasswordResetToken`
One-time password reset tokens.
- Stored as SHA-256 hash only (raw token is emailed to the user)
- Has `expires_at` (30 min) and `used` flag

#### `AuditLog`
Immutable record of all important system events.
- Logs: logins, idea submissions, deletions, admin actions
- Fields: `user`, `action`, `resource`, `resource_id`, `ip_address`, `detail`

---

### `backend/accounts/views.py`

**What it is:** All authentication API endpoints.

| View | Method | URL | What it does |
|------|--------|-----|-------------|
| `RegisterView` | POST | `/api/v1/auth/register/` | Creates account, returns JWT tokens |
| `LoginView` | POST | `/api/v1/auth/login/` | Verifies credentials, returns tokens |
| `LogoutView` | POST | `/api/v1/auth/logout/` | Revokes refresh token |
| `TokenRefreshView` | POST | `/api/v1/auth/token/refresh/` | Issues new token pair |
| `ForgotPasswordView` | POST | `/api/v1/auth/password/forgot/` | Sends reset email |
| `ResetPasswordView` | POST | `/api/v1/auth/password/reset/` | Applies new password |
| `ProfileView` | GET/PATCH | `/api/v1/auth/profile/` | Read or update profile |
| `ChangePasswordView` | POST | `/api/v1/auth/password/change/` | Change password (needs current password) |

**Response format** (all endpoints):
```json
{ "status": "success", "data": { ... }, "errors": [] }
{ "status": "error",   "data": null,    "errors": ["..."] }
```

> **Security note:** `ForgotPasswordView` always returns HTTP 200 even if the email doesn't exist — this prevents attackers from discovering valid email addresses.

---

### `backend/accounts/serializers.py`

**What it is:** Validates and transforms request/response data for auth endpoints. Each serializer maps to one view (e.g., `RegisterSerializer`, `LoginSerializer`, `ProfileSerializer`).

---

### `backend/accounts/urls.py`

**What it is:** Maps URL paths to auth views. Mounted at `/api/v1/auth/`.

```python
register/          → RegisterView
login/             → LoginView
logout/            → LogoutView
token/refresh/     → TokenRefreshView
password/forgot/   → ForgotPasswordView
password/reset/    → ResetPasswordView
password/change/   → ChangePasswordView
profile/           → ProfileView
```

---

### `backend/accounts/middleware.py`

**What it is:** Three custom middleware classes that run on every request:

- **`RateLimitMiddleware`** — limits requests per user/IP to prevent API abuse
- **`LoginAttemptMiddleware`** — blocks IPs that have exceeded 5 failed login attempts
- **`AuditLogMiddleware`** — automatically logs certain actions to `AuditLog`

---

### `backend/accounts/permissions.py`

**What it is:** Custom DRF permission classes.

- **`IsOwnerOrAdminOrMentor`** — allows access only if the user owns the resource, or has admin/mentor role

---

## 5. Backend — Ideas App


### `backend/ideas/models.py`

**What it is:** Database tables for startup idea submissions.

**Models:**

#### `StartupCategory`
Lookup table of startup industries managed by admins.
- Examples: FinTech, HealthTech, AI/ML, Agriculture, Cybersecurity
- Fields: `name`, `slug`, `description`, `is_active`

#### `Idea`
The core entity — what a founder submits for AI analysis.

| Field | Type | Purpose |
|-------|------|---------|
| `id` | UUID | Primary key |
| `founder` | FK → CustomUser | Who submitted it |
| `industry` | FK → StartupCategory | Which sector |
| `startup_name` | CharField | Name of the startup |
| `tagline` | CharField | One-liner description |
| `description` | TextField | Full description |
| `target_audience` | TextField | Who it's for |
| `budget_range` | CharField | e.g. "10k-50k" |
| `business_model` | CharField | e.g. "SaaS", "Marketplace" |
| `development_stage` | CharField | idea / validation / prototype / mvp / growth / scaling |
| `team_size` | Integer | Number of people |
| `logo_url` | CharField | Uploaded logo file path |
| `pitch_deck_url` | CharField | Uploaded PDF path |
| `domain_data` | JSONField | Industry-specific extra data |
| `analysis_status` | CharField | Pipeline progress tracker |
| `is_deleted` | Boolean | Soft-delete flag |

**Analysis Status Flow:**
```
pending → processing → classified → market_analyzed → scored → complete
                                                             ↘ failed
```

---

### `backend/ideas/views.py`

**What it is:** CRUD API for ideas, plus analysis triggers.

| View | Method | URL | What it does |
|------|--------|-----|-------------|
| `IdeaListCreateView` | GET | `/api/v1/ideas/` | Paginated list of your ideas |
| `IdeaListCreateView` | POST | `/api/v1/ideas/` | Submit new idea + trigger analysis |
| `IdeaDetailView` | GET | `/api/v1/ideas/<id>/` | Full idea detail |
| `IdeaDetailView` | PATCH | `/api/v1/ideas/<id>/` | Update idea fields |
| `IdeaDetailView` | DELETE | `/api/v1/ideas/<id>/` | Soft-delete idea |
| `IdeaStatusView` | GET | `/api/v1/ideas/<id>/status/` | Poll analysis progress |
| `IdeaAnalysisView` | GET | `/api/v1/ideas/<id>/analysis/` | Full analysis bundle |
| `IdeaReanalyzeView` | POST | `/api/v1/ideas/<id>/reanalyze/` | Reset and re-run analysis |

**On POST (create idea):**
1. Saves the idea to the database
2. Appends submission data to the industry CSV file (for ML training)
3. Calls `analyze_idea.delay(idea_id)` — queues the analysis as a Celery background task
4. Writes an audit log entry

---

### `backend/ideas/serializers.py`

**What it is:** Validates idea data on create/update.

- `IdeaCreateSerializer` — validates new idea submission fields
- `IdeaUpdateSerializer` — validates partial update (blocks edit when analysis is complete)
- `IdeaSerializer` — read-only representation for API responses
- `IdeaStatusSerializer` — returns just the status field

---

### `backend/ideas/urls.py`

**What it is:** URL patterns for the ideas app.

```
GET/POST  ideas/                    → IdeaListCreateView
GET/PATCH/DELETE  ideas/<uuid:pk>/  → IdeaDetailView
GET  ideas/<uuid:pk>/status/        → IdeaStatusView
GET  ideas/<uuid:pk>/analysis/      → IdeaAnalysisView
POST ideas/<uuid:pk>/reanalyze/     → IdeaReanalyzeView
```

---

## 6. Backend — Analysis App


### `backend/analysis/models.py`

**What it is:** All AI analysis result tables. Every model links back to one `Idea` via a foreign key.

**Models:**

#### `NLPOutput` (1:1 per Idea)
Natural language processing results.
- `tokens` — list of processed word tokens
- `keywords` — keywords with relevance scores `[{word, score}]`
- `keyword_vector` — float array for similarity matching
- `industry_label` — detected industry (e.g. "HealthTech")
- `industry_confidence` — confidence score 0.0–1.0
- `technology_labels` — up to 3 detected technology tags
- `similar_startups` — similar companies found `[{name, similarity, source}]`
- `quality_warning` — True if idea description is too short/vague

#### `InnovationScore` (1:1 per Idea)
5-dimension innovation scoring, all values 0–100.

| Dimension | What it measures |
|-----------|-----------------|
| `novelty` | How new/unique is the idea |
| `practicality` | How feasible to implement |
| `scalability` | Can it grow large |
| `business_value` | Commercial potential |
| `technology_adoption` | Ease of adoption |
| `composite_score` | Weighted average of all 5 |

Also stores `explanations` (text rationale per dimension) and `confidence_intervals`.

#### `MarketIntelligence` (1:1 per Idea)
- `market_opportunity_score` — 0–100
- `competition_level` — low / medium / high
- `demand_analysis` — text description of market demand
- `growth_potential` — low / moderate / high / very_high

#### `RiskAssessment` (1:1 per Idea)
Six risk scores, all 0–100:
- `financial_risk`, `technical_risk`, `operational_risk`
- `market_risk`, `legal_risk`, `overall_risk_score`
- `rationales` — JSON object with text explanation per risk
- `mitigation_recs` — list of risk mitigation recommendations

#### `SWOTAnalysis` (1:1 per Idea)
- `strengths`, `weaknesses`, `opportunities`, `threats` — each a JSON list
- Minimum 3 items required per field (validated in `clean()`)
- `is_customized` — True if the user edited the AI-generated SWOT

#### `Recommendation` (1:1 per Idea)
- `revenue_model` — subscription / freemium / marketplace / commission / licensing / advertising
- `marketing_tactics` — list of marketing strategies
- `acquisition_channels` — list of customer acquisition channels
- `improvement_suggestions` — list of suggestions to improve the idea

#### `CustomerPersona` (Many per Idea)
Multiple buyer personas. Each has:
- `age_range`, `occupation`, `income_range`, `location`
- `pain_points` — list of problems this persona faces
- `buying_behaviour` — text description

#### `BusinessModelCanvas` (1:1 per Idea)
9 standard BMC sections, each stored as a JSON list:
- Customer Segments, Value Proposition, Channels
- Customer Relationships, Revenue Streams, Key Activities
- Key Resources, Key Partners, Cost Structure
- `is_customized` — True if user edited it

#### `CompetitorResult` (Many per Idea)
One record per competitor found.
- `competitor_name`, `similarity_score` (0–1)
- `strengths`, `weaknesses` — JSON lists
- `competition_level` — low / medium / high
- `sort_order` — display order

#### `RoadmapPlan` (1:1 per Idea)
- `phases` — JSON array of phases, each with `start_month`, `end_month`, and `milestones`

#### `TeamRecommendation` (1:1 per Idea)
- `roles` — JSON list of `{role, priority, justification}`

#### `InvestorReadiness` (1:1 per Idea)
- `readiness_score` — 0–100
- `readiness_band` — not_ready / developing / ready
- `missing_requirements` — what's needed before investor approach
- `investment_suggestions` — specific actions to improve readiness

#### `IdeaComparison` (Many per User)
Side-by-side comparison of two ideas.
- `idea_a`, `idea_b` — must be different ideas
- `dimension_scores` — JSON comparison per scoring dimension
- `summary_text` — AI-generated comparison text
- `recommendation` — idea_a / idea_b / equal

---

### `backend/analysis/tasks.py`

**What it is:** Celery background tasks that run the full analysis pipeline.

**Main task — `analyze_idea(idea_id)`:**
1. Sets `analysis_status = processing`
2. Runs NLP (tokenization, keyword extraction, industry classification)
3. Runs scoring (innovation score calculation)
4. Runs market intelligence analysis
5. Runs risk assessment
6. Generates SWOT, recommendations, competitor results, personas, BMC
7. Sets `analysis_status = complete`
8. On any error → sets `analysis_status = failed`

---

### `backend/analysis/urls.py` and `backend/analysis/comparison_urls.py`

**What they are:** URL patterns for fetching analysis results and running comparisons.

---

## 7. Backend — Other Apps


### `backend/scoring/models.py`

**What it is:** Stores admin-configurable scoring weights.

**`ScoreWeightConfig` model:**
- 5 weight fields: `novelty_weight`, `practicality_weight`, `scalability_weight`, `business_value_weight`, `tech_adoption_weight`
- Default: 0.200 each (equal weighting)
- Validation: all 5 weights **must sum to exactly 1.000** (enforced in `save()`)
- `is_active` — only one config should be active at a time
- `updated_by` — FK to the admin who last changed it

---

### `backend/mentor_chat/models.py`

**What it is:** Database tables for the AI mentor chat feature.

**`ChatSession` model:**
- `user` — who started the chat
- `idea` — optionally linked to a specific idea (for context-aware answers)
- `title` — auto-generated or user-set session name

**`ChatMessage` model:**
- `session` — which conversation it belongs to
- `role` — `user` or `assistant`
- `content` — the message text
- `created_at` — timestamp (used to sort messages in order)

---

### `backend/reports/models.py`

**What it is:** Tables for generated documents.

**`Report` model:**
- PDF analysis report for an idea
- `status`: `generating → complete / failed`
- `file_url` — where the PDF is stored after generation

**`PitchDeck` model:**
- Structured pitch deck with 8 sections stored as JSON:
  - problem, solution, market_opportunity, business_model
  - competitors, revenue_model, roadmap, funding_requirement
- `is_customized` — True if user edited the AI-generated content
- `file_url` — exported PDF path

---

### `backend/analytics/`

**What it is:** Provides platform-wide statistics for admins and mentors.

- Total ideas submitted, ideas by status, ideas by industry
- User registration trends
- Analysis completion rates
- Exposed at `/api/v1/analytics/`

---

### `backend/notifications/`

**What it is:** User notification system.

- Creates notifications when analysis completes, reports are ready, etc.
- Exposed at `/api/v1/notifications/`
- Admin can broadcast notifications via admin panel

---

### `backend/admin_panel/`

**What it is:** Admin-only management APIs, separate from Django's built-in admin.

- Manage users (view, activate/deactivate, change roles)
- Manage startup categories
- Configure score weights
- View audit logs
- Manage ML models
- Send notifications
- Exposed at `/api/v1/admin-panel/`

---

### `backend/nlp_engine/`

**What it is:** Core NLP processing logic.

- Text tokenization and cleaning
- Keyword extraction using TF-IDF
- Industry classification using trained ML model
- Similar startup detection using cosine similarity
- Used internally by the `analysis` app tasks

---

### `backend/ml_pipeline/`

**What it is:** Machine learning model training pipeline.

- Scripts to retrain the industry classifier on new data
- Loads CSV data from `ml_models/data/raw/by-industry/`
- New idea submissions are automatically appended to these CSVs

---

## 7a. Backend — ml_models Folder (Detailed)

The `ml_models/` folder is the heart of VentureIQ's AI intelligence. It contains all the trained machine learning models, the data used to train them, and the scripts to retrain them.

```
ml_models/
├── pipeline.py                  ← Master orchestration script
├── train_models.py              ← Training script (original)
├── train_models_real.py         ← Training script (real Kaggle data)
├── validate_models.py           ← Validates all trained models
├── features.py                  ← Feature extraction for inference
├── features_real.py             ← Feature extraction (real data version)
├── generate_dataset.py          ← Generates synthetic training data
├── build_competitor_corpus.py   ← Builds competitor lookup JSON
├── rebuild_training_arrays.py   ← Rebuilds NumPy training arrays
├── diagnose.py                  ← Debug/diagnostics tool
├── test_pipeline.py             ← End-to-end integration test
├── test_different_ideas.py      ← Tests model on sample ideas
├── verify_downloads.py          ← Checks Kaggle downloads are complete
│
├── data/
│   ├── competitor_corpus.json   ← Built competitor database (runtime lookup)
│   ├── raw/
│   │   ├── by-industry/         ← One CSV per industry (live + training data)
│   │   │   ├── Agriculture.csv
│   │   │   ├── AI_ML.csv
│   │   │   ├── Cybersecurity.csv
│   │   │   ├── FoodTech.csv
│   │   │   ├── Healthcare.csv
│   │   │   ├── Social_Media.csv
│   │   │   ├── Technology.csv
│   │   │   └── TravelTech.csv
│   │   └── news-category/       ← News articles for industry classifier
│   └── processed/
│       ├── startup_success.csv
│       ├── bankruptcy_risk.csv
│       ├── crunchbase_features.csv
│       ├── indian_funding.csv
│       ├── industry_text_classification.csv
│       ├── unicorn_features.csv
│       ├── training_arrays/     ← NumPy .npy arrays for fast training
│       └── artifacts/           ← Saved vectorizers and label maps
│
├── trained/                     ← All trained model files (.pkl)
│   ├── novelty.pkl
│   ├── practicality.pkl
│   ├── scalability.pkl
│   ├── business_value.pkl
│   ├── technology_adoption.pkl
│   ├── market_opportunity_score.pkl
│   ├── financial_risk.pkl
│   ├── technical_risk.pkl
│   ├── operational_risk.pkl
│   ├── market_risk.pkl
│   ├── legal_risk.pkl
│   ├── investor_readiness.pkl
│   ├── industry_classifier.pkl  ← LogisticRegression text classifier
│   ├── tfidf_vectorizer.pkl     ← TF-IDF vectorizer for text
│   └── industry_label_map.pkl   ← Label ID → industry name mapping
│
├── preprocess/                  ← Data cleaning scripts
│   ├── run_all.py               ← Runs all preprocessors
│   ├── startup_success.py       ← Cleans startup success dataset
│   ├── bankruptcy.py            ← Cleans bankruptcy/risk dataset
│   ├── crunchbase.py            ← Cleans Crunchbase funding data
│   ├── indian_funding.py        ← Cleans Indian startup funding data
│   ├── news_category.py         ← Cleans news articles for classifier
│   └── unicorn_startups.py      ← Cleans unicorn valuation data
│
└── feature_engineering/         ← Feature extraction for training
    ├── build_all.py             ← Runs all feature builders
    ├── innovation_features.py   ← Features for novelty/scalability/etc.
    ├── market_features.py       ← Features for market opportunity
    ├── risk_features.py         ← Features for financial/market/tech risk
    ├── investor_features.py     ← Features for investor readiness
    └── industry_classifier_features.py ← TF-IDF text features
```

---

### Trained Models — `ml_models/trained/`

There are **15 trained model files** (`.pkl` = Python pickle, a serialized trained model):

**Innovation Scoring Models** (each predicts a score 0–100):

| File | What it predicts | Data source | Algorithm | R² score |
|------|-----------------|-------------|-----------|----------|
| `novelty.pkl` | How unique/new the idea is | Crunchbase (48K startups) | GradientBoosting | 0.72 |
| `practicality.pkl` | How feasible to build | Crunchbase | GradientBoosting | 0.87 |
| `scalability.pkl` | How much it can grow | Crunchbase | GradientBoosting | 1.00 |
| `business_value.pkl` | Commercial potential | Crunchbase | GradientBoosting | 0.96 |
| `technology_adoption.pkl` | Ease of tech adoption | Crunchbase | GradientBoosting | 1.00 |

**Market Model:**

| File | What it predicts | Data source | Algorithm | R² score |
|------|-----------------|-------------|-----------|----------|
| `market_opportunity_score.pkl` | Market size & timing | Crunchbase | GradientBoosting | 0.97 |

**Risk Models** (each predicts a risk score 0–100):

| File | What it predicts | Data source | Algorithm | R² score |
|------|-----------------|-------------|-----------|----------|
| `financial_risk.pkl` | Financial risk | Bankruptcy dataset (6.8K) | GradientBoosting | 0.95 |
| `technical_risk.pkl` | Technical risk | Bankruptcy dataset | GradientBoosting | 0.95 |
| `operational_risk.pkl` | Operational risk | Bankruptcy dataset | GradientBoosting | 0.95 |
| `market_risk.pkl` | Market risk | Bankruptcy dataset | GradientBoosting | 0.94 |
| `legal_risk.pkl` | Legal/compliance risk | Bankruptcy dataset | GradientBoosting | 0.87 |

**Other Models:**

| File | What it predicts | Data source | Algorithm | Performance |
|------|-----------------|-------------|-----------|-------------|
| `investor_readiness.pkl` | Investor-readiness score | Crunchbase | GradientBoosting | R²=0.97 |
| `industry_classifier.pkl` | Which industry the idea belongs to | News articles (50K) | LogisticRegression | Acc=70% |
| `tfidf_vectorizer.pkl` | Converts text to numbers for classifier | — | TF-IDF | — |
| `industry_label_map.pkl` | Maps label IDs to industry names | — | — | — |

> **R² score** = how well the model explains variance (1.0 = perfect, 0 = no better than average). **MAE** threshold ≤ 18 points is required for passing validation.

---

### `pipeline.py`

**What it is:** Master orchestration script that runs the entire ML training pipeline end-to-end.

**5 steps it runs:**
1. **Download** — downloads 6 Kaggle datasets (needs `KAGGLE_API_TOKEN`)
2. **Preprocess** — cleans raw CSV data → `data/processed/`
3. **Features** — extracts numeric features → `data/processed/training_arrays/`
4. **Train** — trains all 13 models → saves `.pkl` to `trained/`
5. **Validate** — checks all models meet R² ≥ 0.25, MAE ≤ 18 thresholds

**How to run:**
```bash
# Full pipeline (downloads Kaggle data):
python -m ml_models.pipeline --full

# Skip download (use existing data):
python -m ml_models.pipeline

# Run one step only:
python -m ml_models.pipeline --step train
python -m ml_models.pipeline --step validate
```

---

### `train_models.py`

**What it is:** Trains all 12 regression models using `GradientBoostingRegressor`.

**What it does:**
- Loads `training_data.json` (or generates it if missing)
- For each of 12 target dimensions: splits data 80/20, fits a GradientBoostingRegressor with 100 estimators
- Evaluates with MAE and R² on the test split
- Saves each trained model as `trained/<target_name>.pkl`

**Models trained:** novelty, practicality, scalability, business_value, technology_adoption, market_opportunity_score, financial_risk, technical_risk, operational_risk, market_risk, legal_risk, investor_readiness

---

### `validate_models.py`

**What it is:** Quality gate that checks all trained models meet minimum performance thresholds before they are used in production.

**Thresholds:**
- Regression models: R² ≥ 0.25, MAE ≤ 18.0
- Industry classifier: Accuracy ≥ 65%

**What it checks:**
1. `.pkl` file exists for every model
2. Model loads without errors
3. Model can make predictions
4. Performance metrics meet thresholds

**Exit codes:** `0` = all passed, `1` = one or more failed

---

### `features.py`

**What it is:** Converts a submitted idea into a fixed-size numeric feature vector used by all scoring models at inference (prediction) time.

**Features extracted (23 total):**

| Feature | Source | Description |
|---------|--------|-------------|
| `description_length` | Idea text | Character count |
| `word_count` | Idea text | Word count |
| `no_quality_warning` | NLP output | 1 if description is good quality |
| `industry_confidence` | NLP output | How confident the classifier is |
| `n_tech_labels` | NLP output | Number of tech tags detected |
| `has_ai_ml` | NLP output | 1 if AI/ML detected in idea |
| `n_keywords` | NLP output | Number of extracted keywords |
| `avg_keyword_score` | NLP output | Average relevance of keywords |
| `n_similar_startups` | NLP output | Similar companies found |
| `max_similarity_score` | NLP output | Highest similarity score found |
| `team_size` | Idea metadata | Number of team members |
| `dev_stage_norm` | Idea metadata | Stage encoded 0–1 (idea=0, scaling=1) |
| `business_model_norm` | Idea metadata | Business model encoded 0–1 |
| `has_pitch_deck` | Idea metadata | 1 if pitch deck was uploaded |
| `industry_*` (9 cols) | NLP output | One-hot encoding for each industry |

---

### `build_competitor_corpus.py`

**What it is:** Builds the `data/competitor_corpus.json` file used at runtime by the competitor analyzer.

**What it does:**
- Reads all 8 by-industry CSVs
- Filters to active and acquired companies only
- Selects top 10 companies per industry (by investor score)
- For each company generates:
  - A 20-dimension keyword vector (for similarity matching)
  - Derived `strengths` (based on funding, stage, business model)
  - Derived `weaknesses` (based on risk, status, industry)
- Saves everything to `competitor_corpus.json`

**When to re-run:** After adding new companies to the by-industry CSVs.
```bash
python -m ml_models.build_competitor_corpus
```

---

### `data/raw/by-industry/` CSVs

**What they are:** 8 CSV files, one per supported industry. These serve two purposes:

1. **Training data** — used to train and retrain ML models
2. **Competitor database** — used at runtime to find similar companies

| File | Industry |
|------|----------|
| `Agriculture.csv` | AgriTech startups |
| `AI_ML.csv` | Artificial Intelligence / Machine Learning |
| `Cybersecurity.csv` | Cybersecurity companies |
| `FoodTech.csv` | Food technology |
| `Healthcare.csv` | HealthTech startups |
| `Social_Media.csv` | Social media platforms |
| `Technology.csv` | General technology |
| `TravelTech.csv` | Travel technology |

**Live data feed:** Every time a user submits a new idea via the platform, a row is automatically appended to the matching CSV (done in `ideas/views.py → _append_to_industry_csv()`). This means the training data grows over time.

---

### `data/processed/` Files

Cleaned and feature-engineered versions of the raw Kaggle datasets, ready for model training.

| File | Source | Records | Used for |
|------|--------|---------|---------|
| `startup_success.csv` | Kaggle: Startup Success | 923 | Success/failure patterns |
| `bankruptcy_risk.csv` | Kaggle: Bankruptcy Prediction | 6,819 | Risk scoring (95 financial ratios) |
| `crunchbase_features.csv` | Kaggle: Crunchbase | 54,294 | Market, innovation, investor readiness |
| `indian_funding.csv` | Kaggle: Indian Startup Funding | 3,044 | Industry + funding patterns |
| `industry_text_classification.csv` | Kaggle: News Category | 50,000 | Industry text classifier training |
| `unicorn_features.csv` | Kaggle: Unicorn Startups | 1,186 | Valuation benchmarks |

---

### `data/competitor_corpus.json`

**What it is:** A pre-built JSON database of Indian startups used at runtime by the competitor analyzer. Generated by `build_competitor_corpus.py`.

**Structure:**
```json
[
  {
    "name": "Ola",
    "vector": [0.1234, 0.5678, ...],  // 20-dim keyword vector
    "metadata": {
      "category": "TravelTech",
      "city": "Bangalore",
      "stage": "scaling",
      "funding_inr": 5000000000,
      "strengths": ["Strong funding base", "Proven traction"],
      "weaknesses": ["High capital dependency"]
    }
  }
]
```

When a user submits a new idea, the competitor analyzer computes cosine similarity between the idea's keyword vector and every entry in this corpus, returning the top matches.

---

### `preprocess/` Scripts

These scripts clean the raw Kaggle CSV files into consistent formats for feature engineering.

| File | What it cleans |
|------|---------------|
| `startup_success.py` | Startup success/failure dataset — encodes categorical fields, handles missing values |
| `bankruptcy.py` | Company bankruptcy data — normalizes 95 financial ratios |
| `crunchbase.py` | Crunchbase investments — extracts funding amounts, categories, stages |
| `indian_funding.py` | Indian startup funding — extracts INR amounts, industry mappings |
| `news_category.py` | News articles — filters to tech/business categories, maps to VentureIQ industries |
| `unicorn_startups.py` | Unicorn data — extracts valuation, country, industry |
| `run_all.py` | Runs all preprocessors above in sequence |

---

### `feature_engineering/` Scripts

These scripts take the cleaned data and create numeric feature arrays ready for scikit-learn model training.

| File | Features it builds |
|------|-------------------|
| `innovation_features.py` | Features for novelty, practicality, scalability, business_value, technology_adoption |
| `market_features.py` | Features for market_opportunity_score |
| `risk_features.py` | Features for all 5 risk dimensions (uses 95 bankruptcy financial ratios) |
| `investor_features.py` | Features for investor_readiness |
| `industry_classifier_features.py` | TF-IDF text features for industry classification |
| `build_all.py` | Runs all feature builders and saves `.npy` arrays to `training_arrays/` |

---

### How the ML Models Are Used at Runtime

When a user submits an idea, this is what happens behind the scenes:

```
Idea submitted
    ↓
analyze_idea.delay() — Celery background task starts
    ↓
nlp_engine/engine.py
  → tfidf_vectorizer.pkl + industry_classifier.pkl
  → predicts industry label (e.g. "HealthTech")
  → extracts keywords, finds similar startups from corpus
    ↓
features.py → extract_features()
  → builds a 23-element numeric vector from the idea
    ↓
scoring/innovation.py
  → novelty.pkl, practicality.pkl, scalability.pkl,
    business_value.pkl, technology_adoption.pkl
  → predicts 5 scores → calculates composite_score
    ↓
scoring/market.py
  → market_opportunity_score.pkl → predicts market score
    ↓
scoring/risk.py
  → financial_risk.pkl, technical_risk.pkl, operational_risk.pkl,
    market_risk.pkl, legal_risk.pkl → predicts 5 risk scores
    ↓
scoring/investor_readiness.py
  → investor_readiness.pkl → predicts readiness score
    ↓
All results saved to analysis models in the database
Idea.analysis_status = "complete"
```

> **Fallback:** All scoring engines have rule-based fallback logic. If a `.pkl` file fails to load, the engine computes scores using weighted formulas instead of the ML model — so the app never crashes even if models are missing.

---

### `backend/competitor/`

**What it is:** Competitor analysis logic.

- Searches for similar companies based on idea description and industry
- Calculates similarity scores
- Returns top N competitors with strengths/weaknesses
- Results stored in `CompetitorResult` model

---

### `backend/generators/`

**What it is:** AI content generators.

- Generates SWOT analysis text
- Generates Business Model Canvas sections
- Generates pitch deck content
- Uses Gemini or Groq (Llama 3) AI APIs

---

## 8. Frontend — Entry Point & Config


### `frontend/package.json`

**What it is:** Defines the project name, scripts, and all JavaScript dependencies.

**Key dependencies:**

| Package | Version | Purpose |
|---------|---------|---------|
| `react` | 18.3.1 | UI component framework |
| `react-dom` | 18.3.1 | Renders React to the browser |
| `react-router-dom` | 6.30.1 | Client-side page routing |
| `@tanstack/react-query` | 5.83.0 | Server state, caching, API calls |
| `axios` | 1.9.0 | HTTP client for API requests |
| `recharts` | 2.15.3 | Charts and graphs |
| `@headlessui/react` | 2.2.4 | Accessible UI components (modals, etc.) |
| `tailwindcss` | 3.4.17 | Utility-first CSS framework |
| `vite` | 6.3.5 | Build tool and dev server |

**Scripts:**
```bash
npm run dev      # Start development server (localhost:5173)
npm run build    # Build for production (outputs to dist/)
npm run preview  # Preview the production build locally
```

---

### `frontend/vite.config.js`

**What it is:** Configuration for Vite (the build tool). Sets up the React plugin and optionally configures the dev proxy to forward API calls to the Django backend.

---

### `frontend/tailwind.config.js`

**What it is:** Tailwind CSS configuration. Tells Tailwind which files to scan for class names so unused CSS is removed in the production build.

---

### `frontend/index.html`

**What it is:** The single HTML file that the browser loads. Contains one `<div id="root">` where the entire React app is injected. All other pages are rendered by React — the server only ever serves this one HTML file.

---

### `frontend/src/main.jsx`

**What it is:** The JavaScript entry point for the app. Bootstraps everything.

**What it sets up:**
- `BrowserRouter` — enables URL-based navigation
- `QueryClientProvider` — React Query with 5-minute cache and 1 retry on failure
- `AuthProvider` — global authentication state (current user, login/logout)
- `ErrorBoundary` — catches any React crashes and shows a friendly error with a "Try Again" button

---

### `frontend/src/App.jsx`

**What it is:** Defines all URL routes and which component renders at each path.

**Public routes** (no login needed):

| Path | Component |
|------|-----------|
| `/` | `LandingPage` |
| `/login` | `LoginPage` |
| `/register` | `RegisterPage` |
| `/forgot-password` | `ForgotPasswordPage` |
| `/reset-password/:token` | `ResetPasswordPage` |

**Protected routes** (require login via `AuthGuard`):

| Path | Component |
|------|-----------|
| `/dashboard` | `DashboardPage` |
| `/ideas` | `IdeaListPage` |
| `/ideas/new` | `IdeaSubmissionPage` |
| `/ideas/:id` | `IdeaDetailPage` |
| `/ideas/:id/analysis/*` | `AnalysisTabsPage` (with nested tabs) |
| `/compare` | `ComparisonPage` |
| `/reports` | `ReportsPage` |
| `/pitch-decks/:id` | `PitchDeckEditorPage` |
| `/mentor` | `MentorChatPage` |
| `/analytics` | `AnalyticsDashboardPage` (admin/mentor only) |
| `/admin/*` | `AdminPanelPage` (admin only) |
| `/profile` | `ProfilePage` |
| `/submission-history` | `SubmissionHistoryPage` |

---

### `frontend/src/context/AuthContext.jsx`

**What it is:** React Context that stores and provides the current user's authentication state to the entire app.

- Stores: `currentUser`, `accessToken`, `refreshToken`
- Provides: `login()`, `logout()`, `register()` functions
- On app load: reads token from `localStorage` to keep users logged in after refresh
- On 401 response: automatically attempts token refresh; logs out if refresh fails

---

### `frontend/src/api/`

**What it is:** Axios API client functions organized by feature.

- `auth.js` — login, register, logout, password reset calls
- `ideas.js` — create, list, fetch, delete idea calls
- `analysis.js` — fetch analysis results
- `mentor.js` — chat API calls
- `reports.js` — generate and download reports

All calls go to `http://localhost:8000/api/v1/` in development.

---

## 9. Frontend — Pages


### `LandingPage.jsx`
The public home/marketing page. Shown to visitors who are not logged in. Contains hero section, feature highlights, and call-to-action buttons to register or log in.

---

### `LoginPage.jsx`
Email + password login form.
- Calls `POST /api/v1/auth/login/`
- On success: stores JWT tokens, redirects to `/dashboard`
- Shows error for wrong credentials
- Links to `/forgot-password` and `/register`

---

### `RegisterPage.jsx`
New account registration form.
- Fields: `full_name`, `email`, `password`, `role`
- Calls `POST /api/v1/auth/register/`
- On success: logs in automatically and redirects to `/dashboard`

---

### `ForgotPasswordPage.jsx`
Enter email address to receive a password reset link.
- Calls `POST /api/v1/auth/password/forgot/`
- Always shows a success message (even if email doesn't exist — to prevent user enumeration)

---

### `ResetPasswordPage.jsx`
Enter new password using the token from the email link.
- Reads `token` from URL params (e.g. `/reset-password/abc123`)
- Calls `POST /api/v1/auth/password/reset/`
- On success: redirects to `/login`

---

### `DashboardPage.jsx`
The main screen after login. Shows:
- Summary stats (total ideas, ideas by status)
- Recent idea submissions
- Quick action buttons (Submit Idea, View Reports)
- Notification indicators

---

### `IdeaListPage.jsx`
Paginated list of all ideas submitted by the logged-in user.
- Each idea shows: name, industry, status badge, created date
- Clicking an idea goes to `IdeaDetailPage`
- "New Idea" button links to `IdeaSubmissionPage`

---

### `IdeaSubmissionPage.jsx`
Multi-field form to submit a new startup idea.
- Fields: startup name, tagline, description, target audience, industry, budget, business model, development stage, team size
- File uploads: logo (PNG/JPG, max 5MB), pitch deck (PDF, max 20MB)
- Industry-specific extra fields rendered dynamically based on selected category
- On submit: calls `POST /api/v1/ideas/`, then polls `/status/` until analysis begins

---

### `IdeaDetailPage.jsx`
Full detail view of a single idea.
- Shows all submitted fields
- Shows current `analysis_status` with a progress indicator
- Links to the analysis tabs when status is `complete`
- Edit and delete buttons (soft-delete)

---

### `AnalysisTabsPage.jsx`
Container page for all analysis results. Renders a tab navigation bar with 8 tabs. Each tab is a separate nested route component (see Section 10).

---

### `ComparisonPage.jsx`
Side-by-side comparison of two ideas.
- Lets user select any two of their own ideas
- Calls the comparison API
- Shows dimension-by-dimension score comparison
- Displays AI-generated recommendation (idea A / idea B / equal)

---

### `ReportsPage.jsx`
PDF report management.
- Lists existing reports for each idea
- "Generate Report" button triggers report creation (async)
- Shows status (generating / complete) with polling
- "Download" button when complete

---

### `PitchDeckEditorPage.jsx`
Edit and export the AI-generated pitch deck.
- Shows 8 editable sections: Problem, Solution, Market Opportunity, Business Model, Competitors, Revenue Model, Roadmap, Funding Requirement
- User can customize any section
- "Export PDF" button generates and downloads the pitch deck

---

### `MentorChatPage.jsx`
AI mentor chat interface.
- Left panel: list of chat sessions
- Right panel: message thread for the selected session
- Messages have role badges: "You" vs "Mentor"
- Can optionally link a session to a specific idea for context-aware answers
- Uses Groq (Llama 3) AI model on the backend

---

### `AnalyticsDashboardPage.jsx`
Platform statistics for admins and mentors only.
- Charts: ideas submitted over time, status distribution, industry breakdown
- User registration trends
- Analysis completion rates
- Uses Recharts for visualization

---

### `AdminPanelPage.jsx`
Admin control panel with nested sub-pages (see Section 11).

---

### `ProfilePage.jsx`
View and edit the current user's profile.
- Shows: avatar, name, email, role, bio
- Edit: full_name, avatar_url, bio, theme (light/dark)
- Change password form

---

### `SubmissionHistoryPage.jsx`
Full history of all idea submissions with filter and sort options. Shows all statuses including deleted ideas (soft-deleted).

---

## 10. Frontend — Analysis Sub-tabs

These are nested route components rendered inside `AnalysisTabsPage.jsx`.

| File | Tab Name | What it shows |
|------|----------|--------------|
| `InnovationTab.jsx` | Innovation | Radar/bar chart of 5 innovation dimensions + composite score + explanations |
| `MarketTab.jsx` | Market | Market opportunity score, competition level, demand analysis, growth potential |
| `RiskTab.jsx` | Risk | 5 risk dimension scores + overall risk + mitigation recommendations |
| `RecommendationsTab.jsx` | Recommendations | Revenue model, marketing tactics, acquisition channels, improvement suggestions |
| `CompetitorsTab.jsx` | Competitors | Table of similar companies with similarity scores, strengths, and weaknesses |
| `TeamTab.jsx` | Team | Recommended team roles with priority and justification |
| `SchemesTab.jsx` | Schemes | Relevant government grants and incubation schemes based on industry |
| `ChartsTab.jsx` | Charts | All scores visualized as charts (bar, radar, pie) using Recharts |

---

## 11. Frontend — Admin Sub-pages

These are nested route components rendered inside `AdminPanelPage.jsx`. Only accessible to users with `role = admin`.

| File | Path | Purpose |
|------|------|---------|
| `UsersAdmin.jsx` | `/admin/users` | View all users, change roles, activate/deactivate accounts |
| `CategoriesAdmin.jsx` | `/admin/categories` | Add, edit, or deactivate startup industry categories |
| `ScoreWeightsAdmin.jsx` | `/admin/score-weights` | Adjust the 5 innovation scoring weights (must sum to 1.0) |
| `AuditLogAdmin.jsx` | `/admin/audit-log` | Read-only view of all audit log entries with filters |
| `MLAdmin.jsx` | `/admin/ml` | Trigger ML model retraining, view model accuracy stats |
| `NotificationsAdmin.jsx` | `/admin/notifications` | Compose and broadcast notifications to users |

---

## 12. Frontend — Components

### `frontend/src/components/AuthGuard.jsx`
**What it is:** A wrapper component that protects routes requiring login.
- If user is not logged in → redirects to `/login`
- If `roles` prop is provided → checks user's role; if not authorized → redirects to `/dashboard`
- Usage: `<AuthGuard roles={['admin']}><AdminPanelPage /></AuthGuard>`

---

### `frontend/src/components/Layout.jsx`
**What it is:** The shared page shell used by all authenticated pages.
- Top navigation bar with logo, user menu, notifications bell
- Side navigation with links to all main sections
- Main content area where page components are rendered
- Responsive — collapses sidebar on mobile

---

## Quick Reference: How Everything Connects

```
User opens browser
  → React app loads (index.html → main.jsx → App.jsx)
  → AuthContext checks localStorage for saved JWT token
  → If logged in: shows Dashboard; If not: shows Landing page

User submits an idea (IdeaSubmissionPage)
  → POST /api/v1/ideas/ → ideas/views.py → saves Idea to DB
  → analyze_idea.delay() → Celery picks it up
  → nlp_engine processes text → scoring calculates scores
  → all results saved to analysis models
  → Idea.analysis_status updated to "complete"
  → Frontend polls /status/ → shows "complete" → user clicks Analysis tab

User views analysis (AnalysisTabsPage)
  → GET /api/v1/ideas/<id>/analysis/ → returns all sub-results in one response
  → Each tab reads its slice of data and renders charts/text

Admin logs in
  → AuthGuard checks role === "admin"
  → Admin Panel shows user management, category management, score weights, audit logs
```

---

*This document was generated to help explain the VentureIQ codebase in detail for academic review.*
