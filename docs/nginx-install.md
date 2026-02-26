# Tổng quan và cài đặt NGINX

## Mục lục

- [1. Tổng quan về NGINX](#1)
- [2. Cài đặt NGINX](#2)
  <a name="1"></a>

## 1. Tổng quan về NGINX

### 1.1 Khái niệm

NGINX là một phần mềm mã nguồn mở chuyên dụng cho:

- Web Serving: Phục vụ nội dung tĩnh (HTML, CSS, JS, Image).
- Reverse Proxy: Làm trung gian điều phối request từ client đến các server phía sau.
- Load Balancing: Cân bằng tải để không server nào bị quá tải.
- API Gateway: Quản lý và bảo mật các luồng API.

### 1.2 Các thành phần cốt lõi của NGINX

Các thành phần chính:

- Master Process:
  - Đọc và kiểm tra cấu hình
  - Quản lý các Worker Process
  - Thực hiện các thao tác đặc quyền
  - Hỗ trợ Hot Deployment
- Worker Process
  - Thực hiện xử lý các request thực tế
  - Mỗi Worker là đơn luồng và chạy trên một vòng lặp sự kiến (Event Loop)
  - Số lượng Worker thường được cấu hình bằng nhân CPU để tận dụng tối đa sức mạng phần cứng

Các cơ chế Event-Driven:

- Event-Driven: thay vì đợi một tác vụ I/O hoàn thành, Worker sẽ đăng ký một sự kiện và chuyển sang xử lý request khác ngay lập tức
- Epoll: NGINX sử dụng các systemcall hiện đại của OS để theo dõi hàng ngàn kết nối đồng thời mà gần như không tốn tài nguyên CPU cho việc chuyển đổi ngữ cảnh
  <a name="2"></a>

## 2. Cài đặt NGINX Nginx có sẵn trong repo mặc định của Ubuntu nên chỉ cần cài đặt bằng `apt` - Cập nhật hệ thống ```bash apt update

- Cài đặt NGINX
  ```bash
  apt install -y nginx
  ```
- Khời động nginx:
  ```bash
  systemctl start nginx
  ```
- Cấu hình nginx tự khởi động cùng hệ thống
  ```bash
  systemctl enable nginx
  ```
- Kiểm tra trạng thái nginx
  ```bash
  systemctl status nginx
  ```
- Truy cập vào địa chỉ để kiểm tra:
  ![ubuntu_install](/imgs/ubuntu_install.png)
