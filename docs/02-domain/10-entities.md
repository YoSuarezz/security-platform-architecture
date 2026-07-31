# Entidades de dominio (catálogo conceptual)

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Autoridad visual | [`diagrams/models/`](diagrams/models/) |
| Nota | Catálogo conceptual. No son clases Java ni esquema físico SurrealDB. |

## Índice de modelos

| BC | Diagrama | Entidades principales (del diagrama) |
|---|---|---|
| Tenants | [01](diagrams/models/01-tenants.png) | `Tenant`, `TipoTenant`, `ConfiguracionTenant`, `TenantSuscripcion` |
| Aplicaciones | [02](diagrams/models/02-aplicaciones.png) | Ver diagrama (p. ej. `Aplicacion`, vínculos a tenant/módulos) |
| Recursos | [03](diagrams/models/03-recursos.png) | `Recurso`, tipología/jerarquía según diagrama |
| Roles | [04](diagrams/models/04-roles.png) | `Rol`, vínculos a recurso/acción |
| Perfiles | [05](diagrams/models/05-perfiles.png) | `Perfil`, vínculo a roles |
| Usuarios | [06](diagrams/models/06-usuarios.png) | `Usuario`, estado |
| Identidad y autenticación | [07](diagrams/models/07-identidad-autenticacion.png) | `UsuarioCredencial` → `ProveedorIdentidad` → `SesionUsuario` → `TokenJWT` → `Claim` |
| Asignaciones | [08](diagrams/models/08-asignaciones.png) | `UsuarioPerfil`, `PlanRol`, `AsignacionPendienteRol` → `UsuarioAplicacionRol` |
| Políticas de acceso | [09](diagrams/models/09-politicas-acceso.png) | `Politica` → `ReglaPolitica` → `CondicionPolitica` |
| Autorización | [10](diagrams/models/10-autorizacion.png) | `InterceptorAcceso` → `SolicitudAcceso` → `MotorAutorizacionOPA` → `DecisionAcceso` → `EventoAcceso` |
| Auditoría | [11](diagrams/models/11-auditoria-seguridad.png) | Evidencia / registros de auditoría (ver diagrama) |

## MVP vs. diferido (esqueleto Etapa 1)

| Entidad / concepto | MVP | Notas |
|---|---|---|
| `Tenant` (activo) | Sí | |
| `Aplicacion` + vínculo tenant | Sí | |
| `Recurso` + acción | Sí | Jerarquía profunda diferida |
| `Rol` + autorización recurso | Sí | |
| `Perfil` / `PerfilRol` | No | Etapa 2 |
| `Usuario` + estado | Sí | |
| Flujo IdP / normalización | Sí (mínimo) | No persistir JWT completo |
| `UsuarioCredencial` local | No | Auth delegada |
| `UsuarioAplicacionRol` vigente | Sí | |
| `UsuarioPerfil` / `PlanRol` / pendiente | No | Etapa 2 |
| `Politica` versionada + Rego | Sí | Estructura Regla/Condición puede ser metadato liviano; no duplicar Rego |
| Cadena Autorización (`SolicitudAcceso`/`DecisionAcceso`) | Sí | |
| `EventoAuditoria` correlacionado | Sí | |
| `HashBlockchain` | No | Investigación |

## Reglas al leer los diagramas anémicos

1. Son **modelos anémicos acotados**: inventarian conceptos y relaciones; la lógica rica vive en casos de uso / invariantes.
2. `MotorAutorizacionOPA` e `InterceptorAcceso` en el modelo de Autorización son **conceptos de dominio/proceso**; en C4 se materializan como OPA (contenedor) y componentes del PEP/PDP.
3. `TokenJWT` en Identidad es concepto de evidencia en tránsito — **no** implica persistencia del token completo (INV-ID-03).

## Criterio de cierre

- [x] Cada BC del diagrama de contextos tiene modelo visual referenciado.
- [x] Flag MVP para camino crítico.
- [ ] Atributos/cardinalidades literales transcritos uno a uno desde cada PNG (opcional si el PNG se mantiene como autoridad).

---

## Navegación

[**← 09 Eventos de dominio**](09-domain-events.md) · [**11 Agregados →**](11-aggregates.md)
