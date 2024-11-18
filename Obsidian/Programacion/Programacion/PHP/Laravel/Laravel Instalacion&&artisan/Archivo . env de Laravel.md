# Ejemplo de archivo .env para proyecto Laravel

# Configuración de la Aplicación
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:rnjgGdNLeN8+Xw/FBPyOW4Esk2nMgz9V3IB4z3cbqLU=
APP_DEBUG=true
APP_URL=http://localhost

# Configuración de Log
LOG_CHANNEL=stack

# Configuración de la Base de Datos
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nombre_de_tu_base_de_datos
DB_USERNAME=usuario_de_tu_base_de_datos
DB_PASSWORD=contraseña_de_tu_base_de_datos

# Configuración de Cache y Sesiones
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

# Configuración de Redis (opcional)
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# Configuración de Email (ejemplo con Mailtrap)
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

# Configuración de AWS (opcional)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

# Configuración de Pusher (opcional)
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

# Mix Config (para Laravel Mix)
MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

# Archivo .env entero

```
APP_NAME=Laravel

APP_ENV=local

APP_KEY=base64:qIBN1RN43yife0S5xfV5BItEGahPWsHW2Ogqag+0ElY=

APP_DEBUG=true

APP_TIMEZONE=UTC

APP_URL=http://localhost

  

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

  

DB_CONNECTION= mysql

DB_HOST= 127.0.0.1

DB_PORT= 3306

DB_DATABASE= beehub

DB_USERNAME= root

DB_PASSWORD=

  

SESSION_DRIVER=database

SESSION_LIFETIME=120

SESSION_ENCRYPT=false

SESSION_PATH=/

SESSION_DOMAIN=null

  

BROADCAST_CONNECTION=reverb

FILESYSTEM_DISK=local

QUEUE_CONNECTION=database

  

CACHE_STORE=database

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

```

[[Laravel instalación]]