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

## Calidad

La arquitectura, las convenciones y los quality gates del backend y del
frontend se definen únicamente en `docs/architecture/backend.md` (§12–§13) y
`docs/architecture/frontend.md` (§3, §5 y §7). Este archivo no las duplica.

## Referencia

- Índice maestro: `docs/README.md`.
- Arquitectura backend: `docs/architecture/backend.md`.
- Arquitectura frontend: `docs/architecture/frontend.md`.
- Subagentes que referencian `docs/` (no duplican sus reglas):
  `agent/backend.md` y `agent/frontend.md`.
