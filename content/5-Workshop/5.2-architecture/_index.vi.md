---
title: "Kiến trúc hệ thống"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---

{{% notice info %}}
Chương này trình bày kiến trúc tổng thể của hệ thống Smart Home IoT, mô tả cách ESP32-S3 tương tác với các dịch vụ AWS và giải thích luồng dữ liệu từ thiết bị vật lý đến nền tảng điện toán đám mây.
{{% /notice %}}

# Kiến trúc hệ thống

Hệ thống Smart Home IoT được xây dựng theo kiến trúc **Serverless**, **Event-Driven** và **Cloud-Native** bằng cách sử dụng hoàn toàn các dịch vụ được quản lý trên Amazon Web Services (AWS).

Khác với các hệ thống IoT truyền thống phải triển khai máy chủ ứng dụng, cơ sở dữ liệu và MQTT Broker riêng, kiến trúc này tận dụng các dịch vụ AWS để thực hiện xác thực thiết bị, truyền nhận dữ liệu, xử lý telemetry, lưu trữ dữ liệu và gửi thông báo theo sự kiện.

Toàn bộ hệ thống được chia thành hai môi trường chính:

- Smart Home Environment
- AWS Cloud

Sơ đồ dưới đây minh họa toàn bộ kiến trúc được triển khai trong workshop.

![Kiến trúc Smart Home IoT](/fcj-workshop-template/images/workshop/5.2/architec.jpg)

*Hình: Kiến trúc đề xuất của hệ thống Smart Home IoT.*

# Tổng quan kiến trúc

Trong hệ thống này, ESP32-S3 đóng vai trò là bộ điều khiển trung tâm. Thiết bị định kỳ đọc dữ liệu từ các cảm biến, đóng gói dữ liệu dưới dạng JSON và gửi lên AWS IoT Core thông qua giao thức MQTT được bảo mật bằng TLS 1.2.

AWS IoT Core xác thực thiết bị bằng X.509 Device Certificate và AWS IoT Policy trước khi cho phép giao tiếp MQTT.

Sau khi nhận được telemetry, AWS IoT Rules Engine sẽ xử lý và định tuyến dữ liệu đến các dịch vụ AWS phù hợp. Dữ liệu cảm biến được lưu vào Amazon DynamoDB, trong khi các sự kiện mở cửa sẽ kích hoạt Amazon SNS gửi email thông báo đến người dùng.

Nhờ sử dụng hoàn toàn các dịch vụ được quản lý trên AWS, hệ thống không cần triển khai máy chủ backend nhưng vẫn đảm bảo khả năng mở rộng, tính bảo mật và độ tin cậy cao.

---

# Các thành phần của hệ thống

Kiến trúc Smart Home được chia thành các thành phần logic sau.

# Smart Home Environment

Đây là lớp thiết bị vật lý được triển khai trong môi trường Smart Home.

## ESP32-S3 Development Board

ESP32-S3 là bộ điều khiển trung tâm của toàn bộ hệ thống.

Thiết bị đảm nhiệm các chức năng sau:

- Kết nối mạng Wi-Fi.
- Đồng bộ thời gian bằng NTP.
- Đọc dữ liệu từ các cảm biến.
- Tạo telemetry theo định dạng JSON.
- Publish MQTT telemetry.
- Subscribe MQTT command.
- Điều khiển relay.

ESP32-S3 là cầu nối giữa hệ thống Smart Home và nền tảng AWS Cloud.

---

## DHT11 Sensor

Cảm biến DHT11 được sử dụng để đo:

- Nhiệt độ môi trường.
- Độ ẩm tương đối.

Các giá trị này được cập nhật định kỳ và gửi lên AWS IoT Core cùng với các thông tin khác của hệ thống.

---

## LDR Sensor

Cảm biến LDR (Light Dependent Resistor) được sử dụng để đo cường độ ánh sáng môi trường.

Giá trị đọc được phản ánh mức độ sáng tại khu vực lắp đặt hệ thống và được lưu trong telemetry.

---

## Magnetic Door Sensor

Cảm biến cửa từ có nhiệm vụ xác định trạng thái đóng hoặc mở cửa.

Khi phát hiện cửa mở (`door_open = true`), AWS IoT Rules Engine sẽ kích hoạt quy tắc gửi cảnh báo đến Amazon SNS.

---

## Relay Module

Relay đại diện cho một thiết bị điện có thể điều khiển từ xa như:

- Đèn.
- Quạt.
- Ổ cắm điện.
- Thiết bị điện dân dụng.

Relay không được điều khiển trực tiếp bằng nút nhấn mà nhận lệnh từ AWS IoT Core thông qua MQTT command.

---

# AWS Cloud

Toàn bộ phía Cloud được xây dựng bằng các dịch vụ AWS được quản lý hoàn toàn.

Workshop được triển khai tại Region:

```text
US East (N. Virginia)
us-east-1
```

---

# AWS IoT Core

AWS IoT Core là thành phần trung tâm của hệ thống.

Dịch vụ này chịu trách nhiệm:

- Quản lý danh tính thiết bị.
- Xác thực thiết bị.
- Truyền nhận MQTT.
- Định tuyến telemetry.

AWS IoT Core bao gồm ba thành phần quan trọng.

## Thing Registry

ESP32-S3 được đăng ký dưới dạng một **AWS IoT Thing**.

Thing Registry lưu trữ thông tin định danh của từng thiết bị IoT và cho phép quản lý nhiều thiết bị trên cùng một hệ thống.

---

## AWS IoT Message Broker

AWS IoT Message Broker chịu trách nhiệm truyền nhận MQTT message giữa ESP32-S3 và các dịch vụ AWS.

Message Broker hỗ trợ:

- MQTT Publish
- MQTT Subscribe
- MQTT QoS
- MQTT over TLS 1.2

ESP32-S3 publish telemetry đến topic:

```text
smarthome/esp32-home-01/telemetry
```

và subscribe topic:

```text
smarthome/esp32-home-01/command
```

để nhận lệnh điều khiển relay.

---

## AWS IoT Rules Engine

AWS IoT Rules Engine tự động xử lý các MQTT message nhận được.

Thay vì xây dựng một backend riêng để xử lý telemetry, Rules Engine sẽ chuyển tiếp dữ liệu trực tiếp đến các dịch vụ AWS khác.

Trong workshop này, hai quy tắc được sử dụng.

### Telemetry Rule

Quy tắc này lưu toàn bộ telemetry vào Amazon DynamoDB.

### Door Alert Rule

Quy tắc này kiểm tra trường:

```text
door_open == true
```

Nếu điều kiện đúng, AWS IoT Rules Engine sẽ publish một thông báo đến Amazon SNS.

---

# Amazon DynamoDB

Amazon DynamoDB là dịch vụ cơ sở dữ liệu NoSQL được AWS quản lý hoàn toàn.

DynamoDB được sử dụng để lưu toàn bộ dữ liệu telemetry của hệ thống Smart Home.

Một bản ghi telemetry bao gồm:

| Thuộc tính | Ý nghĩa |
|------------|---------|
| device_id | Định danh thiết bị |
| timestamp | Thời điểm ghi nhận |
| temperature | Nhiệt độ |
| humidity | Độ ẩm |
| light | Cường độ ánh sáng |
| door_open | Trạng thái cửa |
| relay_on | Trạng thái relay |

Việc sử dụng DynamoDB giúp hệ thống có khả năng mở rộng tốt mà không cần quản lý máy chủ cơ sở dữ liệu.

---

# Amazon SNS

Amazon Simple Notification Service (Amazon SNS) cung cấp cơ chế gửi thông báo theo sự kiện.

Khi AWS IoT Rules Engine phát hiện cửa được mở, SNS sẽ gửi email đến tất cả các Subscriber đã đăng ký.

Luồng xử lý như sau:

```text
Door Sensor

↓

AWS IoT Rules Engine

↓

Amazon SNS Topic

↓

Subscriber Email
```

---

# Xác thực và bảo mật

Một trong những yêu cầu quan trọng của hệ thống IoT là đảm bảo an toàn trong quá trình truyền nhận dữ liệu.

Kiến trúc sử dụng nhiều cơ chế bảo mật khác nhau.

## X.509 Device Certificate

Mỗi ESP32-S3 được cấp một X.509 Device Certificate duy nhất.

Certificate được sử dụng để xác thực thiết bị trước khi kết nối MQTT được thiết lập.

---

## AWS IoT Policy

AWS IoT Policy quy định các thao tác MQTT mà thiết bị được phép thực hiện.

Bao gồm:

- iot:Connect
- iot:Publish
- iot:Subscribe
- iot:Receive

Nhờ đó chỉ những thiết bị được cấp quyền mới có thể giao tiếp với AWS IoT Core.

---

## AWS IAM Role

AWS IAM Role được sử dụng bởi AWS IoT Rules Engine.

IAM Role không xác thực thiết bị mà cấp quyền cho Rules Engine thực hiện các thao tác trên dịch vụ AWS khác.

Trong workshop này IAM Role cho phép:

- Ghi dữ liệu vào Amazon DynamoDB.
- Publish thông báo đến Amazon SNS.

---

# Giám sát hệ thống

Ngoài việc xử lý telemetry, kiến trúc còn sử dụng các dịch vụ giám sát của AWS.

## Amazon CloudWatch

Amazon CloudWatch hỗ trợ:

- Theo dõi Metrics.
- Thu thập Logs.
- Giám sát hoạt động hệ thống.
- Hỗ trợ xử lý lỗi.

---

## AWS CloudTrail

AWS CloudTrail ghi nhận:

- AWS API Calls.
- Thay đổi cấu hình.
- Hoạt động bảo mật.

CloudTrail hỗ trợ kiểm tra lịch sử thao tác và phục vụ mục đích kiểm toán hệ thống.

---

# Luồng hoạt động của hệ thống

Hệ thống Smart Home hoạt động theo trình tự sau.

1. ESP32-S3 kết nối mạng Wi-Fi.
2. Thiết bị đồng bộ thời gian bằng NTP.
3. Đọc dữ liệu từ các cảm biến.
4. Tạo telemetry theo định dạng JSON.
5. Publish telemetry bằng MQTT over TLS.
6. AWS IoT Core xác thực thiết bị.
7. AWS IoT Rules Engine xử lý telemetry.
8. Telemetry được lưu vào Amazon DynamoDB.
9. Nếu cửa mở, Amazon SNS gửi email cảnh báo.
10. AWS IoT Core có thể publish MQTT command.
11. ESP32-S3 nhận command và cập nhật trạng thái relay.

---

# Lý do lựa chọn kiến trúc

Kiến trúc này được lựa chọn vì mang lại nhiều ưu điểm:

- Sử dụng hoàn toàn dịch vụ AWS được quản lý.
- Không cần triển khai máy chủ backend.
- Giao tiếp MQTT bảo mật bằng TLS 1.2.
- Xác thực thiết bị bằng X.509 Device Certificate.
- Xử lý dữ liệu theo mô hình Event-Driven.
- Dễ dàng mở rộng khi số lượng thiết bị tăng.
- Chi phí thấp theo mô hình Pay-as-you-go.
- Dễ tích hợp với các bo mạch ESP32.

Kiến trúc phù hợp cho các hệ thống Smart Home quy mô nhỏ, các dự án nghiên cứu, đào tạo và nguyên mẫu IoT.

{{% notice tip %}}
Trong các chương tiếp theo, workshop sẽ lần lượt hướng dẫn cách tạo tài nguyên trên AWS, cấu hình ESP32-S3 và triển khai từng thành phần của kiến trúc đã trình bày trong chương này.
{{% /notice %}}

**Tiếp theo:** [Điều kiện chuẩn bị](../5.3-prerequisites/)