# Arquitectura objetivo

Estado: base de trabajo, no implementada. Las restricciones acordadas son Angular,
Java 21, monolito modular, CQRS, Clean Architecture, PostgreSQL + Meilisearch y
ejecucion integral en contenedores OCI/Compose compatible con Podman Desktop.
Alcance activo y secuencia: HU-01 -> HU-02 -> HU-04. QR, soporte/disputas y
conciliacion operativa estan diferidos junto con HU-03/HU-05/HU-06.
Los detalles siguientes son propuestas que se refinan mediante OpenSpec y ADR.

## Stack complementario

| Area | Seleccion propuesta | Motivo y trazabilidad |
| --- | --- | --- |
| Web | Angular, TypeScript strict, Router, Reactive Forms, cliente OpenAPI | Pasos/validacion HU-01, pago e historial del pagador HU-02 |
| API/BFF | Spring Boot sobre Java 21, Spring MVC, Bean Validation, Spring Security | Adaptadores REST, sesiones y autorizacion centralizada |
| Contratos | OpenAPI 3.1, JSON Schema, Problem Details RFC 9457 | Errores estables en onboarding, pago y busqueda |
| Identidad | Keycloak local, OIDC; BFF con cookies seguras y proteccion CSRF | Evitar tokens persistidos en el navegador; MFA a definir |
| Dominio | Java puro; commands, queries, puertos y eventos internos | Clean Architecture, CQRS sin event sourcing obligatorio |
| Escritura CQRS | PostgreSQL, Flyway, Spring Data JPA/JDBC en adaptadores | Fuente autoritativa: ledger, pagos, claves unicas, inbox/outbox y auditoria |
| Lectura de busqueda CQRS | Meilisearch por adaptador HTTP/SDK | Movimientos y conceptos de HU-02 CA3; proyeccion eventual, filtrada por propietario y reconstruible |
| Asincronia | Outbox/inbox PostgreSQL y workers del mismo despliegue | Callback/reintentos recuperables, sin broker inicial |
| Resiliencia | Resilience4j, timeouts y backoff con jitter configurables | Proveedores simulados multirriel sin failover financiero ciego |
| Estados web | Consulta autoritativa por referencia via BFF | HU-02; la ausencia de un hit en Meilisearch no prueba ausencia de pago |
| Control de duplicados | Evidencia SQL original/reintento | HU-04 CA4; conciliacion diaria y Spring Batch diferidos con HU-06 |
| Analitica | Eventos de negocio deduplicados y proyecciones SQL | Funnel, duracion, MAU y CAC HU-01/HU-02 |
| Observabilidad | OTel Java agent/SDK, Collector, Prometheus, Tempo, Loki, Grafana | Trazas, metricas, logs y correlacion sin PII |
| Simuladores | WireMock y fixtures sinteticos; webhooks firmados | Exito, rechazo, timeout, tardios, duplicados y fuera de orden |
| Pruebas | JUnit 5, AssertJ, Testcontainers PostgreSQL/Meilisearch, ArchUnit; TestBed y Playwright | Dominio, concurrencia, aislamiento del indice, reindex y E2E; conectividad con Podman por validar |
| Seguridad CI | Secret scanning, SAST, SCA, SBOM CycloneDX, ZAP baseline | Evidencia de controles, no certificacion |
| Entorno | Imagenes OCI, builds multistage y Compose Docker/Podman Desktop | Aplicaciones y dependencias en contenedores; Maven/Node en imagenes, no en el host |
| Agentes | CrewAI en contenedor/perfil de herramientas separado | Diseno/construccion supervisados; sin acceso al plano financiero ni socket del motor |

Meilisearch forma parte obligatoria del read model de busqueda, no es una cache
financiera. La seleccion CQRS se documenta en ADR-0007. PostgreSQL sigue atendiendo
lecturas que necesitan autoridad financiera y analitica SQL. Redis, Kafka y
MongoDB quedan fuera de la base; otras persistencias requieren justificacion y ADR.

## Limites modulares

`identity-onboarding`, `wallet-ledger`, `payments`, `movement-search` y `analytics`.
`merchant`, `reconciliation` y `support` quedan diferidos. Los proveedores/KYC
son puertos, no dominio. La
autorizacion de movimientos y el asiento contable se coordinan mediante una
transaccion local explicita. Ningun modulo escribe tablas de otro por atajo.
El worker y el BFF/API pertenecen inicialmente al mismo backend desplegable.

La transaccion de base de datos nunca incluye una llamada externa. Reserva,
estado y outbox deben tener una frontera atomica; el resultado externo se
incorpora mediante inbox deduplicado y una nueva transaccion validada. Los
detalles de reserva/fondeo y el grafo de estados siguen pendientes de producto.

El mismo outbox alimenta un worker de proyeccion al indice. No hay transaccion
distribuida ni dual write en el command. El worker confirma tareas asincronas de
Meilisearch antes del checkpoint y conserva orden/version por movimiento. El
BFF impone ownership y nunca entrega credenciales de busqueda al navegador.

## Documentos

- [ADR-0001: estructura y limites](adr/0001-modular-monolith-clean-cqrs.md)
- [ADR-0002: datos e integridad financiera](adr/0002-postgresql-ledger-idempotency.md)
- [ADR-0003: integraciones asincronas](adr/0003-provider-events-reconciliation.md)
- [ADR-0004: seguridad](adr/0004-security-baseline.md)
- [ADR-0005: observabilidad y analitica](adr/0005-observability-analytics.md)
- [ADR-0006: OpenSpec y CrewAI](adr/0006-spec-driven-agent-orchestration.md)
- [ADR-0007: CQRS PostgreSQL y Meilisearch](adr/0007-cqrs-postgresql-meilisearch.md)
- [C4: contexto, contenedores y componentes](c4/README.md)
- [Seguridad y modelo de amenazas inicial](security.md)
- [Observabilidad y metricas de negocio](observability.md)

Las versiones exactas se resolveran en Q-16 usando versiones soportadas y
compatibles con Java 21. No hay lockfiles ni dependencias instaladas del producto.