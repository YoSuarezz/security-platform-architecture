# ADR-003: SurrealDB como persistencia principal

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §6.1, §12 |

## Contexto

El dominio combina entidades con atributos propios y relaciones navegables (REBAC, asignaciones, jerarquías).

## Decisión

Usar **SurrealDB** como fuente de verdad del dominio de seguridad (documento + grafo). SQL no es la arquitectura objetivo.

## Consecuencias

- Cada relación de grafo debe justificarse por consulta de autorización.
- Se requieren pruebas técnicas de consultas, transacciones, índices y aislamiento por tenant.
