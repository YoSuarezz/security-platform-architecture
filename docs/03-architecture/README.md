# 03. Architecture

Diseño de alto nivel del sistema — Plataforma Central de Seguridad.

| Campo | Valor |
|---|---|
| Fase | **0.C** |
| Estado | **Accepted (v1.0)** |

## Orden de lectura recomendado

| # | Artefacto | Estado |
|---|---|---|
| 1 | [architecture-overview.md](architecture-overview.md) | Accepted |
| 2 | [architectural-drivers.md](architectural-drivers.md) | Accepted |
| 3 | [quality-attributes.md](quality-attributes.md) | Accepted |
| 4 | [constraints.md](constraints.md) | Accepted |
| 5 | [technology-stack.md](technology-stack.md) | Accepted |
| 6 | [integrations.md](integrations.md) | Accepted |
| 7 | [c4/README.md](c4/README.md) → PNG L1–L3 | Accepted |
| 8 | [modulith-dependency-map.md](modulith-dependency-map.md) | Accepted |
| 9 | [deployment/deployment-mvp.md](deployment/deployment-mvp.md) | Accepted |

## Diagramas C4 — autoridad visual

| Nivel | Archivo |
|---|---|
| L1 Context | [c4/level1-context.png](c4/level1-context.png) |
| L2 Containers | [c4/level2-containers.png](c4/level2-containers.png) |
| L3 PEP | [c4/level3-component-pep.png](c4/level3-component-pep.png) |
| L3 PDP | [c4/level3-component-pdp.png](c4/level3-component-pdp.png) |

> L4 (código) queda fuera de Fase 0.

## Decisiones fijadas

| Decisión | Dónde |
|---|---|
| PEP y PDP separados (WebFlux) | C4 L2, ADR-002 |
| Solo PDP habla con OPA / SurrealDB / Keycloak | C4 L2, ADR-013 |
| 11 módulos Modulith + commons | modulith-dependency-map.md |
| Deployment: solo PEP expuesto | deployment-mvp.md |
| HTTP 401/403/503 | ADR-014 |

## Entrada desde dominio

Revisar primero [`../02-domain/README.md`](../02-domain/README.md) (orden `01`→`12`).
