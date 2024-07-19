
### Paso 1: Configuración de Google API Credentials

Primero, asegúrate de tener configuradas las credenciales de API de Google:

1. Ve a Google Developer Console.
2. Crea un nuevo proyecto o selecciona uno existente.
3. Activa la API de Google que necesitas (por ejemplo, Google+ API o Google People API).
4. Configura las credenciales OAuth 2.0 para obtener tu `CLIENT_ID` y `CLIENT_SECRET`.
5. Asegúrate de establecer correctamente los URI de redireccionamiento autorizados, incluyendo `http://localhost:8000/api/auth/google/login` para entornos locales.

### Paso 2: Configuración de Laravel

Agrega las credenciales de Google a tu archivo `.env`:

`GOOGLE_CLIENT_ID=tu_client_id_de_google GOOGLE_CLIENT_SECRET=tu_client_secret_de_google`

### Paso 3: Instalación de Laravel Socialite

Si aún no lo has hecho, instala Laravel Socialite mediante Composer:

`composer require laravel/socialite`

### Paso 4: Configuración de Laravel Socialite

Edita tu archivo `config/services.php` para agregar la configuración de Google:

`'google' => [     'client_id' => env('GOOGLE_CLIENT_ID'),     'client_secret' => env('GOOGLE_CLIENT_SECRET'),     'redirect' => 'http://localhost:8000/api/auth/google/login', ],`

### Paso 5: Rutas y Controlador

#### Rutas

Define las rutas necesarias en `routes/api.php` para manejar el flujo de autenticación:

`Route::get('/auth/google', 'Auth\SocialiteController@redirectToGoogle'); Route::get('/auth/google/login', 'Auth\SocialiteController@handleGoogleCallback');`

#### Controlador

Crea un controlador `SocialiteController` usando el comando artisan:

`php artisan make:controller Auth/SocialiteController`

Implementa los métodos `redirectToGoogle` y `handleGoogleCallback` en `SocialiteController`:

```
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;
use App\Models\User;
use Illuminate\Support\Facades\Auth;

class SocialiteController extends Controller
{
    public function redirectToGoogle()
    {
        return Socialite::driver('google')->redirect();
    }

    public function handleGoogleCallback()
    {
        try {
            $user = Socialite::driver('google')->user();
            
            $existingUser = User::where('email', $user->getEmail())->first();
            
            if ($existingUser) {
                Auth::login($existingUser);
            } else {
                $newUser = new User();
                $newUser->name = $user->getName();
                $newUser->email = $user->getEmail();
                // Otros campos según necesidad
                $newUser->save();
                
                Auth::login($newUser);
            }
            
            return redirect()->to('/'); // Cambia esto al URL deseado después del login
        } catch (\Exception $e) {
            return redirect()->to('/login'); // Manejo de error, redirige al login por ejemplo
        }
    }
}

```

### Paso 6: Integración en la Aplicación Frontend

Asegúrate de tener un frontend configurado (por ejemplo, con Vue.js o React) que haga uso de estas rutas para iniciar sesión.

### Notas Adicionales

- **Seguridad**: Considera implementar validaciones y medidas de seguridad adicionales, como la verificación de dominios y el manejo seguro de tokens OAuth.
- **Entorno de Producción**: Asegúrate de configurar correctamente los URI de redireccionamiento en tu entorno de producción.
- **Personalización**: Ajusta las funciones y métodos según las necesidades específicas de tu aplicación.

Al seguir estos pasos, deberías poder implementar con éxito el login social con Google utilizando Laravel Socialite en tu aplicación. Asegúrate de probar exhaustivamente en diferentes entornos antes de desplegar en producción.

[[Controller]]  [[Socialite Login Social]]  [[Route-Api]]


