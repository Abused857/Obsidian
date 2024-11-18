
<?php

  

namespace App\Http\Controllers;

  

use Illuminate\Http\Request;

use Illuminate\Support\Facades\DB;

use Carbon\Carbon;

use Illuminate\Support\Facades\Validator;

use App\Models\Language;

use App\Models\UserToken;

use App\Models\User;

use App\Models\Reward;

use App\Models\Action;

use Illuminate\Support\Str;

use App\Models\ActionTranslation;

use Illuminate\Support\Facades\Log;

use Illuminate\Support\Facades\Route;

use App\Models\Setting;

use App\Models\UserPoint;

use Illuminate\Support\Facades\Storage;

  

class RewardController extends Controller

{

    public function create(Request $request)

    {

        $array['error'] = 200;

        try {

            DB::beginTransaction();

  

            //Validation

            $error = 0;

            $message_validator = collect();

            $message_point = '';

            $file_point = '';

            $action_id = '';

            $user_id = '';

            $company_id = '';

            $current_date = Carbon::now();

  
  

            $validator = Validator::make($request->all(), [

                'action_id' => 'required|integer|min:0|exists:db_actions,id,deleted_at,NULL',

                'lang' => 'required',

  

            ]);

  

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            // Check language

            $language = Language::where('abbreviation', Str::upper($request->input('lang')))->first();

  
  

            if ($request->header('Authorization') != '') {

                $token = explode(' ', $request->header('Authorization'));

                $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', $current_date)->first();

                $user_id = $user_token->user_id;

                $user = User::where('id', $user_id)->first();

            }

            if ($user_id) {

                $company_id = $user->company_id;

            }

            if (!$user_id || !$company_id) {

                $error = 1;

                $message_validator = $message_validator->merge(['ask_points' => ['The company is not correct.']]);

            }

            if ($error == 0) {

  

                //ejemplo falta por definir entre otras cosas si la id de admin va a ser la 1 o se filtra por name admin etc

                $sql_admins = DB::table('db_roles')

                    ->select(

                        'db_roles.id',

                        'db_user_roles.user_id as user_id',

                        'db_roles.name as user_role'

  

                    )

                    ->whereNull('db_roles.deleted_at')

                    ->join('db_user_roles', function ($join) {

                        $join->on('db_user_roles.id', '=', 'db_roles.id')

                            ->whereNull('db_user_roles.deleted_at');

                    })

                    ->join('db_users', function ($join) use ($company_id) {

                        $join->on('db_users.id', '=', 'db_user_roles.user_id')

                            ->where('db_roles.name', 'like', '%admin%')

                            ->where('db_roles.company_id', '=', $company_id);

                    })

                    ->get();

  

                //variable capture and check if it exists on data base

                if ($request->input('textArea') != '') {

  

                    $message_point = $request->input('textArea');

                }

  

                if ($request->hasFile('file')) {

                    $file_point = $request->input('file');

                }

  

                $action_id = $request->input('action_id');

                //como solo se va a realizar una vez al igual que crear  estaría bien así o tendría que hacer la consulta

                // $action = Action::where('id', $action_id)->first();

  

                //no se si sirve para algo el nombre de la accion pero bueno hecho queda si hay que borrar pos na

                $action = Action::find($action_id);

  

                if ($action->company_id != $user->company_id) {

                    $error = 1;

                    $message_validator = $message_validator->merge(['reward' => ['action not valid']]);

                }

                $action_reward_exists = Reward::where('db_rewards.action_id', $action_id)->where('db_rewards.user_id', $user_id)->first();

  

                if ($action_reward_exists) {

  

                    $error = 1;

                    $message_validator = $message_validator->merge(['reward' => ['Already asked for points']]);

                }

                $action_translation = ActionTranslation::where('action_id', $action->id)

                    ->where('language_id', $language->id)

                    ->first();

  

                if (!$action_translation) {

  

                    $error = 1;

                    $message_validator = $message_validator->merge(['reward' => ['Action doesnt have translation']]);

                }

  

                if ($error == 0) {

  
  
  
  
  

                    $current_date = Carbon::now()->format('Y-m-d');

                    $askPointInfo = [

                        'action_id' => $action_id,

                        'action_name' => $action_translation->name ? $action_translation->name : null,

                        'user_id' => $user->id,

                        'user_name' => $user->name,

                        'message' => $message_point ? $message_point : null,

                        'status_id' => 1,

                        'company_id' => $user->company_id,

                        'user_email' => $user->email,

                        'user_surname' => $user->surname ? $user->surname : null,

                        'date_request' => $current_date,

                        'action_code' => $action->code ? $action->code : null,

  

                    ];

  

                    Reward::create($askPointInfo);

                }

            }

            if ($error == 1) {

  

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = $message_validator;

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

  

    public function show(Request $request)

    {

        $array['error'] = 200;

        try {

            DB::beginTransaction();

            //Validation

            $error = 0;

            $message_validator = collect();

            $current_date = Carbon::now();

            $validator = Validator::make($request->all(), []);

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

            if ($request->header('Authorization') != '') {

                $token = explode(' ', $request->header('Authorization'));

                $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', $current_date)->first();

                $user_id = $user_token->user_id;

                $user = User::where('id', $user_id)->first();

            }

            if ($user) {

  

                $company_id = $user->company_id;

            }

            if (!$user || !$company_id) {

                $error = 1;

                $message_validator = $message_validator->merge(['rewards asked' => ['The company is not correct.']]);

            }

            if ($error == 0) {

  

                $sql_pending_rewards = DB::table('db_rewards')

                    ->select(

                        'db_rewards.*',

                    )

                    ->whereNull('db_rewards.deleted_at')

                    ->where('db_rewards.company_id', '=', $user->company_id)

                    ->leftJoin('db_reward_files', 'db_reward_files.reward_id', '=', 'db_rewards.id')

                    ->whereNull('db_reward_files.deleted_at');

                $settings = Setting::all();

                if ($request->input('id')) {

                    $sql_pending_rewards->where('db_rewards.id', '=', $request->input('id'));

                }

                //añado un error = 1 en caso de que el status no sea un all o un array de 1 2 3?

                if ($request->input('status') === 'all') {

                } else if ($request->input('status') === null || $request->input('status') == 0) {

  

                    $sql_pending_rewards->where('db_rewards.status_id', '=', 1);

                } else {

  

                    $sql_pending_rewards->where('db_rewards.status_id', '=', $request->input('status'));

                }

  

                if ($request->input('user_id')) {

  

                    $sql_pending_rewards->where('db_rewards.user_id', '=', $request->input('user_id'));

                }

                if ($request->input('action_id')) {

                    $sql_pending_rewards->where('db_rewards.action_id', '=', $request->input('action_id'));

                }

                if ($request->input('user_email')) {

                    $sql_pending_rewards->where('db_rewards.user_email', 'like', '%' . $request->input('user_email') . '%');

                }

                if ($request->input('user_name')) {

                    $sql_pending_rewards->where('db_rewards.user_name', 'like', '%' . $request->input('user_name') . '%');

                }

                if ($request->input('admin_id')) {

                    $sql_pending_rewards->where('db_rewards.admin_id', '=', $request->input('admin_id'));

                }

                if ($request->input('admin_name')) {

                    $sql_pending_rewards->where('db_rewards.admin_name', 'like', '%' . $request->input('admin_name') . '%');

                }

                if ($request->input('admin_email')) {

                    $sql_pending_rewards->where('db_rewards.admin_email', 'like', '%' . $request->input('admin_email') . '%');

                }

                if ($request->input('date')) {

                    $sql_pending_rewards->whereDate('db_rewards.created_at', '=', $request->input('date'));

                }

  

                $order = 'db_rewards.date_request';

                $request_order = $request->input('order');

                if ($request_order != '') {

                    switch ($request->input('order')) {

                        case 'user_id':

                            $order = 'db_rewards.user_id';

                            break;

                        case 'action_id':

                            $order = 'db_rewards.action_id';

                            break;

                        case 'user_email':

                            $order = 'db_rewards.user_email';

                            break;

                        case 'user_name':

                            $order = 'db_rewards.user_name';

                            break;

                        case 'admin_id':

                            $order = 'db_rewards.admin_id';

                            break;

                        case 'admin_name':

                            $order = 'db_rewards.admin_name';

                            break;

                        case 'admin_email';

                            $order = 'db_rewards.admin_email';

                            break;

                        case 'date':

                            $order = 'db_rewards.date';

                            break;

                    }

                }

  

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

                $sql_pending_rewards->groupBy('db_rewards.id')

                    ->orderBy($order, $order_way);

  

                $sql_pending_rewards_count = clone $sql_pending_rewards;

  

                if ($request->input('limit') != '0') {

  

                    $limit = $settings->where('name', '=', 'limit_registers')->first()->value;

  

                    if ($request->input('limit') != '' && $request->input('limit') <= $limit) {

                        $limit = $request->input('limit');

                    }

  

                    $page = 0;

                    if ($request->input('page') != '') {

                        $page = $request->input('page');

                    }

  

                    $data_rewards = $sql_pending_rewards->forPage($page, $limit)->get();

  

                    // If limite = 0 get all results

                } else {

                    $data_rewards = $sql_pending_rewards->get();

                }

  

                $count = DB::table(DB::raw("({$sql_pending_rewards_count->toSql()}) as sub"))->mergeBindings($sql_pending_rewards_count);

                $total_results = $count->count();

  

                $array['data'] = array();

  
  
  

                foreach ($data_rewards as $rewards) {

                    $array_rewards = array();

                    $array_rewards['id'] = $rewards->id;

                    $array_rewards['status_id'] = $rewards->status_id;

                    $array_rewards['message'] = $rewards->message;

                    $array_rewards['message_reviewed'] = $rewards->message_reviewed;

                    $array_rewards['admin_user_id'] = $rewards->admin_user_id;

                    $array_rewards['admin_user_email'] = $rewards->admin_user_email;

                    $array_rewards['admin_user_name'] = $rewards->admin_user_name;

                    $array_rewards['admin_user_surname'] = $rewards->admin_user_surname;

                    $array_rewards['user_id'] = $rewards->user_id;

                    $array_rewards['user_email'] = $rewards->user_email;

                    $array_rewards['user_name'] = $rewards->user_name;

                    $array_rewards['user_surname'] = $rewards->user_surname;

                    $array_rewards['action_id'] = $rewards->action_id;

                    $array_rewards['action_code'] = $rewards->action_code;

                    $array_rewards['date_request'] = $rewards->date_request;

                    $array_rewards['date_reviewed'] = $rewards->date_reviewed;

                    $array_rewards['points'] = $rewards->points;

                    $array_rewards['virtual_amount'] = $rewards->virtual_amount;

                    $array_rewards['token'] = $rewards->token;

                    $array_rewards['file'] = array();

  
  

                    $reward_files = DB::table('db_reward_files')

                        ->select(

                            'db_files.id',

                            'db_files.url',

                            'db_files.name'

                        )

                        ->join('db_files', 'db_files.id', '=', 'db_reward_files.file_id')

                        ->where('db_reward_files.reward_id', $rewards->id)

                        ->whereNull('db_files.deleted_at')

                        ->whereNull('db_reward_files.deleted_at')

                        ->get();

  

                    if ($reward_files) {

                        foreach ($reward_files as $file) {

                            $array_rewards['file'][] = [

                                'id' => $file->id,

                                'url' => Storage::url($file->url),

                                'name' => $file->name,

                            ];

                        }

                    }

  

                    $array['data'][0]['reward'][] = $array_rewards;

                }

                $array['total_results'] = $total_results;

            }

            if ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = $message_validator;

            }

  
  

            DB::commit();

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error';

            $array['error_catc'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = 'Error in ' . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' . $e->getMessage() . ' ' . $e->getLine());

        }

        return response()->json($array, $array['error']);

    }

  

    public function update(Request $request)

    {

        $array['error'] = 200;

  

        try {

            DB::beginTransaction();

  

            $error = 0;

            $message_validator = collect();

            $current_date = Carbon::now();

  

            $validator = Validator::make($request->all(), [

                'ids' => 'required|array',

                'ids.*' => 'exists:db_rewards,id',

                'changeStatus' => ['required', 'string', 'in:approved,rejected']

            ]);

            if ($validator->fails()) {

                $error = 1;

                $message_validator = $message_validator->merge($validator->errors());

            }

  

            if ($request->header('Authorization') != '') {

                $token = explode(' ', $request->header('Authorization'));

                $user_token = UserToken::where('token', '=', $token[1])->where('expired_at', '>=', $current_date)->first();

                $user_id = $user_token->user_id;

                $user = User::where('id', $user_id)->first();

            }

            if ($user_id) {

                $company_id = $user->company_id;

            }

            if (!$user_id || !$company_id) {

                $error = 1;

                $message_validator = $message_validator->merge(['ask_points' => ['The company is not correct.']]);

            }

            $ids = $request->input('ids');

            $status = $request->input('changeStatus') == 'approved' ? 2 : 3;

  

            if ($error == 0) {

  

                DB::table('db_rewards')

                    ->whereIn('id', $ids)

                    ->update([

                        'status_id' => $status,

                        'message_reviewed' => $request->input('message_reviewed'),

                        'admin_user_id' => $user->id,

                        'admin_user_email' => $user->email,

                        'admin_user_name' => $user->name,

                        'admin_user_surname' => $user->surname,

                        'date_reviewed' => $current_date,

                    ]);

                if ($status == 2) {

  

                    $rewards = DB::table('db_rewards')

                        ->whereIn('id', $ids)

                        ->get();

  

                    foreach ($rewards as $reward) {

                        $user_points = DB::table('db_user_points')

                            ->where('db_user_points.user_id', '=', $reward->user_id)

                            ->whereNull('db_user_points.deleted_at')->first();

  

                        if ($user_points) {

                            DB::table('db_user_points')

                                ->where('id', '=', $user_points->id)

                                ->update([

                                    'points' => DB::raw("points + {$reward->points}"),

  

                                ]);

                        } else {

                            DB::table('db_user_points')->insert([

                                'user_id' => $reward->user_id,

                                'points' => $reward->points,

                            ]);

                        }

  

                        $user_wallet = DB::table('db_user_wallet_virtuals')

                            ->where('db_user_wallet_virtuals.user_id', '=', $reward->user_id)

                            ->whereNull('db_user_wallet_virtuals.deleted_at')->first();

  

                        if ($user_wallet) {

                            DB::table('db_user_wallet_virtuals')

                                ->where('id', '=', $user_wallet->id)

                                ->update([

                                    'virtual_amount' => DB::raw("virtual_amount + {$reward->points}"),

                                ]);

                        } else {

                            DB::table('db_user_wallet_virtuals')->insert([

                                'user_id' => $reward->user_id,

                                'virtual_amount' => $reward->points,

                            ]);

                        }

                    }

                }

                DB::commit();

                $array['message'] = $status == 2 ? 'Rewards approved successfully' : 'Rewards rejected successfully';

            }

            if ($error == 1) {

                $array['error'] = 400;

                $array['error_description'] = 'The fields are not the required format.';

                $array['error_inputs'][] = $message_validator;

            }

        } catch (\Exception $e) {

            DB::rollBack();

            $array['error'] = 500;

            $array['error_description'] = 'Internal system error';

            $array['error_catc'] = $e->getMessage() . ' ' . $e->getLine() . ' ' . $e->getFile();

            $action = Route::currentRouteAction();

            list($controller, $method) = explode('@', $action);

            $controllerName = class_basename($controller);

            $errorMessage = 'Error in ' . $controllerName . '@' . $method;

            Log::error($errorMessage . ' ' . $e->getMessage() . ' ' . $e->getLine());

        }

        return response()->json($array, $array['error']);

    }

}