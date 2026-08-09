# CLAUDE.md

> KrestOne — *One Path Every Destination*

Este archivo configura a Claude Code para trabajar en este monorepo. Es
coherente con `AGENTS.md`; la fuente de verdad es `docs/`.

## Regla principal

Ante cualquier tarea que toque código, leer primero `docs/README.md` (índice
maestro) y el documento de arquitectura correspondiente:
`docs/architecture/backend.md` (backend .NET) o
`docs/architecture/frontend.md` (frontend Angular). Los documentos se escriben
en español.

No contradecir la especificación: si una decisión de código contradice un
documento, corregir el código o proponer la actualización del documento; nunca
ignorarlo en silencio.

## Calidad del frontend

En tareas de `frontend/`, además de build y tests, deben pasar ESLint
(`pnpm lint`) y Prettier (`pnpm format:check`) desde `frontend/`. Solo así el
código se considera completo.

## Referencia

Para detalles completos (estructura, verificación, gotchas), leer `AGENTS.md`,
`docs/README.md` y `docs/architecture/`.

Los subagentes especializados autocontenidos viven en `agent/backend.md` y
`agent/frontend.md`.
