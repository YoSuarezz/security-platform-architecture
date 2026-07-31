# Diagramas de dominio

Fuente visual del modelo de dominio. Los Markdown de `docs/02-domain/` interpretan estos diagramas; si hay conflicto, **el diagrama aprobado manda** y el Markdown se actualiza.

**Revisión textual en orden:** volver a [`../README.md`](../README.md) (`01`→`12`). Abrir estos PNG en paralelo con los pasos 3, 4 y 10.

## Índice

| Archivo | Contenido |
|---|---|
| [01-big-picture.png](01-big-picture.png) | Big Picture — dominio `Gestion Seguridad` |
| [02-bounded-contexts.png](02-bounded-contexts.png) | Modelo de contextos delimitados |
| [models/](models/) | Modelo de dominio anémico acotado por contexto |

## Modelos por contexto

| # | Archivo | Contexto |
|---|---|---|
| 01 | [models/01-tenants.png](models/01-tenants.png) | Tenants |
| 02 | [models/02-aplicaciones.png](models/02-aplicaciones.png) | Aplicaciones |
| 03 | [models/03-recursos.png](models/03-recursos.png) | Recursos |
| 04 | [models/04-roles.png](models/04-roles.png) | Roles |
| 05 | [models/05-perfiles.png](models/05-perfiles.png) | Perfiles |
| 06 | [models/06-usuarios.png](models/06-usuarios.png) | Usuarios |
| 07 | [models/07-identidad-autenticacion.png](models/07-identidad-autenticacion.png) | Identidad y autenticación |
| 08 | [models/08-asignaciones.png](models/08-asignaciones.png) | Asignaciones |
| 09 | [models/09-politicas-acceso.png](models/09-politicas-acceso.png) | Políticas de acceso |
| 10 | [models/10-autorizacion.png](models/10-autorizacion.png) | Autorización (evaluación) |
| 11 | [models/11-auditoria-seguridad.png](models/11-auditoria-seguridad.png) | Auditoría de seguridad |

## Convención de nombres

Prefijo numérico = orden de lectura / elaboración. No editar PNG aquí sin versionar el fuente (drawio/u otro) en `docs/assets/diagrams/` cuando exista.
