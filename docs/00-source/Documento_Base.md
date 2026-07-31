# Documento base - Plataforma Central de Seguridad

> **Estado:** documento fundacional vivo.  
> **Propósito:** contexto inicial compartido para el trabajo de grado y fuente de entrada para construir historias de usuario, planificación, diagramas, prototipos y decisiones posteriores.  
> **Última consolidación:** 30 de julio de 2026.  
> **Regla de lectura:** una afirmación marcada como **confirmada** es una decisión tomada; una marcada como **propuesta** orienta el diseño, pero debe validarse antes de implementarse; una marcada como **pendiente** no debe asumirse como requisito.

---

## 1. Resumen ejecutivo

El proyecto busca construir una **plataforma central de seguridad reutilizable e integrable** por otras aplicaciones. No pretende ser solamente una pantalla de inicio de sesión ni una librería de permisos aislada. Su objetivo es concentrar capacidades de identidad, autenticación delegada, autorización por políticas, control de acceso, trazabilidad y auditoría para que las aplicaciones integradas no tengan que reimplementar estas preocupaciones de forma dispersa.

La idea central es que una solicitud dirigida a una aplicación protegida pase por un **Punto de Aplicación de Políticas** (*Policy Enforcement Point*, PEP). El PEP obtiene y normaliza la información relevante de la solicitud; un **Punto de Decisión de Políticas** (*Policy Decision Point*, PDP) construye el contexto de autorización, consulta el modelo de seguridad y evalúa las políticas. El resultado es una decisión explícita, por ejemplo `ALLOW` o `DENY`, que el PEP debe hacer cumplir antes de permitir que la petición continúe hacia la aplicación destino.

La plataforma se orienta a entornos multiaplicación y multitenant. Debe combinar, de manera coherente y auditable, cuatro perspectivas de control de acceso:

- **RBAC** (*Role-Based Access Control*): permisos derivados de roles.
- **ABAC** (*Attribute-Based Access Control*): decisiones condicionadas por atributos del sujeto, recurso, entorno o transacción.
- **REBAC** (*Relationship-Based Access Control*): decisiones basadas en relaciones entre identidades, organizaciones, aplicaciones y recursos.
- **PBAC** (*Policy-Based Access Control*): reglas de decisión expresadas como políticas.

Las decisiones tecnológicas confirmadas son **Java**, **Spring Boot**, **Spring WebFlux** para toda la capa que maneja tráfico HTTP e integraciones remotas, **SurrealDB como persistencia principal**, **OPA y Rego para la evaluación de políticas**, autenticación delegada y auditoría obligatoria como parte del producto. A estas se suma un conjunto de decisiones de ingeniería y construcción, igualmente confirmadas, que gobiernan cómo se construye el componente y no solo con qué se construye: **Clean Architecture** implementada mediante **arquitectura hexagonal**, organización interna con **Spring Modulith**, principios **Cloud Native** con preparación explícita para migrar hacia Kubernetes o una plataforma cloud (**Cloud Enable**), cumplimiento de **12-Factor App** y sus factores extendidos para aplicaciones distribuidas, **Clean Code** como estándar de escritura, **integración continua** desde el inicio, una **estrategia de pruebas** con cobertura superior al 80 % y una **estrategia Git** formalizada antes de escribir la primera historia de usuario funcional. La interfaz web administrativa todavía no está decidida.

---

## 2. Problema que aborda

En un ecosistema con varias aplicaciones, cada equipo suele resolver seguridad dentro de su propio sistema. Esto provoca reglas repetidas, criterios de acceso inconsistentes, integraciones frágiles con proveedores de identidad, dificultad para cambiar una política transversal y poca capacidad de explicar por qué un acceso fue concedido o negado.

El problema no se reduce a saber si un usuario inició sesión. Una decisión de acceso puede depender simultáneamente de:

- quién realiza la solicitud;
- a qué tenant u organización pertenece;
- cuál aplicación intenta usar;
- cuál recurso, funcionalidad o acción solicita;
- qué roles y perfiles tiene asignados;
- qué relaciones vigentes existen entre sujeto, recurso y tenant;
- atributos dinámicos: hora, canal, dispositivo, ubicación, sesión, estado de la transacción o riesgo;
- la versión de las políticas activas.

La plataforma deberá convertir esos elementos en una decisión consistente, verificable y trazable, sin invadir la lógica de negocio propia de cada aplicación protegida.

---

## 3. Visión del producto

### 3.1 Declaración de visión

Proveer un componente de seguridad central que permita a aplicaciones heterogéneas delegar de forma estandarizada la autenticación y, principalmente, la autorización de solicitudes, con políticas versionables, modelo de relaciones, decisiones explicables y evidencia auditable.

### 3.2 Propuesta de valor

Para las aplicaciones integradas, la plataforma debe ofrecer:

- Un punto de integración homogéneo, independiente del lenguaje de la aplicación protegida.
- Un contrato claro para solicitar decisiones de autorización.
- Una única fuente de definición para roles, recursos, asignaciones y políticas.
- Capacidad de modelar relaciones complejas sin limitarse al par usuario-rol.
- Evidencia de cada decisión para auditoría, soporte y depuración.
- Evolución de las políticas sin tener que duplicar reglas en todos los sistemas consumidores.
- Evolución sostenida del propio componente: al aislar el dominio de seguridad de Spring, OPA y SurrealDB mediante arquitectura hexagonal, y al organizar sus capacidades como módulos independientes con Spring Modulith, la plataforma puede crecer, sustituir dependencias o extraer un módulo hacia un servicio propio sin comprometer el resto del sistema.

### 3.3 Límites del producto

La plataforma es responsable de seguridad transversal, no de la lógica funcional de las aplicaciones consumidoras. Por ejemplo, puede decidir si un sujeto puede `actualizar` una factura específica; no debe calcular el valor de la factura ni ejecutar el caso de negocio de facturación.

Tampoco debe convertirse en una copia de datos de negocio completa. Las aplicaciones integradas deben aportar al contexto de autorización únicamente los atributos y relaciones necesarios, bajo contratos definidos y minimizando datos sensibles.

---

## 4. Objetivos

### 4.1 Objetivo general

Diseñar y construir una plataforma central de seguridad que permita a aplicaciones externas delegar la evaluación y aplicación de controles de acceso basados en identidad, atributos, relaciones y políticas, con trazabilidad completa de sus decisiones.

### 4.2 Objetivos específicos

1. Definir un modelo de dominio para tenants, aplicaciones, recursos, identidades, roles, perfiles, asignaciones, políticas y auditoría.
2. Implementar un mecanismo de interceptación o integración que permita proteger aplicaciones de tecnologías distintas.
3. Normalizar la identidad recibida desde proveedores de autenticación sin acoplar el dominio a un proveedor concreto.
4. Construir un contexto de autorización uniforme para cada solicitud.
5. Integrar un motor de políticas OPA con políticas escritas en Rego.
6. Soportar RBAC, ABAC, REBAC y PBAC de manera combinable.
7. Persistir relaciones de seguridad y configuración en SurrealDB, aprovechando sus capacidades de documento y grafo cuando agreguen valor.
8. Registrar decisiones y evidencias suficientes para auditoría y explicación.
9. Validar el enfoque con aplicaciones de referencia implementadas en más de una tecnología.
10. Establecer, antes de iniciar el desarrollo funcional, una línea base de ingeniería (arquitectura hexagonal, módulos, estrategia Git, pipeline de integración continua y estrategia de pruebas) construida sobre principios Cloud Native y 12-Factor, de modo que la plataforma sea mantenible, verificable y evolutiva desde su primer commit.

### 4.3 Criterios de éxito iniciales

La primera versión será útil si puede demostrar, de extremo a extremo, que:

1. Una aplicación puede registrarse o configurarse como aplicación protegida.
2. Una solicitud llega a un PEP antes de alcanzar la aplicación destino.
3. La plataforma identifica o rechaza de forma segura una identidad no válida.
4. La plataforma construye un contexto con sujeto, aplicación, recurso, acción y atributos relevantes.
5. OPA evalúa una política Rego y devuelve una decisión consumible.
6. El PEP permite o bloquea la solicitud según dicha decisión.
7. Cada decisión deja un evento de auditoría correlacionado con la solicitud.
8. Es posible explicar qué política, versión y datos principales llevaron a la decisión, sin exponer secretos.
9. La línea base de ingeniería —repositorio con la estrategia Git definida, pipeline de integración continua y primeras pruebas automatizadas— existe y se ejecuta correctamente antes de que se escriba la primera historia de usuario funcional.

---

## 5. Alcance

### 5.1 Alcance funcional esperado

- Gestión de tenants y aislamiento lógico entre ellos.
- Registro y configuración de aplicaciones consumidoras/protegidas.
- Catálogo de recursos, funcionalidades y acciones protegibles.
- Gestión de usuarios, identidades externas, estado de usuario y sesiones según el alcance elegido.
- Roles, perfiles y asignaciones por usuario, tenant y aplicación.
- Relaciones necesarias para decisiones REBAC.
- Administración, publicación y versionado de políticas de acceso.
- Evaluación de solicitudes de autorización mediante OPA/Rego.
- Enforcement en el borde de la solicitud mediante PEP.
- Auditoría de solicitudes, decisiones, errores y evidencia asociada.
- APIs de integración y, posteriormente, una interfaz de administración reactiva.
- Línea base de ingeniería: estructura modular del repositorio bajo Spring Modulith, estrategia Git, pipeline de integración continua y estrategia de pruebas automatizadas, definidos antes de iniciar las primeras historias de usuario.

### 5.2 Fuera de alcance inicial

Los siguientes temas no deben incluirse en el MVP salvo que se aprueben explícitamente:

- Reemplazar completamente a un proveedor de identidad empresarial maduro.
- Implementar un directorio corporativo, autenticación biométrica o gestión completa de credenciales desde cero.
- Convertir la plataforma en un API Gateway general con balanceo, transformación de tráfico y gestión integral de APIs.
- Sincronizar toda la información de negocio de las aplicaciones integradas.
- Ofrecer garantías criptográficas de blockchain para auditoría. El diagrama menciona `HashBlockchain`; por ahora debe tratarse como una posibilidad de investigación, no como un requisito confirmado.
- Resolver todos los modelos posibles de autorización antes de validar un flujo mínimo funcional.
- Operar la plataforma en un entorno cloud gestionado en producción durante el MVP. El principio Cloud Enable exige que la arquitectura quede preparada para esa migración; no exige que el despliegue productivo en nube ocurra en esta etapa.

---

## 6. Decisiones, hipótesis y pendientes

| Tema | Estado | Decisión o dirección actual | Implicación |
|---|---|---|---|
| Lenguaje principal | Confirmada | Java | Núcleo de la plataforma y servicios de seguridad en Java. |
| Framework base | Confirmada | Spring Boot | Base para APIs, configuración, seguridad, pruebas e integración. |
| Modelo reactivo | Confirmada | Spring WebFlux | Todo componente HTTP de la plataforma —PEP, PDP y los clientes hacia OPA, SurrealDB y el proveedor de identidad— se construye de forma reactiva y no bloqueante. No se permiten componentes bloqueantes salvo aislamiento explícito y justificado. La demo actual usa Spring MVC y deberá refactorizarse antes de convertirse en base de producción. |
| Persistencia principal | Confirmada | SurrealDB | Fuente de verdad del dominio de seguridad; usar registros, documentos, relaciones/grafo y transacciones conforme al diseño validado. |
| Motor de políticas | Confirmada | Open Policy Agent (OPA) | Evalúa reglas externas al código de negocio. |
| Lenguaje de políticas | Confirmada | Rego | Las políticas deben versionarse y ser trazables. |
| Autenticación | Confirmada como enfoque | Delegada y abstraída por proveedor | La plataforma valida y normaliza identidad; no debe depender de un IdP único. |
| Modelos de autorización | Confirmada como objetivo | RBAC + ABAC + REBAC + PBAC | Se implementarán por incrementos; no todos deben estar completos en el MVP. |
| Auditoría | Confirmada | Obligatoria | Las decisiones no pueden ser silenciosas. |
| Arquitectura de software | Confirmada | Clean Architecture mediante arquitectura hexagonal | El dominio de seguridad no depende de Spring, de clientes OPA, de drivers de SurrealDB ni de ningún framework; toda dependencia apunta hacia el dominio. |
| Organización interna | Confirmada | Spring Modulith | Los módulos internos representan capacidades de negocio (tenants, aplicaciones, recursos, identidad, asignaciones, políticas, auditoría), no una separación técnica por capas. |
| Principios de construcción | Confirmada | Cloud Native + 12-Factor App y factores extendidos | Servicios desacoplados, configuración externa, observabilidad, despliegue en contenedores, escalabilidad horizontal, resiliencia y automatización desde el diseño inicial. |
| Preparación para nube | Confirmada | Cloud Enable | La arquitectura queda preparada para migrar hacia Kubernetes o una plataforma cloud; el entorno productivo definitivo no es una decisión de esta etapa. |
| Calidad de código | Confirmada | Clean Code | Estándar obligatorio de legibilidad, cohesión y bajo acoplamiento en todo el código del componente. |
| Integración continua | Confirmada | Pipeline CI/CD desde el inicio | Compilación, análisis estático, pruebas y cobertura se validan en cada cambio, no solo antes de un release. |
| Estrategia de pruebas | Confirmada | Cobertura superior al 80 %, pruebas unitarias y de integración | Las pruebas son parte del diseño, no una etapa posterior. |
| Estrategia Git | Confirmada | Rama principal, rama de desarrollo, feature branches, Pull Requests, revisiones y versionamiento | Gobierna la evolución del código y de las políticas versionadas. |
| Línea base | Confirmada | Debe existir antes de las primeras historias de usuario | Incluye visión, documentación, diagramas, arquitectura, módulos, estrategia Git, estrategia CI/CD, estrategia de pruebas y los primeros casos de uso e historias. |
| Interfaz administrativa | Pendiente | Frontend reactivo por definir | No seleccionar tecnología hasta definir usuarios, casos de uso y restricciones. |
| Despliegue | Confirmado el principio; pendiente el entorno final | Contenedores desde el inicio, con Cloud Enable como preparación hacia Kubernetes o nube | La demo usa Docker Compose. El entorno productivo definitivo (nube, on-premise o híbrido) sigue sin decidirse. |
| Alta disponibilidad y caché | Pendiente | Caché solo como acelerador | Nunca usar caché como fuente de verdad de autorización. |
| Modelo exacto de relaciones y esquema SurrealDB | Pendiente | Derivarlo del dominio y casos de uso | Debe probarse con consultas y reglas REBAC reales. |

### 6.1 Aclaración sobre SurrealDB

La decisión vigente es **usar SurrealDB**, no una base SQL, como persistencia principal del componente. Esta decisión reemplaza la recomendación histórica encontrada en la síntesis anterior, que proponía SQL como fuente de verdad. Esa recomendación se conserva solo como antecedente, no como arquitectura objetivo actual.

SurrealDB se elige porque el dominio requiere tanto datos estructurados como relaciones navegables: usuario-tenant, usuario-rol, rol-recurso, perfil-rol, aplicación-recurso y posibles relaciones de confianza, pertenencia o propiedad necesarias para REBAC. El diseño no debe asumir que una capacidad de grafo elimina la necesidad de integridad, control de concurrencia, auditoría, índices, estrategia de migración o pruebas de desempeño.

### 6.2 Aclaración sobre Spring WebFlux

La programación reactiva deja de ser una dirección deseable y pasa a ser una decisión arquitectónica oficial: todo componente que maneje tráfico HTTP entrante o llamadas salientes hacia OPA, SurrealDB o el proveedor de identidad se construye sobre Spring WebFlux. Esto no significa que el riesgo técnico desaparezca; significa que ese riesgo se valida con una prueba técnica temprana en la Etapa 0, en lugar de decidirse por conveniencia una vez avanzado el desarrollo. La regla de aislamiento se mantiene sin excepción: ningún componente bloqueante puede introducirse en el flujo reactivo sin una justificación explícita y un mecanismo de aislamiento —por ejemplo, un *scheduler* dedicado— documentado en el diseño técnico correspondiente.

---

## 7. Principios de diseño

1. **Denegar por defecto.** Ante ausencia de política aplicable, identidad inválida, contexto incompleto o error que impida una decisión confiable, la posición inicial debe ser negar; las excepciones deben estar justificadas y auditadas.
2. **Separar decisión y aplicación.** El PDP decide; el PEP aplica la decisión. Un interceptor no debe contener reglas complejas de autorización.
3. **Políticas fuera de la lógica de negocio.** Las reglas de acceso deben poder evolucionar sin replicarse en cada aplicación.
4. **Mínimo privilegio.** Se conceden únicamente los permisos necesarios y por el tiempo/contexto requerido.
5. **Defensa en profundidad.** El PEP no sustituye validación de tokens, seguridad de red, protección de secretos, validación de entrada ni controles internos de la aplicación.
6. **Trazabilidad por diseño.** Toda decisión debe ser reconstruible con datos suficientes y protegidos.
7. **Aislamiento de tenant.** El tenant debe ser un dato explícito y validado dentro de cada decisión y consulta sensible.
8. **Contratos estables de integración.** Las aplicaciones no deben conocer detalles internos de SurrealDB, OPA ni de las estructuras de dominio.
9. **Datos mínimos.** Al PDP solo deben llegar atributos necesarios; evitar enviar secretos, credenciales o información personal innecesaria a OPA o a logs.
10. **Evolución incremental.** Primero comprobar el camino crítico, luego ampliar modelos de autorización y capacidades operativas.

Estos principios definen **qué** debe ser cierto sobre cada decisión de seguridad que produce la plataforma. La sección siguiente define **cómo** se construye el software que hace posible sostener esos principios en el tiempo.

---

## 8. Principios de Ingeniería y Construcción

Esta sección recoge las decisiones de ingeniería que gobiernan la construcción del componente, independientemente de qué caso de uso de seguridad se esté implementando en un momento dado. No son preferencias de estilo: cada una de ellas condiciona directamente cómo se diseñan los módulos, cómo se prueban, cómo se despliegan y cómo se les puede confiar una responsabilidad tan sensible como decidir el acceso de terceros a otras aplicaciones.

### 8.1 Harness Engineering

El proyecto se construye bajo una estrategia de ingeniería orientada a calidad, mantenibilidad, automatización, verificabilidad y evolución. Este *harness* no es una técnica puntual, sino el criterio bajo el cual se evalúa cualquier otra decisión de este documento: ninguna elección arquitectónica, de biblioteca o de proceso se adopta porque "funciona", sino porque puede justificarse técnicamente frente a esos cinco criterios. Esta exigencia importa especialmente aquí porque la plataforma no es un componente de negocio más: es el componente del que dependen otras aplicaciones para decidir si un acceso es legítimo. Un defecto de mantenibilidad o de verificabilidad en este proyecto no se queda contenido en él; se traduce en riesgo de autorización para cada aplicación integrada.

### 8.2 Cloud Native

La arquitectura sigue principios Cloud Native como mínimo en seis dimensiones: servicios desacoplados, configuración externa, observabilidad, despliegue mediante contenedores, escalabilidad horizontal y resiliencia, sostenidos por automatización. En esta plataforma esto se traduce en decisiones concretas: el PEP, el PDP, OPA, SurrealDB y la auditoría se piensan como unidades desplegables de forma independiente; los secretos, la ubicación del bundle de políticas y los endpoints del proveedor de identidad se inyectan como configuración externa y nunca se incrustan en el artefacto; el PDP se diseña sin estado propio relevante para poder escalar horizontalmente; y el comportamiento ante la caída de OPA, SurrealDB o el IdP —ya exigido como requisito no funcional en la sección 17— es una consecuencia directa de tratar la resiliencia como principio de diseño y no como un ajuste posterior.

### 8.3 Cloud Enable

Aunque el proyecto no se despliega inicialmente en una nube gestionada, toda la arquitectura debe quedar preparada para migrar hacia Kubernetes o una plataforma cloud sin una reescritura significativa. Esto impone restricciones concretas: no se asume un único host fijo, no se depende del sistema de archivos local para estado que deba sobrevivir a un reinicio del contenedor, y la configuración y los secretos se inyectan de la misma manera sin importar el entorno de ejecución. Cloud Enable es lo que permite que "Despliegue" en la tabla de decisiones esté confirmado como principio —contenedores desde el inicio— y a la vez siga pendiente como entorno final: la plataforma no necesita saber hoy si terminará en Kubernetes, en un proveedor cloud o en un centro de datos propio, siempre que hoy no cierre esa puerta.

### 8.4 Programación Reactiva

Spring WebFlux es la tecnología oficial para todo componente que maneje tráfico HTTP o integraciones remotas. La razón no es estética: el PEP y el PDP dependen de llamadas de red hacia OPA, SurrealDB y el proveedor de identidad en el camino crítico de cada solicitud, y el requisito no funcional de latencia acotada (sección 17) es más alcanzable con un modelo de I/O no bloqueante que reteniendo hilos a la espera de esas respuestas bajo carga concurrente. La contrapartida es una regla estricta: ningún componente bloqueante puede mezclarse en el flujo reactivo sin aislamiento y justificación explícitos. La demostración actual usa `spring-boot-starter-web` y un `HttpClient` bloqueante; esa implementación es evidencia conceptual de interceptación, no una base reactiva, y deberá refactorizarse antes de convertirse en punto de partida de producción.

### 8.5 Arquitectura 12-Factor y factores extendidos

La plataforma se diseña siguiendo 12-Factor App más los factores adicionales relevantes para aplicaciones distribuidas: configuración estrictamente separada del código, procesos sin estado y desechables, tratamiento de OPA, SurrealDB y el proveedor de identidad como servicios adjuntos intercambiables sin cambios de código, logs tratados como flujos de eventos en lugar de archivos gestionados por la aplicación, y paridad entre entornos de desarrollo y producción. Para un componente de seguridad, estos factores no son solo higiene operativa: son la razón por la que se puede sustituir un proveedor de identidad, rotar credenciales o mover el bundle de políticas activo sin tocar el código de dominio, y por la que los eventos de auditoría descritos en la sección 16 pueden tratarse como un flujo observable en lugar de un archivo local frágil.

### 8.6 Clean Code

Todo el código de la plataforma sigue principios de Clean Code: nombres que reflejan el dominio de seguridad y no solo la implementación técnica, funciones pequeñas y con una sola responsabilidad, y bajo acoplamiento entre módulos. En un componente que decide accesos, la legibilidad no es una preferencia de estilo: es lo que permite que un revisor pueda razonar con confianza sobre las condiciones exactas bajo las cuales se concede o se niega un acceso. Código difícil de leer en el PDP o en los adaptadores hacia OPA esconde comportamiento de autorización detrás de complejidad incidental, lo cual es en sí mismo un riesgo de seguridad.

### 8.7 Clean Architecture y Arquitectura Hexagonal

La arquitectura oficial es Clean Architecture, implementada mediante arquitectura hexagonal. Toda dependencia apunta hacia el dominio, y el dominio nunca depende de Spring, de infraestructura, de frameworks, de bases de datos ni de APIs externas. En términos concretos, esto se traduce en las capas descritas en la sección 18.1: el dominio define las invariantes de seguridad —denegación por defecto, aislamiento de tenant, mínimo privilegio— como reglas puras, independientes de cómo se implemente OPA o SurrealDB; los puertos definen el contrato de evaluación de políticas y el contrato de repositorio sobre el modelo de seguridad; y los adaptadores —el cliente OPA, el driver de SurrealDB, el filtro HTTP del PEP— son los únicos lugares donde esas tecnologías concretas aparecen. Esta separación es lo que permite, por ejemplo, probar la lógica de precedencia de políticas (sección 13.5) sin levantar OPA, o cambiar de motor de políticas en el futuro sin reescribir el dominio.

### 8.8 Spring Modulith

La organización interna del proyecto usa Spring Modulith. Los módulos no son una división técnica por capas, sino que representan capacidades de negocio: tenants, aplicaciones, recursos, identidad, asignaciones, políticas de acceso y auditoría, siguiendo los mismos contextos ya identificados en la sección 11 del modelo de dominio. Spring Modulith hace cumplir esos límites de forma verificable —no solo por convención de equipo— evitando que un módulo acceda directamente a las internals de otro, y deja abierta la posibilidad de extraer más adelante un módulo hacia un servicio propio, si su complejidad o sus necesidades de escalado lo justifican, sin que eso implique una reescritura completa de la plataforma.

### 8.9 Pipeline CI/CD

Desde el inicio del proyecto existe integración continua, con al menos compilación, análisis estático, ejecución de pruebas, medición de cobertura y validaciones sobre cada cambio propuesto. Esto es especialmente relevante en un componente de seguridad: una regresión no detectada aquí no se queda contenida en la plataforma, sino que se propaga como una decisión de acceso incorrecta hacia cada aplicación integrada. Por eso el pipeline se construye como parte de la línea base, en la Etapa 0, y no como algo que se agrega "una vez que el proyecto se estabilice".

### 8.10 Estrategia de pruebas

Las pruebas son parte del diseño, no una etapa posterior. El proyecto mantiene una cobertura superior al 80 %, con pruebas unitarias sobre las invariantes del dominio —denegación por defecto, aislamiento de tenant, precedencia de políticas— y pruebas de integración sobre los adaptadores hacia OPA, SurrealDB y el proveedor de identidad. La cobertura no se persigue como métrica de vanidad: el objetivo es que cada rama de decisión relevante —`ALLOW`, `DENY`, `INDETERMINATE`— y cada regla de precedencia entre RBAC, ABAC y REBAC tengan una prueba que la respalde antes de considerarse completa.

### 8.11 Estrategia Git

El proyecto define, desde el inicio, una rama principal protegida, una rama de desarrollo, ramas de feature por historia de usuario, revisión obligatoria mediante Pull Requests y versionamiento explícito del componente. La revisión obligatoria funciona además como un control adicional de cuatro ojos sobre cambios que afectan decisiones de acceso, y el versionamiento explícito es indispensable porque las aplicaciones integradas dependen de la estabilidad del contrato `DecisionAcceso` descrito en la sección 14.2: un cambio incompatible en ese contrato debe ser visible en la versión, no descubrirse en producción.

### 8.12 Línea Base

Antes de escribir la primera funcionalidad de negocio debe existir una línea base arquitectónica que incluya, como mínimo: visión, documentación, diagramas, arquitectura, módulos, estrategia Git, estrategia CI/CD, estrategia de pruebas, y los primeros casos de uso e historias de usuario. Esta línea base es exactamente lo que la Etapa 0 de la estrategia de construcción (sección 19) debe entregar antes de que se declare abierta la Etapa 1; iniciar historias de usuario funcionales sin ella se trata como un riesgo del proyecto, registrado en la sección 20.

---

## 9. Conceptos y arquitectura de referencia

### 9.1 Componentes principales

| Componente | Responsabilidad | No debe hacer |
|---|---|---|
| Cliente | Inicia la solicitud hacia una aplicación protegida. | Decidir sus propios permisos de manera confiable. |
| Aplicación protegida | Implementa su negocio y confía en el resultado de seguridad según el modo de integración. | Duplicar políticas centrales. |
| PEP | Intercepta, extrae contexto básico, solicita decisión y permite/bloquea. | Resolver reglas de negocio complejas ni almacenar políticas como fuente de verdad. |
| PDP / Servicio de autorización | Orquesta la evaluación: obtiene relaciones y atributos, prepara la entrada y solicita evaluación. | Ser un proxy de negocio de las aplicaciones. |
| OPA | Evalúa políticas Rego sobre una entrada determinada y devuelve decisión/razones estructuradas. | Gestionar directamente usuarios, sesiones o persistencia de dominio. |
| SurrealDB | Almacena el modelo de seguridad, relaciones, configuración y, si se decide, auditoría o referencias a ella. | Sustituir por sí sola el motor de políticas. |
| Adaptador de identidad | Valida/normaliza evidencia de identidad proveniente del IdP. | Acoplar el dominio a claims o formatos propios de un solo proveedor. |
| Auditoría | Registra eventos y evidencia de las decisiones. | Registrar tokens, contraseñas u otros secretos. |
| Administración | Permite gestionar configuración, asignaciones y políticas, bajo controles reforzados. | Ser requisito previo para validar el MVP por API. |

### 9.2 Flujo objetivo de una decisión

```text
Cliente
  -> PEP (aplicación, gateway o sidecar)
  -> validación / normalización de identidad
  -> SolicitudAcceso normalizada
  -> PDP / Servicio de autorización
       -> SurrealDB: sujeto, tenant, roles, perfiles, relaciones, recursos
       -> OPA/Rego: evaluación de política
  <- DecisionAcceso (ALLOW / DENY / INDETERMINATE)
  -> Auditoría de seguridad
  -> PEP aplica la decisión
       -> ALLOW: reenvía a aplicación protegida
       -> DENY: responde sin alcanzar la aplicación protegida
```

La decisión interna debería contemplar al menos tres estados: `ALLOW`, `DENY` e `INDETERMINATE` (no fue posible determinar una decisión confiable). Hacia el cliente, el PEP puede convertir `INDETERMINATE` en una negación controlada, normalmente con error 403 o 503 según la causa y la política operativa; el detalle técnico no debe filtrarse.

### 9.3 PEP y PDP

El **PEP** es el componente que hace cumplir el resultado. Puede existir como gateway, proxy inverso, filtro en una aplicación, SDK o sidecar. El **PDP** concentra la decisión y la interacción con OPA. Mantener esta separación permite soportar distintas formas de integración sin duplicar la lógica de autorización; en términos de la arquitectura hexagonal descrita en la sección 8.7, tanto el PEP como el PDP son adaptadores de entrada que invocan casos de uso del dominio, nunca al revés.

La primera demostración implementa un PEP/proxy central. Esa decisión es adecuada para probar heterogeneidad, pero no determina que el producto final solo pueda operar como proxy. La arquitectura debe permitir evaluar, en etapas posteriores, otros adaptadores de enforcement.

---

## 10. Evidencia existente: demostración actual

En `demo-security/` existe una prueba de concepto que demuestra la ubicación de un interceptor transversal delante de aplicaciones de tecnologías diferentes:

- Django, expuesto internamente en el puerto 8001.
- PHP, expuesto internamente en el puerto 8002.
- .NET, expuesto internamente en el puerto 8003.
- Java, expuesto internamente en el puerto 8004.
- Un PEP Java/Spring Boot expuesto en el puerto 8080.

El PEP actual enruta solicitudes a rutas `/django/**`, `/php/**`, `/dotnet/**` y `/java/**`. Un `HandlerInterceptor` registra la llegada de cada solicitud y un proxy HTTP la reenvía a la aplicación configurada. La demostración usa Docker Compose para levantar sus contenedores.

### 10.1 Qué valida la demo

- Que aplicaciones heterogéneas pueden colocarse detrás de un punto de entrada común.
- Que una petición puede ser capturada de manera transversal antes de llegar al destino.
- Que se puede conservar método, cuerpo, parámetros de consulta y la mayoría de cabeceras al reenviar.
- Que el concepto PEP puede demostrarse con un proxy central.

### 10.2 Qué todavía no valida

- Autenticación real ni validación de JWT.
- Autorización basada en roles, atributos o relaciones.
- Llamada a OPA ni ejecución de Rego.
- Persistencia en SurrealDB.
- Decisión `ALLOW`/`DENY` antes del reenvío.
- Auditoría persistente y correlacionada.
- Controles de producción: TLS, límites de tamaño, timeouts completos, reintentos controlados, propagación segura de headers, protección contra *header spoofing*, rate limiting, observabilidad o gestión de errores.
- Reactividad: el PEP usa `spring-boot-starter-web`, `HttpServletRequest` y `java.net.http.HttpClient` bloqueante; no es una implementación WebFlux.

Por tanto, la demo es una **evidencia conceptual de interceptación**, no un prototipo completo de autorización ni una base que deba trasladarse literalmente a producción.

---

## 11. Modelo de dominio inicial

Los diagramas y documentos existentes identifican los siguientes contextos. Son un punto de partida; el modelo físico, los atributos y las cardinalidades finales se definirán desde casos de uso y reglas de negocio. Estos mismos contextos son, además, la base de los módulos de Spring Modulith descritos en la sección 8.8.

### 11.1 Tenants

Entidades identificadas: `Tenant`, `TipoTenant`, `ConfiguracionTenant`, `TenantSuscripcion`.

Responsabilidad: representar la partición organizacional y la configuración de seguridad aplicable. Todo acceso sensible debe poder asociarse con un tenant explícito o con una regla clara para recursos globales.

### 11.2 Aplicaciones

Entidades identificadas: `Aplicacion`, `AplicacionTenant`, `ModuloAplicacion`, `Funcionalidad`.

Responsabilidad: registrar las aplicaciones que delegan seguridad, sus recursos expuestos, configuración de integración, propietarios, estado y asociación con tenants.

### 11.3 Recursos

Entidades identificadas: `Recurso`, `TipoRecurso`, `RecursoJerarquia`.

Responsabilidad: describir elementos protegibles. Un recurso puede ser una ruta, operación, funcionalidad, entidad de negocio o instancia concreta. La jerarquía permite expresar agrupaciones como aplicación -> módulo -> funcionalidad -> acción.

### 11.4 Roles y perfiles

Entidades identificadas: `Rol`, `RolRecurso`, `Perfil`, `PerfilRol`.

Responsabilidad: soportar RBAC. Un rol agrupa autorizaciones sobre recursos y acciones; un perfil agrupa roles para facilitar asignación y administración. Debe definirse si hay herencia de roles, denegaciones explícitas y precedencia entre asignaciones.

### 11.5 Usuarios, identidad y credenciales

Entidades identificadas: `Usuario`, `EstadoUsuario`, `UsuarioCredencial`, `ProveedorIdentidad`, `SesionUsuario`, `TokenJWT`, `Claim`, `TokenConfirmacionEmail`, `TokenResetContrasena`.

Responsabilidad: separar la identidad de dominio del mecanismo de autenticación. El usuario representa el sujeto interno; el proveedor de identidad y las credenciales son mecanismos o enlaces de autenticación. La plataforma no debe registrar tokens JWT completos en auditoría ni depender de que todos los proveedores tengan los mismos claims.

### 11.6 Asignaciones

Entidades identificadas: `UsuarioAplicacionRol`, `UsuarioPerfil`, `PlanRol`, `AsignacionPendienteRol`.

Responsabilidad: expresar la vigencia y el alcance de los privilegios. Una asignación debe contemplar, según el caso, tenant, aplicación, fecha de inicio y vencimiento, estado, origen de la asignación y trazabilidad administrativa.

### 11.7 Políticas de acceso y autorización

Entidades identificadas: `Politica`, `ReglaPolitica`, `CondicionPolitica`, `InterceptorAcceso`, `SolicitudAcceso`, `MotorAutorizacionOPA`, `DecisionAcceso`, `EventoAcceso`.

Responsabilidad: modelar qué política aplica, cómo se versiona y cuál fue el resultado de una evaluación. La fuente ejecutable de políticas será Rego; las entidades de dominio deben servir para su gestión, metadatos, alcance, estado, versión y trazabilidad, sin intentar duplicar innecesariamente toda la semántica de Rego.

### 11.8 Auditoría de seguridad

Entidades identificadas: `EventoAuditoria`, `RegistroAcceso`, `EvidenciaAcceso`, `HashBlockchain`.

Responsabilidad: conservar evidencia de las operaciones relevantes. La retención, inmutabilidad, cifrado, acceso a auditoría y posible encadenamiento de hashes son decisiones posteriores. `HashBlockchain` no debe asumirse como implementación obligatoria.

---

## 12. Estrategia de datos y grafo en SurrealDB

### 12.1 Principio de modelado

SurrealDB será la persistencia principal del dominio de seguridad. El modelo debe usar:

- **Registros/documentos** para entidades con identidad y atributos propios: tenants, aplicaciones, usuarios, roles, recursos, perfiles, políticas y eventos.
- **Relaciones de grafo** para vínculos que necesitan navegarse y evaluarse: pertenencia, asignación, exposición, permiso, propiedad, delegación o jerarquía.
- **Consultas y transacciones** para preservar invariantes críticos, por ejemplo evitar asignaciones incompatibles o cruzadas entre tenants.

No se debe elegir entre documento y grafo por preferencia estética: cada relación debe justificarse por reglas de negocio, necesidad de trazabilidad y patrones de consulta.

### 12.2 Relaciones iniciales candidatas

```text
Usuario --pertenece_a--> Tenant
Usuario --tiene_rol_en--> Aplicacion / Rol
Usuario --tiene_perfil--> Perfil
Perfil --agrupa--> Rol
Rol --autoriza--> Recurso / Accion
Aplicacion --pertenece_a--> Tenant
Aplicacion --expone--> Recurso
Recurso --es_hijo_de--> Recurso
Politica --aplica_a--> Tenant / Aplicacion / Recurso
IdentidadExterna --representa--> Usuario
Sesion --corresponde_a--> Usuario
```

La notación anterior es conceptual, no define aún los nombres físicos de tablas, aristas ni esquemas de SurrealDB.

### 12.3 Consultas que el modelo debe poder responder

- ¿A qué tenant pertenece este sujeto y está activo?
- ¿La aplicación solicitante está registrada y habilitada para ese tenant?
- ¿Qué recurso y acción representan la ruta u operación solicitada?
- ¿Qué roles y perfiles vigentes tiene el sujeto en esta aplicación y tenant?
- ¿Existe una relación válida entre sujeto y recurso que satisfaga REBAC?
- ¿Qué políticas aplican al contexto y cuál es su versión activa?
- ¿Qué decisiones se tomaron para una solicitud/correlación específica?
- ¿Qué accesos y asignaciones deben revisarse si se revoca un rol, usuario o relación?

### 12.4 Invariantes iniciales a validar

- Un usuario no debe obtener privilegios de un tenant distinto salvo que exista una relación explícita y autorizada.
- Un rol y un recurso deben estar acotados a un tenant/aplicación o declararse globales de manera inequívoca.
- Las asignaciones inactivas, vencidas o revocadas no deben participar en la decisión.
- Cada política publicada debe tener versión, estado, responsable y fecha de vigencia.
- Un evento de auditoría debe mantener su correlación aun si se eliminan o anonimizan datos operativos conforme a retención.
- Los identificadores externos nunca deben permitir inferir autorización sin comprobar el alcance de tenant y aplicación.

---

## 13. Modelo de autorización

### 13.1 RBAC

RBAC resuelve casos donde un rol habilita acciones conocidas: por ejemplo, un rol `ADMINISTRADOR_SEGURIDAD` puede gestionar políticas de su tenant. Se apoya en roles, recursos, acciones y asignaciones vigentes.

### 13.2 ABAC

ABAC incorpora atributos al contexto, tales como nivel de riesgo, horario, tipo de canal, estado del usuario, clasificación del recurso, monto de una operación o pertenencia organizacional. Estos atributos deben tener procedencia conocida y validable; no se debe confiar ciegamente en valores enviados por el cliente.

### 13.3 REBAC

REBAC verifica relaciones: un usuario puede editar un documento porque es propietario, pertenece al equipo responsable o fue delegado por el propietario. El grafo en SurrealDB es particularmente relevante para obtener estas relaciones, pero OPA continúa siendo quien evalúa la regla de decisión.

### 13.4 PBAC y OPA/Rego

PBAC expresa criterios en políticas. OPA debe recibir un objeto de entrada normalizado, no tener que consultar libremente el dominio ni recibir el token sin procesar. El PDP será responsable de resolver datos y construir esa entrada.

Ejemplo conceptual de entrada a OPA:

```json
{
  "request": {
    "id": "correlation-id",
    "method": "POST",
    "path": "/orders/123/approve",
    "time": "2026-07-30T12:00:00Z",
    "channel": "web"
  },
  "subject": {
    "id": "user:123",
    "tenant": "tenant:acme",
    "roles": ["approver"],
    "attributes": {"status": "ACTIVE"}
  },
  "resource": {
    "type": "order",
    "id": "123",
    "action": "approve",
    "attributes": {"owner": "user:456", "amount": 200000}
  },
  "relationships": ["subject_is_manager_of_resource_owner"],
  "policy": {"scope": "tenant:acme"}
}
```

El ejemplo no es un contrato definitivo ni autoriza revelar atributos financieros al motor sin una clasificación de datos. Sirve para fijar que OPA evalúa contexto explícito, no una colección ambigua de variables.

### 13.5 Precedencia de políticas: decisión pendiente crítica

Debe definirse antes de producción:

- si una denegación explícita prevalece sobre una autorización;
- cómo se combinan varias políticas aplicables;
- qué ocurre si no hay política aplicable;
- cómo se tratan conflictos entre RBAC, ABAC y REBAC;
- si existen permisos de emergencia y cómo se auditan;
- cómo se publica, revoca o revierte una versión de política.

Como regla provisional de diseño: **denegación por defecto y denegación explícita con prioridad sobre autorización**, salvo que un caso de negocio aprobado defina otra semántica.

---

## 14. Contratos de integración propuestos

### 14.1 Solicitud de autorización normalizada

El PEP debe poder construir o enviar al PDP una `SolicitudAcceso` con al menos:

| Grupo | Campos mínimos |
|---|---|
| Trazabilidad | `requestId`, `correlationId`, fecha/hora, origen de la petición. |
| Sujeto | identificador interno o evidencia para resolverlo, tenant, estado de autenticación. |
| Aplicación | identificador de la aplicación protegida y entorno. |
| Recurso | tipo, identificador si aplica, ruta/operación y acción solicitada. |
| Contexto | método HTTP, canal, IP solo si está justificada, sesión, atributos confiables. |
| Integridad | token o prueba de identidad validable, sin registrar secretos en claro. |

### 14.2 Respuesta de decisión

El PDP debe devolver un objeto estructurado, no solo un booleano:

| Campo | Propósito |
|---|---|
| `decision` | `ALLOW`, `DENY` o `INDETERMINATE`. |
| `decisionId` | Identificador único para auditoría y soporte. |
| `reasonCode` | Código estable y no sensible: `TOKEN_INVALID`, `POLICY_DENY`, `TENANT_MISMATCH`, etc. |
| `policyReferences` | Identificadores/versiones de políticas evaluadas cuando sea seguro exponerlos al consumidor interno. |
| `obligations` | Acciones que el PEP o aplicación debe cumplir, si se adopta este concepto: enmascarar datos, requerir MFA, limitar alcance. |
| `cacheDirective` | Indicación opcional y controlada sobre si una decisión podría reutilizarse. |
| `correlationId` | Permite unir decisión y evento de auditoría. |

### 14.3 Modos de integración a evaluar

| Modo | Ventaja | Riesgo / decisión necesaria |
|---|---|---|
| Proxy/gateway PEP | Centraliza tráfico y facilita integrar tecnologías heterogéneas; ya existe una demo. | Puede convertirse en cuello de botella y requiere controles fuertes de proxy. |
| Filtro/middleware en aplicación | Conserva contexto local y reduce saltos de red. | Requiere SDK/adaptadores por lenguaje y disciplina de adopción. |
| Sidecar | Aísla la preocupación en despliegues con contenedores. | Incrementa complejidad operativa. |
| Llamada directa al PDP | Sencilla para algunos servicios internos. | La aplicación debe aplicar correctamente la decisión; no sustituye un PEP. |

El MVP puede conservar el proxy PEP para la demostración, pero la interfaz hacia el PDP debe diseñarse para que los demás modos sean posibles sin reescribir la lógica de decisión.

---

## 15. Autenticación e identidad

La autenticación se plantea como **delegada**. La plataforma debe abstraer proveedores de identidad y normalizar la evidencia recibida a un contexto interno. Podría integrar OAuth 2.0/OIDC u otros mecanismos aprobados posteriormente, pero no debe asumir que todos los sistemas utilizarán el mismo proveedor.

Flujo conceptual:

1. El PEP o aplicación detecta que no existe una identidad válida.
2. Se inicia o delega el flujo con el proveedor de identidad configurado.
3. La plataforma valida firma, emisor, audiencia, vencimiento y demás condiciones aplicables al token o evidencia.
4. Se resuelve la identidad interna, tenant y estado del sujeto.
5. Se construye un contexto de autorización independiente del formato original de claims.

Reglas de seguridad:

- No modificar el token original para convertirlo en fuente de verdad.
- No usar claims sin validar como autorización suficiente si requieren contraste con asignaciones vigentes.
- No persistir tokens completos, contraseñas ni secretos en auditoría.
- Mantener separación entre autenticación (quién es) y autorización (qué puede hacer).

---

## 16. Auditoría, observabilidad y explicación

### 16.1 Evento mínimo de auditoría

Cada evaluación debe poder generar un evento con:

- `eventId`, `decisionId`, `requestId` y `correlationId`.
- Fecha y hora con zona/UTC.
- Identificador pseudonimizado o controlado del sujeto.
- Tenant, aplicación, recurso y acción.
- Decisión, código de motivo y estado técnico.
- Referencias y versiones de políticas evaluadas.
- Versión del servicio/PDP y, si aplica, del bundle de políticas.
- Duraciones de evaluación y dependencias principales, sin secretos.
- Resultado de enforcement cuando sea posible distinguirlo de la decisión.

### 16.2 Propósito de la evidencia

La auditoría debe responder: quién intentó acceder, a qué, bajo qué contexto, qué reglas participaron, qué se decidió, por qué y cuándo. También debe permitir diferenciar una denegación legítima de un fallo de infraestructura.

### 16.3 Retención y protección

Antes de implementación productiva se debe definir retención, clasificación de datos, cifrado, control de acceso a auditoría, anonimización/depuración y estrategia de inmutabilidad. Los logs de seguridad son información sensible; registrar más datos no equivale automáticamente a auditar mejor.

---

## 17. Requisitos no funcionales

### Seguridad

- Validación estricta de identidad y aislamiento por tenant.
- Comunicación cifrada entre componentes en entornos no locales.
- Gestión segura de secretos y claves fuera del código fuente.
- Protección contra reenvío de cabeceras no confiables y suplantación de contexto.
- Validación de entradas y límites de tamaño/tiempo en PEP y PDP.
- Principio de mínimo privilegio para cuentas técnicas y administración.

### Rendimiento y disponibilidad

- La autorización debe añadir latencia acotada y medible.
- Deben existir timeouts, límites de concurrencia y comportamiento definido ante la caída de OPA, SurrealDB o el IdP.
- El estado de la plataforma debe ser mínimo para permitir escalado horizontal cuando corresponda.
- La caché, si se introduce, debe incluir tenant, sujeto, recurso, acción, contexto relevante y versión de política en su clave o estrategia de invalidación.
- El modelo reactivo del PEP y del PDP debe traducirse en throughput sostenido bajo concurrencia real, no solo en un cambio superficial de API; una migración a WebFlux que retenga hilos bloqueados en la práctica no cumple este requisito.

### Mantenibilidad

- APIs versionadas y contratos documentados.
- Políticas revisables, probables y versionadas como artefactos.
- Separación entre dominio, aplicación, infraestructura e integraciones, siguiendo la arquitectura hexagonal descrita en la sección 8.7.
- Módulos delimitados mediante Spring Modulith, con dependencias entre módulos explícitas y verificables.
- Pruebas automáticas para políticas, contratos, seguridad y flujos extremos, con cobertura superior al 80 % según la estrategia de pruebas de la sección 8.10.
- Pipeline de integración continua activo desde el inicio del proyecto, con compilación, análisis estático, pruebas y cobertura validados en cada cambio.
- Flujo de trabajo Git con rama principal protegida, rama de desarrollo, ramas de feature, revisión obligatoria por Pull Request y versionamiento explícito del componente.

### Observabilidad

- Correlación extremo a extremo.
- Métricas de decisiones, denegaciones, errores, latencia y disponibilidad de dependencias.
- Trazas distribuidas donde el entorno lo permita.
- Alertas sobre picos de denegación, errores de validación o fallos de dependencias.

---

## 18. Arquitectura lógica propuesta

```text
                    +----------------------------+
                    | Proveedor(es) de identidad  |
                    +-------------+--------------+
                                  |
Cliente -> [PEP / adaptador] -> [Servicio de identidad]
             |                       |
             | SolicitudAcceso        v
             +------------------> [PDP / Autorización] ----> [OPA + Rego]
                                      |        |
                                      |        +--------------> [Auditoría]
                                      v
                                [SurrealDB]
                                      |
                                      v
                              DecisionAcceso
                                      |
PEP -- ALLOW --> Aplicación protegida (Java, .NET, Python, PHP, ...)
PEP -- DENY  --> Respuesta controlada al cliente
```

### 18.1 Módulos y capas internas (Clean Architecture + Spring Modulith)

1. **Entrada/adapter:** REST, PEP, manejo de HTTP, autenticación de cliente y validación sintáctica.
2. **Aplicación:** casos de uso como `EvaluarAcceso`, `PublicarPolitica`, `AsignarRol`, `RegistrarAplicacion`.
3. **Dominio:** entidades, invariantes y contratos de repositorio sobre seguridad.
4. **Políticas:** adaptador OPA, preparación de input Rego y lectura de decisión.
5. **Infraestructura:** SurrealDB, proveedor de identidad, mensajería, auditoría, caché y observabilidad.

Las capas de dominio y aplicación forman el núcleo hexagonal descrito en la sección 8.7: no dependen de Spring, OPA ni SurrealDB, y son las únicas capas que definen puertos. Sobre esa misma base, Spring Modulith organiza el código en módulos por capacidad de negocio —tenants, aplicaciones, recursos, identidad, asignaciones, políticas, auditoría— en lugar de organizarlo únicamente por capa técnica, de modo que cada módulo agrupe su propio dominio, sus propios casos de uso y sus propios adaptadores.

Spring WebFlux es la tecnología oficial para los componentes de entrada y para los adaptadores que integran OPA, SurrealDB y el proveedor de identidad, dado su modelo de I/O no bloqueante. Se mantiene, sin excepción, la regla de no mezclar APIs bloqueantes dentro del flujo reactivo sin aislamiento o justificación explícitos. La prueba de concepto actual requiere refactorización antes de convertirse en base de producción, conforme a lo indicado en la sección 6.2.

---

## 19. Estrategia de construcción por etapas

### Etapa 0 - Descubrimiento y línea base arquitectónica

- Convertir este documento en fuente de decisiones controlada.
- Definir actores, casos de uso prioritarios y amenazas iniciales.
- Confirmar límites entre identidad, autorización, auditoría y administración.
- Establecer la línea base de ingeniería: estructura de módulos bajo Spring Modulith, estrategia Git (rama principal, rama de desarrollo, feature branches y flujo de Pull Requests) y pipeline de integración continua con compilación, análisis estático, pruebas y cobertura.
- Definir la estrategia de pruebas objetivo (cobertura superior al 80 %, pruebas unitarias y de integración) antes de escribir las primeras historias de usuario.
- Realizar una prueba técnica de SurrealDB para relaciones, consultas, transacciones, índices y aislamiento por tenant.
- Realizar una prueba OPA/Rego con decisiones representativas.
- Realizar una prueba técnica de Spring WebFlux para el PEP y el PDP, confirmando el aislamiento de dependencias bloqueantes.

### Etapa 1 - Camino crítico (MVP)

- Registrar una aplicación y recurso protegido.
- Resolver una identidad de prueba validada.
- Implementar `SolicitudAcceso` y `DecisionAcceso`.
- Persistir tenant, usuario, rol, asignación y recurso básicos en SurrealDB.
- Evaluar una política Rego RBAC simple mediante OPA.
- Convertir `ALLOW`/`DENY` en enforcement en el PEP.
- Registrar auditoría con correlación.
- Integrar al menos una aplicación de referencia a través de la demo.

### Etapa 2 - Contexto y administración básica

- Catálogo de aplicaciones, recursos, roles, perfiles y asignaciones.
- Gestión de versiones y publicación controlada de políticas.
- Validación de JWT/OIDC con adaptador de identidad.
- API administrativa protegida y pruebas de contrato.
- Métricas, trazas y manejo explícito de errores/dependencias.

### Etapa 3 - ABAC y REBAC

- Definir atributos confiables y fuentes de cada atributo.
- Modelar relaciones necesarias en SurrealDB.
- Probar consultas de grafo y su costo con escenarios reales.
- Implementar políticas Rego que combinen roles, atributos y relaciones.
- Incorporar mecanismos de explicación de decisiones y pruebas de regresión de políticas.

### Etapa 4 - Endurecimiento y validación del trabajo de grado

- Análisis de amenazas y pruebas de seguridad.
- Pruebas de carga, latencia y resiliencia.
- Estrategia de caché, solo si una medición demuestra que es necesaria.
- Integración con varias aplicaciones de referencia.
- Frontend administrativo, si aporta valor a la validación del proyecto.
- Documentación académica, resultados experimentales y comparación contra objetivos.

---

## 20. Riesgos principales y mitigaciones iniciales

| Riesgo | Impacto | Mitigación inicial |
|---|---|---|
| Alcance excesivo: intentar construir IdP, gateway, IAM y auditoría inmutable completos. | Alto | Priorizar el camino autorización-PEP-OPA-auditoría; declarar fuera de alcance lo no esencial. |
| Modelo de grafo sin casos de uso concretos. | Alto | Derivar cada relación de una pregunta de autorización y medirla en una prueba técnica. |
| Políticas ambiguas o contradictorias. | Alto | Definir semántica de precedencia, políticas de prueba y revisión antes de publicar. |
| Fuga de datos sensibles hacia logs, OPA o aplicaciones. | Alto | Clasificación de atributos, minimización de datos, redacción de logs y controles de acceso. |
| PEP proxy inseguro o frágil. | Alto | Validar headers, TLS, límites, timeouts, errores y autenticación entre componentes. |
| Usar caché con decisiones obsoletas. | Alto | Empezar sin caché; si se agrega, versionar/invalidate por cambios de política y asignación. |
| Acoplamiento a un IdP o formato de token. | Medio/alto | Adaptador de identidad y modelo interno normalizado. |
| Adoptar WebFlux con dependencias bloqueantes. | Medio | Prueba técnica, clientes reactivos y límites claros de aislamiento. |
| Falta de evidencia de evaluación. | Medio | Auditoría y métricas desde el MVP. |
| Administración de políticas sin gobernanza. | Alto | Versionado, revisiones, pruebas, promoción y reversión controlada. |
| Iniciar historias de usuario funcionales sin línea base arquitectónica (módulos, Git, CI/CD, pruebas) consolidada. | Alto | Bloquear el inicio de la Etapa 1 hasta cerrar formalmente la Etapa 0. |
| Adoptar Clean Architecture y Spring Modulith solo de forma nominal, sin disciplina real de límites entre módulos. | Medio/alto | Verificación de límites de módulo como parte del pipeline CI/CD, no solo como convención de equipo. |

---

## 21. Preguntas que deben resolverse antes de cerrar arquitectura detallada

1. ¿Cuál es el primer dominio/aplicación real que servirá de caso de validación?
2. ¿Qué proveedor o proveedores de identidad se integrarán en el MVP?
3. ¿Qué actor administrará tenants, aplicaciones, roles y políticas, y con qué separación de funciones?
4. ¿Cuál es la semántica formal de prioridad entre `ALLOW`, `DENY`, ausencia de política y error técnico?
5. ¿Qué atributos de negocio puede aportar cada aplicación y cuáles son fuentes confiables?
6. ¿Qué relaciones REBAC son indispensables en el primer caso de uso?
7. ¿Cómo se identifican de manera estable aplicación, recurso y acción entre tecnologías distintas?
8. ¿Qué datos de auditoría requieren retención, y por cuánto tiempo?
9. ¿Qué comportamiento se requiere cuando OPA, SurrealDB o el proveedor de identidad no están disponibles?
10. ¿El PEP inicial será gateway/proxy central únicamente o también se desarrollarán SDK/filtros?
11. ¿Qué requisitos de latencia, volumen y disponibilidad son exigibles para la demostración y para una versión futura?
12. ¿Qué tecnología de frontend aporta más valor una vez estén definidos los flujos administrativos?

---

## 22. Cómo usar este documento como entrada para futuras IAs

Una IA que reciba este documento debe:

1. Tratar la tabla de decisiones como autoridad sobre la tecnología y diferenciarla de los pendientes.
2. No asumir que la demo implementa autorización completa: solo demuestra interceptación/proxy.
3. Proponer historias de usuario trazables a objetivos y etapas, con criterios de aceptación verificables.
4. Antes de diseñar el esquema físico de SurrealDB, solicitar o derivar casos de autorización concretos y validar las invariantes listadas.
5. Mantener la separación PEP/PDP/OPA y no incluir reglas de negocio en el interceptor.
6. Diseñar políticas, auditoría y contratos de integración junto con la funcionalidad, no como una tarea posterior.
7. Señalar cualquier decisión que contradiga este documento, especialmente si reintroduce SQL como fuente de verdad o trata WebFlux/frontend como decisiones ya cerradas cuando el frontend sigue pendiente.
8. Actualizar este documento cuando una decisión pendiente sea aprobada; no convertir hipótesis en hechos de forma silenciosa.
9. Diseñar y proponer código, módulos y pipelines conforme a la sección 8 (Principios de Ingeniería y Construcción); no proponer atajos que la contradigan sin señalarlo explícitamente.

### 22.1 Prompts de continuación sugeridos

- "A partir de `DOCUMENTO_BASE_PLATAFORMA_SEGURIDAD.md`, crea el backlog del MVP con épicas, historias de usuario, criterios de aceptación y dependencias."
- "Diseña un diagrama C4 de contexto y contenedores respetando las decisiones confirmadas del documento base."
- "Propón un modelo inicial de SurrealDB para el MVP, justificando registros, relaciones, índices e invariantes."
- "Define el contrato OpenAPI para `EvaluarAcceso` y los códigos de error, sin incluir secretos."
- "Escribe casos de prueba Rego para RBAC inicial y casos de denegación por defecto."
- "Haz un análisis de amenazas del flujo PEP-PDP-OPA-SurrealDB y prioriza mitigaciones."
- "Diseña la estructura de módulos de Spring Modulith y el esqueleto de la línea base (Git, CI/CD, pruebas) descrita en la sección 8."

---

## 23. Glosario

| Término | Definición en este proyecto |
|---|---|
| ABAC | Control de acceso basado en atributos del sujeto, recurso, acción y entorno. |
| Acción | Operación protegida sobre un recurso: leer, crear, aprobar, eliminar, etc. |
| Aplicación protegida | Sistema consumidor que delega una parte del control de seguridad a la plataforma. |
| Arquitectura Hexagonal | Patrón que separa el dominio de sus adaptadores mediante puertos, usado para implementar Clean Architecture en este proyecto. |
| Auditoría | Evidencia estructurada de eventos y decisiones de seguridad. |
| Claim | Afirmación incluida en una evidencia de identidad, como un JWT; debe validarse y normalizarse. |
| Cloud Enable | Preparación de la arquitectura para migrar a Kubernetes o una plataforma cloud, sin que ello implique desplegar en nube desde el MVP. |
| Cloud Native | Conjunto de principios de construcción: servicios desacoplados, configuración externa, observabilidad, contenedores, escalabilidad horizontal y resiliencia. |
| Harness Engineering | Estrategia de ingeniería del proyecto orientada a calidad, mantenibilidad, automatización, verificabilidad y evolución. |
| IdP | Proveedor de identidad que autentica usuarios o emite evidencia de identidad. |
| Línea base | Conjunto mínimo de visión, documentación, arquitectura, módulos, estrategias de Git, CI/CD y pruebas que debe existir antes de iniciar historias de usuario funcionales. |
| OPA | Motor de evaluación de políticas Open Policy Agent. |
| PEP | Punto que intercepta una solicitud y aplica la decisión de acceso. |
| PDP | Componente que toma u orquesta una decisión de acceso. |
| PBAC | Control de acceso guiado por políticas. |
| Rego | Lenguaje utilizado por OPA para expresar políticas. |
| REBAC | Control de acceso basado en relaciones entre entidades. |
| Recurso | Elemento que se protege: ruta, función, entidad, documento o instancia de negocio. |
| RBAC | Control de acceso basado en roles. |
| SolicitudAcceso | Representación normalizada de una petición que será evaluada. |
| Spring Modulith | Marco usado para organizar el proyecto en módulos internos verificables, alineados con capacidades de negocio. |
| SurrealDB | Base de datos principal decidida para la plataforma; se usará para datos y relaciones de seguridad. |
| Tenant | Límite organizacional o de aislamiento lógico para datos y autorizaciones. |

---

## 24. Fuentes consolidadas y trazabilidad

| Artefacto | Aporte incorporado |
|---|---|
| `SintesisComponenteSeguridad.md` | Visión de plataforma, separación por capas, PEP/PDP, OPA/Rego, modelos RBAC/ABAC/REBAC/PBAC, autenticación delegada, caché y auditoría. |
| `ContextosSeguridad.md` | Contextos iniciales y entidades: tenants, aplicaciones, recursos, roles, perfiles, usuarios, credenciales y asignaciones. |
| `ContextosSeguridad.drawio` y `.xml` | Contextos de identidad/autenticación, autorización, políticas y auditoría; entidades adicionales y sus dependencias. |
| `demo-security/README.md` | Intención de demostración e interceptor filter. |
| `demo-security/pep/` | Implementación actual del PEP/proxy y sus límites técnicos. |
| `demo-security/docker-compose.yml` | Topología de demostración con Django, PHP, .NET, Java y PEP. |
| `Sintesis_del_componente_de_seguridad.pdf` | Archivo de referencia presente; no fue posible extraer texto automáticamente en este entorno. La síntesis Markdown fue usada como fuente editable equivalente cuando coincidía. |
| `Decisin_estratgica_de_base_de_datos.pdf` | Archivo de referencia presente; no fue posible extraer texto automáticamente en este entorno. La decisión actual comunicada para este documento es SurrealDB como persistencia principal. |
| `EjemploContextos.drawio` y `Modelo de datos Parqueadero.drawio` | Referencias metodológicas de modelado por contextos; no se trasladan entidades del dominio de parqueadero al producto de seguridad. |
| Decisiones de ingeniería y construcción comunicadas por el autor del proyecto | Formalización de Harness Engineering, Cloud Native, Cloud Enable, programación reactiva con Spring WebFlux, 12-Factor extendido, Clean Code, Clean Architecture mediante arquitectura hexagonal, Spring Modulith, pipeline CI/CD, estrategia de pruebas, estrategia Git y línea base como decisiones confirmadas. |

---

## 25. Control de cambios del documento

| Fecha | Cambio |
|---|---|
| 2026-07-30 | Creación del documento fundacional. Se consolida material existente y se registra SurrealDB como decisión principal comunicada. |
| 2026-07-30 | Se formalizan como decisiones confirmadas los principios de ingeniería y construcción: Harness Engineering, Cloud Native, Cloud Enable, programación reactiva con Spring WebFlux, 12-Factor App y factores extendidos, Clean Code, Clean Architecture mediante arquitectura hexagonal, Spring Modulith, pipeline CI/CD, estrategia de pruebas con cobertura superior al 80 %, estrategia Git y línea base. Se incorpora la sección 8, y se actualizan en consecuencia el resumen ejecutivo, los objetivos, el alcance, la tabla de decisiones, los requisitos no funcionales, la arquitectura lógica, la estrategia de construcción por etapas, los riesgos y el glosario. |

Este documento debe evolucionar con decisiones explícitas. Cuando se apruebe un pendiente, se recomienda actualizar la tabla de la sección 6, la sección técnica afectada y esta bitácora para conservar trazabilidad.
