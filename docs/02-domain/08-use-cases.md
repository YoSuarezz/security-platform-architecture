# Casos de uso del camino crítico

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Plantilla | [`docs/templates/use-case-template.md`](../templates/use-case-template.md) |
| Fuente | Documento Base §4.3, §18.1, §19; Fase 0 §0.H |
| App de validación | [`01-validation-application.md`](01-validation-application.md) |
| Precedencia / HTTP | [ADR-012](../01-governance/adr/ADR-012-policy-precedence.md), [ADR-014](../01-governance/adr/ADR-014-decision-http-mapping.md) |

## Índice

| ID | Nombre | Prioridad | BC principal |
|---|---|---|---|
| [UC-01](#uc-01-evaluar-acceso) | Evaluar acceso | P0 | Políticas (+ Identidad, Auditoría) |
| [UC-02](#uc-02-registrar-aplicación) | Registrar aplicación | P0 | Aplicaciones |
| [UC-03](#uc-03-asignar-rol) | Asignar rol | P0 | Asignaciones |
| [UC-04](#uc-04-publicar-política) | Publicar política | P0 | Políticas |
| [UC-05](#uc-05-catalogar-recurso) | Catalogar recurso | P0 | Recursos |
| [UC-06](#uc-06-definir-rol) | Definir rol | P0 | Roles y perfiles |

UC-05 y UC-06 son necesarios para que el esqueleto sea coherente aunque el Documento Base nombre explícitamente los cuatro primeros en §18.1.

---

## UC-01: Evaluar acceso

| Campo | Valor |
|---|---|
| ID | UC-01 |
| Actor primario | PEP (en nombre del Cliente) |
| Actores secundarios | IdP, OPA, SurrealDB, Auditoría |
| Bounded Context | Políticas (orquesta); Identidad; lectura de Tenants/Aplicaciones/Recursos/Asignaciones/Roles |
| Prioridad | P0 |

### Descripción

Dada una solicitud HTTP interceptada, validar/normalizar identidad, construir `SolicitudAcceso`, evaluar políticas y devolver `DecisionAcceso` para que el PEP la aplique, dejando auditoría correlacionada.

### Precondiciones

- Existe tenant conocido o regla explícita para el caso.
- La aplicación protegida está registrada y habilitada (o se deniega).
- Existe al menos una política publicada para el alcance MVP, **o** se aplica denegación por defecto.
- El PEP puede construir los campos mínimos de `SolicitudAcceso` (§14.1).

### Flujo principal (ALLOW)

1. PEP recibe la solicitud y genera/propaga `requestId` / `correlationId`.
2. Se invoca validación de identidad (evidencia presente y válida).
3. Identidad normaliza al sujeto interno + tenant + estado.
4. Políticas crea `SolicitudAcceso`.
5. Se resuelven desde el modelo: aplicación, recurso/acción, roles/perfiles vigentes, atributos mínimos.
6. Se construye entrada normalizada para OPA.
7. OPA evalúa Rego y retorna decisión consumible.
8. Políticas emite `DecisionAcceso` = `ALLOW` con `decisionId`, `reasonCode`, `policyReferences`, `correlationId`.
9. Auditoría registra evento correlacionado.
10. PEP aplica `ALLOW` y reenvía a la aplicación protegida.

### Flujo alterno — DENY por política / identidad / tenant

1. En los pasos 2–7 se detecta identidad inválida, tenant mismatch, asignación insuficiente, deny explícito o ausencia de política (ADR-012).
2. `DecisionAcceso` = `DENY` con `reasonCode` estable no sensible (`TOKEN_INVALID`, `POLICY_DENY`, `NO_APPLICABLE_POLICY`, `TENANT_MISMATCH`, …).
3. Auditoría registra el evento.
4. PEP bloquea; HTTP según ADR-014: token inválido → **401**; resto de DENY → **403**.

### Flujo alterno — INDETERMINATE

1. OPA, SurrealDB o IdP no responden / error no confiable / contexto imposible de completar.
2. `DecisionAcceso` = `INDETERMINATE`.
3. Auditoría registra con estado técnico distinguible.
4. PEP responde **503 Service Unavailable** (ADR-014). Nunca ALLOW. Sin caché como fuente de verdad.

### Postcondiciones

- Existe `DecisionAcceso` en uno de los tres estados.
- Existe (o quedó registrado el fallo de) evento de auditoría correlacionado.
- INV-SEC-01, INV-SEC-05, INV-POL-04, INV-AUD-01 satisfechas.

### Criterios de aceptación

- [ ] ALLOW solo cuando identidad válida + política/asignación lo permiten.
- [ ] DENY no alcanza la app protegida.
- [ ] INDETERMINATE nunca se expone como ALLOW.
- [ ] Evento de auditoría con IDs de correlación y sin secretos.

---

## UC-02: Registrar aplicación

| Campo | Valor |
|---|---|
| ID | UC-02 |
| Actor primario | Administrador de Seguridad |
| Bounded Context | Aplicaciones (+ Tenants) |
| Prioridad | P0 |

### Descripción

Registrar una aplicación protegida mínima asociada a un tenant, con estado habilitado, para que pueda participar en evaluaciones.

### Precondiciones

- El tenant existe y está activo.
- El actor tiene facultad administrativa (en MVP: asumida; separación de funciones pendiente §21.3).

### Flujo principal

1. Admin proporciona identificador/nombre, tenant y metadatos mínimos de integración.
2. El contexto Aplicaciones valida unicidad y asociación al tenant.
3. Se persiste `Aplicacion` en estado habilitado.
4. Se emite `AplicacionRegistrada`.

### Flujos alternos

- Tenant inexistente/inactivo → rechazo.
- Aplicación duplicada → rechazo o actualización controlada (definir en diseño detallado).

### Postcondiciones

- La aplicación puede referenciarse en recursos, asignaciones y evaluaciones.

### Criterios de aceptación

- [ ] Queda asociada a un tenant explícito.
- [ ] Aparece como habilitada en consultas de evaluación.

---

## UC-03: Asignar rol

| Campo | Valor |
|---|---|
| ID | UC-03 |
| Actor primario | Administrador de Seguridad |
| Bounded Context | Asignaciones |
| Prioridad | P0 |

### Descripción

Crear una asignación sujeto↔rol con alcance tenant/aplicación y vigencia, de modo que solo en estado `VIGENTE` participe en UC-01.

### Precondiciones

- Existen `Usuario` (sujeto), `Rol`, `Aplicacion` y `Tenant` válidos.
- El rol está acotado coherentemente (INV-DAT-01).

### Flujo principal

1. Admin selecciona sujeto, rol, aplicación, tenant y ventana de vigencia.
2. Asignaciones valida coherencia de alcance (no cruce indebido de tenant).
3. Se crea asignación `VIGENTE` (o `PENDIENTE` si se exige activación — MVP puede crear directa a `VIGENTE`).
4. Evento `AsignacionCreada` / `AsignacionVigente`.

### Flujos alternos

- Incoherencia de tenant → rechazo (INV-TEN-02, INV-ASN-02).
- Rol/sujeto inexistente → rechazo.

### Postcondiciones

- UC-01 puede incluir el rol en el contexto si la asignación está vigente.

### Criterios de aceptación

- [ ] Asignación vencida/revocada no aparece en evaluación.
- [ ] No se puede asignar rol de otro tenant sin relación explícita.

---

## UC-04: Publicar política

| Campo | Valor |
|---|---|
| ID | UC-04 |
| Actor primario | Administrador de Seguridad |
| Bounded Context | Políticas |
| Prioridad | P0 |

### Descripción

Publicar una versión de política Rego (RBAC simple en MVP) con metadatos obligatorios para que OPA/PDP la evalúen.

### Precondiciones

- Existe artefacto Rego válido para el alcance (tenant/aplicación).
- Metadatos mínimos: versión, responsable, vigencia, estado.

### Flujo principal

1. Admin (o pipeline de políticas) solicita publicación.
2. Se validan metadatos (INV-POL-01).
3. Estado pasa a `PUBLICADA`; versión queda trazable.
4. Evento `PoliticaPublicada`.

### Flujos alternos

- Metadatos incompletos → rechazo.
- Publicación que rompe pruebas Rego → rechazo (cuando exista pipeline de políticas).

### Postcondiciones

- UC-01 puede referenciar `policyReferences` con versión.

### Criterios de aceptación

- [ ] No existe política “activa” sin versión/responsable/vigencia.
- [ ] La versión publicada es la que aparece en auditoría.

---

## UC-05: Catalogar recurso

| Campo | Valor |
|---|---|
| ID | UC-05 |
| Actor primario | Administrador de Seguridad |
| Bounded Context | Recursos |
| Prioridad | P0 |

### Descripción

Registrar un recurso protegible y la acción asociada (p. ej. operación/ruta lógica) ligados a una aplicación.

### Precondiciones

- La aplicación está registrada (UC-02).

### Flujo principal

1. Admin define tipo/identificador de recurso, acción y aplicación.
2. Recursos valida y persiste.
3. Evento `RecursoCatalogado`.

### Postcondiciones

- UC-01 puede resolver recurso/acción; roles pueden autorizarlo (UC-06).

### Criterios de aceptación

- [ ] El recurso está ligado a una aplicación.
- [ ] La acción es explícita (no se infiere solo del verbo HTTP sin catálogo).

---

## UC-06: Definir rol

| Campo | Valor |
|---|---|
| ID | UC-06 |
| Actor primario | Administrador de Seguridad |
| Bounded Context | Roles y perfiles |
| Prioridad | P0 |

### Descripción

Crear un rol RBAC y su autorización sobre un recurso/acción del catálogo.

### Precondiciones

- Recurso/acción existen (UC-05).
- Alcance tenant/aplicación o global inequívoco (INV-DAT-01).

### Flujo principal

1. Admin define nombre de rol y permisos sobre recurso/acción.
2. Se persiste `Rol` + vínculo `RolRecurso`.
3. Evento `RolDefinido`.

### Postcondiciones

- UC-03 puede asignar el rol; UC-01 puede usarlo en contexto RBAC.

---

## Trazabilidad a criterios de éxito (§4.3)

| Criterio de éxito | UC |
|---|---|
| 1. App registrada/configurada | UC-02 |
| 2. Solicitud llega al PEP | UC-01 (borde) |
| 3. Identidad inválida rechazada | UC-01 |
| 4. Contexto sujeto/app/recurso/acción | UC-01, UC-05 |
| 5. OPA evalúa Rego | UC-01, UC-04 |
| 6. PEP enforce ALLOW/DENY | UC-01 |
| 7. Auditoría correlacionada | UC-01 |
| 8. Explicación política/versión (sin secretos) | UC-01, UC-04 |
| 9. Línea base ingeniería | Fuera de dominio (0.G) |

## Criterio de cierre Fase 0.B (casos de uso)

- [x] Flujos ALLOW/DENY/INDETERMINATE en UC-01.
- [x] UC de preparación mínimos para el esqueleto.
- [x] App de validación cerrada (PD-01) — [`01-validation-application.md`](01-validation-application.md).

---

## Navegación

[**← 07 Reglas de negocio**](07-business-rules.md) · [**09 Eventos de dominio →**](09-domain-events.md)
