# ADR-005: Autenticación delegada y abstraída por proveedor

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §15; ADR-013; BC Identidad |

## Contexto

La plataforma **autoriza**. No debe convertirse en un IdP empresarial completo (registro de usuarios finales, MFA productizado, federation hub global). Las organizaciones ya tienen —o tendrán— Keycloak, Entra ID, Auth0, etc.

Si el dominio se acopla a claims de un proveedor concreto (`preferred_username` de Keycloak, `oid` de Azure…), cambiar de IdP obliga a reescribir autorización.

## Por qué se tomó la decisión

1. **Separación autenticar ≠ autorizar** — Identidad valida evidencia; Autorización decide acceso.
2. **Portabilidad** — el MVP usa Keycloak (ADR-013), pero el dominio habla un modelo interno (`SujetoId`, tenant, estado).
3. **Menor superficie** — no almacenamos contraseñas ni tokens completos; no competimos con IdPs maduros.
4. **Claims no bastan solos** — un claim “admin” sin asignación vigente no autoriza (INV-ID-02).

## Decisión

La autenticación es **delegada** a un IdP externo.

La plataforma:

1. Recibe evidencia (p. ej. Bearer JWT).
2. La **valida** (firma, issuer, audience, expiración) vía adaptador.
3. La **normaliza** a un contexto interno de identidad.
4. Contrasta con el modelo propio (usuario, tenant, estado) antes de autorizar.

El dominio **nunca** depende del SDK de un IdP concreto; solo del **puerto de identidad**.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte |
|---|---|---|
| **IdP embebido propio (usuarios/password en la plataforma)** | Control total | Alcance enorme; duplica Keycloak/Auth0; riesgo de seguridad alto |
| **Acoplar dominio a Keycloak Admin API** | Rápido en MVP | Impide migrar IdP; contamina bounded contexts |
| **Confiar claims sin validar firma** | “Simple” | Inaceptable en plataforma de seguridad |
| **Delegación + adaptador + modelo interno** | **Elegida** | Agnosticismo + seguridad |

## Consecuencias

### Positivas

- Cambio de IdP = cambio de adaptador (ADR-013 define el del MVP).
- BC Identidad acotado; BC Usuarios posee el sujeto de dominio.
- Menos secretos y PII de credenciales en SurrealDB.

### Costos / riesgos

- Hay que mantener claim-mapping documentado.
- Disponibilidad del IdP afecta el camino crítico → `INDETERMINATE`/503 (ADR-014).

### Implicaciones

- Proveedor MVP: Keycloak (ADR-013).
- No persistir tokens completos en auditoría (ADR-007).
