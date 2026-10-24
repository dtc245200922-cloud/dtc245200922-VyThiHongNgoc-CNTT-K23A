# WEBSITE QUẢNG BÁ SẢN PHẨM(WORDPRESS)

## 1. Thông tin sinh viên

- Họ tên: Vy Thị Hồng Ngọc.
- Mã sinh viên: DTC245200922.
- Lớp: CNTT K23A.
- Học phần: Triển khai và Quản trị Hệ thống Phần mềm.
- Đề tài: WEBSITE QUẢNG BÁ SẢN PHẨM(WORDPRESS).
- Thương hiệu website: Luna Bags.

## 2. Giới thiệu

Website quảng bá sản phẩm túi xách Luna Bags được xây dựng bằng
WordPress và MySQL, cung cấp hình ảnh, giá tham khảo và thông tin
giới thiệu sản phẩm.

Hệ thống được triển khai bằng Docker Compose, tích hợp Nginx reverse
proxy, phpMyAdmin, Prometheus, Grafana, Loki và Promtail.

Website phục vụ mục đích giới thiệu sản phẩm, chưa tích hợp giỏ hàng,
đặt hàng và thanh toán. Tên và giá sản phẩm là dữ liệu minh họa.

## 3. Các thành phần

| Thành phần | Chức năng |
| --- | --- |
| WordPress | Quản lý nội dung và hiển thị website |
| MySQL | Lưu dữ liệu WordPress |
| phpMyAdmin | Quản trị cơ sở dữ liệu |
| Nginx | Reverse proxy và security headers |
| Nginx Exporter | Chuyển thống kê Nginx thành metrics |
| Prometheus | Thu thập metrics |
| Grafana | Dashboard giám sát và truy vấn log |
| Node Exporter | Metrics môi trường Linux chạy Docker |
| cAdvisor | Metrics tài nguyên container |
| MySQL Exporter | Metrics MySQL |
| Loki | Lưu trữ và truy vấn log |
| Promtail | Thu thập log container gửi đến Loki |

Luồng truy cập:

Trình duyệt → Nginx → WordPress → MySQL.

Luồng dữ liệu giám sát:

Exporter / cAdvisor → Prometheus → Grafana.

Prometheus chủ động scrape endpoint metrics; Grafana truy vấn
Prometheus để hiển thị dashboard.

Luồng log:

Log container → Promtail → Loki → Grafana Explore.

Trên Docker Desktop, metrics của Node Exporter phản ánh môi trường
Linux chạy Docker, không phải toàn bộ hệ điều hành Windows.

## 4. Cấu trúc repository

```text
docker-monitoring/
├── .env.example
├── .gitignore
├── README.md
├── docker-compose.yml
├── docker-compose.override.yml
├── init.sql.example
├── mysql-exporter.cnf.example
├── prometheus.yml
├── loki-config.yml
├── promtail-config.yml
├── nginx/
│   └── default.conf
├── website/
│   └── homepage.html
└── grafana/
    └── dashboards/
        └── wordpress-monitoring.json
```

Các file chứa mật khẩu thực tế gồm `.env`, `init.sql` và
`mysql-exporter.cnf` được tạo tại máy triển khai và được `.gitignore`
loại trừ.

## 5. Yêu cầu môi trường

- Docker Desktop chạy Linux containers.
- Docker Compose v2.
- Git.
- Các cổng 8280, 8182, 9190, 3100, 8181 và 3200 còn trống.
- Kết nối Internet để tải Docker images, ảnh và phông chữ.

Cấu hình mount của cAdvisor và Node Exporter phục vụ môi trường
Docker Desktop Linux. Cần kiểm tra lại đường dẫn khi đổi môi trường.

## 6. Chuẩn bị cấu hình

Clone repository:

```powershell
git clone https://github.com/dtc245200922-cloud/dtc245200922-VyThiHongNgoc-CNTT-K23A.git docker-monitoring
cd docker-monitoring
```

Giữ tên project là `docker-monitoring` vì cấu hình Promtail và bộ lọc
metrics container đang sử dụng tên này.

Nếu chưa có các file cấu hình thực tế, tạo từ bản mẫu:

```powershell
Copy-Item .env.example .env
Copy-Item init.sql.example init.sql
Copy-Item mysql-exporter.cnf.example mysql-exporter.cnf
```

Không ghi đè các file cấu hình đang sử dụng.

Điền mật khẩu riêng trong `.env`:

- `MYSQL_ROOT_PASSWORD`: mật khẩu quản trị MySQL.
- `WORDPRESS_DB_PASSWORD`: mật khẩu tài khoản ứng dụng `wpuser`.

Thay `CHANGE_ME_EXPORTER_PASSWORD` trong cả `init.sql` và
`mysql-exporter.cnf` bằng cùng một mật khẩu riêng cho exporter.

Sử dụng mật khẩu mạnh và khác nhau giữa các tài khoản.
Không đưa mật khẩu thực tế lên GitHub.

Lưu ý:

- `init.sql` chỉ tự chạy khi MySQL khởi tạo thư mục dữ liệu mới.
- Thay đổi `.env` không tự đổi mật khẩu trong database đã có dữ liệu.
- Khi đổi mật khẩu tài khoản hiện có, phải cập nhật trong MySQL
  và cấu hình dịch vụ sử dụng tài khoản đó.
- Tài khoản quản trị WordPress được tạo riêng khi cài đặt website.

## 7. Khởi chạy hệ thống

Trong thư mục dự án:

```powershell
docker compose config --quiet
docker compose up -d
docker compose ps -a
```

Docker Compose tự kết hợp `docker-compose.yml`
và `docker-compose.override.yml`.

Nếu có dịch vụ chưa chạy ổn định:

```powershell
docker compose logs --tail=50 TEN_DICH_VU
```

Thay `TEN_DICH_VU` bằng tên dịch vụ cần kiểm tra.

Ví dụ:

```powershell
docker compose logs --tail=50 wordpress mysql nginx
```

Ở lần chạy đầu, MySQL cần thời gian khởi tạo.
Nếu WordPress báo lỗi kết nối database, kiểm tra log và đợi MySQL sẵn sàng.

## 8. Địa chỉ truy cập

| Dịch vụ | Địa chỉ |
| --- | --- |
| Website Luna Bags qua Nginx | http://localhost:8280 |
| Quản trị WordPress | http://localhost:8280/wp-admin/ |
| phpMyAdmin | http://localhost:8182 |
| Prometheus | http://localhost:9190 |
| Prometheus Targets | http://localhost:9190/targets |
| Grafana | http://localhost:3100 |
| cAdvisor | http://localhost:8181/containers/ |
| Loki readiness | http://localhost:3200/ready |

Cổng thống kê Nginx `8080` và cổng Nginx Exporter `9113`
chỉ sử dụng trong mạng Docker, không publish ra host.

### 8.1. Cài đặt trang chủ Luna Bags

Với database mới:

1. Mở http://localhost:8280.
2. Hoàn tất cài đặt WordPress và tạo tài khoản quản trị.
3. Đăng nhập tại http://localhost:8280/wp-admin/.
4. Chọn Trang → Thêm trang.
5. Đặt tên trang là `Trang chủ`.
6. Thêm khối HTML tùy chỉnh.
7. Mở file `website/homepage.html`.
8. Sao chép toàn bộ nội dung file vào khối HTML tùy chỉnh.
9. Xuất bản trang.
10. Vào Cài đặt → Đọc.
11. Chọn hiển thị một trang tĩnh và đặt trang vừa tạo làm trang chủ.
12. Mở lại http://localhost:8280 để kiểm tra.

Giao diện Luna Bags gồm:

- Tên thương hiệu và menu điều hướng.
- Banner ngang với nội dung bên trái, hai ảnh sản phẩm bên phải.
- Bốn sản phẩm trong hai bảng đặt cạnh nhau, mỗi bảng gồm hai mẫu.
- Mục trải nghiệm chọn túi và danh mục sản phẩm.
- Banner bộ sưu tập, phần giới thiệu, liên hệ và chân trang.
- Giá sản phẩm minh họa từ 159.000đ đến 279.000đ.

Trên laptop, giao diện sử dụng bố cục ngang.
Trên điện thoại, các khối chính tự chuyển thành một cột.

Menu sử dụng liên kết đến các phần trong cùng trang.
Mục “Xem chi tiết” mở thông tin giới thiệu từng mẫu túi.

CSS có quy tắc ẩn tiêu đề trang và phần đầu, chân trang mặc định
của block theme để tránh trùng với giao diện tự thiết kế.

Nếu trình chỉnh sửa có các tab HTML, CSS và JavaScript,
dán toàn bộ file vào tab HTML vì code đã chứa thẻ `<style>`.

Khi cập nhật trang chủ đang có:

1. Mở Trang → Trang chủ → Chỉnh sửa.
2. Chọn khối HTML tùy chỉnh đang chứa giao diện.
3. Thay toàn bộ code cũ bằng nội dung `website/homepage.html`.
4. Nhấn Cập nhật.
5. Mở website và nhấn Ctrl + F5.

Ảnh sử dụng liên kết bên ngoài, phông chữ tải từ Google Fonts.
Việc hiển thị phụ thuộc kết nối Internet và nguồn cung cấp.

Clone repository hoặc chạy `git pull` không tự cập nhật nội dung
đã lưu trong database WordPress. Cần dán code vào trang hoặc
khôi phục dữ liệu từ bản sao lưu.

### 8.2. Đăng nhập phpMyAdmin

- Địa chỉ: http://localhost:8182.
- Máy chủ: `mysql`.
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

Các URL này được Grafana truy cập qua mạng Docker.
Không thay bằng `localhost` trong cấu hình data source.

### 8.4. Import dashboard

1. Mở Grafana → Dashboards → New → Import.
2. Tải lên `grafana/dashboards/wordpress-monitoring.json`.
3. Chọn data source Prometheus nếu được yêu cầu.
4. Hoàn tất import và kiểm tra dữ liệu.

Nếu không tìm thấy data source, chỉnh các panel để sử dụng
data source Prometheus hiện có rồi lưu dashboard.

Dashboard gồm các panel:

- Trạng thái MySQL.
- CPU từng container.
- RAM từng container.
- Số kết nối MySQL.
- Trạng thái Nginx.

Dashboard được import thủ công, chưa cấu hình provisioning tự động.
Cần kiểm tra khả năng tương thích khi sử dụng phiên bản Grafana khác.

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
Để kiểm tra kết nối exporter tới dịch vụ đích, xem thêm
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

Nếu đổi tên project Compose, cập nhật bộ lọc tên container
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

Đợi vài giây rồi chạy lại truy vấn.

Nếu không thấy log:

```powershell
docker compose logs --tail=50 promtail loki nginx
```

Kiểm tra khoảng thời gian truy vấn và tên project
trong cấu hình Docker service discovery của Promtail.

## 11. Hardening hệ thống

Các biện pháp cấu hình:

- MySQL sử dụng mạng database riêng có `internal: true`.
- Không publish cổng MySQL và cổng trực tiếp WordPress.
- Các cổng website, quản trị và giám sát bind về `127.0.0.1`.
- Grafana và Prometheus chạy bằng người dùng non-root.
- Nginx Exporter chạy non-root và filesystem chỉ đọc.
- Bỏ Linux capabilities của các dịch vụ phù hợp.
- Áp dụng `no-new-privileges` cho các dịch vụ phù hợp.
- Mount file cấu hình ở chế độ chỉ đọc.
- Nginx có security headers, ẩn phiên bản và chặn tệp nhạy cảm.
- Tách tài khoản ứng dụng và tài khoản exporter.
- Loại trừ file chứa mật khẩu thực tế khỏi Git.

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

Grafana có thể thuộc nhóm GID 0 nhưng UID khác 0;
điều này không đồng nghĩa tiến trình chạy với UID root.

### 11.3. Kiểm tra mạng database

```powershell
docker network inspect docker-monitoring_db-network --format '{{.Internal}}'
```

Kết quả mong đợi: `true`.

Các container cùng mạng vẫn có thể liên lạc với nhau.
Container gắn nhiều mạng có thể kết nối qua các mạng còn lại.

### 11.4. Kiểm tra hạn chế quyền Grafana

```powershell
docker inspect $(docker compose ps -q grafana) --format '{{json .HostConfig.SecurityOpt}}'
docker inspect $(docker compose ps -q grafana) --format '{{json .HostConfig.CapDrop}}'
```

Kết quả cần chứa:

```text
["no-new-privileges:true"]
["ALL"]
```

### 11.5. Kiểm tra quyền tài khoản MySQL

Đăng nhập MySQL:

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
- Exporter không có quyền ghi dữ liệu, nhưng quyền SELECT còn rộng,
  cho phép đọc mọi database.

Thoát MySQL:

```sql
exit;
```

### 11.6. Kiểm tra file chứa mật khẩu

```powershell
git check-ignore .env init.sql mysql-exporter.cnf
```

Kết quả cần hiện đủ ba file.

`.gitignore` không xóa thông tin đã tồn tại trong lịch sử Git.
Nếu mật khẩu từng được công khai, cần đổi mật khẩu thực tế;
chỉ chuyển sang `.env` chưa đủ.

### 11.7. Giới hạn hiện tại

- Website sử dụng HTTP, chưa cấu hình HTTPS.
- Không phải tất cả container đều chạy non-root.
- cAdvisor còn sử dụng `privileged`.
- Promtail chạy root và truy cập Docker socket.
- Mount socket `:ro` không biến Docker API thành API chỉ đọc.
- Promtail đã hết hỗ trợ, được giữ theo yêu cầu bài thực hành.
- Quyền đọc database của exporter còn rộng.
- Độ mạnh mật khẩu phụ thuộc giá trị người triển khai lựa chọn.
- Một số Docker images sử dụng tag thay đổi theo thời gian;
  cần cố định phiên bản hoặc digest để tăng khả năng tái lập.

## 12. Dữ liệu và sao lưu

MySQL, Grafana, Prometheus, Loki và Promtail sử dụng volume
để lưu dữ liệu theo cấu hình Compose.

Cần sao lưu database WordPress và các tệp quan trọng,
đặc biệt là upload, theme và plugin đã tùy chỉnh.

Có thể xuất database `wpdb` bằng chức năng Export của phpMyAdmin.
Lưu bản sao trong thư mục `backup`, được Git bỏ qua.

Không đưa bản sao lưu chứa dữ liệu hoặc thông tin đăng nhập
lên repository công khai.

File `website/homepage.html` chỉ lưu mã giao diện trang chủ,
không thay thế bản sao lưu đầy đủ website.

## 13. Kiểm thử và demo

Các nội dung cần kiểm tra và chụp minh chứng:

1. Các dịch vụ hiển thị trong `docker compose ps -a`.
2. Website Luna Bags hoạt động qua http://localhost:8280.
3. Menu chuyển đến đúng phần và mở được thông tin sản phẩm.
4. phpMyAdmin truy cập được database `wpdb`.
5. Prometheus có đủ năm target `UP`.
6. `mysql_up = 1` và `nginx_up = 1`.
7. Grafana hiển thị metrics container, Nginx và MySQL.
8. Grafana Explore chạy được ba query LogQL.
9. Nginx trả security headers và chặn `wp-config.php`.
10. UID non-root, mạng database nội bộ và giới hạn quyền container.
11. GitHub có mã trang chủ, cấu hình, dashboard, README và lịch sử commit.

Trạng thái container `Up` chưa đủ để xác nhận mọi chức năng hoạt động.
Cần kiểm tra thêm website, database, metrics và log.

Ảnh minh chứng phải được chụp từ hệ thống thực tế.
Các kết quả chưa kiểm tra cần được ghi rõ, không mặc định là đạt.

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

Không dùng `docker compose down -v` khi cần giữ dữ liệu,
vì tùy chọn `-v` xóa các named volume khai báo trong Compose
và anonymous volume gắn với các container bị xóa.