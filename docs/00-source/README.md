# 00. Source

Documentos originales del proyecto. **No se editan in-place** salvo consolidación versionada con bitácora.

| Archivo | Descripción |
|---|---|
| [Documento_Base.md](Documento_Base.md) | Fuente de verdad: objetivos, decisiones tecnológicas, principios, NFRs, etapas |
| [Fase_0_Linea_Base.md](Fase_0_Linea_Base.md) | Plan de construcción de la Fase 0 |

## Autoridad cuando hay más detalle en diagramas

| Tema | Autoridad |
|---|---|
| Decisiones tecnológicas, principios, alcance, NFRs | `Documento_Base.md` + ADRs |
| Partición de bounded contexts y modelos anémicos por contexto | Diagramas en `docs/02-domain/diagrams/` (11 BC) |
| Contenedores y componentes PEP/PDP | Diagramas C4 en `docs/03-architecture/c4/` |

El Documento Base §11 agrupa algunos contextos; los diagramas de dominio los detallan en **11 contextos**. No contradicen el stack ni los principios: refinan el mapa de dominio. Los Markdown derivados deben alinearse a esos diagramas.
