# Seguridad y amenazas iniciales

Propuesta pendiente de revision. No constituye certificacion ni dictamen legal.
Usar identidades sinteticas; el KYC es un simulador y no verifica identidad real.

## Fronteras de confianza

Navegador no confiable -> BFF/API -> dominio/SQL; callbacks del proveedor
simulado -> inbox validado; operadores -> herramientas privilegiadas; agentes
LLM -> workspace aislado, nunca base financiera. El emisor OIDC y el Collector
son servicios internos con configuracion y acceso independientes.

| Amenaza | Control previsto | Evidencia requerida |
| --- | --- | --- |
| Robo de sesion, XSS, CSRF | BFF, cookies seguras, CSP, sanitizacion, token CSRF | Pruebas de sesion, cabeceras y peticion cruzada |
| Acceso a wallet/movimientos ajenos | Autorizacion por objeto en servidor y scope de busqueda derivado de sesion | Pruebas negativas con dos pagadores, filtros manipulados y permisos desactualizados en indice |
| Doble gasto y replay | Ledger ACID, clave durable, inbox UNIQUE | Carreras y repeticion tras reinicio |
| Webhook falsificado o fuera de orden | Firma/mTLS segun contrato, ventana anti-replay, transiciones validas | Firmas invalidas, duplicados, timestamps antiguos |
| Manipulacion del importe/destino | Validacion en servidor y confirmacion vinculada a la intencion | Payload alterado y confirmacion no coincidente |
| Fuga en busqueda/conceptos | Red interna Meilisearch, claves separadas, allowlist de campos, ownership y retencion | Snippets/facets/conteos sin datos ajenos; sin KYC/tokens/claves indexadas |
| Fuga por logs/LLM | Redaccion, allowlists, contexto sintetico y acceso limitado | Pruebas de ausencia de secretos/PII |
| Dependencias comprometidas | Versiones fijadas, SCA, SBOM y revision de artefactos | Reportes y excepciones con caducidad |
| Interrupcion del proveedor | Timeout, circuit breaker, recuperacion y conciliacion | Inyeccion de fallos sin duplicidad financiera |

## Politicas a cerrar

Roles activos propuestos: pagador y administrador tecnico del laboratorio.
Comercio, soporte, disputas y conciliacion operativa se difieren junto con
HU-03/HU-05/HU-06. MFA, sesiones, retencion, conceptos y limites se cierran en
Q-01/Q-04/Q-11/Q-13/Q-17. No habilitar ajustes de saldo manuales como atajo.
Aplicar aislamiento a volumenes/secretos de PostgreSQL y Meilisearch; el runner
CrewAI no accede a esos almacenes ni al socket de Docker/Podman.

Adoptar una matriz ASVS con version fijada al implementar y mapear cada control
aplicable a responsable, prueba automatizada/manual y evidencia. Niveles altos
no se declaran cumplidos por acumular herramientas.

## Referencias

- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP API Security](https://owasp.org/www-project-api-security/)
- [OAuth Security BCP RFC 9700](https://www.rfc-editor.org/rfc/rfc9700)
- [ADR de seguridad](adr/0004-security-baseline.md)