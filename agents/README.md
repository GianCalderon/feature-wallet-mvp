# Agentes CrewAI

Definiciones documentales para una futura orquestacion con CrewAI. No hay runner,
paquete Python instalado, herramientas conectadas ni llamadas a modelos.
No ejecutar `crewai run` aqui: aun no existe un proyecto ejecutable.

Alcance del contexto y backlog: HU-01 -> HU-02 -> HU-04. HU-03/HU-05/HU-06
quedan diferidas. La busqueda de movimientos/conceptos usa Meilisearch como
proyeccion de PostgreSQL; ninguna herramienta de agente accede a datos financieros.
El futuro runner y los checks se ejecutaran en contenedores OCI, mediante Compose
compatible con Docker y Podman Desktop, sin runtimes de producto en el host.

## Contenido

- [Roles](config/agents.yaml): cinco agentes principales y cuatro de apoyo.
- [Cascarones de especialistas](flows/README.md#agentes-y-cascarones-de-instrucciones):
  perfiles de PO, AI Engineering, Testing, Arquitecto y Security por completar.
- [Skills](skills/README.md): capacidades reutilizables, una carpeta por skill.
- [Asignaciones de skills y perfiles](config/skill-bindings.yaml): manifiesto
  propio del proyecto, no un campo nativo de CrewAI.
- [Tareas de definicion](config/definition-tasks.yaml): refinamiento y diseno.
- [Tareas de construccion](config/construction-tasks.yaml): contratos de ejecucion
  futura, no autorizacion para empezar.
- [Flujo y gates](flows/README.md): etapas y responsabilidad humana.
- [Evidencia de ejecucion](runs/README.md): contrato de resultados futuros.

Se usa el formato YAML clasico de CrewAI (agentes con role/goal/backstory y tareas
con description/expected_output/agent/context). Requerira wiring `@CrewBase` o
carga explicita y un Flow Python posterior. Los YAML por si solos no crean
crews, inyectan archivos, asignan permisos ni implementan aprobaciones.

## Agentes principales

| Rol | ID CrewAI | Cascaron de instrucciones |
| --- | --- | --- |
| PO | product_analyst | [specialists/po.md](specialists/po.md) |
| AI Engineering | ai_engineer | [specialists/ai-engineering.md](specialists/ai-engineering.md) |
| Testing | quality_reviewer | [specialists/testing.md](specialists/testing.md) |
| Arquitecto | solution_architect | [specialists/architect.md](specialists/architect.md) |
| Security | security_reviewer | [specialists/security.md](specialists/security.md) |

Apoyos conservados: ledger_specialist, backend_engineer, frontend_engineer y
platform_engineer. Los nueve roles se definen una sola vez en config/agents.yaml;
las tareas usan esos IDs. Los cinco perfiles son cascarones draft con instrucciones
pendientes, no archivos de agentes de Copilot ni permisos de ejecucion.

El manifiesto skill-bindings.yaml version 2 conserva agents como mapa de skills
y agrega instruction_profiles como mapa de ID -> ruta de perfil relativa a la
raiz del repositorio. Este contrato es propio del laboratorio: no pasar estas
claves al constructor Agent de CrewAI. El futuro runner validara rutas, IDs,
revision y estado de cada perfil antes de cargarlo; debe bloquear borradores o
instrucciones incompletas. Cambiar una etiqueta draft por si sola no constituye
aprobacion. Las carpetas no activan descubrimiento ni ejecucion automatica.

## Contexto que debera aportar el runner

Entradas comunes: `change_id` y `approved_context`. Esta ultima es contenido
autorizado y sanitizado de fuentes, specs, diseno, tareas y ADR, no una simple
ruta que el LLM tenga que abrir ni una declaracion de aprobacion confiable.
El runner verifica las aprobaciones por separado antes de invocar modelos/tools.

En definicion, el contexto puede incluir borradores claramente etiquetados. En
construccion, debe incluir las versiones aprobadas, IDs exactos de tareas,
allowlist de rutas, escenarios y comandos de prueba autorizados. Las tareas
CrewAI no reemplazan ni duplican el backlog: los IDs y estado viven en OpenSpec.

## Limites obligatorios para el futuro runner

- Denegar por defecto shell, red externa, escritura y lectura de secretos.
- Validar rutas canonicas, symlinks y herramientas; bloquear escapes del workspace.
- Mantener repositorio/credenciales del producto fuera del contexto LLM.
- Habilitar escritura solo en worktree/sandbox y rutas de la tarea aprobada.
- No montar socket Docker/Podman en agentes; lifecycle/build bajo control externo.
- Fijar modelo/proveedor/versions, limite de tokens/coste, tiempo y reintentos (Q-15/Q-16).
- Configurar `allow_delegation: false`; no construir cadenas de agentes ilimitadas.
- Separar autor, revisor y aprobador humano. No aceptar autoaprobacion.
- Validar salidas con esquema y checks deterministas, no solo evaluacion por LLM.
- No conceder permisos por `human_input` ni porque una descripcion diga aprobado.

Las tareas de construccion actuales solo pueden producir propuestas textuales
porque no tienen herramientas. Para construir codigo de verdad se necesita
implementar y probar el runner/sandbox primero, en un cambio OpenSpec separado.

## Referencias

- [CrewAI Agents](https://docs.crewai.com/en/concepts/agents)
- [CrewAI Tasks](https://docs.crewai.com/en/concepts/tasks)
- [CrewAI Flows](https://docs.crewai.com/en/concepts/flows)
- [ADR-0006](../architecture/adr/0006-spec-driven-agent-orchestration.md)