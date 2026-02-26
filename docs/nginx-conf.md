# Cấu hình nginx
---
## Mục lục
- [I. Giới thiệu về file cấu hình](#i)
- [II. Cơ chế hoạt động của NGINX](#ii)
- [III. Giải thích file cấu hình](#iii)
  - [1. Main Block](#1)
  - [2. Event block](#2)
  - [3. HTTP block](#3)

---
<a name="i"></a>
## I. Giới thiệu về file cấu hình.

- Mặc định nginx có đường dẫn `/etc/nginx/nginx.conf`
- Nginx quản lý cấu hình theo `Derective` và `Block` chúng có thể nằm lồng ghép với nhau. Những Derective không thuộc block nào sẽ nhóm lại gọi là `Main Block` những cấu hình trên Block này sẽ ảnh hưởng tới toàn bộ server
- Nếu một derective nằm trong block nào đó thì nó có ý nghĩa trong block đó và các block bên trong, khi derective được định nghĩa lại trong các block con thì nó chỉ có trong block con đó
- File cấu hình chính (nginx.conf):
  ```bash
  user www-data;
  worker_processes auto;
  pid /run/nginx.pid;
  error_log /var/log/nginx/error.log;
  include /etc/nginx/modules-enabled/*.conf;

  events {
          worker_connections 768;
          # multi_accept on;
  }

  http {

          ##
          # Basic Settings
          ##

          sendfile on;
          tcp_nopush on;
          types_hash_max_size 2048;
          # server_tokens off;

          # server_names_hash_bucket_size 64;
          # server_name_in_redirect off;

          include /etc/nginx/mime.types;
          default_type application/octet-stream;

          ##
          # SSL Settings
          ##

          ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
          ssl_prefer_server_ciphers on;

          ##
          # Logging Settings
          ##

          access_log /var/log/nginx/access.log;

          ##
          # Gzip Settings
          ##

          gzip on;

          # gzip_vary on;
          # gzip_proxied any;
          # gzip_comp_level 6;
          # gzip_buffers 16 8k;
          # gzip_http_version 1.1;
          # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

          ##
          # Virtual Host Configs
          ##

          include /etc/nginx/conf.d/*.conf;
          include /etc/nginx/sites-enabled/*;
  }


  #mail {
  #       # See sample authentication script at:
  #       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
  #
  #       # auth_http localhost/auth.php;
  #       # pop3_capabilities "TOP" "USER";
  #       # imap_capabilities "IMAP4rev1" "UIDPLUS";
  #
  #       server {
  #               listen     localhost:110;
  #               protocol   pop3;
  #               proxy      on;
  #       }
  #
  #       server {
  #               listen     localhost:143;
  #               protocol   imap;
  #               proxy      on;
  #       }
  #}

- Trong môi trường Production, việc nắm vững sơ đồ thư mục giúp bạn triển khai và fix lỗi nhanh chóng:
  - /etc/nginx/nginx.conf: File cấu hình gốc, chứa các thiết lập toàn cục (Global).
  - /etc/nginx/conf.d/: Nơi chứa các cấu hình bổ sung (thường dùng cho các cấu hình dùng chung hoặc các module nhỏ).
  - /etc/nginx/sites-available/: "Kho" chứa các file cấu hình website. Các file ở đây chưa có hiệu lực ngay.
  - /etc/nginx/sites-enabled/: Chứa các Symbolic Link (liên kết mềm) trỏ từ sites-available sang. Chỉ những file ở đây mới được NGINX thực thi.
  - /var/log/nginx/:
  - access.log: Ghi lại mọi yêu cầu (request) gửi đến server.
  - error.log: Nơi đầu tiên cần kiểm tra khi NGINX gặp lỗi (Debug).
- Thư mục chứa mã nguồn (Web Root):
  - /usr/share/nginx/html/: Thư mục mặc định của gói cài đặt NGINX.
  - /var/www/html/: Thư mục chuẩn thường dùng trong thực tế triển khai web.
<a name="ii"></a>
## II. Cơ chế hoạt động của NGINX
- Khi một Request đi tới, NGINX thực hiện phân loại theo các bước:
- Kiểm tra Port (Listen): Quét qua các file trong sites-enabled để tìm block server có port trùng với request.
- Kiểm tra Server Name: Nếu nhiều file cùng nghe một port (ví dụ port 80), NGINX sẽ đối chiếu header Host của request với directive server_name.
- Xử lý Location: Sau khi tìm được block server phù hợp, NGINX sẽ chạy các luật trong block location để trả về file tĩnh hoặc chuyển tiếp (proxy) dữ liệu.
<a name="iii"></a>
## III. Giải thích file cấu hình
<a name="1"></a>
### 1. MAIN BLOCK.
- `User www-data;` : Cấu hình quy định worker processes được chạy với tài khoản nào, ở đây là nginx
- `work_processes auto` : Câu hình chỉ ra rằng web server được xử lsy bằng 1 CPU Core (processor), giá trị này tương ứng với số CPU Core có trên server. Để kiểm tra số lượng CPU Core trên máy chủ sử dụng lệnh
  ```sh
  nproc

  # hoặc

  cat /proc/cpuinfo
- `error_log /var/log/nginx/error.log;` đường dẫn đến file log của nginx
- `pid /run/nginx.pid;` Số PID của master process, nginx sử dụng master process để quản lý worker process
<a name="2"></a>
### 2. Event Block
- `worker_connections 1024;` Giá trị liên quan đến worker processes, 1024 có nghĩa mỗi worker process sẽ chịu tải là 1024 kết nối cùng lúc. Nếu chúng ta có 2 worker process thì khả năng chịu tải của server là 2048 kết nối tại một thời điểm
<a name="3"></a>
### 3. HTTP Block
- log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    ```sh
    Định nghĩa một mẫu log có tên là main được sử dụng bởi access_log , các thông tin được đưa vào file tương ứng với các 
    biến như $remote_addr, $remote_user ,....
    ```

- access_log  /var/log/nginx/access.log  main;

    ```sh
    Chỉ ra đường dẫn tới file log .
    ```

- `sendfile on;` Cấu hình này gọi đến function sendfile để xử lý việc truyền file .

- `tcp_nopush on;` 

- `tcp_nodelay on;`

- `keepalive_timeout   65;` Xác định thời gian chờ trước khi đóng 1 kết nối, ở đây là 65s.

-  include /etc/nginx/mime.types;
   default_type        application/octet-stream;

    ```sh
    Gọi tới file chứa danh sách các file extension trong nginx
    ```

- `types_hash_max_size 2048;`
  