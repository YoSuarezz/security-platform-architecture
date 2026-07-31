# Atributos de calidad

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| Fuente | Documento Base §17 |

## Atributos priorizados

| Atributo | Requisito (resumen) | Cómo se atiende en arquitectura |
|---|---|---|
| Seguridad | Validación de identidad, aislamiento tenant, secretos fuera de código, anti header-spoofing | PEP valida contexto técnico; PDP valida identidad/alcance; deny-by-default |
| Trazabilidad | Correlación E2E, evidencia de decisión | IDs en Solicitud/Decisión; servicio de auditoría |
| Mantenibilidad | Hexagonal, Modulith, contratos versionados | C4 L3 PDP; ADRs; contratos en 05 |
| Evolutividad | Cambiar IdP/OPA/DB sin reescribir dominio | Puertos/adaptadores |
| Performance | Latencia de autorización acotada y medible | WebFlux; timeouts; sin caché como verdad |
| Disponibilidad / resiliencia | Comportamiento definido si cae OPA/SurrealDB/IdP | `INDETERMINATE` → **503** ([ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md)) |
| Testabilidad | Cobertura >80%; invariantes unitarias; adaptadores en integración | ADR-011; pirámide en 07 |
| Portabilidad / Cloud Enable | Contenedores; sin host fijo | Deployment (pendiente diagrama) |
| Observabilidad | Logs estructurados, métricas de decisión, trazas | Convención en 07; implementación Etapa 2 |

## Escenarios de calidad (mínimos)

| ID | Escenario | Respuesta esperada |
|---|---|---|
| QA-01 | Token inválido | `DENY` + auditoría; app no alcanzada |
| QA-02 | Política niega | `DENY` + `POLICY_DENY` |
| QA-03 | OPA caído | `INDETERMINATE` → **503** + auditoría con fallo técnico (ADR-014) |
| QA-04 | Intento de privilegio cross-tenant | `DENY` / `TENANT_MISMATCH` |
| QA-05 | Cambio de versión de política | Decisiones posteriores referencian nueva versión en evidencia |
