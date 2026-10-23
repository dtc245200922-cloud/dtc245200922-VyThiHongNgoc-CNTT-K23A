# Hồng Ngọc Bags — Triển khai WordPress bằng Docker Compose

## 1. Giới thiệu

Website giới thiệu sản phẩm túi xách, sử dụng WordPress và MySQL.
Hệ thống tích hợp Nginx reverse proxy, phpMyAdmin, giám sát
Prometheus + Grafana và log tập trung Loki + Promtail.

## 2. Các thành phần

| Thành phần | Chức năng |
| --- | --- |
| WordPress | Website giới thiệu sản phẩm |
| MySQL | Lưu dữ liệu WordPress |
| phpMyAdmin | Quản lý database |
| Nginx | Reverse proxy và security headers |
| Prometheus | Thu thập metrics |
| Grafana | Hiển thị dashboard và truy vấn log |
| Node Exporter | Metrics môi trường máy chủ |
| cAdvisor | Metrics tài nguyên container |
| MySQL Exporter | Metrics MySQL |
| Loki | Lưu trữ, truy vấn log |
| Promtail | Thu thập log container gửi đến Loki |

Trên Docker Desktop, metrics máy chủ phản ánh môi trường Linux
chạy Docker, không phải toàn bộ hệ điều hành Windows.

## 3. Yêu cầu

- Docker Desktop chạy Linux containers.
- Docker Compose v2.
- Git.
- Các cổng 8280, 8182, 9190, 3100, 8181 và 3200 còn trống.

## 4. Chuẩn bị cấu hình

Trong PowerShell tại thư mục dự án:

```powershell
Copy-Item .env.example .env
Copy-Item init.sql.example init.sql
Copy-Item mysql-exporter.cnf.example mysql-exporter.cnf
```

Chỉ thực hiện các lệnh sao chép khi chưa có file cấu hình thực tế.
Không ghi đè cấu hình đang sử dụng.

Điền mật khẩu riêng trong `.env`.

Thay `CHANGE_ME_EXPORTER_PASSWORD` trong cả `init.sql`
và `mysql-exporter.cnf` bằng cùng một mật khẩu mạnh.

Không đưa các file chứa mật khẩu thực tế lên GitHub.

Lưu ý: script `init.sql` chỉ tự chạy khi MySQL khởi tạo
thư mục dữ liệu mới. Thay đổi `.env` không tự đổi mật khẩu
tài khoản trong database đã có dữ liệu.

## 5. Khởi chạy

```powershell
docker compose config --quiet
docker compose up -d
docker compose ps -a
```

Docker Compose tự kết hợp `docker-compose.yml`
và `docker-compose.override.yml`.

Nếu có dịch vụ lỗi:

```powershell
docker compose logs --tail=50 TEN_DICH_VU
```

## 6. Địa chỉ truy cập

| Dịch vụ | Địa chỉ |
| --- | --- |
| Website qua Nginx | http://localhost:8280 |
| phpMyAdmin | http://localhost:8182 |
| Prometheus | http://localhost:9190 |
| Grafana | http://localhost:3100 |
| cAdvisor | http://localhost:8181/containers/ |
| Loki readiness | http://localhost:3200/ready |

### WordPress

Khi triển khai với database mới, mở website và hoàn tất cài đặt
WordPress bằng tài khoản quản trị do người triển khai tự tạo.

Nội dung và giao diện đã chỉnh trong WordPress được lưu trong
database và tệp WordPress; clone repository cấu hình không tự
khôi phục trang chủ đã thiết kế.

### phpMyAdmin

- Máy chủ: `mysql`.
- Tài khoản: `wpuser`.
- Mật khẩu: giá trị `WORDPRESS_DB_PASSWORD` trong `.env`.
- Database: `wpdb`.

### Grafana

Với volume Grafana mới, đăng nhập lần đầu bằng `admin/admin`
và đổi mật khẩu khi được yêu cầu.
Nếu đã có volume, sử dụng tài khoản đang được lưu trong Grafana.

Thêm các data source:

- Prometheus: `http://prometheus:9090`.
- Loki: `http://loki:3100`.

Các URL trên dùng mạng nội bộ Docker.

## 7. Kiểm tra giám sát

Mở `http://localhost:9190/targets`.

Kiểm tra các target đã cấu hình:

- prometheus
- node-exporter
- cadvisor
- mysql

Các target cần ở trạng thái UP.

Một số truy vấn PromQL:

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

Bộ lọc tên container và cấu hình Promtail hiện sử dụng
tên project `docker-monitoring`.
Nếu đổi tên project, cần cập nhật các bộ lọc tương ứng.

Dashboard Grafana cần được tạo hoặc import trên lần triển khai mới.

## 8. Truy vấn log bằng LogQL

Trong Grafana, mở Explore, chọn Loki và chọn khoảng thời gian phù hợp.

Toàn bộ log Nginx:

```logql
{service="nginx"}
```

Lọc yêu cầu GET:

```logql
{service="nginx"} |= "GET "
```

Lọc phản hồi HTTP 200:

```logql
{service="nginx"} |= "\" 200 "
```

Tạo log truy cập mới:

```powershell
curl.exe -o NUL http://localhost:8280/
```

Đợi vài giây rồi chạy lại truy vấn.

## 9. Hardening

Các biện pháp đã cấu hình:

- MySQL sử dụng mạng database riêng có `internal: true`.
- Không công khai cổng MySQL và cổng trực tiếp WordPress.
- Các cổng giao diện quản trị, giám sát bind về 127.0.0.1.
- Grafana và Prometheus chạy bằng người dùng non-root.
- Bỏ Linux capabilities của một số dịch vụ.
- Áp dụng no-new-privileges cho các dịch vụ phù hợp.
- Mount các file cấu hình ở chế độ chỉ đọc.
- Nginx có security headers, ẩn phiên bản và chặn tệp nhạy cảm.
- Không lưu mật khẩu thực tế trong Git.

Kiểm tra:

```powershell
curl.exe -I http://localhost:8280
curl.exe -I http://localhost:8280/wp-config.php
docker compose exec grafana id
docker compose exec prometheus id
```

Đường dẫn `/wp-config.php` cần trả về HTTP 403.
UID Grafana và Prometheus cần khác 0.

Giới hạn hiện tại:

- Website sử dụng HTTP, chưa cấu hình HTTPS.
- Không phải tất cả container đều chạy non-root.
- cAdvisor còn sử dụng privileged.
- Promtail truy cập Docker socket, có quyền hạn lớn.
- Promtail đã hết hỗ trợ; sử dụng theo yêu cầu bài thực hành.
- Cần bổ sung exporter Nginx để giám sát metrics web server.

## 10. Dừng và chạy lại

Dừng dịch vụ:

```powershell
docker compose stop
```

Chạy lại:

```powershell
docker compose start
```

Không sử dụng `docker compose down -v` khi cần giữ dữ liệu,
vì tùy chọn `-v` xóa các volume do Compose quản lý.