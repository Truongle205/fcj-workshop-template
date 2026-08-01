---
title: "Đề xuất"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Hệ thống Smart Home IoT trên AWS

## Hệ thống giám sát nhà thông minh an toàn sử dụng công nghệ IoT và các dịch vụ AWS Cloud

---

# 1. Tóm tắt đề xuất

Dự án này đề xuất xây dựng một hệ thống Smart Home IoT sử dụng bo mạch phát triển ESP32-S3 kết hợp với các dịch vụ điện toán đám mây của Amazon Web Services (AWS).

Hệ thống liên tục giám sát các điều kiện môi trường bao gồm nhiệt độ, độ ẩm, cường độ ánh sáng và trạng thái cửa. Dữ liệu từ các cảm biến được truyền an toàn đến AWS IoT Core thông qua giao thức MQTT over TLS 1.2 với cơ chế xác thực bằng chứng chỉ X.509.

AWS IoT Rules Engine xử lý các dữ liệu telemetry nhận được và chuyển chúng đến Amazon DynamoDB để lưu trữ. Khi hệ thống phát hiện cửa được mở, Amazon SNS sẽ tự động gửi email thông báo đến người dùng.

Kiến trúc được đề xuất là một giải pháp IoT gọn nhẹ, an toàn và có khả năng mở rộng, phù hợp cho các ứng dụng nhà thông minh và mục đích học tập.

---

# 2. Phát biểu bài toán

## Những thách thức hiện tại

Các hệ thống giám sát nhà ở truyền thống thường phụ thuộc vào các thiết bị cục bộ và không có cơ chế quản lý tập trung.

Các hệ thống này thường thiếu:

- Khả năng giám sát từ xa theo thời gian thực.
- Cơ chế xác thực thiết bị an toàn.
- Hệ thống lưu trữ telemetry tập trung.
- Cơ chế thông báo sự kiện tự động.
- Khả năng mở rộng dựa trên nền tảng đám mây.

Khi số lượng thiết bị kết nối ngày càng tăng, việc quản lý dữ liệu cảm biến và theo dõi trạng thái hệ thống trở nên khó khăn hơn.

---

## Giải pháp đề xuất

Hệ thống Smart Home IoT được đề xuất sử dụng các dịch vụ được quản lý của AWS nhằm cung cấp khả năng giao tiếp bảo mật, lưu trữ dữ liệu tập trung và giám sát theo thời gian thực.

ESP32-S3 thu thập dữ liệu từ nhiều cảm biến và gửi các thông điệp telemetry đến AWS IoT Core thông qua giao thức MQTT over TLS.

AWS IoT Rules Engine tự động chuyển dữ liệu telemetry đến Amazon DynamoDB để lưu trữ lâu dài.

Khi cảm biến cửa phát hiện sự kiện mở cửa, một AWS IoT Rule khác sẽ gửi thông báo đến Amazon SNS, sau đó Amazon SNS sẽ lập tức gửi email cảnh báo đến những người đăng ký nhận thông báo.

Hệ thống cũng hỗ trợ điều khiển relay từ xa thông qua các MQTT command topic.

---

# Lợi ích

Giải pháp được đề xuất mang lại các lợi ích sau:

- Giao tiếp bảo mật thông qua MQTT over TLS 1.2.
- Xác thực thiết bị bằng chứng chỉ X.509.
- Lưu trữ telemetry tập trung.
- Gửi email thông báo tự động.
- Chi phí vận hành thấp.
- Kiến trúc đơn giản sử dụng hoàn toàn các dịch vụ được quản lý của AWS.
- Dễ dàng mở rộng để tích hợp thêm các thiết bị Smart Home trong tương lai.

---

# 3. Kiến trúc giải pháp

Hệ thống Smart Home IoT bao gồm một thiết bị ESP32-S3 kết nối với nhiều cảm biến môi trường và các dịch vụ AWS Cloud.

ESP32-S3 định kỳ thu thập dữ liệu telemetry và gửi các thông điệp JSON đến AWS IoT Core.

AWS IoT Core xác thực thiết bị bằng chứng chỉ X.509 và AWS IoT Policy trước khi chuyển tiếp dữ liệu đến AWS IoT Rules Engine.

AWS IoT Rules Engine lưu dữ liệu telemetry vào Amazon DynamoDB và gửi cảnh báo mở cửa thông qua Amazon SNS.

Kiến trúc đề xuất được minh họa trong hình dưới đây.

![Kiến trúc hệ thống Smart Home IoT](/fcj-workshop-template/images/workshop/5.2/architec.jpg)

*Hình: Kiến trúc đề xuất của hệ thống Smart Home IoT trên AWS.*

---

# Các dịch vụ AWS sử dụng

- AWS IoT Core
- AWS IoT Rules Engine
- Amazon DynamoDB
- Amazon Simple Notification Service (Amazon SNS)
- AWS Identity and Access Management (AWS IAM)
- Amazon CloudWatch

---

# Thành phần phần cứng

- Bo mạch phát triển ESP32-S3
- Cảm biến nhiệt độ và độ ẩm DHT11
- Cảm biến ánh sáng LDR
- Cảm biến cửa từ
- Module Relay

---

# 4. Triển khai kỹ thuật

## Các giai đoạn triển khai

Dự án được chia thành bốn giai đoạn triển khai.

### Giai đoạn 1

Phân tích yêu cầu và thiết kế kiến trúc hệ thống.

### Giai đoạn 2

Cấu hình AWS IoT Core, bao gồm tạo Thing, chứng chỉ X.509, IoT Policy và kiểm thử MQTT.

### Giai đoạn 3

Phát triển firmware cho ESP32-S3, bao gồm kết nối Wi-Fi, giao tiếp MQTT over TLS, tạo telemetry và điều khiển relay.

### Giai đoạn 4

Tích hợp với Cloud, kiểm thử hệ thống, xác nhận hoạt động và hoàn thiện tài liệu.

---

# Yêu cầu kỹ thuật

### Phần mềm

- Visual Studio Code
- PlatformIO
- AWS Management Console

### Dịch vụ AWS

- AWS IoT Core
- Amazon DynamoDB
- Amazon SNS
- AWS IAM

### Ngôn ngữ lập trình

- C++
- Arduino Framework

### Giao thức truyền thông

- MQTT over TLS 1.2

---

# 5. Tiến độ thực hiện

| Tuần | Nội dung thực hiện |
|---|---|
| Tuần 1 | Phân tích yêu cầu và nghiên cứu AWS |
| Tuần 2 | Cấu hình AWS IoT Core và môi trường phát triển |
| Tuần 3 | Phát triển firmware và cấu hình các dịch vụ Cloud |
| Tuần 4 | Tích hợp ESP32-S3 với AWS IoT Core |
| Tuần 5 | Tối ưu kiến trúc hệ thống và firmware |
| Tuần 6 | Kiểm thử toàn bộ hệ thống |
| Tuần 7 | Hoàn thiện tài liệu và chuẩn bị báo cáo cuối kỳ |

---

# 6. Dự toán chi phí

Dự án sử dụng các dịch vụ thuộc AWS Free Tier bất cứ khi nào có thể.

Chi phí vận hành dự kiến rất thấp vì:

- Lượng thông điệp AWS IoT Core ở mức nhỏ.
- Amazon DynamoDB chỉ lưu trữ dữ liệu telemetry có dung lượng nhỏ.
- Amazon SNS chỉ gửi thông báo khi có sự kiện.
- Amazon CloudWatch chỉ được sử dụng cho mục đích giám sát và ghi log.

Chi phí phần cứng bao gồm:

- Bo mạch phát triển ESP32-S3
- Cảm biến DHT11
- Cảm biến LDR
- Cảm biến cửa từ
- Module Relay

---

# 7. Đánh giá rủi ro

## Các rủi ro tiềm ẩn

- Mất kết nối Wi-Fi.
- Gián đoạn giao tiếp MQTT.
- Cảm biến hoạt động không chính xác.
- Cấu hình AWS sai.
- Email thông báo bị chậm.

---

## Biện pháp giảm thiểu

- Triển khai cơ chế tự động kết nối lại Wi-Fi.
- Triển khai cơ chế tự động kết nối lại MQTT.
- Kiểm tra tính hợp lệ của dữ liệu cảm biến.
- Áp dụng nguyên tắc phân quyền tối thiểu (Least Privilege) trong AWS IAM.
- Kiểm thử AWS IoT Rules trước khi triển khai.

---

# 8. Kết quả mong đợi

Hệ thống Smart Home IoT sau khi hoàn thành sẽ cung cấp:

- Giao tiếp bảo mật giữa ESP32-S3 và AWS IoT Core.
- Giám sát các điều kiện môi trường theo thời gian thực.
- Điều khiển relay từ xa thông qua MQTT.
- Gửi email cảnh báo tự động khi phát hiện cửa mở.
- Lưu trữ telemetry tập trung trên Amazon DynamoDB.
- Kiến trúc có khả năng mở rộng cho các thiết bị Smart Home trong tương lai.

Dự án cũng cung cấp một ví dụ thực tế về việc tích hợp hệ thống nhúng với các dịch vụ AWS Cloud để xây dựng các ứng dụng Internet of Things (IoT).