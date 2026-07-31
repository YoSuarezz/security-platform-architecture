# Guía de presentación — Fases 0.A a 0.C (secciones 00–03)

| Campo | Valor |
|---|---|
| Audiencia | Profesor / evaluador de arquitectura |
| Alcance | Desde fuente de verdad hasta arquitectura de contenedores |
| Duración sugerida | **20–25 minutos** (+ 5 min preguntas) |
| Repositorio | `security-platform-architecture` |
| Objetivo de la charla | Demostrar que existe una **línea base arquitectónica sólida** antes de escribir código de producto |

---

## 1. Mensaje central (di lo primero)

> “Antes de implementar módulos, construimos la fundación: decisiones formales (ADR), dominio con 11 bounded contexts, y arquitectura C4 con PEP/PDP, OPA, SurrealDB y Keycloak. El repositorio documenta **qué se decidió, por qué, y cómo se ve**.”

Tres ideas que el profesor debe recordar:

1. **No empezamos a programar a ciegas** — hay Documento Base + ADRs + dominio + C4.
2. **Separación clara** — autenticar ≠ autorizar; catálogo de políticas ≠ motor OPA; PEP no contiene reglas.
3. **Listo para crecer** — Modulith (11 módulos), Cloud Enable, deny-by-default, auditoría obligatoria.

---

## 2. Agenda recomendada (tiempo)

| Min | Bloque | Prioridad | Qué abrir |
|---|---|---|---|
| 0–2 | Gancho + problema | **Alta** | Hablar (sin pantallas) |
| 2–4 | Estructura del repo | Alta | `README.md` |
| 4–7 | Fuente de verdad | Alta | `docs/00-source/` |
| 7–10 | Gobierno (ADR + glosario) | Alta | `docs/01-governance/adr/` |
| 10–16 | Dominio + diagramas | **Máxima** | Big Picture, BC, 1–2 modelos |
| 16–22 | Arquitectura C4 + deployment | **Máxima** | L1 → L2 → L3 PEP/PDP |
| 22–24 | Decisiones MVP (Keycloak, app académica, precedencia) | Alta | Mencionar ADR-012…014 |
| 24–25 | Cierre + siguientes fases | Media | “04 Data en adelante…” |

Si solo tienes **15 minutos**: salta la tabla completa de ADRs y los modelos 01–11 salvo Autorización; prioriza Big Picture + BC + C4 L2 + L3 PDP.

---

## 3. Qué priorizar vs qué pasar rápido

### Priorizar (mostrar pantalla + explicar)

| Artefacto | Por qué importa al profesor |
|---|---|
| Documento Base (visión + stack confirmado) | Demuestra origen de verdad |
| 2–3 ADR clave (WebFlux, OPA, Modulith o Keycloak) | Gobierno de decisiones |
| Big Picture + 11 Bounded Contexts | Dominio serio, no “solo capas técnicas” |
| Modelo de Autorización (10) | Corazón del producto |
| C4 L2 Contenedores | Arquitectura del sistema en una imagen |
| C4 L3 PDP (hexagonal) | Calidad de diseño interno |
| Deny-by-default + mapeo 401/403/503 | Rigor de seguridad |

### Pasar más rápido (mencionar, no detenerse)

| Artefacto | Cómo decirlo en 10 segundos |
|---|---|
| Glosario completo | “Hay lenguaje ubicuo oficial; evita ambigüedad.” |
| ADR-001…011 todos | “Catorce ADR Accepted; los del Documento Base ya están formalizados.” |
| Modelos Tenants…Perfiles | “Cada BC tiene modelo anémico; no los recorro todos.” |
| Invariantes / reglas / UC en detalle | “Existen UC-01…06 e invariantes trazables al Documento Base.” |
| Agregados / VOs candidatos | “Inventario Accepted como candidatos; se confirman en 04-Data.” |
| Compose YAML línea a línea | “Hay topología MVP documentada; solo muestro el diagrama.” |
| Carpetas 04–08 vacías/plantilla | “Siguiente fase; hoy cerramos 00–03.” |

---

## 4. Guion por bloque (qué decir / qué mostrar)

### Bloque A — Apertura (2 min)

**Decir:**

- “El proyecto es una **Plataforma Central de Seguridad**: otras apps delegan autorización.”
- “Problema: cada sistema reinventa roles, políticas y auditoría → inconsistencia.”
- “Solución: PEP aplica, PDP decide, OPA evalúa políticas, SurrealDB guarda el modelo, Keycloak autentica.”
- “Hoy presento la **Fase 0 documental** (0.A–0.C), no código de producto.”

**Mostrar:** nada, o logo/título del repo.

---

### Bloque B — Estructura del repositorio (2 min)

**Mostrar:** [`README.md`](../README.md)

**Decir:**

- “El repo está organizado por capas de madurez: `00` fuente → `01` gobierno → `02` dominio → `03` arquitectura → luego data, contratos, seguridad, ingeniería.”
- “`00-source` no se edita a la ligera: es la fuente de verdad.”
- “Dominio se revisa en orden `01` a `12` para no perderse.”

**Pasar rápido:** assets, templates, carpetas 04–08.

---

### Bloque C — 00 Source / Fase 0.A base (3 min)

**Mostrar:**

1. `docs/00-source/Documento_Base.md` — scroll al **Resumen ejecutivo** y tabla de decisiones confirmadas (Java, WebFlux, SurrealDB, OPA, hexagonal, Modulith…).
2. Mencionar `Fase_0_Linea_Base.md` — “plan de construcción de la línea base.”

**Decir:**

- “Todo lo demás se deriva de aquí. No inventamos stack en la presentación.”
- “Separación autenticación / autorización ya está en el documento base.”
- “Auditoría es obligatoria; deny-by-default es principio.”

**Frase fuerte:** “Si algo no está en el Documento Base o en un ADR, no es decisión tomada.”

---

### Bloque D — 01 Governance (3 min)

**Mostrar:**

1. `docs/01-governance/adr/README.md` — índice ADR-001…014.
2. Abrir **uno** de estos (elige según el foco del profesor):
   - **ADR-002** WebFlux — “I/O no bloqueante en PEP/PDP.”
   - **ADR-004** OPA/Rego — “políticas fuera del código Java.”
   - **ADR-009** Modulith — “11 módulos = 11 BC.”
   - **ADR-012** Precedencia — “deny explícito gana; sin política → deny.”
   - **ADR-013** Keycloak — “IdP del MVP, arquitectura agnóstica.”

**Decir:**

- “Cada decisión confirmada tiene ADR Accepted: contexto, decisión, alternativas, consecuencias.”
- “Glosario oficial: PEP, PDP, SolicitudAcceso, DecisionAcceso, tenant…”

**Pasar rápido:** no leer ADR-001…011 uno por uno.

---

### Bloque E — 02 Domain (6 min) — el bloque más visual

#### E1. App de validación (30 s)

**Mostrar:** `docs/02-domain/01-validation-application.md`

**Decir:** “Caso real del MVP: **Gestión Académica Universitaria**. Cada universidad = tenant. Recursos: estudiantes, cursos, matrículas… Acciones: crear, consultar, aprobar…”

#### E2. Big Picture — **PRIORIDAD ALTA**

**Mostrar:** `docs/02-domain/diagrams/01-big-picture.png`

**Qué es:** vista Event Storming / Domain Storytelling del dominio raíz **Gestión Seguridad**.

**Qué decir al señalarlo:**

- “Aquí se ve el dominio completo: actores, comandos y flujo hacia la decisión de acceso.”
- “No es un diagrama de clases: es el **mapa narrativo** del negocio de seguridad.”
- “De aquí salen los bounded contexts.”

**Cómo señalar (orden de dedo):**

1. Actores (usuario / admin).
2. Flujo hacia evaluación de acceso.
3. Resultado ALLOW/DENY y auditoría.

#### E3. Bounded Contexts — **PRIORIDAD MÁXIMA**

**Mostrar:** `docs/02-domain/diagrams/02-bounded-contexts.png` (+ `03-bounded-contexts.md`)

**Qué es:** partición del dominio en **11 contextos delimitados**.

**Qué decir (guion corto):**

> “Partimos el dominio en 11 capacidades de negocio, no en capas técnicas.
> Cadena de identidad: Usuarios → Identidad → Autorización.
> Cadena de control: Tenants → Apps → Recursos → Roles → Perfiles → Asignaciones → Políticas.
> Autorización evalúa; Políticas solo catalogan/versionan; OPA ejecuta Rego.
> Auditoría recibe evidencia, no decide.”

**Los 11 (recitar solo nombres, no detallar todos):**

1. Tenants · 2. Aplicaciones · 3. Recursos · 4. Roles · 5. Perfiles  
6. Usuarios · 7. Identidad y autenticación · 8. Asignaciones  
9. Políticas de acceso · 10. Autorización · 11. Auditoría de seguridad

**Punto a enfatizar:** “Usuarios ≠ Identidad ≠ Autorización. Esa separación evita el error clásico de mezclar login con permisos.”

#### E4. Un modelo anémico — Autorización (2 min)

**Mostrar:** `docs/02-domain/diagrams/models/10-autorizacion.png`

**Qué es:** modelo de dominio del BC Autorización (flujo de evaluación).

**Qué decir:**

- “Interceptor → SolicitudAcceso → Motor OPA → DecisionAcceso → EventoAcceso.”
- “Esto es lo que el PDP orquesta en runtime.”
- “Los otros 10 modelos existen (tenants, roles…); no los recorro para no saturar.”

**Opcional si sobra 1 min:** abrir `08-asignaciones.png` o `09-politicas-acceso.png` y decir “catálogo vs evaluación”.

#### E5. Event Storming + UC (1 min, rápido)

**Mostrar:** índice de `05-event-storming.md` o `08-use-cases.md` (solo título UC-01).

**Decir:** “UC-01 Evaluar acceso cubre ALLOW, DENY e INDETERMINATE. El PEP traduce a 401/403/503 según ADR-014.”

---

### Bloque F — 03 Architecture (6 min) — segundo bloque más fuerte

#### F1. C4 Nivel 1 — Contexto

**Mostrar:** `docs/03-architecture/c4/level1-context.png`

**Qué es:** el sistema como caja negra frente a personas y sistemas externos.

**Qué decir:**

- “Actores: usuario final, Administrador de Seguridad.”
- “Externos: apps protegidas, IdP (Keycloak), operación/auditoría.”
- “La plataforma es el centro; las apps no implementan autorización compleja.”

#### F2. C4 Nivel 2 — Contenedores — **PRIORIDAD MÁXIMA**

**Mostrar:** `docs/03-architecture/c4/level2-containers.png`

**Qué es:** descomposición en procesos desplegables.

**Qué decir señalando cada caja:**

| Caja | Frase |
|---|---|
| **PEP** | “Intercepta y aplica. No tiene políticas ni habla con OPA/DB.” |
| **PDP** | “Orquesta la decisión. Único que habla con OPA, SurrealDB y Keycloak.” |
| **OPA** | “Motor Rego. Recibe entrada normalizada, no el token crudo.” |
| **SurrealDB** | “Fuente de verdad del modelo de seguridad (documento + relaciones).” |
| **Auditoría** | “Servicio aparte: evidencia correlacionada.” |

**Frase fuerte:** “Si el PEP conociera OPA o SurrealDB, romperíamos el diseño.”

#### F3. C4 Nivel 3 — PEP

**Mostrar:** `docs/03-architecture/c4/level3-component-pep.png`

**Qué decir (45 s):**

- “Adaptador HTTP → Normalizador de SolicitudAcceso → Puerto de decisión → Cliente PDP → Aplicador (proxy o bloqueo).”
- “Delgado a propósito: enforcement, no inteligencia de negocio de seguridad.”

#### F4. C4 Nivel 3 — PDP — **PRIORIDAD ALTA**

**Mostrar:** `docs/03-architecture/c4/level3-component-pdp.png`

**Qué decir:**

- “Arquitectura hexagonal: caso de uso EvaluarAcceso en el centro.”
- “Puertos: identidad, repositorio, OPA, auditoría.”
- “Los 11 módulos Modulith viven aquí (salvo auditoría, que es contenedor propio).”

#### F5. Deployment + Modulith (1 min, rápido)

**Mostrar:** diagrama mermaid / sección topología en `deployment/deployment-mvp.md` **o** solo decirlo.

**Decir:**

- “MVP en Docker Compose: solo PEP expuesto al exterior; Keycloak self-hosted.”
- “Cloud Enable: mismos servicios → Kubernetes sin reescribir código.”
- “Dependency Map: 11 módulos, dependencias prohibidas verificables luego en CI.”

---

### Bloque G — Decisiones que cierran el MVP (2 min)

**Decir sin abrir todos los archivos:**

| Decisión | Frase |
|---|---|
| App de validación | Gestión Académica; universidad = tenant |
| Precedencia | Deny-by-default; deny explícito gana (ADR-012) |
| IdP | Keycloak; adaptador agnóstico (ADR-013) |
| HTTP | Token inválido 401 · DENY 403 · falla infra 503 (ADR-014) |
| Admin | Un solo Administrador de Seguridad en el MVP |

---

### Bloque H — Cierre (1 min)

**Decir:**

- “00–03 están Accepted: fuente, gobierno, dominio y arquitectura.”
- “Siguiente: 04 modelo de datos SurrealDB, 05 contratos/secuencias/Rego, 06 threat model, 07 ingeniería CI.”
- “La PoC WebFlux valida el stack reactivo como criterio técnico de Etapa 0.”
- “Preguntas.”

**No digas** que “falta todo” — di que **cerraste la fundación** y el desarrollo viene sobre rieles.

---

## 5. Explicación de cada diagrama que subiste

### Dominio

| Archivo | Nombre amigable | Qué comunica en la presentación |
|---|---|---|
| `01-big-picture.png` | Big Picture | Dominio completo de Gestión Seguridad; punto de partida del Event Storming |
| `02-bounded-contexts.png` | Contextos delimitados | 11 BC y dependencias de necesidad entre ellos |
| `models/01-tenants.png` | Modelo Tenants | Partición organizacional / aislamiento |
| `models/02-aplicaciones.png` | Modelo Aplicaciones | Apps que delegan seguridad |
| `models/03-recursos.png` | Modelo Recursos | Qué se protege (jerarquía recurso/acción) |
| `models/04-roles.png` | Modelo Roles | Catálogo RBAC |
| `models/05-perfiles.png` | Modelo Perfiles | Agrupación de roles |
| `models/06-usuarios.png` | Modelo Usuarios | Sujeto de dominio (no el IdP) |
| `models/07-identidad-autenticacion.png` | Modelo Identidad | Evidencia, sesión, claims normalizados |
| `models/08-asignaciones.png` | Modelo Asignaciones | Privilegios vigentes usuario–app–rol |
| `models/09-politicas-acceso.png` | Modelo Políticas | Metadatos/versión de políticas (no OPA) |
| `models/10-autorizacion.png` | Modelo Autorización | Flujo SolicitudAcceso → DecisionAcceso |
| `models/11-auditoria-seguridad.png` | Modelo Auditoría | Evidencia correlacionada |

**En la presentación oral:** Big Picture + BC + modelo 10 (Autorización). El resto: “están en el repo, uno por BC”.

### C4

| Archivo | Nivel | Qué comunica |
|---|---|---|
| `level1-context.png` | Contexto | Frontera del sistema y vecinos |
| `level2-containers.png` | Contenedores | PEP, PDP, OPA, SurrealDB, Auditoría y protocolos |
| `level3-component-pep.png` | Componentes PEP | Pipeline de enforcement delgado |
| `level3-component-pdp.png` | Componentes PDP | Hexagonal + puertos hacia IdP/DB/OPA/Auditoría |

**Orden de revelación C4:** siempre L1 → L2 → L3 (nunca al revés).

---

## 6. Preguntas probables del profesor (y respuestas cortas)

| Pregunta | Respuesta |
|---|---|
| ¿Por qué no microservicios ya? | Modulith primero: límites de módulo claros; extracción futura sin reescritura del dominio (ADR-009 / Documento Base). |
| ¿Por qué SurrealDB? | Documento + grafo para RBAC/REBAC; decisión confirmada ADR-003. |
| ¿Por qué OPA y no `if` en Java? | Políticas versionables, auditables y fuera del código de aplicación (ADR-004). |
| ¿Por qué WebFlux? | Concurrencia en el camino PEP→PDP→OPA sin hilos bloqueantes (ADR-002). |
| ¿El PEP puede decidir solo? | No. Solo enforce. Decide el PDP (+OPA). |
| ¿Qué pasa si cae OPA? | `INDETERMINATE` → **503**, nunca ALLOW (ADR-014). |
| ¿Multi-tenant? | Sí; universidad = tenant en el caso académico. |
| ¿Dónde está el código? | Esta fase es arquitectura. El código de producto viene después de 04–07. |
| ¿11 contextos no son demasiados? | Refinan el Documento Base; Modulith puede agrupar después con ADR, sin borrar el mapa de dominio. |

---

## 7. Checklist 5 minutos antes de presentar

- [ ] Repo abierto en Cursor/VS Code o GitHub en el navegador
- [ ] PNG de dominio y C4 abren (no stubs)
- [ ] Ruta mental: README → Documento Base → ADR índice → Big Picture → BC → Autorización → C4 L1–L3
- [ ] Tener a mano ADR-012, 013, 014 por si preguntan
- [ ] Reloj: si vas tarde, salta modelos 01–09 y ADR largos

---

## 8. Frases de cierre útiles

1. “La deuda técnica más cara es decidir el stack y el dominio *mientras* se programa; por eso cerramos 00–03 primero.”
2. “El diagrama de contextos es la autoridad del dominio; el Documento Base es la autoridad del stack y los principios.”
3. “PEP aplica, PDP decide, OPA evalúa, SurrealDB persiste, Keycloak autentica, Auditoría explica.”

---

## 9. Mapa rápido de archivos para la demo en vivo

```text
README.md
docs/00-source/Documento_Base.md
docs/01-governance/adr/README.md
docs/02-domain/diagrams/01-big-picture.png
docs/02-domain/diagrams/02-bounded-contexts.png
docs/02-domain/diagrams/models/10-autorizacion.png
docs/03-architecture/c4/level1-context.png
docs/03-architecture/c4/level2-containers.png
docs/03-architecture/c4/level3-component-pep.png
docs/03-architecture/c4/level3-component-pdp.png
docs/02-domain/01-validation-application.md   (si preguntan el caso MVP)
docs/03-architecture/deployment/deployment-mvp.md  (si preguntan Docker/K8s)
```

**Empieza la presentación por el README; termina en C4 L2 o L3 PDP.**
