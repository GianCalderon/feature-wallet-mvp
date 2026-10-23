# ADR-0006: OpenSpec antes de construccion con CrewAI

- Fecha: 2026-10-23
- Estado: Propuesto; secuencia de trabajo acordada.
- Referencias activas: HU-01 -> HU-02 -> HU-04; Q-15, Q-16.

## Contexto

Las HU contienen preguntas abiertas. Una orquestacion autonoma no debe convertir
supuestos en politicas financieras ni autorizarse a si misma a construir.

## Decision

OpenSpec spec-driven es la fuente de propuestas, delta specs, diseno y tareas.
CrewAI tendra etapas de refinamiento, diseno, construccion y revision con
aprobaciones humanas entre ellas. Los agentes leen solo el contexto autorizado
del cambio; cada salida debe identificar HU/CA, escenario y evidencia.

El [flujo de especialistas](../../agents/flows/README.md) concreta cinco roles
principales: PO, AI Engineering, Testing, Arquitecto y Security. Ledger, backend,
frontend y plataforma se conservan como apoyos coordinados por el runner, no por
delegacion libre. Los cascarones en agents/specialists/ deben completarse y
revisarse antes de activar cualquier perfil; no sustituyen skills ni tareas.

En esta entrega solo se versionan definiciones declarativas y contratos. No hay
runner Python, herramientas de escritura/shell ni llamadas LLM. El futuro runner
debe imponer gates deterministas, sandbox y allowlists; una instruccion en YAML
o `human_input` no es una frontera de seguridad ni una aprobacion vinculada a Git.

La aprobacion debe vincular change ID, hash de artefactos/commit, alcance,
aprobador y fecha. Cualquier cambio material invalida la aprobacion. Revisores
no aprueban su propia implementacion; el responsable humano decide el avance.

El futuro runner CrewAI y los checks se ejecutaran tambien en contenedores OCI
compatibles con Docker/Podman Desktop. Un runner no recibe el socket del motor;
el arranque/build de contenedores permanece bajo control externo autorizado.
Los contextos solo incluyen HU activas y preservan el orden HU-01 -> HU-02 -> HU-04,
sin habilitar pagos hasta verificar la proteccion idempotente necesaria.

## Alternativas

Agentes con shell irrestricto aceleran prototipos, pero elevan riesgo de fuga de
secretos y cambios no revisados. Copiar tareas desconectadas de OpenSpec crea
dos backlogs divergentes.

## Consecuencias y verificacion

Mas revision explicita, con trazabilidad y costes acotables. Antes de activar,
probar rechazo sin aprobacion, artefactos cambiados, path traversal/symlinks,
instrucciones inyectadas, intentos de leer secretos y limites de coste/tiempo.
CrewAI nunca forma parte del procesamiento de pagos en runtime.