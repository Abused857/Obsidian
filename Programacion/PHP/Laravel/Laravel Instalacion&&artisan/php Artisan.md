# Comandos de `php artisan`


- **clear-compiled:**
  Elimina el archivo compilado de Laravel, `compiled.php`, optimizando el rendimiento al recargar clases.
  - **Palabras clave:** compiled, clear, optimize

- **optimize:**
  Mejora la eficiencia cargando menos archivos de clases.
  - **Palabras clave:** optimize, efficiency

- **config:clear:**
  Limpia la caché de configuraciones, forzando la recarga desde los archivos de configuración.
  - **Palabras clave:** config, clear, cache

- **cache:clear:**
  Elimina el caché de la aplicación, incluyendo rutas y configuraciones.
  - **Palabras clave:** cache, clear

- **route:clear:**
  Borra la caché de rutas, obligando a Laravel a reconstruirlas en la siguiente solicitud.
  - **Palabras clave:** route, clear, rebuild

- **view:clear:**
  Limpia el caché de vistas, asegurando que se vuelvan a generar cuando sea necesario.
  - **Palabras clave:** view, clear, cache

- **config:cache:**
  Crea un archivo de caché de configuraciones para optimizar la carga de configuraciones.
  - **Palabras clave:** config, cache, optimize

- **route:cache:**
  Crea un archivo de caché de rutas para acelerar la carga de rutas en la aplicación.
  - **Palabras clave:** route, cache, optimize

- **view:cache:**
  Crea un archivo de caché de vistas para mejorar el rendimiento al cargar vistas.
  - **Palabras clave:** view, cache, optimize

- **migrate:**
  Ejecuta todas las migraciones pendientes en la base de datos.
  - **Palabras clave:** migrate, database

- **migrate:fresh --seed:**
  Reinicia y vuelve a ejecutar todas las migraciones, sembrando la base de datos con datos de prueba.
  - **Palabras clave:** migrate, fresh, seed, database

- **storage:link:**
  Crea un enlace simbólico para acceder a archivos almacenados desde la web.
  - **Palabras clave:** storage, link, symbolic

- **key:generate:**
  Genera una nueva clave de aplicación para encriptación y sesiones.
  - **Palabras clave:** key, generate, encryption

- **queue:restart:**
  Reinicia todos los trabajos en cola después de realizar cambios.
  - **Palabras clave:** queue, restart

- **route:list:**
  Muestra una lista de todas las rutas registradas en la aplicación.
  - **Palabras clave:** route, list

- **make:model NombreModelo:**
  Crea un nuevo modelo en la aplicación.
  - **Palabras clave:** make, model, create

- **make:controller NombreControlador:**
  Crea un nuevo controlador en la aplicación.
  - **Palabras clave:** make, controller, create

- **make:middleware NombreMiddleware:**
  Crea un nuevo middleware en la aplicación.
  - **Palabras clave:** make, middleware, create

- **make:migration NombreMigracion:**
  Crea una nueva migración de base de datos.
  - **Palabras clave:** make, migration, create

- **db:seed:**
  Ejecuta el seeder para poblar la base de datos con datos de prueba.
  - **Palabras clave:** db, seed, populate

- **make:policy NombrePolitica:**
  Crea una nueva política de autorización.
  - **Palabras clave:** make, policy, create

- **make:job NombreJob:**
  Crea un nuevo trabajo en segundo plano (job).
  - **Palabras clave:** make, job, create

- **make:notification NombreNotificacion:**
  Crea una nueva notificación.
  - **Palabras clave:** make, notification, create

- **make:factory NombreFactoria:**
  Crea una nueva fábrica para generar datos de prueba.
  - **Palabras clave:** make, factory, create

- **make:request NombreRequest:**
  Crea una nueva clase de solicitud (request).
  - **Palabras clave:** make, request, create

- **down:**
  Pone la aplicación en modo de mantenimiento.
  - **Palabras clave:** down, maintenance

- **up:**
  Saca la aplicación del modo de mantenimiento.
  - **Palabras clave:** up, maintenance

- **schedule:run:**
  Ejecuta los trabajos programados definidos en `App\Console\Kernel`.
  - **Palabras clave:** schedule, run, cron

- **vendor:publish:**
  Publica cualquier recurso de un paquete de proveedor que no se haya publicado antes.
  - **Palabras clave:** vendor, publish

- **test:**
  Ejecuta las pruebas de PHPUnit.
  - **Palabras clave:** test, PHPUnit

- **make:test NombrePrueba:**
  Crea una nueva prueba PHPUnit.
  - **Palabras clave:** make, test, create

- **optimize:clear:**
  Elimina todos los archivos generados por el comando `optimize`.
  - **Palabras clave:** optimize, clear

- **config:dump:**
  Muestra la configuración actual en formato PHP.
  - **Palabras clave:** config, dump

- **event:generate:**
  Genera y registra los archivos necesarios para un evento.
  - **Palabras clave:** event, generate

- **session:table:**
  Crea una migración para la tabla de sesiones de la base de datos.
  - **Palabras clave:** session, table, database

- **storage:logs:**
  Muestra los logs almacenados en `storage/logs`.
  - **Palabras clave:** storage, logs

- **make:livewire NombreComponente:**
  Crea un nuevo componente Livewire.
  - **Palabras clave:** make, livewire, create

- **view:publish:**
  Publica un paquete de vistas.
  - **Palabras clave:** view, publish

- **ui:**
  Instala las dependencias y scaffolding de frontend (Bootstrap, Vue, React, etc.).
  - **Palabras clave:** ui, frontend

- **auth:clear-resets:**
  Borra todos los tokens de restablecimiento de contraseña expirados.
  - **Palabras clave:** auth, clear, resets

- **route:cache:**
  Crea un archivo de caché de rutas para acelerar la carga de rutas en la aplicación.
  - **Palabras clave:** route, cache

- **migrate:refresh:**
  Deshace y vuelve a ejecutar todas las migraciones.
  - **Palabras clave:** migrate, refresh, database

- **schedule:work:**
  Inicia el worker de los trabajos programados.
  - **Palabras clave:** schedule, work, cron

- **schedule:finish:**
  Libera cualquier bloqueo de trabajos programados pendientes.
  - **Palabras clave:** schedule, finish, cron

- **make:component NombreComponente:**
  Crea un nuevo componente de Blade.
  - **Palabras clave:** make, component, create

- **db:wipe:**
  Elimina todas las tablas de la base de datos.
  - **Palabras clave:** db, wipe, database

- **app:name:**
  Muestra o establece el nombre de la aplicación.
  - **Palabras clave:** app, name

- **event:list:**
  Lista todos los eventos registrados y sus listeners.
  - **Palabras clave:** event, list

- **event:cache:**
  Crea un archivo de caché de eventos para acelerar su registro.
  - **Palabras clave:** event, cache

- **make:exception NombreExcepcion:**
  Crea una nueva excepción.
  - **Palabras clave:** make, exception, create

- **make:resource NombreRecurso:**
  Crea una nueva clase de recurso.
  - **Palabras clave:** make, resource, create

- **make:channel NombreCanal:**
  Crea un nuevo canal de notificación.
  - **Palabras clave:** make, channel, create

- **make:cast NombreCasteo:**
  Crea un nuevo casteo.
  - **Palabras clave:** make, cast, create

- **serve:**
  Inicia el servidor de desarrollo PHP integrado.
  - **Palabras clave:** serve, development

- **ui:controllers:**
  Instala los controladores de autenticación y registro.
  - **Palabras clave:** ui, controllers

- **ui:auth:**
  Instala el frontend de autenticación.
  - **Palabras clave:** ui, auth, frontend

- **ui:remove:**
  Elimina el frontend de autenticación y registros.
  - **Palabras clave:** ui, remove

  -  **make:notification NombreNotificacion:**
     Crea una nueva notificación
  - **Palabras clave:** make, notification, create
  
  - **make:mail NombreCorreo:** 
  - Crea un nuevo correo (mailable). -
  - **Palabras clave:** make, mail, create
  
  - **make:test NombrePrueba:** 
  - Crea una nueva prueba PHPUnit. -
  - **Palabras clave:** make, test, create

[[php Artisan]]  [[Laravel instalación]]



