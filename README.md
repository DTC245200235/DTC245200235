# Đề 19 — Website tin tức / Cổng thông tin

## Thông tin sinh viên

- Họ tên: Nguyễn Văn Sáng
- MSSV: DTC245200235
- Môn học: Triển khai và Quản trị Hệ thống Phần mềm

## Thành phần hiện tại

- WordPress + theme Foxiz : website tin tức. (có thể dùng theme khác có sẵn template tin tức hoặc theme mặc định)
- MySQL: lưu dữ liệu ứng dụng.
- phpMyAdmin: quản trị database.
- Nginx: reverse proxy, bổ sung security headers và access log JSON.
- Docker Compose: khai báo và vận hành hệ thống.

## Luồng truy cập

Trình duyệt → Nginx:80 → WordPress:80 → MySQL:3306.

Cổng 8080 của Ubuntu được chuyển vào Nginx.
WordPress và MySQL không publish cổng ra host.
phpMyAdmin sử dụng cổng 8081, giới hạn tại localhost.

## Yêu cầu môi trường

- Docker Engine.
- Docker Compose hỗ trợ lệnh docker compose.
- Cổng 8080 và 8081 còn trống.

## Khởi chạy trên môi trường mới

1. Sao chép repository.
2. Chạy lệnh: cp .env.example .env
3. Sửa các giá trị bên trong .env, đặt hai mật khẩu mạnh khác nhau.
4. Chạy lệnh: chmod 600 .env
5. Chạy lệnh: docker compose -f docker-compose.yaml config --quiet
6. Chạy lệnh: docker compose -f docker-compose.yaml up -d
7. Mở http://localhost:8080 và hoàn thành cài đặt WordPress.
8. Cài Foxiz từ gói đã mua hợp lệ và các plugin cần thiết.

Các URL localhost được mở trong Ubuntu chạy Docker.

## Địa chỉ sử dụng

- Website: http://localhost:8080
- Quản trị WordPress: http://localhost:8080/wp-admin/
- phpMyAdmin: http://localhost:8081

Đăng nhập phpMyAdmin bằng MYSQL_USER và MYSQL_PASSWORD trong .env.

## Kiểm tra hệ thống

- Trạng thái: docker compose -f docker-compose.yaml ps
- Cú pháp Nginx: docker compose -f docker-compose.yaml exec nginx nginx -t
- HTTP headers: curl -I http://localhost:8080
- Log Nginx: docker compose -f docker-compose.yaml logs --tail=20 nginx

## Dữ liệu và vận hành

- mysql_data lưu database.
- wordpress_data lưu WordPress, theme, plugin và uploads.
- docker compose -f docker-compose.yaml stop: dừng dịch vụ.
- docker compose -f docker-compose.yaml start: chạy lại dịch vụ đã dừng.
- docker compose -f docker-compose.yaml down: xóa container và network,
  giữ named volume.
- Không dùng down -v nếu cần giữ dữ liệu.

Nội dung website hiện tại nằm trong Docker volume.
Clone repository không tự khôi phục database, ảnh và cấu hình demo.
Quy trình backup/restore sẽ được bổ sung ở giai đoạn tiếp theo.

## Bảo mật đã áp dụng

- Không đưa .env vào Git.
- Tài khoản database riêng cho WordPress.
- MySQL không publish cổng.
- Nginx chỉ tham gia mạng web.
- File cấu hình PHP và Nginx được mount chỉ đọc.
- Nginx bổ sung X-Content-Type-Options, X-Frame-Options
  và Referrer-Policy.
- Giới hạn kích thước file log Docker.

Hệ thống hiện dùng HTTP và security headers, chưa cấu hình HTTPS.

## Minh chứng

Kết quả kiểm tra nằm trong docs/ket-qua.
Ảnh minh họa nằm trong docs/chup-man-hinh.



## Giám sát Prometheus và Grafana

### Thành phần

- Prometheus thu thập và lưu metrics.
- cAdvisor cung cấp CPU và RAM của container.
- Nginx Exporter đọc endpoint stub_status nội bộ.
- MySQL Exporter cung cấp metrics database.
- Grafana hiển thị dashboard.

### Địa chỉ trong trình duyệt Ubuntu

- Prometheus: http://localhost:9090
- Prometheus Targets: http://localhost:9090/targets
- Grafana: http://localhost:3000

Tài khoản Grafana được cấu hình qua GRAFANA_ADMIN_USER và GRAFANA_ADMIN_PASSWORD trong .env khi khởi tạo lần đầu.
Thay đổi biến mật khẩu không tự đổi mật khẩu trong database Grafana đã khởi tạo.

### Cấu hình môi trường

File .env.example cung cấp cấu hình mẫu:

- MYSQL_DATABASE=news_db
- MYSQL_USER=news_user
- MYSQL_PASSWORD=SQL_PASSWORD
- MYSQL_ROOT_PASSWORD=SQL_ROOT_PASSWORD
- MYSQL_EXPORTER_PASSWORD=EXPORTER_PASSWORD
- GRAFANA_ADMIN_USER=admin
- GRAFANA_ADMIN_PASSWORD=GRAFANA_PASSWORD

Khi triển khai mới:

1. Sao chép mẫu bằng: cp .env.example .env
2. Giữ nguyên tên các biến.
3. Thay các giá trị mật khẩu mẫu bằng mật khẩu mạnh, riêng biệt.
4. Có thể đổi database, username MySQL và username Grafana.
5. Giới hạn quyền file bằng: chmod 600 .env

Các giá trị SQL_PASSWORD, SQL_ROOT_PASSWORD, EXPORTER_PASSWORD
và GRAFANA_PASSWORD chỉ là chỗ giữ chỗ, không dùng nguyên mẫu mà phải thay bằng password mạnh.

Thông tin đăng nhập thực tế lấy từ .env, không lấy từ .env.example.
Không đưa .env lên Git.

Tài khoản exporter trong MySQL phải có mật khẩu khớp với
MYSQL_EXPORTER_PASSWORD trong .env.

Với dữ liệu đã khởi tạo, thay đổi .env không tự đổi mật khẩu
MySQL hoặc tài khoản Grafana đã lưu trong database.

### Tài khoản giám sát MySQL

Khi dựng mới database, cần tạo tài khoản exporter trước khi kiểm tra MySQL Exporter.

Mở MySQL bằng tài khoản root:

docker compose -f docker-compose.yaml exec -e MYSQL_HISTFILE=/dev/null mysql mysql -u root -p

Chạy SQL, thay CHANGE_ME bằng đúng MYSQL_EXPORTER_PASSWORD:

CREATE USER 'exporter'@'%' IDENTIFIED BY 'CHANGE_ME' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';

Tài khoản exporter không có quyền ghi nhưng có quyền đọc rộng.
MySQL không publish cổng ra host.

### Dashboard provisioning

- Datasource: grafana/provisioning/datasources/
- Cấu hình nạp dashboard: grafana/provisioning/dashboards/
- Định nghĩa dashboard: grafana/dashboards/

Grafana tự nạp tám dashboard vào thư mục De19.
Nếu chỉnh dashboard trên giao diện, cần xuất lại JSON và
cập nhật file trong repository.

### Lưu trữ

- prometheus_data lưu metrics.
- grafana_data lưu database Grafana.
- Prometheus được cấu hình retention 3 ngày và 512 MB.
- Dung lượng thực tế có thể cao hơn do WAL và dữ liệu đang ghi.

### Kiểm tra

- Bốn target prometheus, nginx, cadvisor, mysql phải UP.
- Chạy nginx_up và mysql_up trong phần Query phải bằng 1.
- Các dashboard phải có dữ liệu thực tế.

### Lưu ý quyền giám sát

cAdvisor hiện chạy privileged và truy cập Docker socket để thu thập thông tin hệ thống. Đây là ngoại lệ có quyền cao của thành phần giám sát, không phải container đã được harden theo nguyên tắc quyền tối thiểu.

## Log tập trung — Loki và Promtail

### Luồng dữ liệu

Container Nginx / WordPress / MySQL
→ Promtail → Loki → Grafana Explore.

Promtail khám phá container qua Docker API, chỉ thu log
của ba service nginx, wordpress, mysql trong project de19-news.

Các nhãn chính:
- service: tên service Docker Compose.
- container: tên container.
- job: docker.

### Cấu hình và lưu trữ

- loki/loki.yaml: cấu hình lưu log và retention.
- promtail/promtail.yaml: cấu hình thu thập và gắn nhãn.
- grafana/provisioning/datasources/loki.yaml: datasource Loki.
- loki_data: lưu dữ liệu Loki.
- promtail_positions: lưu vị trí thu thập log.
- Loki và Promtail không publish cổng ra host.

Loki có retention 24 giờ, xử lý xóa qua Compactor.
Đây không phải giới hạn cứng dung lượng ổ đĩa.

### Kiểm tra

docker compose -f docker-compose.yaml exec prometheus wget -T 10 -qO- http://loki:3100/ready

Kết quả mong đợi: ready.

Mở Grafana tại http://localhost:3000, chọn Explore,
datasource Loki và chạy các truy vấn trong docs/logql.md.

### Giới hạn triển khai

Promtail đã hết hỗ trợ từ ngày 02/03/2026.
Bài thực hành giữ Promtail theo yêu cầu đề.
Hướng nâng cấp thực tế là chuyển sang Grafana Alloy.

Promtail hiện chạy root và truy cập Docker socket.
Đây là ngoại lệ quyền cao của thành phần thu thập log.
Mount socket với :ro không giới hạn Docker API thành chỉ đọc.

## Hardening hệ thống

Chi tiết cấu hình, kết quả kiểm tra và các ngoại lệ:
[docs/hardening.md](docs/hardening.md).

Các biện pháp gồm non-root cho exporter, root filesystem
chỉ đọc, giảm capabilities, no-new-privileges, phân tách
mạng, giới hạn cổng publish, giới hạn quyền database và
security headers Nginx.
