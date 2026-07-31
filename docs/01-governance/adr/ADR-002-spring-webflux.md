# ADR-002: Spring WebFlux para todo el tráfico HTTP e I/O remoto

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §6.2, §8.4 |

## Contexto

El PEP y el PDP dependen de llamadas de red hacia OPA, SurrealDB y el IdP en el camino crítico. Se requiere latencia acotada bajo concurrencia.

## Decisión

Todo componente HTTP e integración remota se construye con **Spring WebFlux**. No se permiten componentes bloqueantes en el flujo reactivo sin aislamiento y justificación explícitos.

## Consecuencias

- La demo actual basada en Spring MVC / HttpClient bloqueante no es base de producción.
- Debe existir una prueba técnica temprana de WebFlux en la Fase 0.
