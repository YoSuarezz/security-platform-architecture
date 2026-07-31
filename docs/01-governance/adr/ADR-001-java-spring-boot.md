# ADR-001: Java y Spring Boot como base de la plataforma

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6; ADR-002, ADR-008, ADR-009 |

## Contexto

La Plataforma Central de Seguridad es un componente de larga vida que debe:

- Exponer APIs e integraciones confiables (PEP, PDP, Auditoría).
- Integrarse con OPA, SurrealDB e IdPs empresariales.
- Soportar pruebas automatizadas, análisis estático y evolución durante años.
- Ser evaluable académicamente y profesionalmente como arquitectura seria.

El lenguaje y el framework definen el ecosistema de talento, librerías de seguridad, tooling de CI y la forma en que se materializan Clean Architecture y Modulith.

## Por qué se tomó la decisión

1. **Madurez del ecosistema de seguridad** — Java tiene librerías, patrones y experiencia consolidada en autenticación, TLS, JWT y hardening de servicios.
2. **Spring Boot como estándar industrial** — reduce tiempo de configuración de producción (actuators, config, testing) sin forzar acoplar el dominio al framework (eso lo resuelve ADR-008).
3. **Alineación con el Documento Base** — decisión confirmada; no renegociable en Fase 0.
4. **Separación runtime plataforma vs apps protegidas** — las apps consumidoras pueden ser Django, PHP, .NET o Java; no comparten el runtime de la plataforma.

## Decisión

Usar **Java** (LTS vigente del proyecto, p. ej. 21) como lenguaje principal y **Spring Boot** como framework base de todos los contenedores Java de la plataforma: **PEP**, **PDP** y **Auditoría**.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte / no elección |
|---|---|---|
| **Kotlin + Spring Boot** | Compatible con el stack | Añade curva y heterogeneidad sin beneficio de dominio; se puede reconsiderar después sin cambiar ADRs de arquitectura |
| **Go** | Excelente para proxies de alto rendimiento | Ecosistema de dominio rico / hexagonal / Modulith menos alineado al Documento Base; reescritura total del plan |
| **.NET** | Maduro en enterprise | Fuera del stack confirmado; fragmentaría el equipo respecto a OPA/SurrealDB demos Java |
| **Node.js / Nest** | Rápido para APIs | Débil para el perfil de plataforma de seguridad de largo plazo definido en el Documento Base |
| **Quarkus / Micronaut** | Cloud-native Java | Válidos, pero Spring Boot es el estándar del documento y del talento esperado |

## Consecuencias

### Positivas

- Un solo lenguaje para PEP, PDP y Auditoría → convenciones, CI y reviews uniformes.
- Ecosistema Spring alineado con WebFlux (ADR-002), Modulith (ADR-009) y hexagonal (ADR-008).
- Apps protegidas heterogéneas sin acoplarse al runtime Java.

### Costos / riesgos

- Disciplina obligatoria para no filtrar Spring al dominio (ADR-008).
- Curva WebFlux si el equipo viene de MVC bloqueante (mitigado por PoC Etapa 0).

### Implicaciones

- Los tres jars Java usan Spring Boot.
- No se introduce un segundo lenguaje en el núcleo sin ADR que lo superseda.
