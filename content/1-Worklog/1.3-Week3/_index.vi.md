---
title: "Nhật ký công việc Tuần 3"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Nhật ký công việc Tuần 3

## Mục tiêu của tuần

Mục tiêu của tuần này là phát triển firmware nhúng để thu thập dữ liệu từ các cảm biến và tạo các thông điệp telemetry.

## Công việc đã thực hiện

| Ngày | Công việc |
|---|---|
| Thứ Hai | Triển khai trình điều khiển (driver) cho cảm biến DHT11 để đo nhiệt độ và độ ẩm. |
| Thứ Ba | Triển khai giao tiếp với cảm biến LDR để phát hiện cường độ ánh sáng môi trường. |
| Thứ Tư | Phát triển giao tiếp với cảm biến cửa từ để giám sát trạng thái cửa. |
| Thứ Năm | Triển khai các chức năng điều khiển relay và kiểm tra hoạt động của relay. |
| Thứ Sáu | Tạo dữ liệu telemetry dạng JSON chứa các giá trị cảm biến và trạng thái thiết bị để phục vụ giao tiếp với Cloud. |

---

## Kết quả đạt được

Firmware trên ESP32-S3 đã thu thập thành công dữ liệu môi trường từ tất cả các cảm biến và tạo được telemetry dạng JSON, sẵn sàng để truyền lên AWS IoT Core.

---

## Kiến thức và kỹ năng đạt được

- Lập trình GPIO trên ESP32
- Giao tiếp với cảm biến DHT11
- Giao tiếp với cảm biến LDR
- Giao tiếp với cảm biến cửa từ
- Điều khiển Relay
- Định dạng dữ liệu JSON
```