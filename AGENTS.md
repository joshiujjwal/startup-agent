# AGENTS.md — StartupAgent

> Standard context file for OpenAI Codex and GPT-based coding agents.
> Read this before writing any code.

---

## Setup

```bash
# 1. Install dependencies
npm install

# 2. Copy environment template and fill in required keys
cp .env.example .env

# 3. Start local Postgres (Docker required)
docker-compose up -d

# 4. Run DB migrations
npm run db:migrate

# 5. Verify everything works
npm test
npm run lint
```

Required env vars (see `.env.example` for full list):
- `OPENAI_API_KEY` — GPT-4o API key with Assistants API access
- `DATABASE_URL` — PostgreSQL connection string
- `CLERK_SECRET_KEY` — Clerk backend secret
- `CLERK_PUBLISHABLE_KEY` — Clerk frontend key

---

## Code Style

### TypeScript
- Strict mode enabled — no `any`, no `as unknown as X` escape hatches
- Use `type` for shapes, `interface` for extensible contracts (e.g. module interfaces)
- Prefer `const` everywhere; `let` only when reassignment is necessary
- All async functions must handle errors explicitly — no unhandled promise rejections
- Export types from `src/types/` — never inline complex types in business logic files
- Use zod for all external data validation (API request bodies, env vars, OpenAI responses)

### Naming
- Files: `kebab-case.ts`
- React components: `PascalCase.tsx`
- Functions/variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- OpenAI tool names: `snake_case` (API requirement)

### React
- Functional components only — no class components
- Custom hooks in `src/hooks/` — prefix with `use`
- Co-locate component styles as Tailwind classes — no separate CSS files
- Use React Query for all server state — no manual `useState` for API data

### Backend
- All Express routes must be typed with `Request<Params, ResBody, ReqBody, Query>`
- Use `next(error)` for all error propagation — no `res.json({ error })` inline
- Middleware order: Clerk auth → rate-limit → route handler → error handler

---

## Testing

**Rule: Write the test before the implementation. Always.**

```bash
npm run test:unit        # Vitest — fast, no DB
npm run test:integration # Vitest + real DB (Docker must be running)
npm run test:e2e         # Playwright — requires dev server on :5173/:3001
npm test                 # All of the above
```

### Test file conventions
- Unit tests: `tests/unit/<path-matching-src>.test.ts`
- Integration tests: `tests/integration/<feature>.test.ts`
- E2E tests: `tests/e2e/<flow>.spec.ts`
- Mock OpenAI with `msw` (Mock Service Worker) — never make real API calls in tests
- Use `vi.mock('src/lib/prisma')` to mock DB in unit tests
- Each test file: one `describe` block per exported function/component

### Coverage expectations
- Agent core (`agent/`) — 90%+ line coverage
- API routes — 80%+ line coverage
- React components — test behaviour, not markup (use Testing Library queries)

---

## PR Instructions

1. **Title format:** `type(scope): description` — e.g. `feat(agent): add ThreadManager expiry check`
2. **Types:** `feat`, `fix`, `test`, `refactor`, `docs`, `chore`
3. **Evidence required in PR description:**
   - Paste test output showing red → green transition
   - For UI changes: attach before/after screenshots
   - For API changes: paste a `curl` example with real response
4. **Never:**
   - Merge with failing tests
   - Remove or skip existing tests
   - Commit `.env` files or API keys
   - Refactor code outside the task scope without a separate PR
5. **Always:**
   - Run `npm run lint && npm run typecheck` before pushing
   - Update `CLAUDE.md` Lessons Learned if something surprised you
   - Keep PRs small — one TODO item per PR is ideal

---

## Architecture Constraints

- The OpenAI Assistants API `run` object is the source of truth for agent state — do not try to reconstruct it from DB
- All user-facing errors must be caught and returned as `{ error: string, code: string }` — never expose stack traces
- The system prompt is assembled fresh on every run from `src/prompts/system-prompt.ts` — do not cache it
- Rate limiting is enforced per `userId` (from Clerk JWT), never per IP alone
