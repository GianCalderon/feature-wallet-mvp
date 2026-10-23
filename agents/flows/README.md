# Contrato de orquestacion futura

Solo diseno; no es un Flow ejecutable ni un esquema nativo de CrewAI. El futuro
runner Python implementara las transiciones y validara aprobaciones fuera del LLM.
Alcance secuencial: HU-01 -> HU-02 -> HU-04; no hay trabajo activo de las otras
HU. El runner sera un contenedor OCI orquestado por Compose Docker/Podman Desktop,
sin acceso al socket del motor ni a PostgreSQL/Meilisearch del producto.

## Flujo de artefactos y desarrollo

El flujo principal usa cinco agentes especializados. Ledger, backend, frontend
y plataforma se conservan como apoyo; el runner los coordina explicitamente,
sin delegacion autonoma libre. El diagrama es el objetivo, no ejecucion disponible.

```mermaid
flowchart TD
    source["HU-01 -> HU-02 -> HU-04\nOriginales, preguntas y cambio OpenSpec"]
    subgraph definition["Definicion y creacion de artefactos"]
        po["PO\nPropuesta, specs, alcance y criterios"]
        architect["Arquitecto\nDiseno, ADR, C4 y contratos"]
        securityDesign["Security\nAmenazas, controles y revision del diseno"]
        testingPlan["Testing\nEscenarios, matriz de cobertura y plan de pruebas"]
        aiPlan["AI Engineering\nPlan tecnico, tareas y handoffs de desarrollo"]
        poScope["PO + Testing\nRevision de alcance, tareas y evidencia esperada"]
    end
    source --> po --> architect --> securityDesign --> testingPlan --> aiPlan --> poScope
    poScope --> approval{"Humano\nArtefactos versionados aprobados y bloqueos cerrados?"}
    approval -->|"No: refinar"| po
    approval -->|"Si"| sandbox["Runner\nValida perfiles, permisos, presupuesto y sandbox OCI"]
    subgraph delivery["Desarrollo de la iniciativa"]
        engineering["AI Engineering\nDesarrollo e integracion de la entrega aprobada"]
        support["Apoyo coordinado por runner\nLedger, backend, frontend y plataforma"]
        architectureReview["Arquitecto\nRevision de conformidad tecnica"]
        securityReview["Security\nRevision independiente de cambios y controles"]
        testingRun["Testing\nVerificacion y reporte de evidencia real"]
    end
    sandbox --> engineering
    support -.->|"Artefactos de apoyo"| engineering
    engineering --> architectureReview --> securityReview --> testingRun
    testingRun --> checks{"Revisiones y checks satisfactorios?"}
    checks -->|"No: corregir dentro del alcance"| engineering
    checks -->|"Limite de correccion agotado"| escalation["Escalamiento humano\nNo ampliar alcance automaticamente"]
    checks -->|"Si"| poAcceptance["PO\nRecomendacion de aceptacion funcional"]
    poAcceptance --> acceptance{"Humano\nAceptar con evidencia?"}
    acceptance -->|"No: devolver observaciones"| engineering
    acceptance -->|"Si"| archive["Cierre supervisado\nVerificar, sincronizar y archivar OpenSpec"]
```

Si cambia materialmente el alcance, specs, diseno o tareas durante la correccion,
se invalida la aprobacion y se vuelve al gate de artefactos; no se continua con
la aprobacion anterior. No habilitar pagos antes de verificar idempotencia.

## Agentes y cascarones de instrucciones

| Agente principal | ID CrewAI | Perfil por completar | Artefactos y responsabilidad |
| --- | --- | --- | --- |
| PO | product_analyst | [po.md](../specialists/po.md) | Proposal, specs, prioridades y recomendacion de aceptacion |
| AI Engineering | ai_engineer | [ai-engineering.md](../specialists/ai-engineering.md) | Plan tecnico, tareas, desarrollo e integracion |
| Testing | quality_reviewer | [testing.md](../specialists/testing.md) | Escenarios, cobertura, plan de pruebas y evidencia |
| Arquitecto | solution_architect | [architect.md](../specialists/architect.md) | Design, ADR, C4, contratos y conformidad |
| Security | security_reviewer | [security.md](../specialists/security.md) | Amenazas, controles y hallazgos de diseno/implementacion |

Los perfiles son documentos propios con estado draft. Sus checklists representan
instrucciones pendientes, no tareas de construccion terminadas. El futuro runner
debera rechazar la activacion de un perfil incompleto/no revisado. No son archivos
de agentes de Copilot ni se cargan automaticamente por existir en el repositorio.

## Fases y configuraciones

| Fase | Configuracion | Permisos previstos |
| --- | --- | --- |
| Refinamiento/diseno | definition-tasks.yaml, secuencial | PO, Arquitecto, Security, Testing y AI Engineering; solo contexto sanitizado |
| Gate | Fuera de CrewAI, control humano/runner | Sin invocar modelo para decidir autorizacion |
| Construccion | construction-tasks.yaml, secuencial inicialmente | AI Engineering y apoyos; sandbox y allowlist por tarea aprobada |
| Revision | Arquitecto, Security y Testing con tareas de revision explicitas | Lectura de diff/evidencia; no alterar codigo revisado |
| Aceptacion/archivo | Responsable humano | No autoaceptar ni hacer commits/despliegues |

## Correspondencia con las tareas

| Etapa del diagrama | Tareas declarativas | Responsable |
| --- | --- | --- |
| Propuesta y specs | refine_requirements | PO |
| Diseno y controles | design_architecture, design_financial_integrity, review_security_design | Arquitecto, apoyo ledger y Security |
| Estrategia de verificacion | define_test_strategy | Testing |
| Plan tecnico | plan_delivery | AI Engineering |
| Revision antes del gate | plan_implementation, review_product_scope | Testing y PO |
| Desarrollo de apoyo | implement_backend, implement_frontend, implement_platform | Backend, frontend y plataforma |
| Integracion | integrate_initiative | AI Engineering |
| Conformidad y seguridad | review_architecture_implementation, review_security_implementation | Arquitecto y Security |
| Verificacion y recomendacion | verify_change, recommend_product_acceptance | Testing y PO |

Las tareas estan en los YAML de definicion/construccion; no ejecutan pasos por
su sola presencia. El gate humano queda fuera de las tareas LLM. Las rondas de
correccion y la invalidacion de aprobaciones son responsabilidad del futuro Flow.

## Estado minimo persistente a implementar

run_id, change_id, revision de artefactos, tareas permitidas, preguntas abiertas,
fase, presupuesto, estado de aprobacion y referencias de evidencia. El registro
de aprobacion se autentica fuera de prompts y esta vinculado a la revision.
El runner invalida la aprobacion si los artefactos cambian.

## Criterios de activacion

1. Completar y revisar los cinco perfiles; cerrar Q-15/Q-16: modelo, coste, versiones y herramientas disponibles.
2. Crear un cambio OpenSpec para runner/Flow y aprobarlo expresamente.
3. Implementar contexto seguro, gate determinista, sandbox y logs sanitizados.
4. Probar arranque del contenedor en Podman Desktop, denegacion sin aprobacion, revision cambiada, escape de rutas,
   instrucciones inyectadas, acceso a secretos y agotamiento de presupuesto.
5. Ejecutar primero un caso documental controlado; habilitar construccion solo
   tras aprobar su evidencia y un cambio de producto acotado.

Los callbacks o la memoria de CrewAI no sustituyen un control de permisos.
Las rondas de correccion deben tener limite; al agotarlo se escala al humano.