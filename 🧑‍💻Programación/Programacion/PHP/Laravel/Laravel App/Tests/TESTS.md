### Paso 1: Creación de una Prueba Básica en Laravel

#### Creación de una prueba

1. Abre tu terminal y navega hasta el directorio raíz de tu proyecto Laravel.
    
2. Usa el comando Artisan para crear una nueva prueba:
    
    `php artisan make:test LevelTranslationTest`
    
    Esto creará un archivo `LevelTranslationTest.php` en el directorio `tests/Feature`.
    
3. Abre el archivo `LevelTranslationTest.php` y escribe tu prueba. Aquí tienes un ejemplo básico:


```
<?php

namespace Tests\Feature;

use App\Models\Level;
use App\Models\Language;
use App\Models\LevelTranslation;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class LevelTranslationTest extends TestCase
{
    use RefreshDatabase;

    public function test_valid_level_translation_instance()
    {
        $level = new Level();
        $language = new Language();

        $create = LevelTranslation::create([
            'level_id' => $level->id,
            'language_id' => $language->id,
            'name' => 'rojo',
            'description' => 'Lorem ipsum'
        ]);

        $this->assertInstanceOf(LevelTranslation::class, $create);
        $this->assertEquals('rojo', $create->name);
        $this->assertEquals('Lorem ipsum', $create->description);

        $this->assertDatabaseHas('db_level_translations', [
            'id' => $create->id,
            'level_id' => $create->level_id,
            'language_id' => $create->language_id,
            'name' => $create->name,
            'description' => $create->description
        ]);
    }
}

```

#### Explicación

- `RefreshDatabase`: Usa este trait para asegurar que tu base de datos se reinicie después de cada prueba, lo que mantiene el entorno de prueba limpio y aislado.
- `assertInstanceOf`, `assertEquals`: Métodos de aserción estándar para verificar la instancia de un objeto, comparar valores y asegurarse de que los datos se han guardado correctamente en la base de datos.

### Paso 2: Ejecución de Pruebas en Laravel

#### Ejecución de una prueba individual

Para ejecutar una prueba individual, usa el comando:

`php artisan test --filter test_valid_level_translation_instance`

Esto ejecutará solo la prueba `test_valid_level_translation_instance` dentro del archivo `LevelTranslationTest.php`.

### Paso 3: Ejecución de Todas las Pruebas

Para ejecutar todas las pruebas en tu proyecto, simplemente usa el comando:

`php artisan test`

Esto ejecutará todas las pruebas dentro de los directorios `tests/Feature` y `tests/Unit`.

### Ejemplos Avanzados de Pruebas en Laravel

#### Prueba con Otras Validaciones y Ejemplos

A continuación se muestra un ejemplo más complejo que implica la creación de múltiples modelos y la validación de una respuesta JSON:

```
public function test_two_users_ranking_creating_2_with_limit_one_and_level()
{
    // Creación de modelos necesarios
    $file = File::create([
        'name' => 'File 1',
        'url' => '/storage/url 1',
    ]);

    $company1 = Company::create([
        'name' => 'Nombre',
        'image_id' => $file->id,
        'date_end' => Carbon::now()->addYear()->format('Y-m-d'),
        'date_start' => '2000-11-11',
    ]);

    $user1 = User::create([
        'name' => 'User 11',
        'surname' => 'Surname 11',
        'email' => 'user11@incentro.com',
        'Company_id' => $company1->id,
        'image_id' => $file->id,
    ]);

    $user2 = User::create([
        'name' => 'User 12',
        'surname' => 'Surname 12',
        'email' => 'user12@incentro.com',
        'Company_id' => $company1->id,
        'image_id' => $file->id,
    ]);

    $token = UserToken::create([
        'token' => 'a',
        'user_id' => $user1->id,
        'expired_at' => Carbon::now()->addYear()->format('Y-m-d'),
    ]);

    $userPoint1 = UserPoint::create([
        'user_id' => $user1->id,
        'points' => 265564,
    ]);

    $userPoint2 = UserPoint::create([
        'user_id' => $user2->id,
        'points' => 564,
    ]);

    $userwallet1 = UserWalletVirtual::create([
        'user_id' => $user1->id,
        'image_id' => $file->id,
        'virtual_amount' => 12132132,
    ]);

    $userwallet2 = UserWalletVirtual::create([
        'user_id' => $user2->id,
        'virtual_amount' => 32132,
    ]);

    $levels = Level::create([
        'Company_id' => $company1->id,
        'image_id' => $file->id,
        'color'=> 'verde',
        'point_range_min'=>1,
        'point_range_max'=>1000000000000,
    ]);

    $levelsTranslation = LevelTranslation::create([
        'level_id'=>$levels->id,
        'name'=>'verde',
    ]);

    // Ejecución de solicitud HTTP y verificación de respuesta JSON
    $response = $this->withHeaders(['Authorization' => 'Bearer ' . $token->token])
                     ->json('GET', '/api/v1/user/ranking', ['limit' => 1]);

    $response->assertJson([
        'error' => 200,
        'data' => [
            [
                'user' => [
                    [
                        'id' => $user1->id,
                        'name' => $user1->name,
                        'surname' => $user1->surname,
                        'email' => $user1->email,
                        'points' => $userPoint1->points,
                        "position"=> 1,
                        "level" => [
                            'level_name'=> 'verde',
                            'image' => [
                                'id' => $file->id,
                                'name'=>$file->name,
                                'url'=>'/storage/' . $file->url,
                            ],
                        ],
                        'file' => [
                            'image' => [
                                [
                                    'id' => $file->id,
                                    'name' => $file->name,
                                    'url' => '/storage/' . $file->url,
                                ],
                            ],
                        ],
                    ],
                ],
            ],
        ],
        'total_results' => 2,
    ]);

    // Verificación adicional de la base de datos
    $this->assertDatabaseHas('db_user_wallet_virtuals', [
        'id' => $userwallet1->id,
        'user_id' => $userwallet1->user_id,
        'virtual_amount' => $userwallet1->virtual_amount,
    ]);

    $this->assertDatabaseHas('db_user_wallet_virtuals', [
        'id' => $userwallet2->id,
        'user_id' => $userwallet2->user_id,
        'virtual_amount' => $userwallet2->virtual_amount,
    ]);

    $this->assertDatabaseHas('db_user_points', [
        'id' => $userPoint1->id,
        'user_id' => $userPoint1->user_id,
        'points' => $userPoint1->points,
    ]);

    $this->assertDatabaseHas('db_user_points', [
        'id' => $userPoint2->id,
        'user_id' => $userPoint2->user_id,
        'points' => $userPoint2->points,
    ]);
}

```

#### Uso de Middlewares y RefreshDatabase

Para probar rutas con y sin middleware, puedes usar métodos como `withoutMiddleware` y `refreshDatabase`. Ejemplo:

```
public function test_something_without_middleware()
{
    $response = $this->withoutMiddleware()
                     ->get('/api/some-route');

    $response->assertStatus(200);
}

```

#### Uso de Variables Reutilizables

Para evitar la repetición de creación de modelos en múltiples pruebas, puedes usar variables definidas en el método `setUp` de tu clase de prueba:

```
class MyTest extends TestCase
{
    protected $user;

    protected function setUp(): void
    {
        parent::setUp();

        $this->user = User::factory()->create();
    }

    public function test_something_with_user()
    {
        // $this->user está disponible aquí
    }
}

```

#### 8. `assertStatus`

Verifica el código de estado HTTP de la respuesta.

```
public function test_get_users_returns_200()
{
    $response = $this->get('/api/users');

    $response->assertStatus(200);
}

```

#### 9. `assertNotFound`

Asegura que la respuesta indique que el recurso solicitado no se encontró (código de estado 404).

```
public function test_get_non_existing_user_returns_not_found()
{
    $response = $this->get('/api/users/999');

    $response->assertNotFound();
}

```

#### 10. `assertUnauthorized`

Comprueba que la solicitud HTTP devuelva un código de estado 401 (no autorizado).

```
public function test_unauthorized_request()
{
    $response = $this->get('/api/admin/dashboard');

    $response->assertUnauthorized();
}

```

#### 11. `assertForbidden`

Verifica que la solicitud HTTP devuelva un código de estado 403 (prohibido).

```
public function test_forbidden_request()
{
    $response = $this->delete('/api/users/1');

    $response->assertForbidden();
}

```

#### 12. `assertSee`

Comprueba que una cadena específica esté presente en el contenido de la respuesta HTML.

```
public function test_dashboard_shows_user_data()
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
                     ->get('/dashboard');

    $response->assertSee($user->name);
}

```

#### 13. `assertDontSee`

Verifica que una cadena específica no esté presente en el contenido de la respuesta HTML.

```
public function test_dashboard_does_not_show_private_data()
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
                     ->get('/dashboard');

    $response->assertDontSee('Admin Panel');
}

```

#### 14. `assertHeader`

Asegura que un encabezado específico esté presente en la respuesta HTTP.

```
public function test_response_contains_content_type_json()
{
    $response = $this->get('/api/users');

    $response->assertHeader('Content-Type', 'application/json');
}

```

#### 15. `assertViewHas`

Comprueba que una vista específica tenga ciertos datos disponibles.

```
public function test_home_page_has_featured_products()
{
    $response = $this->get('/');

    $response->assertViewHas('products', Product::where('featured', true)->get());
}

```

#### 16. `assertSessionMissing`

Verifica que un elemento específico no esté presente en la sesión.

```
public function test_flash_message_is_removed_after_displayed()
{
    $response = $this->post('/contact', ['message' => 'Hello, support!']);

    $response->assertSessionMissing('status');
}

```

#### 17. `assertCookie`

Asegura que una cookie específica esté presente en la respuesta.

```
public function test_response_contains_cookie()
{
    $response = $this->get('/api/users');

    $response->assertCookie('XSRF-TOKEN');
}

```

### Conclusión

Estas comprobaciones adicionales te permiten validar una amplia gama de condiciones y resultados en tus pruebas de Laravel. Cada una cumple un propósito específico, desde verificar el estado y contenido de la respuesta HTTP hasta asegurar la presencia de datos en la vista o en la sesión. Utilizar estas técnicas te ayudará a mejorar la cobertura y la calidad de tus pruebas, garantizando que tu aplicación se comporte correctamente en diferentes escenarios y condiciones.
 [[TESTS]]  [[Controller]]  [[Models en laravel]]  [[Migrations-tablas]]  [[Middleware]]
