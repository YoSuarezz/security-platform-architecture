# ADR-010: Cloud Native, Cloud Enable y 12-Factor

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §8.2, §8.3, §8.5; deployment-mvp; ADR-013 |

## Contexto

El entorno productivo final (cloud público, on-prem, híbrido) **aún no está cerrado** (PD-13). Si el código asume hosts fijos, archivos locales de secretos o estado en disco del contenedor, migrar a Kubernetes implica reescritura.

Cloud Enable significa: construir desde el MVP como si la migración fuera inevitable, sin desplegar ya en un cluster obligatorio.

## Por qué se tomó la decisión

1. **Evitar deuda de despliegue** — el anti-patrón “funciona en mi VM” bloquea escalado.
2. **Paridad dev/CI/prod** — mismo Compose (o overlay) reduce “works on my machine”.
3. **12-Factor** — config/secretos fuera del artefacto; procesos desechables; logs como streams.
4. **Alineado a contenedores ya decididos** — PEP, PDP, OPA, SurrealDB, Auditoría, Keycloak.

## Decisión

Cumplir **Cloud Native** (servicios desacoplados, observabilidad, contenedores, elasticidad), **Cloud Enable** (listo para Kubernetes sin reescritura) y **12-Factor** (y factores extendidos aplicables a sistemas distribuidos).

Reglas concretas del MVP:

| Regla | Aplicación |
|---|---|
| Sin host fijo | DNS internos de Compose/K8s (`pdp`, `opa`, `keycloak`) |
| Sin estado en filesystem del app | Volúmenes solo para SurrealDB / bundles OPA / datos Keycloak |
| Config y secretos externos | Variables de entorno / Secrets; nunca en la imagen |
| Procesos desechables | Reinicio seguro de PEP/PDP/Auditoría |
| Paridad entornos | Mismo topología documentada en `deployment-mvp.md` |

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Deploy “bare metal” con paths absolutos** | Simple en lab | Impide Cloud Enable |
| **Kubernetes obligatorio desde día 1** | Cloud-native puro | Complejidad ops prematura para el MVP académico |
| **Compose ahora + diseño migrable** | **Elegida** | Entrega + preparación |

## Consecuencias

### Positivas

- Camino Compose → Helm/Kustomize sin cambiar dominio.
- Secretos manejables (local `.env`, luego Secret Manager).
- Observabilidad y scaling horizontales viables.

### Costos / riesgos

- Más rigor en config que un jar “con application.properties embebido”.
- Hay que versionar realm Keycloak y bundles Rego como artefactos.

### Implicaciones

- Documentado en `docs/03-architecture/deployment/deployment-mvp.md`.
- CI debe poder levantar la topología mínima.
