## Instalación de Laravel

https://laravel.com/docs/11.x/installation

### Requisitos Previos:
- PHP 7.4 o superior
- Composer
- Servidor web (opcional) puede ser local

### Pasos para la Instalación:

1. **Instalar Laravel:**
   - Ejecutar en la terminal:
     ```
     composer global require laravel/installer
     ```

2. **Crear un Nuevo Proyecto:**
   - Crear un nuevo proyecto Laravel:
     ```
     laravel new nombre-del-proyecto
     ```
   - O alternativamente, utilizando Composer:
     ```
     composer create-project --prefer-dist laravel/laravel nombre-del-proyecto
     ```

3. **Configurar el Entorno:**
   - Navegar al directorio del proyecto:
     ```
     cd nombre-del-proyecto
     ```
   - Copiar archivo de configuración `.env.example`:
     ```
     cp .env.example .env
     ```

[[PHP]]
