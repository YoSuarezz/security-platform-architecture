# ADR-010: Cloud Native, Cloud Enable y 12-Factor

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §8.2, §8.3, §8.5 |

## Contexto

La plataforma debe ser desplegable en contenedores y preparada para migrar a Kubernetes/cloud sin reescritura, aunque el entorno productivo final esté pendiente.

## Decisión

Cumplir principios **Cloud Native**, **Cloud Enable** y **12-Factor** (configuración externa, procesos sin estado, servicios adjuntos, logs como flujos, paridad de entornos).

## Consecuencias

- Sin host fijo ni estado en filesystem local.
- Secretos y configuración fuera del artefacto.
