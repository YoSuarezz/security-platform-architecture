# Architecture Decision Records (ADR)

Registro formal de decisiones arquitectónicas.

## Índice

| ID | Título | Estado |
|---|---|---|
| [ADR-001](ADR-001-java-spring-boot.md) | Java y Spring Boot | Accepted |
| [ADR-002](ADR-002-spring-webflux.md) | Spring WebFlux | Accepted |
| [ADR-003](ADR-003-surrealdb.md) | SurrealDB | Accepted |
| [ADR-004](ADR-004-opa-rego.md) | OPA y Rego | Accepted |
| [ADR-005](ADR-005-delegated-authentication.md) | Autenticación delegada | Accepted |
| [ADR-006](ADR-006-rbac-abac-rebac-pbac.md) | RBAC + ABAC + REBAC + PBAC | Accepted |
| [ADR-007](ADR-007-mandatory-audit.md) | Auditoría obligatoria | Accepted |
| [ADR-008](ADR-008-hexagonal-clean-architecture.md) | Clean Architecture / Hexagonal | Accepted |
| [ADR-009](ADR-009-spring-modulith.md) | Spring Modulith | Accepted |
| [ADR-010](ADR-010-cloud-native-12factor.md) | Cloud Native / 12-Factor | Accepted |
| [ADR-011](ADR-011-engineering-baseline.md) | Línea base de ingeniería | Accepted |

## Cómo agregar un ADR

1. Copiar [`template.md`](template.md) o [`../../templates/adr-template.md`](../../templates/adr-template.md).
2. Numerar secuencialmente (`ADR-012-...`).
3. Actualizar este índice.
4. Si cambia una decisión del Documento Base, actualizar también `docs/00-source/` mediante un cambio controlado.
