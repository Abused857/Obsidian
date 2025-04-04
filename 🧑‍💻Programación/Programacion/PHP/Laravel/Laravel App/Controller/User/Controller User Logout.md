
![[UserLogout 1.pdf]]

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

[[Route-Api]][[Controller]]