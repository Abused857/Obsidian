## Creando un Validator

Para crear una instancia de `Validator`, puedes usar el método `make` proporcionado por la facada `Validator`:

```
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, $rules, $messages);

```


- `$data`: Array de datos a validar.
- `$rules`: Array de reglas de validación.
- `$messages`: (Opcional) Array de mensajes de error personalizados.

## Validando datos

Una vez creado el `Validator`, puedes verificar si los datos son válidos usando el método `fails` o `passes`:

```
if ($validator->fails()) {
    // La validación ha fallado
    $errors = $validator->errors();
} else {
    // La validación ha pasado
}

```

## Reglas de Validación

Aquí tienes una lista de todas las reglas de validación disponibles en Laravel, junto con ejemplos:

### Reglas Básicas

- **accepted**: El campo debe ser aceptado (checkbox o true).
- **active_url**: El campo debe ser una URL válida y activa.
- **after
    
    **: El campo debe ser una fecha posterior a la fecha dada.
- **alpha**: El campo solo debe contener letras.
- **alpha_dash**: El campo solo debe contener letras, números, guiones y guiones bajos.
- **alpha_num**: El campo solo debe contener letras y números.
- **array**: El campo debe ser un array.
- **before
    
    **: El campo debe ser una fecha anterior a la fecha dada.

### Reglas Numéricas

- **between
    
    ,max**: El campo debe tener un valor entre min y max.
- **boolean**: El campo debe ser verdadero o falso.
- **confirmed**: El campo debe tener un campo de confirmación con el mismo valor (e.g., `password` y `password_confirmation`).
- **date**: El campo debe ser una fecha válida.
- **date_equals
    
    **: El campo debe ser una fecha igual a la fecha dada.
- **digits
    
    **: El campo debe ser un número de `value` dígitos.
- **digits_between
    
    ,max**: El campo debe tener un número de dígitos entre `min` y `max`.

### Reglas de Tamaño

- **dimensions**: El archivo subido debe cumplir con las dimensiones especificadas.
- **distinct**: El campo no debe tener valores duplicados.
- **email**: El campo debe ser una dirección de correo electrónico válida.
- **exists
    
    ,column**: El campo debe existir en la base de datos.
- **file**: El campo debe ser un archivo subido.
- **filled**: El campo debe tener un valor.

### Reglas de Formato

- **gt
    
    **: El campo debe ser mayor que el campo especificado.
- **gte
    
    **: El campo debe ser mayor o igual que el campo especificado.
- **image**: El campo debe ser una imagen (jpeg, png, bmp, gif, svg, o webp).
- **in
    
    ,bar,...**: El campo debe estar en la lista de valores especificados.
- **integer**: El campo debe ser un número entero.
- **ip**: El campo debe ser una dirección IP válida.
- **ipv4**: El campo debe ser una dirección IPv4 válida.
- **ipv6**: El campo debe ser una dirección IPv6 válida.

### Reglas Avanzadas

- **json**: El campo debe ser una cadena JSON válida.
- **lt
    
    **: El campo debe ser menor que el campo especificado.
- **lte
    
    **: El campo debe ser menor o igual que el campo especificado.
- **max
    
    **: El campo no debe ser mayor que `value`.
- **mimes
    
    ,bar,...**: El campo debe ser un archivo del tipo MIME especificado.
- **mimetypes
    
    ,bar,...**: El campo debe ser un archivo de uno de los tipos MIME especificados.
- **min
    
    **: El campo no debe ser menor que `value`.
- **not_in
    
    ,bar,...**: El campo no debe estar en la lista de valores especificados.
- **not_regex**: El campo no debe coincidir con el patrón regex especificado.
- **numeric**: El campo debe ser un número.

### Otras Reglas

- **present**: El campo debe estar presente en la entrada.
- **regex**: El campo debe coincidir con el patrón regex especificado.
- **required**: El campo es obligatorio.
- **required_if
    
    ,value**: El campo es obligatorio si el `field` es igual a `value`.
- **required_unless
    
    ,value**: El campo es obligatorio a menos que el `field` sea igual a `value`.
- **required_with
    
    ,bar,...**: El campo es obligatorio si alguno de los `foo, bar,...` está presente.
- **required_with_all
    
    ,bar,...**: El campo es obligatorio si todos los `foo, bar,...` están presentes.
- **required_without
    
    ,bar,...**: El campo es obligatorio si alguno de los `foo, bar,...` no está presente.
- **required_without_all
    
    ,bar,...**: El campo es obligatorio si ninguno de los `foo, bar,...` está presente.
- **same
    
    **: El campo debe coincidir con el campo especificado.
- **size
    
    **: El campo debe tener el tamaño especificado.
- **starts_with
    
    ,bar,...**: El campo debe comenzar con uno de los valores especificados.
- **string**: El campo debe ser una cadena.
- **timezone**: El campo debe ser una zona horaria válida.
- **unique
    
    ,column,except,idColumn**: El campo debe ser único en la tabla de la base de datos.
- **url**: El campo debe ser una URL válida.
- **uuid**: El campo debe ser un UUID válido.

## Mensajes de error personalizados

Puedes personalizar los mensajes de error pasando un array como tercer argumento al método `make`:

```
$messages = [
    'required' => 'El campo :attribute es obligatorio.',
    'email' => 'El campo :attribute debe ser una dirección de correo válida.',
    // más mensajes personalizados
];

$validator = Validator::make($data, $rules, $messages);

```
## Validación condicional

Puedes agregar reglas de validación condicionalmente usando el método `sometimes`:

```
$validator = Validator::make($data, $rules);

$validator->sometimes('email', 'required|email', function ($input) {
    return $input->user_type == 'admin';
});

```

## Validación anidada

Para validar arrays anidados, puedes usar la sintaxis de puntos:

```
$rules = [
    'user.name' => 'required|string',
    'user.email' => 'required|email',
];

$validator = Validator::make($data, $rules);

```

## Ejemplos

### Ejemplo 1: Validación simple

```
$data = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => 'secret123',
    'password_confirmation' => 'secret123',
];

$rules = [
    'name' => 'required|string|max:255',
    'email' => 'required|email|unique:users,email',
    'password' => 'required|string|min:8|confirmed',
];

$validator = Validator::make($data, $rules);

if ($validator->fails()) {
    return redirect()->back()->withErrors($validator)->withInput();
}

```

### Ejemplo 2: Validación con mensajes personalizados

```
$data = [
    'name' => '',
    'email' => 'not-an-email',
];

$rules = [
    'name' => 'required',
    'email' => 'required|email',
];

$messages = [
    'name.required' => 'Por favor, ingrese su nombre.',
    'email.required' => 'El correo electrónico es obligatorio.',
    'email.email' => 'Por favor, ingrese una dirección de correo válida.',
];

$validator = Validator::make($data, $rules, $messages);

if ($validator->fails()) {
    return redirect()->back()->withErrors($validator)->withInput();
}

```
### Ejemplo 3: Validación condicional

```
$data = [
    'user_type' => 'admin',
    'email' => '',
];

$rules = [
    'user_type' => 'required',
];

$validator = Validator::make($data, $rules);

$validator->sometimes('email', 'required|email', function ($input) {
    return $input->user_type == 'admin';
});

if ($validator->fails()) {
    return redirect()->back()->withErrors($validator)->withInput();
}

```


## Ejemplos de reglas de validación para 'page', 'limit' y 'challenge'

### Reglas de validación para 'page'

1. **'integer'**: El campo debe ser un número entero.
    
    
    `'page' => 'integer'`
    
2. **'min:0'**: El campo debe ser un número entero mayor o igual a 0.
    
    `'page' => 'integer|min:0'`
    
3. **'max**
    
    **'**: El campo debe ser un número entero menor o igual a `value`.
    
    `'page' => 'integer|max:100'`
    
4. **'between**
    
    **,max'**: El campo debe ser un número entero entre `min` y `max`.
    

    
    `'page' => 'integer|between:0,100'`
    
5. **'required'**: El campo es obligatorio.
    `'page' => 'required|integer|min:0'`
    
6. **'nullable'**: El campo puede ser nulo.
    
    `'page' => 'nullable|integer|min:0'`
    
7. **'in**
    
    **'**: El campo debe estar en la lista de valores permitidos.
    
    `'page' => 'integer|in:0,1,2,3,4,5'`
    

### Reglas de validación para 'limit'

1. **'integer'**: El campo debe ser un número entero.
    
    
    `'limit' => 'integer'`
    
2. **'min:0'**: El campo debe ser un número entero mayor o igual a 0.
    
    
    `'limit' => 'integer|min:0'`
    
3. **'max**
    
    **'**: El campo debe ser un número entero menor o igual a `value`.
    
    `'limit' => 'integer|max:100'`
    
4. **'between**
    
    **,max'**: El campo debe ser un número entero entre `min` y `max`.
    
    `'limit' => 'integer|between:1,50'`
    
5. **'required'**: El campo es obligatorio.
    
    
    `'limit' => 'required|integer|min:0'`
    
6. **'nullable'**: El campo puede ser nulo.
    
    php
    
    Copiar código
    
    `'limit' => 'nullable|integer|min:0'`
    
7. **'in**
    
    **'**: El campo debe estar en la lista de valores permitidos.
    
    `'limit' => 'integer|in:10,20,30,40,50'`
    

### Reglas de validación para 'challenge'

1. **'boolean'**: El campo debe ser un valor booleano (true o false).

    
    `'challenge' => 'boolean'`
    
2. **'required'**: El campo es obligatorio.
    
    `'challenge' => 'required|boolean'`
    
3. **'nullable'**: El campo puede ser nulo.
    
    `'challenge' => 'nullable|boolean'`
    
4. **'accepted'**: El campo debe ser aceptado (true o yes).
    
    `'challenge' => 'accepted'`
    

### Combinaciones de reglas

1. **'page'**: Campo obligatorio, entero mayor o igual a 0.
    
    `'page' => 'required|integer|min:0'`
    
2. **'limit'**: Campo entero entre 1 y 50.
    
    `'limit' => 'integer|between:1,50'`
    
3. **'challenge'**: Campo obligatorio y booleano.
    `'challenge' => 'required|boolean'`
    
4. **'page' y 'limit'**: Ambos campos son enteros opcionales, con `limit` entre 1 y 100.

    
    `'page' => 'nullable|integer|min:0', 'limit' => 'nullable|integer|between:1,100'`
    
5. **'page' y 'challenge'**: `page` es un entero obligatorio y `challenge` es un booleano opcional.
    
    `'page' => 'required|integer|min:0', 'challenge' => 'nullable|boolean'`
    

### Ejemplo Completo

```
$validator = Validator::make($request->all(), [
    'page' => 'required|integer|min:0',
    'limit' => 'integer|between:1,100',
    'challenge' => 'nullable|boolean'
]);

if ($validator->fails()) {
    $error = 1;
    $message_validator = $message_validator->merge($validator->errors());
}

```

### Validaciones para otros campos

#### Campo 'email'

1. **'email'**: El campo debe ser una dirección de correo electrónico válida.
    
    `'email' => 'email'`
    
2. **'required|email|unique**
    
    **,email'**: El campo es obligatorio, debe ser una dirección de correo electrónico válida y debe ser único en la tabla `users`.
    
    `'email' => 'required|email|unique:users,email'`
    
3. **'email|ends_with:@example.com'**: El campo debe ser una dirección de correo electrónico que termine con `@example.com`.
    
    `'email' => 'email|ends_with:@example.com'`
    

#### Campo 'password'

1. **'required|min:8'**: El campo es obligatorio y debe tener al menos 8 caracteres.
    
    `'password' => 'required|min:8'`
    
2. **'confirmed'**: El campo debe tener un campo de confirmación con el mismo valor (e.g., `password` y `password_confirmation`).
    
    `'password' => 'required|min:8|confirmed'`
    
3. **'regex:/^(?=._[a-z])(?=._[A-Z])(?=.*\d).+$/'**: El campo debe tener al menos una letra mayúscula, una letra minúscula y un dígito.

    
    `'password' => 'required|min:8|regex:/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/'`
    

#### Campo 'name'

1. **'required|string'**: El campo es obligatorio y debe ser una cadena de texto.
    
    `'name' => 'required|string'`
    
2. **'max:255'**: El campo no debe tener más de 255 caracteres.
    
    `'name' => 'required|string|max:255'`
    
3. **'alpha'**: El campo solo debe contener letras.
    
    `'name' => 'required|alpha'`
    

#### Campo 'age'

1. **'required|integer|min:18|max:99'**: El campo es obligatorio, debe ser un número entero y debe estar entre 18 y 99.
    
    `'age' => 'required|integer|min:18|max:99'`
    
2. **'numeric|digits_between:1,3'**: El campo debe ser un número con entre 1 y 3 dígitos.
    
    `'age' => 'numeric|digits_between:1,3'`
    

#### Campo 'phone'

1. **'required|regex:/^(+?1-?)?(\d{3})?[ -.]?(\d{3})[ -.]?(\d{4})$/'**: El campo es obligatorio y debe seguir el formato de número de teléfono de EE.UU.
    
    `'phone' => 'required|regex:/^(\+?1-?)?(\d{3})?[ -.]?(\d{3})[ -.]?(\d{4})$/'`
    
2. **'nullable|numeric'**: El campo puede ser nulo o debe ser un número.
    
    `'phone' => 'nullable|numeric'`
    

#### Campo 'address'

1. **'required|string|max:255'**: El campo es obligatorio, debe ser una cadena de texto y no debe tener más de 255 caracteres.
    
    `'address' => 'required|string|max:255'`
    
2. **'nullable|string'**: El campo puede ser nulo o debe ser una cadena de texto.
    
    `'address' => 'nullable|string'`
    

#### Campo 'date_of_birth'

1. **'required|date'**: El campo es obligatorio y debe ser una fecha válida.
    
    `'date_of_birth' => 'required|date'`
    
2. **'before**
    
    **'**: El campo debe ser una fecha anterior a hoy.
    `'date_of_birth' => 'required|date|before:today'`
    
3. **'after:2000-01-01'**: El campo debe ser una fecha posterior al 1 de enero de 2000.
    
    `'date_of_birth' => 'required|date|after:2000-01-01'`
    

#### Campo 'website'

1. **'nullable|url'**: El campo puede ser nulo o debe ser una URL válida.
    
    `'website' => 'nullable|url'`
    
2. **'active_url'**: El campo debe ser una URL válida y activa.
    
    `'website' => 'active_url'`
    

#### Campo 'photo'

1. **'required|image'**: El campo es obligatorio y debe ser una imagen (jpeg, png, bmp, gif, svg, o webp).
    
    `'photo' => 'required|image'`
    
2. **'mimes**
    
    **,png'**: El campo debe ser una imagen de tipo jpeg o png.
    
    `'photo' => 'required|image|mimes:jpeg,png'`
    
3. **'dimensions**
    
    **=100,min_height=200'**: El campo debe ser una imagen con dimensiones mínimas de 100x200 píxeles.
    
    `'photo' => 'required|image|dimensions:min_width=100,min_height=200'`
    

### Ejemplo Completo

```
$validator = Validator::make($request->all(), [
    'email' => 'required|email|unique:users,email',
    'password' => 'required|min:8|confirmed',
    'name' => 'required|string|max:255',
    'age' => 'required|integer|min:18|max:99',
    'phone' => 'required|regex:/^(\+?1-?)?(\d{3})?[ -.]?(\d{3})[ -.]?(\d{4})$/',
    'address' => 'nullable|string|max:255',
    'date_of_birth' => 'required|date|before:today',
    'website' => 'nullable|url',
    'photo' => 'required|image|mimes:jpeg,png|dimensions:min_width=100,min_height=200'
]);

if ($validator->fails()) {
    $error = 1;
    $message_validator = $message_validator->merge($validator->errors());
}

```

### Paso 1: Validar un ID único o un array de IDs

Primero, debes definir la regla de validación que permita recibir un ID único o un array de IDs. Puedes usar una regla personalizada para esto.

#### Validación

```
$request->validate([
    'ids' => 'required|array',
    'ids.*' => 'integer|exists:db_rewards,id', // Asegura que cada ID en el array exista en la tabla db_rewards
]);

```

#### Ejemplo en el Controlador

Aquí hay un ejemplo de cómo usar el validador en tu controlador:

```
public function updateRewards(Request $request)
{
    // Validar que 'ids' es un array de enteros y que cada ID existe en la tabla db_rewards
    $request->validate([
        'ids' => 'required|array',
        'ids.*' => 'integer|exists:db_rewards,id',
        'status' => 'required|integer|in:1,2,3', // Asegura que el status sea uno de los valores permitidos
    ]);

    // Recuperar los IDs y el nuevo estado desde la solicitud
    $ids = $request->input('ids');
    $status = $request->input('status');

    // Actualizar las recompensas con los IDs proporcionados
    DB::table('db_rewards')
        ->whereIn('id', $ids)
        ->update(['status_id' => $status]);

    return response()->json(['message' => 'Rewards updated successfully.']);
}

```

### Explicación

1. **Validación del Array de IDs:**
    
    - La regla `'ids' => 'required|array'` asegura que el campo `ids` es obligatorio y debe ser un array.
    - La regla `'ids.*' => 'integer|exists:db_rewards,id'` valida que cada elemento en el array `ids` es un entero y que existe en la tabla `db_rewards`.
2. **Validación del Estado:**
    
    - La regla `'status' => 'required|integer|in:1,2,3'` asegura que el campo `status` es obligatorio, es un entero y está en el conjunto de valores permitidos (1, 2, 3).
3. **Actualización de las Recompensas:**
    
    - `whereIn('id', $ids)`: Filtra las recompensas cuyos IDs están en el array `ids`.
    - `update(['status_id' => $status])`: Actualiza el campo `status_id` de las recompensas filtradas con el nuevo estado proporcionado.

### Nota de Obsidian

Puedes documentar este proceso en Obsidian con una nota estructurada. Aquí hay un ejemplo de cómo podrías hacerlo:

```
# Validación y Actualización de Recompensas en Laravel

## Objetivo
Permitir que el validador de Laravel reciba un ID único o un array de IDs y actualizar el estado de las recompensas correspondientes.

## Pasos

### 1. Validar un ID único o un array de IDs

#### Validación

```php
$request->validate([
    'ids' => 'required|array',
    'ids.*' => 'integer|exists:db_rewards,id', // Asegura que cada ID en el array exista en la tabla db_rewards
]);

```

### 2. Ejemplo en el Controlador

```
public function updateRewards(Request $request)
{
    // Validar que 'ids' es un array de enteros y que cada ID existe en la tabla db_rewards
    $request->validate([
        'ids' => 'required|array',
        'ids.*' => 'integer|exists:db_rewards,id',
        'status' => 'required|integer|in:1,2,3', // Asegura que el status sea uno de los valores permitidos
    ]);

    // Recuperar los IDs y el nuevo estado desde la solicitud
    $ids = $request->input('ids');
    $status = $request->input('status');

    // Actualizar las recompensas con los IDs proporcionados
    DB::table('db_rewards')
        ->whereIn('id', $ids)
        ->update(['status_id' => $status]);

    return response()->json(['message' => 'Rewards updated successfully.']);
}

```

### Explicación

1. **Validación del Array de IDs:**
    
    - La regla `'ids' => 'required|array'` asegura que el campo `ids` es obligatorio y debe ser un array.
    - La regla `'ids.*' => 'integer|exists:db_rewards,id'` valida que cada elemento en el array `ids` es un entero y que existe en la tabla `db_rewards`.
2. **Validación del Estado:**
    
    - La regla `'status' => 'required|integer|in:1,2,3'` asegura que el campo `status` es obligatorio, es un entero y está en el conjunto de valores permitidos (1, 2, 3).
3. **Actualización de las Recompensas:**
    
    - `whereIn('id', $ids)`: Filtra las recompensas cuyos IDs están en el array `ids`.
    - `update(['status_id' => $status])`: Actualiza el campo `status_id` de las recompensas filtradas con el nuevo estado proporcionado.

### EJEMPLO FILTRAR POR IDS

```
DB::table('db_rewards')

                ->whereIn('id', $ids)

                ->update([

                    'status_id' => 2, // Cambia el estado a 'aprobado'

                    'date_reviewed' => $current_date,

                    'admin_user_id' => $user->id,

                    'admin_user_name' => $user->name,

                    'admin_user_email' => $user->email,

                ]);

  

            $array['message'] = 'Rewards approved successfully';

        }
```


[[Controller]]