---
title: "Tạo AWS IoT Rule"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.6.1 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo các AWS IoT Rule để xử lý telemetry message từ ESP32-S3. Một rule lưu telemetry, trong khi rule còn lại phát hiện sự kiện mở cửa và kích hoạt luồng thông báo.
{{% /notice %}}

# Tổng quan

AWS IoT Rules Engine cho phép xử lý MQTT message mà không cần triển khai một backend application riêng.

Mỗi rule chứa câu lệnh AWS IoT SQL để chọn message từ một MQTT topic. Khi message thỏa mãn câu lệnh SQL, AWS IoT Core sẽ gọi một hoặc nhiều action đã được cấu hình.

Hệ thống Smart Home sử dụng hai rule:

| Rule | Chức năng |
|---|---|
| Telemetry storage rule | Xử lý toàn bộ telemetry để lưu vào Amazon DynamoDB. |
| Door alert rule | Chỉ xử lý message có `door_open = true`. |

Cả hai rule đều đọc dữ liệu từ topic:

```text
smarthome/esp32-home-01/telemetry
```

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Mở giao diện AWS IoT Rules.
- Tạo IoT SQL Rule.
- Chọn telemetry message từ MQTT topic.
- Thêm điều kiện lọc sự kiện mở cửa.
- Cấu hình rule action.
- Liên kết AWS IAM Role với action.
- Kiểm tra trạng thái hoạt động của rule.

---

# Thời gian thực hiện

**Khoảng 10–15 phút**

---

# Luồng xử lý Rule

```text
ESP32-S3 publish telemetry
             ↓
AWS IoT Message Broker
             ↓
AWS IoT Rules Engine
             ↓
Đánh giá AWS IoT SQL
       ┌─────┴─────┐
       │           │
       ▼           ▼
Telemetry Rule   Door Alert Rule
       │           │
       ▼           ▼
DynamoDB          SNS
```

Mỗi rule được đánh giá độc lập đối với cùng một telemetry message.

---

# Bước 1 - Mở AWS IoT Rules

Đăng nhập AWS Management Console và mở **AWS IoT Core**.

Từ menu bên trái, chọn:

```text
Message routing
→ Rules
```

Trang Rules hiển thị toàn bộ AWS IoT Rule đã cấu hình.

![Trang quản lý AWS IoT Rule](/fcj-workshop-template/images/workshop/5.6.1/iot-rules-dashboard.png)

Chọn **Create rule**.

---

# Bước 2 - Cấu hình Telemetry Rule

Nhập tên rule.

Ví dụ:

```text
StoreSmartHomeTelemetry
```

Có thể thêm phần mô tả:

```text
Stores ESP32-S3 Smart Home telemetry in Amazon DynamoDB.
```

{{% notice note %}}
Tên AWS IoT Rule không được chứa khoảng trắng. Hãy tuân theo giới hạn ký tự được AWS Console hiển thị khi tạo rule.
{{% /notice %}}

---

# Bước 3 - Khai báo câu lệnh Telemetry SQL

Sử dụng câu lệnh:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

Câu truy vấn chọn toàn bộ thuộc tính từ mọi message được publish đến telemetry topic.

Dấu `*` bao gồm các trường như:

```text
device_id
timestamp
dht_valid
temperature
humidity
light
door_open
relay_on
```

Chọn phiên bản AWS IoT SQL được AWS Console khuyến nghị, trừ khi project có yêu cầu tương thích với phiên bản cũ.

---

# Bước 4 - Cấu hình Telemetry Action

Chọn action ghi dữ liệu vào Amazon DynamoDB.

Tùy giao diện AWS Console, action có thể hiển thị là:

```text
DynamoDB
```

hoặc:

```text
Split message into multiple columns of a DynamoDB table
```

Cấu hình action để sử dụng telemetry table của hệ thống.

Phần cấu hình DynamoDB chi tiết sẽ được trình bày trong mục tiếp theo.

![Telemetry Rule lưu dữ liệu](/fcj-workshop-template/images/workshop/5.6.1/telemetry-rule.png)

---

# Bước 5 - Cấu hình IAM Role

AWS IoT Rules Engine cần quyền ghi dữ liệu vào DynamoDB.

Chọn một role có sẵn hoặc tạo role mới trực tiếp từ AWS Console.

Role cần cho phép action:

```text
dynamodb:PutItem
```

Resource nên trỏ đến ARN của telemetry table.

Ví dụ:

```text
arn:aws:dynamodb:us-east-1:ACCOUNT_ID:table/SmartHomeTelemetry
```

Trust relationship cần cho phép AWS IoT assume role:

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

{{% notice warning %}}
Không cấp AdministratorAccess cho IoT Rule Role. Chỉ cấp đúng các quyền mà action cần sử dụng.
{{% /notice %}}

---

# Bước 6 - Tạo Telemetry Rule

Kiểm tra lại cấu hình:

| Thiết lập | Giá trị |
|---|---|
| Rule name | `StoreSmartHomeTelemetry` |
| MQTT topic | `smarthome/esp32-home-01/telemetry` |
| SQL condition | Không có |
| Action | Ghi vào Amazon DynamoDB |
| IAM role | IoT Rule DynamoDB role |
| Status | Enabled |

Chọn **Create**.

Rule sẽ xuất hiện trong danh sách AWS IoT Rules.

---

# Bước 7 - Tạo Door Alert Rule

Tạo một rule mới.

Ví dụ tên:

```text
SmartHomeDoorAlert
```

Phần mô tả:

```text
Publishes an Amazon SNS notification when the door is open.
```

---

# Bước 8 - Khai báo Door Alert SQL

Sử dụng:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

Câu truy vấn lọc telemetry message.

Rule chỉ được kích hoạt khi payload chứa:

```json
{
  "door_open": true
}
```

Message có:

```json
{
  "door_open": false
}
```

sẽ không kích hoạt SNS action.

---

# Bước 9 - Cấu hình SNS Action

Chọn action publish message đến Amazon SNS.

Chọn SNS topic dùng cho cảnh báo Smart Home.

Ví dụ:

```text
SmartHomeDoorAlert
```

SNS topic và email subscription sẽ được cấu hình trong các mục tiếp theo.

AWS IoT Rules Engine cũng cần IAM Role có quyền:

```text
sns:Publish
```

Ví dụ resource:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

![Door Alert Rule](/fcj-workshop-template/images/workshop/5.6.1/door-alert-rule.png)

---

# Bước 10 - Tạo Door Alert Rule

Kiểm tra cấu hình:

| Thiết lập | Giá trị |
|---|---|
| Rule name | `SmartHomeDoorAlert` |
| MQTT topic | `smarthome/esp32-home-01/telemetry` |
| SQL condition | `door_open = true` |
| Action | Publish đến Amazon SNS |
| IAM role | IoT Rule SNS role |
| Status | Enabled |

Chọn **Create**.

---

# Bước 11 - Kiểm tra các Rule

Quay lại:

```text
AWS IoT Core
→ Message routing
→ Rules
```

Kiểm tra hai rule đã tồn tại và được bật:

```text
StoreSmartHomeTelemetry
SmartHomeDoorAlert
```

Mở từng rule và xác nhận:

- MQTT topic chính xác.
- SQL statement hợp lệ.
- Action trỏ đúng AWS resource.
- IAM Role đã được cấu hình.
- Rule đang ở trạng thái enabled.

---

# Giải thích AWS IoT SQL

Telemetry Rule sử dụng:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

Ý nghĩa:

| Thành phần SQL | Ý nghĩa |
|---|---|
| `SELECT *` | Lấy toàn bộ trường trong JSON payload. |
| `FROM` | Đọc message từ MQTT topic được chỉ định. |
| Topic trong dấu nháy | MQTT source topic chính xác. |

Door Alert Rule bổ sung:

```sql
WHERE door_open = true
```

Chỉ các message thỏa điều kiện mới kích hoạt action.

---

# Kiểm tra đầu vào của Rule

Mở AWS IoT MQTT Test Client và publish message đến:

```text
smarthome/esp32-home-01/telemetry
```

Ví dụ:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "dht_valid": true,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

Message này phải khớp:

- Telemetry storage rule.
- Door alert rule.

Đổi thành:

```json
{
  "door_open": false
}
```

Message chỉ khớp telemetry storage rule.

---

# Theo dõi hoạt động của Rule

Có thể kiểm tra hoạt động thông qua AWS IoT và Amazon CloudWatch metric.

Các chỉ báo hữu ích:

- Rule matched.
- Action succeeded.
- Action failed.
- Authorization error.
- Target service error.

Khi dữ liệu không đến destination, kiểm tra:

1. ESP32-S3 publish đúng topic.
2. SQL topic khớp hoàn toàn.
3. Rule đã được bật.
4. IAM Role có đủ quyền.
5. Target resource tồn tại tại `us-east-1`.
6. JSON payload có đúng tên trường.

---

# Lỗi thường gặp

## Rule không được kích hoạt

Nguyên nhân có thể:

- Sai tên topic.
- Rule đang bị disable.
- ESP32-S3 chưa publish telemetry.
- SQL statement sai cú pháp.

---

## Door Alert Rule kích hoạt sai

Đảm bảo payload chứa Boolean:

```json
{
  "door_open": true
}
```

Không gửi dưới dạng chuỗi:

```json
{
  "door_open": "true"
}
```

Boolean JSON và string JSON là hai kiểu dữ liệu khác nhau.

---

## Action thất bại

Nguyên nhân có thể:

- Thiếu IAM Role.
- IAM permission không chính xác.
- DynamoDB table hoặc SNS topic chưa tồn tại.
- Resource nằm ở Region khác.
- Action tham chiếu ARN không đúng.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Telemetry storage rule đã được tạo và bật.
- Door alert rule đã được tạo và bật.
- Hai rule cùng đọc Smart Home telemetry topic.
- Telemetry Rule xử lý toàn bộ message.
- Door Alert Rule chỉ xử lý `door_open = true`.
- IAM Role cấp quyền cho các action.
- Telemetry kiểm thử có thể kích hoạt đúng rule.

{{% notice tip %}}
Trong mục tiếp theo, người thực hiện sẽ tạo và kiểm tra Amazon DynamoDB table dùng để lưu các telemetry record từ AWS IoT Rule.
{{% /notice %}}

**Tiếp theo:** [5.6.2 Lưu Telemetry vào Amazon DynamoDB](../5.6.2-dynamodb/)