# ADR-009: Spring Modulith en todos los contenedores Java

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §8.8, §11; Dependency Map; ADR-001, ADR-008 |

## Contexto

Organizar el código solo por capas técnicas (`controller`, `service`, `repository`) produce un monolito donde cualquier clase puede llamar a cualquier otra. En una plataforma de seguridad eso es peligroso: Autorización termina escribiendo catálogos, Auditoría importa internals de políticas, el PEP acumula reglas.

Spring Modulith permite módulos por **capacidad**, con límites **verificables en CI**, y deja abierta la extracción futura a microservicio sin reescribir el dominio.

Una lectura inicial limitaba Modulith “casi solo al PDP”. Eso deja al PEP y a Auditoría como jars sin disciplina modular — exactamente donde también hay acoplamientos peligrosos (normalización vs enforcement vs cliente PDP; ingesta vs consulta vs retención de auditoría).

## Por qué se tomó la decisión

1. **El Documento Base exige Modulith** como organización interna, no como adorno del PDP.
2. **Misma disciplina en todos los jars Spring Boot** — PEP, PDP y Auditoría evolucionan con las mismas reglas de equipo y los mismos gates de CI.
3. **Alineación a Bounded Contexts** — en el PDP, un módulo ≈ un BC; en PEP/Auditoría, módulos por capacidad interna del contenedor.
4. **Evitar “Modulith nominal”** — el Dependency Map + `ApplicationModuleTest` fallan el build si se viola una dependencia prohibida.
5. **Extracción futura** — un módulo puede convertirse en servicio sin rediseñar el hexágono.

## Decisión

Usar **Spring Modulith** en **todos** los contenedores Java de la plataforma:

| Contenedor C4 | ¿Modulith? | Módulos (resumen) |
|---|---|---|
| **PDP** | Sí | 10 módulos de dominio (`tenants`…`autorizacion`) + `commons` (shared kernel, no módulo) |
| **PEP** | Sí | Módulos de capacidad: `ingress`, `normalization`, `decision-client`, `enforcement` (+ `commons` de contratos) |
| **Auditoría** | Sí | Módulos: `ingestion`, `query`, `retention` (o equivalente) + `commons` de eventos |

OPA, SurrealDB y Keycloak **no** son apps Spring Modulith (son procesos/infra externos).

### Principios

1. Módulos = capacidades, no capas técnicas sueltas.
2. Dependencias solo hacia APIs públicas de otros módulos / published language.
3. Hexagonal **dentro** de cada módulo (ADR-008).
4. Verificación automática en CI (ADR-011).

Detalle operativo: [`docs/03-architecture/modulith-dependency-map.md`](../../03-architecture/modulith-dependency-map.md).

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Monolito por capas sin Modulith** | Rápido | Límites no verificables; riesgo explícito del Documento Base §20 |
| **Microservicios desde el día 1 (11 servicios)** | “Puro” | Ops prematura; distributed monolith sin fronteras de dominio maduras |
| **Modulith solo en PDP** | Parecía pragmático | Disciplina desigual; PEP/Auditoría acumulan acoplamiento invisible |
| **JPMS / ArchUnit solo** | Útil | No reemplaza el modelo de módulos de Spring ni la extracción gradual |
| **Modulith en PEP + PDP + Auditoría** | **Elegida** | Consistencia + CI uniforme + alineación al Documento Base |

## Consecuencias

### Positivas

- Misma herramienta y convención en los tres jars.
- Gates de CI idénticos (falla el build ante dependencia ilegal).
- Preparación real para extraer módulos (p. ej. `auditoria` ya es contenedor; módulos internos siguen siendo ordenados).

### Costos / riesgos

- Más estructura de paquetes en el PEP (aunque sea delgado).
- Hay que mantener tres dependency maps lógicos (uno por contenedor) en el mismo documento.
- Curva Spring Modulith para el equipo.

### Implicaciones

- Actualizar C4 L2/L3 docs: “Spring Boot + WebFlux + Modulith” en PEP, PDP y Auditoría.
- Fase 0.G: `ApplicationModuleTest` por contenedor.
- El BC de dominio sigue siendo 11; la proyección a módulos del PDP es 10 + auditoría en otro jar Modulith.
