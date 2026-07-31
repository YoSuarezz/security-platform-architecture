# ADR-011: Línea base de ingeniería (Git, CI/CD, pruebas, Clean Code)

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §8.6, §8.9, §8.10, §8.11, §8.12 |

## Contexto

Iniciar historias funcionales sin Git, CI, pruebas y estándares definidos es un riesgo explícito del proyecto.

## Decisión

Antes de la Etapa 1 deben existir: **estrategia Git** (main, develop, feature branches, PR obligatorios), **pipeline CI** (compilación, análisis estático, pruebas, cobertura), **estrategia de pruebas** con cobertura **>80 %**, y **Clean Code** como estándar.

## Consecuencias

- La Fase 0.G es bloqueante.
- El gate de límites Modulith forma parte del pipeline.
