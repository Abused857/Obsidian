## Configuración de Servicios Externos (`services.php`)

El archivo `services.php` en Laravel almacena las credenciales para servicios de terceros como Mailgun, AWS, y otros. Proporciona una ubicación centralizada para esta información, facilitando que los paquetes y la propia aplicación accedan de manera convencional a estas credenciales.

### Estructura del Archivo

El archivo `services.php` está estructurado en forma de matriz asociativa, donde cada clave representa un servicio diferente y sus respectivas credenciales.

```
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Third Party Services
    |--------------------------------------------------------------------------
    |
    | This file is for storing the credentials for third party services such
    | as Mailgun, Postmark, AWS and more. This file provides the de facto
    | location for this type of information, allowing packages to have
    | a conventional file to locate the various service credentials.
    |
    */

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

    'slack' => [
        'notifications' => [
            'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
            'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
        ],
    ],

    'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_CLIENT_SECRET'),
        'redirect' => 'http://localhost:8000/api/auth/google/login'
    ],

];

```

### Detalles de los Servicios Configurados

1. **Postmark**
    
    - **Token:** `POSTMARK_TOKEN`
        - Este token se utiliza para autenticar con el servicio de envío de correos Postmark.
2. **SES (Amazon Simple Email Service)**
    
    - **Key:** `AWS_ACCESS_KEY_ID`
    - **Secret:** `AWS_SECRET_ACCESS_KEY`
    - **Region:** `AWS_DEFAULT_REGION`
        - Estas credenciales son necesarias para acceder a AWS SES y enviar correos electrónicos desde la aplicación.
3. **Slack**
    
    - **Bot User OAuth Token:** `SLACK_BOT_USER_OAUTH_TOKEN`
    - **Default Channel:** `SLACK_BOT_USER_DEFAULT_CHANNEL`
        - Utilizadas para integrar la aplicación con Slack para enviar notificaciones y mensajes.
4. **Google**
    
    - **Client ID:** `GOOGLE_CLIENT_ID`
    - **Client Secret:** `GOOGLE_CLIENT_SECRET`
    - **Redirect URI:** `http://localhost:8000/api/auth/google/login`
        - Credenciales para la integración con Google OAuth para autenticación social.

### Uso en la Aplicación

Estas configuraciones se acceden mediante la función `env()`, que obtiene los valores de las variables de entorno definidas en el archivo `.env` correspondiente.
 [[Services.php]]  [[Socialite Login Social]] [[Controller Social Login]] [[Laravel instalación]]