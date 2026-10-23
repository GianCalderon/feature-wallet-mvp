---
name: angular-frontend
description: "Aplicar convenciones transversales de Angular al integrar tareas frontend aprobadas, sin sustituir las skills especificas de HU."
---

# Frontend Angular

## Proposito

Definir practicas comunes para cambios Angular del wallet. Usar junto con la
skill de HU asignada; esta skill no combina ni amplía el alcance funcional.

## Entradas

- Tarea OpenSpec aprobada, criterios y escenarios de la HU asignada.
- Convenciones y contratos aprobados del frontend.
- Plan de Testing y revision de Security aplicables.

## Flujo

1. Revisar los [limites frontend](../../../frontend/README.md) y los patrones
   existentes antes de proponer componentes, servicios o dependencias.
2. Mantener responsabilidades claras: componentes para presentacion y flujo,
   servicios para integraciones y validadores reutilizables cuando corresponda.
3. Usar tipos explicitos, nombres descriptivos y funciones pequenas; evitar
   duplicar reglas de negocio o logica HTTP.
4. Construir layouts responsive y accesibles, con labels, foco y errores
   asociados al control; no depender solo del color.
5. Consumir solo el BFF con sesion protegida. No persistir tokens en
   `localStorage` ni acceder directamente a Meilisearch.
6. Proponer pruebas focalizadas; reportar comandos y evidencia solo si se
   ejecutaron realmente en el entorno disponible.

## Restricciones

- No inventar rutas, APIs, estados, limites financieros ni decisiones abiertas.
- No agregar vistas de HU diferidas ni busqueda por conceptos hasta resolver Q-17.
- No usar HTML dinamico inseguro ni exponer datos personales en logs, URLs o
  telemetria.
- No agregar dependencias ni afirmar que existe un proyecto Angular: aun no hay
  workspace frontend en el repositorio.

## Skills por historia

- [Onboarding HU-01](frontend-onboarding-hu01.md)
- [Pago P2P HU-02](frontend-payment-hu02.md)
- [Reintentos HU-04](frontend-idempotency-hu04.md)