# 02. Domain — Guía de revisión

| Campo | Valor |
|---|---|
| Fase | **0.B** |
| Estado | **Accepted (v1.0)** |
| Cómo revisar | Abre los archivos **en orden numérico** (`01` → `12`). En el explorador de Cursor ya aparecen ordenados. |
| Diagramas | Carpeta [`diagrams/`](diagrams/) — autoridad visual |

---

## Orden de lectura (scroll / archivo siguiente)

Lee de arriba abajo. Cada archivo tiene al final enlaces **← Anterior | Siguiente →**.

| # | Archivo | Qué aporta | Tiempo approx. |
|---|---|---|---|
| 1 | [01-validation-application.md](01-validation-application.md) | App real del MVP, admin, ABAC/REBAC dirección | 5 min |
| 2 | [02-ubiquitous-language.md](02-ubiquitous-language.md) | Vocabulario y estados canónicos | 5 min |
| 3 | [03-bounded-contexts.md](03-bounded-contexts.md) | Los 11 BC + enlace a diagramas | 10 min |
| 4 | [04-context-map.md](04-context-map.md) | Ownership y dependencias prohibidas | 8 min |
| 5 | [05-event-storming.md](05-event-storming.md) | Flujo crítico ALLOW/DENY/INDETERMINATE | 10 min |
| 6 | [06-invariants.md](06-invariants.md) | Condiciones que el sistema no puede violar | 8 min |
| 7 | [07-business-rules.md](07-business-rules.md) | Reglas de negocio trazables | 8 min |
| 8 | [08-use-cases.md](08-use-cases.md) | UC-01…06 camino crítico | 15 min |
| 9 | [09-domain-events.md](09-domain-events.md) | Eventos de dominio vs auditoría | 5 min |
| 10 | [10-entities.md](10-entities.md) | Índice de entidades → PNG por BC | 10 min |
| 11 | [11-aggregates.md](11-aggregates.md) | Fronteras de agregados (candidatos Accepted) | 5 min |
| 12 | [12-value-objects.md](12-value-objects.md) | VOs / shared kernel (candidatos Accepted) | 5 min |

### Diagramas (ver en paralelo, no al final)

| Cuándo | Diagrama |
|---|---|
| Con el paso 3 | [diagrams/01-big-picture.png](diagrams/01-big-picture.png) |
| Con el paso 3–4 | [diagrams/02-bounded-contexts.png](diagrams/02-bounded-contexts.png) |
| Con el paso 10 | [diagrams/models/](diagrams/models/) (01…11) |

---

## Mapa mental de la carpeta

```text
01 App de validación     →  contexto de negocio del MVP
02 Lenguaje              →  cómo se habla
03–04 BC + Context Map   →  cómo se parte el dominio
05 Event Storming        →  qué pasa en runtime
06–08 Invariantes/Reglas/UC →  qué debe cumplirse
09–12 Eventos/Entidades/Agg/VO →  inventario de modelo
```

---

## Los 11 Bounded Contexts (atajo)

| BC | Dónde vive en Modulith |
|---|---|
| Tenants | PDP · `tenants` |
| Aplicaciones | PDP · `aplicaciones` |
| Recursos | PDP · `recursos` |
| Roles | PDP · `roles` |
| Perfiles | PDP · `perfiles` |
| Usuarios | PDP · `usuarios` |
| Identidad y autenticación | PDP · `identidad` |
| Asignaciones | PDP · `asignaciones` |
| Políticas de acceso | PDP · `politicas` |
| Autorización | PDP · `autorizacion` (+ PEP módulos de enforcement) |
| Auditoría de seguridad | Contenedor **Auditoría** · Modulith (`ingestion`, `query`, `retention`) |

> Modulith también organiza el **PEP** (capacidades de enforcement), no solo el PDP — [ADR-009](../01-governance/adr/ADR-009-spring-modulith.md).

Detalle: [03-bounded-contexts.md](03-bounded-contexts.md) · Módulos: [`../03-architecture/modulith-dependency-map.md`](../03-architecture/modulith-dependency-map.md)

---

**Empezar revisión →** [01-validation-application.md](01-validation-application.md)
