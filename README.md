# AI-Guided Academic Project Progress Tracking Platform with Planning & Mentorship Assistance

A full-stack platform that helps student teams take a project idea from a rough concept to a finished submission — with a chain of AI agents that judge feasibility, define scope, recommend a tech stack, build a timeline, track progress, and prep teams for their viva, backed by a persistent AI mentor a student can actually talk to.

Built for the realities of a student project: flaky campus wifi, missing inputs at submission time, and the need for a faculty member to see real signal (not just AI output) across an entire class of teams.

## Table of contents

- [Overview](#overview)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Getting started](#getting-started)
- [Project structure](#project-structure)
- [License](#license)

## Overview

Students submit a project idea and get it run through a chain of specialized AI agents — each one reading the previous agent's output rather than starting from scratch — that assess feasibility, lock down scope, suggest technology, and lay out a week-by-week timeline. From there, the platform stays involved: a conversational mentor for open-ended questions, automatic novelty checks against prior work, team-momentum and skill-gap tracking, viva preparation with mock questions, and auto-generated project documents (synopsis, methodology, progress reports) that are semantically searchable.

Faculty get a digest view across their students rather than having to open every project individually.

## Features

**Agent pipeline** (chained — each stage grounds itself in the previous one's stored output)
- Feasibility — can this team realistically build this?
- Scope — what's in/out of scope, given the feasibility check
- Technology — a concrete recommended stack
- Timeline — a week-by-week plan
- Novelty — checks the idea against prior work
- Risk — flags project risks early
- Calibration — sanity-checks confidence across the pipeline
- Team Momentum — tracks team progress and engagement over time
- Skill Development — identifies skill gaps and growth areas per student
- Viva — mock viva questions and prep
- Faculty Digest — a rolled-up summary for faculty, across students

Every agent run is persisted, so a project's full reasoning history is inspectable later, not just its final output.

**Mentor chat** — a persistent, context-aware conversational assistant for open-ended questions, separate from the fixed-shape structured agents above.

**Viva Studio** — a dedicated space for mock-viva practice and prep.

**Document generation & semantic search** — auto-generates project documents (synopsis, methodology, progress reports) and embeds them with a local sentence-transformers model into pgvector, so a project's own documents become searchable by meaning, not just keyword.

**Evaluation / LLM-as-judge** — an independent judge model scores agent output quality, deliberately configured separately from the agents it's grading to avoid a model inflating its own self-assessment.

**Resilience** — a persistent connectivity banner for flaky wifi (agent runs, chat, and document generation all depend on the network, with no offline queue), branded error/404 boundaries so one crashed page never takes down the whole dashboard's navigation, and a semantic response cache to cut down repeat-query latency and cost.

**Full student dashboard** — projects, submit-idea flow, progress tracking, documents, skills, focus room, concept canvas, project orbit, a "garden of growth" and "journey" view, and a faculty-facing view, all behind auth with a responsive mobile nav.

## Tech stack

**Frontend** — Next.js 16 (App Router, Turbopack) · React 19 · TypeScript · Tailwind CSS 4 · Framer Motion · next-themes (dark/light mode) · react-hot-toast · Supabase JS client

**Backend** — FastAPI (Python) · DSPy (agent signatures/modules) via LiteLLM · Google Gemini · LangGraph (multi-step agent orchestration) · mem0 (agent memory) · sentence-transformers (local embeddings) · pgvector

**Infrastructure** — Supabase (Postgres + pgvector + Auth + Storage)

## Architecture

```
┌─────────────────────┐        HTTP (CORS)        ┌──────────────────────┐
│   Next.js frontend   │ ─────────────────────────▶│   FastAPI backend    │
│  (dashboard, auth,   │◀───────────────────────── │  (agents, mentor,    │
│   mentor chat UI)    │                            │  documents, eval)    │
└──────────┬───────────┘                            └───────────┬──────────┘
           │                                                     │
           │ direct reads/writes                        DSPy → Gemini
           ▼                                                     │
   ┌─────────────────┐                                           ▼
   │    Supabase      │◀──────── agent results, embeddings, feedback
   │ (Postgres +      │
   │  pgvector, Auth) │
   └─────────────────┘
```

The frontend talks to Supabase directly for auth and most reads, and to the FastAPI backend for anything that needs an AI agent to run. Agents read each other's prior stored output from Supabase rather than re-deriving context, so the pipeline stays coherent across stages run at different times.

## Getting started

Full step-by-step setup (Supabase project, migrations in order, env variables, running both servers) is in:
- [`backend/README.md`](./backend/README.md)
- [`frontend/README.md`](./frontend/README.md)

Quick version:

```bash
# Backend
cd backend
python -m venv venv && venv\Scripts\activate   # or: source venv/bin/activate
pip install -r requirements.txt
copy .env.example .env                          # or: cp .env.example .env
# fill in SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, GEMINI_API_KEY
uvicorn app.main:app --reload

# Frontend (separate terminal)
cd frontend
npm install
cp .env.local.example .env.local
# fill in NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, NEXT_PUBLIC_BACKEND_URL
npm run dev
```

Run the Supabase SQL migrations (in `frontend/supabase/` and `backend/supabase/` / `backend/milestone3_migration.sql`) in order via the Supabase SQL editor before starting the backend — see `backend/README.md` for the exact order.

## Project structure

```
.
├── backend/          FastAPI server — agents, mentor, documents, embeddings, evaluation
│   ├── app/
│   │   ├── agents/       one module per AI agent
│   │   ├── core/         config, DB/LLM clients, dspy config
│   │   ├── routers/      API endpoints
│   │   ├── schemas/      request/response shapes
│   │   └── evaluation/   LLM-as-judge scoring
│   ├── scripts/          validation scripts run against real data
│   ├── supabase/         SQL migrations
│   └── tests/
└── frontend/          Next.js app
    ├── app/
    │   ├── (dashboard)/  authenticated app: projects, mentor, viva-studio, docs, etc.
    │   ├── login/ register/
    ├── components/       shared UI, ThemeProvider, ConnectivityBanner, layout
    ├── lib/               navigation config, utils
    └── supabase/          base schema migration
```

## License

MIT — see [`LICENSE`](./LICENSE).
