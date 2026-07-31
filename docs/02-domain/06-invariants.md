# Invariantes de dominio

| Campo | Valor |
|---|---|
| Estado | Accepted (v1.0) |
| Fase | 0.B |
| Fuentes | Documento Base §7, §12.4, §13.5, §15, §16 |

## Convención

| Prefijo | Significado |
|---|---|
| `INV-SEC` | Principio de seguridad transversal |
| `INV-TEN` | Tenant / aislamiento |
| `INV-ASN` | Asignaciones |
| `INV-POL` | Políticas |
| `INV-ID` | Identidad |
| `INV-AUD` | Auditoría |
| `INV-DAT` | Datos / modelo |

Cada invariante debe ser **verificable por prueba** (unitaria o de integración) según la estrategia de pruebas (Fase 0.G).

---

## Seguridad transversal

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-SEC-01 | Ante ausencia de política aplicable, identidad inválida, contexto incompleto o error que impida decidir con confianza, la posición es **denegar**. | §7.1 | Pruebas de `EvaluarAcceso` con esos casos → `DENY` o `INDETERMINATE` (nunca `ALLOW` silencioso) |
| INV-SEC-02 | El PDP decide; el PEP aplica. El interceptor **no** contiene reglas complejas de autorización. | §7.2 | Revisión de límites + pruebas de módulo |
| INV-SEC-03 | Las reglas de acceso viven fuera de la lógica de negocio de las apps (OPA/Rego). | §7.3 | Políticas versionadas; apps de referencia sin duplicar reglas |
| INV-SEC-04 | Solo se conceden privilegios necesarios (mínimo privilegio). | §7.4 | Diseño de roles/asignaciones; tests de no-sobrepermiso |
| INV-SEC-05 | Toda decisión es reconstruible con evidencia suficiente y protegida (trazabilidad por diseño). | §7.6 | Evento de auditoría correlacionado por decisión |
| INV-SEC-06 | Al motor de políticas y a logs solo llegan atributos necesarios; nunca secretos/credenciales/tokens completos. | §7.9, §15 | Clasificación de atributos + tests de redacción |

## Tenant

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-TEN-01 | El tenant es dato **explícito y validado** en cada decisión y consulta sensible. | §7.7 | Rechazo si falta o no coincide |
| INV-TEN-02 | Un usuario no obtiene privilegios de un tenant distinto salvo relación explícita y autorizada. | §12.4 | Tests de aislamiento cruzado |
| INV-TEN-03 | Identificadores externos nunca autorizan sin comprobar alcance de tenant y aplicación. | §12.4 | Tests de sujeto externo vs. alcance |

## Roles, recursos y asignaciones

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-DAT-01 | Un rol y un recurso están acotados a tenant/aplicación o se declaran **globales de forma inequívoca**. | §12.4 | Validación al crear/publicar |
| INV-ASN-01 | Asignaciones inactivas, vencidas o revocadas **no participan** en la decisión. | §12.4 | Solo `VIGENTE` entra al contexto OPA |
| INV-ASN-02 | Una asignación vigente declara alcance (tenant y/o aplicación) coherente con el sujeto y el rol. | §11.6, §12.4 | Transacción al asignar |

## Políticas

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-POL-01 | Toda política **publicada** tiene versión, estado, responsable y fecha de vigencia. | §12.4 | Rechazo de publicación incompleta |
| INV-POL-02 | OPA recibe entrada **normalizada** construida por el PDP; no consulta libremente el dominio ni recibe el token crudo. | §13.4 | Contrato de entrada + tests del adaptador |
| INV-POL-03 | Denegación explícita tiene prioridad sobre autorización; ausencia de política ⇒ denegar (deny-by-default). | §13.5 · [ADR-012](../01-governance/adr/ADR-012-policy-precedence.md) | Tests Rego + UC-01 |
| INV-POL-04 | `DecisionAcceso` es estructurada (`ALLOW`/`DENY`/`INDETERMINATE` + metadatos), nunca un booleano desnudo. | §14.2 | Contrato + tests |

## Identidad

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-ID-01 | Autenticación (quién es) está separada de autorización (qué puede hacer). | §15 | Flujo en dos pasos en UC EvaluarAcceso |
| INV-ID-02 | Claims no validados no bastan como autorización si requieren contraste con asignaciones vigentes. | §15 | Tests de claim “admin” sin asignación → DENY |
| INV-ID-03 | No se persisten tokens completos, contraseñas ni secretos en auditoría. | §15, §16 | Tests de redacción / esquema de evento |

## Auditoría

| ID | Invariante | Origen | Verificación esperada |
|---|---|---|---|
| INV-AUD-01 | Toda evaluación genera (o intenta generar) evidencia con `eventId`, `decisionId`, `requestId`, `correlationId`. | §16.1 | UC EvaluarAcceso postcondición |
| INV-AUD-02 | La correlación se mantiene aunque se anonimicen/eliminen datos operativos según retención. | §12.4 | Diseño de retención (0.F) + tests |
| INV-AUD-03 | La auditoría permite distinguir denegación legítima de fallo de infraestructura. | §16.2 | `reasonCode` + estado técnico en el evento |

## Criterio de cierre

- [x] Invariantes con ID únicos y origen trazable.
- [x] Mapeables a pruebas futuras.
- [x] INV-POL-03 elevada a ADR-012 Accepted.

---

## Navegación

[**← 05 Event Storming**](05-event-storming.md) · [**07 Reglas de negocio →**](07-business-rules.md)
