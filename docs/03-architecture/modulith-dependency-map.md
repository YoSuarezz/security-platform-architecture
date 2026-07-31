# Mapa de módulos y Dependency Map — Spring Modulith

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.1) |
| Fase | 0.C |
| ADR | [ADR-009](../01-governance/adr/ADR-009-spring-modulith.md) — Modulith en **todos** los contenedores Java |
| Alcance | **PEP** + **PDP** + **Auditoría** (cada uno es una aplicación Spring Boot + Modulith) |
| Dominio | 11 Bounded Contexts → módulos de negocio principalmente en el **PDP**; Auditoría es BC + contenedor |

---

## Principio

Spring Modulith **no es exclusivo del PDP**. Todo jar Java de la plataforma organiza su código en módulos verificables:

| Contenedor | Rol Modulith |
|---|---|
| PDP | Módulos alineados a BC de catálogo + autorización |
| PEP | Módulos de capacidad de enforcement (sin dominio de catálogo) |
| Auditoría | Módulos del BC Auditoría (ingesta, consulta, retención) |

OPA, SurrealDB y Keycloak son infra externa (no Modulith).

---

## 1. Contenedor PDP

### Módulos (uno por BC de negocio, excepto auditoría)

| Módulo | BC | Puede depender de (API pública) |
|---|---|---|
| `commons` | Shared kernel (no `@ApplicationModule`) | — |
| `tenants` | Tenants | `commons` |
| `aplicaciones` | Aplicaciones | `commons`, `tenants` |
| `recursos` | Recursos | `commons`, `tenants`, `aplicaciones` |
| `roles` | Roles | `commons`, `tenants`, `recursos` |
| `perfiles` | Perfiles | `commons`, `roles` |
| `usuarios` | Usuarios | `commons`, `tenants` |
| `identidad` | Identidad y autenticación | `commons`, `usuarios` |
| `asignaciones` | Asignaciones | `commons`, `tenants`, `aplicaciones`, `roles`, `perfiles`, `usuarios` |
| `politicas` | Políticas de acceso | `commons`, `tenants`, `aplicaciones`, `recursos`, `roles`, `perfiles`, `asignaciones` (lectura) |
| `autorizacion` | Autorización | `commons`, `identidad`, `asignaciones`, `recursos`, `aplicaciones`, `tenants`, `politicas` (lectura) |

### Por qué 10 módulos de dominio en el PDP (no 11)

El BC **Auditoría de seguridad** se materializa como **contenedor propio** (también Modulith). Así se respeta C4 L2 y se evita que el PDP acumule retención/consulta de evidencia.

### Shared kernel `commons` (PDP)

Solo VOs/IDs/enums: `TenantId`, `SujetoId`, `AplicacionId`, `RecursoId`, `CorrelationId`, `RequestId`, `DecisionId`, `ReasonCode`, `Decision`.

Si necesita lógica de negocio → sale de `commons` al módulo dueño.

### Dependencias prohibidas (PDP)

| Prohibición | Motivo |
|---|---|
| `autorizacion` escribe catálogos | Solo lee para decidir |
| `politicas` → `autorizacion` | Catálogo ≠ runtime |
| `identidad` → `autorizacion` | Autenticar ≠ autorizar |
| `roles` → `asignaciones` | Catálogo ≠ vigencia |
| Módulo de dominio → Spring/OPA/SurrealDB directo | Hexagonal: solo adaptadores |

### Paquetes (PDP)

```text
com.seguridad.pdp
├── commons/
├── tenants/{domain,application,infrastructure}
├── … (un árbol por módulo)
└── autorizacion/
    ├── domain/
    ├── application/          ← UC EvaluarAcceso
    └── infrastructure/{opa,surreal,auditoria-client,…}
```

---

## 2. Contenedor PEP (Modulith)

El PEP **no** posee BC de catálogo. Usa Modulith para no mezclar normalización, cliente PDP y enforcement.

| Módulo | Responsabilidad | Puede depender de |
|---|---|---|
| `commons` | Contratos `SolicitudAcceso` / `DecisionAcceso` (DTOs/VOs de borde) | — |
| `ingress` | Adaptador HTTP de entrada, rate-limit, extracción de metadatos confiables | `commons` |
| `normalization` | Construcción de `SolicitudAcceso` canónica | `commons`, `ingress` (API) |
| `decision-client` | Puerto + adaptador WebClient hacia PDP | `commons` |
| `enforcement` | Aplicar ALLOW (proxy) / DENY / INDETERMINATE → HTTP (ADR-014) | `commons`, `decision-client` |

### Dependencias prohibidas (PEP)

| Prohibición | Motivo |
|---|---|
| Cualquier módulo → SurrealDB / OPA / Keycloak Admin | Solo el PDP integra esas dependencias |
| `enforcement` → lógica de políticas Rego | El PEP no decide |
| `normalization` → `enforcement` salteándose `decision-client` | Toda decisión pasa por el PDP |

### Paquetes (PEP)

```text
com.seguridad.pep
├── commons/
├── ingress/{domain,application,infrastructure}
├── normalization/{domain,application,infrastructure}
├── decision-client/{application,infrastructure}   ← WebClient PDP
└── enforcement/{domain,application,infrastructure}
```

---

## 3. Contenedor Auditoría (Modulith)

| Módulo | Responsabilidad | Puede depender de |
|---|---|---|
| `commons` | Contratos de `EventoAcceso` / published language | — |
| `ingestion` | Recibir y validar eventos del PDP | `commons` |
| `query` | Consultas para Operación / admin | `commons`, `ingestion` (API lectura) |
| `retention` | Políticas de retención/expurgo (cuando PD-10 cierre) | `commons` |

### Dependencias prohibidas (Auditoría)

| Prohibición | Motivo |
|---|---|
| → internals del PDP / OPA / catálogos | Solo published language |
| `query` muta evidencia histórica | Append-oriented; corrección vía nuevos eventos si aplica |

### Paquetes (Auditoría)

```text
com.seguridad.auditoria
├── commons/
├── ingestion/{domain,application,infrastructure}
├── query/{domain,application,infrastructure}
└── retention/{domain,application,infrastructure}
```

---

## 4. Vista C4 ↔ Modulith

```text
+---------------- PEP (Boot + WebFlux + Modulith) ----------------+
|  ingress · normalization · decision-client · enforcement        |
+-------------------------------+---------------------------------+
                                | SolicitudAcceso / DecisionAcceso
                                v
+---------------- PDP (Boot + WebFlux + Modulith) ----------------+
|  tenants … autorizacion (+ commons)                             |
|  adapters: OPA · SurrealDB · Keycloak · cliente Auditoria       |
+---------------+-------------------+-----------------------------+
                |                   |
         OPA / SurrealDB / KC       | EventoAcceso
                                    v
+------------- Auditoria (Boot + WebFlux + Modulith) -------------+
|  ingestion · query · retention                                  |
+-----------------------------------------------------------------+
```

---

## 5. Criterio de cierre

- [x] Modulith en PEP, PDP y Auditoría (ADR-009 v actualizada).
- [x] Módulos PDP alineados a BC (auditoría como contenedor).
- [x] Módulos PEP por capacidad de enforcement.
- [x] Módulos Auditoría por capacidad del BC.
- [x] Dependencias prohibidas documentadas por contenedor.
- [ ] `ApplicationModuleTest` por contenedor en CI (Fase 0.G / ADR-011).
