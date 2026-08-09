# KrestOneWeb

Angular 22 frontend of KrestOne — _One Path Every Destination_.

The source of truth for this project is `docs/` at the repository root.
Read `docs/README.md` and `docs/architecture/frontend.md` before changing code.

## Tooling

- Package manager: **pnpm** only (`pnpm@11.20.0`). Never npm/yarn.
- Stack: standalone components, signal-reactivity, Tailwind v4 (CSS-first).
- Linting: ESLint flat config (`eslint.config.js`, @angular-eslint + typescript-eslint).
- Formatting: Prettier (`.prettierrc`).

## Commands (run from `frontend/`)

- `pnpm start` — development server on :4200.
- `pnpm build` — production build (outputs to `dist/`).
- `pnpm test` — unit tests (Vitest). For a single pass: `pnpm test --watch=false`.
  Do **not** pass `-- --run` to `ng test`: it fails the builder validation.
- `pnpm lint` — ESLint over `src/**/*.ts` and `src/**/*.html`.
- `pnpm format` — Prettier write over the repo (respects `.prettierignore`).
- `pnpm format:check` — Prettier check (no file changes).

Code is considered complete only when lint, format:check, tests and build pass.
