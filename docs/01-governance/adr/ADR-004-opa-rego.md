# ADR-004: OPA y Rego como motor y lenguaje de políticas

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §13.4 |

## Contexto

Las reglas de acceso deben evolucionar fuera de la lógica de negocio de cada aplicación.

## Decisión

Usar **Open Policy Agent (OPA)** con políticas en **Rego**. El PDP construye la entrada normalizada; OPA no consulta libremente el dominio.

## Consecuencias

- Políticas versionables y trazables.
- Contratos de entrada/salida a OPA deben definirse en Fase 0.E.
