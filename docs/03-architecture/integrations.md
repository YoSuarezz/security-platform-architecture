# Integraciones

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| Diagramas | [L1](c4/level1-context.png), [L2](c4/level2-containers.png) |
| IdP MVP | Keycloak — [ADR-013](../01-governance/adr/ADR-013-keycloak-idp.md) |
| HTTP mapping | [ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md) |

## Sistemas externos

| Sistema | Dirección | Protocolo / contrato | Dato mínimo | Contenedor que integra |
|---|---|---|---|---|
| Aplicaciones protegidas | Inbound → PEP; outbound enforce | HTTPS | Request HTTP + evidencia de identidad; respuesta ALLOW/proxy o 401/403/503 | PEP |
| Keycloak (IdP MVP) | Outbound desde PDP | OIDC / OAuth 2.0 | Evidencia validable (no token crudo al dominio) | PDP (adaptador identidad) |
| OPA | Outbound desde PDP | HTTP API OPA | Entrada normalizada / decisión estructurada | PDP (adaptador OPA) |
| SurrealDB | Outbound desde PDP | SurrealQL / driver | Contexto de seguridad (tenant, roles, …) | PDP (adaptador SurrealDB) |
| Operación y auditoría (consumidores) | Inbound consulta | API de auditoría (por definir en 05-contracts) | Evidencia correlacionada | Servicio de auditoría |
| Administrador de Seguridad | Inbound admin | API admin (Etapa 2) | Configuración de catálogos/políticas | PDP / futuros adaptadores admin |

## Modos de integración PEP (Documento Base §14.3)

| Modo | MVP | Notas |
|---|---|---|
| Proxy/gateway PEP | **Sí (objetivo MVP)** | Ya hay evidencia conceptual en demo-security |
| Filtro/middleware en app | Posterior | Requiere SDK por lenguaje |
| Sidecar | Posterior | Complejidad operativa |
| Llamada directa al PDP | Posible interno | La app debe enforce; no sustituye PEP |

**Decisión de diseño (C4):** el contrato hacia el PDP (`SolicitudAcceso` / `DecisionAcceso`) se mantiene estable para permitir otros modos sin reescribir la decisión.

## Contratos lógicos (detalle en 05-contracts)

| Contrato | Productor | Consumidor |
|---|---|---|
| `SolicitudAcceso` | PEP (normalizador) | PDP |
| `DecisionAcceso` | PDP | PEP (aplicador) |
| Entrada OPA normalizada | PDP | OPA |
| Evento de acceso / auditoría | PDP | Servicio de auditoría |

## Mapeo HTTP (ADR-014)

| Causa | HTTP |
|---|---|
| ALLOW | Proxy / continuación |
| DENY (política / sin política) | 403 |
| Token inválido | 401 |
| INDETERMINATE (OPA/SurrealDB/IdP caído) | 503 |

## Pendiente

- OpenAPI concreto (Fase 0.E / diseño detallado).
- Matriz de timeouts/reintentos (números) — PD-11 / NFR.
