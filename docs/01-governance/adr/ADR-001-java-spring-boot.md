# ADR-001: Java y Spring Boot como base de la plataforma

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6 |

## Contexto

Se requiere un lenguaje y framework maduros para APIs, seguridad, pruebas e integración de un componente de autorización central.

## Decisión

Usar **Java** como lenguaje principal y **Spring Boot** como framework base.

## Alternativas consideradas

1. Otros lenguajes/runtimes — descartados por decisión confirmada del Documento Base.

## Consecuencias

- El núcleo, PEP y PDP se construyen en Java/Spring Boot.
- Las aplicaciones protegidas pueden ser heterogéneas; no deben compartir el runtime.
