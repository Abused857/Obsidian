# Dockerfile básica del back para docker con el apache ya instalado, no hace falta nada más.


```
# Use the official PHP image with Apache

FROM php:8.2-apache

  

# Install necessary extensions and tools

RUN apt-get update \

    && apt-get install -y unzip libzip-dev \

    && docker-php-ext-install pdo_mysql zip

  

# Enable Apache modules

RUN a2enmod rewrite

  

# Set the working directory

WORKDIR /var/www/html

  

# Copy project code

  

COPY . /var/www/html

  

RUN chown -R www-data:www-data /var/www/html/

  

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

  

RUN composer install

  

# Set permissions

RUN chown -R www-data:www-data /var/www/html/

  

# Expose port 80

EXPOSE 80

  

# Start Apache

CMD ["apache2-foreground"]

```




# .env backend para conectarse con el host servicio del mysql nombre de la ddbb y credenciales predefinidos ofrecidos por la imagen.


```
APP_URL=host.docker.internal


DB_CONNECTION= mysql

  

DB_HOST=db

  

DB_PORT= 3306

  

DB_DATABASE= beehub

  

DB_USERNAME= root

  

DB_PASSWORD=rootpassword

```


[[docker Laravel]]  [[Docker backend y apache]]  [[Dockerfile frontend y nginx.]]