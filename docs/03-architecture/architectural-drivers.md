# Drivers arquitectónicos

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |

## Drivers principales

| ID | Driver | Implicación arquitectónica | Evidencia |
|---|---|---|---|
| D-01 | Autorización centralizada multi-app / multi-tenant | PEP/PDP + tenant explícito en toda decisión | C4 L1/L2, INV-TEN-* |
| D-02 | Apps heterogéneas (Java, .NET, Python, PHP, …) | PEP como borde común; contrato estable al PDP | C4 L1, modo proxy MVP |
| D-03 | Políticas evolucionables fuera del código de apps | OPA/Rego; BC Políticas ≠ lógica de negocio | ADR-004, C4 L2 |
| D-04 | Decisiones explicables y auditables | Contenedor de auditoría + correlación | ADR-007, C4 L2 |
| D-05 | No acoplar dominio a frameworks/IdP/DB | Hexagonal en PDP (puertos) | ADR-008, C4 L3 PDP |
| D-06 | Latencia acotada bajo concurrencia | WebFlux; sin I/O bloqueante en el camino | ADR-002 |
| D-07 | Preparación cloud sin destino final | Contenedores, config externa, sin estado local | ADR-010 |
| D-08 | Módulos por capacidad de negocio | Spring Modulith alineado a BC | ADR-009, dominio 0.B |
| D-09 | Denegar por defecto / defensa en profundidad | Tri-estado; PEP enforce; PEP ≠ políticas | Principios §7, C4 L3 PEP |
| D-10 | Autenticación delegada | Adaptador IdP; no construir IdP | ADR-005 |

## Drivers descartados / fuera de alcance MVP

| No-driver | Motivo |
|---|---|
| Ser API Gateway genérico | Fuera de alcance §5.2 |
| Reemplazar IdP empresarial | Fuera de alcance |
| Blockchain de auditoría | Investigación, no requisito |
| Frontend admin como prerequisito del MVP | Pendiente; MVP por API |
