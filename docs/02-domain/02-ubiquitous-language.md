# Lenguaje ubicuo

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Autoridad de términos base | [`docs/01-governance/glossary.md`](../01-governance/glossary.md) |

## Regla

No existe un segundo glosario paralelo. El [glosario oficial](../01-governance/glossary.md) es la autoridad. Este documento:

1. Referencia ese glosario.
2. Amplía el vocabulario operativo del dominio (actores, verbos, estados, anti-términos).
3. Prohíbe sinónimos que generan ambigüedad en diseño e implementación.

## Términos operativos del dominio

| Término | Definición operativa | Contexto dueño |
|---|---|---|
| Sujeto | Identidad interna resuelta sobre la que se decide un acceso (`Usuario` o representación equivalente). | Identidad |
| Tenant | Partición organizacional explícita en toda decisión y consulta sensible. | Tenants |
| Aplicación protegida | Sistema que delega autorización a la plataforma. | Aplicaciones |
| Recurso | Elemento protegible (ruta, operación, funcionalidad, entidad o instancia). | Recursos |
| Acción | Operación solicitada sobre un recurso (`leer`, `crear`, `aprobar`, `eliminar`, …). | Recursos / Políticas |
| Rol | Agrupación de autorizaciones sobre recursos y acciones. | Roles y perfiles |
| Perfil | Agrupación de roles para facilitar asignación. | Roles y perfiles |
| Asignación | Vínculo vigente (o no) entre sujeto y privilegio, acotado por tenant/aplicación/tiempo. | Asignaciones |
| Política | Metadatos/estructura versionable de criterios de acceso (`Politica`/`Regla`/`Condicion`); la fuente ejecutable es Rego en OPA. | Políticas de acceso |
| Autorización | Contexto de evaluación: de `SolicitudAcceso` a `DecisionAcceso` vía OPA. Distinto del catálogo de políticas. | Autorización |
| SolicitudAcceso | Representación normalizada de una petición a evaluar. | Autorización |
| DecisionAcceso | Resultado estructurado: `ALLOW`, `DENY` o `INDETERMINATE`. | Autorización |
| Usuarios | Sujeto de dominio (cuenta interna), distinto del mecanismo IdP. | Usuarios |
| Identidad y autenticación | Credenciales, proveedor, sesión, evidencia/claims normalizados. | Identidad y autenticación |
| PEP | Contenedor que intercepta y aplica la decisión. No contiene reglas complejas ni datos. | C4 / Autorización (enforcement) |
| PDP | Contenedor que orquesta la evaluación (BC Autorización + lecturas). | C4 / Autorización |
| Evidencia / Evento de auditoría | Registro correlacionado de una decisión o operación sensible, sin secretos. | Auditoría |
| Identidad externa | Evidencia o enlace hacia un IdP; no es por sí sola autorización. | Identidad |
| Denegación por defecto | Ante duda, ausencia de política, identidad inválida o error no confiable → negar. | Transversal |

## Verbos del dominio (lenguaje de casos de uso)

| Verbo | Significa | No significa |
|---|---|---|
| Evaluar | Construir contexto y obtener `DecisionAcceso`. | Ejecutar lógica de negocio de la app protegida. |
| Aplicar / Enforce | Hacer cumplir `ALLOW`/`DENY` en el borde (PEP). | Reinterpretar la política. |
| Publicar | Poner en vigencia una versión de política. | Editar Rego ad hoc en producción sin versión. |
| Asignar | Crear una asignación de rol/perfil con vigencia. | Inferir privilegios desde claims del token. |
| Revocar | Invalidar una asignación o política publicada. | Borrar evidencia de auditoría. |
| Normalizar | Convertir evidencia de IdP a sujeto/contexto interno. | Confiar en claims sin validar. |
| Correlacionar | Unir solicitud, decisión y auditoría por IDs. | Registrar tokens completos. |

## Estados canónicos

### DecisionAcceso

| Estado | Significado |
|---|---|
| `ALLOW` | Acceso concedido según políticas aplicables. |
| `DENY` | Acceso negado (política, identidad, tenant, etc.). |
| `INDETERMINATE` | No fue posible decidir de forma confiable (OPA/SurrealDB/IdP caído). El PEP responde **503** (ADR-014). Nunca se traduce a ALLOW. |

### Asignación (ciclo de vida mínimo)

| Estado | Significado |
|---|---|
| `PENDIENTE` | Creada pero aún no vigente. |
| `VIGENTE` | Participa en la decisión. |
| `VENCIDA` | Superó vigencia; no participa. |
| `REVOCADA` | Invalidada administrativamente; no participa. |

### Política

| Estado | Significado |
|---|---|
| `BORRADOR` | Editable, no evaluable en producción. |
| `PUBLICADA` | Versión activa (o candidata según alcance). |
| `REVOCADA` / `RETIRADA` | Ya no aplica. |

## Anti-glosario (términos prohibidos o a evitar)

| Evitar | Usar en su lugar | Por qué |
|---|---|---|
| “Permiso” como sinónimo ambiguo de rol/política/decisión | Rol, Asignación, DecisionAcceso o Política según el caso | Confunde RBAC con el resultado de evaluación. |
| “Usuario autenticado = autorizado” | Sujeto autenticado + evaluación de autorización | Viola separación autenticación/autorización. |
| “Login” como capacidad de la plataforma | Autenticación delegada / IdP | La plataforma no es el IdP. |
| “Gateway genérico” | PEP / proxy PEP | Fuera de alcance convertirse en API Gateway completo. |
| “SQL como fuente de verdad” | SurrealDB | Decisión confirmada (ADR-003). |
| “Blockchain de auditoría” como requisito | Evidencia / EventoAuditoria | `HashBlockchain` es investigación, no requisito. |
| “Caché de autorización como verdad” | Decisión evaluada + invalidación por versión | Caché solo acelerador, nunca fuente de verdad. |

## Regla de cambio

Todo término nuevo o renombrado debe:

1. Actualizar el [glosario](../01-governance/glossary.md) si es término canónico.
2. Actualizar esta tabla operativa.
3. Si cambia semántica arquitectónica, crear o actualizar un ADR.

---

## Navegación

[**← 01 App validación**](01-validation-application.md) · [**03 Bounded Contexts →**](03-bounded-contexts.md)
