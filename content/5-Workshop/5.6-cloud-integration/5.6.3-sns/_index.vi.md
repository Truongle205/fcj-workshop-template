---
title: "Cấu hình Amazon SNS"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 5.6.3 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo Amazon Simple Notification Service Topic cho cảnh báo mở cửa. AWS IoT Rules Engine sẽ publish thông báo đến topic này khi telemetry chứa `door_open = true`.
{{% /notice %}}

# Tổng quan

Amazon Simple Notification Service, hay Amazon SNS, là dịch vụ messaging được AWS quản lý hoàn toàn, dùng để phân phối thông báo đến một hoặc nhiều subscriber.

Trong hệ thống Smart Home, Amazon SNS đóng vai trò là thành phần gửi cảnh báo.

Khi ESP32-S3 phát hiện cửa mở, thiết bị publish telemetry có trường:

```json
{
  "door_open": true
}
```

AWS IoT Rules Engine đánh giá message và publish cảnh báo đến SNS topic đã cấu hình.

Luồng thông báo:

```text
ESP32-S3
     ↓
Telemetry message
     ↓
AWS IoT Rules Engine
     ↓
Door Alert Rule
     ↓
Amazon SNS Topic
     ↓
Email Subscription
```

Nhờ Amazon SNS, ESP32-S3 không cần kết nối trực tiếp đến email server.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Mở giao diện Amazon SNS.
- Tạo SNS Topic.
- Phân biệt Standard Topic và FIFO Topic.
- Cấu hình topic cho cảnh báo Smart Home.
- Lấy SNS Topic ARN.
- Liên kết topic với AWS IoT Rule.
- Kiểm tra chức năng publish message.

---

# Thời gian thực hiện

**Khoảng 5–10 phút**

---

# Tại sao chọn Amazon SNS?

Amazon SNS được lựa chọn vì cung cấp:

- Dịch vụ thông báo được quản lý hoàn toàn.
- Tích hợp theo mô hình event-driven với AWS IoT Rules Engine.
- Hỗ trợ email subscription.
- Không cần quản lý SMTP server.
- Có thể phân phối message đến nhiều subscriber.
- Chi phí theo mô hình pay-as-you-go.

ESP32-S3 chỉ gửi telemetry. Các dịch vụ AWS chịu trách nhiệm xử lý cảnh báo.

---

# Bước 1 - Mở Amazon SNS

Đăng nhập AWS Management Console.

Tìm kiếm:

```text
Simple Notification Service
```

Mở **Amazon SNS**.

Từ menu bên trái, chọn:

```text
Topics
```

Sau đó chọn:

```text
Create topic
```

---

# Bước 2 - Chọn loại Topic

Chọn:

```text
Standard
```

Standard SNS Topic phù hợp cho workshop vì cung cấp:

- Throughput cao.
- Thứ tự message theo cơ chế best effort.
- Cơ chế gửi ít nhất một lần.
- Hỗ trợ email subscription.

Không cần FIFO Topic vì cảnh báo mở cửa không yêu cầu thứ tự nghiêm ngặt hoặc loại bỏ message trùng lặp ở mức topic.

---

# Bước 3 - Cấu hình Topic

Nhập tên topic.

Ví dụ:

```text
SmartHomeDoorAlert
```

Có thể nhập Display Name:

```text
Smart Home Door Alert
```

Display Name có thể được hiển thị với một số giao thức nhận thông báo được hỗ trợ.

Giữ các thiết lập còn lại ở giá trị mặc định nếu project không yêu cầu mã hóa hoặc access policy riêng.

Chọn:

```text
Create topic
```

---

# Bước 4 - Kiểm tra SNS Topic

Sau khi tạo thành công, Amazon SNS hiển thị trang thông tin topic.

Kiểm tra:

- Topic name.
- Topic type.
- Topic ARN.
- Region.
- Số lượng subscription.

![Amazon SNS Topic](/images/workshop/5.6.3/sns-topic.png)

Topic ARN có cấu trúc:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

ARN định danh duy nhất SNS Topic và được sử dụng khi cấu hình AWS IoT Rule Action.

{{% notice warning %}}
Không nên công khai AWS Account ID trong ảnh báo cáo. Có thể che hoặc làm mờ Account ID trước khi đưa ảnh lên website.
{{% /notice %}}

---

# Bước 5 - Cấu hình AWS IoT Rule Action

Quay lại:

```text
AWS IoT Core
→ Message routing
→ Rules
→ SmartHomeDoorAlert
```

Mở phần cấu hình action.

Chọn action publish message đến Amazon SNS Topic.

Chọn topic:

```text
SmartHomeDoorAlert
```

hoặc nhập Topic ARN:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

IoT Rule sử dụng SQL:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

Khi điều kiện đúng, telemetry message được publish đến SNS Topic.

---

# Bước 6 - Cấu hình quyền IAM

AWS IoT Rules Engine cần quyền publish đến SNS Topic.

IAM Role gắn với rule phải cho phép:

```text
sns:Publish
```

Ví dụ permission policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert"
    }
  ]
}
```

Trust relationship phải cho phép AWS IoT assume role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

{{% notice note %}}
AWS IAM Role cấp quyền cho AWS IoT Rules Engine. Role này không được ESP32-S3 sử dụng và không liên quan đến quá trình xác thực MQTT của thiết bị.
{{% /notice %}}

---

# Bước 7 - Publish Test Message trực tiếp từ SNS

Trước khi kiểm tra toàn bộ luồng IoT, có thể thử SNS Topic trực tiếp.

Tại trang Topic Details, chọn:

```text
Publish message
```

Nhập Subject:

```text
Smart Home Door Alert Test
```

Nhập Message:

```text
This is a test notification from Amazon SNS.
```

Chọn:

```text
Publish message
```

Ở thời điểm này, message chỉ được gửi đến người nhận khi đã có subscription được tạo và xác nhận.

Phần tạo subscription sẽ được trình bày ở mục tiếp theo.

---

# Bước 8 - Kiểm tra đầu vào AWS IoT Rule

Mở:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Publish đến topic:

```text
smarthome/esp32-home-01/telemetry
```

Payload:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

Vì `door_open = true`, Door Alert Rule sẽ publish message đến Amazon SNS.

Publish message khác với:

```json
{
  "door_open": false
}
```

Message này không được kích hoạt SNS action.

---

# Định dạng Message

Theo cấu hình mặc định, toàn bộ message do IoT SQL chọn có thể được publish đến Amazon SNS.

Ví dụ:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

Có thể tạo nội dung cảnh báo ngắn gọn hơn bằng cách thay đổi IoT SQL statement hoặc bổ sung thành phần xử lý khác.

Trong workshop này, sử dụng toàn bộ telemetry payload là đủ để minh họa luồng cảnh báo.

---

# Bảo mật

SNS Topic không nên cho phép public publish.

Chỉ IAM Role của AWS IoT Rule được cấp quyền publish đến topic.

Khuyến nghị:

- Giới hạn `sns:Publish` theo đúng Topic ARN.
- Không dùng wildcard permission trong production.
- Không cho phép anonymous public publishing.
- Kiểm tra SNS Access Policy.
- Che email trong ảnh chụp màn hình.
- Xóa subscription không sử dụng sau khi kiểm thử.

---

# Giám sát

Amazon SNS cung cấp metric thông qua Amazon CloudWatch.

Một số metric hữu ích:

- Số message được publish.
- Số notification được gửi thành công.
- Số notification thất bại.
- Số notification bị filter.

Nếu rule được kích hoạt nhưng không nhận email, kiểm tra:

1. SNS Topic tồn tại trong `us-east-1`.
2. IoT Rule Action sử dụng đúng Topic ARN.
3. IAM Role có quyền `sns:Publish`.
4. Subscription có trạng thái `Confirmed`.
5. Email không nằm trong Spam hoặc Junk.

---

# Xử lý lỗi

## SNS Topic không nhận Message

Kiểm tra:

- Door Alert Rule đang Enabled.
- SQL condition là `door_open = true`.
- Trường `door_open` trong payload là Boolean.
- IoT Rule Action trỏ đúng Topic.
- IAM Role có `sns:Publish`.

---

## Publish Test Message thành công nhưng không có Email

Trường hợp này xảy ra khi chưa có Email Subscription đã xác nhận.

Tạo và xác nhận Email Subscription ở mục tiếp theo.

---

## Access Denied

Kiểm tra IAM Role Permission Policy.

Resource phải khớp chính xác Topic ARN:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

Đồng thời kiểm tra rule có sử dụng đúng IAM Role hay không.

---

## Nhận nhiều thông báo trùng nhau

Nguyên nhân có thể:

- Có nhiều subscription cùng sử dụng một email.
- Có nhiều IoT Rule cùng khớp một message.
- ESP32-S3 liên tục gửi `door_open = true`.
- SNS sử dụng cơ chế at-least-once delivery.

Trong hệ thống production, có thể bổ sung logic chỉ gửi cảnh báo khi trạng thái cửa chuyển từ đóng sang mở, thay vì gửi ở mọi telemetry message khi cửa vẫn mở.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Standard Amazon SNS Topic đã được tạo.
- Topic ARN đã sẵn sàng.
- Door Alert Rule trỏ đến SNS Topic.
- IAM Role có thể thực hiện `sns:Publish`.
- Có thể publish Test Message đến Topic.
- Telemetry mở cửa có thể kích hoạt SNS workflow.

{{% notice tip %}}
Trong mục tiếp theo, người thực hiện sẽ tạo và xác nhận Email Subscription để nhận thông báo từ SNS Topic.
{{% /notice %}}

**Tiếp theo:** [5.6.4 Cấu hình Email Notification](../5.6.4-email-notification/)