---
title: "Cấu hình Email Notification"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.6.4 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo và xác nhận Email Subscription cho Amazon SNS Topic. Sau khi subscription được xác nhận, sự kiện mở cửa từ hệ thống Smart Home sẽ được gửi đến địa chỉ email đã đăng ký.
{{% /notice %}}

# Tổng quan

Amazon SNS Topic được tạo ở mục trước đã có thể nhận notification từ AWS IoT Rules Engine.

Tuy nhiên, Amazon SNS chưa thể gửi message đến người dùng nếu chưa có subscription được tạo và xác nhận.

Trong workshop này, một địa chỉ email được đăng ký làm notification endpoint.

Luồng thông báo hoàn chỉnh:

```text
Cảm biến phát hiện cửa mở
                ↓
ESP32-S3 publish telemetry
                ↓
AWS IoT Door Alert Rule khớp điều kiện
                ↓
Amazon SNS nhận message
                ↓
Email Subscription đã xác nhận
                ↓
Người dùng nhận cảnh báo
```

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Tạo Amazon SNS Email Subscription.
- Xác nhận subscription qua email.
- Kiểm tra trạng thái subscription.
- Publish test notification.
- Kích hoạt email cảnh báo thông qua AWS IoT Rule.
- Xử lý các lỗi phổ biến khi SNS gửi email.

---

# Thời gian thực hiện

**Khoảng 5–10 phút**

---

# Bước 1 - Mở SNS Topic

Mở AWS Management Console và truy cập:

```text
Amazon SNS
→ Topics
→ SmartHomeDoorAlert
```

Trang chi tiết topic hiển thị:

- Topic ARN.
- Topic type.
- Access policy.
- Danh sách subscription.
- Chức năng publish message.

---

# Bước 2 - Tạo Email Subscription

Tại trang chi tiết topic, chọn:

```text
Create subscription
```

Cấu hình:

| Thiết lập | Giá trị |
|---|---|
| Topic ARN | ARN của `SmartHomeDoorAlert` |
| Protocol | Email |
| Endpoint | Địa chỉ email người nhận |

Ví dụ:

```text
recipient@example.com
```

Chọn:

```text
Create subscription
```

Sau khi tạo, trạng thái ban đầu là:

```text
Pending confirmation
```

{{% notice warning %}}
Cần sử dụng địa chỉ email có thể truy cập ngay. Amazon SNS sẽ không gửi notification nếu subscription chưa được xác nhận.
{{% /notice %}}

---

# Bước 3 - Xác nhận Subscription

Mở hộp thư của địa chỉ email đã cấu hình.

Tìm email từ Amazon SNS với tiêu đề tương tự:

```text
AWS Notification - Subscription Confirmation
```

Mở email và chọn:

```text
Confirm subscription
```

Trình duyệt sẽ hiển thị trang xác nhận subscription đã được kích hoạt.

---

# Bước 4 - Kiểm tra trạng thái Subscription

Quay lại:

```text
Amazon SNS
→ Topics
→ SmartHomeDoorAlert
→ Subscriptions
```

Trạng thái phải là:

```text
Confirmed
```

Danh sách subscription cần có:

| Trường | Giá trị mong đợi |
|---|---|
| Protocol | Email |
| Endpoint | Email đã đăng ký |
| Status | Confirmed |

{{% notice note %}}
Nếu trạng thái vẫn là `Pending confirmation`, Amazon SNS sẽ không gửi alert đến email endpoint.
{{% /notice %}}

---

# Bước 5 - Publish Test Message trực tiếp

Trước khi kiểm tra toàn bộ luồng IoT, gửi test message trực tiếp từ SNS.

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
The Smart Home door notification system is working correctly.
```

Chọn:

```text
Publish message
```

Kiểm tra hộp thư của subscriber.

Email kiểm thử sẽ được gửi sau một khoảng thời gian ngắn.

---

# Bước 6 - Kích hoạt bằng AWS IoT MQTT Test Client

Mở:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Publish message đến:

```text
smarthome/esp32-home-01/telemetry
```

Payload:

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

Message thỏa điều kiện Door Alert Rule:

```sql
WHERE door_open = true
```

AWS IoT Rules Engine publish message đến Amazon SNS.

Amazon SNS sau đó gửi message đến Email Subscription đã xác nhận.

---

# Bước 7 - Kích hoạt từ ESP32-S3

Kết nối cảm biến cửa với ESP32-S3 và chạy firmware.

Khi cửa mở, telemetry chứa:

```json
{
  "door_open": true
}
```

Luồng hoàn chỉnh:

```text
Cửa mở
    ↓
ESP32-S3 phát hiện trạng thái HIGH
    ↓
Telemetry được publish
    ↓
AWS IoT Door Alert Rule khớp điều kiện
    ↓
Amazon SNS publish notification
    ↓
Email được gửi
```

---

# Bước 8 - Kiểm tra Email Notification

Mở email nhận được từ Amazon SNS.

Message có thể chứa telemetry payload do ESP32-S3 gửi.

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

![Email cảnh báo mở cửa](/images/workshop/5.6.4/sns-email-notification.png)

Email này xác nhận toàn bộ luồng cảnh báo Smart Home đã hoạt động thành công.

{{% notice warning %}}
Trước khi đưa ảnh vào báo cáo, cần che địa chỉ email, AWS Account ID, Message ID và các thông tin nhạy cảm khác.
{{% /notice %}}

---

# Hạn chế cảnh báo lặp lại

ESP32-S3 publish telemetry theo chu kỳ.

Nếu cửa vẫn đang mở, mỗi message có thể tiếp tục chứa:

```json
{
  "door_open": true
}
```

Khi đó Door Alert Rule có thể gửi nhiều email liên tiếp.

Trong hệ thống production, nên chỉ gửi cảnh báo khi trạng thái chuyển:

```text
CLOSED → OPEN
```

Các hướng cải tiến:

- Phát hiện state transition trong firmware ESP32-S3.
- Tạo một MQTT topic riêng cho door event.
- Xử lý sự kiện bằng AWS Lambda.
- So sánh trạng thái bằng Device Shadow.
- Thiết lập khoảng thời gian chờ sau mỗi cảnh báo.

Trong workshop hiện tại, cảnh báo lặp lại vẫn được chấp nhận vì mục tiêu chính là minh họa luồng Amazon SNS.

---

# Các yếu tố ảnh hưởng đến Email Delivery

Email có thể bị ảnh hưởng bởi:

- Spam hoặc Junk filter.
- Chính sách bảo mật email của tổ chức.
- Địa chỉ email nhập sai.
- Subscription chưa được xác nhận.
- Nhiều subscription trùng nhau.
- Độ trễ từ email provider.

Khi kiểm tra, xem các thư mục:

```text
Inbox
Spam
Junk
Promotions
Quarantine
```

---

# Giám sát việc gửi Notification

Amazon SNS cung cấp metric thông qua Amazon CloudWatch.

| Metric | Ý nghĩa |
|---|---|
| NumberOfMessagesPublished | Số message được publish vào topic. |
| NumberOfNotificationsDelivered | Số notification gửi thành công. |
| NumberOfNotificationsFailed | Số notification thất bại. |
| NumberOfNotificationsFilteredOut | Số message bị loại bởi filter policy. |

Nếu rule được kích hoạt nhưng không nhận email, so sánh số message đã publish và số notification đã delivery.

---

# Xử lý lỗi

## Subscription vẫn ở trạng thái Pending

Kiểm tra:

- Đã nhận email confirmation chưa.
- Đã mở confirmation link chưa.
- Email endpoint có chính xác không.
- Email confirmation có nằm trong Spam hoặc Junk không.

Nếu cần, xóa subscription pending và tạo lại.

---

## Test trực tiếp từ SNS thành công nhưng IoT Alert không hoạt động

Kiểm tra:

- Door Alert Rule đang Enabled.
- SQL condition chính xác.
- IoT Rule Action trỏ đúng SNS Topic.
- IAM Role có `sns:Publish`.
- Payload dùng Boolean `true`, không phải chuỗi `"true"`.

Đúng:

```json
{
  "door_open": true
}
```

Sai:

```json
{
  "door_open": "true"
}
```

---

## IoT Rule kích hoạt liên tục

Nguyên nhân là telemetry tiếp tục báo cửa đang mở.

Có thể bổ sung state-change detection hoặc cooldown mechanism.

---

## Không nhận được Confirmation Email

Nguyên nhân có thể:

- Sai địa chỉ email.
- Email provider chặn message tự động.
- Hệ thống email của tổ chức chặn subscription.
- Subscription được tạo trong nhầm SNS Topic.

Có thể thử một email khác trong quá trình kiểm thử.

---

# Bảo mật và quyền riêng tư

Cần tuân thủ:

- Không công khai địa chỉ email subscriber.
- Giới hạn `sns:Publish` theo đúng Topic ARN.
- Xóa các subscription không còn sử dụng.
- Không đưa thông tin bí mật vào SNS message.
- Kiểm tra SNS Topic Access Policy.
- Che thông tin nhạy cảm trong screenshot.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Email Subscription đã được tạo.
- Subscription có trạng thái `Confirmed`.
- Test message từ SNS được gửi thành công.
- Door-open telemetry kích hoạt AWS IoT Rule.
- Amazon SNS gửi cảnh báo đến email subscriber.
- Toàn bộ notification workflow hoạt động hoàn chỉnh.

{{% notice tip %}}
Quá trình tích hợp Cloud đã hoàn tất. Trong chương tiếp theo, toàn bộ hệ thống sẽ được kiểm thử theo luồng end-to-end và các tình huống lỗi.
{{% /notice %}}

**Tiếp theo:** [5.7 Kiểm thử hệ thống](../../5.7-testing/)