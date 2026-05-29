# CLAUDE.md — StartupAgent Context

> Keep this file under 200 lines. Update it when you discover non-obvious things.
> This file is the primary context handoff between AI coding sessions.

---

## Project in One Sentence

StartupAgent is a TypeScript/React/Node.js app that wraps GPT-4o (Assistants API) to guide founders through six startup lifecycle stages via a persistent conversational agent with structured tool outputs.

---

## Commands

```bash
# Install all dependencies (root + workspaces)
npm install

# Start dev (frontend on :5173, backend on :3001, concurrently)
npm run dev

# Run all tests
npm test

# Unit tests only (Vitest)
npm run test:unit

# Integration tests only
npm run test:integration

# E2E tests (Playwright — requires dev server running)
npm run test:e2e

# Lint (ESLint + TypeScript check)
npm run lint

# Format (Prettier)
npm run format

# Type check only
npm run typecheck

# Database migrations
npm run db:migrate

# Reset DB + seed demo data
npm run db:reset

# Build for production
npm run build
```

---

## Directory Map

```
src/
  agent/
    agent-runner.ts       # Core run loop: sends messages, handles tool_calls, streams
    thread-manager.ts     # Maps userId → OpenAI thread ID; handles expiry + creation
    tool-registry.ts      # Registers all tool definitions for the Assistants API
  api/
    routes/               # Express route handlers (chat, threads, stage, hypotheses)
    middleware/           # Auth (Clerk JWT), rate-limit, error-handler
    server.ts             # Express app factory
  components/
    ChatInterface/        # Streaming chat UI
    StageProgressBar/     # Lifecycle stage indicator
    ToolResultCard/       # Renders structured tool outputs as cards
    ThreadHistory/        # Sidebar list of past threads
  hooks/
    useChat.ts            # React Query + SSE integration for streaming
    useStage.ts           # Stage read/update hook
  lib/
    openai.ts             # Typed OpenAI SDK wrapper (retry, timeout, token counting)
    prisma.ts             # Prisma client singleton
    logger.ts             # Pino structured logger
  pages/
    Dashboard.tsx         # Main app page
    Onboarding.tsx        # First-login flow
  prompts/
    system-prompt.ts      # Builds stage-aware system prompt from founder profile
    stage-prompts/        # Per-stage instruction chunks injected into system prompt
  types/
    agent.ts              # AgentRun, ToolCall, StreamChunk types
    lifecycle.ts          # LifecycleStage enum + stage metadata
    api.ts                # Request/response types for all API routes

tests/
  unit/                   # Mirror of src/ — one test file per source file
  integration/            # Full request → DB round-trips; OpenAI mocked with msw
  e2e/                    # Playwright tests (login, chat, stage transitions)
```

---

## Non-Obvious Conventions

- **OpenAI Assistants API** — we use **threads + runs**, not the Chat Completions API. A run can have status `requires_action` when tools need to be called; `AgentRunner` handles this loop.
- **Streaming** — backend sends SSE; frontend uses `EventSource`. Each chunk is `data: {"type":"delta","content":"..."}`. End of stream: `data: {"type":"done"}`.
- **Tool naming** — all tools use `snake_case` names (OpenAI requirement). Map to camelCase in TypeScript types.
- **System prompt budget** — the system prompt must stay under 2000 tokens. `system-prompt.ts` enforces this with a token counter; truncate stage history if over budget.
- **Prisma singleton** — import from `src/lib/prisma.ts`, never instantiate `new PrismaClient()` elsewhere (causes connection pool exhaustion in dev).
- **Path aliases** — `@agent/*` → `src/agent`, `@api/*` → `src/api`, `@shared/*` → `src/types`. Configured in `tsconfig.json` and Vite config.
- **Clerk JWT** — backend validates with `@clerk/clerk-sdk-node`. Middleware attaches `req.auth.userId`. Never trust client-sent userId.
- **Environment** — never import from `process.env` directly; use `src/lib/config.ts` which validates all vars at startup with zod.

---

## Workflow (Follow This Order)

1. **Read this file** and `TODO.md` to understand current state
2. **Run tests first:** `npm test` — confirm baseline is green before changing anything
3. Pick the next unchecked task from the current phase in `TODO.md`
4. **Write failing tests first** (red) in `tests/unit/` or `tests/integration/`
5. **Implement** in `src/` until tests pass (green)
6. Run `npm run lint && npm run typecheck` — fix all errors
7. `git diff` — read your own changes before committing
8. Commit with format: `feat(agent): implement ThreadManager expiry (#P1-3)`
9. If anything surprised you, add it to **Lessons Learned** in `TODO.md`
10. Update this file if you discovered a new non-obvious convention

---

## Gotchas

- OpenAI thread IDs expire after 30 days of inactivity — `ThreadManager` must check `lastActiveAt` before resuming
- Running two `AgentRunner` instances on the same thread simultaneously causes a 409 conflict — always check for an active run before starting a new one
- Vitest and Jest have different mock APIs — we use **Vitest** (`vi.mock`, `vi.fn`)
- SSE connections keep the Node.js process alive — ensure `res.end()` is always called, even on errors
- Tailwind CSS purging — add component class names to `safelist` in `tailwind.config.ts` if dynamically constructed
