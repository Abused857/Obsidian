### Paso 1: Crear el Controlador en Laravel

1. **Comando para crear el controlador:** Abre tu terminal en la raíz de tu proyecto Laravel y ejecuta el siguiente comando para crear el controlador `UserController`:

    
    `php artisan make:controller UserController`
    
    Esto generará un nuevo archivo `UserController.php` en `app/Http/Controllers`.
    

### Paso 2: Copiar el Código al Controlador

Abre el archivo `UserController.php` recién creado en tu editor de código y reemplaza su contenido con el siguiente código:

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Validator;
use App\Models\User;
use Carbon\Carbon;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\Storage;
use App\Models\Company;
use App\Models\UserToken;
use App\Models\Setting;
use App\Models\UserRememberToken;
use Laravel\Socialite\Facades\Socialite;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Log;

class UserController extends Controller
{
   public function loginSocial(Request $request)

    {

  

        $array['error'] = 200;

  

        try {

  

            DB::beginTransaction();

  

            //Validation

            $error = 0;

            $message_validator = collect();

  

            $validator = Validator::make($request->all(), [

                'socialToken' => 'required',

                'driver' => 'required',

            ]);

  
  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            if ($error == 0) {

                $driver = $request->input('driver');

                if ($driver != 'google') {

                    $error = 1;

                    $message_validator = $message_validator->merge(['driver' => ['Driver not implemented.']]);

                }

            }

  

            if ($error == 0) {

                $socialToken = $request->input('socialToken');

                try {

                    $user = Socialite::driver($driver)->userFromToken($socialToken);

                    $provider = $user->getId();

                } catch (\Exception $e) {

                    $error = 1;

                    $message_validator = $message_validator->merge(['token' => [$e->getMessage()]]);

                }

            }

  

            if ($error == 0) {

                $data_user = User::where('email', $user->getEmail())->first();

  

                if ($data_user) {

                    if ($driver == 'google') {

                        if ($data_user->google_id && $data_user->google_id != $provider) {

                            $error = 2;

                            $message_validator = $message_validator->merge(['token' => ['The google id field is not valid.']]);

                        }

                    }

                } else {

                    $error = 2;

                    $message_validator = $message_validator->merge(['token' => ['User unauthorized.']]);

                }

            }

  

            if ($error == 0) {

                $data_company = Company::where('id', $data_user->company_id)

                    ->where('db_companies.date_start', '<=', Carbon::now()->format('Y-m-d'))

                    ->where('db_companies.date_end', '>=', Carbon::now()->format('Y-m-d'))

                    ->first();

  

                if (!$data_company) {

                    $error = 2;

                    $message_validator = $message_validator->merge(['token' => ['Invalid company.']]);

                }

            }

  

            if ($error == 0) {

                if ($data_user->bloqued_login == 1) {

                    $error = 3;

                    $message_validator = $message_validator->merge(['token' => ['User bloqued.']]);

                }

            }

  

            if ($error == 0) {

                if ($data_user->bloqued_to >= Carbon::now()) {

                    $error = 3;

                    $message_validator = $message_validator->merge(['token' => ['User bloqued temporarily.']]);

                }

            }

  
  

            if ($error == 0) {

                $token = base64_encode(Str::random(200));

                $remember_token = base64_encode(Str::random(200));

                $time_expired = intval(Setting::where('name', 'token_validate')->first()->value);

                $expired_at = Carbon::now()->addMinutes($time_expired);

  

                $refresh_token = base64_encode(Str::random(200));

                $refresh_expired_at = Carbon::now()->addMinutes(2 * $time_expired);

  

                UserToken::create([

                    'user_id' => $data_user->id,

                    'token' => $token,

                    'expired_at' => $expired_at,

                    'refresh_token' => $refresh_token,

                    'refresh_expired_at' => $refresh_expired_at,

                ]);

  

                UserRememberToken::create([

                    'user_id' => $data_user->id,

                    'token' => $remember_token,

                    'ip' => $request->ip(),

                ]);

  

                if ($driver == 'google' && !$data_user->google_id) {

                    $data_user->google_id = $provider;

                }

  

                $data_user->social_register = 1;

                $data_user->last_login = Carbon::now();

                $data_user->faults_login = 0;

                $data_user->verified_email = 1;

                $data_user->save();

  

                $array['data'][] = [

                    'user_id' => $data_user->id,

                    'token' => 'Bearer ' . $token,

                    'remember_token' => $remember_token,

                    'refresh_token' => $refresh_token

                ];

            } elseif ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][0] = $message_validator;

            } elseif ($error == 2) {

                $array['error'] = 401;

                $array['error_description'] = 'Unauthorized.';

                $array['error_inputs'][0] = $message_validator;

            } elseif ($error == 3) {

                $array['error'] = 403;

                $array['error_description'] = 'Forbidden.';

                $array['error_inputs'][0] = $message_validator;

            }

  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error';

            $array['error_catch'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

  

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }

  
  

    public function logout(Request $request)

    {

        $array['error'] = 200;

        try {

            DB::beginTransaction();

            if ($request->header('Authorization')) {

                $token_api = explode(' ', $request->header('Authorization'));

                if ($request->headers->has('Authorization')) {

                    $token = UserToken::where('token', '=', $token_api[1])->first();

                    if ($token) {

                        $token->delete();

                    } else {

                        $array['error'] = 404;

                        $array['error_description'] = 'Data not found.';

                    }

  

                    if ($request->input('remember_token') != '') {

                        UserRememberToken::where('token', '=', $request->input('remember_token'))->delete();

                    }

                }

            } else {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = ['token' => ['Authorization header is required.']];

            }

  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error.';

            $array['error_catch'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

  

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }

  
  
  

    public function RememberToken(Request $request)

    {

        $array['error'] = 200;

        try {

            DB::beginTransaction();

  
  

            $error = 0;

            $message_validator = collect();

  

            $validator = Validator::make($request->all(), [

                'remember_token' => 'required'

            ]);

  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            if ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][0] = $message_validator;

            } else {

  

                $remember_token = $request->input('remember_token');

  

                $user = User::whereHas('userRemeberToken', function ($query) use ($remember_token) {

                    $query->where('token', $remember_token);

                })

                    ->where('verified_email', '=', 1)

                    ->where('bloqued_login', '!=', 1)

                    ->where(function ($query) {

                        $query->whereNull('bloqued_to')->orWhere('bloqued_to', '<', Carbon::now());

                    })->first();

  
  

                if ($user) {

                    $company = Company::where('id', $user->company_id)

                        ->where('db_companies.date_start', '<=', Carbon::now()->format('Y-m-d'))

                        ->where('db_companies.date_end', '>=', Carbon::now()->format('Y-m-d'))

                        ->first();

                }

  

                if ($user && $company) {

                    $token = base64_encode(Str::random(200));

                    $remember_token = base64_encode(Str::random(200));

  

                    $time_expired = intval(Setting::where('name', 'token_validate')->first()->value);

                    $expired_at = Carbon::now()->addMinutes($time_expired);

  
  

                    $refresh_token = base64_encode(Str::random(200));

                    $refresh_expired_at = Carbon::now()->addMinutes(2 * $time_expired);

  

                    UserRememberToken::where('token', '=', $request->input('remember_token'))->update([

                        'token' => $remember_token,

                        'ip' => $request->ip(),

                    ]);

  

                    UserToken::create([

                        'user_id' => $user->id,

                        'token' => $token,

                        'expired_at' => $expired_at,

                        'refresh_token' => $refresh_token,

                        'refresh_expired_at' => $refresh_expired_at,

                    ]);

  

                    User::where('id', $user->id)->update([

                        'last_login' => Carbon::now()

                    ]);

  

                    $array['data'][] = ['user_id' => $user->id, 'token' => 'Bearer ' . $token, 'remember_token' => $remember_token, 'refresh_token' => $refresh_token, 'profile_completed' => $user->profile_completed];

                } else {

                    $array['error'] = 404;

                    $array['error_description'] = 'Data not found.';

                }

            }

  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error.';

            $array['error_catch'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }

  
  

    public function RefreshToken(Request $request)

    {

  

        $array['error'] = 200;

        try {

            DB::beginTransaction();

  

            $error = 0;

            $message_validator = collect();

  

            $validator = Validator::make($request->all(), [

                'refresh_token' => 'required'

            ]);

  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            if ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][0] = $message_validator;

            } else {

  

                $refresh_token = $request->input('refresh_token');

                $user_token = UserToken::where('refresh_token', '=', $refresh_token)->where('refresh_expired_at', '>=', Carbon::now())->first();

  

                $user = null;

                $company = null;

                if ($user_token != '') {

                    $user = User::where('id', $user_token->user_id)

                        ->where('verified_email', '=', 1)

                        ->where('bloqued_login', '!=', 1)

                        ->where(function ($query) {

                            $query->whereNull('bloqued_to')->orWhere('bloqued_to', '<', Carbon::now());

                        })->first();

  

                    if ($user) {

  

                        $company = Company::where('id', $user->company_id)

                            ->where('db_companies.date_start', '<=', Carbon::now()->format('Y-m-d'))

                            ->where('db_companies.date_end', '>=', Carbon::now()->format('Y-m-d'))

                            ->first();

                    }

                }

  

                if ($user && $company) {

                    $token = base64_encode(Str::random(200));

  

                    $time_expired = intval(Setting::where('name', 'token_validate')->first()->value);

                    $expired_at = Carbon::now()->addMinutes($time_expired);

  

                    $refresh_token = base64_encode(Str::random(200));

                    $refresh_expired_at = Carbon::now()->addMinutes(2 * $time_expired);

  

                    UserToken::create([

                        'user_id' => $user->id,

                        'token' => $token,

                        'expired_at' => $expired_at,

                        'refresh_token' => $refresh_token,

                        'refresh_expired_at' => $refresh_expired_at,

                    ]);

  

                    User::where('id', $user->id)->update([

                        'last_login' => Carbon::now()

                    ]);

  

                    $user_token->delete();

  

                    $array['data'][] = ['user_id' => $user->id, 'token' => 'Bearer ' . $token, 'refresh_token' => $refresh_token, 'profile_completed' => $user->profile_completed];

                } else {

                    $array['error'] = 404;

                    $array['error_description'] = 'Data not found.';

                }

            }

  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error.';

            $array['error_catch'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }

  
  

    public function profile(Request $request)

    {

        $array['error'] = 200;

  
  
  

        try {

            DB::beginTransaction();

  

            $user_id = '';

            if ($request->header('Authorization') != '') {

                $token = explode(' ', $request->header('Authorization'));

                $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', Carbon::now())->first();

                if ($user_token) {

                    $user_id = $user_token->user_id;

                }

            }

  

            $data_user = DB::table('db_users')

                ->select(

                    'db_users.id',

                    'db_users.name',

                    'db_users.surname',

                    'db_users.email',

                    'db_users.image_id',

                    'db_users.registration_date',

                    'db_users.company_id',

                    'db_users.social_register',

                    'db_users.profile_completed',

                    'db_user_points.points',

                    'db_user_wallet_virtuals.virtual_amount',

                )

                ->whereNull('db_users.deleted_at')

                ->where('db_users.id', '=', $user_id)

                ->leftJoin('db_user_points', function ($join) {

                    $join->on('db_users.id', 'db_user_points.user_id')

                        ->whereNull('db_user_points.deleted_at');

                })

                ->leftJoin('db_user_wallet_virtuals', function ($join) {

                    $join->on('db_users.id', 'db_user_wallet_virtuals.user_id')

                        ->whereNull('db_user_wallet_virtuals.deleted_at');

                })

  

                ->first();

  

            // Precompiled SQL

            $sql_file = DB::table('db_files')

                ->select(

                    'id',

                    'name',

                    'url'

                )

                ->whereNull('deleted_at');

  

            $sql_company = DB::table('db_companies')

                ->select(

                    'id',

                    'name',

                    'description',

                    'short_description',

                    'image_id'

                )

                ->whereNull('deleted_at');

            // END Precompiled SQL

  

            $array['data'] = array();

  

            $array_user = array();

            $array_user['id'] = $data_user->id;

            $array_user['name'] = $data_user->name;

            $array_user['surname'] = $data_user->surname;

            $array_user['email'] = $data_user->email;

            $array_user['registration_date'] = $data_user->registration_date;

            $array_user['social_register'] = $data_user->social_register;

            $array_user['profile_completed'] = $data_user->profile_completed;

            $array_user['points'] = round($data_user->points,0);

            $array_user['virtual_amount'] = round($data_user->virtual_amount, 2);

  

            $array_user['file']['image'] = array();

            if ($data_user->image_id != '' && $data_user->image_id != null) {

                $sql_user_image = clone $sql_file;

                $data_user_image = $sql_user_image->where('db_files.id', $data_user->image_id)->first();

                if ($data_user_image) {

                    $array_file = array();

                    $array_file['id'] = $data_user_image->id;

                    $array_file['name'] = $data_user_image->name;

                    $array_file['url'] = Storage::url($data_user_image->url);

                    $array_user['file']['image'][] = $array_file;

                }

            }

  

            $array_user['company'] = null;

            if ($data_user->company_id != '' && $data_user->company_id != null) {

                $sql_user_company = clone $sql_company;

                $data_user_company = $sql_user_company->where('id', $data_user->company_id)->first();

  

                if ($data_user_company) {

                    $array_company = array();

                    $array_company['id'] = $data_user_company->id;

                    $array_company['name'] = $data_user_company->name;

                    $array_company['description'] = $data_user_company->description;

                    $array_company['short_description'] = $data_user_company->short_description;

  

                    $array_company['file']['image'] = array();

                    if ($data_user_company->image_id != '' && $data_user_company->image_id != null) {

                        $sql_image_clone = clone $sql_file;

                        $data_image = $sql_image_clone->where('db_files.id', $data_user_company->image_id)->first();

                        if ($data_image) {

                            $array_image = array();

                            $array_image['id'] = $data_image->id;

                            $array_image['name'] = $data_image->name;

                            $array_image['url'] = Storage::url($data_image->url);

                            $array_company['file']['image'][] = $array_image;

                        }

                    }

  

                    $array_user['company'] = $array_company;

                }

            }

  

            $array['data'][0]['user'][] = $array_user;

  
  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $response['status'] = $e->getCode() ?: 500;

            $response['message'] = $e->getMessage() ?: 'Error getting user profile.';

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }

  

    public function ranking(Request $request)

    {

  

        $array['error'] = 200;

  

        try {

  

            DB::beginTransaction();

  

            $error = 0;

            $message_validator = collect();

  

            $validator = Validator::make($request->all(), [

                'page' => 'integer|min:0',

                'limit' => 'integer|min:0',

  

            ]);

  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            // User and company request

            $user_id = '';

            $company_id = '';

            if ($request->header('Authorization') != '') {

                $token = explode(' ', $request->header('Authorization'));

                $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', Carbon::now())->first();

                if ($user_token) {

                    $user_id = $user_token->user_id;

                }

            }

            $user = User::where('id', $user_id)->first();

            if ($user) {

                $company_id = $user->company_id;

            }

  

            if ($error == 0) {

  

                $sql_user = DB::table('db_users')

                    ->select(

                        'db_users.id',

                        'db_users.name',

                        'db_users.surname',

                        'db_users.email',

                        'db_users.image_id',

                        'db_user_points.points',

                        'db_user_wallet_virtuals.virtual_amount'

                    )

                    ->whereNull('db_users.deleted_at')

                    ->leftJoin('db_user_points', function ($join) {

                        $join->on('db_users.id', 'db_user_points.user_id')

                            ->whereNull('db_user_points.deleted_at');

                    })

                    ->leftJoin('db_user_wallet_virtuals', function ($join) {

                        $join->on('db_users.id', 'db_user_wallet_virtuals.user_id')

                            ->whereNull('db_user_wallet_virtuals.deleted_at');

                    })

                    ->where('db_users.company_id', '=', $company_id);

                $settings = Setting::all();

  

                $sql_file = DB::table('db_files')

                    ->select(

                        'id',

                        'name',

                        'url'

                    )

                    ->whereNull('deleted_at');

  

                // Order

                $order = 'db_user_points.points';

                $request_order = $request->input('order');

  

                if ($request_order != '') {

                    switch ($request->input('order')) {

                        case 'virtual_amount':

                            $order = 'db_user_wallet_virtuals.virtual_amount';

                            break;

                        case 'points':

                            $order = 'db_user_points.points';

                            break;

                        default:

                            $order = 'db_user_points.points';

                            break;

                    }

                }

  

                // Order way

                $order_way = 'desc';

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

                $sql_user->groupBy('db_users.id')

                    ->orderBy($order, $order_way)

                    ->orderBy('db_users.name', 'ASC');

  

                $sql_user_count = clone $sql_user;

  

                // Pagination based on filters and execute of query

                if ($request->input('limit') != '0') {

  

                    $limit = $settings->where('name', '=', 'limit_registers')->first()->value;

  

                    if ($request->input('limit') != '' && $request->input('limit') <= $limit) {

                        $limit = $request->input('limit');

                    }

  

                    $page = 0;

                    if ($request->input('page') != '') {

                        $page = $request->input('page');

                    }

  

                    $data_user = $sql_user->forPage($page, $limit)->get();

  

                    // If limite = 0 get all results

                } else {

                    $data_user = $sql_user->get();

                }

                // ENF pagination

  

                $count = DB::table(DB::raw("({$sql_user_count->toSql()}) as sub"))->mergeBindings($sql_user_count);

                $total_results = $count->count();

                $array['data'] = array();

  

                foreach ($data_user as $user) {

                    $array_user = array();

                    $array_user['id'] = $user->id;

                    $array_user['name'] = $user->name;

                    $array_user['surname'] = $user->surname;

                    $array_user['email'] = $user->email;

                    $array_user['points'] = round($user->points,0);

                    // $array_user['virtual_amount'] = round($user->virtual_amount, 2);

  

                    $array_user['file']['image'] = array();

                    if ($user->image_id != '' && $user->image_id != null) {

                        $sql_user_image = clone $sql_file;

                        $data_user_image = $sql_user_image->where('db_files.id', $user->image_id)->first();

                        if ($data_user_image) {

                            $array_file = array();

                            $array_file['id'] = $data_user_image->id;

                            $array_file['name'] = $data_user_image->name;

                            $array_file['url'] = Storage::url($data_user_image->url);

                            $array_user['file']['image'][] = $array_file;

                        }

                    }

  

                    $array['data'][0]['user'][] = $array_user;

                }

                $array['total_results'] = $total_results;

            } elseif ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = $message_validator;

            }

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $response['status'] = $e->getCode() ?: 500;

            $response['message'] = $e->getMessage() ?: 'Error getting user profile.';

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = "Error in " . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' .  $e->getMessage() . ' ' . $e->getLine());

        }

  

        return response()->json($array, $array['error']);

    }
}


```

### Paso 3: Guardar el Controlador

Guarda el archivo `UserController.php` después de haber copiado el código proporcionado.

### Paso 4: Nota en Obsidian

1. Abre Obsidian y crea una nueva nota con el título que prefieras, por ejemplo, "UserController Laravel".
2. Copia y pega el código del controlador `UserController` en la nota de Obsidian.

Ahora tendrás tu controlador Laravel creado y el código bien organizado en tu aplicación. Asegúrate de revisar y ajustar las configuraciones y nombres de modelos según la estructura de tu proyecto.

Update de una tabla añadiendo/sumando

```
 DB::table('db_user_wallet_virtuals')

                                ->where('id', '=', $user_wallet->id)

                                ->update([

                                    'virtual_amount' => DB::raw("virtual_amount + {$reward->points}"),

                                    'updated_at' => now()

                                ]);
```

[[PHP]]  [[Route-Api]]  