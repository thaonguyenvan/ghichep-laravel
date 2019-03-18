# Tìm hiểu về schedule trong laravel

Schedule trong laravel có thể giải quyết được bài toán lập lịch để chạy một công việc hay một tác vụ nào đó.

Để định nghĩa mới schedule, chúng ta có nhiều cách, cách đơn giản nhất là viết thẳng câu lệnh vào phương thức `schedule` trong `App\Console\Kernel`

``` php
<?php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
use DB;
;
use Carbon\Carbon;

class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [

    ];

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->call(function () {
            DB::table('posts')->insert([
                'title' => 'Day la title bai viet',
                'content' => 'Day la noi dung bai viet',
                'publish' => 1
            ]);
        });
    }

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        require base_path('routes/console.php');
    }
}
```

- Một cách nữa thường được sử dụng hơn đó là ta sẽ tạo ra artisan command

`php artisan make:command PostCommand`

Sau khi chạy câu lệnh trên, ta sẽ có 1 file `app/Console/Commands/PostCommand.php`

Tại đây ta sẽ viết code xử lí trong phương thức `handle`

``` php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use DB;

class PostCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'post:create';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Create new posst';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        DB::table('posts')->insert([
            'title' => 'Nguyen Thi ha',
            'content' => 'Day la noi dung bai viet',
            'publish' => 1
        ]);
    }
}
```

- Tiếp theo ta sẽ quay trở lại `app/Console/Kernel.php` để đăng kí command

```
<?php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
use DB;

class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [
        'App\Console\Commands\PostCommand'
    ];

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('post:create')->everyMinute();
    }

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        require base_path('routes/console.php');
    }
}
```

Như vậy ta có thể thấy cách khai báo schedule trong laravel

`everyMinute()`

Đây là từng phút, ngoài ra ta sẽ còn khá nhiều phương thức khác, tìm hiểu thêm [tại đây](https://laravel.com/docs/5.7/scheduling)

Ok, bây giờ ta sẽ chạy thử, ví dụ trên sẽ insert vào db 1 bảng tin mỗi 1 phút, hãy đảm bảo rằng bạn đã có sẵn db trước khi thực hiện, ở đây mình sẽ chỉ hướng dẫn cách đặt lịch.

Để chạy command, ta sẽ `cd` vào thư mục project và chạy thử câu lệnh

`php artisan list`

Một list các câu lệnh sẽ hiện ra và trong đó sẽ có câu lệnh mà ta vừa khai báo ở trên

<img src="https://i.imgur.com/YUrVK8W.png">

Ta chạy thử câu lệnh này bằng lệnh sau:

`php artisan post:create`

Nếu câu lệnh chạy ok, ko lỗi thì ta có thể bắt đầu đặt lịch vào crontab với cú pháp như sau:

`* * * * * php /var/www/html/spdao/artisan schedule:run >> /dev/null 2>&1`

Cron này sẽ gọi tới command scheduler mỗi phút 1 lần. Khi mà `schedule:run` được thực thi, Laravel sẽ tính toán các task và thực hiện.

- Mặc định thì các scheduled task sẽ được run kể cả khi những task trước nó vẫn đang chạy. Để tránh điểu này, ta có thể sử dụng phương thức `withoutOverlapping`

`$schedule->command('emails:send')->withoutOverlapping();`

Nó sẽ hữu ích đối với những task mà bạn không biết chính xác bh thì nó hoàn thành.

- Ta có thể cấu hình để schedule gửi output ra file bằng phương thức

``` php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
```

hoặc thậm chí là gửi mail

``` php
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com');
```
