### Middleware de Autenticación (`Authentication`)

El middleware `Authentication` que has creado se encarga de verificar la autenticación y la autorización de las solicitudes entrantes antes de que lleguen a su destino final. Aquí tienes una explicación detallada de su funcionamiento y puntos clave:

#### Función `handle`


```
public function handle($request, Closure $next)
{
    $response = null;
    $response = \response()->json(['error' => 401, 'error_description' => 'Unauthorized.'], 401);
    
    // Verifica si se proporciona el encabezado de autorización
    if ($request->header('Authorization')) {
        $token = explode(' ', $request->header('Authorization'));
        
        // Verifica si se proporciona un token válido
        if (isset($token[1])) {
            $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', Carbon::now())->first();
            
            // Verifica si el token es válido y pertenece a un usuario activo
            if ($user_token != '') {
                $user = User::where('id', $user_token->user_id)->first();
                
                // Verifica si la empresa asociada al usuario está activa
                $company = Company::where('id', $user->company_id)
                    ->where('db_companies.date_start', '<=', Carbon::now()->format('Y-m-d'))
                    ->where('db_companies.date_end', '>=', Carbon::now()->format('Y-m-d'))
                    ->first();         
                
                // Verifica condiciones de acceso adicionales (perfil completado, rutas específicas permitidas)
                if ($user && $user->bloqued_login != 1 && ($user->bloqued_to == null || $user->bloqued_to < Carbon::now()) && $company && $user->verified_email == 1) {
                    if ($user->profile_completed == 1 ||
                        ($request->path() == 'api/v1/user/profile' || $request->path() == 'api/v1/company/users' && $request->method() == 'GET') ||
                        $request->path() == 'api/v1/user/register/social' ||
                        ($request->path() == 'api/v1/user' && $request->method() == 'GET') ||
                        $request->path() == 'api/v1/user/logout') {
                        
                        // Si se cumplen las condiciones, permite la solicitud pasar al siguiente middleware o controlador
                        $response = $next($request);
                    } else {
                        // Devuelve un error de "Precondición requerida" si las condiciones no se cumplen
                        $response = \response()->json(['error' => 428, 'error_description' => 'Precondition Required.'], 428);
                    }
                }
            }
        }
    }

    return $response;
}

```
#### Explicación

1. **Verificación de Autorización**: Comprueba si la solicitud incluye un encabezado `Authorization` y extrae el token de acceso.
    
2. **Validación del Token**: Consulta la base de datos para verificar si el token proporcionado es válido y aún no ha expirado.
    
3. **Validación del Usuario**: Verifica si el token pertenece a un usuario válido y activo en el sistema.
    
4. **Verificación de Empresa Activa**: Asegura que la empresa asociada al usuario esté activa en las fechas actuales.
    
5. **Condiciones de Acceso**: Evalúa otras condiciones de acceso, como la completitud del perfil del usuario y las rutas específicas permitidas.
    
6. **Permitir o Denegar Acceso**: Decide si permitir que la solicitud continúe al siguiente middleware o controlador según las condiciones evaluadas.
    

#### Puntos Claves

- **Seguridad**: Asegura que solo usuarios autenticados y verificados, pertenecientes a empresas activas, puedan acceder a rutas protegidas.
    
- **Control de Acceso**: Implementa una lógica detallada para permitir o denegar el acceso basado en varias condiciones como el estado del perfil del usuario y las rutas permitidas.
    
- **Manejo de Errores**: Devuelve respuestas de error claras y significativas (`401 Unauthorized`, `428 Precondition Required`) para comunicar problemas de acceso al cliente.
    

Este middleware es esencial para garantizar la seguridad y la integridad de tu API, asegurando que solo usuarios autorizados puedan realizar operaciones sensibles y protegidas.

### Middleware de Autenticación del Canal (`AuthenticationChannel`)

El middleware `AuthenticationChannel` que has creado se encarga de autenticar las solicitudes entrantes basadas en un token específico del canal. A continuación, te explico su funcionamiento y los puntos clave:

#### Función `handle`

```
public function handle($request, Closure $next)
{
    $error = 0;
    
    // Verifica si se proporciona el encabezado 'ChannelToken'
    if ($request->header('ChannelToken')) {
        $token = explode(' ', $request->header('ChannelToken'));
        
        // Verifica si se proporciona un token válido
        if (isset($token[1])) {
            // Busca el token de autorización del canal en la base de datos
            $channel_token = Channel::where('authorization_token', '=', $token[1])
                                    ->where('id', $request->input('channel_id'))
                                    ->first();
            
            // Verifica si se encuentra un token válido para el canal especificado
            if ($channel_token != '') {
                $error = 0; // No hay errores de autenticación
            } else {
                $error = 1; // Token no válido o canal incorrecto
            }
        } else {
            $error = 1; // Token no válido
        }
    } else {
        $error = 1; // Falta el encabezado 'ChannelToken'
    }

    // Si no hay errores, permite que la solicitud continúe al siguiente middleware o controlador
    if ($error == 0) {
        $response = $next($request);
    } else {
        // Si hay errores, devuelve una respuesta de error de "No autorizado"
        $response = \response()->json(['error' => 401, 'error_description' => 'Unauthorized.'], 401);
    }

    return $response;
}

```


#### Explicación

1. **Verificación de `ChannelToken`**: Comprueba si la solicitud incluye el encabezado `ChannelToken`.
    
2. **Validación del Token**: Divide y verifica la estructura del token para asegurarse de que esté completo y válido.
    
3. **Búsqueda en la Base de Datos**: Consulta la base de datos para validar el token de autorización del canal y asegurarse de que coincida con el canal proporcionado (`channel_id`).
    
4. **Gestión de Errores**: Maneja varios casos de error, como token no válido, falta de encabezado `ChannelToken`, o canal no autorizado.
    
5. **Permitir o Denegar Acceso**: Decide si permitir que la solicitud continúe al siguiente middleware o controlador según la validez del token del canal.
    

#### Puntos Claves

- **Autenticación Basada en Token**: Implementa una capa de autenticación adicional que requiere un token específico del canal para acceder a ciertas partes de la API.
    
- **Seguridad**: Asegura que solo las solicitudes con un token de canal válido y correspondiente al canal correcto puedan acceder a recursos protegidos.
    
- **Manejo de Errores**: Proporciona respuestas de error (`401 Unauthorized`) claras y significativas cuando se encuentran problemas de autenticación.
    

Este middleware es útil para casos donde necesitas proteger ciertas partes de tu API con autenticación adicional basada en tokens específicos del canal, proporcionando una capa adicional de seguridad y control de acceso.

 [[Route-Api]] [[App.php]]