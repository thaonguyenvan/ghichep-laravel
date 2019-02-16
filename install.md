# Hướng dẫn cài đặt laravel trên Windows 10

## Mục lục  

1. Yêu cầu

2. Cài đặt composer

3. Cài đặt laravel và tạo project mới

------------------

### 1. Yêu cầu

Để có thể cài đặt được laravel, có một số yêu cầu sau:

- PHP >= 7.1.3
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
- Ctype PHP Extension
- JSON PHP Extension
- BCMath PHP Extension
- Composer

Trên Windows, bạn có thể cài đặt một số công cụ để tạo web server, ở đây mình sẽ sử dụng xampp

Sau khi cài đặt xampp, bạn có thể xem version của php, lưu ý, ở trên có yêu cầu php >= 7.1.3

<img src="https://i.imgur.com/dMLOjFT.png">

Nếu phiên bản xampp hiện tại có version php thấp hơn, bạn cần update nó trước khi cài laravel.

### 2. Cài đặt composer

Để cài đặt composer, bạn truy cập vào link sau rồi tải về và cài đặt như bình thường

https://getcomposer.org/

Sau khi cài đặt xong, để kiểm tra, mở cmd lên và gõ "composer". Kết quả như sau là thành công

<img src="https://i.imgur.com/noetT8R.png">

### 3. Cài đặt laravel và tạo project mới

Mở cmd, gõ lệnh sau để cài laravel

`composer global require laravel/installer`

Chờ cho quá trình cài đặt hoàn tất, sau đó ta có thể vào thư mục htdocs của xampp để tiến hành khởi tạo project với laravel

Để khởi tạo project mới, chạy cmd sau trong thư mục htdocs

`laravel new blog`

hoặc

`composer create-project --prefer-dist laravel/laravel blog`

Trong đó `blog` chính là tên project mà ta muốn tạo.

Chờ cho đến khi quá trình khởi tạo project hoàn tất, ta truy cập vào localhost theo tên project vừa khởi tạo kèm theo url "/public". Kết quả như sau là ok

<img src="https://i.imgur.com/e4FtA7U.png">

**Tham khảo**

https://laravel.com/docs/5.7/installation
