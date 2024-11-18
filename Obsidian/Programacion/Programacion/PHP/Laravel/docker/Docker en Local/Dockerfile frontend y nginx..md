
# Dockerfile básica del front para docker
`
```
# Usa la imagen oficial de Node.js como base

FROM node:20.16.0-alpine AS build

  

# Establece el directorio de trabajo

WORKDIR /app

  

# Copia los archivos de configuración y las dependencias

COPY package*.json ./

RUN npm install --force

  

# Copia el código fuente de la aplicación

COPY . .

  

# Construye la aplicación

RUN npm run build:prd

  

# Use a lightweight base image for serving the Angular app

FROM nginx:1.21-alpine

  

# Copia los archivos estáticos construidos desde la etapa anterior

COPY --from=build /app/dist/la-pecera-web /usr/share/nginx/html

  

COPY nginx.conf /etc/nginx/nginx.conf

  

# Expose port 80 for the web server

EXPOSE 80

  

# Comando para iniciar Nginx

CMD ["nginx", "-g", "daemon off;"]
```


# nginx básico dentro del front

```
events {}

http {

    include /etc/nginx/mime.types;

    server {

        listen 80;

        server_name localhost;

        root /usr/share/nginx/html;

        index index.html;

        location / {

            try_files $uri $uri/ /index.html;

        }

    }

}
```