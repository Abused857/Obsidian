
# Company Show

Ruta: (get) api/v1/company (middlewares que observan esta ruta [`AuthenticationChannel`]), la cual apunta al controlador CompanyController, método show.

Muestra la información contenida en las tablas lp_companies y lp_files que sería la imagen ligada a cada compañía en cuestión.

Dicha consulta por defecto devolverá todas compañías con la foto de cada una.

Parámetros, filtros que acepta/gestiona la api:

Que tengan que pasar el validator (laravel por detrás crea una nueva instancia de esta clase y comprueba en caso de haber sido enviados, los requisitos que debe cumplir cada parámetro dentro de este validador).

Esto quiere decir que el parámetro page en caso de ser enviado por el front, debe ser un número entero donde su valor mínimo solo puede ser cero (Img 1), por lo tanto en que caso de que no se cumpla algún requisito en los parámetros enviados que necesitan ser validados, se considerará un error se recogerá dicho error y se mostrará en la respuesta (Img 2 ) será un error 400 los campos no están en el formato esperado (Img 3).


![[Pasted image 20240715082932.png]]

Img 1

![[Pasted image 20240715083000.png]]
Img 2

![[Pasted image 20240715083012.png]]
Img 3

Lo siguiente que hace el método en caso de que hasta aquí no se haya enviado ningún param que requiera superar el validator, o los enviados lo hayan superado, es crear una variable con la información de todas las compañías **EJECUTAMOS EL ESQUEMA DE LA CONSULTA NO LA CONSULTA EN SI!!! NO CONTIENE INFO AUN** donde el valor de la propiedad deleted_at sea distinta de null (img 4).

![[Pasted image 20240715083025.png]]
img 4

Con nuestra variable ya generada lo primero que realizamos es un filtrado para evitar iteraciones y gasto computacional innecesario, esto quiere decir, que si pasamos como parámetro id 1, ejecutara el esquema de la consulta e iterará hasta encontrar dicho resultado.

**SEGUIMOS SIN TENER UN RESULTADO COMO TAL EN EL ESQUEMA.**

Por lo tanto en este filtrado opcional los posibles parámetros esperados serán:

**name**: Buscará de entre todas las compañías de nuestra variable, el name, description o short_description que haya sido enviado desde el front (Img 5) ejemplo:

si quieres la compañía que se llame Incentro; param=name, value =Incentro.

![[Pasted image 20240715083049.png]]
**id**: Buscará de entre todas las compañías de nuestra variable el id de la compañia que coincida con el id enviado(img 6) ejemplo:

si quieres la data de la compañía con id 1, param = id, value = 1.

![[Pasted image 20240715083056.png]]
img 6

**url**: Buscará de entre todas las compañías de nuestra variable la url de la compañia que coincida con la url enviada (img 7) ejemplo:

si quieres la data de la compañía con url ('url'), param = url, value = ('url').

![[Pasted image 20240715083107.png]]
Img 7

**active:** Buscará de entre todas las compañías de nuestra variable las que estén activas comprobando que la fecha de comienzo sea anterior a la fecha actual y la fecha de fin sea posterior a la fecha actual (img 8) ejemplo:

El parametro active no tiene que estar vacío y tener el valor 1 (parámetro incluido en el validator img 1), por lo tanto si queremos solo la lista de las compañias activas, param = active , value = 1.

![[Pasted image 20240715083116.png]]
Img 8

Siguiente paso es ya con la lista filtrada o no, volver a iterar sobre nuestra variable con todas las compañías y ordenarlas por defecto por el nombre de la compañía de manera ascendente.

En caso de requerir otra ordenación, se han habilitado las siguientes opciones para la propiedad order (Img 9) que contendrá esta información:

Description, short_description, date_star, date_end, id.

![[Pasted image 20240715083124.png]]
Img 9

**CUIDADO,** date_start y date_end ordenarán por la fecha mas reciente o tardía de la lista, no desde una fecha enviada, no soporta la funcionalidad de ordénamelo por ejemplo a partir de la fecha 2025-10-10.

Ejemplo de una ordenación por la fecha en la que suscribieron las compañias:

param = order, value = date_start (order_way por defectio es asc) por lo tanto se espera en la respuesta que la primera compañia que aparezca sea la primera que se suscribió y la ultima la compañía mas reciente en suscribirse.

En cuanto al order_way ( Img 10) solo hay ascendente o descendente, por defecto si no se envía el param será ascendente, para realizarla descendente:

param = order_way, value = desc.

![[Pasted image 20240715083135.png]]
Img 10

Tras realizar la ordenación haya sido introducida o no (order y order_way tienen default values como se mencionó), se ejecuta la ordenación (Img 11):

![[Pasted image 20240715083143.png]]
Img 11

Paginación, queremos que cada página muestre 5 registros en dos páginas, param = limit value = 5, param = page, value = 2, mostrará los primero 5 registros encontrados en la primera página y los siguientes 5 en la segunda página, el resto no se mostrará si hay menos tampoco esto se realiza en (Img 12)

![[Pasted image 20240715083157.png]]
Img 12

**AQUI OBTENEMOS LOS RESULTADOS** (img 12), si hay un limite y una page (es 0 por defecto page), devolvemos en base a eso todos los resultados que contiene nuestro esquema y lo volcamos en data_company.

Si no hay limit, simplemente lo volcamos sin ninguna paginación.

Por defecto si no se pasa este parámetro la consulta devolverá todos los resultados.

Con todo preparado el resto ya es recorrer las dos tablas y mostrar la información de dichas tablas (img 13).

![[Pasted image 20240715083206.png]]
Img 13

Formato de ejemplo de una respuesta de una compañía en json (img 14).

![[Pasted image 20240715083216.png]]
Img 14

El total results, creamos un clone de nuestra consulta principal antes de la paginación (img 15).

![[Pasted image 20240715083225.png]]
img 15

Esto se debe a que si el front quiere 5 resultados en dos páginas, tras la paginación el count nos va a decir que hay 5 registros (1 página) pero el count son 10 (5 por página(img 16)).

![[Pasted image 20240715083231.png]]
Img 16

```
<?php

  

namespace App\Http\Controllers;

  

use Illuminate\Http\Request;

use Illuminate\Support\Facades\DB;

use Illuminate\Support\Facades\Log;

use App\Models\User;

use Illuminate\Support\Facades\Validator;

use App\Models\UserToken;

use Illuminate\Support\Facades\Storage;

use App\Models\Setting;

use Carbon\Carbon;

  

class CompanyController extends Controller

{

  

    public function show(Request $request)

    {

        $array['error'] = 200;

  

        try {

            DB::beginTransaction();

  

            // Fields validation

            $error = 0;

            $message_validator = collect();

  

            $validator = Validator::make($request->all(), [

                'page' => 'integer|min:0',

                'limit' => 'integer|min:0',

                'active' => 'boolean'

            ]);

  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

            // End field validation

  

            if ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = $message_validator;

            } else {

                $sql_company = DB::table('db_companies')

                    ->select(

                        'db_companies.id',

                        'db_companies.name',

                        'db_companies.short_description',

                        'db_companies.description',

                        'db_companies.url',

                        'db_companies.email',

                        'db_companies.contact_person',

                        'db_companies.date_start',

                        'db_companies.date_end',

                        'db_companies.image_id',

                        'db_companies.social_login_google',

                    )

                    ->whereNull('db_companies.deleted_at');

  

                // Filter

                if ($request->input('name') != '') {

  

                    $sql_company->Where(function ($query) use ($request) {

                        $query->where('db_companies.name', 'Like', '%' . $request->input('name') . '%')

                            ->orWhere('db_companies.description', 'Like', '%' . $request->input('name') . '%')

                            ->orWhere('db_companies.short_description', 'Like', '%' . $request->input('name') . '%');

                    });

                }

  

                if ($request->input('id') != '') {

                    $sql_company->where('db_companies.id', '=', $request->input('id'));

                }

  

                if ($request->input('url') != '') {

                    $sql_company->where('db_companies.url', '=', $request->input('url'));

                }

  

                if ($request->input('active') != '' && $request->input('active') == 1) {

                    $sql_company->where('db_companies.date_start', '<=', Carbon::now()->format('Y-m-d'))

                        ->where('db_companies.date_end', '>=', Carbon::now()->format('Y-m-d'));

                }

                // END Filter

  
  

                // Order

                $order = 'db_companies.name'; // Default order

                $request_order = $request->input('order');

                if ($request_order != '') {

                    switch ($request_order) {

                        case 'name':

                            $order = 'db_companies.name';

                            break;

                        case 'description':

                            $order = 'db_companies.description';

                            break;

                        case 'short_description':

                            $order = 'db_companies.short_description';

                            break;

                        case 'date_start':

                            $order = 'db_companies.date_start';

                            break;

                        case 'date_end':

                            $order = 'db_companies.date_end';

                            break;

                        case 'id':

                            $order = 'db_companies.id';

                            break;

                        default:

                            $order = 'db_companies.name';

                            break;

                    }

                }

                // END Order

  

                // Order_way

                $order_way = 'asc';

                $request_order_way = $request->input('order_way');

                if ($request_order_way != '') {

                    switch ($request->input('order_way')) {

                        case 'asc':

                            $order_way = 'asc';

                            break;

                        case 'desc':

                            $order_way = 'desc';

                            break;

                        default:

                            $order_way = 'asc';

                            break;

                    }

                }

                // END Order_way

  

                $sql_company->orderBy($order, $order_way);

  

                $sub = clone $sql_company;

                // Page based on filters and execute a query

  

                if ($request->input('limit') != '0') {

                    $settings = Setting::where('name', '=', 'limit_registers')->first();

                    $limit = $settings->value;

  

                    if ($request->input('limit') != '' && $request->input('limit') <= $limit) {

                        $limit = $request->input('limit');

                    }

  

                    $page = 0;

                    if ($request->input('page') != '') {

                        $page = $request->input('page');

                    }

  

                    $data_company = $sql_company->forPage($page, $limit)->get();

  

                    // if filter limit = 0 obtain all results

                } else {

                    $data_company = $sql_company->get();

                }

                // Pagination end

  

                $count = DB::table(DB::raw("({$sub->toSql()}) as sub"))->mergeBindings($sub);

  

                $total_results = $count->count();

  

                $array['data'] = array();

                foreach ($data_company as $company) {

                    $array_company = array();

  

                    $array_company['id'] = $company->id;

                    $array_company['name'] = $company->name;

                    $array_company['description'] = $company->description;

                    $array_company['short_description'] = $company->short_description;

                    $array_company['url'] = $company->url;

                    $array_company['email'] = $company->email;

                    $array_company['contact_person'] = $company->contact_person;

                    $array_company['social_login_google'] = $company->social_login_google;

                    $array_company['date_start'] = $company->date_start;

                    $array_company['date_end'] = $company->date_end;

  

                    $array_company['file']['image'] = array();

                    if ($company->image_id != '' && $company->image_id != null) {

                        $data_image = DB::table('db_files')

                            ->select('db_files.id', 'db_files.name', 'db_files.url')

                            ->where('db_files.id', $company->image_id)

                            ->whereNull('db_files.deleted_at')

                            ->first();

                        if ($data_image) {

                            $array_image = array();

                            $array_image['id'] = $data_image->id;

                            $array_image['name'] = $data_image->name;

                            $array_image['url'] = Storage::url($data_image->url);

                            $array_company['file']['image'][] = $array_image;

                        }

                    }

  

                    $array['data'][0]['company'][] = $array_company;

                }

  

                $array['total_results'] = $total_results;

            }

  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error.';

            $array['error_catch'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

        }

  

        return response()->json($array, $array['error']);

    }

}


```

[[Controller]][[Route-Api]]