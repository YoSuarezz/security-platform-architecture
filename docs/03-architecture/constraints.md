# Restricciones

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |

## Restricciones técnicas (confirmadas)

| ID | Restricción | Origen |
|---|---|---|
| C-01 | Java + Spring Boot | ADR-001 |
| C-02 | WebFlux obligatorio en HTTP/I/O remoto; sin bloqueantes sin aislamiento | ADR-002 |
| C-03 | SurrealDB como persistencia principal (no SQL como SoT) | ADR-003 |
| C-04 | OPA/Rego como motor de políticas | ADR-004 |
| C-05 | Dominio sin dependencia a Spring/OPA/SurrealDB/IdP | ADR-008 |
| C-06 | Contenedores desde el inicio; Cloud Enable | ADR-010 |
| C-07 | PEP no implementa reglas complejas ni accede a SurrealDB/OPA | C4 L2/L3, §7.2 |
| C-08 | Solo PDP accede a OPA y SurrealDB en el camino de decisión | C4 L2 |
| C-08b | Spring Modulith obligatorio en PEP, PDP y Auditoría | ADR-009 |

## Restricciones de alcance (MVP)

| ID | Restricción | Origen |
|---|---|---|
| C-09 | No construir IdP completo / directorio / biometría | §5.2 |
| C-10 | No API Gateway genérico | §5.2 |
| C-11 | No sincronizar negocio completo de apps | §5.2 |
| C-12 | No exigir HashBlockchain | §5.2 / §11.8 |
| C-13 | No exigir ABAC/REBAC completos antes del camino RBAC | §6, §19 |
| C-14 | Frontend admin no bloquea MVP | §6 pendiente |

## Restricciones de proceso

| ID | Restricción | Origen |
|---|---|---|
| C-15 | Línea base (Git, CI, pruebas, módulos) antes de historias funcionales | ADR-011, §8.12 |
| C-16 | Cobertura >80% como umbral de diseño | ADR-011 |
| C-17 | Decisiones confirmadas se registran como ADR | Fase 0.A |
