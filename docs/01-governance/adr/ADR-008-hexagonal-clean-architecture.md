# ADR-008: Clean Architecture mediante arquitectura hexagonal

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §8.7, §18.1; ADR-001, ADR-004, ADR-009; C4 L3 PDP/PEP |

## Contexto

Si el dominio de autorización importa Spring, el driver de SurrealDB o el cliente de OPA:

- las pruebas de invariantes requieren infra real;
- cambiar OPA/SurrealDB/IdP toca el corazón del sistema;
- Modulith no puede proteger límites de negocio porque el framework ya atraviesa todo.

Clean Architecture (dependencias hacia adentro) se materializa aquí como **hexágono**: dominio + aplicación en el centro; adaptadores afuera.

## Por qué se tomó la decisión

1. **El dominio de seguridad es el activo** — no el framework.
2. **Sustituibilidad** — IdP (ADR-005/013), OPA, SurrealDB viven detrás de puertos.
3. **Testabilidad** — UC EvaluarAcceso e invariantes se prueban con fakes.
4. **Visible en C4 L3** — el diagrama del PDP ya muestra puertos y adaptadores.

## Decisión

Aplicar **Clean Architecture** implementada con **arquitectura hexagonal** en **todos** los contenedores Java (PEP, PDP, Auditoría):

```text
domain / application  ←──  ports (interfaces)
                              ↑
                     adapters (WebFlux, OPA, SurrealDB, Keycloak, …)
```

Reglas:

1. El dominio **no** importa Spring, OPA, SurrealDB ni SDKs de IdP.
2. Los casos de uso dependen de puertos, no de implementaciones.
3. Los adaptadores implementan puertos y viven en `infrastructure` (por módulo Modulith).

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Layers clásicas acopladas a Spring** | Rápido | Dominio contaminado; difícil de probar |
| **CQRS/Event Sourcing completo** | Potente | Sobreingeniería para MVP |
| **Hexagonal solo en PDP** | “Pragmático” | PEP y Auditoría también evolucionan; disciplina desigual |
| **Hexagonal en todos los jars Java** | **Elegida** | Consistencia + C4 |

## Consecuencias

### Positivas

- Pruebas de dominio sin Docker.
- Cambios de infra localizados.
- Encaja con Modulith (un módulo = domain + application + infrastructure).

### Costos / riesgos

- Más archivos/puertos al inicio.
- Requiere review para evitar “fugas” de framework al dominio.

### Implicaciones

- C4 L3 PEP/PDP son la referencia visual.
- Paquete `commons` solo VOs/IDs, sin lógica de infra.
