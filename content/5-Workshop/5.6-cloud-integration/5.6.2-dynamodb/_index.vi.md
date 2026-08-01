---
title: "Lưu Telemetry vào Amazon DynamoDB"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.6.2 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo bảng Amazon DynamoDB để lưu trữ dữ liệu telemetry do ESP32-S3 gửi lên. AWS IoT Rules Engine sẽ tự động ghi mỗi telemetry message thành một bản ghi trong bảng.
{{% /notice %}}

# Tổng quan

Amazon DynamoDB là dịch vụ cơ sở dữ liệu NoSQL được quản lý hoàn toàn bởi AWS.

Người dùng chỉ cần định nghĩa bảng và khóa chính, trong khi AWS chịu trách nhiệm quản lý hạ tầng, khả năng mở rộng, sao lưu và tính sẵn sàng của dữ liệu.

Trong project này, DynamoDB được sử dụng để lưu toàn bộ telemetry từ hệ thống Smart Home.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Tạo bảng DynamoDB.
- Cấu hình Partition Key và Sort Key.
- Lưu telemetry tự động.
- Kiểm tra dữ liệu đã được ghi.
- Hiểu cách AWS IoT Rule ghi dữ liệu vào DynamoDB.

---

# Thời gian thực hiện

**Khoảng 10 phút**

---

# Tại sao chọn DynamoDB?

Telemetry được gửi liên tục theo chu kỳ.

So với cơ sở dữ liệu quan hệ, DynamoDB không yêu cầu quản lý máy chủ và có khả năng mở rộng tự động.

Các ưu điểm:

- Dịch vụ được quản lý hoàn toàn.
- Tự động mở rộng.
- Độ trễ thấp.
- Khả năng sẵn sàng cao.
- Tích hợp trực tiếp với AWS IoT Rules Engine.

ESP32-S3 không cần giao tiếp trực tiếp với cơ sở dữ liệu.

---

# Bước 1 - Tạo bảng

Mở:

```text
Amazon DynamoDB
→ Tables
→ Create table
```

Cấu hình:

| Thiết lập | Giá trị |
|-----------|----------|
| Table name | SmartHomeTelemetry |
| Partition key | device_id (String) |
| Sort key | timestamp (Number) |

Capacity Mode:

```text
On-demand
```

---

# Bước 2 - Cấu hình IoT Rule

Mở:

```text
AWS IoT Core
→ Message routing
→ Rules
→ StoreSmartHomeTelemetry
```

Chọn bảng:

```text
SmartHomeTelemetry
```

AWS IoT Rule sẽ ghi các trường:

- device_id
- timestamp
- temperature
- humidity
- light
- door_open
- relay_on

IAM Role cần có quyền:

```text
dynamodb:PutItem
```

---

# Bước 3 - Publish Telemetry

Chạy firmware trên ESP32-S3.

Thiết bị sẽ gửi telemetry mỗi:

```text
10 giây
```

Ví dụ payload:

```json
{
  "device_id":"esp32-home-01",
  "timestamp":1784900000,
  "temperature":29.4,
  "humidity":71,
  "light":870,
  "door_open":false,
  "relay_on":false
}
```

AWS IoT Rule sẽ tự động ghi payload vào DynamoDB.

---

# Bước 4 - Kiểm tra dữ liệu

Mở:

```text
Amazon DynamoDB
→ Tables
→ SmartHomeTelemetry
→ Explore table items
```

Kết quả:

![](/fcj-workshop-template/images/workshop/5.6.2/dynamodb-items.png)

Mỗi telemetry message tương ứng với một Item trong bảng.

---

# Luồng dữ liệu

```text
ESP32-S3
      ↓
Telemetry JSON
      ↓
AWS IoT Core
      ↓
AWS IoT Rule
      ↓
Amazon DynamoDB
```

ESP32-S3 không giao tiếp trực tiếp với DynamoDB.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Telemetry được lưu thành công vào Amazon DynamoDB.
- Mỗi message tương ứng với một Item.
- AWS IoT Rules Engine tự động ghi dữ liệu.
- Có thể truy vấn lịch sử telemetry theo thời gian.

{{% notice tip %}}
Trong mục tiếp theo, người thực hiện sẽ tạo Amazon SNS Topic để gửi email cảnh báo khi cửa được mở.
{{% /notice %}}

**Tiếp theo:** [5.6.3 Tạo Amazon SNS](../5.6.3-sns/)