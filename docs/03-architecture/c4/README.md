# Diagramas C4

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| Nota | Los PNG son la fuente de verdad visual. Si se regeneran los diagramas se reemplazan los PNG; los Markdown los describen pero no los sustituyen. |

## Índice

| Nivel | Archivo | Qué muestra |
|---|---|---|
| L1 Context | [level1-context.png](level1-context.png) | Sistema vs actores y sistemas externos |
| L2 Containers | [level2-containers.png](level2-containers.png) | PEP · PDP · OPA · SurrealDB · Auditoría |
| L3 PEP | [level3-component-pep.png](level3-component-pep.png) | HTTP Adapter → Normalizador → Puerto decisión → Adaptador PDP → Aplicador enforcement |
| L3 PDP | [level3-component-pdp.png](level3-component-pdp.png) | HTTP Adapter → EvaluarAcceso → Constructor contexto; puertos: identidad · repositorio · OPA · auditoría |

## Resumen de componentes por nivel

### L1 — Actores y sistemas externos

| Actor / sistema | Tipo | Relación |
|---|---|---|
| Usuario final | Persona | Usa apps protegidas |
| Administrador de Seguridad | Persona | Único admin MVP: tenants, apps, recursos, roles, perfiles, asignaciones, políticas, auditoría |
| Aplicaciones protegidas integradas | Sistema externo | Envía solicitudes al PEP; recibe ALLOW/DENY |
| Proveedor(es) de identidad | Sistema externo | OIDC/OAuth 2.0 — valida identidad con el PDP |
| Operación y auditoría | Sistema externo | Consulta evidencia al servicio de auditoría |

### L2 — Contenedores internos

| Contenedor | Stack | Responsabilidad clave |
|---|---|---|
| PEP | Spring Boot + WebFlux + **Modulith** | Intercepta, normaliza `SolicitudAcceso`, aplica `DecisionAcceso` |
| PDP | Spring Boot + WebFlux + **Modulith** (módulos por BC) | Orquesta: único acceso a OPA / SurrealDB / IdP |
| OPA | Proceso OPA + Rego | Evalúa políticas sobre entrada normalizada |
| SurrealDB | SurrealDB | Fuente de verdad: identidades, tenants, roles, recursos, asignaciones, políticas, referencias de auditoría |
| Auditoría | Spring Boot + WebFlux + **Modulith** | Recibe `EventoAcceso`, persiste evidencia correlacionada, expone a Operación |

### L3 PEP — Componentes

| Componente | Función |
|---|---|
| Adaptador HTTP de entrada | Recibe solicitud; aplica rate-limit/throttle; extrae metadatos confiables |
| Normalizador de SolicitudAcceso | Construye `SolicitudAcceso` (identidad, recurso, acción, contexto, trazabilidad) |
| Puerto de decisión de acceso | Interfaz de salida hacia el PDP |
| Adaptador cliente PDP | Implementa el puerto con WebClient reactivo; maneja timeout/error/correlación |
| Aplicador de decisión / proxy seguro | Enforcement: ALLOW → proxy; DENY/INDETERMINATE → respuesta controlada |

### L3 PDP — Componentes (hexagonal)

| Componente | Función |
|---|---|
| Adaptador HTTP de entrada | Recibe `SolicitudAcceso`; valida contrato técnico |
| UC EvaluarAcceso | Orquesta: llama al Constructor de contexto → Puerto OPA → Puerto auditoría; emite `DecisionAcceso` |
| Constructor de contexto de autorización | Resuelve sujeto, tenant, roles, perfiles, recursos, relaciones, atributos |
| Puerto de identidad | Interfaz → Adaptador IdP (OIDC/OAuth 2.0) |
| Puerto de repositorio de seguridad | Interfaz → Adaptador SurrealDB |
| Puerto de evaluación de políticas | Interfaz → Adaptador OPA |
| Puerto de publicación de auditoría | Interfaz → Adaptador EventoAcceso → Servicio de auditoría |

## Relación con el Mapa de módulos

Spring Modulith aplica a **PEP, PDP y Auditoría** ([ADR-009](../../01-governance/adr/ADR-009-spring-modulith.md)). Detalle: [`../modulith-dependency-map.md`](../modulith-dependency-map.md).

- En el **PDP**, los módulos alinean a Bounded Contexts de negocio; el Constructor de contexto consulta APIs públicas (nunca internals).
- En el **PEP**, los módulos son capacidades de enforcement (`ingress`, `normalization`, `decision-client`, `enforcement`).
- En **Auditoría**, los módulos cubren ingesta, consulta y retención.
