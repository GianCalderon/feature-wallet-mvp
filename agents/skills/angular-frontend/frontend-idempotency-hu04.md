---
name: frontend-idempotency-hu04
description: "Definir estados y reintentos frontend HU-04 que preserven el mismo intento y eviten cobros duplicados."
---

# Frontend Idempotencia HU-04

## Proposito

Definir el comportamiento visible del frontend para reintentos seguros. La
garantia de idempotencia es del backend/PostgreSQL; una pantalla o boton
deshabilitado no la reemplaza.

## Entradas

- HU-04 y criterios de aceptacion activos.
- Contrato aprobado para enviar, consultar y repetir un intento de pago.
- Decision de Q-07 sobre alcance, retencion, payload conflictivo y recuperacion.
- Estados y referencia definidos para HU-02.

## Salidas

- Flujo de reintento ligado al intento original.
- Mensajes diferenciados para resultado conocido y resultado incierto.
- Reglas de interaccion que previenen envios accidentales, sin prometer por si
  solas ausencia de duplicidad.
- Pruebas de repeticion y recuperacion, con evidencia real cuando se ejecuten.

## Flujo

1. Leer el contrato para determinar quien crea y conserva la clave de
   idempotencia. No inventar formato, ventana, alcance o politica de retencion.
2. Tratar el reintento de la misma operacion como el mismo intento: si el
   contrato entrega una clave al cliente, reutilizarla exactamente; no generar
   una clave nueva por doble clic, timeout o repeticion de la solicitud.
3. Bloquear doble submit como proteccion de UX, pero depender de la respuesta
   autoritativa del servidor para confirmar el resultado original.
4. Ante timeout, desconexion, HTTP 202 o respuesta no concluyente, mostrar
   estado pendiente/incierto y consultar el intento por su referencia. No
   iniciar otro pago ni cambiar de destino/riel mientras el resultado original
   pueda ejecutarse.
5. Tras recuperar la respuesta, mostrar el resultado original y su referencia.
   Permitir una nueva operacion solo cuando el contrato confirme que el intento
   anterior es terminal y la UI inicie una accion distinta.
6. No exponer ni registrar la clave de idempotencia. No guardarla en
   `localStorage`; para recuperar tras recarga, usar el mecanismo de consulta
   aprobado por el servidor.

## Restricciones

- No implementar ni simular la garantia financiera en frontend.
- No reintentar automaticamente ni convertir incertidumbre en fallo.
- No persistir datos de pago o claves fuera del mecanismo aprobado.
- No habilitar pagos antes de verificar las invariantes de HU-04.

## Verificacion

Cubrir doble clic, solicitud repetida con la misma clave, misma respuesta
original, timeout/202, recarga o retorno usando referencia, payload distinto y
clave vencida segun contrato. Los casos dependientes de Q-07 quedan bloqueados
hasta su resolucion; no declarar garantias basadas solo en pruebas de UI.