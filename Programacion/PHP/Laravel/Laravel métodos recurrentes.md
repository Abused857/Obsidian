### Métodos Recurrentes en Laravel

#### Carbon

**Uso:** Manipulación de fechas y horas en PHP de una manera sencilla y legible.

**Ejemplo:**

```
use Carbon\Carbon;

// Crear una instancia de Carbon con la fecha y hora actual
$currentDateTime = Carbon::now();

// Formatear una fecha
$formattedDate = $currentDateTime->format('Y-m-d H:i:s');

// Obtener el día de la semana
$dayOfWeek = $currentDateTime->dayOfWeek;

```

#### DB y Raw Queries

**Uso:** Interactuar con la base de datos utilizando consultas SQL puras o métodos de fluidez proporcionados por Laravel.

**Ejemplo:**

```
use Illuminate\Support\Facades\DB;

// Consulta utilizando métodos de fluidez
$users = DB::table('users')
            ->where('status', 1)
            ->orderBy('created_at', 'desc')
            ->get();

// Consulta utilizando SQL crudo
$results = DB::select('SELECT * FROM users WHERE id = ?', [1]);

```

#### Eloquent ORM

**Uso:** Mapeo de objetos para interactuar con la base de datos de manera orientada a objetos.

**Ejemplo:**

```
use App\Models\User;

// Obtener todos los usuarios activos
$activeUsers = User::where('status', 'active')->get();

// Crear un nuevo usuario
$newUser = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password'),
]);

// Actualizar un usuario existente
$user = User::find(1);
$user->name = 'Jane Doe';
$user->save();

```
#### Colecciones

**Uso:** Manipulación de conjuntos de datos en PHP de manera fluida y con métodos poderosos.

**Ejemplo:**

```
use Illuminate\Support\Collection;

// Crear una colección
$collection = collect([1, 2, 3, 4, 5]);

// Filtrar elementos
$filtered = $collection->filter(function ($value, $key) {
    return $value > 2;
});

// Transformar elementos
$transformed = $collection->map(function ($item, $key) {
    return $item * 2;
});

// Obtener el primer elemento
$firstItem = $collection->first();

```

#### Helpers de Laravel

**Uso:** Funciones globales que facilitan tareas comunes en Laravel.

**Ejemplo:**

```
use Illuminate\Support\Facades\Route;

// Generar una URL para una ruta nombrada
$url = route('profile', ['id' => 1]);

// Redireccionar a una ruta
return redirect()->route('dashboard');

use Illuminate\Support\Facades\Config;

// Obtener un valor de configuración
$value = config('app.timezone');

use Illuminate\Support\Str;

// Manipulación de cadenas
$slug = Str::slug('Hello World', '-');

```

### Consultas SQL en Raw o con Funciones

#### Consultas SQL Crudas

**Uso:** Ejecutar consultas SQL directamente utilizando el método `DB::select()`.

**Ejemplo:**

```
use Illuminate\Support\Facades\DB;

// Consulta SQL cruda
$results = DB::select('SELECT * FROM users WHERE id = ?', [1]);

```

#### Consultas Utilizando Funciones de Consulta

**Uso:** Utilizar métodos de consulta fluída como `select`, `where`, `orderBy`, etc., para construir consultas de manera legible y mantenible.

**Ejemplo:**

```
use Illuminate\Support\Facades\DB;

// Consulta utilizando métodos de consulta fluída
$users = DB::table('users')
            ->select('name', 'email')
            ->where('status', '=', 'active')
            ->orderBy('name', 'asc')
            ->get();

```

### Filtrar Input del Request por ID y Consulta a través de Función Anónima

#### Filtrar Input del Request por ID

**Uso:** Obtener y filtrar parámetros del request por ID antes de realizar una consulta.

**Ejemplo:**

```
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

public function getUsersById(Request $request)
{
    $userId = $request->input('id');

    $user = DB::table('users')
                ->where('id', $userId)
                ->first();

    return response()->json($user);
}

```

#### Consulta a través de Función Anónima

**Uso:** Utilizar una función anónima dentro de una consulta para realizar filtros avanzados o personalizados.

**Ejemplo:**

```
use Illuminate\Support\Facades\DB;

// Consulta utilizando función anónima
$users = DB::table('users')
            ->where(function ($query) {
                $query->where('status', '=', 'active')
                      ->orWhere('created_at', '>=', now()->subDays(7));
            })
            ->get();

```

### Ejemplos de Eloquent en Laravel

#### Creación y Persistencia

**Uso:** Crear nuevos registros en la base de datos utilizando Eloquent.

**Ejemplo:**

```
use App\Models\User;

// Crear un nuevo usuario
$user = new User();
$user->name = 'John Doe';
$user->email = 'john.doe@example.com';
$user->password = bcrypt('password123');
$user->save();

```

#### Consulta por ID

**Uso:** Obtener un registro específico por su ID utilizando Eloquent.

**Ejemplo:**

```
use App\Models\Post;

// Obtener un post por su ID
$post = Post::find(1);

// Verificar si el post existe
if ($post) {
    // Hacer algo con el post
}

```

#### Consulta con Restricciones

**Uso:** Realizar consultas con condiciones utilizando métodos de Eloquent.

**Ejemplo:**

```
use App\Models\Product;

// Obtener productos que estén activos y cuyo precio sea mayor a 100
$products = Product::where('active', true)
                    ->where('price', '>', 100)
                    ->get();

```

#### Actualización de Registros

**Uso:** Actualizar registros existentes utilizando Eloquent.

**Ejemplo:**

```
use App\Models\Employee;

// Buscar un empleado por ID y actualizar su salario
$employee = Employee::find(1);
if ($employee) {
    $employee->salary = 50000;
    $employee->save();
}

```

#### Eliminación de Registros

**Uso:** Eliminar registros utilizando Eloquent.

**Ejemplo:**

```
use App\Models\Customer;

// Eliminar un cliente por ID
$customer = Customer::find(1);
if ($customer) {
    $customer->delete();
}

```

```
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}

// Obtener todos los comentarios de un post
$post = Post::find(1);
$comments = $post->comments;

```

### Conclusiones

Estos ejemplos muestran cómo utilizar Eloquent para crear, consultar, actualizar y eliminar registros en la base de datos, así como establecer y utilizar relaciones entre modelos en Laravel. Eloquent simplifica enormemente la interacción con la base de datos al proporcionar una interfaz orientada a objetos que sigue el patrón Active Record. Utilizar Eloquent no solo mejora la legibilidad del código, sino que también promueve buenas prácticas de desarrollo al trabajar con datos en aplicaciones Laravel.



[[Laravel instalación]][[PHP]]