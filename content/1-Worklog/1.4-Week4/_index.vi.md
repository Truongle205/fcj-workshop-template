---
title: "Nhật ký công việc Tuần 4"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Nhật ký công việc Tuần 4

## Mục tiêu của tuần

Mục tiêu của tuần này là tích hợp firmware trên ESP32-S3 với AWS IoT Core thông qua MQTT over TLS và triển khai cơ chế giao tiếp hai chiều.

## Công việc đã thực hiện

| Ngày | Công việc |
|---|---|
| Thứ Hai | Triển khai kết nối MQTT over TLS sử dụng AWS IoT Endpoint và chứng chỉ X.509. |
| Thứ Ba | Gửi dữ liệu telemetry của hệ thống Smart Home theo chu kỳ đến MQTT topic trên AWS IoT Core. |
| Thứ Tư | Triển khai các hàm callback của MQTT để nhận các thông điệp từ Cloud. |
| Thứ Năm | Phát triển chức năng điều khiển relay từ xa thông qua MQTT command topic. |
| Thứ Sáu | Kiểm tra cơ chế giao tiếp hai chiều giữa ESP32-S3 và AWS IoT Core thông qua các thao tác publish và subscribe. |

---

## Kết quả đạt được

ESP32-S3 đã thiết lập thành công kết nối MQTT bảo mật với AWS IoT Core. Dữ liệu telemetry được gửi lên theo chu kỳ và các lệnh điều khiển relay từ AWS IoT Core được nhận và thực thi chính xác.

---

## Kiến thức và kỹ năng đạt được

- MQTT over TLS
- MQTT Publish/Subscribe
- MQTT Client trên ESP32
- Điều khiển Relay từ xa
- Xác thực TLS
- Tích hợp thiết bị IoT
```