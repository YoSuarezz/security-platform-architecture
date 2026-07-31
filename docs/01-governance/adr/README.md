# Architecture Decision Records (ADR)

Registro formal de decisiones arquitectónicas.

| Campo | Valor |
|---|---|
| Estado | Accepted — ADR-001…014 profundizados (contexto, por qué, alternativas, consecuencias) |
| Nota | Cada ADR documenta fundamentación completa; ADR-009 aplica Modulith a PEP, PDP y Auditoría |

## Índice

| ID | Título | Estado |
|---|---|---|
| [ADR-001](ADR-001-java-spring-boot.md) | Java y Spring Boot como base | Accepted |
| [ADR-002](ADR-002-spring-webflux.md) | Spring WebFlux para HTTP / I/O remoto | Accepted |
| [ADR-003](ADR-003-surrealdb.md) | SurrealDB como persistencia principal | Accepted |
| [ADR-004](ADR-004-opa-rego.md) | OPA y Rego como motor de políticas | Accepted |
| [ADR-005](ADR-005-delegated-authentication.md) | Autenticación delegada y abstraída | Accepted |
| [ADR-006](ADR-006-rbac-abac-rebac-pbac.md) | RBAC + ABAC + REBAC + PBAC combinables | Accepted |
| [ADR-007](ADR-007-mandatory-audit.md) | Auditoría obligatoria | Accepted |
| [ADR-008](ADR-008-hexagonal-clean-architecture.md) | Clean Architecture / Arquitectura Hexagonal | Accepted |
| [ADR-009](ADR-009-spring-modulith.md) | Spring Modulith en PEP, PDP y Auditoría | Accepted |
| [ADR-010](ADR-010-cloud-native-12factor.md) | Cloud Native / Cloud Enable / 12-Factor | Accepted |
| [ADR-011](ADR-011-engineering-baseline.md) | Línea base de ingeniería (Git, CI, tests, Clean Code) | Accepted |
| [ADR-012](ADR-012-policy-precedence.md) | Precedencia deny-by-default + deny explícito gana | Accepted |
| [ADR-013](ADR-013-keycloak-idp.md) | Keycloak self-hosted como IdP del MVP | Accepted |
| [ADR-014](ADR-014-decision-http-mapping.md) | Mapeo HTTP 401/403/503 e INDETERMINATE | Accepted |

## Cómo agregar un ADR

1. Copiar [`template.md`](template.md).
2. Numerar secuencialmente (`ADR-015-…`).
3. Actualizar este índice.
4. Si la decisión afecta el Documento Base, actualizar también `docs/00-source/Documento_Base.md` (bitácora §25).
