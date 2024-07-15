#### Generar Modelos y Migraciones

- **Crear un Modelo:**
    
    `php artisan make:model nombreModelo`
    
    Este comando genera un nuevo modelo en Laravel.
    
- **Ejecutar Migraciones y Sembrar Datos:**
    
    
    `php artisan migrate --seed`
    
    Migra la base de datos y opcionalmente llena datos de prueba.
    
- **Crear una Migración para Crear una Tabla:**

    
    `php artisan make:migration create_nombreTabla_table`
    
    Crea una nueva migración para crear una tabla en la base de datos.
    
- **Des-hacer la Última Migración:**

    
    `php artisan migrate:rollback`
    
    Revierte la última migración realizada en la base de datos.
    

#### Servir la Aplicación

- **Iniciar Servidor de Desarrollo:**
    

    
    `php artisan serve`
    
    Inicia un servidor de desarrollo local.
    
- **Especificar Puerto para el Servidor:**
    
    
    `php artisan serve --port=3307`
    
    Inicia el servidor en un puerto específico (ej. 3307).
    

#### Gestión de Rutas y Archivos

- **Listar Todas las Rutas:**
    
    
    `php artisan route:list`
    
    Muestra todas las rutas registradas en la aplicación.
    
- **Crear Archivo de Rutas API:**
    
    
    `php artisan make:api`
    
    Genera un archivo de rutas específico para API.
    

#### Creación de Notificaciones y Correos Electrónicos

- **Crear una Notificación:**
    

    `php artisan make:notification NombreNotificacion`
    
    Genera un archivo de notificación en `app/Notifications`.
    
- **Crear un Correo Electrónico:**
    
    `php artisan make:mail NombreCorreo`
    
    Genera un archivo de correo electrónico en `app/Mail`.


[[php Artisan]]  