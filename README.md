# Plataforma Central de Seguridad - Arquitectura

Repositorio que contiene toda la documentación de arquitectura, diseño y decisiones del proyecto.

---

## Estructura

### 00. Source

Documentos originales del proyecto.

**Contenido**

- Documento base
- Documento de Fase 0
- Fuente de verdad desde la cual se deriva toda la documentación

---

### 01. Governance

Gobierno de arquitectura.

**Contenido**

- Architecture Decision Records (ADR)
- Plantilla de ADR
- Glosario oficial
- Lenguaje ubicuo
- Reglas para registrar decisiones

---

### 02. Domain

Modelo de dominio.

**Contenido**

- Event Storming
- Bounded Contexts
- Context Map
- Casos de uso
- Entidades
- Agregados
- Objetos de Valor
- Eventos de dominio
- Reglas de negocio
- Invariantes

---

### 03. Architecture

Diseño de alto nivel del sistema.

**Contenido**

- Visión arquitectónica
- Drivers arquitectónicos
- Restricciones
- Atributos de calidad
- Diagramas C4
- Arquitectura de despliegue
- Tecnologías utilizadas
- Integraciones

---

### 04. Data

Arquitectura de datos.

**Contenido**

- Modelo conceptual
- Modelo lógico
- Modelo de persistencia
- Catálogo de entidades
- Índices
- Estrategia de migración

---

### 05. Contracts

Contratos entre componentes.

**Contenido**

- APIs REST
- Eventos
- Máquinas de estado
- Diagramas de secuencia
- Contratos de integración
- Políticas OPA/Rego

---

### 06. Security

Arquitectura de seguridad.

**Contenido**

- Autenticación
- Autorización
- Modelo de amenazas
- Auditoría
- Riesgos
- Políticas de acceso

---

### 07. Engineering

Prácticas de ingeniería.

**Contenido**

- Convenciones del repositorio
- Estrategia Git
- CI/CD
- Estándares de código
- Estrategia de pruebas
- Observabilidad
- Versionamiento

---

### 08. Baseline

Resultado consolidado de la Fase 0.

**Contenido**

- Baseline v1.0
- Checklist de aceptación
- Decisiones pendientes
- Estado final de la línea base

---

### Assets

Recursos gráficos utilizados por toda la documentación.

---

### Templates

Plantillas para mantener consistencia entre documentos.

- ADR
- Casos de uso
- Diagramas
- Decisiones
- Secuencias

---

## Cómo usar este repositorio

1. Lee primero [`docs/00-source/Documento_Base.md`](docs/00-source/Documento_Base.md). Es la fuente de verdad.
2. Consulta el plan de Fase 0 en [`docs/00-source/Fase_0_Linea_Base.md`](docs/00-source/Fase_0_Linea_Base.md).
3. Toda decisión nueva se registra como ADR en [`docs/01-governance/adr/`](docs/01-governance/adr/).
4. Los documentos de `00-source/` **no se editan**. Se actualizan solo cuando se consolida un cambio aprobado y se versiona la fuente.

## Estado

| Sección | Estado |
|---|---|
| 00 Source | ✅ Accepted |
| 01 Governance | ✅ Accepted (ADR-001…014 + glosario) |
| 02 Domain | ✅ Accepted — orden `01`→`12` ([guía](docs/02-domain/README.md)) |
| 03 Architecture | ✅ Accepted — C4, Deployment, Modulith |
| 04 Data | Pendiente (Fase 0.D) |
| 05 Contracts | Pendiente (Fase 0.E) |
| 06 Security | Pendiente (Fase 0.F) |
| 07 Engineering | Pendiente (Fase 0.G) |
| 08 Baseline | Pendiente (Fase 0.H) |
