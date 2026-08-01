---
title: "Nhật ký công việc Tuần 5"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Nhật ký công việc Tuần 5

## Mục tiêu của tuần

Mục tiêu của tuần này là nâng cao độ tin cậy của firmware, tối ưu quá trình giao tiếp với AWS IoT Core và cải thiện tính ổn định tổng thể của hệ thống nhúng.

## Công việc đã thực hiện

| Ngày | Công việc |
|---|---|
| Thứ Hai | Cải thiện quá trình tạo dữ liệu telemetry dạng JSON và tối ưu định dạng dữ liệu cảm biến. |
| Thứ Ba | Triển khai cơ chế tự động kết nối lại Wi-Fi trong điều kiện mạng không ổn định. |
| Thứ Tư | Cải thiện cơ chế tự động kết nối lại MQTT và tự động subscribe lại sau khi mất kết nối. |
| Thứ Năm | Bổ sung cơ chế kiểm tra tính hợp lệ của dữ liệu cảm biến và xử lý lỗi đối với các giá trị DHT11 không hợp lệ. |
| Thứ Sáu | Tối ưu hiệu năng firmware và kiểm tra khả năng gửi telemetry ổn định trong quá trình hệ thống hoạt động liên tục. |

---

## Kết quả đạt được

Firmware của hệ thống nhúng đã trở nên ổn định và có khả năng chịu lỗi tốt hơn. ESP32-S3 có thể tự động khôi phục sau các sự cố mất kết nối tạm thời trong khi vẫn duy trì việc truyền dữ liệu telemetry một cách ổn định.

---

## Kiến thức và kỹ năng đạt được

- Tối ưu firmware
- Tự động kết nối lại Wi-Fi
- Tự động kết nối lại MQTT
- Xử lý ngoại lệ (Exception Handling)
- Kiểm tra tính hợp lệ của dữ liệu cảm biến
- Nâng cao độ tin cậy của hệ thống nhúng
```