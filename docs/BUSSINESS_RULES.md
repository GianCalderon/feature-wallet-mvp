# REGLAS DE NEGOCIO PARA LA SOLUCIÓN

## TECNOLOGIA

- Web
- Web/Responsive

## NAVAGADORES

- Edge
- Google Chrome
- Safari

## ONBOARDING WALLET

Para realizar el alta del cliente se debe solicitar al usuario los siguientes datos:

- Tipo de Documento (DNI)
- Número de Documento (Campo númerico, longitud máxica de 8 caracteres)
- Número de Celular (Campo númerico del 0-9, longitud máxima de 9 caracteres)
- Correo (Alfanúmerico, sin caracteres especiales, con un dominio válido)
- Contraseña (Alfanúmerico con un máximo de 20 caracteres)

### Consideraciones del Onboarding:

- Si el cliente ya existe debe devolver un mensaje de validación indicando "El cliente ya existe"
- Si el cliente aún no existe luego de completar los campos obligatorios y seleccionar botón Registrar, debe devolver un mensaje de validación indicando "El cliente fue creado con éxito" mostrando la pantalla "Home del Wallet"

## LOGIN (El cliente de estar registrado previamente.)

- Se deben completar los siguientes campos (Tipo de Documento, Número de Documento y Contraseña)
- Si los datos son correctos luego de seleccionar el botón "Ingresar" debe mostrar la pantalla "Home del Wallet"
- Si los datos son incorrectos debe devolver un mensaje de validación indicando "Datos incorrectos"

## REALIZAR PAGO

Para realizar un pago se debe tener en cuenta lo siguiente:

- El cliente debe contar con saldo disponible mayor o igual al monto que desea transaccionar.
- Si no existe saldo disponible debe mostrar un mensaje indicando "Fondos insuficientes"
- El monto mínimo a pagar debe ser
- El monto máximo a pagar debe ser
- Luego de realizar el pago se debe actualizar el saldo disponible.
- Luego de realizar el pago se deben mostrar las últimas 10 trasacciones en un vista "Mis movimientos"
- Si se desea consultar mas de 10 movimientos debe exitir una opción "Ver mas movimientos", el cual debe devolver los movimientos de los últimos 90 días de manera incremental, al hacer scroll a la vista.
- Todos los pagos deben estar registrados en una base de datos "payments_history"


