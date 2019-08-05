---
layout: article
title: Một số ví dụ rule Nginx
tags: [lemp, nginx]
---

Khác với Apache, Nginx không sử dụng file .htaccess nên khi bạn cần rewrite url sẽ phải convert qua rule của Nginx. Trong bài viết này, mình sẽ đưa ra một số ví dụ các rule của Nginx sử dụng để rewrite url, redirect và một số cấu hình cần thiết khác.
### 1. Canonical URLs
Đảm bảo website của bạn được search engine index một đường dẫn duy nhất.

**www.mydomain.com -> mydomain.com**

Redirect toàn bộ request từ www.mydomain.com sang mydomain.com, kèm theo cả đường dẫn và tham số phía sau url.

```
server {
      server_name www.mydomain.com;
      rewrite ^(.*) $scheme://mydomain.com$1 permanent;
}
```

**mydomain.com -> www.mydomain.com**

Redirect toàn bộ request từ mydomain.com sang www.mydomain.com, kèm theo cả đường dẫn và tham số phía sau url.
```
server {
      server_name mydomain.com;
      rewrite ^(.*) $scheme://www.mydomain.com$1 permanent;
}
```
### 2. SSL Sites
Google hiện tại đang sử dụng SSL là một yếu tố xếp hạng, nếu bạn sử dụng các site eCommerce, hãy sử dụng SSL.

Rule bên dưới sẽ redirect tất cả request sử dụng HTTPS, trường hợp này giả sử bạn sử dụng www.domain.com làm tên miền chính.

```
# Tells the browser to always force SSL.
if ($scheme != "https") {
    rewrite ^ https://www.mydomain.com$uri permanent;
}
if ($host != "www.mydomain.com") {
    rewrite ^ https://www.mydomain.com$uri permanent;
}
```
### 3. Bắt buộc sử dụng SSL
Bắt buộc phải sử dụng SSL với một đường dẫn bất kỳ:
```
set $redirect false; 
if ($request_uri ~ ^/manager(\/)?$ ) { 
    set $redirect true; 
} 
if ($scheme = https) { 
    set $redirect false; 
} 
if ($redirect = true) { 
    return 301 https://www.domain.com$request_uri; 
}
```
### 4. Tối ưu Browser Caching
Bằng việc sử dụng browser caching, website của bạn sẽ load nhanh hơn rất nhiều kể từ sau lần visit đầu tiên:
```
location ~* \.(?:ico|css|js|jpe?g|png|gif|svg|pdf|mov|mp4|mp3|woff)$ {
    expires 7d;
    add_header Pragma public;
    add_header Cache-Control "public";
    gzip_vary on;
}
```
### 5. Giới hạn IP truy cập
Trong trường hợp bạn muốn chỉ một số IP được phép truy cập, hãy sử dụng rule bên dưới:
```
location /manager/ {
    # allow anyone in 192.168.1.0/24
    allow   192.168.1.0/24;
    
    # allow one workstation
    allow 127.0.0.1;

    # drop rest of the world 
    deny all;
}
```
### 6. Block IP
Khác với trường hợp chỉ cho phép một số IP được truy cập vào, rule bên dưới sẽ block một số IP xác định trước.
```
location /manager/ {
    # block one workstation
    deny 192.168.1.1;

    # block anyone in 127.0.1.1/24
    deny   127.0.1.1/24;

    # allow rest of the world 
    allow all;
}
```
### 7. Ngăn những site khác sử dụng hình ảnh
Với đoạn code dưới, bạn sẽ hạn chế những site khác sử dụng hình ảnh trực tiếp:
```
location ~* \.(gif|png|jpe?g)$ {
     valid_referers none blocked ~.google. ~.bing. ~.yahoo. .domain.com *.domain.com;
     if ($invalid_referer) {
        return   403;
    }
}
```
Nếu bạn muốn chặn sử dụng ảnh trong 1 thư mục cụ thể:
```
location /wp-content/ {
     valid_referers none blocked ~.google. ~.bing. ~.yahoo. .domain.com *.domain.com;
     if ($invalid_referer) {
        return   403;
    }
}
```
Nếu bạn muốn thay vì báo lỗi mà hiển thị một hình ảnh khác, hãy sử dụng code bên dưới:
```
location ~* \.(gif|png|jpe?g)$ {
     valid_referers none blocked ~.google. ~.bing. ~.yahoo. .domain.com *.domain.com;
     if ($invalid_referer) {
        rewrite (.*)\.(jpg|jpeg|png|gif)$ http://www.domain.com/images/warning.jpg;
    }
}
```
**Lưu ý**: Nếu trong cấu hình Nginx của website đã tồn tại đoạn cấu hình riêng về file hình ảnh/video thì bạn phải gộp chung với đoạn cấu hình đó chứ không tách riêng rẽ. Ví dụ:
```
location ~* \.(3gp|gif|jpg|jpeg|png|ico|wmv|avi|asf|asx|mpg|mpeg|mp4|pls|mp3|mid|wav|swf|flv|exe|zip|tar|rar|gz|tgz|bz2|uha|7z|doc|docx|xls|xlsx|pdf|iso|eot|svg|ttf|woff)$ {
		valid_referers none blocked ~.google. ~.bing. ~.yahoo. hocvps.com *.hocvps.com;
		if ($invalid_referer) {
			return 403;
			}
		gzip_static off;
		add_header Pragma public;
		add_header Cache-Control "public, must-revalidate, proxy-revalidate";
		access_log off;
		expires 30d;
		break;
        }
```
### 8. Bảo vệ thư mục bằng mật khẩu
Đầu tiên, bạn cần sử dụng [tool](http://www.htaccesstools.com/htpasswd-generator/) này để tạo file .htpasswd, sau đó dùng đoạn code bên dưới, giả sử mình lưu ở ```/root/.htpasswd```
```
location /protectme/ {
    auth_basic "Restricted";
    auth_basic_user_file /root/.htpasswd;
}
```
### 9. Rewrite URL
Ví dụ bên dưới giả sử bạn muốn rewrite (không phải redirect) đường dẫn http://domain.com/listing/123 thành http://domain.com/listing.php?id=123

```
rewrite ^/listing/(.*)$ /listing.php?id=$1 last;
```
(.*) ở đây là regular expression, đại diện cho bất kỳ ký tự nào. Nếu thêm regular expression, bạn sử dụng tương ứng $2, $3…
### 10. Redirect URL cũ sang URL mới
Redirect /someoldarticle.html sang /some/newarticle.html
```
rewrite ^/someoldarticle\.html /some/newarticle.html permanent;
```
### 11. Redirect domain cũ sang domain mới
Redirect http://olddomain.com và toàn bộ đường dẫn con sang domain mới http://newdomain.com
```
rewrite ^(.*) http://newdomain.com$1 permanent;
```
### 12. Redirect IP server sang domain bất kỳ
Mục đích khi truy cập vào IP server, thay vì hiển thị nội dung mặc định của Nginx, người dùng sẽ được tự động redirect đến một địa chỉ website nào đó, ví dụ ```domain.com```.

Bạn hãy mở file cấu hình domain chính, tìm dòng nào có nội dung là listen 80 default_server; thì thay bằng listen 80; (xóa default_server). Tiếp theo, copy paste đoạn cấu hình dưới lên trên cùng file .conf, khởi động lại Nginx là xong:

```
server {
        listen       80  default_server;
        server_name  _;

        rewrite ^(.*) http://domain.com$1 permanent;
}
```
### 13. Cài WordPress trong Sub-folder
Để chạy được WordPress ở thư mục con, bạn hãy thêm đoạn code sau vào đằng trước block ```location / { ... }```
```
location /wordpress/ {
    try_files $uri $uri/ /wordpress/index.php?$args;
}
```
### 14. Hiển thị nội dung thư mục
Nếu thư mục không có file index, khi truy cập bạn có thể gặp thông báo lỗi “403 Forbidden“, không hiển thị những file bên trong đó. Để cho phép người dùng xem được thư mục này, bạn hãy chỉnh lại autoindex là on.
```
location / {
    autoindex on;
}
```
hoặc chỉnh lại là off để không hiển thị nội dung thư mục:
```
location / {
    autoindex off;
}
```
### 15. Redirect link 404
Thông thường link 404 sẽ hiển thị thông báo lỗi của Nginx hoặc trang 404 của WordPress. Nếu bạn muốn hiển thị nội dung của trang chủ, có thể thêm đoạn code sau lên trước block ```location / { ... }```
```
error_page 404 /index.php;
```
Nội dung index.php sẽ xuất hiện, tuy nhiên header status vẫn là 404. Nếu muốn header 200 thì bạn chuyển đoạn code thành: error_page 404 =200 /index.php;

Ngoài ra có một số tool tự động convert sang rule Nginx bạn có thể tham khảo thêm như:
1. http://www.anilcetin.com
2. http://labs.gidix.de/nginx/
3. https://winginx.com/en/htaccess

