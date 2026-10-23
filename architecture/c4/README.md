# Diagramas C4

Modelo objetivo, no despliegue existente. Diagramas Mermaid con notacion C4:
personas, sistema, contenedores y componentes con responsabilidades y tecnologias.
Los nodos externos representan simuladores locales en el laboratorio.
Alcance: HU-01 -> HU-02 -> HU-04. No se diagraman como activos comercios,
soporte/disputas ni conciliacion operativa. Todo nodo ejecutable se desplegara
en contenedores OCI con Compose compatible con Docker y Podman Desktop.

## Nivel 1: Contexto

```mermaid
flowchart LR
    payer["Persona: pagador"]
    wallet["Sistema: Seven Team Wallet\nOnboarding, P2P, idempotencia e historial buscable"]
    rail["Sistema externo simulado:\nrieles de pago"]
    kyc["Sistema externo simulado:\nverificacion KYC"]
    payer -->|"Onboarding, transferencia y confirmacion"| wallet
    wallet -->|"Intencion, consulta y resultado"| rail
    rail -->|"Callbacks autenticados"| wallet
    wallet -->|"Validacion sintetica"| kyc
```

## Nivel 2: Contenedores

```mermaid
flowchart TB
    users["Persona: pagador"]
    subgraph boundary["Sistema Seven Team Wallet - OCI / Compose en Podman Desktop"]
        web["Contenedor: Web Angular\nOnboarding, pago e historial"]
        api["Contenedor: Backend Java 21 / Spring Boot\nBFF, API REST, modulos y workers"]
        db[("Contenedor: PostgreSQL\nWrite model, ledger, idempotencia e inbox/outbox")]
        search[("Contenedor: Meilisearch\nRead model de movimientos y conceptos")]
        oidc["Contenedor: Keycloak\nOIDC e identidad local"]
        collector["Contenedor: OTel Collector\nRedaccion y exportacion OTLP"]
        metrics["Contenedor: Prometheus\nMetricas"]
        traces["Contenedor: Tempo\nTrazas"]
        logs["Contenedor: Loki\nLogs"]
        grafana["Contenedor: Grafana\nDashboards operativos"]
    end
    sims["Contenedor externo simulado\nWireMock: rieles y KYC"]
    users --> web
    web -->|"HTTPS, cookie de sesion, REST"| api
    api -->|"OIDC, tokens en servidor"| oidc
    api -->|"SQL: commands y lecturas autoritativas"| db
    api -->|"Worker outbox: proyeccion; BFF: busqueda con ownership"| search
    api -->|"HTTPS por puertos"| sims
    sims -->|"Webhook autenticado"| api
    api -->|"OTLP"| collector
    collector --> metrics
    collector --> traces
    collector --> logs
    grafana --> metrics
    grafana --> traces
    grafana --> logs
```

PostgreSQL es tecnologia compartida, no licencia para compartir tablas entre
modulos; Keycloak usa base/credenciales propias. El navegador no recibe tokens
OIDC persistentes. El worker no es un microservicio separado en este modelo.
Meilisearch no tiene acceso desde el navegador y no es autoridad financiera.
Su volumen es independiente del SQL. Un perfil de herramientas separado alojara
CrewAI y OpenSpec, sin acceso a datos financieros ni socket del motor.

## Nivel 3: Componentes del backend

```mermaid
flowchart LR
    inbound["Adaptadores de entrada\nREST/BFF, webhooks y scheduler"]
    auth["Seguridad\nSesion, CSRF, RBAC y ownership"]
    onboarding["identity-onboarding\nPasos y activacion"]
    payments["payments\nCommands, estados, idempotencia y queries"]
    ledger["wallet-ledger\nCuentas, reservas y doble partida"]
    movementSearch["movement-search\nQueries por concepto y movimientos autorizados"]
    projection["Worker de proyeccion\nOutbox, version y checkpoint de indexacion"]
    analytics["analytics\nEventos y proyecciones de negocio"]
    ports["Puertos de salida\nRepositorios, proveedores, KYC y eventos"]
    sql["Adaptadores SQL\nPostgreSQL, inbox y outbox"]
    index["Adaptador de busqueda\nMeilisearch HTTP/SDK"]
    external["Adaptadores externos\nRieles y KYC simulados"]
    inbound --> auth
    auth --> onboarding
    auth --> payments
    auth --> movementSearch
    payments -->|"API interna, transaccion local"| ledger
    projection -->|"Lee outbox SQL; confirma tareas del indice"| ports
    onboarding --> ports
    payments --> ports
    ledger --> ports
    movementSearch -->|"Busqueda y verificacion de ownership SQL"| ports
    analytics --> ports
    ports -.->|"Implementado por"| sql
    ports -.->|"Implementado por"| index
    ports -.->|"Implementado por"| external
```

Dentro de cada modulo: adaptadores -> application -> domain. Las flechas entre
componentes son colaboraciones, no dependencias del dominio sobre SQL. Analitica
consume eventos durables via worker; no condiciona el commit financiero.

## Vistas complementarias

CrewAI y OpenSpec son herramientas de desarrollo fuera del sistema de pagos:
el [flujo de agentes](../../agents/flows/README.md) no tiene acceso al ledger.
Nivel 4 (codigo) se agrega solo cuando exista codigo que diagramar. Al crear
Compose se anadira una vista de despliegue verificada con Podman Desktop y Docker.