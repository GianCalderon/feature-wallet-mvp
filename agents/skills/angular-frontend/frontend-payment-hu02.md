---
name: frontend-payment-hu02
description: "Definir el flujo frontend de pago P2P HU-02 identificado por DNI, con monto y confirmacion de resultado sin inferir estados financieros."
---

# Frontend Pago P2P HU-02

## Proposito

Delimitar la experiencia frontend para iniciar y consultar un pago P2P de bajo
monto, con DNI como identificador del destino. El DNI no es autenticacion ni
prueba de identidad.

## Entradas

- HU-02 y criterios de aceptacion activos.
- Contratos aprobados de consulta de destino, creacion/consulta de pago e
  historial.
- Diseno aprobado y resoluciones aplicables de Q-04 (monto), Q-05 (estados) y
  Q-11 (sesiones).
- Proteccion de idempotencia de HU-04 verificada antes de habilitar pagos.

## Salidas

- Flujo responsive para DNI destino, monto, confirmacion y resultado.
- Identidad minima del destino y monto visibles antes de confirmar.
- Referencia e historial para pagos exitosos, segun contrato aprobado.
- Estados de carga, validacion, pendiente/incierto, terminal y error, con
  pruebas/evidencia identificadas.

## Flujo

1. Confirmar que la tarea OpenSpec cubre HU-02 y que HU-04 protege la ejecucion.
   No habilitar una accion de pago si falta esa verificacion.
2. Tratar el DNI como string. Aplicar el formato aprobado; si aun no existe,
   proponer ocho digitos ASCII y marcarlo como pendiente de confirmacion.
3. Consultar el destino solo mediante el BFF. Mostrar la identidad minima
   devuelta por el contrato para que el pagador la revise; no exponer datos de
   terceros ni asumir que el DNI por si solo valida la identidad.
4. Representar el monto con moneda explicita PEN y precision decimal. No usar
   calculos `float`/`double` ni inventar limites, redondeos o reglas de bajo
   monto mientras Q-04 siga abierto.
5. Presentar identidad y monto antes de confirmar; evitar doble submit y cambios
   silenciosos en destino o monto tras la confirmacion.
6. Mostrar como terminal solo los estados autoritativos del contrato. HTTP 202,
   timeout, desconexion o estado desconocido significan pendiente/incierto, no
   pago fallido; consultar la referencia antes de permitir otro intento.
7. Mostrar referencia en exito e incluir el movimiento en el historial segun
   contrato. No agregar busqueda por concepto/filtros antes de resolver Q-17.

## Restricciones

- No guardar tokens ni DNI en `localStorage`, logs, URLs o telemetria.
- No llamar directamente a PostgreSQL, Meilisearch o proveedores desde el
  navegador.
- No inferir identidad, autorizacion, saldo ni resultado financiero en UI.
- No iniciar construccion antes del gate humano y el control de HU-04.

## Verificacion

Cubrir DNI invalido, destino no encontrado o respuesta inconsistente, revision
de monto/destino, submit repetido, HTTP 202, timeout, cada estado terminal,
referencia e historial. Probar accesibilidad y layouts responsive; adjuntar solo
resultados realmente ejecutados.