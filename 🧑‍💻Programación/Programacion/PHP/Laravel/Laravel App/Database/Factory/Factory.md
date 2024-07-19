### Comando para Crear un Factory en Laravel

Para crear un factory en Laravel, se utiliza el comando `artisan make:factory`. Este comando genera una nueva clase de factory en la carpeta `database/factories`.

`php artisan make:factory UserFactory --model=User`

### Ejemplo de Factory en Laravel

#### Factory: `UserFactory`

```
<?php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    /**
     * The name of the factory's corresponding model.
     *
     * @var string
     */
    protected $model = User::class;

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'document_type_id' => function () {
                return \App\Models\DocumentType::factory()->create()->id;
            },
            'gender_id' => function () {
                return \App\Models\Gender::factory()->create()->id;
            },
            'language_id' => function () {
                return \App\Models\Language::factory()->create()->id;
            },
            'town_id' => function () {
                return \App\Models\Town::factory()->create()->id;
            },
            'state_id' => function () {
                return \App\Models\State::factory()->create()->id;
            },
            'country_id' => function () {
                return \App\Models\Country::factory()->create()->id;
            },
            'company_id' => function () {
                return \App\Models\Company::factory()->create()->id;
            },
            'worker_type_id' => function () {
                return \App\Models\WorkerType::factory()->create()->id;
            },
            'image_id' => function () {
                $file = \App\Models\File::factory()->create();
                return $file->id;
            },
            'name' => $this->faker->firstName,
            'surname' => $this->faker->lastName,
            'document_number' => $this->faker->unique()->randomNumber(),
            'birthdate' => $this->faker->date(),
            'email' => $this->faker->unique()->safeEmail,
            'email2' => $this->faker->unique()->safeEmail,
            'password' => bcrypt('password'),
            'faults_login' => 0,
            'bloqued_login' => false,
            'bloqued_to' => null,
            'telephone1' => $this->faker->phoneNumber,
            'telephone2' => $this->faker->phoneNumber,
            'telephone3' => $this->faker->phoneNumber,
            'address' => $this->faker->address,
            'postal_code' => $this->faker->postcode,
            'observations' => $this->faker->sentence,
            'google_id' => $this->faker->uuid,
            'profile_completed' => false,
            'social_register' => false,
            'last_login' => null,
            'registration_date' => null,
            'two_factor' => false,
            'verified_email' => false,
        ];
    }
}

```

- **Relaciones**: Utiliza funciones para crear y asignar IDs de otros modelos.
- **Atributos Faker**: Usa métodos de Faker para generar datos de prueba.

---

### Atributos Faker Comunes

A continuación, se muestra una lista de algunos métodos de Faker que puedes usar en tus factories para generar datos de prueba:

- **`$this->faker->name`**: Genera un nombre completo.
- **`$this->faker->firstName`**: Genera un primer nombre.
- **`$this->faker->lastName`**: Genera un apellido.
- **`$this->faker->email`**: Genera un email.
- **`$this->faker->safeEmail`**: Genera un email seguro.
- **`$this->faker->unique()->safeEmail`**: Genera un email seguro único.
- **`$this->faker->randomNumber`**: Genera un número aleatorio.
- **`$this->faker->randomFloat($nbMaxDecimals, $min, $max)`**: Genera un número flotante aleatorio.
- **`$this->faker->date`**: Genera una fecha.
- **`$this->faker->dateTime`**: Genera una fecha y hora.
- **`$this->faker->dateTimeBetween($startDate, $endDate)`**: Genera una fecha y hora entre dos fechas.
- **`$this->faker->phoneNumber`**: Genera un número de teléfono.
- **`$this->faker->address`**: Genera una dirección.
- **`$this->faker->city`**: Genera una ciudad.
- **`$this->faker->state`**: Genera un estado.
- **`$this->faker->country`**: Genera un país.
- **`$this->faker->postcode`**: Genera un código postal.
- **`$this->faker->sentence`**: Genera una oración.
- **`$this->faker->paragraph`**: Genera un párrafo.
- **`$this->faker->text($maxNbChars)`**: Genera un texto con un máximo de caracteres.
- **`$this->faker->uuid`**: Genera un UUID.
- **`$this->faker->boolean`**: Genera un valor booleano.
- **`$this->faker->imageUrl($width, $height)`**: Genera una URL de imagen.
- **`$this->faker->company`**: Genera un nombre de compañía.

---

### Ejemplo de Creación de Objetos en Tests o Seeders

Para crear objetos utilizando factories en tus tests o seeders, puedes utilizar los métodos `create` y `make`.

#### En un Test

```
public function testUserCreation()
{
    $user = \App\Models\User::factory()->create();

    $this->assertDatabaseHas('db_users', [
        'id' => $user->id,
    ]);
}

```

#### En un Seeder

```
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        User::factory()->count(50)->create();
    }
}

```


- **`create`**: Persiste el modelo en la base de datos.
- **`make`**: Crea una instancia del modelo pero no la persiste.

---

### Conclusión

Los factories en Laravel son una herramienta poderosa para generar datos de prueba de manera sencilla y eficiente. Puedes definir el estado por defecto de tus modelos y utilizar Faker para generar datos variados y realistas. Además, la creación de relaciones entre modelos en las factories facilita la creación de datos complejos para pruebas y seeders.

 [[Models en laravel]] [[Laravel instalación]] [[TESTS]]