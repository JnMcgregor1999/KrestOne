# Especificación de Arquitectura del Frontend (Angular)

> KrestOne — _One Path Every Destination_
> Última revisión: 2026-08-09

## 1. Propósito y contexto

Este documento define la arquitectura del frontend, construido sobre Angular 22.
Comparte el mismo principio rector que el backend: **un solo camino** — una
arquitectura, unas convenciones y un orden.

En el momento de esta revisión el frontend es un **scaffold**: arranca con
`bootstrapApplication` y un componente raíz `App`, sin rutas reales ni dominio
de negocio. Este documento es la referencia a partir de la cual se construye el
resto.

## 2. Principios de diseño

1. **Convención sobre configuración**: un solo estilo en todo el frontend;
   nada de patrones exóticos.
2. **Angular moderno**: standalone components y signal-reactividad. Sin
   `NgModule` salvo que el framework lo exija.
3. **Estilos con Tailwind v4**: CSS-first, sin archivo de configuración.
4. **Dependencias mínimas**: nada se instala si el framework o un paquete ya
   presente lo resuelve.
5. **Lenguaje**: el código y los nombres se escriben en inglés; los
   documentos en español.

## 3. Herramientas

| Herramienta     | Detalle                                                                                                        |
| --------------- | -------------------------------------------------------------------------------------------------------------- |
| Framework       | Angular 22 (`@angular/core` ^22.1)                                                                             |
| Package manager | **pnpm** (`packageManager: pnpm@11.20.0`). Nunca npm/yarn.                                                     |
| Tests           | Vitest + jsdom vía el builder `@angular/build:unit-test`                                                       |
| Estilos         | Tailwind v4 CSS-first (`@tailwindcss/postcss` + `@import 'tailwindcss'`)                                       |
| Lint            | ESLint flat config (`eslint.config.js`): `@angular-eslint` 22 + `typescript-eslint` + `eslint-config-prettier` |
| Formato         | Prettier (`.prettierrc`: singleQuote, printWidth 100, parser angular para HTML)                                |

Dependencias de runtime actuales: `@angular/forms` ^22.1 además del core/router
de Angular y `rxjs`. Solo se añade un paquete si el framework o una dependencia
presente no lo resuelve. En devDependencies, además de las de build/tests, vive
el stack de ESLint (`@angular-eslint/*`, `eslint`, `typescript-eslint`,
`@eslint/js`, `eslint-config-prettier`) y Prettier.

### Scripts (desde `frontend/`)

- `pnpm start` — servidor de desarrollo en :4200.
- `pnpm build` — build de producción.
- `pnpm test` — tests; para una sola pasada `pnpm test --watch=false`.
  **No** pasar `-- --run` a `ng test`: falla la validación del builder.
- `pnpm lint` — ESLint (`ng lint`) sobre `src/**/*.ts` y `src/**/*.html`.
- `pnpm format` — Prettier en modo write sobre todo el repo, respetando
  `.prettierignore`.
- `pnpm format:check` — Prettier en modo check (no modifica archivos).

**Decisión**: ESLint + scripts `lint`/`format` son la herramienta de calidad
oficial del frontend. La regla de "código completo" (sección 7) incluye
`pnpm lint` y `pnpm format:check` limpios.

## 4. Estructura de carpetas

```
frontend/
├── src/
│   ├── index.html
│   ├── main.ts               # bootstrapApplication
│   ├── styles.css            # @import 'tailwindcss'
│   └── app/
│       ├── app.config.ts     # providers (ApplicationConfig)
│       ├── app.ts            # componente raíz (clase App)
│       ├── app.html
│       ├── app.css
│       ├── app.routes.ts     # definición de rutas (Routes)
│       └── app.spec.ts       # tests co-locados
├── public/                   # assets estáticos (favicon.ico)
├── angular.json
├── eslint.config.js          # flat config de ESLint (@angular-eslint + prettier)
├── .editorconfig             # reglas de editor (indentación, fin de línea)
├── .postcssrc.json           # configuración de PostCSS (@tailwindcss/postcss)
├── .prettierignore           # excluye node_modules/, dist/, .angular/, pnpm-lock.yaml
├── package.json
├── pnpm-lock.yaml
├── .prettierrc
└── tsconfig.json / tsconfig.app.json / tsconfig.spec.json
```

`dist/` es la salida de build (`pnpm build`) y no se versiona ni se documenta
como fuente.

## 5. Convenciones de código

### Standalone + signals

- Componentes **standalone**: sin `NgModule`, se registran por importación
  directa en su plantilla o rutas.
- Estado reactivo con **signals** (`signal`, `computed`, `effect`); no se usa
  `Zone.js` como fuente primaria de estado de componente.
- Arranque con `bootstrapApplication` en `src/main.ts` (ver `src/main.ts`).

### Nombrado y estructura de archivos

- La clase del componente raíz es **`App`** (no `AppComponent`), con archivos
  co-locados `app.ts` / `app.html` / `app.css`.
- Las rutas viven en `src/app/app.routes.ts` y exportan la constante `routes`.
- Cada componente nuevo sigue el mismo patrón co-locado
  (`xxx.ts` / `xxx.html` / `xxx.css` + `xxx.spec.ts`).

### Estilos

- Tailwind v4 es **CSS-first**: no existe `tailwind.config.js` y **no debe
  crearse**. La configuración vive en `src/styles.css` vía
  `@import 'tailwindcss'` y `@tailwindcss/postcss`.
- CSS propio solo para casos que Tailwind no resuelve; clases utilitarias
  preferidas.

### Lint y formato

- **ESLint** con flat config (`eslint.config.js`): reglas recomendadas de
  `@eslint/js`, `typescript-eslint` y `@angular-eslint` (TS + plantillas),
  cerrado con `eslint-config-prettier` para desactivar reglas en conflicto con
  Prettier. Selectores con prefix `app` (atributos camelCase, elementos
  kebab-case).
- **Prettier** con `singleQuote` y `printWidth: 100`. HTML parseado con el parser
  `angular`.
- Scripts: `pnpm lint`, `pnpm format` (write) y `pnpm format:check` (check).
  `.prettierignore` excluye `node_modules/`, `dist/`, `.angular/` y
  `pnpm-lock.yaml`.

## 6. Persistencia y servicios (estrategia)

- **Sin decisión aún**: no hay servicios HTTP ni persistencia local definidos.
- Cuando surjan, los servicios de acceso a datos/API se agrupan por feature y
  se exponen vía providers de `ApplicationConfig` (`app.config.ts`), inyectados
  con la DI de Angular.
- El estado global (si se necesita) se modela con signals/servicios de
  aplicación, no con librerías externas de state management, salvo que este
  documento lo actualice.

## 7. Estrategia de pruebas

- **Vitest + jsdom** a través del builder `@angular/build:unit-test`.
- Los tests son **co-locados** junto a su componente/feature (`*.spec.ts`).
- Una sola pasada: `pnpm test --watch=false`. Nunca `-- --run`.
- Regla: el código se considera completo solo cuando sus tests relevantes
  pasan, `pnpm lint` y `pnpm format:check` están limpios y `pnpm build`
  compila.

## 8. Roadmap (siguientes pasos ordenados)

1. **Definir la navegación base**: primeras rutas reales en `app.routes.ts`
   (layout y páginas vacías).
2. **Crear el primer feature**: componente standalone con signals y sus tests.
3. **Establecer el estado global** si el dominio lo requiere.

## 9. Referencia

- Documento maestro: `docs/README.md`.
- Arquitectura backend: `docs/architecture/backend.md`.
