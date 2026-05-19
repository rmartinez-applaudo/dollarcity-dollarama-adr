# ADR: Monorepo, Migración a Vertical Slice Architecture y adopción de .NET Aspire en AccessQR

---

### Submitters

- Ricardo Martínez (Applaudo Studios)

---

## Change Log

| Fecha      | Versión | Autor             | Descripción                                              |
|------------|---------|-------------------|----------------------------------------------------------|
| 2026-05-19 | 1.0     | Ricardo Martínez  | ADR inicial: monorepo, Vertical Slice y .NET Aspire      |

- [pending]() 2026-05-19

---

## Referenced Use Case(s)

- **AccessQR – Consolidación de codebase y modernización de arquitectura backend**

Esta ADR abarca tres decisiones arquitectónicas relacionadas y que se habilitan mutuamente:
1. Unificación de los repositorios de Backend y Frontend en un único monorepo.
2. Migración de la arquitectura del Backend de Clean Architecture a Vertical Slice Architecture.
3. Adopción de .NET Aspire para la orquestación de herramientas externas en el entorno de desarrollo local, comenzando por Keycloak.

---

## Context

El proyecto **AccessQR** de Dollarcity cuenta actualmente con dos repositorios independientes:

- `dollarcity-access-qr-be`: Backend en **.NET** con **Clean Architecture** (proyectos `Domain`, `Application`, `Infrastructure`, `Web`).
- `dollarcity-access-qr-fe`: Frontend en **TypeScript/React** organizado como un **pnpm workspace** con múltiples aplicaciones (`webapp`, `backoffice`, `common`, `keycloak-theme`).

### Motivaciones para el monorepo

La separación en dos repositorios genera fricción operacional concreta:

- **Sincronización de contratos API**: Cambios en endpoints del Backend requieren coordinar ramas, PRs y versiones entre dos repos distintos para actualizar el cliente HTTP del Frontend. 
- **Toolchain duplicado**: Ambos repos mantienen configuraciones independientes de CI/CD, linters, editores (`.vscode`, `.cursor`, etc.), `mise.toml` y herramientas de calidad de código, incrementando el costo en tiempo de mantenimiento.
- **Onboarding costoso**: Un nuevo desarrollador debe clonar y configurar dos repositorios para trabajar en cualquier funcionalidad full-stack.
- **Revisiones de código fragmentadas**: Un PR full-stack requiere abrir PRs en dos repositorios, perdiendo contexto y cohesión en la revisión.
- **Contexto para tools AI**: un repositorio unificado garantiza que todo el contexto que surge del codigo esta en un solo lugar y con esto aumentar la probabilidad de consistencia y coherencia de la fase de tasks del `speckit`.

### Motivaciones para Vertical Slice Architecture

La arquitectura actual de Clean Architecture organiza el código por **capas técnicas** (`Domain`, `Application`, `Infrastructure`, `Web`), lo que provoca:

- **Alto acoplamiento transversal**: Agregar o modificar una feature (ej. gestión de accesos QR) implica tocar cuatro proyectos distintos con sus respectivas interfaces, handlers, repositorios y endpoints.
- **Carpetas compartidas que crecen sin control**: `Application/Common` y `Domain/Common` concentran abstracciones que todas las features consumen, creando un punto de cambio de alto riesgo.
- **Dificultad para razonar sobre una feature completa**: El código de una sola historia de usuario está distribuido en múltiples capas y proyectos, dificultando la comprensión y el mantenimiento.

Vertical Slice Architecture organiza el código por **feature** en lugar de por capa, colocando toda la lógica relacionada (command/query, validación, persistencia, endpoint) en una sola unidad cohesiva, lo que reduce el acoplamiento accidental y mejora la navegabilidad del código.

### Motivaciones para .NET Aspire

El backend depende de **Keycloak** como proveedor de identidad. Actualmente, los desarrolladores gestionan esta dependencia manualmente (Docker Compose, instalación local, etc.), lo cual:

- Introduce inconsistencias entre entornos de desarrollo.
- Requiere pasos manuales de configuración no versionados.
- Dificulta la incorporación de nuevas herramientas externas en el futuro.

**.NET Aspire** es el orquestador de infraestructura local oficial del ecosistema .NET. Permite declarar y levantar dependencias externas (contenedores, servicios) de forma reproducible, integrada al ciclo de vida de la solución.

---

## Proposed Design

### 1. Estructura del monorepo

El monorepo `dollarcity-access-qr` tendrá la siguiente estructura raíz:

```
dollarcity-access-qr/
├── backend/                  # Código fuente del Backend (.NET)
│   ├── src/
│   │   ├── Web/              # API entry point + Aspire AppHost
│   │   ├── Features/         # Módulos de Vertical Slice (ver sección 2)
│   │   └── Infrastructure/   # Persistencia, clientes externos, identidad
│   ├── tests/
│   └── AccessQr.sln
├── frontend/                 # Código fuente del Frontend (pnpm workspace)
│   ├── webapp/
│   ├── backoffice/
│   ├── common/
│   └── keycloak-theme/
├── shared-specs/             # submodulo de dollarcity-app-specs
├── infra/                    # Infraestructura cloud (Azure, IaC)
├── docs/                     # Documentación técnica
├── .github/                  # Pipelines CI/CD unificados
└── mise.toml                 # Toolchain unificado
```

**Puntos clave:**
- Los repositorios `dollarcity-access-qr-be` y `dollarcity-access-qr-fe` se migran a subcarpetas `backend/` y `frontend/` del nuevo monorepo.
- Las configuraciones de herramientas (`.editorconfig`, `mise.toml`, linters) se unifican a nivel raíz, permitiendo sobrescrituras por subcarpeta donde sea necesario.
- Cada subcarpeta (`backend/`, `frontend/`) mantiene su propio `.gitignore` y puede tener su historia de Git preservada mediante `git subtree` o `git filter-repo` durante la migración.
- por medio de APM se administran los contextos e instrucciones de ambos stacks, Csharp y React

### 2. Migración a Vertical Slice Architecture (Backend)

La nueva estructura del Backend elimina los proyectos `Domain` y `Application` como capas independientes. En su lugar, el código se organiza por **feature slice**:

```
backend/src/
├── Features/
│   ├── AccessPoints/
│   │   ├── CreateAccessPoint/
│   │   │   ├── CreateAccessPointCommand.cs
│   │   │   ├── CreateAccessPointHandler.cs
│   │   │   ├── CreateAccessPointValidator.cs
│   │   │   └── CreateAccessPointEndpoint.cs
│   │   ├── GetAccessPoint/
│   │   │   └── ...
│   │   └── AccessPointsModule.cs   # Registro de DI del feature
│   ├── Users/
│   │   └── ...
│   └── QrCodes/
│       └── ...
├── Infrastructure/
│   ├── Data/                        # EF Core DbContext, Migrations
│   ├── Identity/                    # Integración Keycloak
│   └── DependencyInjection.cs
└── Web/
    ├── Program.cs
    └── AppHost/                     # Proyecto .NET Aspire AppHost
```

**Capas conservadas:**
- `Infrastructure`: Se mantiene como proyecto separado dado que la persistencia y los clientes de identidad son transversales a todos los slices.
- `Web`: Entry point de la API ASP.NET Core y host de Aspire.

**Patrones adoptados:**
- Cada slice es autocontenida: define su propio command/query (MediatR), validador (FluentValidation) y endpoint (minimal API o controller).
- Los slices pueden compartir tipos de dominio livianos (Value Objects, Entities) a través de un módulo `SharedKernel` dentro de `Features/`, en lugar de un proyecto `Domain` separado.
- Los tests reflejan la misma estructura de slices: `tests/Features/AccessPoints/CreateAccessPoint/`.

### 3. .NET Aspire para orquestación de Keycloak

Se agrega un proyecto **AppHost** de .NET Aspire dentro de `backend/src/Web/AppHost/`:

```csharp
// AppHost/Program.cs
var builder = DistributedApplication.CreateBuilder(args);

var keycloak = builder.AddKeycloakContainer("keycloak")
    .WithRealmImport("../../../infra/keycloak/realm-export.json");

var api = builder.AddProject<Projects.Web>("api")
    .WithReference(keycloak);

builder.Build().Run();
```

**Responsabilidades de Aspire:**
- Levantar un contenedor de **Keycloak** con el realm de AccessQR preconfigurado al ejecutar el proyecto en modo desarrollo.
- Inyectar automáticamente la URL y credenciales de Keycloak como variables de entorno en el proyecto `Web`.
- Proveer el **Aspire Dashboard** para observabilidad local (logs, trazas, métricas) sin configuración adicional.
- Servir como punto de extensión para agregar otras dependencias externas en el futuro (bases de datos, caches, message brokers).

**Impacto en configuración:**
- `appsettings.Development.json` deja de requerir la URL de Keycloak hardcodeada; Aspire la inyecta en runtime.
- El `docker-compose.yml` de desarrollo (si existe) queda deprecado en favor del AppHost.

### 4. Impacto en CI/CD

- Los pipelines se consolidan en `.github/` a nivel raíz del monorepo.
- Se implementa **path filtering** para ejecutar únicamente el pipeline correspondiente según los archivos modificados (`backend/**` → pipeline .NET; `frontend/**` → pipeline Node.js).
- El AppHost de Aspire se utiliza exclusivamente en desarrollo local; en CI y producción, Keycloak se gestiona a través de la infraestructura existente en Azure.

---

## Considerations

### Alternativa A: Mantener repositorios separados con mejoras de coordinación

- **Descripción**: Continuar con dos repos y mejorar la sincronización mediante submódulos más estrictos o scripts de automatización.
- **Ventajas**: Sin migración; menor riesgo inmediato.
- **Desventajas**: No resuelve los problemas de raíz de coordinación y onboarding. La deuda técnica de dos toolchains persiste.
- **Conclusión**: Descartada. Aplaza el problema sin resolverlo.

---

### Alternativa B: Mantener Clean Architecture con refactoring de carpetas

- **Descripción**: Reestructurar las carpetas dentro de Clean Architecture para reducir la dispersión de features.
- **Ventajas**: Menor cambio estructural.
- **Desventajas**: No elimina el acoplamiento de capas. Las mismas fricciones de navegación y cambio de feature persisten.
- **Conclusión**: Descartada. Vertical Slice es una evolución más coherente con el tamaño y el ritmo de cambio del proyecto.

---

### Alternativa C: Docker Compose en lugar de .NET Aspire

- **Descripción**: Centralizar Keycloak y otras dependencias en un `docker-compose.yml` a nivel de monorepo.
- **Ventajas**: Independiente del stack tecnológico del Backend.
- **Desventajas**: No integra con el ciclo de vida de la aplicación .NET; requiere gestión manual adicional. No provee observabilidad integrada.
- **Conclusión**: Descartada para el entorno de desarrollo local. Aspire ofrece mejor integración y experiencia de desarrollo para proyectos .NET. Docker Compose puede coexistir para otros entornos.

---

### Consideraciones sobre la migración de historia de Git

Preservar la historia de commits de ambos repositorios durante la migración al monorepo no sera necesario ya que se aprovecha la ausencia de lógica de negocio en BE para correr la generacion inicial en el repo que antes era `dollarcity-access-qr-fe` solo se renombra. de esa manera la transferencia a este nuevo monorepo va a ser seamless

---

## Decision

Se aprueba:

1. **Monorepo unificado** para `dollarcity-access-qr` con `backend/` y `frontend/` como subcarpetas principales, historia de Git preservada, y `shared-specs` como submodulo de Spec-Driven-Development.

2. **Migración del Backend a Vertical Slice Architecture**, manteniendo `Infrastructure` como proyecto transversal y organizando toda la lógica de negocio en slices autocontenidos bajo `Features/`.

3. **Adopción de .NET Aspire** mediante un proyecto `AppHost` para orquestar Keycloak (y futuras dependencias externas) en el entorno de desarrollo local. En CI y producción, la infraestructura se gestiona por los mecanismos existentes en Azure.

### Requerimientos no cubiertos por esta ADR

- Mapeo de las specs hacia los slices específicos resultantes del spec-driven-development
- Configuración detallada de pipelines CI/CD con path filtering (se definirá en ADR o ticket de CI/CD).

---

## References

- [Vertical Slice Architecture – Jimmy Bogard](https://www.jimmybogard.com/vertical-slice-architecture/)
- [.NET Aspire – Documentación oficial](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/aspire-overview)
- [.NET Aspire Keycloak integration](https://learn.microsoft.com/en-us/dotnet/aspire/authentication/keycloak-component)
- [git filter-repo – Rewrite history to subdirectory](https://github.com/newren/git-filter-repo)
- [Monorepo patterns – Nx / Turborepo](https://turbo.build/repo/docs/handbook/what-is-a-monorepo)
- Plantilla ADR EdgeX Foundry: https://docs.edgexfoundry.org/2.3/design/adr/template/
