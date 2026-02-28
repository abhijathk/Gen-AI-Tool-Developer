# D — Plan Mode Output (Component & Class Breakdown)

**Project:** Async Standup Automation Agent
**Prepared by:** Abhijath Kottikkal
**Date:** February 27, 2026

> **Note:** This document is a build plan only. No code is generated.
> The solution is broken into small, testable components following the order: **API → DB → Web**.

---

## Project Structure (Planned)

```
standup-agent/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI app entry point
│   │   ├── config.py                # App configuration (env vars, settings)
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── team_routes.py       # /api/team endpoints
│   │   │   ├── sprint_routes.py     # /api/sprints endpoints
│   │   │   ├── standup_routes.py    # /api/standups endpoints
│   │   │   ├── summary_routes.py    # /api/summaries endpoints
│   │   │   ├── blocker_routes.py    # /api/blockers endpoints
│   │   │   └── health_routes.py     # /api/sprint/health endpoint
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── team_member.py       # TeamMember SQLAlchemy model
│   │   │   ├── sprint.py            # Sprint SQLAlchemy model
│   │   │   ├── standup.py           # Standup SQLAlchemy model
│   │   │   ├── summary.py           # Summary SQLAlchemy model
│   │   │   └── blocker.py           # Blocker SQLAlchemy model
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── team_schema.py       # Pydantic schemas for team member
│   │   │   ├── sprint_schema.py     # Pydantic schemas for sprint
│   │   │   ├── standup_schema.py    # Pydantic schemas for standup
│   │   │   ├── summary_schema.py    # Pydantic schemas for summary
│   │   │   └── blocker_schema.py    # Pydantic schemas for blocker
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── standup_service.py   # Business logic for standups
│   │   │   ├── summary_service.py   # Summary generation orchestration
│   │   │   ├── blocker_service.py   # Blocker extraction and management
│   │   │   └── health_service.py    # Sprint health computation
│   │   ├── llm/
│   │   │   ├── __init__.py
│   │   │   ├── client.py            # OpenAI API client wrapper
│   │   │   └── prompts.py           # Prompt templates for summarization
│   │   └── db/
│   │       ├── __init__.py
│   │       ├── database.py          # SQLAlchemy engine & session setup
│   │       └── migrations/          # Alembic migration files
│   ├── tests/
│   │   ├── test_team_api.py
│   │   ├── test_standup_api.py
│   │   ├── test_summary_api.py
│   │   ├── test_blocker_api.py
│   │   └── test_health_api.py
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── App.jsx                  # Root component with routing
│   │   ├── pages/
│   │   │   ├── SubmitStandup.jsx    # Standup submission form
│   │   │   ├── DailySummary.jsx     # Summary dashboard page
│   │   │   ├── SprintHealth.jsx     # Sprint health view
│   │   │   └── TeamManagement.jsx   # Team member CRUD page
│   │   ├── components/
│   │   │   ├── StandupForm.jsx      # Reusable standup form component
│   │   │   ├── SummaryCard.jsx      # Summary display card
│   │   │   ├── BlockerList.jsx      # Blocker list with status badges
│   │   │   ├── SentimentBadge.jsx   # Sentiment indicator component
│   │   │   ├── HealthGauge.jsx      # Sprint health visual indicator
│   │   │   ├── NudgePanel.jsx       # Coaching nudges display
│   │   │   └── Navbar.jsx           # Navigation bar
│   │   └── services/
│   │       └── api.js               # Axios API client configuration
│   ├── package.json
│   └── tailwind.config.js
└── README.md
```

---

## Build Order & Component Breakdown

### Phase 1: API Layer (Build First)

The API is built and tested independently before any UI work.

#### Step 1.1 — Project Setup
- **What:** Initialize FastAPI project, install dependencies, configure environment variables
- **Classes/Modules:**
  - `main.py` — FastAPI application instance, CORS middleware, router registration
  - `config.py` — `Settings` class using Pydantic `BaseSettings` to load env vars (DB URL, API key, OpenAI key)
- **Test:** App starts, returns 200 on health check endpoint

#### Step 1.2 — Team Member API
- **What:** CRUD endpoints for team members
- **Classes/Modules:**
  - `team_routes.py` — `POST /api/team`, `GET /api/team`, `GET /api/team/{id}`
  - `team_schema.py` — `TeamMemberCreate` (input), `TeamMemberResponse` (output) Pydantic models
- **Test:** Create a member, list members, retrieve by ID

#### Step 1.3 — Sprint API
- **What:** CRUD endpoints for sprints
- **Classes/Modules:**
  - `sprint_routes.py` — `POST /api/sprints`, `GET /api/sprints/current`, `GET /api/sprints/{id}`
  - `sprint_schema.py` — `SprintCreate`, `SprintResponse` Pydantic models
- **Test:** Create a sprint, retrieve current sprint

#### Step 1.4 — Standup Submission API
- **What:** Submit and retrieve standup updates
- **Classes/Modules:**
  - `standup_routes.py` — `POST /api/standups`, `GET /api/standups?date={date}`
  - `standup_schema.py` — `StandupCreate`, `StandupResponse` Pydantic models
  - `standup_service.py` — `StandupService` class with `create_standup()`, `get_standups_by_date()` methods
- **Test:** Submit a standup, retrieve standups by date, validate required fields

#### Step 1.5 — LLM Integration
- **What:** OpenAI API client and prompt templates for summarization
- **Classes/Modules:**
  - `client.py` — `LLMClient` class with `generate_summary(standups, sprint_context)` method
  - `prompts.py` — `build_summary_prompt(standups, sprint)` function returning the formatted prompt string
- **Test:** Mock LLM call, verify prompt includes all standup data, verify JSON parsing of response

#### Step 1.6 — Summary Generation API
- **What:** Trigger summary generation and retrieve past summaries
- **Classes/Modules:**
  - `summary_routes.py` — `POST /api/summaries/generate`, `GET /api/summaries/{date}`
  - `summary_schema.py` — `SummaryResponse` Pydantic model (includes blockers, sentiment, nudges)
  - `summary_service.py` — `SummaryService` class with `generate_daily_summary(date)` method that:
    1. Fetches all standups for the date
    2. Fetches sprint context
    3. Calls LLM client
    4. Parses response into blockers, sentiment, health, nudges
    5. Stores summary and blockers in DB
- **Test:** Generate summary with mock LLM, verify blockers are extracted and stored

#### Step 1.7 — Blocker API
- **What:** List and manage blockers
- **Classes/Modules:**
  - `blocker_routes.py` — `GET /api/blockers?status=open`, `PATCH /api/blockers/{id}`
  - `blocker_schema.py` — `BlockerResponse`, `BlockerUpdate` Pydantic models
  - `blocker_service.py` — `BlockerService` class with `get_open_blockers()`, `resolve_blocker(id)`, `get_blocker_aging()` methods
- **Test:** List open blockers, resolve a blocker, verify aging calculation

#### Step 1.8 — Sprint Health API
- **What:** Compute and return sprint health score
- **Classes/Modules:**
  - `health_routes.py` — `GET /api/sprint/health`
  - `health_service.py` — `HealthService` class with `compute_health(sprint_id)` method that aggregates:
    - Submission rate (% of team who submitted today)
    - Open blocker count
    - Average sentiment
    - Days remaining vs. work remaining
- **Test:** Verify health score computation with known inputs

---

### Phase 2: DB Layer (Build Alongside API)

Database models and migrations are built as each API endpoint is developed.

#### Step 2.1 — Database Setup
- **What:** SQLAlchemy engine, session factory, base model class
- **Classes/Modules:**
  - `database.py` — `engine`, `SessionLocal`, `Base`, `get_db()` dependency function
- **Config:** SQLite for development (`sqlite:///./standup.db`), PostgreSQL connection string for production

#### Step 2.2 — TeamMember Model
- **What:** SQLAlchemy model for team_members table
- **Class:** `TeamMember` — columns: `id` (UUID), `name`, `role`, `email`, `created_at`

#### Step 2.3 — Sprint Model
- **What:** SQLAlchemy model for sprints table
- **Class:** `Sprint` — columns: `id` (UUID), `name`, `goal`, `start_date`, `end_date`, `created_at`

#### Step 2.4 — Standup Model
- **What:** SQLAlchemy model for standups table
- **Class:** `Standup` — columns: `id` (UUID), `team_member_id` (FK), `sprint_id` (FK), `yesterday`, `today`, `blockers`, `sentiment`, `submitted_at`
- **Relationships:** `team_member` → TeamMember, `sprint` → Sprint

#### Step 2.5 — Summary Model
- **What:** SQLAlchemy model for summaries table
- **Class:** `Summary` — columns: `id` (UUID), `sprint_id` (FK), `date`, `summary_text`, `sprint_health`, `nudges` (JSON), `generated_at`
- **Constraint:** Unique on `(sprint_id, date)` — one summary per sprint per day

#### Step 2.6 — Blocker Model
- **What:** SQLAlchemy model for blockers table
- **Class:** `Blocker` — columns: `id` (UUID), `standup_id` (FK), `description`, `owner_id` (FK), `severity`, `status`, `created_at`, `resolved_at`
- **Relationships:** `standup` → Standup, `owner` → TeamMember

#### Step 2.7 — Migrations
- **What:** Alembic setup for schema versioning
- **Action:** Initialize Alembic, generate initial migration from all models, apply migration

---

### Phase 3: Web Layer (Build Last)

The frontend consumes the fully tested API.

#### Step 3.1 — Project Setup
- **What:** Initialize React app (Vite), install Tailwind CSS, configure Axios base URL
- **Files:** `App.jsx`, `api.js`, `tailwind.config.js`
- **Test:** App renders, Axios connects to backend

#### Step 3.2 — Navigation & Layout
- **What:** Navbar component with links to all pages, responsive layout wrapper
- **Component:** `Navbar.jsx`
- **Routes:** `/submit`, `/summary`, `/health`, `/team`

#### Step 3.3 — Team Management Page
- **What:** Simple page to add team members and view existing members
- **Components:** `TeamManagement.jsx` (page) — form + table
- **API Calls:** `GET /api/team`, `POST /api/team`
- **Test:** Add a member via form, verify it appears in table

#### Step 3.4 — Submit Standup Page
- **What:** Form with team member dropdown, yesterday/today/blockers text areas, submit button
- **Components:** `SubmitStandup.jsx` (page), `StandupForm.jsx` (reusable form)
- **API Calls:** `GET /api/team` (populate dropdown), `POST /api/standups`
- **Test:** Submit a standup, verify success toast, fields cleared

#### Step 3.5 — Daily Summary Page
- **What:** Display generated summary, blocker list, sentiment indicators, nudges
- **Components:** `DailySummary.jsx` (page), `SummaryCard.jsx`, `BlockerList.jsx`, `SentimentBadge.jsx`, `NudgePanel.jsx`
- **API Calls:** `POST /api/summaries/generate`, `GET /api/summaries/{date}`
- **Test:** Generate summary, verify all sections render with data

#### Step 3.6 — Sprint Health Page
- **What:** Visual sprint health display with color-coded gauge, stats cards
- **Components:** `SprintHealth.jsx` (page), `HealthGauge.jsx`
- **API Calls:** `GET /api/sprint/health`
- **Test:** Health page renders with correct color coding for each status

---

## Class/Module Summary Table

| Module | Class/Function | Responsibility |
|--------|---------------|----------------|
| `main.py` | `app` (FastAPI instance) | Application entry point, middleware, router registration |
| `config.py` | `Settings` | Environment variable management |
| `database.py` | `engine`, `SessionLocal`, `get_db()` | Database connection and session management |
| `team_member.py` | `TeamMember` | ORM model for team members |
| `sprint.py` | `Sprint` | ORM model for sprints |
| `standup.py` | `Standup` | ORM model for standup entries |
| `summary.py` | `Summary` | ORM model for generated summaries |
| `blocker.py` | `Blocker` | ORM model for extracted blockers |
| `standup_service.py` | `StandupService` | Create/retrieve standup entries |
| `summary_service.py` | `SummaryService` | Orchestrate LLM call, parse results, store summary + blockers |
| `blocker_service.py` | `BlockerService` | Query/update blockers, compute aging |
| `health_service.py` | `HealthService` | Compute sprint health from aggregated data |
| `client.py` | `LLMClient` | Wrapper around OpenAI API for summary generation |
| `prompts.py` | `build_summary_prompt()` | Construct the prompt sent to the LLM |
| `api.js` | `apiClient` | Axios instance configured with base URL and headers |

---

## Testing Strategy

| Phase | Test Type | Tool | What Is Tested |
|-------|-----------|------|----------------|
| API | Unit tests | pytest | Each service class method in isolation with mocked DB |
| API | Integration tests | pytest + httpx | Full request → response cycle for each endpoint |
| LLM | Unit tests | pytest + mock | Prompt construction and response parsing (mocked API) |
| DB | Migration tests | Alembic | Schema applies cleanly, rollback works |
| Web | Manual testing | Browser | Form submissions, dashboard rendering, navigation |

---

## Dependencies (Planned)

### Backend (`requirements.txt`)
```
fastapi
uvicorn
sqlalchemy
alembic
pydantic
pydantic-settings
openai
python-dotenv
httpx
pytest
```

### Frontend (`package.json`)
```
react
react-dom
react-router-dom
axios
tailwindcss
@tailwindcss/forms
```

---

## Summary

This plan breaks the Async Standup Automation Agent into **18 discrete, testable steps** across three phases:

- **Phase 1 (API):** 8 steps — build and test all REST endpoints with LLM integration
- **Phase 2 (DB):** 7 steps — build ORM models and migrations alongside API development
- **Phase 3 (Web):** 6 steps — build React frontend consuming the tested API

Each step produces a working, testable increment. No step depends on complex infrastructure. The entire solution runs as a single FastAPI process + a React dev server.

---

## Assignment Requirement Mapping (Section D)

- **Copilot Plan mode output:** This document is plan-only and intentionally contains no implementation code.
- **Break into smaller components/classes:** Covered through module/class breakdown and step-by-step service/model/component definitions.
- **Required order followed:** API first (Phase 1), DB second (Phase 2), Web third (Phase 3).
- **Design guidelines followed:** Plan is simple, incrementally testable, and avoids complex architecture.
