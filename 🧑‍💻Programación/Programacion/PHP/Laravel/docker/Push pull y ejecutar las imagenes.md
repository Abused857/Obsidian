## **Configuración de Docker en Debian Bookworm**

### **Pasos para Instalar Docker en Debian Bookworm**

**Eliminar la Lista de Repositorios Incorrecta:**

sudo rm /etc/apt/sources.list.d/docker.list

**Agregar la Clave GPG Oficial de Docker:**

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

Configurar el Repositorio Estable de Docker para Debian:

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Actualizar la Base de Datos de Paquetes:

 sudo apt-get update

Instalar Docker:

sudo apt-get install docker-ce docker-ce-cli containerd.io

### **Autenticación y Gestión de Imágenes**

 **Iniciar Sesión en Docker Hub:**

sudo docker login

- - ngresa el nombre de usuario y contraseña de Docker Hub.
- **Etiquetar y Subir Imágenes a Docker Hub:**

docker tag beehub-api-app cagigasdev/abu:beehub-api-app
docker tag nginx cagigasdev/abu:nginx
docker tag mysql:5.7 cagigasdev/abu:mysql-5.7
docker tag phpmyadmin/phpmyadmin cagigasdev/abu:phpmyadmin

docker push cagigasdev/abu:beehub-api-app
docker push cagigasdev/abu:nginx
docker push cagigasdev/abu:mysql-5.7
docker push cagigasdev/abu:phpmyadmin

### **Descargar y Ejecutar Imágenes en la Máquina Virtual**

**Descargar Imágenes desde Docker Hub:**

sudo docker pull cagigasdev/abu:nginx
sudo docker pull cagigasdev/abu:beehub-api-app
sudo docker pull cagigasdev/abu:mysql
sudo docker pull cagigasdev/abu:phpmyadmin

Verificar Imágenes Disponibles Localmente:

sudo docker images

**rear y Ejecutar Contenedores:**

**Para `nginx`:**

sudo docker run -d --name nginx-container -p 80:80 cagigasdev/abu:nginx


Para `beehub-api-app`:


sudo docker run -d --name beehub-api-container -p 9000:9000 cagigasdev/abu:beehub-api-app

Para `mysql`:

sudo docker run -d --name mysql-container -e MYSQL_ROOT_PASSWORD=root_password -p 3306:3306 cagigasdev/abu:mysql

Para `phpmyadmin`:

sudo docker run -d --name phpmyadmin-container -p 8080:80 --link mysql-container:db -e PMA_HOST=db cagigasdev/abu:phpmyadmin

**Configurar Redes y Volúmenes (si es necesario):**

**Crear una Red:**

sudo docker network create my-network


Reiniciar Contenedores: 


sudo docker restart nginx-container
sudo docker restart beehub-api-container
sudo docker restart phpmyadmin-container
sudo docker restart mysql-container

### **Acceso y Configuración Adicional**

**Acceder al Contenedor de la Aplicación:**

sudo docker exec -it beehub-api-container /bin/bash


**Actualizar el Archivo `.env`:**

- Accede al contenedor y edita el archivo `.env` para configurar el `DB_HOST`:

nano /var/www/html/.env

- - Cambia la línea `DB_HOST` a `mysql-container`.
- **Reiniciar el Contenedor de Laravel:**

sudo docker restart beehub-api-container

Ejecutar Migraciones:

sudo docker exec -it beehub-api-container /bin/bash php artisan migrate:fresh --seed

Verificar la Red:

sudo docker network inspect my-network

Revisar los Registros del Contenedor de Laravel:

sudo docker logs beehub-api-container

### **Configurar Reinicio Automático de Contenedores**

Para asegurar que los contenedores se reinicien automáticamente al iniciar la máquina virtual:

sudo docker update --restart always nginx-container
sudo docker update --restart always beehub-api-container
sudo docker update --restart always phpmyadmin-container
sudo docker update --restart always mysql-container

archivo de configuración de nginx en la ruta nano /etc/nginx/conf.d/default.conf:

server {

    listen 80;

    server_name localhost;

  

    root /var/www/html/public;

    index index.php index.html index.htm;

  

    location / {

        try_files $uri $uri/ /index.php?$query_string;

    }

  

    location ~ \.php$ {

        include fastcgi_params;

        fastcgi_pass app:9000;  

        fastcgi_index index.php;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }

  

    location ~ /\.ht {

        deny all;

    }

}
[[Laravel instalación]][[docker Laravel]]





