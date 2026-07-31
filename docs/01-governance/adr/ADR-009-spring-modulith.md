# ADR-009: Organización interna con Spring Modulith

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §8.8, §11 |

## Contexto

Se requiere organización modular por capacidad de negocio, no solo por capas técnicas, con límites verificables.

## Decisión

Usar **Spring Modulith**. Los módulos alinean con los contextos: tenants, aplicaciones, recursos, identidad, asignaciones, políticas, auditoría.

## Consecuencias

- El Dependency Map se verifica en CI.
- Evita Modulith “solo nominal”.
