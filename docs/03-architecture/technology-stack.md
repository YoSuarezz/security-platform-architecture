# Technology stack

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| ADRs | ADR-001…ADR-014 |
| Diagrama | [level2-containers.png](c4/level2-containers.png) |

## Confirmado

| Capa | Tecnología | ADR / origen |
|---|---|---|
| Lenguaje | Java | ADR-001 |
| Framework servicios | Spring Boot | ADR-001 |
| HTTP / I/O remoto | Spring WebFlux | ADR-002 |
| Persistencia | SurrealDB (SurrealQL) | ADR-003 |
| Motor de políticas | OPA + Rego | ADR-004 |
| Autenticación | Delegada (OIDC / OAuth 2.0) | ADR-005 |
| Organización interna | Spring Modulith en **PEP, PDP y Auditoría** | ADR-009 |
| Estilo | Clean Architecture / Hexagonal | ADR-008 |
| Construcción | Cloud Native, Cloud Enable, 12-Factor, Clean Code, CI, tests >80% | ADR-010, ADR-011 |

## Contenedores × stack

| Contenedor | Stack |
|---|---|
| PEP | Java, Spring Boot, WebFlux, **Modulith**, cliente HTTP reactivo al PDP |
| PDP | Java, Spring Boot, WebFlux, **Modulith**, adaptadores IdP / SurrealDB / OPA / Auditoría |
| OPA | Binario/imagen OPA, políticas Rego |
| SurrealDB | Servidor SurrealDB |
| Auditoría | Java, Spring Boot, WebFlux, **Modulith** |
| Keycloak | IdP OIDC/OAuth2 (MVP) |

## IdP del MVP (cerrado)

| Componente | Tecnología |
|---|---|
| IdP | **Keycloak** (self-hosted, Docker) — [ADR-013](../01-governance/adr/ADR-013-keycloak-idp.md) |
| Protocolos | OpenID Connect + OAuth 2.0 |

## Pendiente (no inventar)

| Tema | Estado |
|---|---|
| Frontend administrativo | PD-12 / Etapa 4 |
| Entorno productivo final (K8s/cloud/on-prem) | Cloud Enable sí; destino no (PD-13) |
| Bus de mensajería para auditoría | No decidido; L2 muestra publicación de eventos PDP→Auditoría |

## Explícitamente no es el stack objetivo

| Evitar | Motivo |
|---|---|
| Spring MVC bloqueante en PEP/PDP | ADR-002; demo actual no es base |
| SQL como fuente de verdad | ADR-003 |
| Reglas de autorización en el interceptor PEP | Principio §7.2 / C4 L3 PEP |
