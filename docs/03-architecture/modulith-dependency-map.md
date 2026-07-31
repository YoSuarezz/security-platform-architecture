# Mapa de módulos y Dependency Map — Spring Modulith

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.C |
| Decisión de agrupación | **11 módulos** — uno por Bounded Context de [`docs/02-domain/03-bounded-contexts.md`](../02-domain/03-bounded-contexts.md) |
| ADR | ADR-009 (Spring Modulith) |
| Contenedor principal | **PDP** (Spring Boot + WebFlux): módulos `tenants` … `autorizacion` |
| Contenedor separado | **Auditoría** (propio jar; `auditoria` es su único módulo interno) |
| Contenedor delgado | **PEP** (sin módulos de negocio; solo infraestructura de interceptación) |

---

## Decisión de agrupación (tomada con autoridad)

Se mantienen **11 módulos** separados, uno por BC, porque:

1. Los diagramas elaborados (`diagrams/models/01…11`) modelan responsabilidades distintas y no solapadas.
2. Roles y Perfiles son BC separados en el diagrama (3.4 y 3.5) con entidades distintas.
3. Usuarios e Identidad son BC separados: `Usuarios` posee al sujeto de dominio; `Identidad` posee credenciales/sesión/claims — fusionarlos acoplaría autenticación con identidad de dominio, violando ADR-005.
4. Autorización y Políticas de acceso son BC distintos: `Políticas` gestiona metadatos/versión; `Autorización` orquesta la evaluación — fusionarlos pondría el catálogo dentro del runtime de decisión.

Si el equipo decide consolidar en el futuro (p. ej. `roles` + `perfiles` → `acceso-rbac`), se documenta mediante ADR y se actualiza este mapa. **No se adelanta esa fusión sin caso técnico demostrado.**

---

## Distribución por contenedor C4

```text
┌──────────────────────────────────────────────────────┐
│  Contenedor: PDP (Spring Boot + WebFlux + Modulith)  │
│                                                      │
│  Módulos de dominio/aplicación:                      │
│  ┌──────────┐  ┌─────────────┐  ┌──────────┐        │
│  │ tenants  │  │ aplicaciones│  │ recursos │        │
│  └──────────┘  └─────────────┘  └──────────┘        │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐          │
│  │  roles   │  │ perfiles │  │ usuarios  │          │
│  └──────────┘  └──────────┘  └───────────┘          │
│  ┌───────────┐  ┌─────────────┐                     │
│  │ identidad │  │ asignaciones│                     │
│  └───────────┘  └─────────────┘                     │
│  ┌───────────┐  ┌──────────────┐                    │
│  │ politicas │  │ autorizacion │                    │
│  └───────────┘  └──────────────┘                    │
│                                                      │
│  commons (shared kernel — NO módulo Modulith,        │
│            paquete transversal de VOs/IDs)           │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│  Contenedor: Auditoría (jar independiente)   │
│  Módulo interno: auditoria                   │
│  Consume: Published Language (eventos)       │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│  Contenedor: PEP (jar independiente)         │
│  Sin módulos de dominio.                     │
│  Paquetes: normalization, enforcement,       │
│            pdp-client (adaptador HTTP PDP)   │
└──────────────────────────────────────────────┘
```

---

## Dependency Map (reglas Spring Modulith)

### Dependencias permitidas en el PDP

```text
commons ←── (todos los módulos; solo consume VOs/IDs, no lógica)

tenants       ←── aplicaciones
                   ←── recursos
                   ←── usuarios
                   ←── asignaciones
                   ←── politicas (lectura)
                   ←── autorizacion

aplicaciones  ←── recursos
                   ←── asignaciones
                   ←── autorizacion

recursos      ←── roles (RolRecurso)
                   ←── autorizacion

roles         ←── perfiles (PerfilRol)
                   ←── asignaciones
                   ←── autorizacion

perfiles      ←── asignaciones
                   ←── autorizacion

usuarios      ←── identidad
                   ←── asignaciones
                   ←── autorizacion

identidad     ←── autorizacion

asignaciones  ←── politicas (lectura de asignaciones vigentes)
                   ←── autorizacion

politicas     ←── autorizacion (lectura de metadatos/versión)

autorizacion  (no puede depender de internals de ningún módulo
               excepto a través de interfaces públicas / eventos)
```

### Tabla de dependencias

| Módulo | Puede importar API de | NO puede importar |
|---|---|---|
| `commons` | — (base) | Ningún módulo |
| `tenants` | `commons` | Todos los demás |
| `aplicaciones` | `commons`, `tenants` | `usuarios`, `identidad`, `asignaciones`, `politicas`, `autorizacion`, `auditoria` |
| `recursos` | `commons`, `tenants`, `aplicaciones` | `roles`, `perfiles`, `usuarios`, `identidad`, `asignaciones`, `politicas`, `autorizacion` |
| `roles` | `commons`, `tenants`, `recursos` | `perfiles`, `usuarios`, `identidad`, `asignaciones`, `politicas`, `autorizacion` |
| `perfiles` | `commons`, `roles` | `usuarios`, `identidad`, `asignaciones`, `politicas`, `autorizacion` |
| `usuarios` | `commons`, `tenants` | `identidad`, `asignaciones`, `politicas`, `autorizacion` |
| `identidad` | `commons`, `usuarios` | `asignaciones`, `politicas`, `autorizacion` |
| `asignaciones` | `commons`, `tenants`, `aplicaciones`, `roles`, `perfiles`, `usuarios` | `identidad` (solo lee `UsuarioId`), `politicas`, `autorizacion` |
| `politicas` | `commons`, `tenants`, `aplicaciones`, `recursos`, `roles`, `perfiles`, `asignaciones` (solo lectura API pública) | `identidad`, `autorizacion`, `auditoria` (no internals) |
| `autorizacion` | `commons`, `identidad`, `asignaciones`, `recursos`, `aplicaciones`, `tenants`, `politicas` (solo API pública) | Internals de cualquier módulo; no escribe en catálogos |
| `auditoria` (jar propio) | Solo Published Language / eventos (`commons`) | Internals de PDP; OPA; catálogos |

### Shared kernel `commons`

**No es un módulo Modulith** (no tiene `@ApplicationModule`). Es un paquete transversal con únicamente:

| Contenido | Tipo |
|---|---|
| `TenantId` | Value Object |
| `SujetoId` | Value Object |
| `AplicacionId` | Value Object |
| `RecursoId` | Value Object |
| `CorrelationId` | Value Object |
| `RequestId` | Value Object |
| `DecisionId` | Value Object |
| `ReasonCode` | Enum |
| `Decision` (`ALLOW`/`DENY`/`INDETERMINATE`) | Enum |

**Regla:** si un concepto de `commons` necesita lógica de negocio, sale de `commons` y entra al módulo que lo posee.

---

## Dependencias prohibidas (verificadas en CI)

Estas reglas deben convertirse en verificaciones de `ApplicationModuleTest` o equivalente en la Fase 0.G:

| Prohibición | Motivo |
|---|---|
| `autorizacion` → escribir en `tenants`, `aplicaciones`, `recursos`, `roles`, `perfiles`, `usuarios`, `asignaciones` | Solo lee para decidir; no administra catálogos |
| `politicas` → `autorizacion` | El catálogo no conoce el runtime de evaluación |
| `identidad` → `autorizacion` | Autenticación no autoriza |
| `roles` → `asignaciones` | El catálogo no gestiona vigencia |
| `recursos` → `asignaciones` | Un recurso no conoce quién lo tiene asignado |
| `auditoria` → internals del PDP | Consume solo eventos / published language |
| `PEP` → `SurrealDB` / `OPA` directamente | Solo PDP tiene esos adaptadores |
| Cualquier módulo de dominio → Spring / OPA / SurrealDB directamente | Hexagonal: solo adaptadores en capa infra |

---

## Estructura de paquetes del PDP (convención)

```
com.seguridad.pdp
├── commons/                       ← VOs/IDs transversales
├── tenants/
│   ├── domain/                    ← entidades, invariantes, puertos de repositorio
│   ├── application/               ← casos de uso
│   └── infrastructure/            ← adaptadores (SurrealDB, etc.)
├── aplicaciones/ …
├── recursos/ …
├── roles/ …
├── perfiles/ …
├── usuarios/ …
├── identidad/
│   ├── domain/
│   ├── application/
│   └── infrastructure/            ← adaptador IdP
├── asignaciones/ …
├── politicas/ …
└── autorizacion/
    ├── domain/                    ← SolicitudAcceso, DecisionAcceso, invariantes
    ├── application/               ← UC EvaluarAcceso
    └── infrastructure/
        ├── opa/                   ← adaptador OPA
        ├── idp/                   ← (delega a identidad)
        └── auditoria/             ← adaptador publicación EventoAcceso
```

Cada módulo respeta la arquitectura hexagonal del PDP (C4 L3): el dominio no importa Spring, OPA, SurrealDB ni ningún framework.

---

## Criterio de cierre (aprobado)

- [x] Decisión de 11 módulos tomada y justificada.
- [x] `commons` como paquete transversal (no módulo Modulith).
- [x] Tabla de dependencias permitidas/prohibidas.
- [x] Distribución por contenedor C4 alineada.
- [x] Convención de paquetes.
- [ ] Convertir en `ApplicationModuleTest` en el repositorio de implementación (Fase 0.G).
