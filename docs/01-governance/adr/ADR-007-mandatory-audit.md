# ADR-007: Auditoría obligatoria de decisiones

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §16 |

## Contexto

Las decisiones de acceso no pueden ser silenciosas; deben ser explicables y correlacionables.

## Decisión

La **auditoría es obligatoria**. Toda evaluación genera evidencia con correlación (`requestId`, `decisionId`, `correlationId`) sin registrar secretos ni tokens completos.

## Consecuencias

- Retención, cifrado y acceso a auditoría se especifican en Fase 0.F.
- `HashBlockchain` no es requisito; es investigación.
