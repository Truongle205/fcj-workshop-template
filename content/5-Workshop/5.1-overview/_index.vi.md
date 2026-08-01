---
title: "Tổng quan"
date: 2026-08-01
weight: 1
chapter: false
pre: " <b> 5.1 </b> "
---

{{% notice info %}}
Workshop này hướng dẫn xây dựng hệ thống Smart Home Internet of Things (IoT) sử dụng bo mạch ESP32-S3 kết hợp với các dịch vụ đám mây của Amazon Web Services (AWS). Hệ thống có khả năng thu thập dữ liệu từ các cảm biến, truyền dữ liệu bảo mật thông qua MQTT over TLS 1.2, lưu trữ telemetry trên Amazon DynamoDB và gửi email cảnh báo bằng Amazon Simple Notification Service (Amazon SNS).
{{% /notice %}}

# Hệ thống Smart Home IoT trên AWS

## Giới thiệu

Internet of Things (IoT) là một trong những công nghệ quan trọng của các hệ thống thông minh hiện nay. Bằng cách kết nối các thiết bị vật lý với nền tảng điện toán đám mây, dữ liệu có thể được thu thập, xử lý, lưu trữ và trao đổi theo thời gian thực. Trong đó, Smart Home là một trong những ứng dụng IoT phổ biến nhất, cho phép người dùng theo dõi môi trường sống và điều khiển các thiết bị trong gia đình từ xa.

Amazon Web Services (AWS) cung cấp một hệ sinh thái IoT hoàn chỉnh, giúp xây dựng các ứng dụng IoT an toàn và có khả năng mở rộng mà không cần triển khai hay quản lý máy chủ riêng. Thay vì xây dựng backend truyền thống, AWS IoT Core cùng với các dịch vụ AWS khác đảm nhiệm việc xác thực thiết bị, truyền nhận dữ liệu, lưu trữ telemetry và xử lý các sự kiện phát sinh.

Trong workshop này, hệ thống Smart Home được xây dựng bằng bo mạch ESP32-S3 và các dịch vụ AWS. ESP32-S3 sẽ định kỳ đọc dữ liệu từ các cảm biến, đóng gói dữ liệu dưới dạng JSON và gửi lên AWS IoT Core thông qua giao thức MQTT được bảo mật bằng TLS 1.2.

AWS IoT Core xác thực thiết bị bằng X.509 Device Certificate, sau đó chuyển các telemetry message đến AWS IoT Rules Engine để xử lý. Dữ liệu được lưu vào Amazon DynamoDB, đồng thời khi phát hiện cửa mở, Amazon SNS sẽ tự động gửi email cảnh báo đến người dùng. Ngoài ra, ESP32-S3 còn subscribe một MQTT command topic để nhận lệnh điều khiển relay từ AWS IoT Core.

Toàn bộ hệ thống được xây dựng theo mô hình **serverless** và **event-driven**, giúp giảm chi phí vận hành, tăng tính bảo mật và dễ dàng mở rộng trong tương lai.

---

# Mục tiêu của Workshop

Sau khi hoàn thành workshop này, người thực hiện sẽ có thể:

- Hiểu kiến trúc tổng thể của một hệ thống Smart Home IoT trên AWS.
- Thiết lập môi trường phát triển ESP32-S3 bằng PlatformIO.
- Kết nối ESP32-S3 với mạng Wi-Fi ở chế độ Station Mode.
- Đồng bộ thời gian hệ thống bằng Network Time Protocol (NTP).
- Tạo và cấu hình AWS IoT Thing.
- Tạo và kích hoạt X.509 Device Certificate.
- Cấu hình AWS IoT Policy cho thiết bị.
- Thiết lập kết nối MQTT over TLS 1.2 giữa ESP32-S3 và AWS IoT Core.
- Gửi dữ liệu telemetry theo định dạng JSON.
- Subscribe MQTT command topic để điều khiển relay từ xa.
- Tạo AWS IoT Rule để xử lý telemetry.
- Lưu dữ liệu vào Amazon DynamoDB.
- Gửi email cảnh báo bằng Amazon SNS.
- Kiểm thử toàn bộ luồng hoạt động của hệ thống Smart Home.

---

# Tổng quan hệ thống

Hệ thống Smart Home được chia thành hai thành phần chính:

- **Thiết bị Smart Home**
- **AWS Cloud**

ESP32-S3 đóng vai trò là bộ điều khiển trung tâm của hệ thống. Thiết bị định kỳ đọc dữ liệu từ các cảm biến, đóng gói thành telemetry JSON và gửi an toàn lên AWS IoT Core.

AWS IoT Core xác thực thiết bị, tiếp nhận MQTT message và chuyển dữ liệu đến AWS IoT Rules Engine để xử lý.

Các bản ghi telemetry được lưu trong Amazon DynamoDB, trong khi các sự kiện mở cửa sẽ kích hoạt Amazon SNS gửi email cảnh báo đến người dùng đã đăng ký. Đồng thời AWS IoT Core cũng có thể gửi MQTT command xuống ESP32-S3 để điều khiển relay từ xa.

Quá trình này tạo thành một hệ thống IoT hoàn chỉnh bao gồm thu thập dữ liệu, truyền dữ liệu bảo mật, lưu trữ trên cloud, xử lý sự kiện và điều khiển thiết bị hai chiều.

---

# Thành phần phần cứng

Mô hình Smart Home sử dụng các thiết bị phần cứng sau.

| Thành phần | Mô tả |
|------------|------|
| ESP32-S3 Development Board | Bộ điều khiển trung tâm thực hiện đọc cảm biến, giao tiếp MQTT và điều khiển relay. |
| DHT11 Sensor | Đo nhiệt độ và độ ẩm môi trường. |
| LDR Sensor | Đo cường độ ánh sáng môi trường. |
| Magnetic Door Sensor | Phát hiện trạng thái đóng hoặc mở cửa. |
| Relay Module | Điều khiển thiết bị điện như đèn hoặc quạt. |
| Wi-Fi Network | Kết nối ESP32-S3 với AWS IoT Core thông qua Internet. |

---

# Các dịch vụ AWS sử dụng

Workshop sử dụng các dịch vụ AWS sau.

| Dịch vụ AWS | Chức năng |
|-------------|-----------|
| AWS IoT Core | Cung cấp giao tiếp MQTT và xác thực thiết bị. |
| AWS IoT Rules Engine | Xử lý và định tuyến telemetry đến các dịch vụ AWS khác. |
| Amazon DynamoDB | Lưu trữ dữ liệu telemetry của hệ thống Smart Home. |
| Amazon Simple Notification Service (Amazon SNS) | Gửi email cảnh báo khi phát hiện cửa mở. |
| AWS Identity and Access Management (AWS IAM) | Cấp quyền cho AWS IoT Rule truy cập DynamoDB và SNS. |
| Amazon CloudWatch | Theo dõi log, metric và trạng thái hoạt động của hệ thống. |
| AWS CloudTrail | Ghi nhận các hoạt động API và thay đổi cấu hình trên AWS. |

---

# Các chức năng của hệ thống

Hệ thống Smart Home được xây dựng với các chức năng sau.

| Chức năng | Mô tả |
|-----------|------|
| Giám sát nhiệt độ | Đọc dữ liệu nhiệt độ từ cảm biến DHT11. |
| Giám sát độ ẩm | Đọc dữ liệu độ ẩm từ cảm biến DHT11. |
| Giám sát ánh sáng | Đo cường độ ánh sáng bằng cảm biến LDR. |
| Giám sát cửa | Xác định trạng thái đóng hoặc mở cửa. |
| Điều khiển relay từ xa | Điều khiển relay thông qua MQTT command. |
| Giao tiếp bảo mật | Sử dụng MQTT over TLS 1.2 kết hợp X.509 Certificate. |
| Gửi telemetry lên Cloud | Gửi dữ liệu cảm biến lên AWS IoT Core. |
| Lưu trữ dữ liệu | Lưu telemetry vào Amazon DynamoDB. |
| Gửi email cảnh báo | Gửi email thông qua Amazon SNS khi cửa mở. |

---

# MQTT Topic

Hệ thống sử dụng hai MQTT topic.

| MQTT Topic | Chiều truyền | Chức năng |
|------------|-------------|-----------|
| `smarthome/esp32-home-01/telemetry` | Publish | Gửi telemetry từ ESP32-S3 lên AWS IoT Core. |
| `smarthome/esp32-home-01/command` | Subscribe | Nhận lệnh điều khiển relay từ AWS IoT Core. |

---

# Ví dụ Telemetry

ESP32-S3 định kỳ gửi telemetry theo định dạng JSON.

```json
{
    "device_id": "esp32-home-01",
    "temperature": 29.6,
    "humidity": 71.0,
    "light": 842,
    "door_open": false,
    "relay_on": true
}
```

Ý nghĩa của từng trường dữ liệu.

| Trường dữ liệu | Mô tả |
|---------------|------|
| device_id | Định danh của thiết bị Smart Home. |
| temperature | Nhiệt độ môi trường (°C). |
| humidity | Độ ẩm tương đối (%RH). |
| light | Giá trị cường độ ánh sáng đọc từ cảm biến LDR. |
| door_open | Trạng thái cửa (true = mở, false = đóng). |
| relay_on | Trạng thái hiện tại của relay. |

---

# Ví dụ Command

Để điều khiển relay từ xa, AWS IoT Core sẽ publish JSON command như sau.

```json
{
    "relay_on": true
}
```

ESP32-S3 subscribe command topic, phân tích JSON và cập nhật trạng thái relay tương ứng.

---

# Luồng hoạt động của hệ thống

Sau khi hoàn thành workshop, hệ thống hoạt động theo trình tự sau.

1. ESP32-S3 đọc nhiệt độ, độ ẩm, ánh sáng và trạng thái cửa.
2. Thiết bị đóng gói dữ liệu thành JSON telemetry.
3. Telemetry được gửi bảo mật đến AWS IoT Core bằng MQTT over TLS 1.2.
4. AWS IoT Core xác thực thiết bị bằng X.509 Device Certificate.
5. AWS IoT Rules Engine xử lý telemetry message.
6. Telemetry được lưu vào Amazon DynamoDB.
7. Khi cửa được mở, Amazon SNS tự động gửi email cảnh báo.
8. AWS IoT Core có thể publish MQTT command để điều khiển relay.
9. ESP32-S3 nhận command và thay đổi trạng thái relay.

---

# Kết quả đạt được

Sau khi hoàn thành workshop, người thực hiện sẽ xây dựng thành công một hệ thống Smart Home IoT có khả năng:

- Xác thực thiết bị bằng X.509 Device Certificate.
- Thiết lập kết nối MQTT over TLS 1.2 với AWS IoT Core.
- Gửi dữ liệu telemetry theo thời gian thực.
- Xử lý dữ liệu bằng AWS IoT Rules Engine.
- Lưu telemetry vào Amazon DynamoDB.
- Gửi email cảnh báo thông qua Amazon SNS.
- Điều khiển relay từ xa bằng MQTT command.

Hệ thống là một ví dụ thực tế về việc xây dựng ứng dụng IoT sử dụng các dịch vụ AWS được quản lý hoàn toàn, không cần triển khai máy chủ riêng.

{{% notice tip %}}
Trước khi bắt đầu các bước triển khai, nên xem trước sơ đồ kiến trúc của hệ thống ở mục tiếp theo để hiểu rõ luồng dữ liệu giữa ESP32-S3 và các dịch vụ AWS.
{{% /notice %}}

**Tiếp theo:** [Kiến trúc hệ thống](../5.2-architecture/)