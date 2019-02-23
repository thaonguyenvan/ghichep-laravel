# Hướng dẫn gửi mail với mailable

- Thay đổi thông thin trong file `env`

```
MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=thaodeptrai@gmail.com
MAIL_PASSWORD=*******
MAIL_ENCRYPTION=tls
```

- Ta có thể tạo một Controller có tên `SendEmailController`, tại đây ta sẽ import thư viện mailable

`php artisan make:controller SendEmailController`

``` php
use Illuminate\Support\Facades\Mail;
```

Cũng tại Controller này, ta có thể gửi mail thông qua câu lệnh

``` php
$users = User::all();
        foreach ($users as $user) {
            Mail::raw("this is test mail", function ($mail) use ($user) {
                $mail->from('thaodeptrai@gmail.com');
                $mail->to($user->email)
                    ->subject('Word of the Day');
            });
        }
```

- Ngoài ra để gửi mail với nội dung là một view được thiết kế sẵn, ta sẽ tạo thêm 1 class là sendmail bằng câu lệnh sau

`php artisan make:mail SendMail`

Sau khi khởi tạo, ta sẽ có 1 file SendMail.php nằm trong `App\Mail\SendMail.php `

Tại đây ta sẽ define một biến `$data`, với biến `$data` này, ta có thể truyền thêm thông số ở thời điểm khởi tạo cho view và gửi tới người dùng

``` php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendMail extends Mailable
{
    use Queueable, SerializesModels;

    public $data;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct($data)
    {
        $this->data = $data;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('thaonvdeptrai@gmail.com','thaodeptrai')->subject('Test mail')->view('email-template')->with('data', $this->data);

    }
}
```

Ok, mọi người cũng có thể thấy ta có 1 method là view() để có thể truyền vào email một template có sẵn. Giờ ta sẽ tạo 1 file view với cái tên đã được define đó là `email-template.blade.php`. File này sẽ được viết dưới dạng của blade template. Vì view này được truyền 1 biến là `$data` nên ta có thể sử dụng nó để hiển thị ra ngoài như sau.

```
<p>This is test email {{$data}}</p>
```

- Tiếp theo ta sẽ viết hàm xử lí để gửi mail, tại đây tôi sẽ viết nó tại SendEmailController

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;
use App\Mail\SendMail;

class SendEmailController extends Controller
{
    function send(Request $request)
    {
     $this->validate($request, [
      'name'     =>  'required',
      'email'  =>  'required|email',
      'message' =>  'required'
     ]);

        $data = array(
            'name'      =>  $request->name,
            'message'   =>   $request->message
        );

     Mail::to('thaodeptrai2@gmail.com')->send(new SendMail($data));
     return back()->with('success', 'Thanks for contacting us!');
    }
}

?>
```

Như ta thấy, nhiệm vụ của function này sẽ là get lấy dữ liệu, gán nó cho `$data` rồi gọi tới object SendMail để gửi mail.


**Tham khảo:**

https://www.webslesson.info/2018/09/simple-way-to-sending-an-email-in-laravel.html

https://kipalog.com/posts/Send-mail-with-Laravel--gmail
