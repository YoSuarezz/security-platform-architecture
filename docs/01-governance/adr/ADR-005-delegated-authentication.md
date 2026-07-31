# ADR-005: Autenticación delegada y abstraída por proveedor

| Campo | Valor |
|---|---|
| Estado | Accepted |
| Fecha | 2026-07-30 |
| Decisores | Autor del proyecto |
| Relacionado | Documento Base §6, §15 |

## Contexto

La plataforma no debe acoplarse a un único IdP ni reemplazar un proveedor de identidad empresarial maduro.

## Decisión

La autenticación es **delegada**. La plataforma valida y normaliza evidencia de identidad a un contexto interno, independiente del formato original de claims.

## Consecuencias

- Adaptador de identidad obligatorio.
- El proveedor concreto del MVP queda pendiente (pregunta §21.2) y se cerrará con ADR posterior.
