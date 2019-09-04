---
layout: article
title: Một số thủ thuật tối ưu LEMP server trên CentOS
tags: [lemp, nginx, linux, centos]
---

Sau khi cài đặt xong LEMP trên CentOS, bạn sẽ cần thực hiện một số tùy chỉnh để đảm bảo hệ thống hoạt động nhanh hơn, tăng tốc website. Bên dưới là một số thao tác mình thực hiện mỗi khi cài đặt server.

## 1. Kích hoạt Gzip Compression
Mở file cài đặt nginx
```
sudo nano /etc/nginx/nginx.conf
```
Chỉnh phần cài đặt gzip
```
...
http {
...

# enable gzip compression
gzip on;
gzip_disable "msie6";
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
# end gzip configuration
}
...
```
Reload nginx
```
sudo /etc/init.d/nginx reload
```
## 2. Make Browsers Cache Static Files On nginx
Mở file block server mặc định hoặc file custom của bạn
```
sudo nano /etc/nginx/conf.d/default.conf
```
Thay đổi cài đặt như bên dưới.
```
[...]
server {
        [...]
        location ~*  \.(jpg|jpeg|png|gif|ico|css|js)$  {
              expires 365d;
        }
        [...]
}
[...]
```
Reload nginx
```
sudo /etc/init.d/nginx reload
```