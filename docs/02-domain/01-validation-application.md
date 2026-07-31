# Aplicación de validación del MVP

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Cierra | PD-01, PD-06 (admin), PD-07 (dirección ABAC), PD-08 (REBAC inicial) |
| Dominio | Educación superior |

## Decisión PD-01

La primera aplicación de validación del MVP es un **Sistema de Gestión Académica Universitaria**.

| Aspecto | Valor |
|---|---|
| Nombre lógico | `gestion-academica` (código de aplicación en plataforma) |
| Dominio | Educación superior |
| Tenant | Cada **universidad** es un tenant independiente |

### Recursos iniciales (catálogo MVP)

| Recurso | Código sugerido | Notas |
|---|---|---|
| Usuarios | `usuarios` | Usuarios de la app académica (no confundir con BC Usuarios de la plataforma) |
| Estudiantes | `estudiantes` | |
| Docentes | `docentes` | |
| Cursos | `cursos` | |
| Matrículas | `matriculas` | |
| Calificaciones | `calificaciones` | |

> Los códigos son IDs de plataforma (PD-09 / principio §21.7). No se usan paths HTTP crudos como verdad única.

### Acciones iniciales

| Acción | Código |
|---|---|
| Crear | `crear` |
| Consultar | `consultar` |
| Actualizar | `actualizar` |
| Eliminar | `eliminar` |
| Aprobar | `aprobar` |
| Rechazar | `rechazar` |
| Asignar | `asignar` |

### Justificación

Dominio fácil de entender; permite demostrar RBAC (MVP), ABAC y luego REBAC con políticas realistas sin depender de un negocio propietario.

Las apps técnicas de `demo-security` (Django, PHP, .NET, Java) siguen como **apps protegidas de referencia tecnológica**; `gestion-academica` es el **dominio de negocio** que valida políticas y datos.

---

## Decisión PD-06 — Administración del MVP

Existe un único rol administrativo de la plataforma:

**Administrador de Seguridad (Security Administrator)**

Responsabilidades en el MVP:

- Gestión de tenants
- Registro de aplicaciones
- Gestión de recursos
- Gestión de roles
- Gestión de perfiles
- Gestión de asignaciones
- Gestión de políticas
- Consulta de auditoría

No hay separación de funciones (Super Admin / Tenant Admin / Policy Manager) en el MVP. Podrá incorporarse después sin cambiar los BC (solo roles/asignaciones de la propia plataforma).

Actores de UC-02…UC-06: **Administrador de Seguridad**.

---

## PD-07 — Atributos ABAC (dirección, Etapa 3)

**Estado:** pendiente de detalle fino en Etapa 3. Dirección aceptada:

| Atributo previsto | Origen típico |
|---|---|
| Tenant | Plataforma |
| Rol | Asignaciones |
| Aplicación | Catálogo |
| Recurso | Catálogo / solicitud |
| Acción | Solicitud |
| Hora | Entorno |
| Dirección IP | Entorno (PEP) |
| Claims del token | IdP → Identidad (normalizados) |
| Estado del usuario | Usuarios / Identidad |

Los atributos de negocio específicos los define cada aplicación protegida. No bloquean el esqueleto RBAC del MVP.

---

## PD-08 — Relaciones REBAC

### Primera relación (MVP / base)

```
Usuario pertenece a Tenant
```

### Posteriores (Etapa 3+)

- Usuario pertenece a Equipo
- Usuario es propietario de Recurso
- Usuario administra Aplicación

La primera relación ya está implícita en el modelo multi-tenant; las demás se formalizan en 04-Data / Etapa 3.

---

## Navegación

[**← Índice**](README.md) · [**02 Lenguaje ubicuo →**](02-ubiquitous-language.md)
