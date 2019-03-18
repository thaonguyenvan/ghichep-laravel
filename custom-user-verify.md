# Custom User Email Verification / Activation Laravel

### Creating New Modal and Migration for VerifyUser

We need to create a new table in our database which is holding the verification code. This code is sent to the user email for account activation. Let go with the given Laravel Artisan command:

`php artisan make:modal VerifyUser -m`

You can run this command in command prompt, terminal, git bash or any of the IDE which provides the CLI interface. After running this command, generate a modal `VerifyUser.php` in the App folder. This command also create a migration file `verify_user` for `VerifyUser.php` modal under database/migration folder because we are using `-m` option.

### Update the Migration file and Migrate Table

Now its time to update the migration file and migrate table. The migration file is available in `database/migrations` folder and file name you get in the console when you create the modal command. Let replace the default code with the given code.

```
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateVerifyUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('verify_users', function (Blueprint $table) {
            $table->integer('user_id');
            $table->string('token');
            $table->timestamps();
        });
    }
    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('verify_users');
    }
}
```

Not that `user_id` will be the foreign key of the primary key from the user’s table. Also, modify the migration file for the user’s table to include a new boolean field to maintain the status of account verification.

`$table->boolean('verified')->default(false);`

Now the time to refresh the DB using migrate:refresh command.

`php artisan migrate:refresh`

Use of the above command, All the present table’s rollbacked and recreated. Please note that use of this caused lost all the present data.

With this, `verify_userstable` created in your database with following structure.

<img src="https://i.imgur.com/SMH0F53.png">

and the `users` table is modified, and having verified field. Please check the given screenshot

<img src="https://i.imgur.com/TbXRb7S.png">

### Define Relations in Modal

Now, our tables are ready. Let’s now go ahead and specify the one-to-one relationship between `User` and `VerifyUser` Model.

Add following method to the `User.php` Model class. Because Laravel already provides the default `User Modal`.

```
public function verifyUser()
{
  return $this->hasOne('App\VerifyUser');
}
```

Update following class to the `VerifyUser.php` Model class. Because we already create this modal when we create the migration.

```
class VerifyUser extends Model
{
    protected $guarded = [];

    public function user()
    {
        return $this->belongsTo('App\User', 'user_id');
    }
}
```

### Send Verification Email to Registered User Email

To send email’s make sure you have your mail properties set up in `.env` file of your project. Laravel provides the default configuration for Mailtrap SMTP. You need to add your Mailtrap API credentials in the .env file.

In Laravel, all the emails sent by the Mailable class. To generate one such class for our verify user’s email on registration. Let’s generate a class with artisan command. Run the given below command in your terminal.

`php artisan make:mail VerifyMail`

After above command is executed, A new Class named `VerifyMail.php` will be generated under `App/Mail`. This class will have a build method which defines which view file to send an email.

```
<?php
namespace App\Mail;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;
class VerifyMail extends Mailable
{
  use Queueable, SerializesModels;

  public $user;

  /**
  * Create a new message instance.
  *
  * @return void
  */
  public function __construct($user)
  {
    $this->user = $user;
  }
  /**
  * Build the message.
  *
  * @return $this
  */
  public function build()
  {
    return $this->view('emails.verifyUser');
  }
}
```

We will create a emails folder under `resources/views` in which we will keep our email views. Also as you can see we have declared the `$user` as public and we are setting the variable in the constructor. Variables are declared as a public is by default available in the view file. Thus you can directly refer to the variable to get the user-related data.

Let’s go ahead and create the view file to send an email. Create a new file  `verifyUser.blade.php` under `resources/views/emails`and add following contents into it.

```
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome Email</title>
  </head>
  <body>
    <h2>Welcome to the site {{$user['name']}}</h2>
    <br/>
    Your registered email-id is {{$user['email']}} , Please click on the below link to verify your email account
    <br/>
    <a href="{{url('user/verify', $user->verifyUser->token)}}">Verify Email</a>
  </body>
</html>
```

Now since we have the necessary code ready to send the verification email to the user. Let’s modify our `RegistrationController` class to send emails. This class located under `App/Http/Controllers/Auth` and modify the create method as given below:

```
protected function create(array $data)
{
  $user = User::create([
    'name' => $data['name'],
    'email' => $data['email'],
    'password' => bcrypt($data['password']),
  ]);
  $verifyUser = VerifyUser::create([
    'user_id' => $user->id,
    'token' => sha1(time())
  ]);
  \Mail::to($user->email)->send(new VerifyMail($user));
  return $user;
}
```

We are generating a new random token using `sha1(time())` and storing it in the `verify_users` table against the `user_id`, After that, we use the Mail facade to send `VerifyMail` to the users.

<img src="https://i.imgur.com/baBq0CQ.png">

### Setup Route and Controller for Verify User Functionality

Let’s now functionality of user verification that is the code that will be executed when the user clicks on the link sent to his email account.

Create a route for verify token, It will be better if you keep it together with your Authentication routes.

`Route::get('/user/verify/{token}', 'Auth\RegisterController@verifyUser');`

Since the functionality is related to User Registration we will create a new method `verifyUser` in `RegisterController.php`

```
public function verifyUser($token)
{
  $verifyUser = VerifyUser::where('token', $token)->first();
  if(isset($verifyUser) ){
    $user = $verifyUser->user;
    if(!$user->verified) {
      $verifyUser->user->verified = 1;
      $verifyUser->user->save();
      $status = "Your e-mail is verified. You can now login.";
    } else {
      $status = "Your e-mail is already verified. You can now login.";
    }
  } else {
    return redirect('/login')->with('warning', "Sorry your email cannot be identified.");
  }
  return redirect('/login')->with('status', $status);
}
```

The verifyuser method accepts a token from the URL and it goes ahead and finds the user that is associated with that token. It confirms that the user exists and is not verified yet and then goes ahead and changes the verification status in the database. Various status and warning messages are viewed when redirecting the user to display in the view file.

### Restricting User Access for Un-Verified Users

Now, we have our email verification and user account activation process. And need one more important thing to be done before mark this complete. We should not be allowed the un-verified user’s to access the application pages until they are verified. Thus we need to apply checks at two places “after user Login” and “after new user Registration”.

```
public function authenticated(Request $request, $user)
{
  if (!$user->verified) {
    auth()->logout();
    return back()->with('warning', 'You need to confirm your account. We have sent you an activation code, please check your email.');
  }
  return redirect()->intended($this->redirectPath());
}
```

The authenticated method is executed just after the user is authenticated. We will override this and will use this to check if the user is activated. If not we will log him out and send back to login page with the warning message.

Modify RegisterController.php and override the registered method from RegistersUsers

```
protected function registered(Request $request, $user)
{
  $this->guard()->logout();
  return redirect('/login')->with('status', 'We sent you an activation code. Check your email and click on the link to verify.');
}
```

The registered method is executed just after the user is registered into the application, we will override and modify this to log him out and send him back to login with the status message.

Last we need to modify our `login.blade.php` file which is located under `resources/views/auth`. Add the following code to display the appropriate status messages.

```
@if (session('status'))
  <div class="alert alert-success">
    {{ session('status') }}
  </div>
@endif
@if (session('warning'))
  <div class="alert alert-warning">
    {{ session('warning') }}
  </div>
@endif
```

**Cre: **

https://codebriefly.com/custom-user-email-verification-activation-laravel/
