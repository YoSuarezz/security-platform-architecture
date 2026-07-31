# Deployment

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |

## Documento

| Artefacto | Descripción |
|---|---|
| [deployment-mvp.md](deployment-mvp.md) | Topología Docker Compose MVP, Keycloak, Cloud Enable, variables de entorno |

## Contenido cubierto

1. Diagrama de despliegue MVP (PEP, PDP, OPA, SurrealDB, Auditoría, Keycloak, apps)
2. Anotaciones Cloud Enable (sin host fijo, secretos externos, paridad CI)
3. Camino a Kubernetes sin reescritura de código
4. Mapeo HTTP de decisiones (referencia ADR-014)

## Entradas

- Contenedores: [`../c4/level2-containers.png`](../c4/level2-containers.png)
- ADR-010 · ADR-013 · ADR-014
- App de validación: [`../../02-domain/01-validation-application.md`](../../02-domain/01-validation-application.md)
