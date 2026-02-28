# FeedFlow AI
## Intelligent Feedback Management & Ticket Routing System
### Product Requirements Document — v1.0 | February 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Overview](#2-project-overview)
3. [Goals & Success Metrics](#3-goals--success-metrics)
4. [Scope](#4-scope)
5. [Functional Requirements](#5-functional-requirements)
6. [Data Model](#6-data-model)
7. [Technical Architecture](#7-technical-architecture)
8. [Non-Functional Requirements](#8-non-functional-requirements)
9. [User Stories](#9-user-stories)
10. [Milestones & Timeline](#10-milestones--timeline)
11. [Risks & Mitigations](#11-risks--mitigations)
12. [Open Questions](#12-open-questions)
13. [Appendix — Environment Variables](#13-appendix--environment-variables)

---

## 1. Executive Summary

FeedFlow AI is a full-stack web application that combines structured feedback collection with AI-powered triage and routing. When a user submits feedback, a LangChain.js pipeline powered by Google Gemini automatically extracts the **category**, **priority level**, **sentiment**, and **responsible team** — then stores the enriched record in the database and optionally notifies the assigned team via email.

The result is a zero-manual-triage feedback loop that improves response times and ensures every piece of feedback reaches the right people.

---

## 2. Project Overview

| Field | Details |
|-------|---------|
| **Project Title** | FeedFlow AI — Intelligent Feedback Management & Ticket Routing System |
| **Problem Statement** | Teams receive high volumes of unstructured feedback with no automatic triage, causing delays and misrouting. |
| **Proposed Solution** | A SPA + REST API with an embedded LLM pipeline that enriches every submission with category, priority, sentiment, and team assignment. |
| **Primary Users** | End users submitting feedback; internal teams receiving routed tickets. |
| **Tech Stack** | React/Next.js + TypeScript (frontend), Node.js/Express or Next.js API Routes (backend), MongoDB or Supabase, LangChain.js + Gemini, Nodemailer (optional email) |

---

## 3. Goals & Success Metrics

### 3.1 Goals

- Provide an intuitive interface for submitting and browsing feedback
- Automate extraction of category, priority, sentiment, and team via LLM
- Persist all raw and enriched data in the database
- Enable real-time search and filtering across all feedback fields
- *(Optional)* Notify the correct team via email on submission

### 3.2 Success Metrics

| Metric | Target |
|--------|--------|
| LLM enrichment accuracy (category + priority) | ≥ 90% |
| API response time (feedback creation) | < 2 seconds |
| Feedback list load time | < 1 second |
| Search result latency | < 500 ms |
| Email delivery rate (optional) | ≥ 95% |

---

## 4. Scope

### 4.1 In Scope

- Feedback submission form with free-text input
- Database storage of raw and LLM-enriched feedback
- LangChain.js pipeline: category, priority, sentiment, team extraction
- Feedback list SPA with search and filter (name, category, priority)
- Create Feedback modal
- Optional email notification to team address
- REST API (Node.js/Express or Next.js API Routes, TypeScript)
- Deployment to a public hosting platform

### 4.2 Out of Scope *(v1)*

- User authentication / access control
- Analytics dashboards
- Native mobile applications
- Multi-language support
- SLA tracking and escalations

---

## 5. Functional Requirements

### 5.1 Frontend

| ID | Feature | Description | Priority |
|----|---------|-------------|----------|
| FE-01 | Feedback List View | Paginated list showing title, category, priority badge, sentiment, and assigned team | Must Have |
| FE-02 | Create Feedback Modal | Form with submitter name + feedback text; calls POST /api/feedbacks on submit | Must Have |
| FE-03 | Search & Filter Bar | Search by name; dropdowns for category and priority; real-time updates | Must Have |
| FE-04 | Feedback Detail View | Expandable row or side panel showing full text + all LLM-enriched fields | Should Have |
| FE-05 | Email Prompt on Load | On app load, prompt for team email addresses for optional notifications | Nice to Have |
| FE-06 | Loading & Error States | Skeleton loaders during fetch; toast notifications for errors and successes | Should Have |

### 5.2 Backend API

| ID | Endpoint | Description | Priority |
|----|----------|-------------|----------|
| BE-01 | `POST /api/feedbacks` | Accept raw feedback, trigger LLM pipeline, persist enriched document, return result | Must Have |
| BE-02 | `GET /api/feedbacks` | Return all feedbacks; support query params: `search`, `category`, `priority` | Must Have |
| BE-03 | `GET /api/feedbacks/:id` | Return a single enriched feedback document by ID | Should Have |
| BE-04 | `DELETE /api/feedbacks/:id` | Soft-delete or hard-delete a feedback record | Nice to Have |
| BE-05 | `POST /api/config/email` | Store team email addresses for notification routing | Nice to Have |

### 5.3 LLM Pipeline (LangChain.js + Google Gemini)

| ID | Extracted Field | Possible Values | Priority |
|----|----------------|-----------------|----------|
| LLM-01 | `category` | Bug Report \| Feature Request \| Performance \| UX/Design \| Security \| General | Must Have |
| LLM-02 | `priority` | Critical \| High \| Medium \| Low | Must Have |
| LLM-03 | `sentiment` | Positive \| Neutral \| Negative | Must Have |
| LLM-04 | `assignedTeam` | Engineering \| Product \| Design \| Security \| Support \| Management | Must Have |
| LLM-05 | `summary` | One-sentence summary of feedback (max 120 chars) | Should Have |

### 5.4 Email Notifications *(Optional)*

- Triggered on successful feedback creation
- Email sent to the address mapped to the LLM-assigned team
- Email body includes: submitter name, feedback text, category, priority, sentiment
- Implemented via Nodemailer with SMTP or a transactional email provider

---

## 6. Data Model

### 6.1 Feedback Document

```typescript
interface Feedback {
  _id: string;               // Auto-generated ID (ObjectId or UUID)
  submitterName: string;     // Name entered by the user
  feedbackText: string;      // Raw feedback content (max 2000 chars)
  category: FeedbackCategory | null;  // LLM-extracted
  priority: Priority | null;          // LLM-extracted
  sentiment: Sentiment | null;        // LLM-extracted
  assignedTeam: Team | null;          // LLM-extracted
  summary: string | null;             // LLM-generated summary
  createdAt: Date;
  updatedAt: Date;
}

type FeedbackCategory = 'Bug Report' | 'Feature Request' | 'Performance' | 'UX/Design' | 'Security' | 'General';
type Priority        = 'Critical' | 'High' | 'Medium' | 'Low';
type Sentiment       = 'Positive' | 'Neutral' | 'Negative';
type Team            = 'Engineering' | 'Product' | 'Design' | 'Security' | 'Support' | 'Management';
```

### 6.2 LLM Output Schema (Zod)

```typescript
import { z } from 'zod';

export const LLMOutputSchema = z.object({
  category:     z.enum(['Bug Report', 'Feature Request', 'Performance', 'UX/Design', 'Security', 'General']),
  priority:     z.enum(['Critical', 'High', 'Medium', 'Low']),
  sentiment:    z.enum(['Positive', 'Neutral', 'Negative']),
  assignedTeam: z.enum(['Engineering', 'Product', 'Design', 'Security', 'Support', 'Management']),
  summary:      z.string().max(120),
});
```

---

## 7. Technical Architecture

### 7.1 System Components

| Layer | Technology | Responsibility |
|-------|-----------|----------------|
| Frontend | React 18 + TypeScript (Vite or Next.js) | SPA — feedback list, modals, search/filter |
| Styling | Tailwind CSS + shadcn/ui | Utility-first responsive styling |
| HTTP Client | Axios / Fetch API | REST calls from browser to backend |
| Backend | Node.js + Express + TypeScript **or** Next.js API Routes | REST API, business logic, orchestration |
| LLM Pipeline | LangChain.js + Google Gemini | Field extraction via structured output |
| Database | MongoDB (Atlas) **or** Supabase (PostgreSQL) | Persistence of enriched feedback |
| Email | Nodemailer (optional) | SMTP-based team notifications |
| Hosting | Render / Railway / Vercel | Public deployment |

### 7.2 Data Flow

```
User submits feedback
        │
        ▼
POST /api/feedbacks
        │
        ▼
Validate input (Zod)
        │
        ▼
LangChain.js + Gemini
  → category, priority,
    sentiment, team, summary
        │
        ▼
Save enriched doc to DB
        │
        ├─── (optional) Send email to team
        │
        ▼
Return enriched feedback to frontend
        │
        ▼
Frontend updates list in real time
```

### 7.3 Project Structure (Next.js variant)

```
feedflow-ai/
├── app/
│   ├── api/
│   │   └── feedbacks/
│   │       ├── route.ts          # GET, POST /api/feedbacks
│   │       └── [id]/route.ts     # GET, DELETE /api/feedbacks/:id
│   ├── page.tsx                  # Feedback list + modals
│   └── layout.tsx
├── components/
│   ├── FeedbackList.tsx
│   ├── CreateFeedbackModal.tsx
│   ├── SearchFilterBar.tsx
│   └── FeedbackCard.tsx
├── lib/
│   ├── db.ts                     # DB connection (Mongoose or Supabase client)
│   ├── llm-pipeline.ts           # LangChain.js + Gemini pipeline
│   └── email.ts                  # Nodemailer helper
├── types/
│   └── feedback.ts               # Shared TypeScript types
├── .env.local
└── README.md
```

---

## 8. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| **Performance** | Feedback list loads within 1 s; API responds in < 2 s |
| **Scalability** | Schema and indexes support up to 100,000 records without degradation |
| **Reliability** | LLM pipeline failures fall back gracefully — feedback saved with `null` enrichment fields rather than lost |
| **Security** | API keys stored as environment variables; never committed to source control |
| **Accessibility** | WCAG 2.1 AA compliance for all interactive UI components |
| **Maintainability** | TypeScript strict mode on both frontend and backend; ESLint + Prettier enforced |
| **Portability** | Docker Compose file for one-command local setup |

---

## 9. User Stories

| ID | As a... | I want to... | So that... |
|----|---------|-------------|------------|
| US-01 | End user | Submit feedback via a simple form | The team can act on my input quickly |
| US-02 | End user | See my feedback listed with a priority badge | I know it was received and categorised |
| US-03 | Team member | Search feedbacks by submitter name | I can find specific submissions fast |
| US-04 | Team member | Filter feedbacks by category and priority | I can focus on the most critical items |
| US-05 | Team member | Receive an email when high-priority feedback is created | I can respond without checking the app constantly |
| US-06 | Developer | Run the project with a single Docker Compose command | I can onboard quickly without manual setup |

---

## 10. Milestones & Timeline

| Phase | Milestone | Deliverables | Est. Duration |
|-------|-----------|-------------|---------------|
| 1 | Project Setup | Repo scaffold, TypeScript config, ESLint, Prettier, Docker Compose, DB connection | 0.5 days |
| 2 | Backend Core | API endpoints (POST + GET), schema/model, input validation | 1 day |
| 3 | LLM Pipeline | LangChain.js + Gemini, structured output schema, error fallback | 1 day |
| 4 | Frontend Core | Feedback List, Create Modal, API integration | 1.5 days |
| 5 | Search & Filter | Search bar, category/priority filters, debounced queries | 0.5 days |
| 6 | Email *(Optional)* | Nodemailer setup, email-on-load prompt, team routing | 0.5 days |
| 7 | Polish & Deploy | Error handling, loading states, hosting, live URL | 0.5 days |
| | **Total** | | **~6 days** |

---

## 11. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Gemini API rate limits in free tier | Medium | High | Request queuing; cache results for identical inputs |
| LLM returns malformed JSON | Medium | Medium | LangChain structured output parser with Zod schema; default to `null` on failure |
| DB cold starts on free tier | Low | Medium | Connection pooling; retry logic with exponential backoff |
| Email delivery blocked by SMTP provider | Low | Low | Feature is optional; document alternative SMTP config in README |
| Scope creep (auth, analytics) | Medium | Medium | Strictly defer v2 features; reference this PRD as the baseline |

---

## 12. Open Questions

- [ ] Should soft-deleted feedbacks remain visible with a toggle, or be fully hidden?
- [ ] Is pagination (e.g. 20 per page) required in v1, or is infinite scroll acceptable?
- [ ] Should LLM enrichment run **synchronously** in the API response, or be queued **asynchronously**?
- [ ] What is the maximum feedback text length — 2,000 characters, or more?
- [ ] For the optional email feature, should all teams share one inbox or have individual addresses?

---

## 13. Appendix — Environment Variables

| Variable | Description |
|----------|-------------|
| `GEMINI_API_KEY` | Google Gemini API key (from Google AI Studio free tier) |
| `MONGODB_URI` | MongoDB connection string — for MERN stack |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL — for Next.js + Supabase stack |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon key — for Next.js + Supabase stack |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key (server-side only) |
| `PORT` | Express server port (default: `3001`) — MERN only |
| `SMTP_HOST` | SMTP server hostname *(optional)* |
| `SMTP_PORT` | SMTP port, typically `587` or `465` *(optional)* |
| `SMTP_USER` | SMTP authentication username *(optional)* |
| `SMTP_PASS` | SMTP authentication password *(optional)* |
| `VITE_API_BASE_URL` | Frontend base URL pointing to the backend API — MERN only |

---

*FeedFlow AI — PRD v1.0 · Confidential · February 2026*
