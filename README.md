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

## Giai đoạn 2: Giám sát bằng Prometheus và Grafana

### Chuẩn bị tài khoản exporter

Với bản triển khai mới, sao chép hai file mẫu:

```powershell
Copy-Item init.sql.example init.sql
Copy-Item mysql-exporter.cnf.example mysql-exporter.cnf
```

Thay mật khẩu mẫu trong hai file bằng cùng một mật khẩu mạnh.
Không ghi đè các file cấu hình thực tế nếu chúng đã tồn tại.

`init.sql` chỉ tự chạy khi MySQL khởi tạo dữ liệu lần đầu.
Nếu database đã tồn tại, cần tạo tài khoản exporter bằng SQL
với quyền quản trị trước khi sử dụng MySQL Exporter.

### Khởi chạy

```powershell
docker compose config --quiet
docker compose up -d
```

### Truy cập

- Prometheus: http://localhost:9190
- Grafana: http://localhost:3100
- cAdvisor: http://localhost:8181/containers/

Trong Grafana, thêm nguồn dữ liệu Prometheus với URL:

```text
http://prometheus:9090
```

Với volume Grafana mới, đăng nhập lần đầu bằng admin/admin
và đổi mật khẩu. Với volume đã tồn tại, sử dụng tài khoản hiện tại.

### Kiểm tra

Mở http://localhost:9190/targets và kiểm tra các target UP:
prometheus, node-exporter, cadvisor, mysql.

Các truy vấn PromQL:

```promql
mysql_up
```

```promql
mysql_global_status_threads_connected
```

```promql
sum by (name) (
  rate(container_cpu_usage_seconds_total{
    job="cadvisor",
    name=~"docker-monitoring-.*"
  }[5m])
) * 100
```

```promql
container_memory_working_set_bytes{
  job="cadvisor",
  name=~"docker-monitoring-.*"
} / 1024 / 1024
```

Bộ lọc container sử dụng tên project docker-monitoring.
Dashboard cần được tạo hoặc import trên lần triển khai mới.

Node Exporter trên Docker Desktop phản ánh môi trường Linux
chạy Docker. cAdvisor còn sử dụng privileged.
Metrics Nginx cần được bổ sung bằng exporter riêng.