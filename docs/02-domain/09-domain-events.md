# Eventos de dominio

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Relacionado | [`05-event-storming.md`](05-event-storming.md), Fase 0.E (catálogo de auditoría) |

## Distinción importante

| Tipo | Propósito | Dueño |
|---|---|---|
| **Evento de dominio** | Hecho significativo del modelo de seguridad (alguien publicó una política, se asignó un rol, se emitió una decisión). | Bounded context emisor |
| **Evento de auditoría** | Evidencia persistente para explicación, soporte y cumplimiento. | BC Auditoría |

Un evento de dominio **puede originar** un evento de auditoría; no son lo mismo. El esquema detallado de auditoría se cierra en Fase 0.E/0.F.

## Catálogo mínimo (camino crítico)

| Evento | BC emisor | Cuándo | Datos mínimos (sin secretos) | ¿Auditable? |
|---|---|---|---|---|
| `AplicacionRegistrada` | Aplicaciones | UC-02 | `aplicacionId`, `tenantId`, timestamp | Sí (admin) |
| `RecursoCatalogado` | Recursos | UC-05 | `recursoId`, `aplicacionId`, `accion`, timestamp | Sí (admin) |
| `RolDefinido` | Roles y perfiles | UC-06 | `rolId`, alcance, recursos/acciones, timestamp | Sí (admin) |
| `AsignacionCreada` | Asignaciones | UC-03 | `asignacionId`, `sujetoId`, `rolId`, alcance, vigencia | Sí |
| `AsignacionVigente` | Asignaciones | Activación | ids + estado | Sí |
| `AsignacionRevocada` | Asignaciones | Revocación | ids + motivo código | Sí |
| `AsignacionVencida` | Asignaciones | Fin de vigencia | ids | Sí |
| `PoliticaPublicada` | Políticas | UC-04 | `politicaId`, `version`, alcance, responsable, vigencia | Sí |
| `PoliticaRevocada` | Políticas | Retiro | `politicaId`, `version` | Sí |
| `IdentidadValidada` | Identidad | UC-01 | `sujetoId`, `tenantId`, `idpRef` (no token) | Selectivo |
| `IdentidadRechazada` | Identidad | UC-01 | `reasonCode`, `correlationId` | Sí |
| `SolicitudAccesoCreada` | Políticas | UC-01 | ids de correlación, app, recurso, acción | Suele fundirse en auditoría de decisión |
| `DecisionAccesoEmitida` | Políticas | UC-01 | `decisionId`, decisión, `reasonCode`, `policyReferences`, `correlationId` | **Obligatorio** |
| `EventoAuditoriaRegistrado` | Auditoría | Post-decisión | ids de correlación + payload de evidencia | Es el registro mismo |

## Eventos diferidos (no MVP)

| Evento | Motivo de diferir |
|---|---|
| Eventos ricos de REBAC (`RelacionCreada`, …) | Etapa 3 |
| Eventos de ABAC / clasificación de atributos | Etapa 3 |
| Eventos de UI administrativa | Frontend pendiente |

## Reglas de publicación

1. Los eventos no transportan tokens, contraseñas ni PII innecesaria (INV-SEC-06, INV-ID-03).
2. Todo `DecisionAccesoEmitida` debe poder correlacionarse con un intento de `EventoAuditoriaRegistrado`.
3. Los nombres usan pasado perfecto / pretérito (`…Emitida`, `…Registrada`), no imperativo.
4. El transporte (in-process Modulith events vs. bus) se decide en Fase 0.C/0.G; este catálogo es semántico.

## Criterio de cierre

- [x] Catálogo mínimo alineado al Event Storming y a UC P0.
- [ ] Esquema de auditoría formal en 0.E (campos §16.1).

---

## Navegación

[**← 08 Casos de uso**](08-use-cases.md) · [**10 Entidades →**](10-entities.md)
