# Documentación de KrestOne

> **One Path Every Destination**

Este directorio es la **fuente de verdad** del proyecto. Los agentes de IA y los
desarrolladores deben leer estos documentos antes de escribir o modificar código,
para que el proyecto mantenga un orden y una coherencia entre backend y frontend.

## Mapa de documentos

| Documento | Contenido | Estado |
| --- | --- | --- |
| [architecture/backend.md](architecture/backend.md) | Especificación de arquitectura del backend .NET | Definido |
| [architecture/frontend.md](architecture/frontend.md) | Especificación de arquitectura del frontend Angular | Definido |
| `architecture/data.md` | Modelo de datos y estrategia de persistencia | Pendiente |

## Reglas de uso para agentes de IA

1. **Leer antes de actuar**: ante cualquier tarea que toque backend, leer
   `architecture/backend.md` primero.
2. **No contradecir la especificación**: si una decisión de código contradice un
   documento, corregir el código o proponer la actualización del documento; nunca
   ignorarlo en silencio.
3. **Lenguaje**: los documentos se escriben en español con términos técnicos en
   inglés.
4. **Todo cambio arquitectónico documentado**: cualquier cambio en capas,
   dependencias o flujo debe reflejarse aquí antes o junto con el código.

## Configuración de agentes de IA

El repo incluye archivos de configuración para que los agentes de IA trabajen
con estas reglas sin repetirlas en cada sesión:

| Herramienta | Archivo | Contenido |
| --- | --- | --- |
| Claude Code | `CLAUDE.md` (raíz) | Índice que referencia `docs/` (reglas y documentos de arquitectura) |
| opencode | `AGENTS.md` (raíz) | Instrucciones y referencia a `docs/` |
| opencode | `agent/` (raíz) | Subagentes `backend.md` y `frontend.md` especializados (autocontenidos) |

Estos archivos son un reflejo de las reglas de este documento: si cambia una
regla aquí, debe reflejarse también en ellos.

## Referencias rápidas

- `README.md` (raíz) — presentación del monorepo e índice hacia `docs/`.
- `frontend/README.md` — referencia local del subproyecto `frontend/`
  (herramientas y comandos), coherente con
  `docs/architecture/frontend.md`.

## Cómo evolucionar esta documentación

- Cada documento vive en `docs/` y sigue la estructura de su propia tabla de
  contenidos.
- Cuando un documento entra en conflicto con la realidad del código, el
  documento se actualiza (es la fuente de verdad).
- Un documento se marca como "Pendiente" hasta que la sección de roadmap del
  documento correspondiente la cierre.
