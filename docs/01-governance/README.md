# 01. Governance

Gobierno de arquitectura del proyecto.

## Propósito

Registrar, versionar y hacer trazables las decisiones arquitectónicas. Ninguna decisión confirmada del documento base debe existir sin un ADR correspondiente.

## Contenido

| Artefacto | Descripción |
|---|---|
| [adr/](adr/) | Architecture Decision Records |
| [adr/template.md](adr/template.md) | Plantilla oficial de ADR |
| [glossary.md](glossary.md) | Glosario oficial / lenguaje ubicuo |

## Reglas

1. Toda decisión confirmada del documento base se formaliza como ADR con estado `Accepted`.
2. Una decisión pendiente no se implementa hasta convertirse en ADR `Accepted`.
3. Los ADR no se borran: se marcan `Deprecated` o `Superseded by ADR-XXX`.
4. El glosario es la autoridad de nombres para diagramas, políticas Rego, eventos y datos.

## Fase

Corresponde a la **Fase 0.A** del plan de línea base.
