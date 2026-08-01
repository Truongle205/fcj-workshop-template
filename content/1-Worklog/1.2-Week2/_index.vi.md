---
title: "Nhật ký công việc Tuần 2"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Nhật ký công việc Tuần 2

## Mục tiêu của tuần

Mục tiêu của tuần này là chuẩn bị môi trường phát triển nhúng và thiết lập kết nối ban đầu giữa bo mạch ESP32-S3 và AWS IoT Core.

## Công việc đã thực hiện

| Ngày | Công việc |
|---|---|
| Thứ Hai | Cài đặt PlatformIO, cấu hình môi trường phát triển cho ESP32-S3 và kiểm tra kết nối với bo mạch. |
| Thứ Ba | Tạo dự án PlatformIO và cấu hình các thư viện cần thiết cho dự án. |
| Thứ Tư | Triển khai chức năng kết nối Wi-Fi và kiểm tra khả năng truy cập Internet của ESP32-S3. |
| Thứ Năm | Cấu hình đồng bộ thời gian bằng Network Time Protocol (NTP) để phục vụ việc xác thực chứng chỉ TLS. |
| Thứ Sáu | Thêm Root CA Certificate, Device Certificate và Private Key vào dự án firmware để chuẩn bị cho việc giao tiếp MQTT over TLS với AWS IoT Core. |

---

## Kết quả đạt được

Môi trường phát triển nhúng đã được chuẩn bị hoàn chỉnh. ESP32-S3 có thể kết nối thành công với mạng Wi-Fi, đồng bộ thời gian hệ thống bằng NTP và sẵn sàng thiết lập kết nối MQTT bảo mật với AWS IoT Core.

---

## Kiến thức và kỹ năng đạt được

- PlatformIO
- Phát triển ứng dụng trên ESP32-S3
- Lập trình kết nối Wi-Fi
- Network Time Protocol (NTP)
- Quản lý chứng chỉ TLS
- Thiết lập môi trường phát triển hệ thống nhúng