# ADR-006: Modelos de autorización combinables (RBAC + ABAC + REBAC + PBAC)

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §13 |

## Contexto

Una decisión de acceso puede depender de roles, atributos, relaciones y políticas.

## Decisión

Soportar **RBAC, ABAC, REBAC y PBAC** de forma combinable, implementados por incrementos. El MVP no exige todos completos.

## Consecuencias

- Semántica de precedencia debe cerrarse antes de producción (ADR dedicado pendiente).
- Camino crítico inicial: RBAC simple vía OPA.
