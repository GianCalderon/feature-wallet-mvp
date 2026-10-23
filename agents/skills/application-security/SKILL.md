---
name: application-security
description: "Usar para amenazas, ASVS, autenticacion propia con claves hasheadas, sesion/CSRF, autorizacion por objeto, privacidad de busqueda y aislamiento de agentes."
---

# Seguridad de aplicaciones

## Procedimiento

1. Consultar el [modelo de amenazas](../../../architecture/security.md) y las
   decisiones de seguridad pendientes del cambio autorizado.
2. Revisar la tabla de usuarios (Argon2id, sal unica, tiempo constante, limite de
   intentos, sin enumeracion), sesion BFF de servidor, CSRF, minimo privilegio y
   ownership del lado servidor; no usar controles Angular ni datos obsoletos del
   indice como autorizacion. No hay Keycloak/OIDC ni tokens en el navegador.
3. Verificar filtros de busqueda derivados de sesion y proteccion de resultados,
   snippets, facets y conteos; sin claves Meilisearch en el navegador.
4. Minimizar datos indexados y redactar telemetria; no exponer claves, hashes,
   datos de identidad, tokens, secretos o claves idempotentes en indice, logs ni contexto LLM.
5. Revisar sandbox, rutas y gates; no conceder tools ni sockets del motor a
   agentes. Mapear controles ASVS aplicables a pruebas y evidencia.

## Salida y verificacion

Hallazgos por severidad, amenaza, control y prueba. Diferenciar propuesta de
control verificado; no afirmar certificacion bancaria ni conformidad regulatoria.