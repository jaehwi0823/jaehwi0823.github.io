---
layout: article
title: Cloud Computing - Triển khai 1 dịch vụ cơ sở hạ tầng (Vultr) & triển khai ứng dụng Web
tags: [cloud computing, vultr, lemp]
---
Trong bài này tôi sẽ trình bày về sự hiểu biết của mình về **(Cloud Computing)** . 

Cụ thể hơn trong bài viết này tôi sẽ tìm hiểu về dịch vụ cho thuê cơ sở hạ tầng của Vultr (Infrastructure Cloud) & Cài đặt cái gói (LEMP Stack) để triển khai 1 ứng dụng website chạy trên VPS (Virtual Private Server).

## 1. Các yêu cầu cần thiết
1. Một tài khoản Vultr đã kích hoạt thanh toán.
2. Công cụ:
  - Webserver: Nginx
  - Terminal/Putty
  - FTP, FileZilla Client
  - Git
  - IDE: Intelli IDEA hoặc Visual Studio Code
3. Ngôn ngữ lập trình: PHP, Node.JS
4. Database: Mariadb (MySQL), MongoDb
5. Hệ điều hành: Centos 7

## 2. Giới thiệu về Vultr
Vultr là nhà cung cấp dịch vụ VPS Server, Cloud Server với 100% phần cứng SSD, 16 datacenter location. Ưu điểm của Vultr là giá rẻ, hiệu năng cao và cài đặt dễ dàng, nhanh chóng.

### 2.1. Tổng quan về Vultr
![Vultr Logo](/assets/images/vultr-logo.png)

Vultr, được thành lập vào năm 2014, đang thực hiện sứ mệnh trao quyền cho các nhà phát triển và doanh nghiệp bằng cách đơn giản hóa việc triển khai cơ sở hạ tầng thông qua nền tảng đám mây tiên tiến.

Vultr có vị trí chiến lược trong 16 trung tâm dữ liệu trên toàn cầu và cung cấp việc cung cấp dịch vụ đám mây: public cloud, storage, ...

**Sự khác biệt**
- **Geographic Footprint**: Vultr có 16 Datacenter tại 16 thành phố giải rác trên thế giới. Điều này sẽ cung cấp các dịch vụ đám mây gần bạn nhất.
- **One Click Apps**: Nhanh chóng khởi tạo các ứng dụng dễ dàng, đơn giản chỉ bằng một cú nhấp chuột. Hỗ trợ các ứng dụng như: WordPress, Prestashop, Magento, Joomla, Docker, Gitlab, Drupal, Cpanel (có tính phí bản quyền), LAMP, LEMP, Mediawiki, Minecraff, NextCloud, OpenLiteSpeed, OpenVPN, ownCloud, Plesk Onyx, Webmin.
- **Full Resource Control**: Được cung cấp tài khoản root/administrator để quản lý resource.
- **Upload ISO / Mount ISO**: Tạo các tùy chọn OS, hầu như không giới hạn bằng cách tải lên file ISO và cài đặt OS tùy chỉnh đó.
- **Linux, Windows and BSD**: Vultr thực sự hỗ trợ một loạt các bản phân phối Linux, Windows và BSD phổ biến.
- **No Long Term Contracts**: Thanh toán bằng giờ sử dụng.

### 2.2. Công nghệ của Vultr
#### 2.2.1. Công nghệ ảo hóa
Vultr dùng KVM ( Kernel-based Virtual Machine ) trên 100% cơ sở hạ tầng.

{:.info}
KVM (viết tắt của Kernel Virtualization Machine) là công nghệ ảo hóa phần cứng. Có nghĩa là OS (hệ điều hành) chính mô phỏng phần cứng cho các OS khác để chạy trên đó. Nó hoạt động tương tự như một người quản lý siêu việt chia sẻ công bằng các tài nguyên như disk (ổ đĩa), network IO và CPU.

KVM VPs không có tài nguyên dùng chung, tất cả đã được mặc định sẵn và chia sẻ. Điều đó có nghĩa là RAM của mỗi VPs KVM đã được phân bổ sẵn cho từng gói VPs để sở hữu, tận dụng hoàn toàn 100% và không bị chia sẻ ra ngoài. Đồng nghĩa với việc VPs KVM sẽ hoạt động ổn định hơn mà không bị ảnh hưởng bởi các VPs khác cùng hoạt động trên hệ thống. Tương tự điều đó với disk space, tài nguyên ổ cứng cũng đã được xác định sẵn và không bị vay mượn bởi các VPs khác.

### 2.3. Các dịch vụ
Vultr cung cấp 4 dịch vụ chính:
- **Cloud Compute**: cung cấp dịch vụ thuê máy chủ ảo sử dụng CPU Intel và 100% máy chủ sử dụng SSD.
- **Dedicated Cloud**:  Dedicated server là một loại lưu trữ trên internet mà người sử dụng có thể thuê toàn bộ một máy chủ, không hề chia sẻ với bất cứ ai.
- **Bare Metal**: là dịch vụ dedicated server. Nhưng được triển khai trên nền tảng cloud của Vultr nên thừa hưởng những tính năng như: thanh toán theo giờ, nâng cấp mở rộng, API và hỗ trợ nhiều location khác nhau.
- **Block Storage**: Lưu trữ nhanh và có thể lưu trữ được hỗ trợ bởi SSD với dung lượng lên tới 10TB.

### 2.4. Bảng giá dịch vụ
So với các gói cloud server ở [DigitalOcean](https://www.digitalocean.com/), Vultr có giá tương đương nhưng lúc nào cấu hình cũng vượt trội hơn một chút. Đặc biệt gói 3.5$/tháng được 512MB RAM, 10GB SSD, 0.5TB Bandwidth quá tuyệt vời, khó có thể tìm được nhà cung cấp nào đưa ra được mức giá tương tự.

Bảng giá các gói VPS hiện nay ở Vultr như sau:

- Đối với dịch vụ **SSD Cloud Instances**:

![SSD Cloud Instances Price](/assets/images/price-vultr.png)

- Đối với dịch vụ **High Frequency Compute**:

![High Frequency Compute Price](/assets/images/high-frequency-compute-price.png)

Cũng giống như **DigitalOcean**, Vultr tính tiền theo giờ sử dụng, dùng bao nhiêu tính tiền bấy nhiêu, do đó bạn sẽ không cần bận tâm khi muốn ngưng sử dụng dịch vụ bất kỳ lúc nào bạn muốn.

Mặc dù mới tham gia vào thị trường cloud server, sau **DigitalOcean** nhưng Vultr đang ngày càng tỏ ra vượt trội hơn với mức giá thành hợp lý, cấu hình cao, hiệu năng tốt, hoạt động ổn định.

## 3. Triển khai dịch vụ
Vultr có trang dashboard để quản lý cũng như giám sát tài nguyên sử dụng các dịch vụ đang sử dụng.

### 3.1. Triển khai mới dịch vụ
Để triển khai 1 dịch vụ trên Vultr truy cập vào tab deloy (dấu +) hoặc tại [đây](https://my.vultr.com/deploy/).

Tiến hành chọn dịch vụ sử dụng

![Service Vultr](/assets/images/select-service-vultr.png)

Tiếp theo chọn **Server Location**

![Server Location Vultr](/assets/images/select-server-location-vultr.png)

Sau đó chọn **Server Type**. Trong phần **Server Type** sẽ cho phép chọn OS hoặc chọn Application.

![Server Type Vultr](/assets/images/server-type-vultr.png)

Tiếp theo sẽ chọn Server Size. Trong phần Server Size sẽ cho phép chọn các plan phù hợp với nhu cầu sử dụng.

![Server Size Vultr](/assets/images/server-size-vultr.png)

Cuối cùng thiết lập các **Additional Features**(Enable IPv6, Enable Private Networking), **Startup Script** , **SSH Keys**, **Server Hostname & Label** và nhấn **Deloy Now** để tạo mới 1 dịch vụ.

### 3.2. Nâng cấp dịch vụ
Trong trường hợp muốn nâng cấp về tài nguyên như: RAM, CPU, Storage, IP Public... 

Tại trang quản lý của Vultr sẽ cho phép nâng cấp plan đang sử dụng để thuê thêm tài nguyên hoặc thuê thêm IP Public.

Chọn plan cần nâng cấp, chọn gói và nhấn **Upgrade** để nâng cấp dịch vụ. Vì Vultr tính tiền theo giờ thế nên nó sẽ tính tiền plan mới khi mình upgrade thành công.

### 3.3. Xóa bỏ 1 dịch vụ
Để hủy 1 dịch vụ trên Vultr cần phải truy cập trang quản lý, Chọn dịch vụ nhấn ký tự (...) và chọn Server destroy. Nhấn xác nhận quá trình destroy khá nhanh khoảng 3p đã hủy thành công 1 dịch vụ.

## 4. Triển khai ứng dụng web
Trong bài viết này tôi sẽ cài LEMP Stack trên hệ điều hành Centos. Và đẩy source lên trên server và tiến hành chạy 1 ứng dụng web.

Thường khi chúng ta sử dụng hosting của các nhà cung cấp dịch vụ như tenten, z.com... thường sẽ có trang dashboard để quản lý tài nguyên, source code website. Họ sử dụng control panel để tạo ra admin panel cho người dùng sử dụng. Hiện tại có khá nhiều control panel như: Cpanel, CWP(Control Web Page) Control Panel, Sentora, VestaCP, Webuzo, ISPConfig, VirtualMin, Plesk, DirectAdmin, ... 

Bản chất các control này sẽ cài các gói LEMP (Linux, Engine x, Mysql, PHP) hoặc LAMP (Linux, Apache, Mysql, PHP) để có thể tạo ra môi trường để source code website có thể chạy được. Các nhà cung cấp hosting sẽ cấu hình và tạo ra 1 control admin panel để giúp người dùng sử dụng mà không cần biết cấu hình như thế nào. Trong trường hợp này tôi sử dụng 1 dịch vụ cơ sở hạ tầng của Vultr cung cấp là: 1 server Centos 7 - Ram 512MB, 10GB SSD, 0.5TB băng thông. Để có thể chạy 1 ứng dụng web tôi cần phải cài gói LAMP hoặc LEMP và đẩy source code của tôi lên server.

### 4.1. Cài đặt LEMP (Nginx, MySQL, PHP)

{:.info}
LEMP server là một server chạy Linux, Nginx (đọc là Engine x), MySql và PHP (hoặc Perl/Python). Nó tương tự như LAMP server ngoại trừ việc web server nền tảng được giám sát bằng Nginx thay vì Apache.

**Cài đặt nginx**

Thêm CentOS 7 EPEL repository
```
sudo yum install epel-release
```
Cài đặt nginx trên Centos 7
```
sudo yum install nginx
```
Khởi chạy nginx
```
sudo systemctl start nginx
```
Nếu virtual private của bạn đang sử dụng firewall, run câu lệnh sau để cho phép HTTP và HTTPS traffic:
```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
Để kiểm tra nginx hoạt động hay chưa hãy kiểm tra bằng cách truy cập vào public IP trên webbrowser.
```
http://server_domain_name_or_IP/
```
Nếu nhìn thấy giao diện mặc định CentOS 7 Nginx web page thì đã cài thành công nginx.

![Nginx Default](/assets/images/nginx-default.png)

**Cài đặt PHP 7**

Bây giờ cài đặt PHP, ở đây tôi cài PHP 7. Tiến hành cài đặt PHP 7 từ remi-php72 rpm repository.
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

## PHP 7.2 ##
yum --enablerepo=remi,remi-php72 install -y nginx php-fpm php-common
```

Cài đặt các module php cần thiết. Dùng câu lệnh sau để hiển thị danh sách module có sẵn trong yum repositories.
```
yum --enablerepo=remi-php72 search php
```

Một số module PHP thông dụng:
- **OPcache (php-opcache)** – The Zend OPcache provides faster PHP execution through opcode caching and optimization.
- **APCu (php-pecl-apc)** – APCu userland caching
- **CLI (php-cli)** – Command-line interface for PHP
- **PEAR (php-pear)** – PHP Extension and Application Repository framework
- **PDO (php-pdo)** – A database access abstraction module for PHP applications
- **MySQL (php-mysqlnd)** – A module for PHP applications that use MySQL databases
- **PostgreSQL (php-pgsql)** – A PostgreSQL database module for PHP
- **MongoDB (php-pecl-mongo)** – PHP MongoDB database driver
- **SQLite (php-pecl-sqlite)** – Extension for the SQLite Embeddable SQL Database Engine
- **Memcache (php-pecl-memcache)** – Extension to work with the Memcached caching daemon
- **Memcached (php-pecl-memcached)** – Extension to work with the Memcached caching daemon
- **GD (php-gd)** – A module for PHP applications for using the gd graphics library
- **XML (php-xml)** – A module for PHP applications which use XML
- **MBString (php-mbstring)** – A module for PHP applications which need multi-byte string handling
- **MCrypt (php-mcrypt)** – Standard PHP module provides mcrypt library support

Để cài đặt bạn hãy sử dụng lệnh ```yum --enablerepo=remi,remi-php56 install ten_module```.

Cài các module php cần thiết sau:
```
yum --enablerepo=remi,remi-php56 install -y php-opcache php-pecl-apcu php-cli php-pear php-pdo php-mysqlnd php-pgsql php-pecl-mongo php-pecl-sqlite php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml
```

**Cài đặt MySQL 5.6 (MariaDB)**

Thêm MariaDB repo

{:.info}
Lưu ý: Không cần thực hiện trên CentOS 7. Phiên bản MariaDB trên CentOS 7 hiện nay là 5.5.37

Cài đặt hoặc update MariaDB

```
## CentOS 7 ##
yum install -y mariadb mariadb-server
```

Khởi động MariaDB và tự động chạy khi boot
```
## CentOS 7 ##
systemctl start mariadb.service
systemctl enable mariadb.service
```

Cấu hình MariaDB
- Set (Change) root password
- Remove anonymous users
- Disallow root login remotely
- Remove test database and access to it
- Reload privilege tables

Chạy lệnh sau để cài đặt
```
/usr/bin/mysql_secure_installation
```
Ngay bước đầu tiên bạn sẽ bị hỏi root password, do mới cài đặt nên tất nhiên chưa có password, nhấn Enter để tiếp tục.

Output tương tự như sau:
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
### 4.2. Tạo Block Server Nginx
#### 4.2.1. DNS Domain
Để DNS bạn cần truy cập vào trang quản lý DNS của nhà cung cấp domain. Tại đây thêm 1 record Type A: Host record: php, Type: A, Value: địa chỉ ip VPS.

![DNS](/assets/images/dns-tenten.png)

Với bản ghi trên sẽ DNS trỏ về subdomain là: php.yourdomain.com
#### 4.2.2. Config Nginx
Để DNS có thể trỏ đúng về webser thì ta cần phải thêm 1 block server. Ở Apche2 virtual host được định nghĩa là virtual host còn đối với nginx lại được định nghĩa là block server.
Mỗi 1 block server được định nghĩa trong thư mục ```cd /etc/nginx/conf.d``` để thêm 1 block server chỉ cần thêm 1 file ```.conf```.
Để thêm block server gõ lệnh sau
```
cd /etc/nginx/conf.d
```
Tạo mới 1 block server .conf
```
nano php.yourdomain.conf
```
Gõ đoạn sau vào file php.yourdomain.conf
```
server {
    listen 80;
    server_name yourdomain www.yourdomain;

    location / {
        root   /home/php/public_html;
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }

    error_page  404              /404.html;
    location = /404.html {
        # /home/php/public_html la thu muc chua source code
        root   /home/php/public_html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        # /home/php/public_html la thu muc chua source code
        root   /home/php/public_html;
    }
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /home/php/public_html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
### 4.3. Upload source code lên hosting
#### 4.3.1. Tạo MySQL Database
Gõ lệnh
```
mysql -u root -p
```
Tạo mới 1 Database mới
```
CREATE DATABASE cloudcomputing_db;
```
Tạo mới user. Lệnh dưới đây sẽ tạo ra 1 user mới là **cloudcomputing** pass word là **cloudcomputing_demo**
```
CREATE USER 'cloudcomputing'@'localhost' IDENTIFIED BY 'cloudcomputing_demo';
``` 
Phân quyền cho user cloudcomputing
```
GRANT ALL PRIVILEGES ON cloudcomputing_db. * TO 'cloudcomputing'@'localhost';
```
Reload privileges.
```
FLUSH PRIVILEGES;
```
Tạo 1 Table Guests
```
CREATE TABLE Guests ( id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY, firstname VARCHAR(30) NOT NULL, lastname VARCHAR(30) NOT NULL, email VARCHAR(50), register_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP );
```
Để thoát khỏi mysql gõ lệnh
```
exit;
```
#### 4.3.2. Upload source code
Có nhiều cách để đẩy source code lên hosting, có nhiều cách để upload source code lên hosting. Với 1 VPS thì sẽ không có trang **Admin Panel** để truy cập file manager để upload source code. Có thể sử dụng ftp (File Transfer Protocol), git, **eXtplorer** (Trình quản lý file cho hosting tương tự file manager).

Trong bài viết này tôi sử dụng git. Để clone 1 repo sử dụng lênh ```git clone namerepo```.

Để ví dụ 1 ứng dụng web quản lý sách đơn giản. Tôi sẽ clone repo [mvc-php-example](https://github.com/tuanictu97/mvc-php-example.git)
```
cd /home/php/public_html
git clone https://github.com/tuanictu97/mvc-php-example.git .
```







