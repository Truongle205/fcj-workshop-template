---
title: "Tích hợp dịch vụ Cloud"
date: 2026-08-01
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---

{{% notice info %}}
Trong chương này, AWS IoT Rules Engine sẽ xử lý telemetry message từ ESP32-S3 và chuyển dữ liệu đến Amazon DynamoDB để lưu trữ, đồng thời gửi cảnh báo mở cửa thông qua Amazon SNS.
{{% /notice %}}

# Tích hợp dịch vụ Cloud

Đến thời điểm này, ESP32-S3 đã có thể kết nối bảo mật với AWS IoT Core, gửi telemetry, nhận command và cập nhật trạng thái relay.

Bước tiếp theo là tích hợp AWS IoT Core với các dịch vụ AWS khác.

Hệ thống sử dụng AWS IoT Rules Engine để đánh giá MQTT message và tự động kích hoạt các action phù hợp.

Hai luồng Cloud chính được triển khai:

```text
Telemetry message
      ↓
AWS IoT Rules Engine
      ├── Lưu dữ liệu vào Amazon DynamoDB
      └── Gửi cảnh báo đến Amazon SNS
```

Kiến trúc event-driven này không yêu cầu triển khai application server riêng.

---

# Mục tiêu

Sau khi hoàn thành chương này, người thực hiện sẽ có thể:

- Hiểu cách AWS IoT Rules Engine xử lý MQTT message.
- Tạo IoT SQL Rule.
- Lưu telemetry vào Amazon DynamoDB.
- Tạo Amazon SNS Topic.
- Tạo và xác nhận Email Subscription.
- Gửi cảnh báo mở cửa tự động.
- Cấu hình AWS IAM cho các IoT Rule Action.
- Kiểm tra toàn bộ luồng dữ liệu phía Cloud.

---

# Các dịch vụ sử dụng

| Dịch vụ AWS | Chức năng |
|---|---|
| AWS IoT Rules Engine | Đánh giá MQTT message và kích hoạt action. |
| Amazon DynamoDB | Lưu các bản ghi telemetry. |
| Amazon SNS | Gửi thông báo khi phát hiện cửa mở. |
| AWS IAM | Cấp quyền cho IoT Rule truy cập DynamoDB và SNS. |
| Amazon CloudWatch | Theo dõi metric và lỗi thực thi IoT Rule. |

---

# MQTT Telemetry Topic

Các rule trong chương này xử lý message được publish đến:

```text
smarthome/esp32-home-01/telemetry
```

Một telemetry message điển hình:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "dht_valid": true,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": false,
  "relay_on": false
}
```

---

# AWS IoT Rules Engine

AWS IoT Rules Engine sử dụng cú pháp tương tự SQL, được gọi là AWS IoT SQL, để đánh giá message.

Một rule thường gồm:

- Tên rule.
- Câu lệnh SQL.
- Một hoặc nhiều action.
- AWS IAM Role.
- Error action nếu cần.

Câu truy vấn telemetry cơ bản:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

Câu lệnh này lấy toàn bộ trường dữ liệu trong mỗi telemetry message.

Đối với cảnh báo mở cửa, thêm điều kiện:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

Chỉ những message có `door_open = true` mới kích hoạt alert action.

---

# Quy trình tích hợp Cloud

Chương này được chia thành bốn mục:

```text
5.6.1 Tạo AWS IoT Rule
        ↓
5.6.2 Lưu Telemetry vào Amazon DynamoDB
        ↓
5.6.3 Tạo Amazon SNS Topic
        ↓
5.6.4 Cấu hình Email Notification
```

Mỗi mục chỉ tập trung vào một nhiệm vụ phía Cloud.

---

# Luồng lưu dữ liệu Telemetry

```text
ESP32-S3
   ↓
MQTT telemetry topic
   ↓
AWS IoT Core
   ↓
Telemetry IoT Rule
   ↓
Amazon DynamoDB table
```

Bảng DynamoDB lưu các giá trị như:

- Device ID.
- Timestamp.
- Nhiệt độ.
- Độ ẩm.
- Cường độ ánh sáng.
- Trạng thái cửa.
- Trạng thái relay.

---

# Luồng cảnh báo mở cửa

```text
Cảm biến phát hiện cửa mở
              ↓
ESP32-S3 publish door_open = true
              ↓
AWS IoT Rules Engine đánh giá message
              ↓
Điều kiện Door Alert Rule được thỏa mãn
              ↓
Amazon SNS Topic
              ↓
Email Subscriber đã xác nhận
```

Cảnh báo được xử lý tự động mà ESP32-S3 không cần trực tiếp gửi email.

---

# Phân quyền AWS IAM

AWS IoT Rules Engine cần quyền để thực hiện action trên các dịch vụ AWS khác.

Một AWS IAM Role được liên kết với IoT Rule.

Role cấp các quyền:

```text
dynamodb:PutItem
sns:Publish
```

Trust relationship cần cho phép AWS IoT assume role.

Ví dụ principal:

```json
{
  "Principal": {
    "Service": "iot.amazonaws.com"
  }
}
```

{{% notice note %}}
AWS IAM không nằm trong luồng telemetry. IAM chỉ cung cấp quyền để AWS IoT Rules Engine gọi các action trên DynamoDB và SNS.
{{% /notice %}}

---

# Xử lý lỗi

AWS IoT Rule có thể cấu hình Error Action.

Error Action được kích hoạt khi action chính thất bại, ví dụ:

- DynamoDB table không tồn tại.
- IAM Role thiếu quyền.
- SNS Topic ARN không chính xác.
- Payload không có trường dữ liệu theo cấu hình.

Khi xử lý lỗi, cần kiểm tra:

- Rule đã được bật.
- MQTT topic khớp hoàn toàn.
- Câu lệnh SQL hợp lệ.
- IAM Role có đủ quyền.
- Target resource tồn tại trong cùng Region.
- Amazon CloudWatch không ghi nhận lỗi thực thi rule.

---

# Thời gian thực hiện

| Nội dung | Thời gian |
|---|---:|
| 5.6.1 Tạo AWS IoT Rule | 10 phút |
| 5.6.2 Cấu hình Amazon DynamoDB | 10 phút |
| 5.6.3 Cấu hình Amazon SNS | 5 phút |
| 5.6.4 Cấu hình Email Notification | 5 phút |

**Tổng thời gian thực hiện khoảng 30 phút.**

---

# Kết quả đạt được

Sau khi hoàn thành chương này:

- AWS IoT Rules Engine xử lý telemetry message.
- Telemetry được lưu trong Amazon DynamoDB.
- Sự kiện mở cửa được lọc bằng điều kiện IoT SQL.
- Amazon SNS nhận alert message.
- Email subscriber đã xác nhận nhận được cảnh báo.
- IAM Role cho phép các rule action hoạt động thành công.

{{% notice tip %}}
Trong mục tiếp theo, người thực hiện sẽ tạo các AWS IoT Rule để định tuyến telemetry đến các dịch vụ AWS cần thiết.
{{% /notice %}}

**Tiếp theo:** [5.6.1 Tạo AWS IoT Rule](5.6.1-iot-rules/)