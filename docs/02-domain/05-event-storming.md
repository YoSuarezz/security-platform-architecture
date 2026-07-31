# Event Storming — Flujo crítico de autorización

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0 — baseline documental) |
| Nota | Sesión formal de ratificación opcional para Baseline v1.0 del repo; no bloquea Accepted de 02-domain |
| Fase | 0.B |
| Alcance | Camino crítico Etapa 1 / historias de esqueleto |
| Fuente | Documento Base §9.2, §14, §15, §16, §18.1, §19 |
| Diagramas | [`diagrams/01-big-picture.png`](diagrams/01-big-picture.png), [`diagrams/02-bounded-contexts.png`](diagrams/02-bounded-contexts.png), modelo Autorización [`diagrams/models/10-autorizacion.png`](diagrams/models/10-autorizacion.png) |

> Este artefacto captura el resultado **estructurado** del Event Storming del flujo crítico. No sustituye una sesión presencial con sticky notes; la fija como modelo compartido derivado de decisiones ya confirmadas. Cualquier cambio de semántica debe actualizar este documento y, si aplica, un ADR.

## 1. Hotspot / dominio bajo estudio

**Decisión de acceso a una aplicación protegida** mediante PEP → Identidad → PDP/Políticas → SurrealDB → OPA → Auditoría → Enforcement.

## 2. Actores

| Actor | Tipo | Rol en el flujo |
|---|---|---|
| Cliente | Externo | Inicia la solicitud HTTP hacia la app protegida. |
| Aplicación protegida | Sistema externo | Destino del tráfico si `ALLOW`. |
| Administrador de Seguridad | Humano | Único admin MVP: registra apps, asigna roles, publica políticas, gestiona catálogos y consulta auditoría. |
| IdP | Sistema externo | Emite evidencia de identidad. |
| OPA | Sistema externo | Evalúa Rego. |
| Plataforma (PEP/PDP/… ) | Sistema | Orquesta y aplica. |

## 3. Pivote temporal del flujo crítico (Big Picture)

Orden lógico de dominio (no de capas técnicas):

```text
[Preparación]
AplicacionRegistrada
RecursoCatalogado
RolDefinido
AsignacionCreada / AsignacionVigente
PoliticaPublicada

[Solicitud]
SolicitudRecibidaEnPEP
IdentidadDetectadaOAusente
IdentidadValidada  |  IdentidadRechazada
SujetoNormalizado
SolicitudAccesoCreada
ContextoAutorizacionConstruido
DatosSeguridadResueltos          (tenant, roles, recurso, relaciones mínimas)
EntradaOPAPreparada
PoliticaEvaluada
DecisionAccesoEmitida            (ALLOW | DENY | INDETERMINATE)
EventoAuditoriaRegistrado
DecisionAplicadaPorPEP
  ├─ SolicitudReenviadaAAplicacion     (ALLOW)
  └─ AccesoBloqueadoAlCliente          (DENY / INDETERMINATE→negación controlada)
```

## 4. Eventos de dominio (naranja)

| Evento | Emisor (BC) | Disparado por | Notas |
|---|---|---|---|
| `AplicacionRegistrada` | Aplicaciones | Admin / UC RegistrarAplicacion | Precondición del camino crítico |
| `RecursoCatalogado` | Recursos | Admin / registro conjunto | Relaciona app + acción |
| `RolDefinido` | Roles y perfiles | Admin | RBAC mínimo |
| `AsignacionCreada` | Asignaciones | Admin / UC AsignarRol | Puede quedar PENDIENTE |
| `AsignacionActivada` / `AsignacionVigente` | Asignaciones | Tiempo o aprobación | Solo vigentes entran a decisión |
| `AsignacionRevocada` / `AsignacionVencida` | Asignaciones | Admin / tiempo | Dejan de participar |
| `PoliticaPublicada` | Políticas | Admin / UC PublicarPolitica | Versión + vigencia |
| `PoliticaRevocada` | Políticas | Admin | |
| `SolicitudRecibida` | (adaptador PEP) | Cliente | Hecho de borde; alimenta Políticas/Identidad |
| `IdentidadValidada` | Identidad | Evidencia IdP OK | |
| `IdentidadRechazada` | Identidad | Token inválido / ausente no recuperable | Suele producir DENY |
| `SujetoNormalizado` | Identidad | Post-validación | Independiente del formato de claims |
| `SolicitudAccesoCreada` | Políticas | UC EvaluarAcceso | Published language |
| `ContextoAutorizacionListo` | Políticas | Datos resueltos | |
| `DecisionAccesoEmitida` | Políticas | Respuesta OPA + reglas locales | Tri-estado |
| `EventoAuditoriaRegistrado` | Auditoría | Tras decisión (y/o enforcement) | Correlación obligatoria |
| `AccesoPermitido` | (PEP aplica) | Decision ALLOW | Hecho de enforcement |
| `AccesoDenegado` | (PEP aplica) | DENY o INDETERMINATE controlado | |

## 5. Comandos (azul)

| Comando | Actor | BC | Resultado esperado |
|---|---|---|---|
| `RegistrarAplicacion` | Admin | Aplicaciones | `AplicacionRegistrada` |
| `CatalogarRecurso` | Admin | Recursos | `RecursoCatalogado` |
| `DefinirRol` | Admin | Roles | `RolDefinido` |
| `AsignarRol` | Admin | Asignaciones | `AsignacionCreada` (+ activación) |
| `PublicarPolitica` | Admin | Políticas | `PoliticaPublicada` |
| `EvaluarAcceso` | PEP / sistema | Políticas | `DecisionAccesoEmitida` + auditoría |
| `ValidarIdentidad` | PEP / sistema | Identidad | `IdentidadValidada` o `IdentidadRechazada` |
| `AplicarDecision` | PEP | (adaptador) | Reenvío o bloqueo |

## 6. Agregados / políticas de consistencia (candidatos — ver aggregates.md)

| Zona | Candidato | Invariantes locales |
|---|---|---|
| Asignación | `AsignacionPrivilegio` | No vigente fuera de fechas; tenant/app coherentes |
| Política | `PoliticaVersionada` | Publicada ⇒ versión + responsable + vigencia |
| Evaluación | `SolicitudAcceso` (ciclo de evaluación) | Siempre termina en decisión + intento de auditoría |
| Tenant | `Tenant` | Estado activo/inactivo afecta decisiones |

## 7. Políticas de proceso / reglas en el flujo

| Punto del flujo | Regla |
|---|---|
| Antes de evaluar | Identidad inválida → no se evalúa política de negocio; `DENY` (`TOKEN_INVALID` o equivalente) |
| Sin política aplicable | `DENY` por defecto ([ADR-012](../01-governance/adr/ADR-012-policy-precedence.md)) |
| Error OPA / SurrealDB / IdP | `INDETERMINATE` → PEP responde **503** ([ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md)) |
| Token inválido | `DENY` → PEP responde **401** |
| Deny explícito / allow ausente | `DENY` → PEP responde **403** |
| Tras cualquier decisión | Debe existir intento de `EventoAuditoriaRegistrado` correlacionado |
| Claims del cliente | Nunca bastan solos para autorizar sin contraste con asignaciones/políticas |

## 8. Lecturas del modelo (preguntas que el flujo obliga a responder)

Derivadas de §12.3; cada una debe tener respaldo en el modelo de datos (Fase 0.D):

1. ¿Tenant del sujeto y está activo?
2. ¿Aplicación registrada y habilitada para ese tenant?
3. ¿Recurso y acción de la operación?
4. ¿Roles/perfiles vigentes del sujeto en app+tenant?
5. ¿Política activa y versión?
6. (Posterior) ¿Relación REBAC válida?
7. ¿Decisión previa / correlación para soporte?

## 9. Bounded contexts tocados por el camino crítico

Alineado a los **11 BC** del diagrama de contextos:

| Fase del flujo | Contextos |
|---|---|
| Preparación | Tenants, Aplicaciones, Recursos, Roles, (Perfiles), Usuarios, Asignaciones, Políticas de acceso |
| Runtime | Identidad y autenticación → **Autorización** (lee catálogos + metadatos de políticas; invoca OPA) → Auditoría |
| Enforcement | PEP materializa interceptor/aplicador del BC Autorización (sin datos ni políticas) |

## 10. Decisiones de dominio aplicadas a este storming

| # | Pregunta (§21) | Estado | Dónde |
|---|---|---|---|
| 1 | App de validación | **Cerrada** — Gestión Académica Universitaria | [`01-validation-application.md`](01-validation-application.md) |
| 3 | Quién administra | **Cerrada** — Administrador de Seguridad único (MVP) | idem |
| 5 | Atributos ABAC | Dirección aceptada; detalle Etapa 3 | idem |
| 6 | REBAC | Primera: Usuario ∈ Tenant; resto Etapa 3+ | idem |
| 7 | IDs estables app/recurso/acción | Principio: códigos de plataforma (formalizar en 0.D/0.E) | PD-09 |

## 11. Criterio de cierre

- [x] Flujo ALLOW / DENY / INDETERMINATE representado.
- [x] Eventos y comandos del camino crítico listados.
- [x] Alineado a los 11 Bounded Contexts.
- [x] Decisiones PD-01/PD-02/PD-04/PD-06 reflejadas.
- [x] Baseline documental Accepted.

---

## Navegación

[**← 04 Context Map**](04-context-map.md) · [**06 Invariantes →**](06-invariants.md)
