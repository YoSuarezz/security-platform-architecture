# ADR-006: Modelos de autorización combinables (RBAC + ABAC + REBAC + PBAC)

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §13; ADR-004; ADR-012; app validación académica |

## Contexto

Una decisión real de acceso raramente es “solo rol”. Ejemplo académico:

- Rol **Docente** (RBAC) puede calificar.
- Solo en el **curso que imparte** (REBAC / atributo de relación).
- Solo en **horario lectivo** o desde red corporativa (ABAC de entorno).
- Expresado como **política versionada** (PBAC) evaluada por OPA.

Limitarse a un único modelo obliga a forzar el dominio o a reescribir cuando aparezca el siguiente requisito.

## Por qué se tomó la decisión

1. **El Documento Base exige los cuatro modelos** como visión del producto.
2. **Incrementalidad** — el MVP demuestra RBAC (+ deny-by-default) sin pretender ABAC/REBAC completos el día uno.
3. **OPA como punto de composición** — RBAC/ABAC/REBAC se expresan como datos de entrada + Rego, no como cuatro motores distintos.
4. **SurrealDB** soporta el salto a REBAC sin cambiar de persistencia (ADR-003).

## Decisión

Soportar **RBAC, ABAC, REBAC y PBAC** de forma **combinable**, implementados por etapas:

| Etapa | Enfoque |
|---|---|
| MVP / Etapa 1 | RBAC + PBAC (OPA) + deny-by-default (ADR-012) |
| Etapa 3 | ABAC (atributos de entorno/negocio) + REBAC mínimo (p. ej. Usuario ∈ Tenant; luego propietario/equipo) |

Ningún modelo invalida a los otros: se componen en la entrada normalizada y en las políticas.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Solo RBAC** | Suficiente para demos simples | Queda corto ante el alcance del Documento Base y el caso académico |
| **Solo ABAC** | Flexible | Pierde simplicidad operativa de roles para admin MVP |
| **Cuatro motores separados** | “Puro” | Complejidad explosiva; OPA ya compone |
| **Combinables por etapas vía OPA** | **Elegida** | Visión completa + entrega incremental |

## Consecuencias

### Positivas

- Roadmap claro sin reescritura del PDP.
- Caso académico puede crecer de roles a relaciones sin cambiar C4.
- Precedencia única (ADR-012) aplica a todos los modelos.

### Costos / riesgos

- Hay que resistir la tentación de implementar ABAC/REBAC “completos” en el MVP.
- Atributos ABAC confiables y relaciones REBAC se detallan en Etapa 3 (dirección en `01-validation-application`).

### Implicaciones

- Catálogo de roles/asignaciones primero; grafo REBAC después.
- Tests Rego deben crecer por etapa sin romper deny-by-default.
