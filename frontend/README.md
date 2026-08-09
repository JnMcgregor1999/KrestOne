# KrestOneWeb

Frontend Angular 22 de KrestOne — _One Path Every Destination_.

La fuente de verdad de este proyecto es `docs/` en la raíz del repositorio.
Lee `docs/README.md` y `docs/architecture/frontend.md` antes de cambiar código.

## Herramientas

- Package manager: **solo pnpm** (`pnpm@11.20.0`). Nunca npm/yarn.
- Stack: standalone components, signal-reactividad, Tailwind v4 (CSS-first).
- Lint: ESLint flat config (`eslint.config.js`, @angular-eslint + typescript-eslint).
- Formato: Prettier (`.prettierrc`).

## Comandos (desde `frontend/`)

- `pnpm start` — servidor de desarrollo en :4200.
- `pnpm build` — build de producción (salida en `dist/`).
- `pnpm test` — tests unitarios (Vitest). Para una sola pasada:
  `pnpm test --watch=false`. **No** pasar `-- --run` a `ng test`: falla la
  validación del builder.
- `pnpm lint` — ESLint sobre `src/**/*.ts` y `src/**/*.html`.
- `pnpm format` — Prettier en modo write sobre el repo (respeta `.prettierignore`).
- `pnpm format:check` — Prettier en modo check (no modifica archivos).

El código se considera completo solo cuando pasan lint, format:check, tests y
build.
