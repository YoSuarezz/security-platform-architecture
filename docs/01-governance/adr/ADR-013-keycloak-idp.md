# ADR-013: Keycloak como IdP del MVP

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Actualizado | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §15, §21.2; ADR-005, ADR-010; PD-03 |

## Contexto

ADR-005 fija autenticación **delegada y abstraída**. Para el MVP hace falta un IdP concreto que permita:

- emitir y validar tokens OIDC/OAuth2 en laboratorio;
- crear realms/clients/users reproducibles;
- elaborar threat model de identidad sin depender de un SaaS de pago;
- encajar en Docker Compose (Cloud Enable).

Sin proveedor concreto, el adaptador de identidad queda como interfaz vacía y no se puede demostrar UC-01 extremo a extremo.

## Por qué se tomó la decisión

1. **Open Source y self-hosted** — control total del lab académico/profesional sin vendor lock-in de facturación.
2. **OIDC/OAuth2 maduros** — alineados al estándar que el adaptador ya asume.
3. **Docker-first** — coherente con ADR-010 y la topología MVP.
4. **Patrón empresarial habitual** — transferable a currículum y a entornos reales.
5. **Agnosticismo preservado** — Keycloak es el adaptador del MVP, no un tipo del dominio.

## Decisión

El IdP del MVP es **Keycloak (self-hosted)**, desplegado en la red interna de Docker Compose.

- Protocolos: **OpenID Connect** y **OAuth 2.0**.
- El PDP valida tokens (issuer, audience, JWKS) vía puerto de identidad.
- La arquitectura permanece agnóstica: cambiar de IdP = nuevo adaptador + config, no cambio de BC Autorización.

## Alternativas consideradas

| Alternativa | Evaluación | Motivo de descarte / aplazamiento |
|---|---|---|
| **Auth0** | Excelente DX SaaS | Dependencia externa, costos, menos control del lab |
| **Azure AD / Entra ID** | Estándar enterprise | Acopla el MVP a un cloud vendor; setup más pesado para Compose local |
| **Google Identity** | Fácil para demos | Mismo problema de vendor; menos control de claims/realm |
| **Dex / Ory Hydra** | OSS ligeros | Menos features out-of-the-box para admin de usuarios/roles del lab |
| **Keycloak self-hosted** | **Elegida** | OSS + OIDC + Docker + control |

## Consecuencias

### Positivas

- E2E reproducible en Compose.
- Migración futura a Entra/Auth0 sin tocar dominio (solo adaptador).
- Threat model de identidad con superficie conocida.

### Costos / riesgos

- Operar Keycloak (realm, clients, users, upgrades de imagen).
- Versionar configuración de realm como artefacto de despliegue.
- Disponibilidad del IdP → camino crítico (ADR-014).

### Implicaciones

- Servicio `keycloak` en `deployment-mvp.md`.
- Variables `IDP_ISSUER_URL`, `IDP_AUDIENCE`, `IDP_JWKS_URL`.
- 06-Security usa Keycloak como referencia concreta del MVP.
