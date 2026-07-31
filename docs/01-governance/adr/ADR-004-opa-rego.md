# ADR-004: OPA y Rego como motor y lenguaje de políticas

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §13.4; ADR-012; Fase 0.E |

## Contexto

Si las reglas de acceso viven como `if/else` dentro de cada aplicación o dentro del PEP:

- se duplican por lenguaje (Java, PHP, .NET…);
- no hay versionado ni explicación uniforme;
- cambiar una política transversal exige redeploy de N sistemas;
- la auditoría no puede citar “qué política y versión” decidió.

La plataforma debe concentrar la **evaluación** de políticas de forma externa al código de negocio y al interceptor.

## Por qué se tomó la decisión

1. **Separación política vs código** — PBAC real: las reglas evolucionan como artefactos versionables.
2. **OPA es estándar de facto** para policy-as-code en cloud-native.
3. **Rego** expresa reglas declarativas auditables y testeables unitariamente.
4. **El PDP orquesta; OPA decide sobre entrada normalizada** — OPA no navega libremente el dominio ni recibe el token crudo (principio de mínimo privilegio de datos).

## Decisión

Usar **Open Policy Agent (OPA)** con políticas en **Rego** como motor de evaluación.

Contrato obligatorio:

- El **PDP** construye la entrada normalizada (sujeto, tenant, app, recurso, acción, roles/atributos/relaciones relevantes).
- OPA **no** consulta SurrealDB ni el IdP por su cuenta.
- La salida se traduce a `DecisionAcceso` (`ALLOW` / `DENY`; errores de infra → `INDETERMINATE`, ADR-014).
- Los metadatos de política (versión, estado) viven en el BC Políticas; el bundle Rego es el artefacto ejecutable.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **Reglas en Java (hardcoded)** | Simple al inicio | No escala multi-app; no versionable; contradice PBAC |
| **Motor propio de reglas** | Control total | Costo enorme; reinventar OPA |
| **Casbin / otros engines** | Válidos en algunos stacks | Menos ecosistema cloud-native / tooling que OPA en este contexto |
| **XACML** | Estándar histórico | Complejidad y tooling inferior al ecosistema OPA actual |
| **OPA + Rego** | **Elegida** | Estándar, testeable, alineado al Documento Base |

## Consecuencias

### Positivas

- Políticas versionables, explicables (`policyReferences` en la decisión).
- Mismo motor para apps heterogéneas.
- Tests Rego independientes del runtime Java.

### Costos / riesgos

- Curva Rego y disciplina de bundles.
- Contrato de entrada/salida debe cerrarse en **05-Contracts**.
- Latencia de hop HTTP PDP→OPA (mitigar con co-locación en Compose/K8s y timeouts claros).

### Implicaciones

- Precedencia deny-by-default (ADR-012) se implementa en Rego + orquestación PDP.
- Threat model debe cubrir manipulación de entrada a OPA y bundles.
