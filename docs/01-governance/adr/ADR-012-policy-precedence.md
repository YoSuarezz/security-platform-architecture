# ADR-012: Precedencia de políticas y semántica de decisión

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §7.1, §13.5, §21.4; PD-02; INV-POL-03; ADR-004, ADR-014 |

## Contexto

Cuando varias políticas pueden aplicar a la misma `SolicitudAcceso`, o cuando **ninguna** aplica, el motor debe comportarse de forma idéntica en Rego, en tests y en UC-01. El Documento Base dejaba la precedencia como **provisional**. Sin ADR Accepted:

- dos desarrolladores escriben Rego incompatible;
- un allow “silencioso” puede tapar un deny crítico;
- no hay criterio objetivo de prueba.

## Por qué se tomó la decisión

1. **Principio de denegar por defecto (§7.1)** — la ausencia de permiso explícito no es autorización.
2. **Seguridad > conveniencia** — es preferible un falso negativo operativo (hay que publicar allow) a un falso positivo (acceso indebido).
3. **Composición predecible** — “deny gana” es la semántica más segura cuando coexisten políticas de distintos autores/tenant/app.
4. **Separar “sin política” de “error de infra”** — lo primero es DENY; lo segundo es INDETERMINATE (ADR-014), no se confunden.

## Decisión

> **Deny-by-default + una denegación explícita tiene prioridad sobre cualquier autorización.**

### Orden de evaluación

1. **Deny explícito** — si alguna política aplicable deniega → `DENY`.
2. **Allow explícito** — si alguna política aplicable autoriza y no hay deny → `ALLOW`.
3. **Ninguna política aplica** → `DENY` por defecto (`NO_APPLICABLE_POLICY` o equivalente).

Los fallos de OPA / SurrealDB / IdP **no** son “ninguna política aplica”; son `INDETERMINATE` (ADR-014).

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Allow-by-default** | Cómodo para demos | Contradice §7.1; riesgo inaceptable |
| **Primera política que aplica gana** | Simple de implementar | Un allow temprano oculta un deny posterior |
| **Allow gana sobre deny** | “Productivo” | Permite bypass por política permisiva mal publicada |
| **Deny-by-default + deny explícito gana** | **Elegida** | Máxima seguridad predicible |

## Consecuencias

### Positivas

- Semántica única para Rego, tests y UC-01.
- INV-POL-03 deja de ser provisional.
- Obliga a políticas explícitas (buena higiene).

### Costos / riesgos

- Más trabajo de publicación de allows legítimos.
- Operadores deben entender que el silencio = deny.

### Implicaciones

- Casos de prueba mínimos obligatorios: deny vs allow, solo allow, ninguna política, error infra ≠ deny-by-default.
- Documentar en contratos Rego (05-Contracts).
