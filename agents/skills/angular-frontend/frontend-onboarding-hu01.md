---
name: frontend-onboarding-hu01
description: "Definir el onboarding frontend HU-01 con nombre, correo electronico, DNI y numero de celular, validaciones accesibles y datos ficticios."
---

# Frontend Onboarding HU-01

## Proposito

Delimitar el trabajo frontend del onboarding de pagadores. Esta skill propone
estructura y validaciones; no afirma que exista una aplicacion implementada ni
resuelve decisiones de KYC o retencion.

## Entradas

- HU-01 y criterios de aceptacion activos.
- Diseno y contrato de onboarding aprobados.
- Version/patrones Angular y politica de validacion disponibles.
- Resolucion de Q-01 para datos minimos, KYC simulado y retencion, cuando afecte
  al flujo.

## Salidas

- Flujo responsive con pasos y progreso visibles.
- Formulario con nombre, correo electronico, DNI y numero de celular.
- Revisión editable antes del envio, estados de respuesta y errores accionables.
- Pruebas propuestas o ejecutadas con evidencia claramente identificada.

## Flujo

1. Confirmar que la tarea OpenSpec autoriza HU-01 y que el contrato soporta los
   cuatro campos; reportar cualquier incompatibilidad como bloqueo.
2. Usar controles tipados y validacion centralizada conforme a los patrones del
   proyecto. No duplicar reglas entre pantalla, modelo y servicio.
3. Proponer las siguientes validaciones de formato, sujetas a aprobacion del
   contrato: nombre no vacio tras quitar espacios externos; correo validado sin
   una expresion regular restrictiva; DNI como string de ocho digitos ASCII;
   celular peruano local de nueve digitos que comienza con 9, con `+51` separado.
4. Mantener `documentType` fijo en `DNI` y enviar el documento como string; no
   convertirlo a numero ni eliminar ceros iniciales.
5. Asociar cada error a su campo con texto, label, estado accesible y foco
   recuperable. Permitir corregir y revisar los datos antes del envio.
6. Mostrar carga y resultado; bloquear doble envio. No afirmar activacion hasta
   recibir el estado autoritativo definido por el contrato.
7. Limpiar datos personales al completar o abandonar el flujo; no incluirlos en
   logs, URLs, analitica ni almacenamiento persistente del navegador.

## Restricciones

- Usar solo identidades sinteticas. DNI y celular no prueban identidad ni
  sustituyen el KYC simulado.
- No inventar endpoint, pasos KYC, consentimiento, politica de retencion ni
  eventos analiticos; resolver las preguntas abiertas con PO/Security.
- No iniciar construccion antes de la aprobacion humana del cambio.

## Verificacion

Cubrir campos requeridos, formatos propuestos, errores accesibles, revision y
edicion, doble envio, respuesta de activacion y limpieza de datos. Ejecutar
TestBed/Playwright solo cuando existan proyecto, comandos y entorno; documentar
lo pendiente sin simular resultados.