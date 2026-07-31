# Context Map

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Diagrama | [`diagrams/02-bounded-contexts.png`](diagrams/02-bounded-contexts.png) |

## Notación

| Tipo | Significado |
|---|---|
| **Customer–Supplier (C-S)** | El cliente necesita contrato estable del proveedor. |
| **Conformist** | Adopta IDs/estados del proveedor sin negociar. |
| **Shared Kernel** | Núcleo mínimo compartido (`TenantId`, `SujetoId`, `CorrelationId`, …). |
| **ACL** | Traduce modelo externo (IdP, OPA) al modelo interno. |
| **Published Language** | `SolicitudAcceso` / `DecisionAcceso` / `EventoAcceso`. |
| **«needed of»** | Relación del diagrama de contextos: dependencia de conocimiento/existencia. |

## Mapa (derivado del diagrama + ownership)

```text
Usuarios --«needed of»--> Identidad y autenticación --«needed of»--> Autorización
                                                                      |
                    Tenants -> Aplicaciones -> Recursos -> Roles -> Perfiles -> Asignaciones
                                                                      |              |
                                                                      +----> Políticas de acceso
                                                                      |              |
                                                                      v              v
                                                               Auditoría de seguridad
```

## Ownership

| Concepto | Dueño (BC) | Consumidores principales |
|---|---|---|
| `Tenant` | Tenants | Aplicaciones, Asignaciones, Autorización, Políticas |
| `Aplicacion` | Aplicaciones | Recursos, Asignaciones, Autorización |
| `Recurso` / acción | Recursos | Roles, Autorización |
| `Rol` | Roles | Perfiles, Asignaciones, Políticas, Autorización |
| `Perfil` | Perfiles | Asignaciones, Políticas |
| `Usuario` | Usuarios | Identidad, Asignaciones, Políticas, Autorización |
| Credencial / Sesión / Claims normalizados | Identidad y autenticación | Autorización (vía ACL) |
| `UsuarioAplicacionRol` y generadores | Asignaciones | Autorización (solo vigentes) |
| `Politica` / `ReglaPolitica` / `CondicionPolitica` | Políticas de acceso | Autorización (metadatos/versión); OPA (Rego) |
| `SolicitudAcceso` / `DecisionAcceso` / `EventoAcceso` | Autorización | PEP, Auditoría (Published Language) |
| `EventoAuditoria` / evidencia | Auditoría de seguridad | Operación (consulta) |

## Relaciones clave

| Desde | Hacia | Tipo | Notas |
|---|---|---|---|
| Usuarios | Identidad y autenticación | «needed of» / C-S | Diagrama 07 |
| Identidad | Autorización | «needed of» / C-S | Autorización no autentica |
| Autorización | Tenants…Asignaciones | Lectura C-S | Construye contexto |
| Asignaciones | Usuarios, Roles, Perfiles, Apps, Tenants | «needed of» | Diagrama 08 |
| Políticas de acceso | Usuarios, Roles, Perfiles, Apps, Tenants, Asignaciones | «needed of» | Diagrama 09 (metadatos; Rego no duplica todo) |
| Autorización | OPA | ACL | Motor externo |
| Identidad | IdP | ACL | Evidencia externa |
| Autorización / Políticas | Auditoría | Published Language / Conformist de IDs | Correlación obligatoria |

## Dependencias prohibidas (para Modulith / C4)

| Prohibido | Motivo |
|---|---|
| Autorización escribiendo catálogos (Roles, Apps, …) | Solo lee para decidir |
| PEP → SurrealDB / OPA | C4 L2: solo PDP habla con OPA y SurrealDB |
| Auditoría → OPA | Fuera de su responsabilidad |
| Identidad → Autorización (decidir) | Separación autenticación/autorización |
| Políticas de acceso → ejecutar Rego internamente como verdad | OPA es el motor; el BC gestiona metadatos/versión |

## Criterio de cierre

- [x] Ownership alineado a los 11 contextos del diagrama.
- [x] Separación Autorización vs Políticas vs Identidad vs Usuarios.

---

## Navegación

[**← 03 Bounded Contexts**](03-bounded-contexts.md) · [**05 Event Storming →**](05-event-storming.md)
