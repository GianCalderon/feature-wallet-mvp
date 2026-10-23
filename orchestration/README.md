# Orquestacion de infraestructura local

Este directorio se reserva para Compose del laboratorio: contenedores OCI
compatibles con Docker y Podman Desktop (CLI local detectado). No contiene
todavia compose.yaml ni servicios ejecutables. La orquestacion de agentes esta
separada en [agents/](../agents/README.md).

## Servicios previstos

| Servicio | Proposito |
| --- | --- |
| PostgreSQL | Base wallet y bases/credenciales separadas para identidad |
| Meilisearch | Read model reconstruible para movimientos y conceptos; red interna y volumen propio |
| Keycloak | OIDC local, usuarios sinteticos y roles |
| WireMock | Rieles de pago y KYC simulados con fallos reproducibles |
| Backend | BFF/API Java 21, workers inbox/outbox y proyeccion Meilisearch; sin job diario HU-06 |
| Frontend | Angular, mismo origen mediante proxy cuando aplique |
| OTel Collector | Recepcion OTLP, redaccion y exportacion |
| Prometheus, Tempo, Loki, Grafana | Metricas, trazas, logs y dashboards |
| CrewAI / herramientas | Futuro runner y checks/OpenSpec en contenedores separados, activacion autorizada |

HU activas, en orden: HU-01 -> HU-02 -> HU-04. Las otras historias no habilitan
servicios o modulos adicionales. Todo componente ejecutable, incluidos build y
pruebas, se empaqueta en contenedores; no se exige Java/Node/Python en el host.

## Podman Desktop en Windows

Requisitos del futuro arranque: CLI `podman`, una maquina/conexion Podman operativa
y un proveedor Compose compatible. Podman Desktop instalado no garantiza por si
solo que `podman compose` tenga proveedor. Comprobar version, conexion y proveedor
antes de ejecutar; no instalar Docker Desktop ni modificar la maquina sin permiso.

`podman --version` y `podman compose version` son comprobaciones de herramientas,
no comandos de arranque del producto. Cuando exista Compose, documentar comandos
equivalentes de `podman compose` y `docker compose`, con ruta explicita al archivo.
No son motores que deban ejecutarse simultaneamente.

Comprobado el 2026-10-23: CLI Podman 6.0.2 y proveedor externo podman-compose
1.6.0 mediante `podman compose version`. No se arranco ni verifico una maquina
Podman ni se levantaron servicios. El proveedor instalado usa Python en el host;
esto es tooling de Compose, no un runtime de las aplicaciones o de CrewAI.
La compatibilidad del futuro Compose completo sigue pendiente de prueba real.

## Requisitos del futuro Compose

Perfiles para aplicacion, observabilidad y herramientas; el arranque completo
incluira todos los servicios del producto y los checks necesarios. CrewAI se
activa solo con permiso especifico. Healthchecks, volumenes claramente
identificados y seeds sinteticos reproducibles. Puertos locales acotados a
loopback cuando sea posible, redes internas y credenciales de laboratorio via
variables no versionadas. Imagenes fijadas, recursos limitados y usuarios sin
privilegios donde se soporte. Nunca montar socket Docker/Podman en agentes.
Imagenes multistage para Angular/Java y una imagen de herramientas fijada para
CrewAI. No depender de extensiones exclusivas de Docker: verificar soporte real
de healthchecks, dependencias y permisos de volumen en el proveedor Compose local.

TLS/sesion segura del BFF requieren un perfil HTTPS local documentado o una
excepcion de desarrollo explicita; no desactivar protecciones silenciosamente.
No exponer PostgreSQL, Meilisearch, Keycloak de administracion o dashboards a internet.
Separar credenciales admin/indexacion/busqueda y no entregarlas al frontend.

## Evidencia requerida al implementarlo

Validar config, build y healthchecks con `podman compose` y comprobar compatibilidad
de la misma definicion con `docker compose` cuando este disponible. Arranque
reproducible desde imagenes/volumenes documentados; comprobar
traza API -> worker, ausencia de secretos/PII y escenarios WireMock de timeout,
duplicado y evento fuera de orden. Probar Meilisearch caido/atrasado, indexacion
recuperable, aislamiento por usuario, busqueda por concepto y reconstruccion con
pagos concurrentes sin tocar el ledger. Documentar arranque/parada,
backup/restore y limpieza de volumenes con advertencia de perdida de datos.

No hay comando de arranque disponible hasta aprobar y construir ese cambio.