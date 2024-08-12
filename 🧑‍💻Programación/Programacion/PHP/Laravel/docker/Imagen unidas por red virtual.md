
### Documentación para Configuración de Aplicaciones y Base de Datos en Docker

Esta documentación explica la configuración para desplegar una aplicación Laravel y una base de datos MySQL usando Docker. Ambas aplicaciones están conectadas a través de una red virtual definida en Docker Compose. A continuación, se describen los archivos `docker-compose.yml`, `Dockerfile` y cómo configurar la red virtual para una comunicación efectiva entre los servicios.

#### Configuración de la Aplicación Laravel

##### docker-compose.yml (Aplicación)

Este archivo define los detalles del servicio para la aplicación Laravel, incluyendo la construcción de la imagen, los volúmenes, los puertos y la red. 

version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel_app
    volumes:
      - .:/var/www/html
    ports:
      - "8000:80"
    networks:
      - my_network

networks:
  my_network:
    external: true


##### Explicación

- **`version: '3.8'`**: Define la versión de la sintaxis del archivo Docker Compose. (Omitible/borrable).
    
- **`services`**: Define los servicios que se ejecutarán.
    
    - **`app`**: Servicio para la aplicación Laravel.
        - **`build`**: Especifica el contexto de construcción y el Dockerfile que se usará para construir la imagen.
        - **`container_name`**: Define un nombre específico para el contenedor (`laravel_app`).
        - **`volumes`**: Mapea el directorio actual del host (`.`) al directorio `/var/www/html` en el contenedor. Esto permite que los cambios en el código fuente del host se reflejen en el contenedor.
        - **`ports`**: Expone el puerto 80 del contenedor en el puerto 8000 del host, permitiendo acceder a la aplicación en `http://localhost:8000`.
        - **`networks`**: Conecta el servicio a una red externa llamada `my_network`.
- **`networks`**: Define una red externa llamada `my_network`. Esta red debe estar creada previamente en Docker, y se utiliza para conectar el servicio `app` con el servicio de base de datos.
    

#### Configuración de la Base de Datos MySQL

##### docker-compose.yml (Base de Datos)

Este archivo define los servicios para la base de datos MySQL y phpMyAdmin, junto con los volúmenes y la red.


version: '3.8'

services:
  db:
    image: mysql:5.7
    container_name: mysql_database
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: beehub
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - my_network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8080:80"
    depends_on:
      - db
    networks:
      - my_network

volumes:
  dbdata:

networks:
  my_network:
    external: true
##### Explicación

- **`version: '3.8'`**: Define la versión de la sintaxis del archivo Docker Compose. (Se puede borrar esto para evitar el warning que da)
    
- **`services`**: Define los servicios que se ejecutarán.
    
    - **`db`**: Servicio para la base de datos MySQL.
        
        - **`image`**: Utiliza la imagen oficial de MySQL 5.7.
        - **`container_name`**: Define un nombre específico para el contenedor (`mysql_database`).
        - **`environment`**: Configura las variables de entorno para la base de datos, como la contraseña del root y el nombre de la base de datos.
        - **`ports`**: Expone el puerto 3306 del contenedor en el puerto 3306 del host, permitiendo acceder a la base de datos desde el host.
        - **`volumes`**: Mapea el volumen `dbdata` al directorio `/var/lib/mysql` en el contenedor para persistir los datos de la base de datos.
        - **`networks`**: Conecta el servicio a una red externa llamada `my_network`.
    - **`phpmyadmin`**: Servicio para phpMyAdmin.
        
        - **`image`**: Utiliza la imagen oficial de phpMyAdmin.
        - **`container_name`**: Define un nombre específico para el contenedor (`phpmyadmin`).
        - **`environment`**: Configura las variables de entorno para phpMyAdmin, incluyendo el host de la base de datos y la contraseña de root.
        - **`ports`**: Expone el puerto 80 del contenedor en el puerto 8080 del host, permitiendo acceder a phpMyAdmin en `http://localhost:8080`.
        - **`depends_on`**: Define que `phpmyadmin` debe esperar a que el contenedor `db` esté listo antes de iniciar.
        - **`networks`**: Conecta el servicio a la misma red externa `my_network`.
- **`volumes`**: Define un volumen llamado `dbdata` para almacenar los datos de la base de datos.
    
- **`networks`**: Define una red externa llamada `my_network`, que debe estar creada previamente en Docker.
    

#### Dockerfile para la Base de Datos

El Dockerfile para la base de datos se utiliza para personalizar la imagen de MySQL.

# Usa la imagen oficial de MySQL como base
FROM mysql:5.7

# Configura las variables de entorno para MySQL
ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_DATABASE=beehub


##### Explicación

- **`FROM mysql:5.7`**: Utiliza la imagen oficial de MySQL 5.7 como base.
- **`ENV MYSQL_ROOT_PASSWORD=root`**: Establece la contraseña del usuario root.
- **`ENV MYSQL_DATABASE=beehub`**: Crea una base de datos llamada `beehub`.

#### Consideraciones Adicionales

1. **Red Virtual (`my_network`)**: Ambos archivos Docker Compose utilizan la misma red virtual `my_network` para permitir que los contenedores `app` y `db` se comuniquen entre sí. Asegúrese de que esta red esté creada antes de iniciar los contenedores.
    
    Crear la red externa:
    
docker network create my_network

- **Persistencia de Datos**: El volumen `dbdata` asegura que los datos de la base de datos se mantengan entre reinicios del contenedor y se persistan a largo plazo.
    
- **Seguridad**: Asegúrese de manejar las contraseñas y las credenciales de manera segura, especialmente en entornos de producción.
    
- **Acceso a la Aplicación y phpMyAdmin**: Acceda a la aplicación Laravel en `http://localhost:8000` y a phpMyAdmin en `http://localhost:8080` para la gestión de la base de datos.

[[Laravel instalación]][[PHP]][[docker Laravel]]
