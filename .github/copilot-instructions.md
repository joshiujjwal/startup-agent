# GitHub Copilot Instructions — StartupAgent

## Stack

- **Language:** TypeScript (strict mode, no `any`)
- **Frontend:** React 18, Vite, Tailwind CSS, React Query, Clerk (auth)
- **Backend:** Node.js, Express, Prisma ORM, PostgreSQL
- **AI:** OpenAI GPT-4o via Assistants API (threads, runs, tool calls)
- **Testing:** Vitest (unit/integration), Playwright (e2e), msw (API mocking)

---

## Coding Conventions

### TypeScript
- Always provide explicit return types on exported functions
- Use zod schemas to validate all inputs at system boundaries (API routes, OpenAI responses)
- Prefer `Result<T, E>` pattern over throwing for recoverable errors in agent logic
- Path aliases: `@agent/*`, `@api/*`, `@shared/*` — use these, not relative `../../` imports

### React Components
- One component per file, filename matches component name
- Props interface defined in same file, named `<ComponentName>Props`
- Use Tailwind utility classes — no inline styles, no CSS modules
- Streaming UI: use `ReadableStream` or SSE via `EventSource`, not WebSockets

### Express Routes
- All route handlers are `async` and wrapped with error catching
- Validate request body with zod before processing
- Return consistent shape: `{ data: T }` on success, `{ error: string, code: string }` on failure

### OpenAI Integration
- Import the OpenAI client only from `src/lib/openai.ts` — never instantiate directly
- Tool definitions live in `src/agent/tool-registry.ts` — one export per lifecycle module
- Always check run status before submitting tool outputs: must be `requires_action`

---

## Testing Conventions

- **Write tests before implementation** — if you're writing src code, the test file must exist first
- Test files live in `tests/` mirroring `src/` structure
- Mock OpenAI API with msw handlers in `tests/mocks/openai-handlers.ts`
- Never use `test.only` or `test.skip` in committed code
- Prefer `describe` + `it` over bare `test` calls for readability

---

## What NOT to Do

- Do **not** refactor existing working code unless the task explicitly asks for it
- Do **not** remove, skip, or comment out existing tests
- Do **not** commit secrets, API keys, or `.env` files
- Do **not** use `console.log` in production code — use the pino logger from `src/lib/logger.ts`
- Do **not** bypass TypeScript with `// @ts-ignore` or `as any`
- Do **not** make real OpenAI API calls in any test — always mock
- Do **not** add new npm packages without checking bundle size impact (use bundlephobia)
- Do **not** use `useEffect` for data fetching — use React Query hooks

---

## File Templates

### New API route (`src/api/routes/<name>.ts`)
```typescript
import { Router } from 'express'
import { z } from 'zod'
import { requireAuth } from '@api/middleware/auth'

const router = Router()

const RequestSchema = z.object({ /* ... */ })

router.post('/', requireAuth, async (req, res, next) => {
  try {
    const body = RequestSchema.parse(req.body)
    // implementation
    res.json({ data: result })
  } catch (error) {
    next(error)
  }
})

export default router
```

### New agent tool (`src/agent/tools/<name>.ts`)
```typescript
import type { Tool } from 'openai/resources/beta/assistants'

export const myToolDefinition: Tool = {
  type: 'function',
  function: {
    name: 'my_tool_name',       // snake_case required by OpenAI
    description: '...',
    parameters: { /* JSON Schema */ }
  }
}

export async function executeMyTool(input: unknown): Promise<MyToolOutput> {
  const parsed = MyToolInputSchema.parse(input)
  // implementation
}
```
