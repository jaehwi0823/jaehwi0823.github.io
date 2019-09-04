---
layout: article
title: Một số thủ thuật tối ưu LEMP server trên Ubuntu
tags: [lemp, nginx, linux, ubuntu]
---

Trong bài viết này mình sẽ nói về một số thủ thuật để tối ưu LEMP server cho nó hoạt động ổn định, hiệu quả hơn. Không phải tất cả thủ thuật đều tốt, bạn nên sử dụng cẩn thận nhé.

## 1. Tự khởi động lại server khi hết bộ nhớ (không nên sử dụng)
```
sudo nano /etc/sysctl.conf
```
Thêm vào dưới cùng 2 dòng sau
```
vm.panic_on_oom=1
kernel.panic=10
```
The vm.panic_on_oom=1 line enables panic on OOM; the kernel.panic=10 line tells the kernel to reboot ten seconds after panicking.
## 2. Tự động optimize MySQL table hàng tuần
Mở file crontab
```
crontab -e
```
Thêm dòng sau vào dưới cùng
```
@weekly mysqlcheck -o --user=root --password=<your password here> -A
```
Bạn có thể tự chạy mysqlcheck trước để xem câu lệnh đúng chưa.
## 3. Kích hoạt Gzip Compression
Mở file cài đặt nginx
```
sudo nano /etc/nginx/nginx.conf
```
Chỉnh phần cài đặt gzip
```
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
```
Reload nginx
```
sudo /etc/init.d/nginx reload
```
## 4. Make Browsers Cache Static Files On nginx
Thủ thuật này bắt trình duyệt sẽ phải lưu cache các file tĩnh (images, CSS và Javascript) và load lại chúng thay vì gửi yêu cầu tới server tải lại.

Mở file block server mặc định hoặc file custom của bạn.

```
sudo nano /etc/nginx/sites-available/default
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

Trên đây là những thủ thuật để có thể tối ưu lemp. Chúc các bạn thành công !