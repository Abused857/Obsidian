
#### Dockerfile

El Dockerfile configura una imagen Docker para la aplicación Laravel con Apache y PHP 8.2, e instala las extensiones necesarias de PHP, Composer, y configura Apache.

# Utiliza una imagen oficial de PHP con Apache
FROM php:8.2-apache

# Instala las extensiones de PHP requeridas
RUN docker-php-ext-install pdo pdo_mysql

# Habilita el módulo de reescritura de Apache
RUN a2enmod rewrite

# Instala Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copia el archivo de configuración de Apache
COPY apache-config.conf /etc/apache2/sites-available/000-default.conf

# Configura el directorio de trabajo
WORKDIR /var/www/html

# Copia los archivos del proyecto al contenedor
COPY . /var/www/html

# Instala las dependencias de Composer
RUN composer install

# Da permisos de escritura al directorio de almacenamiento y caché
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache /var/www/html/public

# Expone el puerto 80 para Apache
EXPOSE 80

# Inicia el servidor de Apache
CMD ["apache2-foreground"]


#### docker-compose.yml

Este archivo define los servicios necesarios para la aplicación, incluyendo una base de datos MariaDB, phpMyAdmin para la gestión de la base de datos, y el contenedor de la aplicación.


services:
  db:
    image: mariadb:10.4
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: beehub
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8080:80"

  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/var/www/html
    ports:
      - "8000:80"
    depends_on:
      - db

volumes:
  db_data:

##### Descripción de los Servicios

1. **db**: Servicio de base de datos utilizando MariaDB. La base de datos `beehub` se crea automáticamente y el acceso root está protegido por una contraseña especificada.
    
2. **phpmyadmin**: Servicio de phpMyAdmin para la administración de la base de datos MariaDB. Se expone en el puerto 8080.
    
3. **app**: Servicio de la aplicación Laravel. Se construye a partir del Dockerfile incluido y expone la aplicación en el puerto 8000. Este servicio depende del servicio de base de datos `db`.
    

#### .env

El archivo `.env` contiene todas las variables de entorno necesarias para configurar la aplicación Laravel. Aquí se define la configuración de la aplicación, la base de datos, y otros servicios externos.


APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:J/axx+gfsa5ShFkoIl2mxhTAMuuGFnWn2v7Qhso+QZw=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=db      
DB_PORT=3306
DB_DATABASE=beehub
DB_USERNAME=root
DB_PASSWORD=root

CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=database

SESSION_LIFETIME=120

APP_LOCALE=en
APP_FALLBACK_LOCALE=en
APP_FAKER_LOCALE=en_US

APP_MAINTENANCE_DRIVER=file
APP_MAINTENANCE_STORE=database

BCRYPT_ROUNDS=12

LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null

BROADCAST_CONNECTION=reverb
FILESYSTEM_DISK=local

CACHE_STORE=database
CACHE_PREFIX=

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=f96d6c2535c4f1
MAIL_PASSWORD=847538a5806a35
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

VITE_APP_NAME="${APP_NAME}"

# GOOGLE_CLIENT_ID=
# GOOGLE_CLIENT_SECRET=

REVERB_APP_ID=865244
REVERB_APP_KEY=ns87jknfzx0pmqxncjgn
REVERB_APP_SECRET=hqf5yhl3hnw79a5rwgbr
REVERB_HOST="localhost"
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"

FILESYSTEM_DRIVER=gcs

GOOGLE_CLOUD_PROJECT_ID=
GOOGLE_CLOUD_KEY_FILE=
GOOGLE_CLOUD_BUCKET=
GOOGLE_CLOUD_PATH_PREFIX=
GOOGLE_CLOUD_STORAGE_API_URI=
GOOGLE_CLOUD_STORAGE_API_ENDPOINT=


#### Consideraciones Adicionales

- **Seguridad**: Asegúrese de no exponer las credenciales de producción en el archivo `.env`. Se recomienda utilizar un sistema de gestión de secretos en un entorno de producción.
- **Volúmenes y Permisos**: Es importante manejar correctamente los volúmenes y permisos de archivos, especialmente para el almacenamiento y caché, para asegurar el correcto funcionamiento de la aplicación.
- **Configuración de Red**: Verifique que los puertos expuestos no entren en conflicto con otros servicios en la máquina host.

Esta documentación está destinada a ayudar tanto en la configuración inicial como en el mantenimiento continuo del entorno Docker para esta aplicación Laravel.

##### Levantar Contenedores

Para iniciar todos los servicios definidos en el archivo `docker-compose.yml`, utilice:

`docker-compose up -d`

El flag `-d` (detached) ejecuta los contenedores en segundo plano.

##### Detener Contenedores

Para detener y eliminar todos los contenedores, redes y volúmenes definidos en el archivo `docker-compose.yml`:

`docker-compose down`

##### Ejecutar Comandos Artisan

Para ejecutar comandos de Artisan, como migraciones, dentro del contenedor de la aplicación, utilice `docker-compose exec` seguido del nombre del servicio y el comando deseado. Por ejemplo, para ejecutar migraciones:

`docker-compose exec app php artisan migrate`

Si necesita acceder a una terminal dentro del contenedor de la aplicación para ejecutar otros comandos:

`docker-compose exec app bash`

Esto abrirá una sesión de terminal dentro del contenedor, desde donde se pueden ejecutar comandos adicionales de Artisan o cualquier otro comando necesario.


[[docker Laravel]]