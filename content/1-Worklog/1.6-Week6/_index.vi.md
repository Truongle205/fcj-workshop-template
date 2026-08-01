---
title: "Nhật ký công việc Tuần 6"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Nhật ký công việc Tuần 6

## Mục tiêu của tuần

Mục tiêu của tuần này là kiểm thử firmware nhúng trong điều kiện hoạt động thực tế và xác minh khả năng giao tiếp ổn định giữa ESP32-S3 và AWS IoT Core.

## Công việc đã thực hiện

| Ngày | Công việc |
|---|---|
| Thứ Hai | Kiểm tra dữ liệu từ cảm biến nhiệt độ, độ ẩm, ánh sáng và cảm biến cửa trong nhiều điều kiện hoạt động khác nhau. |
| Thứ Ba | Xác minh việc gửi dữ liệu telemetry theo chu kỳ từ ESP32-S3 lên AWS IoT Core. |
| Thứ Tư | Kiểm tra chức năng điều khiển relay từ xa thông qua MQTT command topic. |
| Thứ Năm | Thực hiện kiểm thử cơ chế tự động kết nối lại Wi-Fi và MQTT bằng cách mô phỏng các sự cố mất kết nối mạng tạm thời. |
| Thứ Sáu | Kiểm tra độ ổn định của firmware trong quá trình hoạt động liên tục và theo dõi mức sử dụng bộ nhớ thông qua Serial Monitor. |

---

## Kết quả đạt được

Firmware nhúng hoạt động ổn định trong suốt quá trình kiểm thử. Dữ liệu từ các cảm biến được gửi chính xác lên AWS IoT Core, chức năng điều khiển relay từ xa hoạt động đúng như mong đợi và firmware có thể tự động khôi phục sau các sự cố mất kết nối mạng tạm thời.

---

## Kiến thức và kỹ năng đạt được

- Kiểm thử firmware
- Hiệu chuẩn cảm biến
- Kiểm thử giao tiếp MQTT
- Kiểm tra chức năng điều khiển Relay
- Khôi phục kết nối Wi-Fi
- Gỡ lỗi hệ thống nhúng
```