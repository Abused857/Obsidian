![[UserSocialLogin 1.pdf]]


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

 [[Controller]]  [[Route-Api]]   [[Services.php]]