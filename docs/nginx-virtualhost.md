# Cách tạo virtual host (Server Block)

# Mục lục

- [Virtual Host là gì?](#virtual-host)
- [Cấu hình nhiều sử dụng nhiều Virtual Host trên cùng một server](#multiple-host)
- #### Virtual Host là gì?
  - "Virtual Host" là một kỹ thuật cho phép nhiều Website có thể dùng chung một địa chỉ ip. Thuật ngữ này được sử dụng với các Website sử dụng Apache server. Trong các website sử dụng Nginx server thì nó được gọi là Server Block
  - Đây là kỹ thuật dùng để cấu hình cho webserver khi bạn muốn có nhiều website với các tên miền khác nhau được sử dụng chung trên một máy chủ
  nginx-webserver
- #### Cấu hình sử dụng nhiều Virtual Host trên cùng một server
  - Đây là cách cấu hình sao cho nhiều website sử dụng chung một ip duy nhất. Theo các bước sau (Sử dụng Ubuntu):
    - Bước 1: Cài đặt Nginx
    - Bước 2: Cấu hình domain chính cho server
    - Bước 3: Cấu hình tạo các virtual host (ServerBlock)
    - Bước 4: Thực hiện trỏ host trên client đến kiểm tra kết quả
  - Bước 1: Cài đặt Nginx: Xem chi tiết [tại đây](/docs/nginx-install.md)
  - Bước 2: Cấu hình domain chính cho server
    - Ta tiến hành sửa nội dung file cấu hình chính nginx tại `/etc/nginx/sites-available/default`. Tìm tới dòng có nội dung `server_name ...;` trong vùng cấu hình của http. Thay ... bằng tên miền mà bạn muốn sử dụng. Ví dụ
    ```bash
    server {
        ...
        server_name www.test-nginx.comm test-nginx.com;
    }
    ```
  - Bước 3: Cấu hình tạo các Virtual Host tại `/etc/nginx/sites-available/`
    - 1: Tạo Virtual Host 1
    `vi /etc/nginx/sites-available/vhost1`
    Tiếp theo ta cần thêm nội dung cấu hình cho file vừa tạo trên với nội dung:
    ```bash
    server {
      listen 80;
      server_name vhost1.com www.vhost1.com;
      root /var/www/vhost1;
      index index.html;
    }
    ```
    Tạo thư mục chứa website cho virtual host này:
    ```bash
    #Tạo thư mục chứa website
    mkdir /var/www/vhost1
    #Phân quyền sở hữu cho user
    chown www-data:www-data -R /var/www/vhost1
    ```
    Tạo file index.html:
    ```bash
    #tạo file index.html
    vi /var/www/vhost1/index.html

    #Viết file HTML đơn giản
    <DOCTYPE html>
    <html>
      <head>
        <title>www.vhost1.com</title>
      </head>
      <body>
        <h1>Success: You Have Set Up a Virtual Host</h1>
        <h1>www.vhost1.com and vhost1.com</h1>
      </body>
    </html>
    ```
    Link vhost1 trong `/etc/nginx/sites-available/` với `/etc/nginx/sites-enabled/`
    ```bash
    ln -s /etc/nginx/sites-available/vhost1 /etc/nginx/sites-enabled/
    ```
    - 2: Tạo Virtual Host 2
    `vi /etc/nginx/sites-available/vhost2`
    Tiếp theo ta cần thêm nội dung cấu hình cho file vừa tạo trên với nội dung:
    ```bash
    server {
      listen 80;
      server_name vhost2.com www.vhost2.com;
      root /var/www/vhost2;
      index index.html;
    }
    ```
    Tạo thư mục chứa website cho virtual host này:
    ```bash
    #Tạo thư mục chứa website
    mkdir /var/www/vhost2
    #Phân quyền sở hữu cho user
    chown www-data:www-data -R /var/www/vhost2
    ```
    Tạo file index.html:
    ```bash
    #tạo file index.html
    vi /var/www/vhost2/index.html

    #Viết file HTML đơn giản
    <DOCTYPE html>
    <html>
      <head>
        <title>www.vhost2.com</title>
      </head>
      <body>
        <h1>Success: You Have Set Up a Virtual Host</h1>
        <h1>www.vhost2.com and vhost2.com</h1>
      </body>
    </html>
    ```
    Link vhost2 trong `/etc/nginx/sites-available/` với `/etc/nginx/sites-enabled/`
    ```bash
    ln -s /etc/nginx/sites-available/vhost2 /etc/nginx/sites-enabled/
    ```
  - Bước 4: Tiến hành cấu hình trỏ host trên client để kiểm tra bằng việc thêm nội dung sau vào file `C:\Windows\System32\drivers\etc/hosts` trên client theo dạng:
  ip-address server_name[s]
  - Kết quả:
    - Khi truy cập test-nginx.com hoặc www.test-nginx.com
    ![virtual-host-1](/imgs/virtual-host-1.png)
    - Khi truy cập vhost1.com hoặc www.vhost1.com
    ![virtual-host2](/imgs/virtual-host2.png)
    - Khi truy cập vhost2.com hoặc www.vhost3.com
    ![virtual-host3](/imgs/virtual-host3.png)

