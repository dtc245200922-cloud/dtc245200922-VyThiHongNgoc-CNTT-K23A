# Hồng Ngọc Bags

## Giai đoạn 1: Website, database và reverse proxy

Các dịch vụ: WordPress, MySQL, phpMyAdmin và Nginx.

### Khởi chạy

Sao chép `.env.example` thành `.env`, điền mật khẩu riêng:

```powershell
Copy-Item .env.example .env
docker compose config --quiet
docker compose up -d
```

Nếu đã có `.env`, giữ nguyên file hiện tại.

### Truy cập

- Website: http://localhost:8280
- phpMyAdmin: http://localhost:8182
- Database: wpdb
- Tài khoản database: wpuser
- Mật khẩu: WORDPRESS_DB_PASSWORD trong .env

Với database mới, hoàn tất cài đặt WordPress trên trình duyệt.
Nội dung trang chủ tùy chỉnh cần được thiết lập riêng.

### Bảo mật

- Nginx reverse proxy có security headers.
- Chặn truy cập wp-config.php và các tệp ẩn.
- MySQL sử dụng mạng nội bộ riêng.
- Không công khai cổng MySQL.
- Thông tin đăng nhập thực tế được Git bỏ qua.

### Kiểm tra

```powershell
docker compose ps -a
curl.exe -I http://localhost:8280
curl.exe -I http://localhost:8280/wp-config.php
```

Không chạy `docker compose down -v` khi cần giữ dữ liệu.