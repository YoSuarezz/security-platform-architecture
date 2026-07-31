# Decisiones pendientes

| Campo | Valor |
|---|---|
| Estado | Living document |
| Fuentes | Documento Base §6, §13.5, §21; avance Fase 0 |
| Última actualización | 2026-07-31 |

## Cerradas (2026-07-31)

| ID | Decisión | Artefacto |
|---|---|---|
| PD-01 | App de validación: **Sistema de Gestión Académica Universitaria** | [`01-validation-application.md`](../02-domain/01-validation-application.md) |
| PD-02 | Deny-by-default + deny explícito gana | [ADR-012](../01-governance/adr/ADR-012-policy-precedence.md) |
| PD-03 | IdP MVP: **Keycloak self-hosted** | [ADR-013](../01-governance/adr/ADR-013-keycloak-idp.md) |
| PD-04 | Matriz 401 / 403 / 503; sin caché como verdad | [ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md) |
| PD-06 | Un solo **Administrador de Seguridad** en el MVP | [`01-validation-application.md`](../02-domain/01-validation-application.md) |
| PD-07 | Dirección ABAC (detalle Etapa 3) | [`01-validation-application.md`](../02-domain/01-validation-application.md) |
| PD-08 | REBAC inicial: Usuario pertenece a Tenant (+ futuras) | [`01-validation-application.md`](../02-domain/01-validation-application.md) |

## Bloqueantes restantes / trabajo pendiente

| ID | Pregunta | Fase | Estado | Nota |
|---|---|---|---|---|
| PD-05 | PEP solo proxy o también SDK/filtros (§21.10) | 0.C / 0.E | Parcial | MVP = proxy; interfaz PDP debe permitir otros modos |
| — | PoC técnica WebFlux | Etapa 0 | **Pendiente de ejecutar** | Criterio de cierre acordado; falta implementar y documentar resultado |
| — | Ratificación formal Event Storming en sesión | 0.B | **Pendiente de sesión** | Documento listo; falta revisión contra Big Picture / BC / Context Map |

## Importantes pero no bloquean el esqueleto RBAC

| ID | Pregunta | Fase | Estado |
|---|---|---|---|
| PD-09 | Identificación estable app/recurso/acción entre tecnologías (§21.7) | 0.D / 0.E | Principio: IDs de plataforma (códigos en validation-application) |
| PD-10 | Retención de auditoría (§21.8) | 0.F | Abierta |
| PD-11 | Latencia/volumen/disponibilidad exigibles (§21.11) | 0.C / NFR | Abierta |
| PD-12 | Tecnología frontend admin (§21.12) | Etapa 4 | Pendiente (decisión de producto) |
| PD-13 | Entorno productivo final (cloud/on-prem/híbrido) | Post-MVP | Cloud Enable sí; destino no |
| PD-14 | Herencia de roles / denegaciones explícitas entre asignaciones (§11.4) | Pre-prod | Abierta (RB-14) |
| PD-15 | `HashBlockchain` | Investigación | No requisito |

## Cerradas en gobierno (ADR)

Ver [`docs/01-governance/adr/README.md`](../01-governance/adr/README.md) — ADR-001…ADR-014 Accepted.
