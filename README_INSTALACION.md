# Reserva web: cliente y mascota nuevos

## Objetivo
Permitir que una persona haga una reserva desde la web aunque el cliente y/o la mascota todavía no estén registrados.

El flujo usa las tablas existentes:

`cliente -> mascota -> reserva`

No se modifica ninguna migración, columna ni estructura de tabla.

## Archivos incluidos

- `app/Http/Controllers/ReservaController.php`
- `app/Http/Requests/PublicReservaRequest.php` (nuevo)
- `resources/views/reserva/publica.blade.php`
- `resources/views/reserva/create.blade.php`
- `routes/web.php`

## Qué hace

1. El usuario entra a `/reservar`.
2. Completa sus datos.
3. Completa los datos de la mascota.
4. Selecciona fecha y horario.
5. El servidor busca el cliente por correo.
6. Si no existe, crea el cliente.
7. Busca la mascota por `cliente_id + nombre`.
8. Si no existe, crea la mascota.
9. Crea la reserva con `mascota_id`.
10. Todo el alta de cliente + mascota + reserva se ejecuta dentro de una transacción.

Si el cliente y mascota ya existen, se reutilizan y solo se crea la reserva.

## Instalación

Copiar los archivos respetando sus carpetas sobre el proyecto Laravel.

**No ejecutar `php artisan migrate` por estos cambios.**

La ruta pública es:

`GET /reservar`

El envío del formulario es:

`POST /reservar`

La reserva interna del personal continúa en:

`POST /reserva`

## Validaciones

- Cliente: nombre, correo, teléfono y dirección.
- Mascota: nombre, especie, raza, fecha de nacimiento, peso y sexo.
- Fecha de reserva no anterior a hoy.
- Hora no pasada cuando se reserva para hoy.
- Hora dentro del horario de atención.
- El servidor calcula `hora_fin` sumando 20 minutos.
- Se vuelve a comprobar el horario antes de insertar.
- Se mantiene la restricción única existente `(fecha, hora_inicio)`.
- Campo honeypot `sitio_web` prohibido.

## Pruebas realizadas

- `php -l app/Http/Requests/PublicReservaRequest.php` -> OK
- `php -l app/Http/Controllers/ReservaController.php` -> OK
- `php -l routes/web.php` -> OK
- Compilación directa del Blade -> OK
- `php artisan route:list --path=reserva` -> rutas públicas e internas presentes

`php artisan view:cache` no pudo ejecutarse en el entorno de prueba porque la instalación de PHP no tiene la extensión `DOM` (`DOMDocument`). Esto es una limitación del entorno, no un error de sintaxis del Blade.
