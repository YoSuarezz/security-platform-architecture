# ADR-011: Línea base de ingeniería (Git, CI/CD, pruebas, Clean Code)

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §8.6, §8.9–§8.12; ADR-009; Fase 0.G |

## Contexto

Arrancar historias funcionales sin:

- estrategia Git,
- pipeline CI,
- umbral de pruebas,
- estándar de Clean Code,
- gate de límites Modulith,

es el camino más corto a deuda técnica invisible. El Documento Base lo marca como riesgo explícito (§20): “Modulith solo nominal”, “sin CI”, cobertura insuficiente en la rama de decisión.

## Por qué se tomó la decisión

1. **La calidad no se “añade después”** en un componente que decide accesos.
2. **Gates mecánicos > convenciones de honor** — el build falla si se rompe un límite de módulo o baja la cobertura.
3. **Clean Code en seguridad** — código ilegible en el PDP es riesgo de seguridad, no solo de estilo.
4. **Orden Fase 0** — 0.G convierte principios en pipelines antes de Etapa 1.

## Decisión

Antes de la Etapa 1 (historias funcionales del esqueleto) deben existir:

| Elemento | Mínimo |
|---|---|
| **Estrategia Git** | `main` protegida, `develop`, feature branches, PR obligatorios, code review |
| **Pipeline CI** | compile → análisis estático → tests → cobertura → **verificación Modulith** (PEP, PDP, Auditoría) |
| **Pruebas** | pirámide; cobertura **>80 %** en la rama de decisión ALLOW/DENY/INDETERMINATE |
| **Clean Code** | nombres de dominio, funciones pequeñas, sin complejidad incidental en autorización |
| **CD esqueleto** | despliegue a entorno de lab (Compose) automatizable, aunque el destino prod esté pendiente |

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **“Primero features, después CI”** | Velocidad aparente | Deuda inmediata; contradice Documento Base |
| **Solo tests manuales** | Barato | Inaceptable en plataforma de seguridad |
| **Cobertura 100 % global** | Idealista | Coste absurdo; el umbral >80 % en rama crítica es el acordado |
| **Línea base formal antes de Etapa 1** | **Elegida** | Reduce riesgo §20 |

## Consecuencias

### Positivas

- Etapa 1 nace sobre rieles.
- Modulith deja de ser nominal (ADR-009).
- Reviews más objetivos (DoD verificable).

### Costos / riesgos

- Inversión inicial en 0.G.
- Disciplina de PR puede sentirse “lenta” al inicio — es deliberado.

### Implicaciones

- Fase 0.G es bloqueante para Baseline v1.0 de producto.
- DoD incluye explícitamente los tres estados de decisión y el gate Modulith.
