# Visión arquitectónica

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| Diagramas | [`c4/`](c4/) |

## Resumen

La **Plataforma Central de Seguridad** concentra identidad normalizada, autorización por políticas, control de acceso y trazabilidad auditable para aplicaciones integradas heterogéneas. Soporta RBAC, ABAC, REBAC y PBAC de forma incremental.

```text
Usuario final → Aplicaciones protegidas → PEP → PDP → OPA / SurrealDB / IdP
                                         ↓         ↓
                                    Enforcement   Auditoría
```

## Niveles C4

| Nivel | Qué responde | Diagrama |
|---|---|---|
| L1 | ¿Quién usa el sistema y con qué sistemas habla? | [level1-context.png](c4/level1-context.png) |
| L2 | ¿Qué contenedores lo componen? | [level2-containers.png](c4/level2-containers.png) |
| L3 PEP | ¿Cómo enforce sin políticas? | [level3-component-pep.png](c4/level3-component-pep.png) |
| L3 PDP | ¿Cómo decide en arquitectura hexagonal? | [level3-component-pdp.png](c4/level3-component-pdp.png) |

## Contenedores internos (L2)

| Contenedor | Tecnología | Responsabilidad |
|---|---|---|
| PEP | Spring Boot + WebFlux | Interceptar, normalizar `SolicitudAcceso`, aplicar `DecisionAcceso` |
| PDP | Spring Boot + WebFlux | Orquestar evaluación, único acceso a OPA/SurrealDB/IdP en el camino de decisión |
| OPA | Proceso/contenedor OPA | Evaluar Rego sobre entrada normalizada |
| SurrealDB | SurrealDB | Fuente de verdad del dominio de seguridad |
| Servicio de auditoría | Spring Boot + WebFlux | Proteger, correlacionar y exponer evidencia |

## Estilo arquitectónico

- **Hexagonal / Clean Architecture** en el PDP (puertos de identidad, repositorio, políticas, auditoría).
- **Separación PEP/PDP** (decisión ≠ aplicación).
- **Reactivo** (WebFlux) en PEP, PDP y Auditoría.
- **Cloud Native / Cloud Enable / 12-Factor** (ADR-010).
- **Modulith** en todos los contenedores Java (PEP, PDP, Auditoría) — ver [`modulith-dependency-map.md`](modulith-dependency-map.md) y ADR-009.

## Fuera de este overview

- Deployment detallado → `deployment/`
- Stack y drivers → `technology-stack.md`, `architectural-drivers.md`
- Contratos → `docs/05-contracts/`
