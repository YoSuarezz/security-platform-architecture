# Value Objects (candidatos)

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0 — candidatos baseline) |
| Fase | 0.B inventario → tipado en diseño táctico / commons Modulith |
| Nota | Sin tipos Java. Nombres de lenguaje ubicuo. Alineados a `commons` en el Dependency Map. |

## Candidatos transversales (Shared Kernel pequeño)

| VO | Significado | Reglas |
|---|---|---|
| `TenantId` | Identificador de tenant | Obligatorio en decisiones sensibles |
| `SujetoId` | Identificador interno de sujeto | Distinto del subject del token crudo |
| `AplicacionId` | Identificador de app protegida | |
| `RecursoId` | Identificador de recurso | |
| `CorrelationId` | Correlación extremo a extremo | Se propaga PEP→PDP→Auditoría |
| `RequestId` | Id de la solicitud | |
| `DecisionId` | Id único de decisión | |
| `ReasonCode` | Código estable no sensible | Catálogo cerrado progresivo |
| `PolicyVersion` | Versión de política publicada | |

## Candidatos de autorización

| VO | Significado | Reglas |
|---|---|---|
| `Accion` | Operación pedida (`approve`, `read`, …) | Explícita en catálogo |
| `Decision` | `ALLOW` \| `DENY` \| `INDETERMINATE` | Solo esos valores |
| `Vigencia` | Intervalo inicio/fin | Usada por asignaciones y políticas |
| `AlcanceAsignacion` | Tenant + aplicación (+ opcional entorno) | Coherente con INV-ASN-02 |
| `ReferenciaPolitica` | Id + versión evaluada | Va a `DecisionAcceso` y auditoría |

## Candidatos de identidad

| VO | Significado | Reglas |
|---|---|---|
| `EvidenciaIdentidad` | Referencia validable (no el token persistido) | No se audita en claro el token completo |
| `EstadoSujeto` | p. ej. ACTIVE / DISABLED | |

## Diferidos

| VO | Cuándo |
|---|---|
| Atributos ABAC tipados / clasificación | Etapa 3 |
| Relación REBAC tipada | Etapa 3 |

## Criterio de cierre 02-Domain

- [x] Candidatos del camino crítico listados y Accepted como baseline.
- [ ] Tipado/validación exacta en diseño detallado / commons (no bloquea Accepted de 02).

---

## Navegación

[**← 11 Agregados**](11-aggregates.md) · [**Volver al índice →**](README.md)
