# ADR-014: Mapeo HTTP de DecisionAcceso e INDETERMINATE

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §14.2, §21.9; PD-04; ADR-012 |

## Contexto

El PEP debe traducir `DecisionAcceso` a respuestas HTTP sin filtrar detalle interno. Faltaba la matriz operativa 401/403/503.

## Decisión

| Causa | DecisionAcceso interna | Respuesta HTTP al cliente |
|---|---|---|
| Política aplica y deniega | `DENY` | **403 Forbidden** |
| Ninguna política aplicable | `DENY` (deny-by-default, ADR-012) | **403 Forbidden** |
| Token inválido / ausente / no validable | `DENY` (`TOKEN_INVALID` o equivalente) | **401 Unauthorized** |
| Error de infraestructura (OPA, SurrealDB o IdP no disponible / error no confiable) | `INDETERMINATE` | **503 Service Unavailable** |
| Política aplica y autoriza | `ALLOW` | Proxy / continuación según modo PEP |

### Reglas adicionales

1. **No se usa caché como fuente de verdad** para autorización.
2. `INDETERMINATE` **nunca** se traduce a `ALLOW`.
3. El cuerpo de respuesta al cliente no expone stack traces, mensajes de OPA ni detalles internos; la causa técnica completa vive en auditoría.

## Alternativas consideradas

1. **Siempre 403 ante cualquier fallo** — rechazada: confunde caídas de infra con denegación de política y dificulta operación.
2. **Degradación con caché de última decisión** — rechazada: contradice denegar por defecto y crea ventana de autorización obsoleta.
3. **Matriz 401 / 403 / 503 según causa** — **elegida**.

## Consecuencias

### Positivas

- Operación puede distinguir fallos de infra (503) de denegaciones (403) y de auth (401).
- Criterios de aceptación de UC-01 y QA-03 quedan cerrados.

### Implicaciones

- Adaptadores PEP/PDP deben clasificar errores antes de emitir `DecisionAcceso`.
- Sequence diagrams de 05-Contracts deben reflejar esta matriz.
