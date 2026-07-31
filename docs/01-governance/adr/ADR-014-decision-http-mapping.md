# ADR-014: Mapeo HTTP de DecisionAcceso e INDETERMINATE

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §14.2, §21.9; PD-04; ADR-012; UC-01 |

## Contexto

El PEP habla HTTP con clientes y apps. Internamente el PDP emite `DecisionAcceso` en tres estados. Sin una matriz HTTP estable:

- operaciones no distingue “política negó” de “OPA cayó”;
- clientes reintentan mal (reintentar un 403 de política no tiene sentido; reintentar un 503 sí);
- se filtra detalle interno (stack traces, mensajes Rego) o se oculta demasiado.

## Por qué se tomó la decisión

1. **Semántica HTTP estándar** — 401 autenticación, 403 autorización, 503 dependencia caída.
2. **Operabilidad** — alertas y SLOs distintos para 503 vs 403.
3. **Seguridad** — fallo de infra nunca se traduce a ALLOW; no hay caché como verdad.
4. **Coherencia con deny-by-default** — “sin política” es DENY→403, no un 404 ambiguo ni un 200 vacío.

## Decisión

| Causa | DecisionAcceso interna | HTTP al cliente |
|---|---|---|
| Política aplica y deniega | `DENY` | **403 Forbidden** |
| Ninguna política aplicable | `DENY` (ADR-012) | **403 Forbidden** |
| Token inválido / ausente / no validable | `DENY` (`TOKEN_INVALID`) | **401 Unauthorized** |
| Error de infra (OPA, SurrealDB o IdP no disponible / error no confiable) | `INDETERMINATE` | **503 Service Unavailable** |
| Política aplica y autoriza | `ALLOW` | Proxy / continuación según modo PEP |

### Reglas adicionales

1. **No se usa caché como fuente de verdad** de autorización.
2. `INDETERMINATE` **nunca** → ALLOW.
3. El cuerpo al cliente no expone internals; el detalle técnico completo va a auditoría (ADR-007).

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Siempre 403** | Simple | Confunde outage con deny; mala operación |
| **Siempre 500 en errores** | Genérico | Menos preciso que 503 para dependencias |
| **Degradación con caché de última decisión** | Alta disponibilidad aparente | Ventana de autorización obsoleta; viola deny-by-default |
| **Fail-open (ALLOW si OPA cae)** | Disponibilidad máxima | Inaceptable en plataforma de seguridad |
| **Matriz 401 / 403 / 503** | **Elegida** | Clara para clientes y ops |

## Consecuencias

### Positivas

- UC-01 y QA-03 cerrados.
- Clientes pueden reintentar solo 503.
- Auditoría conserva la causa técnica distinguible.

### Costos / riesgos

- El PDP/PEP deben clasificar errores con cuidado (timeout OPA ≠ POLICY_DENY).
- Hay que documentar timeouts/reintentos numéricos (PD-11 / NFR).

### Implicaciones

- Sequence diagrams en 05-Contracts reflejan esta matriz.
- Monitoreo: tasa de 503 = salud de dependencias; tasa de 403 = políticas/asignaciones.
