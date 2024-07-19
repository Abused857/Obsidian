
![[userRefreshTOken.pdf]]




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
 [[controller]]  [[Route-Api]] 