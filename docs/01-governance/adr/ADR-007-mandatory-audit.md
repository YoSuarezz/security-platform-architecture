# ADR-007: Auditoría obligatoria de decisiones

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §16; ADR-014; BC Auditoría |

## Contexto

Una plataforma de seguridad que no puede explicar *por qué* concedió o negó un acceso:

- no sirve para soporte ni compliance;
- no permite depurar políticas;
- no genera confianza institucional.

La auditoría no es un “log opcional”: es parte del producto. Toda evaluación —ALLOW, DENY o INDETERMINATE— debe dejar evidencia correlacionable.

## Por qué se tomó la decisión

1. **Exigencia del Documento Base** — auditoría obligatoria, no best-effort.
2. **Correlación extremo a extremo** — `requestId` / `correlationId` / `decisionId` unen PEP → PDP → OPA → Auditoría.
3. **Privacidad** — auditar la decisión, no secretos: sin tokens completos, contraseñas ni PII innecesaria.
4. **Contenedor separado** — el servicio de Auditoría no decide ni evalúa Rego; reduce acoplamiento y permite escalar/retener distinto.

## Decisión

La **auditoría es obligatoria** para toda evaluación de acceso y para operaciones administrativas sensibles (alta de app, publicación de política, asignación de rol, etc.).

Requisitos mínimos de evidencia:

- Identificadores de correlación.
- Resultado (`ALLOW` / `DENY` / `INDETERMINATE`) y `reasonCode`.
- Referencias de política/versión cuando aplique.
- Sujeto, tenant, app, recurso, acción (en forma no excesivamente sensible).
- Marca temporal.

Prohibido persistir: tokens completos, secretos, passwords, payloads crudos innecesarios.

`HashBlockchain` **no** es requisito de producto (investigación opcional).

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Solo logs de aplicación** | Barato | No estructurado; difícil de consultar; fácil de omitir |
| **Auditoría best-effort (si falla, igual ALLOW)** | “Disponibilidad” | Inaceptable: decisión sin evidencia |
| **Auditoría embebida solo en PDP sin servicio** | Simple | Dificulta retención/consulta y viola separación del C4 |
| **Servicio dedicado + obligatoriedad** | **Elegida** | Producto + arquitectura C4 |

## Consecuencias

### Positivas

- Trazabilidad real del camino crítico.
- Soporte a threat model y compliance futuros.
- UC-01 exige postcondición de auditoría.

### Costos / riesgos

- Latencia adicional del hop a Auditoría (async aceptable si la decisión ya se emitió, pero el *intento* de registro es obligatorio; fallo de auditoría se registra como incidente técnico).
- Retención, cifrado y RBAC sobre la propia auditoría → Fase 0.F / PD-10.

### Implicaciones

- Contenedor Auditoría con Spring Boot + WebFlux + Modulith (ADR-009).
- Esquemas de evento en 05-Contracts.
