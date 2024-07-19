### Comando para Crear un Seeder en Laravel

Para crear un seeder en Laravel, se utiliza el comando `artisan make:seeder`. Este comando genera una nueva clase de seeder en la carpeta `database/seeders`.

`php artisan make:seeder CompanySeeder`

---

### Comando para Ejecutar un Seeder

Para ejecutar un seeder y poblar la base de datos con los datos definidos en el seeder, se utiliza el siguiente comando:

`php artisan db:seed --class=CompanySeeder`

Este comando ejecuta específicamente el seeder `CompanySeeder`. Si deseas ejecutar todos los seeders registrados en `DatabaseSeeder`, puedes simplemente usar:

`php artisan db:seed`

---

### Ejemplo de Seeder: `CompanySeeder`


```
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Company;

class CompanySeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Company::factory()->count(10)->create();
    }
}

```

Este seeder utiliza el factory de `Company` para crear 10 registros en la tabla `companies`.

---

### Información Adicional

#### Registering Seeders in `DatabaseSeeder`

Para que Laravel ejecute automáticamente todos tus seeders, debes registrarlos en la clase `DatabaseSeeder` ubicada en `database/seeders/DatabaseSeeder.php`.

```
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            CompanySeeder::class,
            // Añade otros seeders aquí
        ]);
    }
}

```

#### Revertir y Vuelva a Ejecutar Seeders

A veces, puede ser necesario revertir y volver a ejecutar los seeders. Para esto, puedes usar los comandos de migración junto con los seeders:

1. **Revertir Migraciones**:
    
    `php artisan migrate:refresh`
    
    Esto revertirá todas las migraciones y luego volverá a ejecutarlas, lo que borrará y volverá a poblar la base de datos con los seeders registrados en `DatabaseSeeder`.
    
2. **Revertir y Ejecutar Seeders**:
    
    `php artisan migrate:refresh --seed`
    
    Esto hace lo mismo que el comando anterior, pero también ejecuta los seeders automáticamente después de las migraciones.
    

#### Seeders en Ambientes de Pruebas

Es común utilizar seeders para preparar datos de prueba en ambientes de desarrollo y pruebas. Puedes configurar el archivo `.env` de tu aplicación para especificar qué seeders deben ejecutarse automáticamente en estos ambientes.

`APP_ENV=testing`

Luego, en el archivo de configuración de `config/database.php`, puedes añadir lógica condicional para ejecutar seeders específicos en diferentes ambientes.

#### Uso de Faker en Seeders

Puedes utilizar la biblioteca Faker para generar datos de prueba realistas en tus seeders. Faker es extremadamente útil para poblar la base de datos con datos variados y realistas.

```
use Faker\Factory as Faker;

class CompanySeeder extends Seeder
{
    public function run()
    {
        $faker = Faker::create();

        foreach (range(1, 10) as $index) {
            Company::create([
                'name' => $faker->company,
                'email' => $faker->companyEmail,
                'contact_person' => $faker->name,
                'url' => $faker->url,
                // Añade otros campos aquí
            ]);
        }
    }
}

```

Este enfoque te permite tener un control más granular sobre los datos que se insertan, y puedes utilizar cualquier método de Faker para generar los valores necesarios.

---

### Conclusión

Los seeders en Laravel son una herramienta poderosa para poblar tu base de datos con datos iniciales o de prueba. Al combinarlos con factories y Faker, puedes generar datos realistas y variados de manera rápida y eficiente. Recuerda registrar tus seeders en `DatabaseSeeder` para una ejecución ordenada y utilizar los comandos de artisan para gestionar el proceso de seeders.
 [[Migrations-tablas]]  [[Laravel instalación]]  [[Factory]] [[Models en laravel]]