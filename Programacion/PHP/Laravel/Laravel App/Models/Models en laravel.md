### Comando para Crear un Modelo en Laravel

Para crear un modelo en Laravel, utilizas el comando `artisan make:model`. Este comando genera una nueva clase de modelo en la carpeta `app/Models`.

`php artisan make:model Action php artisan make:model User php artisan make:model Company`

---

### Ejemplo de Modelos en Laravel

#### Modelo: `Action`

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Action extends Model
{
    use HasFactory, SoftDeletes;

    /**
     * Table associated
     *
     * @var string
     */
    protected $table = 'db_actions';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'company_id',
        'code',
        'date_start',
        'date_end',
        'points',
        'virtual_amount',
        'challenge',
    ];

    /**
     * Relationship with Company
     */
    public function company()
    {
        return $this->belongsTo(Company::class, 'company_id');
    }
}

```


- **`use HasFactory, SoftDeletes;`**: Utiliza los traits para añadir funcionalidad de fábrica y borrado suave.
- **`protected $table`**: Define el nombre de la tabla asociada con el modelo.
- **`protected $fillable`**: Especifica los atributos que son asignables en masa.
- **Relaciones**:
    - **`belongsTo(Company::class, 'company_id')`**: Define una relación de pertenencia con el modelo `Company`.

---

#### Modelo: `User`

```
<?php

namespace App\Models;

use Illuminate\Auth\Authenticatable;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model implements AuthenticatableContract
{
    use HasFactory, SoftDeletes, Authenticatable;

    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'db_users';

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = ['password'];

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'document_type_id',
        'gender_id',
        'language_id',
        'town_id',
        'state_id',
        'country_id',
        'company_id',
        'worker_type_id',
        'image_id',
        'name',
        'surname',
        'document_number',
        'birthdate',
        'email',
        'email2',
        'faults_login',
        'bloqued_login',
        'bloqued_to',
        'telephone1',
        'telephone2',
        'telephone3',
        'address',
        'postal_code',
        'observations',
        'google_id',
        'profile_completed',
        'social_register',
        'last_login',
        'registration_date',
        'two_factor',
        'verified_email',
    ];

    /**
     * Relationships
     */
    public function userRemeberToken()
    {
        return $this->hasMany('App\Models\UserRememberToken', 'user_id');
    }

    public function documentType()
    {
        return $this->belongsTo(DocumentType::class);
    }

    public function gender()
    {
        return $this->belongsTo(Gender::class);
    }

    public function language()
    {
        return $this->belongsTo(Language::class);
    }

    public function town()
    {
        return $this->belongsTo(Town::class);
    }

    public function state()
    {
        return $this->belongsTo(State::class);
    }

    public function country()
    {
        return $this->belongsTo(Country::class);
    }

    public function company()
    {
        return $this->belongsTo(Company::class);
    }

    public function workerType()
    {
        return $this->belongsTo(WorkerType::class);
    }

    public function image()
    {
        return $this->belongsTo(File::class);
    }
}

```

- **Traits y Contratos**:
    - **`Authenticatable`**: Añade soporte de autenticación.
    - **`AuthenticatableContract`**: Implementa el contrato de autenticación.
    - **`SoftDeletes`**: Añade funcionalidad de borrado suave.
- **`protected $hidden`**: Especifica los atributos que deben estar ocultos en las salidas del modelo (por ejemplo, contraseñas).
- **Relaciones**:
    - Múltiples relaciones `belongsTo` para definir la pertenencia a otros modelos.

Modelo: `Company`

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Company extends Model
{
    use HasFactory, SoftDeletes;

    /**
     * Table associated.
     *
     * @var string
     */
    protected $table = 'db_companies';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'short_description',
        'description',
        'url',
        'email',
        'contact_person',
        'date_start',
        'date_end',
        'image_id',
        'social_login_google',
    ];

    /**
     * Relationships
     */
    public function image()
    {
        return $this->belongsTo(File::class, 'image_id');
    }

    public function users()
    {
        return $this->hasMany(User::class, 'company_id');
    }
}

```

- **Traits**:
    - **`HasFactory`**: Utiliza el trait de fábrica para la generación de instancias del modelo.
    - **`SoftDeletes`**: Añade funcionalidad de borrado suave.
- **`protected $table`**: Define el nombre de la tabla asociada con el modelo.
- **`protected $fillable`**: Especifica los atributos que son asignables en masa.
- **Relaciones**:
    - **`belongsTo(File::class, 'image_id')`**: Define una relación de pertenencia con el modelo `File`.
    - **`hasMany(User::class, 'company_id')`**: Define una relación de uno a muchos con el modelo `User`.

---

### Características Adicionales

- **SoftDeletes**: Permite realizar borrados suaves, es decir, los registros no se eliminan físicamente de la base de datos, sino que se marca con una columna `deleted_at`.
- **HasFactory**: Proporciona métodos de fábrica para la creación de instancias de modelos para pruebas.
- **Relaciones**: Definir relaciones entre modelos es esencial para trabajar con datos relacionados en Laravel. Algunos métodos comunes incluyen:
    - **`belongsTo`**: Define una relación de pertenencia a otro modelo.
    - **`hasMany`**: Define una relación de uno a muchos.
    - **`hasOne`**: Define una relación de uno a uno.
    - **`belongsToMany`**: Define una relación de muchos a muchos.

Estos ejemplos ilustran cómo definir modelos en Laravel y cómo establecer relaciones entre ellos. Puedes expandir estos conceptos para ajustarlos a las necesidades específicas de tu aplicación.

[[Comandos recurrentes php artisan]][[php Artisan]]