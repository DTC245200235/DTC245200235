# Hardening hệ thống — Đề 19

## 1. Container không chạy bằng root

Áp dụng cho nginx-exporter và mysql-exporter:

- user: "65534:65534"
- Đã kiểm tra tiến trình thực tế bằng docker top.
- UID của hai tiến trình exporter là 65534.

## 2. Hạn chế quyền container

Áp dụng cho hai exporter:

- read_only: true
- cap_drop: ALL
- security_opt: no-new-privileges:true

Đã kiểm tra cấu hình thực tế bằng docker inspect.
Sau thay đổi, nginx_up và mysql_up vẫn bằng 1.

## 3. Phân tách mạng theo chức năng

- Nginx: web.
- WordPress: web và database.
- MySQL: database.
- Prometheus: monitoring.
- MySQL Exporter: database và monitoring.

Nginx không tham gia mạng database.
Prometheus lấy metrics từ exporter, không trực tiếp đăng nhập MySQL.

Đây là phân tách mạng Docker, không phải cô lập tuyệt đối.
Các container chung mạng vẫn có thể kết nối theo cổng dịch vụ.

## 4. Giới hạn cổng publish

Chỉ publish các cổng cần sử dụng, bind tại 127.0.0.1:

- 8080: Nginx.
- 8081: phpMyAdmin.
- 9090: Prometheus.
- 3000: Grafana.

WordPress, MySQL, exporters, cAdvisor, Loki và Promtail
không publish cổng ra host.

## 5. Giới hạn phạm vi quyền database

WordPress dùng tài khoản riêng, không dùng root.

Tài khoản ứng dụng chỉ được cấp quyền trên news_db.
Phép thử SELECT vào mysql.user bị từ chối quyền.

Tài khoản MySQL Exporter được tạo riêng.
Exporter có quyền đọc rộng theo cấu hình mẫu, không có quyền ghi dữ liệu. Chưa coi đây là quyền tối thiểu tuyệt đối.

## 6. Security headers Nginx

- X-Content-Type-Options: nosniff.
- X-Frame-Options: SAMEORIGIN.
- Referrer-Policy: strict-origin-when-cross-origin.

Đã kiểm tra bằng curl -I.
Hệ thống hiện dùng HTTP và security headers, chưa có HTTPS.

## 7. Quản lý file cấu hình và bí mật

- .env không được đưa lên Git.
- .env.example chỉ chứa giá trị mẫu.
- File .env được đặt quyền 600.
- File cấu hình Nginx và PHP được mount chỉ đọc.
- Gói theme thương mại được loại khỏi repository.

.env không được mã hóa. Người có quyền quản trị Docker
vẫn có thể xem biến môi trường của container.

## 8. Ngoại lệ và giới hạn

- cAdvisor chạy privileged để giám sát hệ thống.
- Promtail chạy root và truy cập Docker socket.
- Mount socket với :ro không giới hạn Docker API thành chỉ đọc.
- Promtail đã hết hỗ trợ; hướng nâng cấp là Grafana Alloy.
- Chưa áp dụng non-root hoặc read-only cho mọi container.

## Minh chứng

- docs/ket-qua/08-exporter-hardening.txt
- docs/ket-qua/09-network-isolation.txt
- docs/ket-qua/10-published-ports.txt
- docs/ket-qua/11-security-headers.txt
- docs/chup-man-hinh/15-database-permissions.png
