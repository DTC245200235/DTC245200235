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
