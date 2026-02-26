# Cấu hình Nginx làm reverse proxy cho apache

---

## Mục lục

- [Mục đích](#1)
- [Mô hình](#2)
- [Cài đặt chi tiết](#3)

## <a name="1">Mục đích</a>

---

- Apache là một `Open Source Webserver` phổ biến nhất hiện nay bởi vì có rất nhiều software hỗ trợ -> Điều Nginx không có
- Tuy nhiên, có một nhược điểm của Apache đó là nó kém linh hoạt, xử lý khá chậm và chiếm rất nhiều bộ nhớ mỗi khi cần xử lý dữ liệu.
- Còn với nginx luôn có khả năng xử lý nhanh hơn apache, linh hoạt hơn và nhẹ hơn apache rất nhiều. Cách cấu hình nginx theo đánh giá là đơn giản và gọn gàng hơn.
- Nginx rất đa nhiệm do đó để tối ưu hóa hơn cho webserver người ta thường sử dụng song hành Nginx và Apache.

## <a name="2">Mô hình</a>

---

![Reverse Proxy Model](/imgs/reverse-proxy-nginx.jpg)

## <a name="3">Cài đặt chi tiết</a>

---

- Reverse Proxy đơn giản là việc sử dụng Nginx để chuyển tiếp request của client sang apache chứ request của client không đi thằng đến Apache.
- Cấu hình ở bài Lab này sử dụng 2 VM Ubuntu (Nginx: 10.1.1.136 Và Apache: 10.1.1.132).Gồm các bước sau:
  - Bước 1: Cài đặt Nginx trên VM1
  - Bước 2: Cấu hình Reverse Proxy trong `/etc/nginx/sites-available/default`
  - Bước 3: Cài đặt Apache trên VM2

---

- Bước 1: Cài đặt Nginx ([tại đây](/docs/nginx-install.md))
- Bước 2: Chỉnh sửa file cấu hình `/etc/nginx/sites-available/default`

  ```bash
  #Chỉnh sửa file cấu hình với vim
  vi /etc/nginx/sites-available/default

  #Chỉnh sửa file theo sau
  server {
      listen 80 default_server;
      listen [::]:80 default_server;
      server_name _;

      proxy_redirect off;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;

      location / {
          proxy_pass http://10.1.1.132/;
      }

  }
  #Kiểm tra bằng
  nginx -t
  #Khởi động lại nginx
  systemctl restart nginx
  ```

- Bước 3: Cài đặt Apache
  - Cài đặt apache2

  ```bash
      sudo apt update
      sudo apt install apache2 -y
  ```

  - Kiểm tra
    ![apache](/imgs/apache2.png)

- Kết quả đạt được
  - Khi truy cập vào 10.1.1.136:
    ![reverse proxy](/imgs/reverse-proxy.png)
