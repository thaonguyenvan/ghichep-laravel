# Hướng dẫn cài đặt laravel trên CentOS 7

## Mục lục

1. Yêu cầu

2. Chuẩn bị môi trường cài đặt

3. Cài đặt composer và laravel

--------------

###  1. Yêu cầu

- PHP >= 7.1.3 with OpenSSL, PDO, Mbstring, Tokenizer, XML, Ctype and JSON PHP Extensions.
- Composer – an application-level package manager for the PHP.

### 2. Chuẩn bị môi trường cài đặt

Để cài đặt Laravel trên CentOS 7, ta sẽ cần môi trường web server. Bạn có thể sử dụng LEMP(Nginx, PHP, MySQL) hoặc LAMP(Apache,PHP,MySQL).

Ở hướng dẫn này, mình sẽ sử dụng LAMP. Vậy nên trước tiên, ta sẽ cài LAMP.

- Cài đặt remi và epel repo

```
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

- Cài đặt apache

`yum install httpd -y`

Khởi động apache

```
systemctl start httpd
systemctl enable httpd
```

Thêm rule cho firewalld

```
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```

- Cài đặt MySQL

```
yum install mariadb-server php-mysql
systemctl start mariadb.service
/usr/bin/mysql_secure_installation
```

- Cài đặt php 7.2


```
yum install yum-utils
yum-config-manager --enable remi-php72
yum install php php-fpm php-common php-xml php-mbstring php-json php-zip
```

### 3. Cài đặt composer và laravel

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/bin/composer
chmod +x /usr/bin/composer
```

Sau khi cài đặt xong composer, ta tiến hành tạo project mới

```
cd /var/www/html/
sudo composer create-project --prefer-dist laravel/laravel testsite
```

Trong đó `testsite` là tên project muốn tạo.

Tiếp thao, ta sẽ thay đổi cấu hình thư mục root (vì file index.php của laravel đặt tại thư mục public)

`vi /etc/httpd/conf/httpd.conf`

`DocumentRoot "/var/www/html/testsite/public"`

Khởi động lại httpd

`systemctl restart httpd`

Thay đổi permission

```
chown -R apache:apache /var/www/html/testsite/
chmod -R 755 /var/www/html/testsite/storage
```

**Lưu ý:**

Nếu bạn gặp phải lỗi 404 not found khi truy cập tới các route của website, thêm đoạn cấu hình sau vào file cấu hình httpd để enable `.htacess`

```
<Directory /var/www/html/testsite/public>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

Sau đó restart lại httpd

**Tham khảo:**

https://www.tecmint.com/install-laravel-in-centos/

https://www.hugeserver.com/kb/install-laravel5-php7-apache-centos7/
