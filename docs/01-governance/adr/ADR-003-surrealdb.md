# ADR-003: SurrealDB como persistencia principal

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6.1, §12; ADR-006 (REBAC); Fase 0.D |

## Contexto

El modelo de seguridad combina:

- **Entidades con atributos** (tenant, usuario, rol, política versionada, vigencia).
- **Relaciones navegables** (usuario–tenant, usuario–rol–app, propietario de recurso, pertenencia a equipo).
- Consultas típicas del PDP del tipo: “¿este sujeto tiene asignación vigente sobre este recurso en este tenant?”.

Una base solo relacional obliga a unir muchas tablas o a materializar grafos a mano. Una base solo documental dificulta REBAC. Se necesita un store que combine documento y grafo sin dos fuentes de verdad.

## Por qué se tomó la decisión

1. **Ajuste al dominio** — RBAC + futuro REBAC encajan en documento + relaciones.
2. **Una sola fuente de verdad** — evita el patrón “Postgres + Neo4j” con doble escritura.
3. **Decisión confirmada del Documento Base** — no se reabre en Fase 0.
4. **Consultas de autorización justifican el grafo** — cada arista debe servir a una pregunta real del PDP (no modelar “por si acaso”).

## Decisión

Usar **SurrealDB** como **persistencia principal** del dominio de seguridad (tenants, apps, recursos, usuarios, identidad referenciada, roles, perfiles, asignaciones, metadatos de políticas, referencias de auditoría).

SQL relacional **no** es la arquitectura objetivo del dominio de seguridad.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **PostgreSQL puro** | Maduro, equipo amplio | REBAC y navegación de relaciones se vuelven costosas/complejas; no es el objetivo del Documento Base |
| **PostgreSQL + Neo4j** | Potente | Dos stores, transacciones distribuidas, mayor ops; deuda temprana |
| **MongoDB** | Buen documento | Débil para relaciones tipadas de autorización |
| **Neo4j solo** | Excelente grafo | Atributos documentales y ops adicionales; fuera de la decisión confirmada |
| **SurrealDB** | **Elegida** | Documento + grafo en un motor; alineado al Documento Base |

## Consecuencias

### Positivas

- Modelo único para atributos y relaciones.
- Facilita Etapa 3 (REBAC) sin migrar de motor.
- Coherente con consultas del Constructor de contexto del PDP.

### Costos / riesgos

- Madurez/ecosistema menor que Postgres → mitigar con pruebas técnicas (consultas, índices, transacciones, aislamiento por tenant) en 0.D.
- Curva SurrealQL y drivers reactivos.
- Cada relación de grafo debe justificarse por caso de uso de autorización.

### Implicaciones

- El modelo lógico/físico se define en **04-Data**.
- Solo el PDP (y Auditoría para evidencia) acceden a SurrealDB; el PEP no.
