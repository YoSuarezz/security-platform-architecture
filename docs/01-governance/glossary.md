# Glosario oficial

> **Autoridad de nombres** del proyecto. Derivado del glosario del Documento Base (§23).  
> Toda documentación, diagrama, política Rego, evento y campo de datos debe usar estos términos.  
> Estado: `Accepted` (v1.0 — se amplía solo mediante ADR o actualización controlada).

| Término | Definición en este proyecto |
|---|---|
| ABAC | Control de acceso basado en atributos del sujeto, recurso, acción y entorno. |
| Acción | Operación protegida sobre un recurso: leer, crear, aprobar, eliminar, etc. |
| Aplicación protegida | Sistema consumidor que delega una parte del control de seguridad a la plataforma. |
| Arquitectura Hexagonal | Patrón que separa el dominio de sus adaptadores mediante puertos, usado para implementar Clean Architecture en este proyecto. |
| Auditoría | Evidencia estructurada de eventos y decisiones de seguridad. |
| Claim | Afirmación incluida en una evidencia de identidad, como un JWT; debe validarse y normalizarse. |
| Cloud Enable | Preparación de la arquitectura para migrar a Kubernetes o una plataforma cloud, sin que ello implique desplegar en nube desde el MVP. |
| Cloud Native | Conjunto de principios de construcción: servicios desacoplados, configuración externa, observabilidad, contenedores, escalabilidad horizontal y resiliencia. |
| DecisionAcceso | Resultado estructurado de una evaluación: `ALLOW`, `DENY` o `INDETERMINATE`. |
| Harness Engineering | Estrategia de ingeniería del proyecto orientada a calidad, mantenibilidad, automatización, verificabilidad y evolución. |
| IdP | Proveedor de identidad que autentica usuarios o emite evidencia de identidad. |
| Línea base | Conjunto mínimo de visión, documentación, arquitectura, módulos, estrategias de Git, CI/CD y pruebas que debe existir antes de iniciar historias de usuario funcionales. |
| OPA | Motor de evaluación de políticas Open Policy Agent. |
| PBAC | Control de acceso guiado por políticas. |
| PDP | Componente que toma u orquesta una decisión de acceso. |
| PEP | Punto que intercepta una solicitud y aplica la decisión de acceso. |
| RBAC | Control de acceso basado en roles. |
| REBAC | Control de acceso basado en relaciones entre entidades. |
| Recurso | Elemento que se protege: ruta, función, entidad, documento o instancia de negocio. |
| Rego | Lenguaje utilizado por OPA para expresar políticas. |
| SolicitudAcceso | Representación normalizada de una petición que será evaluada. |
| Spring Modulith | Marco usado para organizar el proyecto en módulos internos verificables, alineados con capacidades de negocio. |
| SurrealDB | Base de datos principal decidida para la plataforma; se usará para datos y relaciones de seguridad. |
| Tenant | Límite organizacional o de aislamiento lógico para datos y autorizaciones. |
| Usuarios (BC) | Contexto del sujeto de dominio interno; distinto de Identidad y autenticación. |
| Identidad y autenticación (BC) | Contexto que valida/normaliza evidencia del IdP (sesión, claims); no autoriza. |
| Autorización (BC) | Contexto de evaluación `SolicitudAcceso` → OPA → `DecisionAcceso` → `EventoAcceso`. |
| Políticas de acceso (BC) | Catálogo/metadatos/versión de políticas; no sustituye a OPA como motor. |
| Perfiles (BC) | Agrupación de roles para facilitar asignación; distinto de Roles y de Asignaciones. |
| Keycloak | IdP del MVP (self-hosted); OIDC/OAuth2. La arquitectura permanece agnóstica al proveedor (ADR-013). |
| Administrador de Seguridad | Único rol administrativo de la plataforma en el MVP; gestiona tenants, apps, recursos, roles, perfiles, asignaciones, políticas y consulta auditoría. |
| Gestión Académica Universitaria | Primera aplicación de validación del MVP (`gestion-academica`); dominio educación superior; cada universidad es un tenant. |
| Deny-by-default | Semántica: sin política aplicable ⇒ DENY; deny explícito gana sobre allow (ADR-012). |

## Regla de cambio

Ningún término se renombra ni redefine sin actualizar este archivo y, si el cambio implica una decisión arquitectónica, un ADR.
