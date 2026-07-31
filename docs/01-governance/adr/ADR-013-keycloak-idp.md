# ADR-013: Keycloak como IdP del MVP

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-31 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §15, §21.2; ADR-005; PD-03 |

## Contexto

ADR-005 fija autenticación delegada y abstraída. Faltaba el proveedor concreto del MVP para desplegar, mapear claims y elaborar el threat model de identidad.

## Decisión

El IdP del MVP es **Keycloak (self-hosted)**, desplegado vía Docker Compose junto al resto de la topología MVP.

Protocolos: **OpenID Connect** y **OAuth 2.0**.

La arquitectura permanece **agnóstica al proveedor**: el PDP habla solo con el puerto de identidad; Keycloak es el adaptador concreto del MVP, no un acoplamiento de dominio.

## Alternativas consideradas

1. **Auth0** — SaaS; válido, pero introduce dependencia externa y costos desde el MVP.
2. **Azure AD / Entra ID** — acopla el MVP a un cloud vendor concreto.
3. **Google Identity** — similar limitación de vendor.
4. **Keycloak self-hosted** — **elegida**: OSS, OIDC/OAuth2, Docker-friendly, patrón empresarial habitual.

## Consecuencias

### Positivas

- Despliegue local reproducible (Cloud Enable).
- Migración futura a otro IdP sin cambiar el dominio (solo el adaptador).
- Claims y realms controlables en el laboratorio de validación.

### Negativas / costos

- Operación propia de Keycloak (realm, clients, users) en el entorno MVP.
- Hay que versionar configuración de realm/client como artefacto de despliegue.

### Implicaciones

- El servicio `keycloak` se añade a la topología de deployment MVP.
- El adaptador de identidad del PDP valida tokens emitidos por Keycloak (issuer, audience, firma JWKS).
- Documentación de 06-Security y 05-Contracts usará Keycloak como referencia concreta.
