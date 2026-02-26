#Hướng dẫn cấu hình Load Balancing với NGINX trên Ubuntu

Tài liệu này hướng dẫn chi tiết cách thiết lập cân bằng tải (Load Balancing) sử dụng NGINX trên hệ điều hành Ubuntu, tối ưu cho môi trường sử dụng apt, apache2, và ufw.

##1. Mô hình triển khai

       Client (Trình duyệt)
               |
               v
    NGINX (Load Balancer IP: 10.10.20.30)
      /                 \
     v                   v
Apache Backend 1      Apache Backend 2
(10.10.20.10)         (10.10.20.20)


- Mô tả: NGINX đóng vai trò là Reverse Proxy và Load Balancer, tiếp nhận các yêu cầu từ Client và phân phối chúng đến các máy chủ Apache phía sau (Backends).

##2. Cấu hình các Server Backend (Apache)
Thực hiện các bước sau trên cả máy chủ 10.10.20.10 và 10.10.20.20.
###2.1. Cài đặt Apache2
```
sudo apt update
sudo apt install apache2 -y
```
###2.2. Kích hoạt dịch vụ
```
sudo systemctl start apache2
sudo systemctl enable apache2
```
###2.3. Tạo nội dung thử nghiệm
- Tạo file index.html khác nhau trên mỗi máy để dễ dàng kiểm tra quá trình Load Balancing.
- Trên Backend 1:
    ```
    echo "<h1>Chào mừng đến với WEB 1 (Backend 1)</h1>" | sudo tee /var/www/html/index.html
    ```

- Trên Backend 2:
    ```
    echo "<h1>Chào mừng đến với WEB 2 (Backend 2)</h1>" | sudo tee /var/www/html/index.html
    ```

##3. Cài đặt và Cấu hình NGINX (Load Balancer)
Thực hiện trên máy chủ 10.10.20.30.

###3.1. Cài đặt NGINX

sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx


3.2. Cấu hình Firewall (UFW)

sudo ufw allow 'Nginx Full'


4. Các thuật toán cân bằng tải

Tất cả các cấu hình dưới đây được thực hiện trong file /etc/nginx/nginx.conf hoặc tạo file mới trong /etc/nginx/conf.d/loadbalancer.conf.

4.1. Weighted Load Balancing (Trọng số)

Phù hợp khi các server backend có cấu hình phần cứng không đồng nhất.

http {
    upstream backends {
        # Server 1 nhận 3 phần request, Server 2 nhận 2 phần
        server 10.10.20.10:80 weight=3;
        server 10.10.20.20:80 weight=2;
    }

    server {
        listen 80 default_server;

        location / {
            proxy_pass http://backends;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}


4.2. Round Robin (Mặc định)

NGINX luân phiên gửi request đến từng server theo tỉ lệ 1:1.

upstream backends {
    server 10.10.20.10:80;
    server 10.10.20.20:80;
}


4.3. Least Connection (Ít kết nối nhất)

Gửi request đến server đang có ít phiên làm việc (session) nhất. Rất hiệu quả cho các ứng dụng có thời gian xử lý request dài.

upstream backends {
    least_conn;
    server 10.10.20.10:80;
    server 10.10.20.20:80;
}


4.4. Passive Health Check

NGINX tự động theo dõi trạng thái phản hồi của backend để tạm thời loại bỏ server bị lỗi.

upstream backends {
    # Nếu fail 3 lần trong 5 giây, tạm ngưng gửi request trong vòng tiếp theo
    server 10.10.20.10:80 max_fails=3 fail_timeout=5s;
    server 10.10.20.20:80 max_fails=3 fail_timeout=5s;
}


5. Kiểm tra kết quả

Sau khi thay đổi cấu hình, luôn kiểm tra cú pháp và khởi động lại NGINX:

sudo nginx -t
sudo systemctl restart nginx


Mở trình duyệt và truy cập: http://10.10.20.30. Hãy nhấn F5 (Refresh) liên tục để thấy sự thay đổi giữa nội dung của WEB 1 và WEB 2.

6. Lưu ý quan trọng cho Production

Session Persistence: Nếu ứng dụng yêu cầu giữ phiên đăng nhập, hãy cân nhắc dùng thuật toán ip_hash hoặc sử dụng Redis/Memcached để lưu session tập trung.

Bảo mật: Backend Apache nên cấu hình chỉ cho phép nhận kết nối từ IP của Load Balancer.

High Availability (HA): Để tránh NGINX trở thành "Single Point of Failure", nên kết hợp với Keepalived để tạo mô hình Active-Passive với IP ảo (VIP).

Monitoring: Luôn cài đặt các công cụ giám sát (như Prometheus/Grafana) để theo dõi sức khỏe của các node backend.

Tài liệu tham khảo: NGINX Load Balancing Official Docs