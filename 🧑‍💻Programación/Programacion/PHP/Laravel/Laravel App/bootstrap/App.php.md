```
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        channels: __DIR__.'/../routes/channels.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'AuthenticationChannel' => \App\Http\Middleware\AuthenticationChannel::class,
            'Authentication' => \App\Http\Middleware\Authentication::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();

```

### Ejemplo de Uso en las Rutas API

En tu archivo de rutas `api.php`, puedes aplicar los middlewares utilizando sus alias:

```
use Illuminate\Support\Facades\Route;

// Rutas públicas sin autenticación
Route::post('user/loginsocial', [\App\Http\Controllers\UserController::class, 'loginSocial']);

// Rutas que requieren autenticación del canal
Route::middleware('AuthenticationChannel')->group(function () {
    Route::post('user/remembertoken', [\App\Http\Controllers\UserController::class, 'RememberToken']);
    Route::post('user/refreshtoken', [\App\Http\Controllers\UserController::class, 'RefreshToken']);
});

// Rutas que requieren autenticación completa
Route::middleware('Authentication')->group(function () {
    Route::post('user/logout', [\App\Http\Controllers\UserController::class, 'logout']);
    Route::get('user/profile', [\App\Http\Controllers\UserController::class, 'profile']);
    Route::get('user/ranking', [\App\Http\Controllers\UserController::class, 'ranking']);
    
    Route::post('/validtoken', [\App\Http\Controllers\GeneralController::class, 'validtoken']);
    
    Route::get('/article', [\App\Http\Controllers\ArticleController::class, 'show']);
    
    Route::post('action/favourite', [\App\Http\Controllers\ActionsController::class, 'createUserActionFavourite']);
    Route::delete('action/favourite', [\App\Http\Controllers\ActionsController::class, 'deleteFavourite']);
    Route::get('/action', [\App\Http\Controllers\ActionsController::class, 'show']);
    Route::post('/action/askpoints', [\App\Http\Controllers\ActionsController::class, 'askPoints']);
    
    Route::get('/wiki', [\App\Http\Controllers\WikiController::class, 'show']);
});

```

### Explicación

1. **Middleware Aliases**: En el archivo `app.php`, defines aliases para los middlewares `AuthenticationChannel` y `Authentication`. Esto permite referenciar los middlewares por nombres más cortos y claros en las rutas.
    
2. **Uso en las Rutas**: En el archivo `api.php`, aplicas los middlewares utilizando los aliases. Por ejemplo, `Route::middleware('AuthenticationChannel')` asegura que las rutas dentro de ese grupo solo sean accesibles si pasan la autenticación del canal. De manera similar, `Route::middleware('Authentication')` garantiza que las rutas dentro de ese grupo requieran una autenticación completa del usuario.
    
3. **Funcionamiento**: Cada grupo de rutas especifica qué middlewares deben aplicarse. Las rutas públicas o menos sensibles pueden no requerir autenticación, mientras que las rutas que acceden a datos sensibles o realizan acciones importantes pueden requerir autenticación del canal o del usuario.
    

Al configurar los middlewares de esta manera, puedes controlar de manera efectiva el acceso a diferentes partes de tu API, garantizando la seguridad y la autorización adecuadas para cada solicitud.

Para superar los dos middlewares `AuthenticationChannel` y `Authentication` en tus rutas API, debes cumplir con las condiciones que cada uno verifica antes de permitir el acceso a la solicitud. Aquí tienes un resumen de lo que necesitas hacer para cada middleware:

### Middleware `AuthenticationChannel`

Este middleware verifica la autenticación basada en un token específico del canal. Para superarlo, tu solicitud debe incluir un encabezado `ChannelToken` con el token de autorización correcto que coincida con un registro en la base de datos `Channel`.

#### Ejemplo de Superación:

```
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'ChannelToken' => 'your_channel_token_here',
])->post('http://example.com/api/v1/user/remembertoken');

echo $response->status(); // Debería devolver 200 si la autenticación del canal es exitosa

```

### Middleware `Authentication`

Este middleware realiza varias verificaciones, incluida la validación de tokens de usuario y la comprobación de otras condiciones como la empresa activa, el estado de bloqueo del usuario, el perfil completado, etc.

#### Ejemplo de Superación:

Para superar este middleware, debes incluir un encabezado `Authorization` con un token válido para un usuario existente y cumplir con las condiciones específicas que se verifican dentro del middleware. Aquí hay un ejemplo básico:

```
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Authorization' => 'Bearer your_user_token_here',
])->get('http://example.com/api/v1/user/profile');

echo $response->status(); // Debería devolver 200 si la autenticación del usuario y las condiciones del middleware son satisfactorias

```

### Consideraciones:

- Asegúrate de manejar correctamente los tokens y los encabezados necesarios según la implementación específica en tu aplicación.
- Los ejemplos anteriores utilizan `Illuminate\Support\Facades\Http` para enviar solicitudes HTTP y son simplificados para demostrar el principio de superación de los middlewares. En una implementación real, la estructura y la lógica pueden variar según los requisitos de tu aplicación y el diseño de tus rutas y controladores.
- Es fundamental entender las condiciones exactas que cada middleware verifica para garantizar que tus solicitudes puedan pasar exitosamente a través de ellos.
 [[Laravel instalación]] [[Middleware]]