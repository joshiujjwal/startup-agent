# StartupAgent — Feature Specification

**Version:** 0.1.0-draft  
**Author:** joshiujjwal  
**Status:** Draft — not yet implementation-ready

---

## 1. Overview

### Problem Statement

Founders are generalists forced to context-switch across wildly different domains (product, fundraising, people, growth) with no institutional memory and no always-available expert. Coaches are expensive; advisors are sparse; Google is noise.

### Solution

StartupAgent is a conversational AI agent (powered by GPT-4o with tool use) that acts as a knowledgeable co-founder. It maintains a persistent thread per user, knows which lifecycle stage the startup is in, and proactively structures thinking rather than just answering questions.

### Non-Goals (v0.1)

- Not a project management tool (no Kanban boards)
- Not a document editor (outputs are structured chat cards, not Google Docs)
- Not a financial modelling tool (cap table is illustrative, not auditable)
- No multi-user collaboration in v0.1

---

## 2. Functional Requirements

### 2.1 Authentication & Onboarding

- [ ] User signs up with email/Google via Clerk
- [ ] On first login, agent asks 5 onboarding questions: idea one-liner, industry, stage, team size, biggest challenge
- [ ] Founder profile is saved to `User` table and injected into every system prompt

### 2.2 Persistent Threads

- [ ] Each user has one active OpenAI thread at a time (resumable across sessions)
- [ ] Thread ID stored in Postgres; refreshed if expired (>30 days)
- [ ] Full message history visible in sidebar; user can start a new thread

### 2.3 Lifecycle Stage Management

- [ ] Six stages: `IDEATION → VALIDATION → MVP → FUNDRAISING → HIRING → GROWTH`
- [ ] Stage is stored per-user in Postgres
- [ ] Agent detects readiness signals in conversation and suggests stage advance (e.g. "You've validated 3 hypotheses — ready to scope MVP?")
- [ ] User can manually override stage via UI toggle
- [ ] Stage shown persistently in UI header

### 2.4 Ideation Module

- [ ] Tool: `refine_idea(rawIdea: string)` → returns structured `IdeaCanvas` (problem, solution, ICP, differentiation)
- [ ] Tool: `generate_icp(ideaCanvas)` → returns 3 ICP personas with pain scores
- [ ] Tool: `competitive_landscape(ideaCanvas)` → lists 5 closest competitors with moat analysis

### 2.5 Validation Module

- [ ] Tool: `build_hypothesis(claim: string)` → returns testable hypothesis with metrics
- [ ] Tool: `design_experiment(hypothesis)` → returns lean experiment plan (riskiest assumption, method, success criteria, timeline)
- [ ] Tool: `log_signal(experimentId, result, strength: 'weak'|'moderate'|'strong')` → persists to DB
- [ ] Validation dashboard: list of hypotheses, status, signal strength visualised

### 2.6 MVP Module

- [ ] Tool: `scope_mvp(validatedHypotheses[])` → returns feature list with build/buy/defer tags
- [ ] Tool: `create_milestone_plan(features[])` → 4-week sprint breakdown
- [ ] Tool: `generate_tech_spec(feature)` → one-page technical spec per feature

### 2.7 Fundraising Module

- [ ] Tool: `draft_pitch_narrative(founderProfile, mvpScope)` → returns narrative arc (problem → solution → traction → ask)
- [ ] Tool: `model_cap_table(founders[], investment: number, valuation: number)` → returns post-money cap table
- [ ] Tool: `diligence_checklist(stage: 'pre-seed'|'seed'|'series-a')` → returns stage-appropriate checklist

### 2.8 Hiring Module

- [ ] Tool: `write_jd(role, seniority, companyStage)` → returns structured JD (responsibilities, requirements, culture note)
- [ ] Tool: `first_10_hires_plan(currentTeam[], productRoadmap)` → prioritised hiring order with rationale
- [ ] Tool: `culture_fit_rubric(values[])` → returns interview scorecard

### 2.9 Growth Module

- [ ] Tool: `design_growth_loop(productDescription, targetICP)` → returns acquisition + retention + referral loop diagram as JSON
- [ ] Tool: `score_channels(stage, icp, budget)` → ranks acquisition channels by ICE score
- [ ] Tool: `build_okrs(stage, quarter, goals[])` → returns OKR tree (objective → key results)

### 2.10 Streaming Chat Interface

- [ ] Agent responses streamed via SSE (`text/event-stream`)
- [ ] Tool results rendered as structured cards (not raw JSON)
- [ ] Typing indicator while streaming
- [ ] User can cancel an in-flight generation

---

## 3. Non-Functional Requirements

| Requirement        | Target                                      |
|--------------------|---------------------------------------------|
| Response latency   | First token < 1.5s (p95)                    |
| Availability       | 99.5% uptime (single-region to start)       |
| Token cost cap     | $0.05 per conversation turn (enforced guard) |
| Auth security      | Clerk-managed; all API routes require JWT   |
| Data retention     | User data deletable on request (GDPR-aware) |
| Accessibility      | WCAG 2.1 AA on all interactive elements     |

---

## 4. Data Model

```prisma
model User {
  id             String          @id @default(cuid())
  clerkId        String          @unique
  email          String          @unique
  founderProfile Json            // { idea, industry, stage, teamSize, biggestChallenge }
  currentStage   LifecycleStage  @default(IDEATION)
  threads        Thread[]
  createdAt      DateTime        @default(now())
}

model Thread {
  id            String    @id @default(cuid())
  userId        String
  user          User      @relation(fields: [userId], references: [id])
  openaiThreadId String   @unique
  title         String?
  isActive      Boolean   @default(true)
  messages      Message[]
  createdAt     DateTime  @default(now())
  lastActiveAt  DateTime  @default(now())
}

model Message {
  id        String   @id @default(cuid())
  threadId  String
  thread    Thread   @relation(fields: [threadId], references: [id])
  role      String   // 'user' | 'assistant' | 'tool'
  content   String
  toolName  String?
  toolInput Json?
  toolOutput Json?
  tokens    Int?
  createdAt DateTime @default(now())
}

model Hypothesis {
  id           String   @id @default(cuid())
  userId       String
  claim        String
  testMethod   String
  successMetric String
  result       String?
  signalStrength String?  // 'weak' | 'moderate' | 'strong'
  createdAt    DateTime @default(now())
}

enum LifecycleStage {
  IDEATION
  VALIDATION
  MVP
  FUNDRAISING
  HIRING
  GROWTH
}
```

---

## 5. API Design

| Method | Path                        | Auth | Description                              |
|--------|-----------------------------|------|------------------------------------------|
| POST   | `/api/auth/webhook`         | –    | Clerk webhook for user provisioning      |
| POST   | `/api/chat`                 | JWT  | Send message; returns SSE stream         |
| GET    | `/api/threads`              | JWT  | List user's threads                      |
| GET    | `/api/threads/:id`          | JWT  | Get thread with messages                 |
| POST   | `/api/threads`              | JWT  | Create new thread                        |
| PATCH  | `/api/stage`                | JWT  | Update lifecycle stage                   |
| GET    | `/api/hypotheses`           | JWT  | List all hypotheses for user             |
| POST   | `/api/hypotheses`           | JWT  | Log a new hypothesis / signal            |
| GET    | `/api/usage`                | JWT  | Token usage stats for current user       |

---

## 6. Test Plan

### Unit Tests
- `openai.ts` wrapper: retry logic, timeout, malformed response handling
- `ThreadManager`: create, resume, expire scenarios
- `AgentRunner`: tool call dispatch, streaming chunk handling
- Each module tool: valid input → correct output shape, invalid input → typed error
- System prompt builder: stage injection, profile injection, token budget guard

### Integration Tests
- Full chat round-trip with mocked OpenAI (nock or msw)
- DB: thread persists correctly, message saved after run completes
- Stage transition: signal detected → stage updated in DB

### E2E Tests (Playwright)
- New user onboarding flow
- Send message → receive streamed response
- Stage advance prompt appears after validation signals
- Mobile viewport: chat interface usable on 375px width

### Edge Cases
- OpenAI API timeout (>30s) → graceful error card, no broken state
- Thread expired → auto-create new thread, inform user
- Token limit hit mid-conversation → summarise and continue
- Concurrent messages from same user → queue, do not duplicate runs

---

## 7. Open Questions

- [ ] Should we support multiple active threads (one per startup) vs one thread per user?
- [ ] How do we handle fine-tuning vs prompt engineering for domain-specific advice?
- [ ] Should tool results be stored as `Message` rows or a separate `ToolResult` table?
- [ ] Rate limiting strategy: per-user token budget per day vs per-request cost cap?
- [ ] Webhook vs polling for Clerk user events?
