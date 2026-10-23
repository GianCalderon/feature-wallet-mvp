# Seven Team Wallet Lab

Caso de estudio de una wallet para Peru, con dinero ficticio, identidades
sinteticas e integraciones simuladas. Alcance activo y secuencia:
HU-01 onboarding -> HU-02 confirmacion de pago P2P -> HU-04 idempotencia.
Comercios/QR, soporte/disputas y conciliacion operativa quedan diferidos; los
originales se conservan sin cambios.

## Estado

Repositorio de estructura y documentacion. No contiene todavia aplicaciones
ejecutables, infraestructura levantada ni agentes en ejecucion. OpenSpec guiara
la definicion y las tareas; CrewAI orquestara despues el diseno y la construccion
con revisiones humanas.

## Mapa

| Ruta | Contenido |
| --- | --- |
| [AGENTS.md](AGENTS.md) | Reglas compartidas para asistentes |
| [docs/requirements/](docs/requirements/README.md) | Epica y seis HU originales, trazabilidad y preguntas |
| [architecture/](architecture/README.md) | Stack, ADR, C4, seguridad y observabilidad objetivo |
| [openspec/](openspec/README.md) | Configuracion y cambio inicial de definicion |
| [backend/](backend/README.md) | Limites previstos del backend Java 21 |
| [frontend/](frontend/README.md) | Canales Angular previstos |
| [orchestration/](orchestration/README.md) | Plan de infraestructura local Docker Compose |
| [agents/](agents/README.md) | Roles, tareas y contrato de orquestacion CrewAI |

## Base acordada

Angular responsive para onboarding, pago e historial del pagador. Java 21,
Spring Boot, monolito modular, CQRS y Clean Architecture. PostgreSQL para
escritura, ledger e idempotencia; Meilisearch para busqueda de movimientos y por
conceptos con consistencia eventual, nunca como autoridad financiera.
OpenTelemetry para trazas, metricas y logs correlacionados.

Todo el entorno ejecutable se levantara en contenedores OCI mediante Compose
compatible con Docker y Podman Desktop: frontend, backend, PostgreSQL,
Meilisearch, identidad, simuladores, OTel y futuro runner CrewAI. No se requiere
instalar runtimes de producto en el host. Todavia no existe Compose ejecutable.

La seleccion detallada se registra como propuesta en los ADR. Las versiones
exactas se fijaran y comprobaran al crear los esqueletos, no se asume que
dependencias no instaladas ya son compatibles.

## Siguiente paso

Revisar las [decisiones pendientes](docs/requirements/open-questions.md) y el
[cambio de definicion](openspec/changes/define-wallet-mvp/proposal.md).
Completar especificaciones y tareas con OpenSpec; aprobarlas antes de habilitar
la construccion con CrewAI. No hay comandos de arranque del producto en esta fase.

## Verificacion documental

Preparacion del 2026-10-23: siete originales comprobados con SHA-256; enlaces
locales revisados; YAML parseado y referencias entre agentes/tareas verificadas.
OpenSpec 1.14.3 reconoce los artefactos y valida el cambio inicial en modo strict.
Esto verifica estructura documental, no comportamiento de pagos ni ejecucion
CrewAI. Las tareas de definicion permanecen pendientes de trabajo y aprobacion.