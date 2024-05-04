# Hướng dẫn cài đặt GPM Private Server trên Ubuntu 22.04
Link video hướng dẫn: [https://youtu.be/z1xqG31W6gs](https://youtu.be/z1xqG31W6gs)

# Các câu lệnh
## Bước 1: Cài các gói cần thiết
```
> sudo apt-get install php libapache2-mod-php php-dev php-zip php-curl php-pear php-mbstring php-mysql php-gd php-xml curl -y
> sudo apt install apache2
> sudo systemctl start apache2
> sudo systemctl enable apache2
> sudo apt install mariadb-server -y
> sudo apt install unzip
> curl -sS https://getcomposer.org/installer | php
> sudo mv composer.phar /usr/local/bin/composer
> sudo chmod +x /usr/local/bin/composer
```
### Sau bước 1 kiểm tra:
- apache hoạt động hay chưa bằng cách truy cập IP của máy Ubuntu
- phiên bản php phải là 8.1, kiểm tra bằng câu lệnh: `php -v`

## Bước 2: Tải code private server lên thư mục /var/www/html/GPM_PrivateServer và cấu hình
```
> sudo mkdir /var/www/html/GPM_PrivateServer
> sudo chmod 777 /var/www/html/GPM_PrivateServer
```
Tiến hành Upload file code lên thư mục /var/www/html/GPM_PrivateServer (xem thêm tại video)
```
> cd /var/www/html/GPM_PrivateServer
> unzip full_v12-2023.zip
> php artisan
> php artisan storage:link
```
**Chú ý:** Trong trường hợp chuyển private server từ host khác về máy này thì cần sao lưu thư mục /public/storage/profiles trước sau đó đó xóa đi bằng lệnh `rm -rf public/storage/` rồi chạy lại lệnh `php artisan storage:link`

## Bước 3: Tạo cơ sở dữ liệu
```
> sudo mysql -u root -p
> create database IF NOT EXISTS `gpmlogin_db` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
> ALTER USER 'root'@'localhost' IDENTIFIED BY '@Abc12345678@';
> FLUSH PRIVILEGES;
> exit
> mysql -u root -p gpmlogin_db < gpmlogin.sql

> sudo chmod 755 /var/www/html/GPM_PrivateServer
> sudo chown -R www-data:www-data /var/www/html/GPM_PrivateServer
> sudo find /var/www/html/GPM_PrivateServer -type d -exec chmod 755 {} \;
> sudo find /var/www/html/GPM_PrivateServer -type f -exec chmod 644 {} \;
```
**Chú ý:** Tên cơ sở dữ liệu là *gpmlogin_db*

## Bước 4: Sửa file .env *(chú ý trong trình soạn thảo nano ấn Ctrl+X rồi ấn y để lưu, Ctrl+W để tìm kiếm)*
```
> sudo nano /var/www/html/GPM_PrivateServer/.env
```
Sửa: 
- Tên cơ sử dỡ liệu
- Tài khoản, mật khẩu truy cập mysql
- Thông tin S3 hoặc Digital Ocean (nếu lưu profile ở S3, DO)

## Bước 5: Cấu hình apache, php
```
> sudo nano /etc/apache2/sites-available/000-default.conf
```
Rồi đổi site mặc định từ cổng 80 sang cổng 81
```
> sudo nano /etc/php/8.1/apache2/php.ini
```
Rồi nâng **post_max_size** và **upload_max_filesize** trong file php.ini.
Tiếp theo tạo file cấu hình apache
```
> sudo nano /etc/apache2/sites-available/GPM_PrivateServer.conf
```
Nhập nội dung dưới đây vào file (**chú ý** tệp cấu hình sử dụng cổng 80)
```
<VirtualHost *:80>
     ServerAdmin admin@admin.com
     DocumentRoot /var/www/html/GPM_PrivateServer/public
     # ServerName abc.com

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

     <Directory /var/www/html/GPM_PrivateServer>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>
</VirtualHost>
```
Tiếp tục cấu hình apache
```
> sudo a2ensite GPM_PrivateServer.conf
> sudo systemctl reload apache2
> sudo a2enmod rewrite
> sudo systemctl reload apache2
> sudo systemctl restart apache2
```
Đến đây là xong rồi!

## Một số nội dung bổ sung
### Cấu hình thêm (tùy chọn)
Trường hợp sử dụng tường lửa cần mở cho apache đi qua
```
> sudo ufw allow 'Apache'
```

Nếu cầu mở thêm cổng cho apache thì cấu hình thêm tại:
```
> sudo nano /etc/apache2/ports.conf
```


### Xử lý lỗi
- Could not get lock /var/lib/dpkg/lock. It is held by process xxxx (unattended-upgr)
```
> sudo systemctl stop unattended-upgrades
> sudo rm /var/lib/dpkg/lock-frontend
> sudo rm /var/lib/apt/lists/lock
> sudo rm /var/lib/dpkg/lock
> sudo rm /var/cache/apt/archives/lock
```
