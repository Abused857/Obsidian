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


# Laravel Framework 11.14.0

## Usage


command [options] [arguments]

## Options

- `-h, --help`: Display help for the given command. When no command is given display help for the list command
- `-q, --quiet`: Do not output any message
- `-V, --version`: Display this application version
- `--ansi|--no-ansi`: Force (or disable --no-ansi) ANSI output
- `-n, --no-interaction`: Do not ask any interactive question
- `--env[=ENV]`: The environment the command should run under
- `-v|vv|vvv, --verbose`: Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

## Available Commands

### General

- `about`: Display basic information about your application
- `clear-compiled`: Remove the compiled class file
- `completion`: Dump the shell completion script
- `db`: Start a new database CLI session
- `docs`: Access the Laravel documentation
- `down`: Put the application into maintenance/demo mode
- `env`: Display the current framework environment
- `help`: Display help for a command
- `inspire`: Display an inspiring quote
- `list`: List commands
- `migrate`: Run the database migrations
- `optimize`: Cache framework bootstrap, configuration, and metadata to increase performance
- `serve`: Serve the application on the PHP development server
- `test`: Run the application tests
- `tinker`: Interact with your application
- `up`: Bring the application out of maintenance mode

### Auth

- `auth:clear-resets`: Flush expired password reset tokens

### Cache

- `cache:clear`: Flush the application cache
- `cache:forget`: Remove an item from the cache
- `cache:prune-stale-tags`: Prune stale cache tags from the cache (Redis only)

### Channel

- `channel:list`: List all registered private broadcast channels

### Config

- `config:cache`: Create a cache file for faster configuration loading
- `config:clear`: Remove the configuration cache file
- `config:publish`: Publish configuration files to your application
- `config:show`: Display all of the values for a given configuration file or key

### Database

- `db:monitor`: Monitor the number of connections on the specified database
- `db:seed`: Seed the database with records
- `db:show`: Display information about the given database
- `db:table`: Display information about the given database table
- `db:wipe`: Drop all tables, views, and types

### Environment

- `env:decrypt`: Decrypt an environment file
- `env:encrypt`: Encrypt an environment file

### Event

- `event:cache`: Discover and cache the application's events and listeners
- `event:clear`: Clear all cached events and listeners
- `event:list`: List the application's events and listeners

### Install

- `install:api`: Create an API routes file and install Laravel Sanctum or Laravel Passport
- `install:broadcasting`: Create a broadcasting channel routes file

### Key

- `key:generate`: Set the application key

### Language

- `lang:publish`: Publish all language files that are available for customization

### Make

- `make:cache-table [cache:table]`: Create a migration for the cache database table
- `make:cast`: Create a new custom Eloquent cast class
- `make:channel`: Create a new channel class
- `make:class`: Create a new class
- `make:command`: Create a new Artisan command
- `make:component`: Create a new view component class
- `make:controller`: Create a new controller class
- `make:enum`: Create a new enum
- `make:event`: Create a new event class
- `make:exception`: Create a new custom exception class
- `make:factory`: Create a new model factory
- `make:interface`: Create a new interface
- `make:job`: Create a new job class
- `make:listener`: Create a new event listener class
- `make:mail`: Create a new email class
- `make:middleware`: Create a new middleware class
- `make:migration`: Create a new migration file
- `make:model`: Create a new Eloquent model class
- `make:notification`: Create a new notification class
- `make:notifications-table [notifications:table]`: Create a migration for the notifications table
- `make:observer`: Create a new observer class
- `make:policy`: Create a new policy class
- `make:provider`: Create a new service provider class
- `make:queue-batches-table [queue:batches-table]`: Create a migration for the batches database table
- `make:queue-failed-table [queue:failed-table]`: Create a migration for the failed queue jobs database table
- `make:queue-table [queue:table]`: Create a migration for the queue jobs database table
- `make:request`: Create a new form request class
- `make:resource`: Create a new resource
- `make:rule`: Create a new validation rule
- `make:scope`: Create a new scope class
- `make:seeder`: Create a new seeder class
- `make:session-table [session:table]`: Create a migration for the session database table
- `make:test`: Create a new test class
- `make:trait`: Create a new trait
- `make:view`: Create a new view

### Migrate

- `migrate:fresh`: Drop all tables and re-run all migrations
- `migrate:install`: Create the migration repository
- `migrate:refresh`: Reset and re-run all migrations
- `migrate:reset`: Rollback all database migrations
- `migrate:rollback`: Rollback the last database migration
- `migrate:status`: Show the status of each migration

### Model

- `model:prune`: Prune models that are no longer needed
- `model:show`: Show information about an Eloquent model

### Optimize

- `optimize:clear`: Remove the cached bootstrap files

### Package

- `package:discover`: Rebuild the cached package manifest

### Queue

- `queue:clear`: Delete all of the jobs from the specified queue
- `queue:failed`: List all of the failed queue jobs
- `queue:flush`: Flush all of the failed queue jobs
- `queue:forget`: Delete a failed queue job
- `queue:listen`: Listen to a given queue
- `queue:monitor`: Monitor the size of the specified queues
- `queue:prune-batches`: Prune stale entries from the batches database
- `queue:prune-failed`: Prune stale entries from the failed jobs table
- `queue:restart`: Restart queue worker daemons after their current job
- `queue:retry`: Retry a failed queue job
- `queue:retry-batch`: Retry the failed jobs for a batch
- `queue:work`: Start processing jobs on the queue as a daemon

### Reverb

- `reverb:install`: Install the Reverb dependencies
- `reverb:restart`: Restart the Reverb server
- `reverb:start`: Start the Reverb server

### Route

- `route:cache`: Create a route cache file for faster route registration
- `route:clear`: Remove the route cache file
- `route:list`: List all registered routes

### Sail

- `sail:add`: Add a service to an existing Sail installation
- `sail:install`: Install Laravel Sail's default Docker Compose file
- `sail:publish`: Publish the Laravel Sail Docker files

### Sanctum

- `sanctum:prune-expired`: Prune tokens expired for more than specified number of hours

### Schedule

- `schedule:clear-cache`: Delete the cached mutex files created by scheduler
- `schedule:interrupt`: Interrupt the current schedule run
- `schedule:list`: List all scheduled tasks
- `schedule:run`: Run the scheduled commands
- `schedule:test`: Run a scheduled command
- `schedule:work`: Start the schedule worker

### Schema

- `schema:dump`: Dump the given database schema

### Storage

- `storage:link`: Create the symbolic links configured for the application
- `storage:unlink`: Delete existing symbolic links configured for the application

### Stub

- `stub:publish`: Publish all stubs that are available for customization

### Vendor

- `vendor:publish`: Publish any publishable assets from vendor packages

### View

- `view:cache`: Compile all of the application's Blade templates
- `view:clear`: Clear all compiled view files

[[php Artisan]]  [[Laravel instalación]]



