# ADR-008: Clean Architecture mediante arquitectura hexagonal

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §8.7, §18.1 |

## Contexto

El dominio de seguridad no debe acoplarse a Spring, OPA, SurrealDB ni APIs externas.

## Decisión

Aplicar **Clean Architecture** implementada con **arquitectura hexagonal**: toda dependencia apunta hacia el dominio; OPA, SurrealDB e IdP viven solo en adaptadores.

## Consecuencias

- Puertos de evaluación de políticas y repositorio de seguridad son contratos del dominio.
- Facilita pruebas de invariantes sin levantar infraestructura.
