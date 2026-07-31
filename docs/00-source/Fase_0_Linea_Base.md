# Fase 0 - Línea Base Arquitectónica (Plataforma Central de Seguridad)

> **Rol de este documento:** plan de construcción de la Fase 0 (Etapa 0 de la sección 19 del documento base), producido desde el rol de Chief Software Architect.
> **Fuente de verdad:** `DOCUMENTO_BASE_PLATAFORMA_SEGURIDAD_ACTUALIZADO.md`. Este documento no cuestiona ni redefine objetivos, alcance, arquitectura, restricciones, decisiones técnicas, principios de ingeniería ni lineamientos de desarrollo. Los da por establecidos y construye lo que debe existir **entre** ese documento y la primera línea de código.
> **Alcance de este documento:** exclusivamente actividades previas al desarrollo. No contiene código, estructura de carpetas, clases, entidades Java, repositorios ni definiciones de API concretas.
> **Criterio de decisión para cada artefacto:** Harness Engineering (sección 8.1) — mantenibilidad, escalabilidad, verificabilidad, automatización, evolución — aplicado con el filtro adicional de **reducción de riesgo real**, no de completitud documental. Un artefacto solo entra en la línea base si evita un reproceso identificable o una decisión que sería costosa de revertir después.

---

## 0. Cómo leer este documento

Cada artefacto se evalúa con las mismas siete preguntas que pediste: necesidad, justificación, riesgo que evita, momento, prioridad, dependencias y nivel de detalle. Para no producir "documentación por documentación", los artefactos se agrupan cuando responden la misma pregunta arquitectónica, y se eliminan los que no aportan valor verificable en este proyecto. Todas las fusiones y eliminaciones se justifican explícitamente en la sección 9.

Prioridades usadas: **P0** (bloquea el inicio de la Etapa 1, sin excepción), **P1** (necesario antes de que exista tráfico real entre PEP-PDP-OPA-SurrealDB), **P2** (necesario antes de ABAC/REBAC o de producción, no antes del MVP).

---

## 1. Orden óptimo y por qué

El orden no es arbitrario: cada fase existe porque la siguiente no puede tomarse decisiones informadas sin ella.

```
0.A Gobierno y alcance del propio proceso de arquitectura
      |
      v
0.B Dominio (lenguaje ubicuo, contextos, casos de uso, reglas)
      |  <- sin esto, cualquier diagrama de arquitectura describe cajas sin semántica de negocio
      v
0.C Arquitectura de sistema (C4, despliegue, integraciones)
      |  <- sin el dominio (0.B) no se sabe qué es un "contenedor" ni un "componente" con sentido
      v
0.D Datos (modelo conceptual -> lógico -> SurrealDB)
      |  <- depende de 0.B (entidades/relaciones) y 0.C (qué componente posee qué dato)
      v
0.E Contratos y comportamiento (secuencia, estados, eventos, precedencia de políticas)
      |  <- depende de 0.C (componentes) y 0.D (qué se persiste/consulta)
      v
0.F Seguridad del propio componente (threat modeling, OWASP, auditoría)
      |  <- solo es posible modelar amenazas sobre una arquitectura y flujos ya definidos (0.C-0.E)
      v
0.G Ingeniería de construcción (Git, CI/CD, calidad, pruebas, ADR)
      |  <- convierte todo lo anterior en algo ejecutable y verificable
      v
0.H Primeras historias de "esqueleto" + Baseline v1.0
```

Justificación de las dependencias críticas:

- **Dominio antes que diagramas de sistema.** El documento base ya advierte (sección 22, punto 4) que antes de diseñar el esquema físico de SurrealDB hay que derivar casos de autorización concretos. Lo mismo aplica a C4: un Container Diagram dibujado antes de fijar el lenguaje ubicuo termina describiendo tecnología (PEP, PDP, OPA) sin conectar con las invariantes de negocio de la sección 7, y se rehace en cuanto aparece el primer caso de uso real.
- **Arquitectura de sistema antes que datos.** El modelo de datos (sección 12) depende de qué componente resuelve qué pregunta (sección 12.3). Sin el Container/Component Diagram no se sabe si una consulta de relaciones ocurre en el PDP, en un adaptador o en SurrealDB directamente.
- **Datos y contratos antes que threat modeling.** No se puede modelar amenazas de forma útil sobre un sistema abstracto; STRIDE/threat modeling necesita flujos de datos concretos (sección 13, 14 y 16 ya materializadas).
- **Ingeniería de construcción al final, pero no después.** Git, CI/CD y estrategia de pruebas no dependen del contenido de negocio, así que podrían ir en paralelo desde el día uno; se colocan como penúltimo bloque solo porque su *criterio de aceptación* (qué se prueba, qué gates de calidad aplican a qué módulo) sí depende de que existan los módulos de Spring Modulith (0.C) y las invariantes de dominio (0.B). En la práctica, el repositorio, la rama protegida y el pipeline vacío deben crearse en paralelo con 0.A, no esperar a 0.G.

---

## 2. Fases de la Línea Base

### Fase 0.A — Gobierno del proceso de arquitectura

**Objetivo.** Establecer cómo se gobiernan las decisiones de aquí en adelante, para que ninguna decisión de las fases siguientes se pierda, se contradiga tácitamente o dependa de la memoria del autor.

**Entregables.**
- Catálogo de **ADR (Architecture Decision Records)**, inicializado con las decisiones ya confirmadas en la sección 6 del documento base transcritas como ADRs formales (una decisión = un ADR, con estado `Aceptada`).
- Plantilla única de ADR (contexto, decisión, alternativas consideradas, consecuencias, estado).
- **Glosario / lenguaje ubicuo** como documento vivo separado (ver fusión en sección 9): arranca con el glosario ya existente (sección 23 del documento base) y se congela como referencia obligatoria para todo nombre usado en diagramas, políticas Rego, eventos y campos de datos.

**Criterios de aceptación.**
- Cada decisión confirmada de la sección 6 tiene un ADR con identificador único y es citable por número desde cualquier otro documento.
- No existe ninguna decisión "confirmada" en el documento base sin ADR correspondiente.

**Dependencias.** Ninguna; es el punto de entrada.

**Decisiones que deben quedar cerradas al salir de esta fase.**
- Formato y numeración de ADR.
- Autoridad de aprobación de un ADR (quién puede pasar un pendiente a confirmado).

**Riesgos que evita.** Que una decisión ya tomada (p. ej. SurrealDB sobre SQL, sección 6.1) se "reabra" implícitamente meses después por desconocimiento, o que dos documentos posteriores describan la misma decisión de forma distinta.

---

### Fase 0.B — Dominio

**Objetivo.** Convertir los contextos ya identificados (sección 11 del documento base) y las preguntas abiertas (sección 21) en un modelo de dominio explícito, con lenguaje compartido, antes de dibujar un solo componente técnico.

**Entregables.**
- **Event Storming** (o **Domain Storytelling**, no ambos — ver sección 9) del flujo crítico: registro de aplicación, resolución de identidad, evaluación de acceso, decisión, enforcement, auditoría. Debe cubrir explícitamente los tres estados `ALLOW`/`DENY`/`INDETERMINATE` (sección 9.2).
- **Bounded Contexts** formalizados a partir de la sección 11: Tenants, Aplicaciones, Recursos, Identidad, Roles/Perfiles, Asignaciones, Políticas, Auditoría. Estos son, por diseño, los mismos límites que luego impone Spring Modulith (sección 8.8) — no se definen dos veces.
- **Context Map** entre esos ocho contextos, indicando tipo de relación (cliente-proveedor, conformista, kernel compartido) y quién es dueño de qué entidad cuando dos contextos la mencionan (p. ej. `Rol` en Roles/Perfiles vs. su uso en Políticas).
- **Casos de uso** del camino crítico de la Etapa 1 (sección 19): `EvaluarAcceso`, `RegistrarAplicacion`, `AsignarRol`, `PublicarPolitica`, ya nombrados en la sección 18.1. Cada uno con precondición, flujo principal, flujos alternos (`DENY`, `INDETERMINATE`) y postcondición de auditoría.
- **Reglas de negocio / invariantes** consolidadas a partir de las secciones 7, 12.4 y 13.5 en una lista única, trazable, con identificador (p. ej. `INV-01: denegación por defecto`).
- **Respuestas registradas** a las preguntas 1, 3, 5, 6, 7 de la sección 21 (las que son de dominio; las de infraestructura/latencia se resuelven en 0.C/0.F).

**Lo que se descarta deliberadamente en esta fase:** agregados, entidades y value objects en sentido de diseño táctico DDD detallado. El documento base ya advierte que el modelo físico, atributos y cardinalidades se definen desde casos de uso reales; formalizar agregados/VOs antes de tener el primer caso de uso de referencia (pregunta 1 de la sección 21) es trabajo que se rehace. Se posponen a diseño detallado, ya fuera de esta línea base.

**Criterios de aceptación.**
- Los ocho contextos y sus relaciones están dibujados y aprobados; ningún término del glosario (0.A) contradice un término usado en Event Storming.
- Existe al menos una respuesta explícita y aprobada a "¿cuál es el primer dominio/aplicación real de validación?" (pregunta 1, sección 21) — sin esto, todo lo posterior (C4 de contexto, modelo de datos, threat model) trabaja sobre un caso hipotético y se rehace.
- Cada caso de uso del camino crítico tiene sus tres desenlaces (`ALLOW`/`DENY`/`INDETERMINATE`) documentados.

**Dependencias.** Fase 0.A (glosario base, ADRs de las decisiones que acotan el dominio, p. ej. autenticación delegada).

**Riesgos que evita.** Modelo de grafo sin casos de uso concretos (riesgo ya identificado en la sección 20 del documento base); Context Diagram y Component Diagram que se dibujan dos veces porque la primera versión no reflejaba los contextos reales.

---

### Fase 0.C — Arquitectura de sistema (C4 + despliegue + integraciones)

**Objetivo.** Representar la arquitectura ya decidida (hexagonal, Spring Modulith, reactiva, Cloud Native/Enable) en los niveles de abstracción que un equipo necesita para no reinterpretarla cada uno a su manera.

**Entregables (con fusión aplicada, ver sección 9):**
- **C4 Nivel 1 — Context Diagram.** Sistema "Plataforma Central de Seguridad" y sus actores/sistemas externos: aplicaciones protegidas heterogéneas (Django, PHP, .NET, Java), proveedor(es) de identidad, administrador de seguridad. Nivel de detalle: una página, sin tecnología interna.
- **C4 Nivel 2 — Container Diagram.** PEP, PDP, OPA, SurrealDB, módulo de Auditoría, futura Administración — alineado 1:1 con la tabla de componentes de la sección 9.1 y el flujo de la sección 9.2. Incluye el límite reactivo (WebFlux) como anotación explícita en cada contenedor que maneja HTTP o I/O remoto (sección 8.4).
- **C4 Nivel 3 — Component Diagram**, limitado a los dos contenedores críticos del MVP (PEP y PDP), mostrando los puertos y adaptadores de la sección 18.1 (entrada/adapter, aplicación, dominio, políticas, infraestructura). No se hace Nivel 3 de contenedores fuera del camino crítico (Administración, por ejemplo) porque no existen en la Etapa 1.
- **Deployment Diagram**, coherente con Cloud Enable (sección 8.3): contenedores Docker para la demo/MVP, con anotación explícita de qué restricciones de hoy (sin host fijo, sin filesystem local con estado) preparan la migración a Kubernetes, sin comprometerse hoy a un entorno productivo (pregunta pendiente en sección 6).
- **Mapa de módulos y Dependency Map de Spring Modulith**, derivado directamente de los Bounded Contexts de 0.B (no redefinidos, solo proyectados a módulos técnicos), mostrando qué módulo puede depender de cuál y marcando explícitamente las dependencias prohibidas (p. ej. Auditoría no puede depender de Políticas).
- **Contexto de integraciones**: una tabla (no diagrama aparte) de cada sistema externo (IdP, aplicaciones de referencia, OPA como proceso externo) con protocolo, dirección, dato mínimo intercambiado y modo de integración a evaluar (tabla de la sección 14.3), dejando registrada la respuesta a la pregunta 10 de la sección 21.

**Lo que se fusiona o descarta:** Activity Diagrams y State Diagrams genéricos se descartan como artefactos independientes (ver sección 9); su contenido útil se absorbe en 0.E (Sequence Diagrams + máquina de estados de `DecisionAcceso`/asignaciones). "Event Flow" como documento aparte se fusiona con el Container Diagram (ya es el flujo de la sección 9.2 dibujado) y con el catálogo de eventos de auditoría de 0.E.

**Criterios de aceptación.**
- Todo elemento del Container Diagram aparece en la tabla de componentes de la sección 9.1 del documento base, y viceversa; no hay cajas nuevas no justificadas.
- El Component Diagram del PDP muestra explícitamente el puerto de evaluación de políticas y el puerto de repositorio de seguridad como los únicos puntos de dependencia hacia OPA y SurrealDB (verificable contra sección 8.7).
- El Dependency Map de módulos es el artefacto contra el que se configuran las reglas de verificación de Spring Modulith en CI (0.G); si un módulo no está en el mapa, no puede existir en el código.

**Dependencias.** Fase 0.B (contextos = módulos).

**Riesgos que evita.** Adoptar Spring Modulith solo nominalmente sin disciplina real de límites (riesgo explícito en sección 20); PEP proxy que termina absorbiendo lógica de negocio compleja porque nadie dibujó el límite de responsabilidad (sección 9.1, columna "no debe hacer").

---

### Fase 0.D — Datos

**Objetivo.** Definir cómo se modela el dominio de seguridad en SurrealDB, con la mezcla de documento y grafo justificada por consulta real, no por preferencia estética (principio explícito de la sección 12.1).

**Entregables.**
- **Modelo conceptual**: entidades de la sección 11 con sus relaciones de la sección 12.2, sin tipos de dato ni claves — esencialmente una versión formalizada y aprobada del diagrama ya presente en el documento base.
- **Modelo lógico**: para cada entidad, decisión explícita de "registro/documento" vs. "relación de grafo", justificada contra las consultas de la sección 12.3 (una fila por consulta, indicando qué relación de grafo la resuelve). Este es el artefacto que reemplaza a un ER Diagram clásico (ver fusión, sección 9) porque SurrealDB no es relacional puro y un ER tradicional forzaría una notación que no representa las aristas de grafo.
- **Catálogo de entidades**: nombre, contexto dueño (de 0.B), atributos mínimos conocidos, y explícitamente qué invariante de la sección 12.4 protege.
- **Estrategia de persistencia**: transacciones e invariantes críticas (evitar asignaciones cruzadas entre tenants), estrategia de índices para las consultas de la sección 12.3, y una nota explícita de que no se modela físicamente ninguna relación sin una pregunta de autorización que la origine (mitigación del riesgo "modelo de grafo sin casos de uso concretos", sección 20).
- **Registro de resultado de la prueba técnica de SurrealDB** (ya exigida en la Etapa 0 del documento base, sección 19): relaciones, consultas, transacciones, índices y aislamiento por tenant, ejecutada sobre el modelo lógico de este mismo entregable, no sobre un ejemplo genérico.

**Modelo físico:** se pospone fuera de la línea base. Fijar nombres físicos de tablas/aristas antes de escribir la primera migración es trabajo que, según el propio documento base (sección 12.2: "la notación anterior es conceptual"), se espera que cambie con el primer caso de uso implementado. La línea base fija el modelo lógico y la estrategia; el físico se define al implementar `EvaluarAcceso`.

**Criterios de aceptación.**
- Cada relación de grafo del modelo lógico está asociada a al menos una consulta de la sección 12.3 y a al menos una invariante de la sección 12.4.
- La prueba técnica de SurrealDB se ejecutó contra el modelo lógico aprobado y sus resultados (latencia de consultas de grafo, comportamiento transaccional) están documentados y pueden invalidar decisiones de modelado si aparecen problemas — es una prueba, no una demostración de que "funciona".

**Dependencias.** Fase 0.B (entidades y casos de uso), Fase 0.C (qué componente consulta qué).

**Riesgos que evita.** El riesgo explícito de la sección 20 ("modelo de grafo sin casos de uso concretos") y el de subestimar necesidad de integridad/concurrencia/migración solo porque existe capacidad de grafo (advertencia explícita de la sección 6.1).

---

### Fase 0.E — Contratos y comportamiento

**Objetivo.** Fijar cómo se comportan los componentes ya dibujados (0.C) sobre los datos ya modelados (0.D), de forma que la Etapa 1 no tenga que inventar semántica de contrato mientras programa.

**Entregables.**
- **Sequence Diagrams** del camino crítico: flujo `ALLOW`, flujo `DENY` por política, flujo `INDETERMINATE` por caída de OPA/SurrealDB/IdP, y flujo de auditoría asociado a cada uno. Se basan directamente en el flujo textual de la sección 9.2, formalizado como diagrama.
- **State Diagram único**, aplicado a las dos máquinas de estado que realmente lo necesitan: ciclo de vida de una `DecisionAcceso`/`SolicitudAcceso` (creada → evaluada → aplicada → auditada) y ciclo de vida de una asignación (`UsuarioAplicacionRol`: pendiente → vigente → vencida/revocada, sección 11.6). No se producen máquinas de estado para entidades sin transiciones no triviales (p. ej. `Tenant` no las necesita en esta fase).
- **Contrato de `SolicitudAcceso`** y **contrato de `DecisionAcceso`**, formalizando las tablas de las secciones 14.1 y 14.2 como especificación (no como OpenAPI todavía — eso es diseño detallado, fuera de la línea base, según la restricción "no propongas APIs").
- **Decisión cerrada de precedencia de políticas** (sección 13.5): esta es la única decisión pendiente crítica que el propio documento base marca como bloqueante. Debe resolverse explícitamente aquí, como ADR (0.A), antes de escribir la primera política Rego. La regla provisional del documento (denegación por defecto + denegación explícita con prioridad) se ratifica o se reemplaza formalmente, no se hereda por omisión.
- **Ejemplo de entrada/salida a OPA** (formalización del ejemplo de la sección 13.4) con clasificación de qué atributos pueden viajar a OPA y cuáles no (mínimo privilegio de datos, sección 7.9), y **registro de resultado de la prueba técnica OPA/Rego** de la Etapa 0.
- **Catálogo mínimo de eventos de auditoría**, formalizando la lista de la sección 16.1 con tipo, campos y correlación — este catálogo es lo único que sobrevive de un "Event Flow" separado (ver fusión, sección 9).

**Criterios de aceptación.**
- Existe un ADR de precedencia de políticas con estado `Aceptada`, no `Propuesta`.
- Cada Sequence Diagram tiene su evento de auditoría correspondiente en el catálogo; no hay un flujo de decisión sin evidencia asociada (principio de trazabilidad por diseño, sección 7.6).
- El contrato de `DecisionAcceso` incluye los tres estados y todos los campos mínimos de la tabla de la sección 14.2.

**Dependencias.** Fase 0.C (componentes), Fase 0.D (qué se persiste y consulta).

**Riesgos que evita.** Políticas ambiguas o contradictorias (riesgo explícito, sección 20); un cambio incompatible en el contrato de `DecisionAcceso` que se descubre en producción en vez de en el versionamiento (razón explícita de la estrategia Git, sección 8.11).

---

### Fase 0.F — Seguridad del propio componente

**Objetivo.** Aplicar a la plataforma de seguridad el mismo rigor que ella exige a las aplicaciones que protege, antes de que exista una sola línea de enforcement real.

**Entregables.**
- **Threat Model** (STRIDE o equivalente) sobre el flujo Cliente → PEP → PDP → OPA/SurrealDB → Auditoría ya diagramado en 0.C/0.E, cubriendo explícitamente: *header spoofing* y reenvío de cabeceras no confiables (riesgo ya listado en sección 20 y sección 17), suplantación de contexto de tenant, manipulación de atributos ABAC no validados, fuga de datos sensibles hacia OPA o logs.
- **Checklist OWASP** relevante (API Security Top 10, dado que el producto es fundamentalmente una API de decisión de acceso), mapeado contra los requisitos no funcionales de seguridad ya definidos en la sección 17 — no se redefinen requisitos, se verifica cobertura.
- **Especificación de autenticación e identidad** como documento de diseño (no código): formalización del flujo conceptual de la sección 15, con la respuesta cerrada a la pregunta 2 de la sección 21 (qué proveedor(es) de identidad entran en el MVP).
- **Especificación de auditoría**: retención, clasificación de datos, cifrado, control de acceso a la auditoría y postura sobre `HashBlockchain` (que el documento base ya marca como investigación, no requisito — este documento no la convierte en requisito sin una decisión explícita nueva). Responde la pregunta 8 de la sección 21.
- **Matriz de riesgos actualizada**: la tabla de la sección 20 del documento base, con columna adicional de estado de mitigación al cierre de la Fase 0 (mitigado en diseño / mitigado en proceso / abierto y aceptado).

**Criterios de aceptación.**
- Cada fila de la matriz de riesgos de "impacto alto" tiene una mitigación verificable en un artefacto concreto de 0.C/0.D/0.E (no una intención genérica).
- La especificación de auditoría no contradice el principio de datos mínimos (sección 7.9): se revisó explícitamente qué campos NO se registran.

**Dependencias.** Fases 0.C, 0.D, 0.E (no se puede modelar amenazas sobre una arquitectura no definida).

**Riesgos que evita.** Fuga de datos sensibles hacia logs/OPA/aplicaciones (riesgo explícito, sección 20); administración de políticas sin gobernanza.

---

### Fase 0.G — Ingeniería de construcción

**Objetivo.** Convertir los principios ya confirmados en la sección 8 del documento base en configuración ejecutable y verificable del repositorio, sin esperar a que exista funcionalidad de negocio.

**Entregables.**
- **Estrategia Git formalizada**: rama principal protegida, rama de desarrollo, convención de feature branches por historia de usuario, flujo de Pull Request, número mínimo de revisores (control de cuatro ojos, sección 8.11), convención de commits, y política de versionamiento semántico del componente (crítico porque el contrato `DecisionAcceso` debe versionarse, sección 8.11).
- **Definition of Ready** y **Definition of Done**, con el DoD incluyendo explícitamente: cobertura de pruebas >80 % sobre la rama de decisión (`ALLOW`/`DENY`/`INDETERMINATE`), paso de análisis estático, verificación de límites de módulo (Spring Modulith) y, si aplica, actualización de ADR/documentación de contrato.
- **Convenciones de código**: Java, Spring, paquetes y módulos — al nivel de convención (nombrado, organización por Spring Modulith, no diseño de clases concretas), coherente con Clean Code (sección 8.6).
- **Pipeline CI** (y esqueleto de CD, aunque el despliegue final sea pendiente): compilación, análisis estático, pruebas, cobertura, y **verificación automática de límites de módulo de Spring Modulith** — este último paso es lo que convierte el riesgo "Modulith solo nominal" (sección 20) en un gate de pipeline, no en una convención de honor.
- **Estrategia de testing**: pirámide de pruebas (unitaria sobre invariantes de dominio, integración sobre adaptadores OPA/SurrealDB/IdP, contractual sobre `SolicitudAcceso`/`DecisionAcceso`), umbral de cobertura del 80 % ya confirmado, quality gates concretos (qué bloquea un PR), y decisión explícita — sí o no, y por qué — sobre introducir mutation testing en esta etapa (ver análisis en sección 8).
- **Gestión de secretos y configuración**: mecanismo (variables de entorno / vault local) para credenciales de SurrealDB, OPA y del futuro IdP, coherente con 12-Factor (sección 8.5) — a nivel de decisión y convención, no de implementación.
- **Observabilidad mínima de la línea base**: convención de logging estructurado, formato de correlación (`requestId`/`correlationId`) que debe existir desde el primer commit porque atraviesa todos los módulos.

**Criterios de aceptación.**
- El pipeline CI existe y se ejecuta en verde contra un repositorio vacío o con solo el esqueleto de módulos, antes de que exista la primera historia de usuario funcional (mandato explícito de la sección 8.12 y 8.9).
- El gate de verificación de límites de Spring Modulith está activo y falla intencionalmente si se introduce una dependencia prohibida definida en el Dependency Map de 0.C (prueba de humo de la propia línea base).
- DoD y DoR están aprobados y publicados antes de escribir la primera historia de usuario.

**Dependencias.** Fase 0.C (mapa de módulos a verificar), Fase 0.A (puede iniciarse en paralelo con 0.A en la práctica, ver nota de la sección 1).

**Riesgos que evita.** Iniciar historias de usuario funcionales sin línea base consolidada (riesgo explícito y ya marcado como bloqueante en la sección 20); pipeline "agregado una vez que el proyecto se estabilice" (advertencia explícita de la sección 8.9).

---

### Fase 0.H — Historias de esqueleto y cierre de Baseline v1.0

**Objetivo.** Escribir el número mínimo de historias de usuario necesarias para *demostrar que la arquitectura funciona de extremo a extremo*, no para entregar valor funcional. Esto no es la Etapa 1 (MVP funcional); es la validación de que la línea base sostiene el flujo completo.

**Historias mínimas de esqueleto (justificadas una por una):**

| # | Historia | Por qué es mínima e indispensable |
|---|---|---|
| E-1 | Como operador de plataforma, puedo registrar una aplicación protegida mínima (nombre, tenant, un recurso) para que exista un sujeto de prueba en el catálogo. | Sin esto no hay nada que proteger; valida el módulo Aplicaciones y la persistencia básica en SurrealDB. |
| E-2 | Como PEP, puedo recibir una solicitud, construir una `SolicitudAcceso` normalizada y enviarla al PDP. | Valida el contrato de la sección 14.1 y el límite hexagonal PEP→puerto de entrada del dominio. |
| E-3 | Como PDP, puedo resolver tenant, rol y recurso desde SurrealDB para una solicitud dada. | Valida el modelo lógico de datos (0.D) contra una consulta real, no una prueba técnica aislada. |
| E-4 | Como PDP, puedo enviar el contexto normalizado a OPA y recibir una decisión Rego RBAC simple. | Valida el contrato OPA (sección 13.4) y la prueba técnica de OPA de forma integrada, no aislada. |
| E-5 | Como PEP, puedo aplicar `ALLOW`/`DENY` y bloquear o reenviar la solicitud en consecuencia. | Cierra el ciclo de enforcement; es el criterio de éxito 6 de la sección 4.3. |
| E-6 | Como sistema, cada evaluación de las historias anteriores genera un evento de auditoría correlacionado. | Valida el catálogo de eventos de 0.E y el criterio de éxito 7 de la sección 4.3; sin esta historia, el esqueleto no es "auditable por diseño", solo "funcional". |
| E-7 | Como desarrollador, puedo confirmar en CI que las cinco historias anteriores están cubiertas por pruebas unitarias e integración y que el pipeline verifica los límites de módulo. | Cierra el lazo con 0.G: la línea base no es un documento, es un pipeline verde sobre código real, aunque mínimo. |

No se incluyen historias de ABAC, REBAC, administración de políticas ni frontend en el esqueleto: esas pertenecen a las Etapas 2-4 del documento base y su inclusión aquí sería alcance excesivo (riesgo explícito, sección 20).

**Criterios de aceptación de la Fase 0.H.**
- Las siete historias están implementadas contra la arquitectura y el modelo de datos de la línea base, ejecutándose en el pipeline de 0.G, con evidencia de auditoría verificable — esto es literalmente el criterio de éxito 1-9 de la sección 4.3 del documento base, ya que las historias E-1 a E-7 cubren uno a uno esos nueve criterios.
- Se firma formalmente el cierre de Baseline v1.0 (sección 3) y se abre la Etapa 1.

**Dependencias.** Todas las fases anteriores.

**Riesgos que evita.** Que la línea base quede en documentos sin validar que realmente sostiene el flujo PEP-PDP-OPA-SurrealDB-Auditoría bajo la arquitectura reactiva y modular decidida.

---

## 3. Baseline v1.0 — definición exacta

Al cerrar la Fase 0, **Baseline v1.0** está compuesta exactamente por:

**Gobierno (0.A)**
1. Catálogo de ADR con todas las decisiones de la sección 6 formalizadas.
2. Glosario / lenguaje ubicuo aprobado.

**Dominio (0.B)**
3. Event Storming (o Domain Storytelling) del flujo crítico.
4. Bounded Contexts + Context Map (8 contextos).
5. Casos de uso del camino crítico con sus tres desenlaces.
6. Catálogo de invariantes de negocio con identificador.
7. Respuestas cerradas a las preguntas 1, 3, 5, 6, 7 de la sección 21.

**Arquitectura de sistema (0.C)**
8. C4 Nivel 1 (Contexto).
9. C4 Nivel 2 (Contenedores).
10. C4 Nivel 3 (Componentes) — solo PEP y PDP.
11. Deployment Diagram.
12. Mapa de módulos + Dependency Map de Spring Modulith.
13. Tabla de contexto de integraciones.

**Datos (0.D)**
14. Modelo conceptual.
15. Modelo lógico (documento vs. grafo, justificado por consulta).
16. Catálogo de entidades.
17. Estrategia de persistencia e invariantes.
18. Resultado documentado de la prueba técnica de SurrealDB.

**Contratos y comportamiento (0.E)**
19. Sequence Diagrams (ALLOW, DENY, INDETERMINATE, auditoría).
20. State Diagram de `DecisionAcceso`/`SolicitudAcceso` y de asignaciones.
21. Contratos de `SolicitudAcceso` y `DecisionAcceso`.
22. ADR de precedencia de políticas (decisión cerrada, no provisional).
23. Ejemplo de entrada/salida OPA con clasificación de datos.
24. Resultado documentado de la prueba técnica OPA/Rego.
25. Catálogo mínimo de eventos de auditoría.

**Seguridad (0.F)**
26. Threat Model del flujo completo.
27. Checklist OWASP API Security mapeado a requisitos no funcionales.
28. Especificación de autenticación e identidad, con proveedor(es) del MVP decididos.
29. Especificación de auditoría (retención, clasificación, cifrado, acceso).
30. Matriz de riesgos actualizada con estado de mitigación.

**Ingeniería de construcción (0.G)**
31. Estrategia Git formalizada y activa en el repositorio.
32. Definition of Ready y Definition of Done publicados.
33. Convenciones de código (Java/Spring/paquetes/módulos).
34. Pipeline CI activo, en verde, con gate de verificación de límites de Spring Modulith.
35. Estrategia de testing con umbral de cobertura y quality gates definidos.
36. Convención de gestión de secretos/configuración.
37. Convención de logging estructurado y correlación.

**Validación de esqueleto (0.H)**
38. Siete historias de usuario mínimas (E-1 a E-7), implementadas y verdes en CI, con auditoría correlacionada verificable.

Ningún otro artefacto es obligatorio para declarar cerrada la Fase 0. Cualquier artefacto adicional (modelo físico de SurrealDB, OpenAPI de `EvaluarAcceso`, wireframes de administración, análisis de mutation testing) pertenece a etapas posteriores y no debe bloquear la apertura de la Etapa 1.

---

## 4. Roadmap resumido de fases

| Fase | Objetivo | Entregables clave | Depende de | Prioridad |
|---|---|---|---|---|
| 0.A Gobierno | Fijar cómo se registran decisiones | ADRs, glosario | — | P0 |
| 0.B Dominio | Lenguaje y casos de uso compartidos | Event Storming, Bounded Contexts, Context Map, casos de uso, invariantes | 0.A | P0 |
| 0.C Sistema | Arquitectura de contenedores/componentes/módulos | C4 (1-3), Deployment, Dependency Map | 0.B | P0 |
| 0.D Datos | Modelo de persistencia SurrealDB | Modelo conceptual/lógico, catálogo entidades, prueba técnica DB | 0.B, 0.C | P0 |
| 0.E Contratos | Comportamiento y contratos de integración | Sequence/State Diagrams, contratos SolicitudAcceso/DecisionAcceso, ADR precedencia, prueba técnica OPA | 0.C, 0.D | P0/P1 |
| 0.F Seguridad | Amenazas y auditoría del propio componente | Threat Model, OWASP checklist, spec identidad/auditoría, matriz de riesgos | 0.C, 0.D, 0.E | P1 |
| 0.G Ingeniería | Repositorio ejecutable y verificable | Git, CI/CD, DoD/DoR, testing, convenciones | 0.C (en paralelo con 0.A) | P0 |
| 0.H Esqueleto | Validar la arquitectura de extremo a extremo | 7 historias E-1..E-7, pipeline verde | Todas | P0 |

---

## 5. Artefactos analizados por categoría

Formato por artefacto: **¿Necesario? · Por qué / riesgo que evita · Cuándo · Prioridad · Depende de / alimenta a · Nivel de detalle.**

### Arquitectura

| Artefacto | ¿Necesario? | Justificación y riesgo que evita | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Context Diagram (C4-1) | Sí | Fija el límite del sistema frente a IdP y aplicaciones protegidas; evita que el equipo confunda "plataforma" con "una app más". | 0.C | P0 | Una página, sin tecnología. |
| Container Diagram (C4-2) | Sí | Es la proyección directa de la sección 9.1; sin él cada desarrollador infiere el límite PEP/PDP a su manera. | 0.C | P0 | Contenedores + protocolos, sin clases internas. |
| Component Diagram (C4-3) | Sí, acotado a PEP+PDP | Hacerlo de todos los contenedores es trabajo prematuro (Administración no existe aún); en PEP/PDP es donde vive la separación hexagonal crítica. | 0.C | P0 | Puertos y adaptadores, no clases. |
| Deployment Diagram | Sí | Materializa Cloud Enable (sección 8.3) como restricción verificable, no como aspiración. | 0.C | P1 | Topología de contenedores + anotación de qué es reemplazable. |
| Contexto de integraciones | Sí, como tabla (fusionado, no diagrama aparte) | Responde la pregunta 10 de la sección 21 sin duplicar el Container Diagram. | 0.C | P1 | Tabla: sistema, protocolo, dato mínimo, modo. |
| Arquitectura Hexagonal (documento) | No como documento aparte | Ya está descrita en la sección 8.7 y 18.1 del documento base; documentarla de nuevo es redundante. Se referencia, no se reescribe. | — | — | — |
| Arquitectura Reactiva (documento) | No como documento aparte | Igual que el anterior (sección 8.4); su único entregable nuevo es el **resultado de la prueba técnica de WebFlux**, que sí es necesario. | 0.C/0.G | P0 (la prueba, no el documento) | Informe corto de la prueba técnica. |
| Spring Modulith (documento) | No como documento aparte | Se materializa como Mapa de módulos + Dependency Map + gate de CI; un documento narrativo adicional no reduce riesgo. | 0.C/0.G | — | — |
| Mapa de módulos / Dependency Map | Sí | Es el artefacto contra el que se verifica en CI; sin él, "Modulith" es solo una palabra en el pipeline. | 0.C | P0 | Explícito: quién puede depender de quién. |
| Event Flow | No como documento aparte | Se fusiona con Container Diagram (flujo) + catálogo de eventos de auditoría (0.E). | — | — | — |
| Sequence Diagrams | Sí | Es donde se decide, antes de programar, qué pasa exactamente en `DENY` e `INDETERMINATE`, no solo en el camino feliz. | 0.E | P0 | Uno por desenlace (ALLOW/DENY/INDETERMINATE) + auditoría. |
| Activity Diagrams | No | Redundante con Sequence Diagrams para este dominio (poca lógica de proceso paralelo/humano); no aporta sobre lo que ya cubre 0.B (casos de uso) y 0.E (secuencia). | — | — | — |
| State Diagrams | Sí, acotado | Solo para `DecisionAcceso`/`SolicitudAcceso` y asignaciones, que sí tienen transiciones no triviales (vigente/vencida/revocada). | 0.E | P1 | Dos máquinas de estado, no una por entidad. |

### Dominio

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Event Storming | Sí | Es la forma más rápida de validar el flujo completo con el propio autor del proyecto antes de comprometerlo a diagramas formales. | 0.B | P0 | Big picture, sesión única. |
| Domain Storytelling | Alternativa a Event Storming, no adicional | Cubren la misma pregunta; elegir uno evita duplicar esfuerzo. Se recomienda Event Storming por ya existir vocabulario técnico (PEP/PDP/OPA) más que actores humanos. | 0.B | — | — |
| Bounded Contexts | Sí | Ya insinuados en la sección 11; formalizarlos evita que se conviertan en "módulos técnicos por capa" en vez de por capacidad de negocio. | 0.B | P0 | 8 contextos, con responsabilidad de una frase cada uno (ya existe en sección 11, se ratifica). |
| Context Map | Sí | Sin él, dos módulos pueden reclamar la misma entidad (p. ej. `Rol`) sin que nadie note el conflicto hasta el código. | 0.B | P0 | Tipo de relación entre los 8 contextos. |
| Ubiquitous Language | Sí (fusionado con Glosario de 0.A) | Ya existe como glosario en la sección 23; no se crea un segundo documento, se congela el existente como autoridad. | 0.A | P0 | Tabla término-definición, ya existente. |
| Agregados | No en esta fase | Diseño táctico DDD detallado; se define junto con el primer caso de uso real de implementación, no antes (evita rehacer, sección 22 punto 4). | Diseño detallado (fuera de Fase 0) | — | — |
| Entidades / Value Objects | No en esta fase | Misma razón que agregados; además la restricción explícita del prompt excluye entidades Java. | Diseño detallado | — | — |
| Servicios de dominio | No en esta fase | Se derivan naturalmente de los casos de uso ya fijados (`EvaluarAcceso`, etc.); nombrarlos ahora es anticipar diseño de clases. | Diseño detallado | — | — |
| Reglas de negocio / invariantes | Sí | Ya parcialmente listadas (secciones 7, 12.4, 13.5); consolidarlas con identificador las hace verificables por prueba unitaria (sección 8.10). | 0.B | P0 | Lista con ID, una frase, trazable a sección origen. |
| Casos de uso | Sí | Nombrados en 18.1 pero no especificados; sin flujo alterno documentado, `DENY`/`INDETERMINATE` se improvisan en código. | 0.B | P0 | Precondición/flujo/postcondición, sin firma de método. |
| Políticas (como concepto de dominio) | Sí (fusionado con 0.E) | Su modelo conceptual pertenece a 0.B (qué es una política), su contrato de evaluación a 0.E; no se separan en un tercer documento. | 0.B/0.E | P1 | — |

### Datos

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Modelo conceptual | Sí | Punto de partida obligatorio antes de decidir documento vs. grafo por entidad. | 0.D | P0 | Entidades + relaciones, sin tipos de dato. |
| Modelo lógico | Sí | Es donde se justifica cada relación de grafo contra una consulta real (principio 12.1); el artefacto de mayor riesgo si se omite. | 0.D | P0 | Por entidad: documento o grafo, y por qué. |
| Modelo físico | No en esta fase | El propio documento base trata la notación de relaciones como conceptual, no física; fijarlo antes del primer caso de uso implementado se contradice con la evidencia de que cambiará. | Etapa 1 (implementación) | — | — |
| ER Diagram | No, fusionado en Modelo lógico | Un ER clásico no representa bien aristas de grafo; el modelo lógico ya cumple la misma función con notación más honesta para SurrealDB. | — | — | — |
| Catálogo de entidades | Sí | Trazabilidad entidad → contexto dueño → invariante protegida; evita entidades "huérfanas" sin responsable. | 0.D | P1 | Tabla simple. |
| Estrategia de persistencia | Sí | Cubre transacciones, índices e invariantes críticas explícitamente exigidas en la sección 12. | 0.D | P0 | Reglas + qué invariante protege cada una. |

### Ingeniería

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Estrategia Git (branching, PR, code review) | Sí | Exigencia explícita y confirmada (sección 8.11); condición para abrir la Etapa 1. | 0.G | P0 | Documento corto + configuración real del repo. |
| Definition of Done / Ready | Sí | Sin DoD explícito, "cobertura >80%" (sección 8.10) no es verificable por historia individual. | 0.G | P0 | Checklist. |
| Convenciones Java/Spring/paquetes/módulos | Sí, a nivel de convención | Necesario para que Clean Code (8.6) y Modulith (8.8) sean consistentes entre desarrolladores; no incluye diseño de clases. | 0.G | P1 | Guía corta + ejemplos de nombrado. |

### Calidad

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Estrategia de testing / pirámide | Sí | Ya exigida (8.10); formalizar qué capa prueba qué evita que toda la cobertura se concentre en pruebas triviales. | 0.G | P0 | Por capa: qué se prueba y con qué tipo de prueba. |
| Cobertura / Quality Gates | Sí | Umbral ya confirmado (>80%); el gate debe ser mecánico en CI, no una revisión manual. | 0.G | P0 | Umbral + qué bloquea merge. |
| SonarQube (o equivalente) | Sí, como herramienta del gate anterior, no como documento aparte | Es la implementación del quality gate, no un artefacto de diseño distinto. | 0.G | P1 | Configuración, no documento. |
| Mutation Testing | No en la línea base | Aporta valor sobre todo cuando ya existe una base de pruebas de invariantes crítica; introducirlo antes de tener las primeras 7 historias (0.H) es medir la calidad de pruebas que aún no existen. Se registra como práctica a evaluar en la Etapa 2, no se descarta. | Etapa 2+ | P2 | — |

### DevOps

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Pipeline CI | Sí | Mandato explícito de existir "desde el inicio" (8.9); es el artefacto más citado como bloqueante en la sección 20. | 0.G | P0 | Compilación, análisis estático, pruebas, cobertura, gate de módulos. |
| Pipeline CD | Parcial | El entorno productivo está pendiente (sección 6); se prepara el esqueleto (build de imagen, publicación) sin decidir destino final. | 0.G | P2 | Esqueleto, no despliegue real. |
| Versionado / Releases | Sí (fusionado con Estrategia Git) | Ya exigido en 8.11 como parte de la misma decisión; no es un documento separado. | 0.G | P1 | — |
| Gestión de secretos / variables de entorno | Sí | Exigencia de 12-Factor (8.5); decisión de mecanismo, no implementación. | 0.G | P1 | Convención + herramienta elegida. |
| Observabilidad / logging / métricas / trazabilidad | Parcial en Fase 0 | Solo la convención de correlación (`requestId`/`correlationId`) y formato de log estructurado son necesarios ahora, porque atraviesan todos los módulos desde el primer commit; métricas y trazas distribuidas completas son de la Etapa 2 (sección 19). | 0.G (convención) / Etapa 2 (implementación) | P1 (convención) | Formato de log + campos obligatorios. |

### Seguridad

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| Threat Modeling | Sí | El propio documento es sobre seguridad; no amenazarlo a sí mismo antes de construirlo es la contradicción de mayor riesgo posible. | 0.F | P1 | STRIDE sobre el flujo de 0.C/0.E. |
| Riesgos (matriz) | Sí (ya existe, se actualiza) | La sección 20 ya lista riesgos; el trabajo de la línea base es cerrarlos con mitigación verificable, no reescribirlos. | 0.F | P0 | Tabla existente + columna de estado. |
| OWASP | Sí | La plataforma expone una API crítica (decisión de acceso); OWASP API Security es el checklist con mayor aplicabilidad directa. | 0.F | P1 | Checklist mapeado a NFRs existentes (sección 17). |
| Autenticación (especificación) | Sí | Formaliza la sección 15 y cierra la pregunta 2 de la sección 21. | 0.F | P0 | Flujo + proveedor(es) decididos. |
| Autorización (especificación) | No como documento nuevo | Ya cubierta extensamente por las secciones 13-14 del documento base y por 0.E (contratos); un documento adicional duplicaría contenido sin reducir riesgo. | — | — | — |
| Auditoría (especificación) | Sí | Formaliza retención/cifrado/acceso, pendientes explícitos de la sección 16.3. | 0.F | P1 | Política concreta, no solo principios. |

### Documentación

| Artefacto | ¿Necesario? | Justificación | Cuándo | Prioridad | Nivel de detalle |
|---|---|---|---|---|---|
| ADR | Sí | Es el mecanismo central de gobierno de decisiones de todo este plan. | 0.A | P0 | Uno por decisión, plantilla corta. |
| Catálogo de decisiones | No aparte | Es exactamente la tabla de la sección 6 del documento base + el catálogo de ADR; mantener un tercer catálogo sería redundante. | — | — | — |
| Glosario | Sí (ya existe) | Sección 23 del documento base, se ratifica como autoridad. | 0.A | P0 | Existente, congelado como referencia. |
| Diccionario de datos | Sí (fusionado con Catálogo de entidades) | Mismo contenido que 0.D con otro nombre; se evita duplicar. | 0.D | — | — |
| Diccionario de eventos | Sí (fusionado con Catálogo de eventos de auditoría) | Mismo caso que el anterior. | 0.E | — | — |
| Catálogo de APIs | No en esta fase | Implica diseño de endpoints/OpenAPI, explícitamente fuera de alcance de la Fase 0 por restricción del propio encargo. | Diseño detallado | — | — |
| Convenciones REST / errores | No en esta fase | Misma razón; se define junto con el primer contrato de API real en la Etapa 1. | Diseño detallado | — | — |

### UX

| Artefacto | ¿Necesario? | Justificación |
|---|---|---|
| User Journey / Story Mapping / Personas / Wireframes / Navegación | No en la Fase 0 | La interfaz administrativa está explícitamente marcada como **pendiente** en la tabla de decisiones (sección 6); invertir en UX de algo cuya existencia y tecnología no están decididas es el tipo exacto de trabajo prematuro que el propio documento base pide evitar (sección 5.2, "no seleccionar tecnología hasta definir usuarios, casos de uso y restricciones"). Se retoma en la Etapa 4 (sección 19), solo si el frontend aporta valor a la validación del trabajo de grado. |

### Gestión

Ya cubierto en la Fase 0.H: únicamente las siete historias de esqueleto (E-1 a E-7), que existen solo para validar que la arquitectura funciona de extremo a extremo, no para entregar funcionalidad completa. No se generan épicas ni backlog del MVP en la Fase 0: ese backlog (con historias funcionales reales) es, según el propio documento base (sección 22.1), el resultado de un prompt de continuación posterior, una vez cerrada esta línea base.

---

## 6. Fusiones y eliminaciones aplicadas (resumen)

**Fusiones:**
- Activity Diagrams + State Diagrams → un State Diagram acotado a `DecisionAcceso` y asignaciones dentro de 0.E; el resto del comportamiento ya lo cubren casos de uso (0.B) y Sequence Diagrams (0.E).
- Event Flow (documento aparte) → Container Diagram (0.C) + Catálogo de eventos de auditoría (0.E).
- ER Diagram → Modelo lógico de datos (0.D), porque SurrealDB no es relacional puro y el ER clásico distorsiona las relaciones de grafo.
- Diccionario de datos → Catálogo de entidades (0.D).
- Diccionario de eventos → Catálogo de eventos de auditoría (0.E).
- Documentos narrativos de "Arquitectura Hexagonal", "Arquitectura Reactiva" y "Spring Modulith" → no se reescriben como documentos nuevos; ya están definidos en las secciones 8.4, 8.7 y 8.8 del documento base y se referencian, no se duplican. Lo único nuevo que producen es evidencia empírica: los resultados de las pruebas técnicas correspondientes.
- Catálogo de decisiones → tabla de la sección 6 del documento base + catálogo de ADR (0.A); no se crea un tercer registro.
- Versionado/Releases → parte de la Estrategia Git (0.G), no un documento independiente.
- SonarQube → configuración del quality gate (0.G), no un documento de diseño.
- Autorización (especificación aparte) → ya cubierta por las secciones 13-14 del documento base más los contratos de 0.E.

**Eliminaciones (no se producen en la Fase 0):**
- Agregados, entidades, value objects, servicios de dominio en sentido de diseño táctico DDD — se posponen a diseño detallado, después de validar el primer caso de uso real.
- Modelo físico de SurrealDB, Catálogo de APIs, convenciones REST/errores — pertenecen a diseño detallado/Etapa 1, no a la línea base, y están explícitamente excluidos por la restricción de no proponer APIs.
- Mutation testing — se evalúa en Etapa 2, cuando ya exista una base de pruebas sobre la que medir su calidad.
- UX (User Journey, Story Mapping, Personas, Wireframes, Navegación) — la interfaz administrativa sigue pendiente por decisión explícita del documento base; producir estos artefactos ahora sería trabajo especulativo sobre una tecnología/alcance no decidido.
- Backlog completo del MVP con épicas — pertenece a un prompt de continuación posterior (sección 22.1 del documento base), no a la Fase 0.

---

## 7. Riesgos que esta línea base evita en conjunto

Esta lista no repite la matriz de riesgos de la sección 20 del documento base; muestra qué combinación de fases neutraliza cada riesgo de mayor impacto:

- **Alcance excesivo** → neutralizado por 0.B (un solo caso de validación decidido) + Baseline v1.0 explícita (sección 3) que excluye backlog, UX y modelo físico.
- **Modelo de grafo sin casos de uso** → neutralizado por 0.D (cada relación justificada por consulta) + prueba técnica de SurrealDB.
- **Políticas ambiguas o contradictorias** → neutralizado por el ADR de precedencia de políticas (0.E), cerrado antes de escribir Rego.
- **Fuga de datos sensibles** → neutralizado por el Threat Model + clasificación de atributos hacia OPA (0.F).
- **PEP proxy inseguro o frágil** → neutralizado por Component Diagram con límites explícitos (0.C) + Threat Model (0.F).
- **Adoptar WebFlux con dependencias bloqueantes** → neutralizado por la prueba técnica de WebFlux exigida en 0.C/0.G, con resultado documentado antes de construir sobre ella.
- **Modulith solo nominal** → neutralizado por el Dependency Map (0.C) convertido en gate mecánico de CI (0.G), no en convención.
- **Iniciar historias sin línea base** → neutralizado estructuralmente: 0.H es la única fase que autoriza abrir la Etapa 1, y solo lo hace si las siete historias de esqueleto pasan en un pipeline verde.

---

## 8. Nota de cierre

Este plan no reemplaza al documento base; lo opera. Cuando una fase produzca una decisión nueva (por ejemplo, el ADR de precedencia de políticas, o el proveedor de identidad elegido), esa decisión debe reflejarse también en la tabla de la sección 6 y en la bitácora de control de cambios (sección 25) del documento base, tal como este mismo exige en su cierre. Este documento, a su vez, debe marcarse como cerrado y fechado en el momento en que se firme el cierre de Baseline v1.0 (sección 3), y no debe editarse retroactivamente sin dejar constancia del cambio.
