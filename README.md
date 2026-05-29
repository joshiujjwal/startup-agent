# StartupAgent 🚀

> **An AI co-pilot that guides founders through the full startup lifecycle.**

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-TypeScript%20%7C%20React%20%7C%20Node.js%20%7C%20GPT--4o-blue)
![License](https://img.shields.io/badge/license-MIT-green)

StartupAgent acts as an always-on startup co-pilot — helping founders navigate ideation, validation, MVP planning, fundraising, hiring, and growth — all through a conversational AI interface powered by OpenAI GPT-4o.

---

## ✨ Features

| Lifecycle Stage     | What StartupAgent Does                                              |
|---------------------|---------------------------------------------------------------------|
| **Ideation**        | Refines raw ideas, identifies target users, suggests positioning    |
| **Validation**      | Generates hypothesis-driven experiments, tracks signal/noise       |
| **MVP Planning**    | Scopes ruthlessly, creates prioritised build roadmap               |
| **Fundraising Prep**| Drafts pitch narrative, models cap table, preps diligence docs     |
| **Hiring**          | Writes JDs, defines culture fit criteria, plans first 10 hires     |
| **Growth**          | Builds growth loops, identifies acquisition channels, sets OKRs    |

---

## 🛠 Tech Stack

- **Frontend** — React 18, TypeScript, Tailwind CSS, Vite
- **Backend** — Node.js, Express, TypeScript
- **AI** — OpenAI GPT-4o via Assistants API (threads + tools)
- **Database** — PostgreSQL (via Prisma ORM)
- **Auth** — Clerk
- **Testing** — Vitest (unit/integration), Playwright (e2e)
- **CI** — GitHub Actions

---

## 🚀 Getting Started

```bash
# Clone
git clone https://github.com/joshiujjwal/startup-agent.git
cd startup-agent

# Install dependencies
npm install

# Set up environment
cp .env.example .env
# Fill in OPENAI_API_KEY, DATABASE_URL, CLERK_SECRET_KEY, etc.

# Run DB migrations
npm run db:migrate

# Start dev servers (frontend + backend concurrently)
npm run dev

# Run all tests
npm test

# Lint & format
npm run lint
npm run format
```

---

## 📁 Project Structure

```
startup-agent/
├── src/
│   ├── agent/          # GPT-4o agent logic, tools, thread management
│   ├── api/            # Express routes and middleware
│   ├── components/     # React UI components
│   ├── hooks/          # Custom React hooks
│   ├── lib/            # Shared utilities (db, openai client, logger)
│   ├── pages/          # React page-level components
│   ├── prompts/        # System prompts and lifecycle stage templates
│   └── types/          # Shared TypeScript type definitions
├── tests/
│   ├── unit/           # Vitest unit tests (mirrors src/)
│   ├── integration/    # API + DB integration tests
│   └── e2e/            # Playwright end-to-end tests
├── docs/
│   ├── spec.md         # Feature specification
│   └── adr/            # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   └── skills/
├── CLAUDE.md           # Context for Claude AI agent
├── AGENTS.md           # Context for OpenAI Codex / GPT agents
└── TODO.md             # Evidence-gated task breakdown
```

---

## 🤝 Contributing

1. **Read** `TODO.md` and pick an unchecked task from the current phase
2. **Write failing tests first** (red phase) before any implementation
3. **Implement** until tests pass (green phase) — no skipping steps
4. **Review your diff** manually before committing
5. **Commit** with a descriptive message referencing the TODO item
6. **Open a PR** — include evidence: test output, screenshots, or logs
7. **Update** `CLAUDE.md` / `AGENTS.md` if you learned something new

> PRs without evidence (failing tests → passing tests) will not be merged.

---

## 📄 License

MIT © [joshiujjwal](https://github.com/joshiujjwal)
