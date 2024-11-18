

# actualizar paquetes

>sudo apt update


# instalar git en maquina virtual debian

>sudo apt install git

# comprobar que git esta instalado y la version

>git --version

# configuracion global de git username

>git config --global user.name "Joe"

# configuracion global de git email

>git config --global user.email "gcagigas88@gmail.com"

# comprobar la configuración de git user y email

>git config --list

# editar manualmente la configuración del user y email

> nano ~/.gitconfig

# creamos nuestra carpeta donde guardaremos el front back y docker compose

> sudo mkdir dockerBeehub

# nos movemos a ella

> cd dockerBeehub

# permisos de bitbucket

> nos logeamos y en la rueda de settings seleccionamos contraseñas de aplicaciones y creamos una y guardamos ese token de momento

# hacemos git clone del repo que queramos y añadimos sudo delante para permisos, yo estoy empezando por el backend 

> sudo git clone https:.....


# auth git clone bitbucket

ahora en el permiso metemos el token que guardamos anteriormente

# comprobación de repositorio y ver contenido

> ls  -la {nombre de la carpeta}

# añadimos el repositorio como que es una dir segura/safe

>git config --global --add safe.directory /home/german_martinez/dockerBeehub/beehub-api
# nos movemos dentro del repo y vemos en que rama estamos

> git status / git branch

# mostramos todas las ramas en remoto incluido que tiene el repo

> git branch -a 

# o los cambios en el repo 

> git log --all --graph --decorate --oneline  / se sale presionando 'q'

# actualizamos todo

> sudo git fetch
# nos movemos a la rama remota  

 > sudo git checkout -b {nombre de la rama} {ubicación remota}
 
 
# traemos cualquier cambio

> sudo git pull


# creamos .env si no esta incluido y las variables deberían verse tal que así en este caso, comprobrar si corresponde con tu puerto de db nombre del host y demás


>DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=beehub
DB_USERNAME=user
DB_PASSWORD=userpassword
DB_COLLATION=utf8mb4_unicode_ci
APP_URL=host.docker.intern

# docker prueba .env personal


>
>APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:qIBN1RN43yife0S5xfV5BItEGahPWsHW2Ogqag+0ElY=
APP_DEBUG=true
APP_TIMEZONE=UTC
APP_URL=host.docker.internal
APP_LOCALE=en
APP_FALLBACK_LOCALE=en
APP_FAKER_LOCALE=en_US
APP_MAINTENANCE_DRIVER=file
APP_MAINTENANCE_STORE=database
BCRYPT_ROUNDS=12
LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=beehub
DB_USERNAME=user
DB_PASSWORD=userpassword
DB_COLLATION=utf8mb4_unicode_ci
SESSION_DRIVER=database
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null
BROADCAST_CONNECTION=reverb
FILESYSTEM_DISK=local
QUEUE_CONNECTION=database
CACHE_STORE=file
CACHE_PREFIX=
MEMCACHED_HOST=127.0.0.1
REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
MAIL_MAILER=
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=
MAIL_FROM_NAME="${APP_NAME}"
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false
VITE_APP_NAME="${APP_NAME}"
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
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

# si en la raiz al hacer ls -la no aparece .env usamos touch para crearle el archivo

>sudo touch .env

# accedemos a el y añadimos la data control+o guardamos, intro para confirmar y control+x para salir del edito

>sudo nano .env

# repetimos el mismo proceso con el front, aqui no necesitamos hacer nada extra como .env, simplemente hacemos el clone movemos a la rama de docker y ya simplemente la mantenemos actualizada con pull al origin

> git status / git branch -a
> sudo git checkout -b {nombre de la rama} {ubicación remota}
> sudo git pull / fetch
> 


# nos movemos a la raiz y creamos nuestro docker-compose.yml se deberia de ver la estructura tal que asi

>sudo touch docker-compose.yml

|---/beehub-api
|	        ---Dockerfile
|    
|---/beehub-web
|	        ---Dockerfile
|    
|---docker-compose.yml

# el composer hara el build del front y back yendo a cada carpeta, y el traefik, mysql y phpmyadmin se genera a partir de una imagen en la nube

Importante tener en cuenta que se hizo un redirect a solicitudes https porque el OAuth de google fuera de test (localhsot) así lo pide.

>services:
  traefik:
    image: traefik:v2.6
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=german.martinez@incentro.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
    ports:
      - "80:80"     # HTTP
      - "443:443"   # HTTPS
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "traefik-config:/etc/traefik"
      - "traefik-data:/letsencrypt"
    networks:
      - app-network
    restart: always
  frontend:
    build:
      context: ./beehub-web
      dockerfile: Dockerfile
    image: frontend
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`dvl.beehub.cloud`)"
      - "traefik.http.services.frontend.loadbalancer.server.port=80"
      - "traefik.http.routers.frontend.entrypoints=websecure"  # Forzar HTTPS
      - "traefik.http.routers.frontend.tls.certresolver=myresolver"
    networks:
      - app-network
    restart: always
  backend:
    build:
      context: ./beehub-api
      dockerfile: Dockerfile
    image: backend
    depends_on:
      - db
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.backend.rule=Host(`dvl.beehub.cloud`) && PathPrefix(`/api`)"
      - "traefik.http.services.backend.loadbalancer.server.port=80"
      - "traefik.http.routers.backend.entrypoints=websecure"  # Forzar HTTPS
      - "traefik.http.routers.backend.tls.certresolver=myresolver"
    networks:
      - app-network
    restart: always
      phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: rootpassword
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.phpmyadmin.rule=Host(`ddbb.beehub.cloud`)"
      - "traefik.http.services.phpmyadmin.loadbalancer.server.port=80"
      - "traefik.http.routers.phpmyadmin.entrypoints=websecure"  # Forzar HTTPS
      - "traefik.http.routers.phpmyadmin.tls.certresolver=myresolver"
    networks:
      - app-network
    restart: always
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: beehub
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - data:/var/lib/mysql
    networks:
      - app-network
    restart: always
networks:
  app-network:
    driver: bridge
volumes:
  data:
  traefik-config:
  traefik-data:


# ejecutamos

>sudo docker compose up --build -d

# comprobamos que esten levantados

>sudo docker ps

# ejecutar php artisan migrate:fresh --seed para que se  carguen datos de prueba incluido el channel token


>sudo docker exec -it dockerbeehub-backend-1 php artisan migrate:fresh --seed

Tener en cuenta que este añadido el client id en el front en enviroment.ts  y el channel token en la misma ubicación, se puede hacer en local en la rama correspondiente y luego hacer el pull en el vm o mediante codigo moverte en la vm en el front hasta el archivo y añadirlo.

El channel token se genera tras ejecutar el php artisan migrate:fresh --seed en la base de datos desplegada en db _channels el id 2,

se pondria en el front y se volvería a hacer sudo docker compose up --build  -d

# comandos antes de hacer compose up

# parar los contenedores

>sudo docker stop $(sudo docker ps -aq)

# eliminar contenedores

>sudo docker rm $(sudo docker ps -aq)

# eliminar imagenes

>sudo docker rmi $(sudo docker images -q)


# eliminar builds

>sudo docker builder prune -a



# eliminar volumenes (cuidado)

en el volumen es donde se guarda toda la info de la base de datos, da igual la cantidad de docker compose up que hagáis que no afecta al volumen y los datos van a persistir, si se hace el php migrate o demás comando pues ya cambiará, usar este comando si de verdad es necesario utilizarle, con los anteriores es suficiente para tener el docker limpio sin repetir imagenes/builds etc, el volumen no se sobreescribe.

>sudo docker volume prune



# iniciar los contenedores

>sudo docker start $(sudo docker ps -a -q)



[[Instalar docker en maquina virtual debian]] [[docker Laravel]]