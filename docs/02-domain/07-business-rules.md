# Reglas de negocio

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Relacionado | [`06-invariants.md`](06-invariants.md), [`08-use-cases.md`](08-use-cases.md) |

## Relación con invariantes

- **Invariante**: condición que el modelo **nunca** debe violar (se prueba siempre).
- **Regla de negocio**: comportamiento esperado en un escenario (puede tener excepciones documentadas).

Las reglas siguientes derivan del Documento Base; no introducen alcance fuera del MVP salvo que se marque como *futuro*.

---

## RB-01 — Separación decisión / aplicación

| Campo | Valor |
|---|---|
| Enunciado | Toda solicitud protegida se evalúa en el PDP y se aplica en el PEP. |
| Invariantes | INV-SEC-02 |
| Aplica a | UC-01 EvaluarAcceso |
| Excepciones | Ninguna en el camino crítico |

## RB-02 — Denegar por defecto

| Campo | Valor |
|---|---|
| Enunciado | Si no hay política aplicable, la identidad es inválida, el contexto está incompleto o hay error no confiable, el resultado no puede ser `ALLOW`. |
| Invariantes | INV-SEC-01, INV-POL-03 |
| Aplica a | UC-01 |
| Nota | Precedencia formal pendiente de ADR en 0.E |

## RB-03 — Tri-estado de decisión

| Campo | Valor |
|---|---|
| Enunciado | La decisión interna es `ALLOW`, `DENY` o `INDETERMINATE`. El PEP mapea: token inválido→401, DENY→403, INDETERMINATE→503 ([ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md)). Sin filtrar detalle técnico sensible. |
| Invariantes | INV-POL-04, INV-AUD-03 |
| Aplica a | UC-01 |

## RB-04 — Aislamiento por tenant

| Campo | Valor |
|---|---|
| Enunciado | No se conceden privilegios cruzando tenants sin relación explícita autorizada. Toda evaluación exige tenant válido. |
| Invariantes | INV-TEN-01, INV-TEN-02, INV-TEN-03 |
| Aplica a | UC-01, UC-03 AsignarRol |

## RB-05 — Solo asignaciones vigentes

| Campo | Valor |
|---|---|
| Enunciado | Roles/perfiles considerados en la evaluación son únicamente los de asignaciones en estado `VIGENTE`. |
| Invariantes | INV-ASN-01 |
| Aplica a | UC-01, UC-03 |

## RB-06 — Autenticación delegada

| Campo | Valor |
|---|---|
| Enunciado | La plataforma valida y normaliza evidencia del IdP; no es el proveedor de identidad ni sustituye un IdP empresarial. |
| Invariantes | INV-ID-01, INV-ID-02 |
| Aplica a | UC-01 |
| ADR | ADR-005 |

## RB-07 — Políticas versionadas

| Campo | Valor |
|---|---|
| Enunciado | Solo políticas publicadas con versión, responsable y vigencia participan en evaluación productiva. |
| Invariantes | INV-POL-01 |
| Aplica a | UC-04 PublicarPolitica, UC-01 |

## RB-08 — OPA evalúa contexto explícito

| Campo | Valor |
|---|---|
| Enunciado | OPA no navega el dominio ni recibe tokens crudos; evalúa la entrada que construye el PDP. |
| Invariantes | INV-POL-02, INV-SEC-06 |
| Aplica a | UC-01 |
| ADR | ADR-004 |

## RB-09 — Auditoría no silenciosa

| Campo | Valor |
|---|---|
| Enunciado | Cada evaluación deja evidencia correlacionada sin secretos. |
| Invariantes | INV-SEC-05, INV-AUD-01, INV-ID-03 |
| Aplica a | UC-01 |

## RB-10 — La plataforma no ejecuta negocio de la app

| Campo | Valor |
|---|---|
| Enunciado | Puede decidir si un sujeto puede `actualizar` una factura; no calcula la factura ni ejecuta el caso de negocio. |
| Origen | §3.3 |
| Aplica a | Todos los UC de runtime |

## RB-11 — Datos mínimos desde apps integradas

| Campo | Valor |
|---|---|
| Enunciado | Las apps aportan solo atributos/relaciones necesarios al contexto, bajo contrato; no se sincroniza el negocio completo. |
| Origen | §3.3, §5.2 |
| Aplica a | UC-01 (futuro ABAC/REBAC) |

## RB-12 — Registro previo de aplicación y recurso

| Campo | Valor |
|---|---|
| Enunciado | No se evalúa acceso productivo a una aplicación/recurso no registrados (o se deniega por defecto). |
| Aplica a | UC-02 RegistrarAplicacion, UC-01 |
| MVP | Una app + un recurso bastan para el esqueleto |

## RB-13 — RBAC primero; ABAC/REBAC por incremento

| Campo | Valor |
|---|---|
| Enunciado | El camino crítico valida RBAC simple vía Rego. ABAC y REBAC se añaden en etapas posteriores sin romper el contrato de decisión. |
| Origen | §6, §13, §19 |
| ADR | ADR-006 |

## RB-14 — Herencia de roles y denegaciones explícitas (pendiente)

| Campo | Valor |
|---|---|
| Enunciado | Si existe herencia de roles o denegaciones explícitas entre asignaciones, su precedencia debe definirse antes de producción. |
| Origen | §11.4, §13.5 |
| Estado | **Pendiente** — no asumir herencia en MVP |

## Criterio de cierre

- [x] Reglas del camino crítico documentadas y enlazadas a invariantes/UC.
- [x] RB-02/INV-POL-03 cerradas por ADR-012.
- [ ] RB-14 decidida antes de Etapa 2/3 según necesidad (PD-14 — no bloquea 02).

---

## Navegación

[**← 06 Invariantes**](06-invariants.md) · [**08 Casos de uso →**](08-use-cases.md)
