---
layout: article
title: Nginx Amplify – Công cụ giám sát Nginx miễn phí tuyệt vời
tags: [lemp, nginx, linux]
---

Nginx Amplify là công cụ tuyệt vời giúp giám sát tình trạng NGINX và Server theo thời gian thực, qua đó giúp phân tích và tối ưu các ứng dụng hoạt động dựa trên NGINX.

Đặc biệt, với bản cập nhật mới nhất, Nginx Amplify theo dõi toàn bộ hệ thống cài đặt LEMP Stack – Linux, Nginx, PHP-FPM và cả MySQL.

![Nginx Amplify](/assets/images/nginx-amplify.jpg)

Công cụ hoạt động trên nền tảng điện toán đám mây, được cài đặt thông qua một module của Nginx kết nối với API cung cấp miễn phí bởi Nginx.com, giúp bạn dễ dàng theo dõi hoạt động của hệ thống cũng như chủ động xác định các vấn đề:
- Hiển thị trực quan, chính xác dễ hiểu thông qua số liệu, biểu đồ tình trạng server: hoạt động CPU, RAM, bộ nhớ ổ đĩa, băng thông…
- Cung cấp các lời khuyên và gợi ý thông minh nhằm cải thiện và tối ưu hóa hiệu suất Nginx.
- Nhận thông báo qua mail khi có lỗi xảy ra trong quá trình vận hành của các ứng dụng cũng như hỗ trợ xác định vấn đề phát sinh ở đâu…
- Dự đoán về khả năng và hiệu suất hoạt động của các ứng dụng.
- Tích hợp và theo dõi nhiều hệ thống chạy Nginx chỉ trong 1 tài khoản.

Toàn bộ thông tin được hiển thị trên website https://amplify.nginx.com, theo dõi rất thuận tiện. Nếu bạn đang sử dụng webserver Nginx, nên cài đặt thêm công cụ này nha.

## 1. Cài đặt Nginx Amplify
Yêu cầu hệ thống cho Nginx Amplify:
- Chỉ hoạt động trên hệ điều hành Linux, cụ thể: 
  - RHEL/CentOS/OEL 6 và RHEL/CentOS/OEL 7
  - Ubuntu 12.04, 14.04, 16.04, 17.04, 17.10
  - Debian 7, 8, 9
  - Amazon Linux 2017.09, Gentoo Linux (phiên bản Experimental Ebuild)
- Chỉ hoạt động với Python 2.6, 2.7 và chưa hỗ trợ Python 3.0.

### 1.1. Đăng kí tài khoản Nginx Amplify
Làm theo các bước sau để đăng kí tài khoản.
- Truy cập trang đăng kí [Amplify Sign Up](https://amplify.nginx.com/signup/)
- Điền đầy đủ thông tin và chấp nhận Terms of Service rồi click Sign Up
- Tại cửa sổ tiếp theo, bạn cung cấp thêm một số thông tin về bản thân để Nginx support được tốt hơn.

Như vậy, bạn đã hoàn tất đăng kí. Lưu ý, Nginx Amplify sẽ bắt đầu giám sát hệ thống của bạn khi và chỉ khi bạn cài đặt thành công Amplify Agent trên mỗi server và kết nối thông qua API.

### 1.2. Cài đặt NGINX Amplify Agent trên hệ thống
Công cụ thu thập dữ liệu về hoạt động hệ thống thông qua việc cài đặt Nginx Amplify Agent.

Click vào biểu tượng **+New System** góc trên bên trái sau khi login bạn sẽ thấy hướng dẫn cài đặt Agent lên hệ thống của mình. Ví dụ như trong hình:

![Install Nginx Amplify Agent](/assets/images/install-nginx-amplify-agent.png)

1. Kết nối SSH đến server
2. Sử dụng curl hoặc wget để download script install (trước đó nên chuyển về thư mục gốc ```cd /root```)
3. Chạy lệnh cài đặt Amplify Agent

Nếu không có vấn đề gì xảy ra, bạn sẽ nhận được thông báo cài đặt thành công như bên dưới:

![Installed Amplify Agent](/assets/images/installed-amplify-agent.png)

Bạn chờ 1 vài phút sẽ thấy hệ thống hiển thị trên khu vực **Systems** và các đồ thị ở mục **Graphs**. Giao diện Nginx Amplify sẽ trông giống như ảnh ở phần 2.

Để hiển đầy đủ các thông số nâng cao, bạn cần cấu hình thêm Stub Status, Log Format và MySQL metrics. Để thay đổi Hostname thủ công, bạn hãy edit file ```/etc/amplify-agent/agent.conf``` rồi restart lại agent:

```
service amplify-agent restart
```

## 2. Cài đặt nâng cao
### 2.1. Thông số Stub Status
Quay trở lại website, nhấn nút ```Continue``` để đến bước 2, cấu hình ```stub_status``

![Config Stub Status](/assets/images/config-stub_status.png)

Bạn cần chỉnh sửa cấu hình Nginx domain .conf thủ công theo như hướng dẫn rồi reload Nginx để áp dụng:
```
 nginx -s reload
```
Bước này có thể phát sinh lỗi và tốn thời gian nhất, để kiểm tra cấu hình ```stub_status``` chuẩn, bạn có thể chạy ```curl``` đến localhost như sau là được:
```
$ curl http://localhost/nginx_status
Active connections: 2
server accepts handled requests
 344014 344014 661581
Reading: 0 Writing: 1 Waiting: 1
```

### 2.2. Thông số Log
Bạn cần [cấu hình nâng cao Log Format](https://amplify.nginx.com/docs/guide-metrics-and-metadata.html#additional-nginx-metrics) để hiển thị đầy đủ các thống số Nginx.

![Config log file](/assets/images/config-log-file.png)

Cụ thể, cấu hình log_format  main_ext trong Nginx:
```
user  nginx;
worker_processes  1;
worker_rlimit_nofile 260000;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
	worker_connections  2048;
	accept_mutex off;
	accept_mutex_delay 200ms;
	use epoll;
	#multi_accept on;
}

http {
	include       /etc/nginx/mime.types;
	default_type  application/octet-stream;
        log_format  main_ext  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" '
                      '"$host" sn="$server_name" '
                      'rt=$request_time '
                      'ua="$upstream_addr" us="$upstream_status" '
                      'ut="$upstream_response_time" ul="$upstream_response_length" '
                      'cs=$upstream_cache_status' ;
	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	              '$status $body_bytes_sent "$http_referer" '
	              '"$http_user_agent" "$http_x_forwarded_for"';

	#Disable IFRAME
	add_header X-Frame-Options SAMEORIGIN;	
	#Prevent Cross-site scripting (XSS) attacks
	add_header X-XSS-Protection "1; mode=block";	
	#Prevent MIME-sniffing
	add_header X-Content-Type-Options nosniff;
	
	access_log  off;
	sendfile on;
	tcp_nopush on;
	tcp_nodelay off;
	types_hash_max_size 2048;
	server_tokens off;
	server_names_hash_bucket_size 128;
	client_max_body_size 0;
	client_body_buffer_size 256k;
	client_body_in_file_only off;
	client_body_timeout 60s;
	client_header_buffer_size 256k;
	client_header_timeout  20s;
	large_client_header_buffers 8 256k;
	keepalive_timeout 10;
	keepalive_disable msie6;
	reset_timedout_connection on;
	send_timeout 60s;
	
	gzip on;
	gzip_static on;
	gzip_disable "msie6";
	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_buffers 16 8k;
	gzip_http_version 1.1;
	gzip_types text/plain text/css application/json text/javascript application/javascript text/xml application/xml application/xml+rss;
	include /etc/nginx/conf.d/*.conf;
}
```
– Chỉnh sửa log cho từng domain lưu trong ```/etc/nginx/conf.d``, ví dụ:
```
access_log /home/kazeuraki.net/logs/access.log main_ext;
```
```
error_log /home/kazeuraki.net/logs/error.log warn;
```
```
error_log /home/kazeuraki.net/logs/nginx_error.log warn;
```
Xong bạn khởi động lại Nginx ```service nginx restart```
### 2.3. Thông số MySQL
Để thống kê số liệu MySQL, Nginx Amplify cần được kết nối vào hệ thống cơ sở dữ liệu.
Tạo user MySQL: bạn có thể sử dụng user cũ(root/admin…) hoặc tạo user mới bằng câu lệnh trên MySQL.
```
# mysql -u root -p
CREATE USER 'amplify-agent'@'localhost' IDENTIFIED BY 'xxxxxx';
```
Kiểm tra user có thể truy cập thông số MySQL
```
# mysql -u amplify-agent -p
show global status;
```
Một loạt thông số hiện ra là OK

| Variable_name | Value |
| Aborted_clients | 0 |
| … | … |
|452 rows in set (0.00sec)|

Cập nhật Amplify Agent lên phiên bản mới nhất.
```
## Đối với CentOS/Red Hat
yum update nginx-amplify-agent -y
## Đối với Ubuntu/Debian
apt-get update
apt-get install nginx-amplify-agent
```
Chỉnh sửa cấu hình Amplify Agent giống như dưới:
```
# nano /etc/amplify-agent/agent.conf
```
```
[extensions]
phpfpm = True
mysql = True

[mysql]
#host =
#port =
unix_socket = /var/lib/mysql/mysql.sock
user = amplify-agent
password = amplify-agent
```
Trong đó,
- user và password: sử dụng user đã kiểm tra bước trên
- unix_socket: Socket của MySQL, tùy hệ thống bạn quy định.

Khởi động lại Amplify Agent
```
# service amplify-agent restart
# service nginx restart
```
## 3. Sử dụng Nginx Amplify
Với cấu hình nâng cao Amplify, bạn sẽ theo dõi được toàn bộ các thông số về hệ thống LEMP.
![Amplify Graphs](/assets/images/amplify-graphs.png)
### 3.1. Graphs – Đồ thị giám sát
![Nginx Amplify Graphs](/assets/images/nginx-amplify-graphs.png)

Tại trang Graphs, các bạn sẽ thấy các đồ thị về hệ thống và Nginx server, bao gồm:
- **System**: CPU Usage %, Load Average, Memory, Network Traffic, Disk Usage, Disk I/O, Disk Latency, CPU Usage I/O, Disk IOPS, Swap.
- **Nginx**: Connections/s, Requests/s, Current Connections, Current Requests, HTTP Errors, HTTP Version, Workers, File Descriptors, CPU Usage %, Memory Usage, Traffic, Disk I/O, Request Time, Upstream Response Time, Disk Buffered, Upstream Errors.

### 3.2. Dashboard – Bảng giám sát, điều khiển
Tại trang NGINX Amplify **Dashboards**, bạn có thể tùy biến cấu hình biểu đồ để thiết lập báo cáo theo các số liệu tùy chọn từ các số liệu đã thu thập. Bên cạnh đó, bạn có thể thiết lập báo cáo và giám sát Nginx trên 1 server cụ thể hoặc một nhóm các server, cũng như có thể theo dõi 1 service nhỏ hoặc toàn bộ ứng dụng đã cài đặt.

Dưới đây, mình sẽ ví dụ thiết lập 1 biểu đồ cơ bản báo cáo về tình trạng của HTTP [401 Unauthorized](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_Client_Error)

Tại menu thả xuống của Dashboards, chọn Create Dashboard. Chọn Add Graph (góc trên bên phải) tại vùng làm việc mới hiện ra. Tại bảng **Add graph to dashboard**:
- Title: Điền tên cho biểu đồ, ví dụ 401 Errors
- Chose metric to display: Chọn các số liệu liên quan từ menu thả xuống, trong trường hợp này là **nginx.http.status.4xx**. Trong các số liệu của Nginx Amplify, bạn có thể chọn system cho các số liệu về hệ thống và **nginx** cho số liệu về NGINX server
- Chọn hệ thống Nginx muốn giám sát từ danh sách thả xuống của **Click to select NGINX** (có thể chọn nhiều hệ thống)
Để lọc số liệu chọn **Apply Filter**. Menu **Click to select filter key** xuất hiện chọn giá trị lọc ```$status``` cho HTTP status code. Menu bên cạnh **Type or select filter value** hiện ra gõ 401 và chọn **Use 401 as value**. Kết quả cuối cùng thành ```$status ~ 401```.

![Add Graph to Dashboard](/assets/images/add-graph-to-dashboard.png)

### 3.3. Analyzer – Báo cáo về thiết lập hệ thống
![Amplify Analyzer](/assets/images/amplify-analyzer.png)
Khi bạn cài đặt phần mềm lên hệ thống, Nginx Amplify sẽ phân tích cấu hình cài đặt và thiết lập báo cáo chi tiết về thông tin trạng thái và các lời khuyên về cấu hình, trong đó:
- Status information: Tình trạng hiện tại về NGINX của bạn, bao gồm phiên bản, tổng quan về cấu hình (bao gồm cả các modules), các khuyến cáo bảo mật, máy chủ ảo và thông tin SSL/TLS.
- Static analysis: một phân tích về cấu hình của bạn mà cung cấp các khuyến nghị để cải thiện hiệu suất, bảo mật và độ tin cậy của các ứng dụng Nginx

Khi bạn cài đặt Nginx Amplify Agent lên hệ thống, bạn nên xem xét tất cả các phần của báo cáo, bao gồm:
- Version và Overview: tìm kiếm và chỉ ra tất cả các hiểm họa nghi ngờ.
- Security: đánh giá và chỉ ra các khuyến cáo về bảo mật có thể áp dụng
- Virtual servers: Xem xét lại các máy chủ ảo để xem chi tiết cấu hình cũng như thông tin SSL/TLS.
- SSL: kiểm tra tình trạng chứng chỉ SSL, đảm bảo vẫn trong thời hạn.
- Static analysis: Kiểm tra và cảnh báo bất kì mối nguy hiểm nào.

###3.4. Alerts – Cảnh báo hệ thống
Để thực sự giám sát Nginx theo thời gian thực, Nginx Amplify cung cấp bạn công cụ Alerts để cảnh báo bạn khi giá trị của 1 số liệu cụ thể vượt quá mức bạn đề ra. Trong một số trường hợp, Alerts giúp cảnh báo trước khi vấn đề xảy ra thực tế đối với người dùng, đảm bảo rằng bạn không phải là người cuối cùng được biết khi có vấn đề.

Chú ý:
- Bạn không thể tạo Alerts dựa trên số liệu tổng hoặc số liệu trung bình, chỉ có thể dựa trên số liệu chính xác.
- Mặc định, Alerts không lọc theo hostname nên thông thường Alerts cảnh báo bạn khi số liệu của cả hệ thống vượt quá mức đề ra. Để cảnh báo cụ thể, bạn cần chỉ định hostname khi cấu hình Alerts.
- Mỗi tài khoản có 1 Alerts mặc định là cảnh báo khi Nginx Amplify service ngừng hơn 2 phút.

Để tạo 1 cảnh báo mới, chọn thanh Add new alert ở cuối trang Alerts. Giao diện Create New Alerts hiện ra:
- Chose metric to alert: Chọn số liệu cần được cảnh báo
- Over period of: cảnh báo nếu số liệu vượt quá trong ngưỡng thời gian
- Threshold: Ngưỡng số liệu cần cảnh báo, có các giá trị  ```< = > ```
- Systems: Cảnh báo áp dụng cho toàn hệ thống hay 1 server cụ thể
- Email: Điền email nhận thông báo khi Alerts bị kích hoạt. Nếu email này chưa từng được sử dụng với Nginx Amplify thì hệ thống sẽ gửi 1 mail xác nhận

Ví dụ về thiết lập cảnh báo khi Inbound Traffic vượt quá 1 Mbps trong khoảng thời gian trên 2 phút.
![Amplify Alerts](/assets/images/amplify-alerts.png)

Như vậy, qua bài viết, mình đã cung cấp các bạn tổng quan về Nginx Amplify cũng như hướng dẫn cài đặt, sử dụng. Mong rằng qua đó các bạn có thể sử dụng Nginx Amplify một cách tốt nhất.

Tham khảo thêm hướng dẫn sử dụng Amplify [tại đây](https://github.com/nginxinc/nginx-amplify-doc/blob/master/amplify-guide.md).
