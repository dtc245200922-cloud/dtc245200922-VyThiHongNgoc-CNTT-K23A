# WEBSITE QUẢNG BÁ SẢN PHẨM(WORDPRESS)

## 1. Thông tin sinh viên và đề tài

- Họ tên: Vy Thị Hồng Ngọc.
- Mã sinh viên: DTC245200922.
- Lớp: CNTT K23A.
- Học phần: Triển khai và Quản trị Hệ thống Phần mềm.
- Đề tài: WEBSITE QUẢNG BÁ SẢN PHẨM(WORDPRESS)
## 2. Giới thiệu hệ thống

Website giới thiệu sản phẩm túi xách, sử dụng WordPress và MySQL.

Hệ thống tích hợp Nginx reverse proxy, phpMyAdmin, Prometheus,
Grafana, Loki và Promtail. Tất cả dịch vụ được triển khai bằng
Docker Compose.

Tên và giá sản phẩm trên website là dữ liệu minh họa.
Website chưa tích hợp giỏ hàng và thanh toán.

## 3. Các thành phần

| Thành phần | Chức năng |
| --- | --- |
| WordPress | Website giới thiệu sản phẩm |
| MySQL | Lưu dữ liệu WordPress |
| phpMyAdmin | Quản lý database |
| Nginx | Reverse proxy và security headers |
| Nginx Exporter | Chuyển thống kê Nginx thành metrics Prometheus |
| Prometheus | Thu thập metrics |
| Grafana | Hiển thị dashboard và truy vấn log |
| Node Exporter | Metrics môi trường máy chủ |
| cAdvisor | Metrics tài nguyên container |
| MySQL Exporter | Metrics MySQL |
| Loki | Lưu trữ và truy vấn log |
| Promtail | Thu thập log container gửi đến Loki |

Luồng truy cập website:

Trình duyệt → Nginx → WordPress → MySQL.

Luồng giám sát:

Exporter / cAdvisor → Prometheus → Grafana.

Luồng log:

Log container → Promtail → Loki → Grafana Explore.

Trên Docker Desktop, metrics máy chủ phản ánh môi trường Linux
chạy Docker, không phải toàn bộ hệ điều hành Windows.

## 4. Các file chính

| File hoặc thư mục | Nội dung |
| --- | --- |
| docker-compose.yml | Cấu hình ứng dụng, database và monitoring |
| docker-compose.override.yml | Cấu hình phpMyAdmin, Nginx, Nginx Exporter, Loki và Promtail |
| nginx/default.conf | Reverse proxy, security headers và stub_status |
| prometheus.yml | Danh sách target thu thập metrics |
| loki-config.yml | Cấu hình Loki |
| promtail-config.yml | Cấu hình thu thập log container |
| .env.example | Mẫu biến môi trường |
| init.sql.example | Mẫu tạo tài khoản exporter |
| mysql-exporter.cnf.example | Mẫu kết nối MySQL Exporter |
| website/homepage.html | Mã giao diện trang chủ |
| grafana/dashboards/wordpress-monitoring.json | Dashboard Grafana đã xuất |
| .gitignore | Loại trừ mật khẩu và bản sao lưu khỏi Git |

Các file cấu hình thực tế `.env`, `init.sql`,
`mysql-exporter.cnf` được tạo tại máy triển khai và được
`.gitignore` loại trừ.

## 5. Yêu cầu môi trường

- Docker Desktop chạy Linux containers.
- Docker Compose v2.
- Git.
- Các cổng 8280, 8182, 9190, 3100, 8181 và 3200 còn trống.
- Kết nối Internet để tải Docker images, ảnh và phông chữ.

Cấu hình mount tài nguyên máy chủ của cAdvisor và Node Exporter
được sử dụng trong môi trường Docker Desktop Linux.
Khi triển khai trên môi trường khác, cần kiểm tra lại các đường dẫn mount.

## 6. Chuẩn bị cấu hình

Clone repository:

```powershell
git clone https://github.com/dtc245200922-cloud/dtc245200922-VyThiHongNgoc-CNTT-K23A.git docker-monitoring
cd docker-monitoring
```

Giữ tên project là `docker-monitoring` vì các bộ lọc container
và cấu hình Promtail đang sử dụng tên này.

Tạo các file cấu hình thực tế:

```powershell
Copy-Item .env.example .env
Copy-Item init.sql.example init.sql
Copy-Item mysql-exporter.cnf.example mysql-exporter.cnf
```

Chỉ sao chép khi chưa có các file cấu hình thực tế.
Không ghi đè cấu hình đang sử dụng.

Điền mật khẩu riêng vào `.env` cho:

- `MYSQL_ROOT_PASSWORD`: mật khẩu quản trị MySQL.
- `WORDPRESS_DB_PASSWORD`: mật khẩu tài khoản `wpuser`.

Thay `CHANGE_ME_EXPORTER_PASSWORD` trong cả `init.sql`
và `mysql-exporter.cnf` bằng cùng một mật khẩu riêng cho exporter.

Sử dụng mật khẩu mạnh, khác nhau giữa các tài khoản.
Không đưa các file chứa mật khẩu thực tế lên GitHub.

Lưu ý:

- `init.sql` chỉ tự chạy khi MySQL khởi tạo thư mục dữ liệu mới.
- Thay đổi `.env` không tự đổi mật khẩu trong database đã có dữ liệu.
- Khi đổi mật khẩu tài khoản hiện có, phải cập nhật trong MySQL
  và cấu hình dịch vụ sử dụng tài khoản đó.
- Mật khẩu quản trị WordPress được tạo riêng khi cài đặt website;
  không phải mật khẩu tài khoản MySQL.

## 7. Khởi chạy

Trong thư mục dự án:

```powershell
docker compose config --quiet
docker compose up -d
docker compose ps -a
```

Docker Compose tự kết hợp `docker-compose.yml`
và `docker-compose.override.yml`.

Nếu có dịch vụ lỗi, thay `TEN_DICH_VU` bằng tên dịch vụ cần kiểm tra:

```powershell
docker compose logs --tail=50 TEN_DICH_VU
```

Ví dụ:

```powershell
docker compose logs --tail=50 wordpress mysql nginx
```

MySQL cần thời gian khởi động ở lần chạy đầu.
Nếu WordPress báo lỗi kết nối database, kiểm tra log và đợi MySQL sẵn sàng.

## 8. Địa chỉ truy cập

| Dịch vụ | Địa chỉ |
| --- | --- |
| Website qua Nginx | http://localhost:8280 |
| Quản trị WordPress | http://localhost:8280/wp-admin/ |
| phpMyAdmin | http://localhost:8182 |
| Prometheus | http://localhost:9190 |
| Prometheus Targets | http://localhost:9190/targets |
| Grafana | http://localhost:3100 |
| cAdvisor | http://localhost:8181/containers/ |
| Loki readiness | http://localhost:3200/ready |

Cổng thống kê Nginx `8080` và cổng Nginx Exporter `9113`
chỉ được sử dụng trong mạng Docker, không publish ra máy chủ.

### 8.1. Cài đặt và khôi phục giao diện WordPress

Với database mới:

1. Mở http://localhost:8280.
2. Hoàn tất cài đặt WordPress và tạo tài khoản quản trị riêng.
3. Đăng nhập tại http://localhost:8280/wp-admin/.
4. Chọn Pages / Trang → Add Page / Thêm trang.
5. Thêm khối Custom HTML / HTML tùy chỉnh.
6. Mở file `website/homepage.html` trong repository.
7. Copy toàn bộ nội dung vào khối HTML tùy chỉnh.
8. Xuất bản trang.
9. Vào Settings / Cài đặt → Reading / Đọc.
10. Chọn trang tĩnh và đặt trang vừa tạo làm trang chủ.
11. Mở lại http://localhost:8280 để kiểm tra.

Có thể đặt tên trang là `Trang chủ`. CSS của giao diện có
quy tắc ẩn tiêu đề trang khi hiển thị website.

Nếu trình chỉnh sửa mở cửa sổ có các tab HTML, CSS và JavaScript,
dán toàn bộ nội dung `homepage.html` vào tab HTML vì file đã chứa
CSS trong các thẻ `<style>`.

Giao diện trên laptop có banner hai cột và bốn sản phẩm trên một hàng.
Trên màn hình nhỏ, bố cục tự chuyển thành ít cột hơn.

Ảnh sản phẩm sử dụng liên kết bên ngoài và phông chữ tải từ Google Fonts.
Việc hiển thị các tài nguyên này phụ thuộc kết nối Internet và nguồn cung cấp.

Clone repository không tự khôi phục nội dung WordPress trong database.
File HTML dùng để dựng lại trang chủ; dữ liệu khác cần bản sao lưu riêng.

### 8.2. Đăng nhập phpMyAdmin

- Địa chỉ: http://localhost:8182.
- Máy chủ được cấu hình là `mysql`.
- Tài khoản: `wpuser`.
- Mật khẩu: giá trị `WORDPRESS_DB_PASSWORD` trong `.env`.
- Database: `wpdb`.

Tài khoản `wpuser` có quyền trên database `wpdb`,
không phải tài khoản quản trị toàn bộ MySQL.

### 8.3. Thiết lập Grafana

Với volume Grafana mới, đăng nhập lần đầu bằng `admin/admin`
và đổi mật khẩu khi được yêu cầu.

Nếu đã có volume, sử dụng tài khoản đang được lưu trong Grafana.

Thêm các data source:

| Loại | URL nội bộ Docker |
| --- | --- |
| Prometheus | http://prometheus:9090 |
| Loki | http://loki:3100 |

Chọn Save & test để kiểm tra kết nối.

Các URL trên được Grafana truy cập qua mạng Docker.
Không thay bằng `localhost` trong cấu hình data source.

### 8.4. Import dashboard

1. Mở Grafana → Dashboards.
2. Chọn New → Import hoặc mục Import trên giao diện đang sử dụng.
3. Tải lên file `grafana/dashboards/wordpress-monitoring.json`.
4. Chọn data source Prometheus đã tạo nếu được yêu cầu.
5. Hoàn tất import và kiểm tra dữ liệu.

Nếu xuất hiện lỗi không tìm thấy data source, chỉnh các panel
để sử dụng data source Prometheus hiện có rồi lưu dashboard.

File JSON được xuất từ Grafana của dự án.
Khi sử dụng phiên bản Grafana khác, cần kiểm tra khả năng tương thích.

Dashboard gồm các panel giám sát:

- Trạng thái MySQL.
- CPU từng container.
- RAM từng container.
- Số kết nối MySQL.
- Trạng thái Nginx.

Dashboard được import thủ công, chưa cấu hình tự động provisioning.

## 9. Kiểm tra giám sát

Mở http://localhost:9190/targets.

Kiểm tra năm target:

- prometheus
- node-exporter
- cadvisor
- mysql
- nginx

Các target cần ở trạng thái `UP`.

`UP` xác nhận Prometheus truy cập được endpoint metrics.
Để kiểm tra kết nối từ exporter tới dịch vụ, xem thêm
`mysql_up` và `nginx_up`.

### Trạng thái MySQL

```promql
mysql_up
```

Kết quả mong đợi: `1`.

### Số kết nối MySQL

```promql
mysql_global_status_threads_connected
```

### Trạng thái Nginx

```promql
nginx_up
```

Kết quả mong đợi: `1`.

### Số kết nối đang hoạt động của Nginx

```promql
nginx_connections_active
```

### Tốc độ yêu cầu HTTP qua Nginx

```promql
rate(nginx_http_requests_total[5m])
```

Đơn vị: yêu cầu mỗi giây.
Cần đủ mẫu metrics trong khoảng thời gian truy vấn.

### CPU từng container

```promql
sum by (name) (
  rate(container_cpu_usage_seconds_total{
    job="cadvisor",
    name=~"docker-monitoring-.*"
  }[5m])
) * 100
```

Giá trị 100 tương ứng mức sử dụng một lõi CPU,
không phải 100% tổng CPU máy chủ.

### RAM từng container, đơn vị MiB

```promql
container_memory_working_set_bytes{
  job="cadvisor",
  name=~"docker-monitoring-.*"
} / 1024 / 1024
```

Nếu đổi tên project Compose, cần cập nhật bộ lọc tên container
và cấu hình Promtail tương ứng.

## 10. Truy vấn log bằng LogQL

Trong Grafana:

1. Mở Explore.
2. Chọn data source Loki.
3. Chọn khoảng thời gian phù hợp, ví dụ Last 15 minutes.
4. Chuyển sang chế độ Code và chạy truy vấn.

### Query 1: Toàn bộ log Nginx

```logql
{service="nginx"}
```

### Query 2: Lọc yêu cầu GET

```logql
{service="nginx"} |= "GET "
```

### Query 3: Lọc phản hồi HTTP 200

```logql
{service="nginx"} |= "\" 200 "
```

Truy vấn thứ ba lọc theo định dạng access log hiện tại của Nginx.

Tạo log truy cập mới:

```powershell
curl.exe -o NUL http://localhost:8280/
```

Đợi vài giây rồi chạy lại truy vấn trong Grafana.

Nếu không thấy log:

```powershell
docker compose logs --tail=50 promtail loki nginx
```

Kiểm tra khoảng thời gian truy vấn và tên project
trong cấu hình Docker service discovery của Promtail.

## 11. Hardening

Các biện pháp cấu hình trong dự án:

- MySQL sử dụng mạng database riêng có `internal: true`.
- Không publish cổng MySQL và cổng trực tiếp WordPress.
- Các cổng website, giao diện quản trị và monitoring bind về `127.0.0.1`.
- Grafana và Prometheus chạy bằng người dùng non-root.
- Nginx Exporter được cấu hình non-root và filesystem chỉ đọc.
- Loại bỏ Linux capabilities của các dịch vụ phù hợp.
- Áp dụng `no-new-privileges` cho các dịch vụ phù hợp.
- Mount các file cấu hình ở chế độ chỉ đọc.
- Nginx có security headers, ẩn phiên bản và chặn tệp nhạy cảm.
- Tài khoản ứng dụng và tài khoản exporter được tách riêng.
- Mật khẩu triển khai được đặt trong các file cấu hình cục bộ
  được `.gitignore` loại trừ.

### 11.1. Kiểm tra security headers

```powershell
curl.exe -I http://localhost:8280
```

Các header mong đợi:

```text
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Kiểm tra chặn tệp cấu hình:

```powershell
curl.exe -I http://localhost:8280/wp-config.php
```

Kết quả mong đợi: HTTP `403`.

### 11.2. Kiểm tra non-root

```powershell
docker compose exec grafana id
docker compose exec prometheus id
```

UID cần khác `0`.

### 11.3. Kiểm tra mạng nội bộ database

```powershell
docker network inspect docker-monitoring_db-network --format '{{.Internal}}'
```

Kết quả mong đợi: `true`.

Cấu hình này giới hạn kết nối ra ngoài của mạng database.
Các container cùng mạng vẫn có thể liên lạc với nhau;
container gắn nhiều mạng vẫn có kết nối qua các mạng còn lại.

### 11.4. Kiểm tra hạn chế quyền Grafana

```powershell
docker inspect $(docker compose ps -q grafana) --format '{{json .HostConfig.SecurityOpt}}'
docker inspect $(docker compose ps -q grafana) --format '{{json .HostConfig.CapDrop}}'
```

Kết quả cần chứa `no-new-privileges:true` và `ALL`.

### 11.5. Kiểm tra quyền tài khoản MySQL

Đăng nhập bằng tài khoản quản trị MySQL:

```powershell
docker compose exec mysql mysql -u root -p
```

Chạy trong MySQL:

```sql
SHOW GRANTS FOR 'wpuser'@'%';
SHOW GRANTS FOR 'exporter'@'%';
```

Phân quyền hiện tại:

- `wpuser`: toàn quyền trong `wpdb`.
- `exporter`: `SELECT`, `PROCESS`, `REPLICATION CLIENT` trên `*.*`.
- Exporter không có quyền ghi dữ liệu, nhưng quyền `SELECT`
  hiện cho phép đọc mọi database.

Thoát MySQL:

```sql
exit;
```

### 11.6. Kiểm tra loại trừ file chứa mật khẩu

```powershell
git check-ignore .env init.sql mysql-exporter.cnf
```

Kết quả cần hiện đủ ba file.

`.gitignore` không xóa thông tin đã được commit trước đó khỏi lịch sử Git.
Nếu mật khẩu từng được công khai, cần đổi mật khẩu thực tế của
tài khoản tương ứng; chỉ chuyển sang `.env` chưa đủ.

### 11.7. Giới hạn hiện tại

- Website sử dụng HTTP, chưa cấu hình HTTPS.
- Không phải tất cả container đều chạy non-root.
- cAdvisor còn sử dụng `privileged`.
- Promtail truy cập Docker socket nên có quyền hạn lớn;
  mount socket `:ro` không biến Docker API thành API chỉ đọc.
- Promtail đã hết hỗ trợ, được giữ lại theo yêu cầu bài thực hành.
- Quyền đọc database của exporter còn rộng.
- Độ mạnh mật khẩu phụ thuộc giá trị người triển khai lựa chọn.
- Một số Docker images sử dụng tag `latest` hoặc tag thay đổi theo thời gian;
  cần cố định phiên bản hoặc digest nếu muốn tái lập môi trường chặt chẽ.

## 12. Dữ liệu và sao lưu

MySQL, Grafana, Prometheus, Loki và Promtail sử dụng volume
để lưu dữ liệu theo cấu hình Compose.

Cần sao lưu database WordPress và các tệp WordPress quan trọng,
đặc biệt là nội dung upload, theme và plugin đã tùy chỉnh.

Có thể xuất database `wpdb` bằng chức năng Export của phpMyAdmin.
Lưu bản sao trong thư mục `backup`, được Git bỏ qua.

Không đưa bản sao lưu database chứa dữ liệu hoặc thông tin
đăng nhập lên repository công khai.

File `website/homepage.html` chỉ lưu mã giao diện trang chủ,
không thay thế bản sao lưu đầy đủ website.

## 13. Minh chứng và demo

Các nội dung cần trình bày:

1. Website hoạt động qua http://localhost:8280.
2. phpMyAdmin truy cập được database `wpdb`.
3. Prometheus có đủ năm target `UP`.
4. `mysql_up = 1` và `nginx_up = 1`.
5. Grafana hiển thị metrics container, Nginx và MySQL.
6. Grafana Explore truy vấn được ba query LogQL.
7. Security headers và chặn truy cập `wp-config.php`.
8. UID non-root, mạng database nội bộ và hạn chế quyền container.
9. Repository có mã trang chủ, cấu hình, dashboard và lịch sử commit.

Chụp screenshot từ hệ thống thực tế để đưa vào báo cáo.

## 14. Dừng và chạy lại

Dừng dịch vụ:

```powershell
docker compose stop
```

Chạy lại các container đã tồn tại:

```powershell
docker compose start
```

Áp dụng thay đổi cấu hình Compose:

```powershell
docker compose up -d
```

Kiểm tra trạng thái:

```powershell
docker compose ps -a
```

Không sử dụng `docker compose down -v` khi cần giữ dữ liệu,
vì tùy chọn `-v` xóa các named volume khai báo trong Compose
và anonymous volume gắn với các container bị xóa.