### Comando para Crear una Migración en Laravel

Para crear una migración en Laravel, utiliza el siguiente comando:

`php artisan make:migration create_table_name`

### Ejemplo de Migración para Crear la Tabla `db_languages`

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLpLanguagesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('db_languages', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name')->nullable();
            $table->string('abbreviation')->nullable();
            $table->string('flag')->nullable();
            $table->integer('order')->nullable();
            $table->timestamp('created_at')->default(\DB::raw('CURRENT_TIMESTAMP'));
            $table->timestamp('updated_at')->default(\DB::raw('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP'));
            $table->softDeletes();
        });

        DB::table('db_languages')->insert([
            'id' => 1,
            'name' => 'spanish',
            'abbreviation' => 'ES',
            'flag' => 'spanish.gif',
            'order' => 100,
        ]);

        DB::table('db_languages')->insert([
            'id' => 2,
            'name' => 'english',
            'abbreviation' => 'EN',
            'flag' => 'english.gif',
            'order' => 99,
        ]);
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('db_languages');
    }
}

```

### Ejemplo de Migración para Crear una FK Recursiva

#### Migración para Crear la Tabla `db_files`

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLpFilesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('db_files', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name')->nullable();
            $table->decimal('size', 20, 5)->nullable();
            $table->string('mimetype')->nullable();
            $table->text('url')->nullable();

            $table->unsignedBigInteger('type_id')->nullable();
            $table->foreign('type_id')->references('id')->on('db_file_types');
            
            $table->boolean('private')->default(0);
            $table->boolean('gallery')->default(1);
            $table->dateTime('date')->nullable();
            $table->integer('order')->nullable();
            $table->timestamp('created_at')->default(\DB::raw('CURRENT_TIMESTAMP'));
            $table->timestamp('updated_at')->default(\DB::raw('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP'));
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('db_files');
    }
}

```

Migración para Crear la Tabla `db_companies` con FK Recursiva

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLpCompaniesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('db_companies', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name')->nullable();
            $table->text('short_description')->nullable();
            $table->longText('description')->nullable();
            $table->string('url')->nullable();
            $table->string('email')->nullable();
            $table->string('contact_person')->nullable();
            $table->date('date_start')->nullable();
            $table->date('date_end')->nullable();

            $table->unsignedBigInteger('image_id')->nullable();
            $table->foreign('image_id')->references('id')->on('db_files');

            $table->boolean('social_login_google')->default(0);
            $table->timestamp('created_at')->default(\DB::raw('CURRENT_TIMESTAMP'));
            $table->timestamp('updated_at')->default(\DB::raw('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP'));
            $table->softDeletes();
        });

        // Insert company_id into db_files to avoid migration problems
        Schema::table('db_files', function (Blueprint $table) {
            $table->unsignedBigInteger('company_id')->nullable();
            $table->foreign('company_id')->references('id')->on('db_companies');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('db_files', function (Blueprint $table) {
            $table->dropForeign(['company_id']);
            $table->dropColumn('company_id');
        });

        Schema::dropIfExists('db_companies');
    }
}
´´´

```

### Opciones de Columnas en Migraciones de Laravel

A continuación, se presentan varias opciones de columnas que puedes usar en tus migraciones, con ejemplos de cada una:

- **`bigIncrements`**: Incremento automático de tipo bigint
    
    `$table->bigIncrements('id');`
    
- **`increments`**: Incremento automático de tipo int
    
    `$table->increments('id');`
    
- **`string`**: Cadena de caracteres
    
    `$table->string('name')->nullable();`
    
- **`text`**: Texto largo
    
    `$table->text('description')->nullable();`
    
- **`longText`**: Texto muy largo
    
    `$table->longText('content')->nullable();`
    
- **`integer`**: Entero
    
    `$table->integer('order')->nullable();`
    
- **`unsignedBigInteger`**: Entero sin signo de tipo bigint
    
    `$table->unsignedBigInteger('user_id')->nullable();`
    
- **`boolean`**: Valor booleano
    
    `$table->boolean('active')->default(0);`
    
- **`dateTime`**: Fecha y hora
    
    `$table->dateTime('published_at')->nullable();`
    
- **`date`**: Fecha
    
    
    `$table->date('birthdate')->nullable();`
    
- **`timestamp`**: Marca de tiempo
    
    `$table->timestamp('created_at')->default(\DB::raw('CURRENT_TIMESTAMP'));`
    
- **`softDeletes`**: Eliminación lógica
    
    `$table->softDeletes();`
    
- **`decimal`**: Número decimal
    
    `$table->decimal('price', 8, 2)->nullable();`
    
- **`foreign`**: Llave foránea
    
    `$table->unsignedBigInteger('role_id'); $table->foreign('role_id')->references('id')->on('roles');`

**`timestamp`**: Marca de tiempo con actualización automática

`$table->timestamp('updated_at')->default(\DB::raw('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMES`

- **`enum`**: Tipo enumerado
    
    `$table->enum('type', ['main', 'gallery'])->nullable();`
    

### Opciones de Columnas en Migraciones de Laravel (Continuación)

A continuación, se incluyen más ejemplos de opciones de columnas que puedes usar en tus migraciones de Laravel:

- **`char`**: Cadena de caracteres fija
    
    `$table->char('code', 4)->nullable();`
    
- **`json`**: Campo JSON
    
    `$table->json('options')->nullable();`
    
- **`jsonb`**: Campo JSONB (mejor rendimiento en PostgreSQL)
    
    `$table->jsonb('attributes')->nullable();`
    
- **`binary`**: Datos binarios
    
    `$table->binary('data')->nullable();`
    
- **`uuid`**: UUID (identificador único universal)
    
    `$table->uuid('uuid')->nullable();`
    
- **`ipAddress`**: Dirección IP
    
    `$table->ipAddress('visitor')->nullable();`
    
- **`macAddress`**: Dirección MAC
    
    `$table->macAddress('device')->nullable();`
    
- **`geometry`**: Datos geométricos
    
    `$table->geometry('location')->nullable();`
    
- **`point`**: Punto geoespacial
    
    `$table->point('position')->nullable();`
    
- **`lineString`**: Línea geoespacial
    
    `$table->lineString('path')->nullable();`
    
- **`polygon`**: Polígono geoespacial
    
    `$table->polygon('area')->nullable();`
    
- **`multiPoint`**: Múltiples puntos geoespaciales
    
    `$table->multiPoint('positions')->nullable();`
    
- **`multiLineString`**: Múltiples líneas geoespaciales
    
    `$table->multiLineString('paths')->nullable();`
    
- **`multiPolygon`**: Múltiples polígonos geoespaciales
    
    `$table->multiPolygon('areas')->nullable();`
    
- **`geometryCollection`**: Colección de geometrías
    
    `$table->geometryCollection('geometries')->nullable();`

- **`enum`**: Tipo enumerado
    
    `$table->enum('type', ['main', 'gallery'])->nullable();`
    

### Opciones de Columnas en Migraciones de Laravel (Continuación)

A continuación, se incluyen más ejemplos de opciones de columnas que puedes usar en tus migraciones de Laravel:

- **`char`**: Cadena de caracteres fija
    `$table->char('code', 4)->nullable();`
    
- **`json`**: Campo JSON
    
    `$table->json('options')->nullable();`
    
- **`jsonb`**: Campo JSONB (mejor rendimiento en PostgreSQL)
    
    `$table->jsonb('attributes')->nullable();`
    
- **`binary`**: Datos binarios
    
    `$table->binary('data')->nullable();`
    
- **`uuid`**: UUID (identificador único universal)
    
    `$table->uuid('uuid')->nullable();`
    
- **`ipAddress`**: Dirección IP
    
    `$table->ipAddress('visitor')->nullable();`
    
- **`macAddress`**: Dirección MAC
    
    `$table->macAddress('device')->nullable();`
    
- **`geometry`**: Datos geométricos
    
    `$table->geometry('location')->nullable();`
    
- **`point`**: Punto geoespacial
    
    `$table->point('position')->nullable();`
    
- **`lineString`**: Línea geoespacial
    
    `$table->lineString('path')->nullable();`
    
- **`polygon`**: Polígono geoespacial
    
    `$table->polygon('area')->nullable();`
    
- **`multiPoint`**: Múltiples puntos geoespaciales
    
    `$table->multiPoint('positions')->nullable();`
    
- **`multiLineString`**: Múltiples líneas geoespaciales
    
    `$table->multiLineString('paths')->nullable();`
    
- **`multiPolygon`**: Múltiples polígonos geoespaciales
    
    `$table->multiPolygon('areas')->nullable();`
    
- **`geometryCollection`**: Colección de geometrías
    
    `$table->geometryCollection('geometries')->nullable();`

  [[Comandos recurrentes php artisan]]  [[Laravel instalación]] 