# Especificación de Arquitectura del Backend (.NET)

> KrestOne — _One Path Every Destination_
> Última revisión: 2026-08-09

## 1. Propósito y contexto

KrestOne es el nombre del producto: _Krest_ (cima) + _One_ (uno), con la visión
**"One Path Every Destination"**. Esta frase es también una directriz técnica:
el proyecto debe tener **un solo camino** (una arquitectura, unas convenciones y
un orden) hacia cualquier destino de negocio que surja.

Esta especificación define la arquitectura del backend, construido sobre .NET 10.
En el momento de esta revisión el backend es un **scaffold**: los cinco proyectos
existen. Este documento es la referencia a partir de la cual se construye el resto.

## 2. Principios de diseño

1. **Dependencias hacia el dominio**: todas las capas apuntan al centro
   (`Domain`). Nada de nivel superior conoce detalles de implementación de nivel
   inferior.
2. **Persistencia desacoplada**: el modelo de dominio y los casos de uso no
   dependen de la base de datos. El proveedor concreto se decide más adelante.
3. **Integraciones aisladas**: todo servicio de terceros vive en `Extender`,
   nunca se filtra a las demás capas.
4. **Consistencia sobre complejidad**: se prefieren soluciones convencionales y
   repetibles a patrones exóticos. Un solo estilo en todo el backend.
5. **Configuración explícita**: nada de secretos ni valores ambientales
   hardcodeados en el código.
6. **Composición en la raíz**: la capa `Api` es la única que compone e inyecta
   dependencias.

## 3. Estructura de proyectos y reglas de dependencia

Solución: `backend/KrestOneService.slnx`

| Proyecto                                  | Responsabilidad                                                                   | Referencias permitidas  |
| ----------------------------------------- | --------------------------------------------------------------------------------- | ----------------------- |
| `FactorK.KrestOne.Service.Api`            | Punto de entrada, composición de DI, HTTP, middleware, OpenAPI                    | Los otros 4             |
| `FactorK.KrestOne.Service.Application`    | Casos de uso, servicios de aplicación, DTOs, validación, contratos de repositorio | `Domain`                |
| `FactorK.KrestOne.Service.Domain`         | Entidades, Value Objects, Domain Events, enums, lógica de negocio pura            | **Ninguna**             |
| `FactorK.KrestOne.Service.Extender`       | Integraciones externas (adaptadores a servicios de terceros)                      | `Application`, `Domain` |
| `FactorK.KrestOne.Service.Infrastructure` | Persistencia, repositorios, configuración de bajo nivel                           | `Application`, `Domain` |

### Reglas de dependencia (invariantes)

- `Domain` **no referencia** nada.
- `Application` solo referencia `Domain`.
- `Extender` e `Infrastructure` solo referencian `Application` y `Domain`, nunca
  entre sí.
- `Api` referencia todos los proyectos, pero **no contiene lógica de negocio**:
  solo orquestación de arranque y definición del contrato HTTP.
- Ninguna capa inferior conoce HTTP, DI ni configuración de la aplicación.

### Gotcha de mayúsculas (crítico)

Algunos `.csproj` escribían los `ProjectReference` con el nombre `Factork`
(k minúscula) mientras que los directorios y nombres reales son `FactorK`
(K mayúscula). Compila en macOS por ser case-insensitive, pero **fallaría en
Linux/CI**. Todas las referencias usan ya el casing correcto `FactorK.*`.

**Regla (invariante)**: al tocar referencias de proyecto, usar siempre el
casing real `FactorK.*`. Si se reintroduce `Factork`, el build en CI fallará.

## 4. Capa Domain

Convenciones de carpeta:

```
Domain/
├── Entities/
├── ValueObjects/
├── Enums/
└── Events/
```

- **Entidades**: objetos con identidad propia (`Id`), mutable dentro de su
  invariantes. No contienen lógica de infraestructura.
- **Value Objects**: objetos inmutables sin identidad, comparados por valor.
- **Enums**: estados y clasificaciones propias del dominio.
- **Domain Events**: hechos ocurridos en el dominio que `Application` puede
  consumir; no disparan efectos secundarios por sí solos.
- El dominio **no** expone objetos de persistencia ni DTOs de la API.

## 5. Capa Application

Convenciones de carpeta:

```
Application/
├── Services/            # Interfaz + implementación por caso de uso/feature
├── Dtos/
├── Common/
│   ├── Validation/
│   └── Results/         # Result objects y errores tipados
└── Abstractions/        # Contratos de repositorio y UnitOfWork
```

### Servicios de aplicación

Estilo elegido: **servicios de aplicación tradicionales** (interfaces +
implementaciones). **No se usa MediatR** ni CQRS.

Reglas:

- Un servicio expone una **interfaz** (`IXxxService`) en `Services/` y su
  implementación (`XxxService`) en el mismo proyecto.
- Cada método del servicio ejecuta **un caso de uso completo**: validar → cargar
  → orquestar → guardar → devolver.
- Los servicios reciben sus dependencias (repositorios, servicios de integración)
  por **constructor** (inyección de dependencias), nunca se crean internamente.
- Los servicios no conocen HTTP, la base de datos concreta ni los clientes de
  integración: usan las **abstracciones** de `Application` y `Domain`.

### DTOs y mapeo

- La API expone **DTOs**, nunca entidades de dominio.
- El mapeo entidad → DTO vive en `Application` (no en `Api`).
- El mapeo DTO → entidad lo manejan los propios servicios de aplicación.

### Validación

- La validación de **invariantes de negocio** vive en el dominio o en el servicio
  de aplicación que las impone.
- La validación de **entrada del contrato HTTP** vive en `Api`.
- Los errores de validación se representan con tipos explícitos (ver sección 10).

## 6. Capa Infrastructure

Convenciones de carpeta:

```
Infrastructure/
├── Persistence/
│   ├── Repositories/
│   └── Configurations/
└── Options/
```

Responsabilidades:

- **Persistencia**: implementación concreta de los contratos de repositorio y del
  `IUnitOfWork` definidos en `Application`. El proveedor está **aún por decidir**
  (EF Core + SQL Server o PostgreSQL son los candidatos preferidos).
- **Configuración de bajo nivel**: mapeos, opciones de conexión, seed de datos.
- **Sin lógica de negocio**: `Infrastructure` traduce lo que la base de datos
  entiende, no decide reglas.

## 7. Capa Extender

Convenciones de carpeta:

```
Extender/
├── Clients/             # Clientes HTTP/adaptadores por servicio externo
├── Models/              # Contratos de datos de los servicios externos
└── Abstractions/        # Interfaces consumidas por Application
```

Responsabilidades:

- **Integraciones externas**: adaptadores a servicios de terceros (APIs REST,
  brokers de mensajería, email, pagos, etc.).
- Cada integración se modela con una **interfaz** que `Application` define como
  abstracción, y una **implementación concreta** que vive en `Extender`.
- Los clientes externos son **aislados**: sus tipos y dependencias (p. ej.
  `System.Net.Http`) no se filtran fuera de `Extender`.
- Toda integración debe ser **fallible de forma controlada**: timeouts,
  reintentos y degradación gestionados aquí, nunca en el dominio.

## 8. Capa Api

Convenciones de carpeta:

```
Api/
├── Endpoints/           # Minimal APIs o controllers
├── Middleware/
├── Dtos/                # Contratos de entrada/salida HTTP
├── Validation/
└── Configuration/       # Extensiones de IServiceCollection
```

Responsabilidades:

- **Composición raíz**: registro de todos los servicios, repositorios y
  clientes en el contenedor DI (registros agrupados en extensiones de
  `IServiceCollection`).
- **Contrato HTTP**: minimal APIs o controllers que delegan en los servicios de
  aplicación. Sin lógica de negocio en los endpoints.
- **Pipeline**: middleware global (manejo de errores, correlación, etc.).
- **OpenAPI**: documentado solo cuando `ASPNETCORE_ENVIRONMENT=Development`.
- **HTTP**: `Program.cs` es el punto de entrada y raíz de la composición.

## 9. Persistencia (estrategia)

- **Decisión actual**: proveedor de base de datos **aún por decidir**.
- Los casos de uso dependen de **abstracciones** (`IUnitOfWork`, repositorios)
  definidas en `Application`/`Domain`, por lo que el proveedor puede decidirse o
  cambiarse sin tocar las capas superiores.
- Candidatos preferidos (a evaluar en `docs/architecture/data.md`):
  1. EF Core + SQL Server
  2. EF Core + PostgreSQL (Npgsql)
- Ninguna entidad de dominio debe acoplarse a atributos o tipos de un ORM
  concreto (se evalúan mapeos por configuración/`IEntityTypeConfiguration`).

## 10. Manejo de errores y cross-cutting

- **Result objects**: los servicios de aplicación devuelven resultados tipados
  (`Success`/`Failure` con errores), en lugar de depender de excepciones para el
  flujo esperado. Las excepciones se reservan para errores inesperados.
- **Excepciones de dominio**: definidas en `Domain`, traducidas a respuestas HTTP
  adecuadas por `Api`.
- **Manejo centralizado**: middleware global de excepciones en `Api` que
  transforma cualquier error no controlado en una respuesta consistente
  (`ProblemDetails`).
- **Logging**: vía `ILogger<T>` estándar de .NET, con contextos estructurados.
  Sin librerías externas salvo que la sección de despliegue lo requiera.
- **Validación**: véase sección 5.

## 11. Configuración y secretos

- Todo valor configurable va en `appsettings.json` (y `appsettings.{Environment}.json`)
  con el **Options pattern** (`IOptions<T>`), o en variables de entorno.
- **Nunca** se comprometen secretos (contraseñas, tokens, connection strings
  reales) al repositorio.
- Los entornos se seleccionan con `ASPNETCORE_ENVIRONMENT`; solo `Development`
  habilita OpenAPI.

## 12. Estrategia de pruebas

Actualmente **no existen test projects**; se establece la estructura objetivo:

- **Tests de unidad** (`FactorK.KrestOne.Service.Tests.Unit`): lógica de dominio,
  servicios de aplicación con dependencias simuladas (mocks). No tocan red ni
  base de datos.
- **Tests de integración** (`FactorK.KrestOne.Service.Tests.Integration`):
  repositorios contra el proveedor real (local), clientes de `Extender` con
  contenedores/sandboxes.
- Regla: el código se considera completo solo cuando sus pruebas relevantes
  pasan y el proyecto compila.

Esta sección (junto con la sección 13) es la **fuente canónica** de la
definición de "código completo" y de verificación del backend. Los archivos
raíz y subagentes (`AGENTS.md`, `CLAUDE.md`, `agent/`) solo la referencian y no
duplican estas reglas.

## 13. Despliegue

- Objetivo de despliegue: **aún no definido** (Azure App Services, contenedores
  Docker u on-premises son opciones abiertas).
- Mientras tanto, el backend se mantiene **12-factor ready**:
  - Configuración por entorno (variables de entorno).
  - Stateless: sin estado en memoria dependiente de una instancia.
  - Contenedores como formato de entrega preferido: la definición del
    **Dockerfile en `backend/` es un objetivo** (ver roadmap, sección 14), no
    una pieza ya existente.
  - `dotnet build KrestOneService.slnx` como verificación mínima de salud.

### Ejecución local

- Perfiles definidos en `Properties/launchSettings.json`:
  - `http`: `http://localhost:5202`
  - `https`: `https://localhost:7199` + `http://localhost:5202`
- Ambos perfiles fijan `ASPNETCORE_ENVIRONMENT=Development`; solo ese entorno
  expone OpenAPI (ver sección 11).
- `appsettings.Development.json` **no se versiona**: está excluido por
  `.gitignore` para evitar que configuraciones locales o secretos de desarrollo
  lleguen al repositorio.

## 14. Roadmap (siguientes pasos ordenados)

1. **Definir el dominio**: primer agregado/entidad de negocio real en `Domain`.
2. **Decidir persistencia**: evaluar proveedor y redactar
   `docs/architecture/data.md`.
3. **Levantar la capa Application**: primer servicio de aplicación con su
   interfaz, DTOs y validación.
4. **Implementar repositorios** en `Infrastructure`.
5. **Crear los test projects** y adoptar la estrategia de la sección 12.
6. **Definir despliegue** y Dockerfile.
7. Redactar la **especificación de la API HTTP** (contratos, versionado) en
   cuanto existan endpoints reales.
