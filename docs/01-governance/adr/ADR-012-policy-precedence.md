# ADR-012: Precedencia de políticas y semántica de decisión

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §13.5, §21.4; PD-02; INV-POL-03 |

## Contexto

El Documento Base marcaba como provisional la regla de precedencia ALLOW/DENY/ausencia. Sin una decisión formal Accepted no se pueden escribir políticas Rego ni tests de evaluación sin ambigüedad.

## Decisión

Se confirma formalmente:

> **Deny-by-default + una denegación explícita tiene prioridad sobre cualquier autorización.**

### Orden de evaluación

1. **Deny explícito** — si alguna política aplicable deniega → `DENY`.
2. **Allow explícito** — si alguna política aplicable autoriza y no hay deny → `ALLOW`.
3. **Ninguna política aplica** → `DENY` por defecto.

### Relación con errores de infraestructura

Los fallos de OPA, SurrealDB o IdP **no** se interpretan como “ninguna política aplica”. Se modelan como `INDETERMINATE` (ver ADR-014).

## Alternativas consideradas

1. **Primera política que aplica gana** — rechazada: permite que un allow anterior oculte un deny crítico.
2. **Allow-by-default** — rechazada: contradice el principio de denegar por defecto del Documento Base §7.1.
3. **Deny-by-default + deny explícito gana** — **elegida**.

## Consecuencias

### Positivas

- Semántica única para Rego, tests y UC-01.
- INV-POL-03 deja de ser provisional.
- Reduce riesgo de autorización accidental por omisión de política.

### Negativas / costos

- Políticas deben ser explícitas; no hay “permiso implícito por silencio”.
- Composición de múltiples políticas exige disciplina en el orden conceptual (deny siempre gana).

### Implicaciones

- Toda política Rego del MVP debe respetar este orden.
- Casos de prueba mínimos: deny explícito vs allow, ausencia de política, allow único.
