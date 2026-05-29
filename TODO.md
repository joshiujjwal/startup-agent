# StartupAgent — Task Breakdown

## How to Use This File

**Workflow per task:**
1. Write tests FIRST (red phase) — no implementation yet
2. Implement until tests pass (green phase)
3. `git diff` — review manually, no surprises
4. Commit with message referencing the task (e.g. `feat: agent thread manager (#P1-3)`)
5. If you learned something non-obvious, add it to `CLAUDE.md` → `Lessons Learned`
6. Only start the next phase after all tasks in the current phase have ✅ passing tests + human review

**Evidence gates:** Each phase is locked until the prior phase's tests are green in CI.

---

## Phase 0: Foundation ⬜

- [ ] **P0-1** Initialise monorepo with npm workspaces (`packages/client`, `packages/server`)
- [ ] **P0-2** Configure TypeScript (`tsconfig.json`) with strict mode + path aliases (`@agent/*`, `@api/*`, `@shared/*`)
- [ ] **P0-3** Add ESLint (typescript-eslint, react-hooks plugin) + Prettier; enforce via `lint-staged` + `husky`
- [ ] **P0-4** Set up Vitest for unit tests; write a smoke test (`1+1===2`) that must pass in CI
- [ ] **P0-5** Set up Playwright for e2e; scaffold a "page loads" test
- [ ] **P0-6** Create `.github/workflows/ci.yml` — runs lint, unit tests, integration tests on every PR
- [ ] **P0-7** Add `docker-compose.yml` for local Postgres + Redis (for rate-limiting / caching)
- [ ] **P0-8** Set up Prisma: `schema.prisma` with `User`, `Thread`, `Message`, `LifecycleStage` models
- [ ] **P0-9** Add `.env.example` with all required keys documented (never commit real values)
- [ ] **P0-10** Review all AI config files (`CLAUDE.md`, `AGENTS.md`, `copilot-instructions.md`) — update if needed

**Phase 0 evidence gate:** CI passes on a clean branch with zero lint errors and smoke test green ✅

---

## Phase 1: Core Agent Engine ⬜

- [ ] **P1-1** Write unit tests for OpenAI client wrapper (`src/lib/openai.ts`) — mock API responses
- [ ] **P1-2** Implement `src/lib/openai.ts` — typed wrapper around `openai` SDK with retry + timeout
- [ ] **P1-3** Write tests for `ThreadManager` (`src/agent/thread-manager.ts`) — create/resume/expire threads
- [ ] **P1-4** Implement `ThreadManager` — maps `userId` → OpenAI thread ID, persists in Postgres via Prisma
- [ ] **P1-5** Write tests for `AgentRunner` (`src/agent/agent-runner.ts`) — handles tool calls, streams responses
- [ ] **P1-6** Implement `AgentRunner` — run loop with tool_call dispatch, handles `requires_action` status
- [ ] **P1-7** Write tests for system prompt builder (`src/prompts/system-prompt.ts`) — stage-aware prompt assembly
- [ ] **P1-8** Implement system prompt builder — injects lifecycle stage context, founder profile, prior decisions
- [ ] **P1-9** Integration test: end-to-end message round-trip (mocked OpenAI) through ThreadManager → AgentRunner
- [ ] **P1-10** Manual test: send a real message, confirm GPT-4o responds in dev environment

**Phase 1 evidence gate:** All unit + integration tests green; manual round-trip confirmed with screenshot ✅

---

## Phase 2: Lifecycle Stage Modules ⬜

- [ ] **P2-1** Define `LifecycleStage` enum: `IDEATION | VALIDATION | MVP | FUNDRAISING | HIRING | GROWTH`
- [ ] **P2-2** Write + implement `IdeationModule` — idea refinement tool, ICP generation, positioning canvas
- [ ] **P2-3** Write + implement `ValidationModule` — hypothesis builder, experiment tracker, signal scoring
- [ ] **P2-4** Write + implement `MVPModule` — feature scoping tool, build/buy/defer matrix, milestone planner
- [ ] **P2-5** Write + implement `FundraisingModule` — pitch narrative drafter, cap table modeller, diligence checklist
- [ ] **P2-6** Write + implement `HiringModule` — JD generator, culture-fit rubric, first-10-hires planner
- [ ] **P2-7** Write + implement `GrowthModule` — growth loop designer, channel scoring, OKR builder
- [ ] **P2-8** Write + implement stage-transition logic — agent detects readiness signals and proposes stage advance
- [ ] **P2-9** REST API endpoints: `POST /api/chat`, `GET /api/threads/:id`, `PATCH /api/stage`
- [ ] **P2-10** Integration tests for all 6 lifecycle modules with mocked OpenAI tool responses
- [ ] **P2-11** Manual walkthrough: simulate a founder going from IDEATION → VALIDATION with real GPT-4o

**Phase 2 evidence gate:** All 6 modules have green tests; manual walkthrough recorded ✅

---

## Phase 3: React Frontend ⬜

- [ ] **P3-1** Scaffold Vite + React 18 + TypeScript + Tailwind CSS client in `packages/client`
- [ ] **P3-2** Auth flow with Clerk — sign-up, sign-in, protected routes
- [ ] **P3-3** `ChatInterface` component — streaming message display, input box, loading state
- [ ] **P3-4** `StageProgressBar` component — shows current lifecycle stage, allows manual override
- [ ] **P3-5** `ToolResultCard` component — renders structured output from agent tools (tables, checklists, canvases)
- [ ] **P3-6** `ThreadHistory` sidebar — list past conversations, resume a thread
- [ ] **P3-7** Dashboard page — stage overview, recent activity, quick-action shortcuts
- [ ] **P3-8** Connect frontend to backend API via React Query (`@tanstack/react-query`)
- [ ] **P3-9** Add Playwright e2e tests: login → send first message → receive response → stage shown correctly
- [ ] **P3-10** Responsive design audit (mobile + desktop); fix any layout breakage

**Phase 3 evidence gate:** Playwright e2e passes; responsive screenshots attached to PR ✅

---

## Phase 4: Polish & Harden ⬜

- [ ] **P4-1** Rate limiting on API (`express-rate-limit` + Redis) — 60 req/min per user
- [ ] **P4-2** Input sanitisation and max token guard on all user messages
- [ ] **P4-3** Error boundary in React UI — graceful degradation if agent fails
- [ ] **P4-4** OpenAI cost tracking — log token usage per request to Postgres; expose `/api/usage` endpoint
- [ ] **P4-5** Add streaming SSE support to chat API (`text/event-stream`)
- [ ] **P4-6** Observability — structured logging (pino), request IDs, error tracking (Sentry)
- [ ] **P4-7** Security audit: review all API routes for auth gaps, OWASP Top 10 checklist
- [ ] **P4-8** Performance: lighthouse score ≥ 90 on dashboard page
- [ ] **P4-9** Seed script for demo founder profile (`npm run db:seed`)
- [ ] **P4-10** Load test API with k6: 50 concurrent users, p95 latency < 2s

**Phase 4 evidence gate:** Security checklist signed off; load test results in PR ✅

---

## Phase 5: Ship ⬜

- [ ] **P5-1** Production Dockerfile (multi-stage build, non-root user)
- [ ] **P5-2** Deploy backend to Railway / Render; deploy frontend to Vercel
- [ ] **P5-3** Configure production environment variables in hosting platforms
- [ ] **P5-4** Set up custom domain + TLS
- [ ] **P5-5** Add `CHANGELOG.md` with v0.1.0 entry
- [ ] **P5-6** Tag release `v0.1.0` and publish GitHub Release with release notes
- [ ] **P5-7** Announce in one public channel (HN, Twitter/X, or Indie Hackers)

**Phase 5 evidence gate:** Production URL accessible; v0.1.0 release published ✅

---

## Parking Lot 🅿️

- Multi-founder collaboration (shared threads)
- Slack / Notion integration for exporting agent outputs
- Investor CRM built on top of FundraisingModule
- Fine-tuned model on YC batch company narratives
- Mobile app (React Native)
- Voice interface (Whisper + TTS)

---

## Lessons Learned 📝

> _Add entries here as you discover non-obvious things. Format: `[date] — finding`._

- Example: `[2024-01-01] — OpenAI thread IDs expire after 30 days; implement TTL check on resume`
