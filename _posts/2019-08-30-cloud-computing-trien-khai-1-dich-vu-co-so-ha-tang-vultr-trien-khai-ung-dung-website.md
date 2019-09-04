---
layout: article
title: Cloud Computing - Triển khai 1 dịch vụ cơ sở hạ tầng (Vultr) & triển khai ứng dụng Website
tags: [cloud computing, vultr, lemp]
---
Trong bài này tôi sẽ trình bày về sự hiểu biết của mình về **(Cloud Computing)** . 

Cụ thể hơn trong bài viết này tôi sẽ tìm hiểu về dịch vụ cho thuê cơ sở hạ tầng của Vultr (Infrastructure Cloud) & Cài đặt cái gói (LEMP Stack) để triển khai 1 ứng dụng website chạy trên VPS (Virtial Private Server).

## 1. Các yêu cầu cần thiết
1. Một tài khoản Vultr đã kích hoạt thanh toán.
2. Công cụ:
  - LEMP Stack
  - Phần mềm remote VPS: Terminal/Putty
  - IDE Intelli IDEA
3. Ngôn ngữ lập trình: PHP hoặc Node.JS
4. OS: Centos 7

## 2. Giới thiệu về Vultr
Vultr là nhà cung cấp dịch vụ VPS Server, Cloud Server với 100% phần cứng SSD, 15 datacenter location trải dài trên khắp thế giới. Ưu điểm của Vultr là giá rẻ, hiệu năng cao và cài đặt dễ dàng, nhanh chóng.

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

KVM VPS không có tài nguyên dùng chung, tất cả đã được mặc định sẵn và chia sẻ. Điều đó có nghĩa là RAM của mỗi VPS KVM đã được phân bổ sẵn cho từng gói VPS để sở hữu, tận dụng hoàn toàn 100% và không bị chia sẻ ra ngoài. Đồng nghĩa với việc VPS KVM sẽ hoạt động ổn định hơn mà không bị ảnh hưởng bởi các VPS khác cùng hoạt động trên hệ thống. Tương tự điều đó với disk space, tài nguyên ổ cứng cũng đã được xác định sẵn và không bị vay mượn bởi các VPS khác.

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

### 4.1. Cài đặt LEMP

{:.info}
LEMP server là một server chạy Linux, Nginx (đọc là Engine x), MySql và PHP (hoặc Perl/Python). Nó tương tự như LAMP server ngoại trừ việc web server nền tảng được giám sát bằng Nginx thay vì Apache.

### 4.2. Đẩy source code lên host
Có nhiều cách để đẩy source code lên hosting. Với 1 VPS thì sẽ không có trang admin cpanel để truy cập file manager để upload source code. Giải pháp là có thể sử dụng git hoặc cài thêm eXtplorer (Trình quản lý file cho hosting tương tự file manager).







