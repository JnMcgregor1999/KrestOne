# AGENTS.md

Two-app monorepo: `backend/` (.NET 10 web API) + `frontend/` (Angular 22). Git repo on `main` with remote `origin` (GitHub).

**Source of truth is `docs/`.** Read it before touching code and never contradict it silently. Architecture, conventions and quality gates are defined only there:

- `docs/README.md` — master index (incl. quality gates overview).
- `docs/architecture/backend.md` — .NET backend architecture; "definition of done" in §12–13.
- `docs/architecture/frontend.md` — Angular frontend architecture; "definition of done" in §3, §5, §7.

Specialized subagents live in `.opencode/agent/backend.md` and `.opencode/agent/frontend.md`; they reference `docs/` and do not duplicate its rules.

If a code decision contradicts a spec document, fix the code or propose updating the document; never ignore it. The documents are written in Spanish; follow them.
