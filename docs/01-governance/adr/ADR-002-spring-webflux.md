# ADR-002: Spring WebFlux para todo el tráfico HTTP e I/O remoto

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6.2, §8.4; ADR-001; PoC WebFlux Etapa 0 |

## Contexto

En el camino crítico de autorización, una sola solicitud del cliente puede disparar varias llamadas remotas encadenadas:

```text
Cliente → PEP → PDP → IdP / SurrealDB / OPA → Auditoría
```

Bajo concurrencia, un modelo bloqueante (un hilo por request esperando I/O) satura el pool de hilos, aumenta latencia p99 y degrada el PEP — exactamente el componente que debe permanecer disponible.

El Documento Base exige programación reactiva no como preferencia estética, sino como respuesta a ese perfil de carga.

## Por qué se tomó la decisión

1. **El cuello de botella es I/O, no CPU** — esperar OPA/DB/IdP no debe ocupar un hilo del servidor.
2. **PEP y PDP son proxies/orquestadores** — el patrón natural es no bloqueante.
3. **Consistencia de stack** — mezclar MVC bloqueante en PEP y WebFlux en PDP crea dos modelos mentales y bugs de “llamar bloqueante dentro de reactivo”.
4. **Preparación Cloud Native** — servicios elásticos bajo carga concurrente sin overprovisioning de hilos.

## Decisión

Todo componente que maneje **tráfico HTTP o I/O remoto** en la plataforma se construye con **Spring WebFlux** (Project Reactor + Netty).

Reglas operativas:

1. Cliente HTTP hacia PDP/OPA/IdP/Auditoría: **WebClient** (u otro cliente reactivo equivalente), no `RestTemplate` / `HttpClient` bloqueante en el flujo principal.
2. Adaptadores a SurrealDB: API no bloqueante o aislamiento explícito si el driver lo exige (documentado y justificado).
3. Está **prohibido** introducir llamadas bloqueantes en el camino crítico sin aislamiento (`boundedElastic` u homólogo) **y** ADR/justificación en el PR.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Spring MVC + pool de hilos grande** | Funciona en demos | No escala con el fan-out OPA/DB/IdP; contradice §8.4 |
| **WebFlux solo en PEP, MVC en PDP** | Reduce alcance | Dos paradigmas; el PDP es quien más I/O remoto hace |
| **Vert.x / Netty puro** | Alto rendimiento | Pierde productividad Spring Boot/Modulith del Documento Base |
| **Kotlin coroutines** | Alternativa válida | Fuera del stack Java confirmado (ADR-001) |
| **WebFlux en todos los contenedores Java** | **Elegida** | Un solo modelo de concurrencia |

## Consecuencias

### Positivas

- Mejor uso de recursos bajo concurrencia en el camino crítico.
- Modelo único para PEP, PDP y Auditoría.
- Alineado a Cloud Enable (ADR-010).

### Costos / riesgos

- Curva de aprendizaje (backpressure, schedulers, testing reactivo).
- La demo histórica MVC/HttpClient **no** es base de producción.
- Debe existir **PoC WebFlux** (Etapa 0) que mida latencia/concurrencia y demuestre ausencia de I/O bloqueante en el flujo principal.

### Implicaciones

- Criterio de DoD técnico: ninguna llamada bloqueante no aislada en el camino ALLOW/DENY/INDETERMINATE.
- Tests de integración del camino crítico usan clientes reactivos.
