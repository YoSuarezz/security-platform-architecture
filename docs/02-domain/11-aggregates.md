# Agregados (candidatos)

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0 — candidatos baseline) |
| Fase | Inventario 0.B → refinamiento en 04-Data / diseño táctico |
| Advertencia | Son **candidatos Accepted** para la línea base, no fronteras congeladas de implementación. Se confirman con transacciones SurrealDB reales en 0.D. |

## Criterio usado

Un agregado agrupa entidades que deben cambiar **atómicas** respecto a una invariante local.

## Candidatos MVP

| Agregado raíz | Contiene / controla | Invariantes locales | BC |
|---|---|---|---|
| `Tenant` | Configuración mínima de estado | Tenant activo/inactivo afecta decisiones | Tenants |
| `Aplicacion` | Asociación a tenant, estado habilitado | App habilitada solo si tenant válido | Aplicaciones |
| `Recurso` | Acción(es) asociadas, vínculo a app | Recurso siempre referencia aplicación | Recursos |
| `Rol` | Autorizaciones `RolRecurso` | Alcance tenant/app o global inequívoco | Roles |
| `AsignacionPrivilegio` (`UsuarioAplicacionRol`) | Vigencia, estado, alcance | Solo estados válidos; no cruce indebido de tenant | Asignaciones |
| `PoliticaVersionada` | Versión publicada, vigencia, responsable, referencia a bundle Rego | INV-POL-01 al publicar | Políticas |
| `EvaluacionAcceso` (proceso) | `SolicitudAcceso` → `DecisionAcceso` | Siempre termina en tri-estado; dispara auditoría | Políticas |
| `Usuario` | Estado de sujeto, enlace a identidad externa | Estado ACTIVE (u homólogo) para participar | Identidad |

## No agregados (por ahora)

| Concepto | Por qué |
|---|---|
| OPA / bundle Rego completo | Infraestructura/política ejecutable; metadatos en `PoliticaVersionada` |
| PEP | Adaptador |
| `HashBlockchain` | Fuera de alcance |

## Próximo paso

App de validación cerrada ([`01-validation-application.md`](01-validation-application.md)). En 0.D se confirman fronteras con transacciones SurrealDB. **No** generar clases ni esquemas a partir de esta tabla como si fueran definitivas.

## Criterio de cierre 02-Domain

- [x] Candidatos documentados y Accepted como baseline.
- [ ] Confirmados con modelo lógico + prueba técnica de persistencia (04-Data — no bloquea Accepted de 02).

---

## Navegación

[**← 10 Entidades**](10-entities.md) · [**12 Value Objects →**](12-value-objects.md)
