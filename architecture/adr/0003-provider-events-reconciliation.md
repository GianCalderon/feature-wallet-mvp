# ADR-0003: Eventos de proveedor, incertidumbre y conciliacion

- Fecha: 2026-10-23
- Estado: Propuesto.
- Referencias activas: HU-02, HU-04; Q-05, Q-07. HU-03/HU-05/HU-06 diferidas.

## Contexto

Los rieles son multiples, los resultados asincronos y las transferencias
ejecutadas no admiten cancelacion unilateral uniforme.

## Decision

Puertos por capacidad de proveedor y adaptadores simulados. Outbox/inbox durable
en PostgreSQL, workers recuperables del monolito, entrega al menos una vez y
consumidores idempotentes. No prometer exactly-once extremo a extremo.

Callbacks con autenticidad verificada, ID de evento unico, correlacion,
proteccion contra replay y validacion de transicion/version. Duplicados o eventos
fuera de orden no pueden degradar un resultado terminal ni duplicar asientos.
Estados tecnicos de recibido/en proceso/incierto son propuestas pendientes de
Q-05; la HU exige un unico resultado terminal exitoso, fallido o expirado.

Timeout no significa fallo ni expiracion segura. En incertidumbre consultar el
riel original y conciliar; no reenviar a otro proveedor ni liberar reservas sin
evidencia suficiente. Expiracion y callback tardio deben resolverse en el modelo
de estados antes de construir. Un reverso es otra operacion con autorizacion y
asientos compensatorios; no cambia retroactivamente el resultado del pago original.

La conciliacion diaria de HU-06 queda fuera del alcance activo. Como referencia
para una ampliacion futura, compararia intencion de wallet, evidencia del proveedor y
asientos del ledger. Mantener snapshot/corte, totales por moneda y canal,
descalces clasificados, investigacion auditada y reruns idempotentes. Proponer
UTC para timestamps y America/Lima para fechas operativas; Q-09 decide el corte.
En este incremento solo se conserva evidencia de originales/reintentos (HU-04
CA4) y recuperacion del resultado del pago; no hay modulo ni job diario HU-06.

## Alternativas

Kafka inicial agrega operacion sin eliminar idempotencia. Solo callbacks deja
huecos cuando se pierden eventos. Solo polling retrasa la visibilidad comercial.

## Consecuencias y verificacion

Disenar leases y reintentos acotados para workers. Probar duplicados, perdida,
firmas invalidas, eventos tardios, caidas del worker y ejecuciones repetidas de
control de duplicados. El simulador debe producir intencionalmente estos escenarios.