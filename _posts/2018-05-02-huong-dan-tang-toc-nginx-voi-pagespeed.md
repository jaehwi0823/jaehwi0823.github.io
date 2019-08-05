---
layout: article
title: Hướng dẫn tăng tốc Nginx với Pagespeed
tags: [lemp, nginx]
---
Nginx bản thân nó đã là một web server có hiệu suất hoạt động rất tốt. Tuy nhiên vẫn có nhiều cách để tối ưu thêm và một trong số đó là sử dụng module được phát triển bởi Google có tên [PageSpeed](https://developers.google.com/speed/pagespeed/module) (ngx_pagespeed)

![PageSpeed](/assets/images/ngx_pagespeed.png)

**ngx_pagespeed** tăng tốc website của bạn và giảm thời gian load đáng kể bằng cách tự động áp dụng các kỹ thuật tối ưu hóa page và các thành phần tĩnh như CSS, Javascript, Image.

Một số filter hay của ngx_pagespeed:
- **Collapse Whitespace**: giảm băng thông sử dụng bằng cách thay thế nhiều khoảng trắng (whitespace) trong HTML bằng 1 khoảng trắng mà thôi.
- **Canonicalize JavaScript Libraries**: giảm băng thông sử dụng bằng cách tự động sử dụng các thư viện Javascript phổ biến trên server free (vd như của Google).
- **Combine CSS**: giảm số lượng HTTP requests bằng cách kết hợp nhiều file CSS thành một file.
- **Combine JavaScript**: giảm số lượng HTTP requests bằng cách kết hợp nhiều file JavaSript thành một file.
- **Extend Cache**: giảm băng thông sử dụng bằng cách tối ưu chức năng cache của browser.
- **Flatten CSS Imports**: giảm số lượng HTTP request bằng cách xóa @import trong file CSS.
- **Lazyload Images**: làm chậm lại việc load các hình ảnh ko được hiển trị trên trình duyệt người dùng.
- **Minify JavaScript**: giảm băng thông sử dụng bằng cách tối ưu kích thước file Javascript.
- **Optimize Images**: tối ưu hóa hình ảnh bằng cách sử dụng inline images, nén hình ảnh, hoặc convert GIF sang PNG.
- **Pre-Resolve DNS**: giảm thời gian phân giải DNS bằng cách phân giải trước DNS sử dụng HTML.

Và còn rất nhiều filter và ví dụ minh họa khác của ngx_pagespeed trong [PageSpeed Filter Examples](http://ngxpagespeed.com/ngx_pagespeed_example/).

Chúng ta không thể cài đặt ngx_pagespeed như một module riêng lẻ mà cần phải cài đặt bằng cách biên dịch lại Nginx từ mã nguồn ban đầu.

## 1. Biên dịch Nginx với ngx_pagespeed
### 1.1. Chuẩn bị
**a. Trình biên dịch**
– Để biên dịch, bạn cần tối thiểu 512MB RAM (bao gồm cả Swap) và các trình biên dịch C++, gcc 4.8 hoặc clang 3.3 trở lên.
Trên CentOS 6:
```
# yum -y install gcc-c++ pcre-devel zlib-devel make unzip libuuid-devel
#  rpm --import http://linuxsoft.cern.ch/cern/slc6X/i386/RPM-GPG-KEY-cern
#  wget -O /etc/yum.repos.d/slc6-devtoolset.repo http://linuxsoft.cern.ch/cern/devtoolset/slc6-devtoolset.repo
#  yum install devtoolset-2-gcc-c++ devtoolset-2-binutils
#  scl enable devtoolset-2 bash
```
Trên CentOS 7:
```
# yum -y install gcc-c++ pcre-devel zlib-devel make unzip libuuid-devel
```
Trên Debian hoặc Ubuntu:
```
# sudo apt-get install build-essential zlib1g-dev libpcre3 libpcre3-dev unzip uuid-dev gcc-mozilla
```
Kiểm tra phiên bản GCC:
```
# gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-16)
```
**b. Mã nguồn Nginx**
– Tải mã nguồn Nginx (phiên bản mới nhất 1.14.0)). Bên cạnh đó, quá trình biên dịch sẽ tích hợp thêm OpenSSL (phiên bản mới nhất 1.1.1-pre8). Giải nén vào thư mục /usr/local/src.

```
# cd /usr/local/src
# wget http://nginx.org/download/nginx-1.14.0.tar.gz && tar -xzvf nginx-1.14.0.tar.gz
# wget https://www.openssl.org/source/openssl-1.1.1-pre8.tar.gz && tar -xzvf openssl-1.1.1-pre8.tar.gz
```
**c. Mã nguồn PageSpeed**

– Tải mã nguồn ngx_pagespeed (phiên bản mới nhất 1.13.35.2-stable) cùng PSOL(PageSpeed Optimization Libraries). Giải nén vào thư mục /usr/local/src.
```
# cd /usr/local/src 
# NPS_VERSION=1.13.35.2-stable
# wget https://github.com/apache/incubator-pagespeed-ngx/archive/v${NPS_VERSION}.zip
# unzip v${NPS_VERSION}.zip
# nps_dir=$(find . -name "*pagespeed-ngx-${NPS_VERSION}" -type d)
# cd "$nps_dir"
# NPS_RELEASE_NUMBER=${NPS_VERSION/beta/}
# NPS_RELEASE_NUMBER=${NPS_VERSION/stable/}
# psol_url=https://dl.google.com/dl/page-speed/psol/${NPS_RELEASE_NUMBER}.tar.gz
# [ -e scripts/format_binary_url.sh ] && psol_url=$(scripts/format_binary_url.sh PSOL_BINARY_URL)
# wget ${psol_url}
# tar -xzvf $(basename ${psol_url})
```
### 1.2. Biên dịch và thay thế Nginx
Tiến hành biên dịch lại Nginx bằng cách giữ nguyên cấu hình ban đầu, thêm module **PageSpeed**.
- Truy cập thư mục mã nguồn Nginx vừa tải:
```
# cd /usr/local/src/nginx-1.14.0
```
- Lưu lại các tham số cấu hình cùng module sử dụng của Nginx đang chạy trên VPS.
```
# nginx -V
nginx version: nginx/1.12.1
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC)
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```
- Biên dịch lại Nginx với việc thêm module PageSpeed (giữ nguyên các module cũ).
```
# ./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --with-openssl=/usr/local/src/openssl-1.1.1-pre8 --add-module=/usr/local/src/incubator-pagespeed-ngx-1.13.35.2-stable/
```
- Thay thế Nginx trên VPS bằng Nginx vừa được biên dịch lại:
```
# make
# make install
```
Thông báo make[1]: Leaving directory `/usr/local/src/nginx-1.12.2' mà không có error là OK.

**Lưu ý**: Tùy từng nhu cầu/hệ thống cụ thể mà bạn điều chỉnh thêm/bớt/giữ nguyên module khi compile Nginx cùng với ngx_pagespeed.
- Khởi động lại Nginx và kiểm tra. Kết quả như sau thì đã tích hợp thành công ngx_pagespeed vào Nginx.
```
# service nginx restart && nginx -V
```
```
nginx version: nginx/1.14.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)
built with OpenSSL 1.1.1-pre8 (beta) 20 Jun 2018
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --with-openssl=/usr/local/src/openssl-1.1.1-pre8 --add-module=/usr/local/src/incubator-pagespeed-ngx-1.13.35.2-stable/
```
## 2. Cấu hình module ngx_pagespeed
- Trước khi tiến hành cấu hình, bạn cần tạo thư mục cache cho PageSpeed.
```
# mkdir /var/ngx_pagespeed_cache
# chown nginx:nginx /var/ngx_pagespeed_cache
```
- Để kích hoạt và cấu hình ngx_pagespeed, bạn cần chỉnh sửa file configuration của Nginx (/etc/nginx/nginx.conf) hoặc Nginx conf dành cho domain (chèn trong block server).

Cụ thể, ngx_pagespeed có rất nhiều filter khác nhau, tùy theo mục đích sử dụng mà các bạn lựa chọn cho phù hợp. Có 2 level khác nhau bạn có thể sử dụng là CoreFilters(mặc định) và PassThrough.

### 2.1. CoreFilters
CoreFilters là một tập hợp các filter được Google xác nhận là an toàn với hầu hết các website. Do đó, cách này phù hợp với các bạn newbie mới tìm hiểu. Nếu muốn, bạn có thể disable một filter bất kỳ khỏi CoreFilters hoặc thêm một filter khác vào.

Đây là một ví dụ cấu hình ngx_pagespeed với CoreFilters:
```
# enable ngx_pagespeed
        pagespeed on;
        pagespeed FileCachePath /var/ngx_pagespeed_cache;

        # let's speed up PageSpeed by storing it in the super duper fast memcached
        # pagespeed MemcachedThreads 1;
        # pagespeed MemcachedServers "localhost:11211";

        # enable CoreFilters
        pagespeed RewriteLevel CoreFilters;

        # disable particular filter(s) in CoreFilters
        pagespeed DisableFilters rewrite_images;

        # enable additional filter(s) selectively
        pagespeed EnableFilters collapse_whitespace;
        pagespeed EnableFilters lazyload_images;
        pagespeed EnableFilters insert_dns_prefetch;
```
Xem danh sách toàn bộ filter có trong CoreFilters tại [đây](https://developers.google.com/speed/pagespeed/module/config_filters).

### 2.2. PassThrough Filters
Với các bạn đã có nhiều kiến thức, trải nghiệm thì nên sử dụng PassThrough. Khi đó sẽ cần tự kích hoạt những filter cần dùng.

Cấu hình ví dụ với PassThrough:

```
# enable ngx_pagespeed
        pagespeed on;

        pagespeed FileCachePath /var/ngx_pagespeed_cache;

        # let's speed up PageSpeed by storing it in the super duper fast memcached
        # pagespeed MemcachedThreads 1;
        # pagespeed MemcachedServers "localhost:11211";

        # disable CoreFilters
        pagespeed RewriteLevel PassThrough;

        # enable collapse whitespace filter
        pagespeed EnableFilters collapse_whitespace;

        # enable JavaScript library offload
        pagespeed EnableFilters canonicalize_javascript_libraries;

        # combine multiple CSS files into one
        pagespeed EnableFilters combine_css;

        # combine multiple JavaScript files into one
        pagespeed EnableFilters combine_javascript;

        # remove tags with default attributes
        pagespeed EnableFilters elide_attributes;

        # improve resource cacheability
        pagespeed EnableFilters extend_cache;

        # flatten CSS files by replacing @import with the imported file
        pagespeed EnableFilters flatten_css_imports;
        pagespeed CssFlattenMaxBytes 5120;

        # defer the loading of images which are not visible to the client
        pagespeed EnableFilters lazyload_images;

        # enable JavaScript minification
        pagespeed EnableFilters rewrite_javascript;

        # enable image optimization
        pagespeed EnableFilters rewrite_images;

        # pre-solve DNS lookup
        pagespeed EnableFilters insert_dns_prefetch;

        # rewrite CSS to load page-rendering CSS rules first.
        pagespeed EnableFilters prioritize_critical_css;
```
Khởi động lại web server để các thay đổi có tác dụng
```
# service nginx restart
```
Sau đó, bạn có thể kiểm tra tại [Is Mod PageSpeed Working](https://ismodpagespeedworking.com/). ngx_pagespeed cùng với [memcache](https://hocvps.com/huong-dan-cai-dat-memcached-day-du-tren-centos-6/), [Zend Opcache](https://hocvps.com/cai-dat-va-cau-hinh-php-zend-opcache/) là một trong số những module mình khuyến khích mọi người sử dụng để tối ưu hóa nginx server của bạn.

