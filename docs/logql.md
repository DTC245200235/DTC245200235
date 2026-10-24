
# Truy vấn LogQL — Đề 19

Thực hiện trong Grafana → Explore → datasource Loki → Code.
Chọn khoảng thời gian có log cần tìm.

## 1. Tìm request theo nội dung

```logql
{service="nginx"} |= "de19-log-test-404"
```

Chọn log Nginx và lọc dòng chứa chuỗi chỉ định.

## 2. Phân tích JSON và lọc HTTP 404

```logql
{service="nginx"}
| json
| status = 404
| __error__ = ""
```

Phân tích các trường JSON, lọc mã HTTP 404 và loại dòng gặp lỗi trong quá trình phân tích hoặc lọc.

## 3. Đếm response 404 trong cửa sổ 5 phút

```logql
sum(
  count_over_time(
    {service="nginx"}
    | json
    | status = 404
    | __error__ = ""
    [5m]
  )
)
```

Kết quả là số dòng log phù hợp trong 5 phút trước mỗi thời điểm đánh giá.

Tạo dữ liệu thử bằng lệnh:

```bash
curl -sS -o /dev/null http://localhost:8080/de19-log-test-404
curl -sS -o /dev/null http://localhost:8080/de19-log-test-404-b
```

## 4. Xem log WordPress/Apache

```logql
{service="wordpress"}
```

Truy cập website để tạo access log mới.

## 5. Xem log MySQL

```logql
{service="mysql"}
```

Đây là log stdout/stderr của container MySQL, không phải bản ghi tất cả câu SQL.

Nếu cần kiểm tra bằng log khởi động mới, lưu công việc đang làm rồi restart riêng MySQL. Database sẽ gián đoạn ngắn trong quá trình khởi động lại.

## Minh chứng

Ảnh truy vấn và kết quả được lưu trong docs/chup-man-hinh/.
