---
name: java-backend
description: "Usar para tareas backend Java 21/Spring Boot, dominio puro, comandos PostgreSQL y proyeccion/busqueda Meilisearch."
---

# Backend Java

## Procedimiento

1. Revisar los [limites backend](../../../backend/README.md), contratos y tareas
   OpenSpec autorizadas. Si faltan aprobacion o herramientas, entregar propuesta.
2. Separar dominio sin Spring/JPA, aplicacion y adaptadores; aplicar validacion
   y autorizacion en servidor, no solo en frontend.
3. Confirmar movimiento/estado/outbox atomicamente en PostgreSQL. No incluir
   llamadas externas ni escrituras Meilisearch en la transaccion financiera.
4. Proyectar con ID estable, control de orden/version y checkpoint solo tras
   indexacion confirmada. Mantener consulta SQL por referencia y ownership actual.
5. Proponer o ejecutar, solo con permisos, checks en contenedores contra motores
   reales: dominio, limites, concurrencia, fallos del indice y reconstruccion.

## Salida y verificacion

Por tarea: HU/CA, escenario, archivos, cambios propuestos/aplicados y comandos
realmente ejecutados con resultado. No inventar builds ni implementacion existente.