# AGENTS.md

Two-app monorepo: `backend/` (.NET 10 web API) + `frontend/` (Angular 22). Git repo on `main` with remote `origin` (GitHub).

**Source of truth is `docs/`.** Read it before touching code and never contradict it silently:

- `docs/README.md` — master index.
- `docs/architecture/backend.md` — .NET backend architecture (layers, dependencies, conventions, gotchas).
- `docs/architecture/frontend.md` — Angular frontend architecture (conventions, tooling, testing).

Self-contained specialized subagents live in `agent/backend.md` and `agent/frontend.md`.

Frontend quality gates: ESLint (`pnpm lint`) + Prettier (`pnpm format:check`) from `frontend/` must pass alongside build and tests before code is considered complete.

If a code decision contradicts a spec document, fix the code or propose updating the document; never ignore it. The documents are written in Spanish; follow them.
