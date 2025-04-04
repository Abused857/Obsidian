
MAIL_MAILER=smtp
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your_email@example.com
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your_email@example.com
MAIL_FROM_NAME="${APP_NAME}"


composer require guzzlehttp/guzzle
composer require illuminate/mail


use App\Mail\EmailBlade;
use Illuminate\Support\Facades\Mail;

$data = ['key' => 'value'];
$template = 'template1'; // Cambia esto a 'template2', 'template3', o 'template4' según sea necesario.

Mail::to('example@example.com')->send(new EmailBlade($data, $template));


namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class EmailBlade extends Mailable
{
    use Queueable, SerializesModels;

    public $data;
    protected $template;

    /**
     * Create a new message instance.
     */
    public function __construct($data, $template)
    {
        $this->data = $data;
        $this->template = $template;
    }

    /**
     * Build the message.
     */
    public function build()
    {
        return $this->markdown('emails.' . $this->template)
                    ->with('data', $this->data)
                    ->subject('Funciona cabron');
    }
}




  if ($error == 0) {

  

                $sql_admins = DB::table('db_roles')

                    ->select(

                        'db_roles.id',

                        'db_user_roles.user_id as user_id',

                        'db_roles.name as user_role',

                        'db_users.email as email'

  

                    )

                    ->whereNull('db_roles.deleted_at')

                    ->join('db_user_roles', 'db_user_roles.role_id', '=', 'db_roles.id')

                    ->whereNull('db_user_roles.deleted_at')

                    ->join('db_users', 'db_users.id', '=', 'db_user_roles.user_id')

                    ->where('db_roles.name', 'like', '%admin%')

                    ->where('db_roles.company_id', '=', $company_id)

                    ->get();

  

                //variable capture and check if it exists on data base

                if ($request->input('textArea') != '') {

  

                    $message_point = $request->input('textArea');

                }

  

                if ($request->hasFile('file')) {

                    $file_point = $request->input('file');

                }

  

                $action_id = $request->input('action_id');

                $action_exists = Action::where('db_actions.id', $action_id)->where('db_actions.company_id', $company_id)->first();

                if (!$action_exists) {

                    $error = 1;

                    $message_validator = $message_validator->merge(['action' => ['The action doesn´t exists']]);

                }

                if($error == 0){

                    /*

                    $askPointInfo = array();

                    $askPointInfo['action_id'] = $action_exists->id;

                    $askPointInfo['action_name'] = $action_exists->name;

                    $askPointInfo['user_id'] = $user_id;

                    $askPointInfo['user_name'] = $user-> name;

  

                    if($message_point != null){

                        $askPointInfo['message']= $message_point;

                    }else{$askPointInfo['message'] = null;}

                    $arrayEmail['user'][]= $askPointInfo;

                    */

                    $askData = [

                        'action_id' => $action_exists->id,

                        'action_name' => $action_exists->name,

                        'user_id' => $user_id,

                        'user_name' => $user->name,

                    ];

                    if ($message_point != null) {

                        $askPointInfo['message'] = $message_point;

                    } else {

                        $askData['message'] = null;

                    }

                    foreach ($sql_admins as $admin) {

                        $admin_email = $admin->email;

                        Mail::to($admin_email)->send(new EmailBlade($askData));

                    }

  

                    /*

                       foreach($sql_admins as $admin)

                    {

                        $admin_email = $admin->email;

                        Notification::route('mail', $admin_email)->notify(new ApiCalled($arrayEmail));

                    }

                    try {

                        $emailContent = json_encode($arrayEmail, JSON_PRETTY_PRINT);

                        Mail::raw($emailContent, function ($message) {

                            $message->to('abused1988@gmail.com')->subject('Subject of the email');

                        });

                        return 'Email sent successfully!';

                    } catch (\Exception $e) {

                        Log::error('Email sending failed: ' . $e->getMessage());

                    }

                        */

                }

            }

@component('mail::message')

New request:

  

User ID: {{ $data['user_id'] }}<br>

User Name: {{ $data['user_name'] }}<br>

Action ID: {{ $data['action_id'] }}<br>

Action Name: {{ $data['action_name'] }}<br>

  

@if(isset($userData['message']))

Message: {{ $userData['message'] }}<br>

@endif

  
  

@component('mail::button', ['url' => url('/')])

Link to template accept

@endcomponent

@component('mail::button', ['url' => url('/')])

Link to reject template

@endcomponent

  
  

{{ config('app.name') }}

@endcomponent


@component('mail::message')

New request:

  

User ID: {{ $data['user_id'] }}<br>

User Name: {{ $data['user_name'] }}<br>

Action ID: {{ $data['action_id'] }}<br>

Action Name: {{ $data['action_name'] }}<br>

  

@if(isset($userData['message']))

Message: {{ $userData['message'] }}<br>

@endif

  
  

@component('mail::button', ['url' => url('/')])

Link to template accept

@endcomponent

@component('mail::button', ['url' => url('/')])

Link to reject template

@endcomponent

  
  

{{ config('app.name') }}

@endcomponent


php artisan make:mail OrderShipped --markdown=emails.orders.shipped