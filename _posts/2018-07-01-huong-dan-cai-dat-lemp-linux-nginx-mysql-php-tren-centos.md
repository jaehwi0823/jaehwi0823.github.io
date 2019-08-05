---
layout: article
title: Huớng dẫn cài đặt LEMP (Linux, Nginx, MySQL, PHP) trên Centos
tags: [lemp, nginx, mysql, php, centos]
---
## 1. Giới thiệu về LEMP
LEMP server là một server chạy Linux, Nginx (đọc là Engine x), MySql và PHP (hoặc Perl/Python). Nó tương tự như LAMP server ngoại trừ việc web server nền tảng được giám sát bằng Nginx thay vì Apache.

![Lemp](/assets/images/lemp-logos.gif)

Các thành phần cấu thành LEMP stack cũng gần tương tự với LAMP, chỉ khác là Apache sẽ được thay thế bởi nginx. Nginx được đọc là "engine-x", giải thích cho chữ E trong "LEPM", nginx cũng là một ứng dụng HTTP proxy nhưng không có được danh tiếng ấn tượng như Apache, tuy nhiên, nó có ưu điểm là cho phép xử lý tốc độ tải cao hơn đối với các HTTP request.

Nginx giờ đây, đã đạt được sự thu hút đáng kể đối với người dùng khi nó bắt đầu được nhiều người sử dụng từ năm 2008 và hiện trở thành ứng dụng web server tiếng tăm thứ 2 sau Apache khi đề cập các active site theo báo cáo của Netcraft.

### 1.1 Phân biệt LEMP và LAMP Stack
Như đã nói, khác biệt cơ bản giữa LAMP và LEMP stack là ở 2 thành phần Apache và Nginx. Vậy việc sử dụng nginx và Apache sẽ tạo ra những khác biệt gì? Chúng ta sẽ cùng so sánh riêng 2 phần mềm này để thấy được rõ hơn sự khác biệt:

**Apache:**

- Apache đã được sử dụng từ lâu (từ những năm 1995), có rất nhiều các module được viết và cả người dùng tham gia vào mở rộng hệ chức năng cho Apache.

- Phương pháp process/thread-oriented – sẽ bắt đầu chậm lại khi xuất hiện tải nặng, cần tạo ra các quy trình mới dẫn đến tiêu thụ nhiều RAM hơn, bên cạnh đó, cũng tạo ra các thread mới cạnh tranh các tài nguyên CPU và RAM;

- Giới hạn phải được thiết lập để đảm bảo rằng tài nguyên không bị quá tải, khi đạt đến giới hạn, các kết nối bổ sung sẽ bị từ chối;

- Yếu tố hạn chế trong điều chỉnh Apache: bộ nhớ và thế vị cho các dead-locked threads cạnh tranh cho cùng một CPU và bộ nhớ.

**Nginx:**
- Ứng dụng web server mã nguồn mở được viết để giải quyết các vấn đề về hiệu suất và khả năng mở rộng có liên quan đến Apache.

- Phương pháp Event-driven, không đồng bộ và không bị chặn, không tạo các process mới cho mỗi request từ web.

- Đặt số lượng cho các worker process và mỗi worker có thể xử lý hàng nghìn kết nối đồng thời

- Các module sẽ được chèn vào trong thời gian biên dịch, có trình biên dịch mã PHP bên trong (không cần đến module PHP).

## 2. Cài đặt LEMP trên Centos 7/6.5/5.10
### 2.1. Cài đặt Nginx và PHP trên CentOS 7/6.5/5.10
Đầu tiên bạn cần chuẩn bị một server CentOS mới tinh chưa cài gì cả. Kiểm tra lại xem hostname và file host đã chính xác chưa trước khi bắt đầu.

**Bước 1: Thêm repo cần thiết**

**CentOS 7/6.5/5.10 EPEL repository**
```
yum install epel-release
```
**CentOS 7/6.5/5.10 Remi repository**
```
## CentOS 7 ##
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

## CentOS 6 ##
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

## CentOS 5 ##
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-5.rpm
```
**CentOS 7/6.5/5.10 Nginx repository**
```
## CentOS 7 ##
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

## CentOS 6 ##
rpm -Uvh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm

## CentOS 5 ##
rpm -Uvh http://nginx.org/packages/centos/5/noarch/RPMS/nginx-release-centos-5-0.el5.ngx.noarch.rpm
```

**Bước 2: Cài đặt Nginx, PHP**

**CentOS 7/6.5/5.10**

```
## PHP 5.3 ##
yum install -y nginx php-fpm php-common

## PHP 5.4 ##
yum --enablerepo=remi install -y nginx php-fpm php-common

## PHP 5.5 ##
yum --enablerepo=remi,remi-php55 install -y nginx php-fpm php-common

## PHP 5.6 ##
yum --enablerepo=remi,remi-php56 install -y nginx php-fpm php-common

## PHP 7.0 ##
yum --enablerepo=remi,remi-php70 install -y nginx php-fpm php-common

## PHP 7.1 ##
yum --enablerepo=remi,remi-php71 install -y nginx php-fpm php-common
```

**Bước 1: Cài đặt PHP module**

Một số module PHP thông dụng:
- OPcache (php-opcache) – The Zend OPcache provides faster PHP execution through opcode caching and optimization.
- APCu (php-pecl-apc) – APCu userland caching
- CLI (php-cli) – Command-line interface for PHP
- PEAR (php-pear) – PHP Extension and Application Repository framework
- PDO (php-pdo) – A database access abstraction module for PHP applications
- MySQL (php-mysqlnd) – A module for PHP applications that use MySQL databases
P- ostgreSQL (php-pgsql) – A PostgreSQL database module for PHP
- MongoDB (php-pecl-mongo) – PHP MongoDB database driver
- SQLite (php-pecl-sqlite) – Extension for the SQLite Embeddable SQL Database Engine
- Memcache (php-pecl-memcache) – Extension to work with the - Memcached caching daemon
- Memcached (php-pecl-memcached) – Extension to work with the Memcached caching daemon
- GD (php-gd) – A module for PHP applications for using the gd graphics library
- XML (php-xml) – A module for PHP applications which use XML
- MBString (php-mbstring) – A module for PHP applications which need multi-byte string handling
- MCrypt (php-mcrypt) – Standard PHP module provides mcrypt library support

Để cài đặt bạn hãy sử dụng lệnh ```yum --enablerepo=remi,remi-php56 install ten_module```. Ví dụ:

```
yum --enablerepo=remi,remi-php56 install -y php-opcache php-pecl-apcu php-cli php-pear php-pdo php-mysqlnd php-pgsql php-pecl-mongo php-pecl-sqlite php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml
```

**Bước 4: Stop httpd (Apache) server, Start Nginx và PHP-FPM**

**Stop httpd (Apache)**
```
## CentOS 7 ##
systemctl stop httpd.service

## CentOS 6.5/5.10 ##
service httpd stop
```
**Start Nginx**
```
## CentOS 7 ##
systemctl start nginx.service
 
## CentOS 6.5/5.10 ##
service nginx start
```
**Start PHP-FPM**
```
## CentOS 7 ##
systemctl start php-fpm.service

## CentOS 6.5/5.10 ##
service php-fpm start
```
**Bước 5: Tự động khởi động Nginx, PHP-FPM và tắt httpd**

**Tắt httpd (Apache) khi boot**
```
## CentOS 7 ##
systemctl disable httpd.service
 
## CentOS 6.5/5.10 ##
chkconfig httpd off
```
**Autostart Nginx**
```
## CentOS 7 ##
systemctl enable nginx.service
 
## CentOS 6.5/5.10 ##
chkconfig --add nginx
chkconfig --levels 235 nginx on
```
**Autostart PHP-FPM**
```
## CentOS 7 ##
systemctl enable php-fpm.service
 
## CentOS 6.5/5.10 ##
chkconfig --add php-fpm
chkconfig --levels 235 php-fpm on
```
**Bước 6: Cấu hình Nginx và PHP-FPM**

**Cấu hình Nginx**
– Thay đổi worker_processes
```
nano /etc/nginx/nginx.conf
```

Chỉnh worker_processes bằng với số processor VPS của bạn
- Cấu hình nginx virtual hosts
```
nano /etc/nginx/conf.d/default.conf
```
Bạn thay đổi thông tin như bên dưới:
```
#
# The default server
#
server {
    listen       80 default_server;
    server_name example.com;

    location / {
        root   /usr/share/nginx/html;
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }

    error_page  404              /404.html;
    location = /404.html {
        root   /usr/share/nginx/html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
Các đoạn bôi đỏ là cần phải thay đổi.
- Restart Nginx

```
## CentOS 7 ##
systemctl restart nginx.service
 
## CentOS 6.5/5.10 ##
service nginx restart
```

**Cấu hình PHP-FPM**

- Chỉnh user và group

```
nano /etc/php-fpm.d/www.conf
```
Thay user và group = apache sang nginx
```
[...]
 ; Unix user/group of processes
 ; Note: The user is mandatory. If the group is not set, the default user's group
 ; will be used.
 ; RPM: apache Choosed to be able to access some dir as httpd
 user = nginx
 ; RPM: Keep a group allowed to write in log dir.
 group = nginx
 [...]
 ```
 - Restart PHP-FPM

```
 ## CentOS 7 ##
systemctl restart php-fpm.service

## CentOS 6.5/5.10 ##
service php-fpm restart
```
Bước 7: Test cấu hình Nginx và PHP-FPM
```
nano /usr/share/nginx/html/info.php
```
Thêm đoạn sau vào
```
<?php
phpinfo();
?>
```
Test thử bằng link: http://<ip-address>/info.php. Nếu bạn thấy thông tin về PHP hiện ra thì đã cài đặt thành công.

**Lưu ý**: nếu bạn truy cập thẳng vào IP mà báo lỗi không kết nối được thì hãy open port http:
```
service iptables start
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
service iptables save
```

### 2.2. Cài đặt MariaDB trên CentOS 7/6.5/5.10
**Bước 1: Thêm MariaDB repo**

```
## CentOS 6/5 MariaDB 5.5 ##
wget -O /etc/yum.repos.d/MariaDB.repo http://mariadb.if-not-true-then-false.com/centos/$(rpm -E %centos)/$(uname -i)/5

## CentOS 6/5 MariaDB 10.0 ##
wget -O /etc/yum.repos.d/MariaDB.repo http://mariadb.if-not-true-then-false.com/centos/$(rpm -E %centos)/$(uname -i)/10
```
**Bước 2: Cài đặt hoặc update MariaDB**
```
## CentOS 7 ##
yum install -y mariadb mariadb-server

## CentOS 6.5/5.10 ##
yum install -y MariaDB MariaDB-server
```
**Bước 3: Khởi động MariaDB và tự động chạy khi boot**
```
## CentOS 7 ##
systemctl start mariadb.service
systemctl enable mariadb.service

## CentOS 6.5/5.10 ##
service mysql start
chkconfig --levels 235 mysql on
```
**Bước 3: Cấu hình MariaDB**
- Set (Change) root password
- Remove anonymous users
- Disallow root login remotely
- Remove test database and access to it
- Reload privilege tables

– Bắt đầu cài đặt
```
/usr/bin/mysql_secure_installation
```
Ngay bước đầu tiên bạn sẽ bị hỏi root password, do mới cài đặt nên tất nhiên chưa có password, nhấn Enter để tiếp tục.
– Output tương tự như sau:
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we\'ll need the current
password for the root user. If you\'ve just installed MariaDB, and
you haven\'t set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
... Success!

Normally, root should only be allowed to connect from \'localhost\'. This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MariaDB comes with a database named \'test\' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
- Dropping test database...
... Success!
- Removing privileges on test database...
... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
... Success!

Cleaning up...

All done! If you\'ve completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
Như vậy là bạn đã hoàn thành việc cài đặt LEMP stack trên CentOS rồi đó.  Chúc các bạn thành công !