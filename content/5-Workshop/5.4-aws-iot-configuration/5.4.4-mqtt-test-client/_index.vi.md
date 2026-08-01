---
title: "Kiểm tra kết nối MQTT"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ sử dụng AWS IoT MQTT Test Client để kiểm tra khả năng Publish và Subscribe của AWS IoT Core trước khi kết nối ESP32-S3.
{{% /notice %}}

# Tổng quan

Trước khi nạp chương trình cho ESP32-S3, nên kiểm tra hoạt động của AWS IoT Core bằng công cụ **MQTT Test Client**.

Đây là công cụ được tích hợp sẵn trong AWS IoT Core, cho phép gửi và nhận MQTT message trực tiếp trên AWS Management Console mà không cần thiết bị phần cứng.

Việc kiểm tra trước giúp xác nhận rằng topic MQTT đã được cấu hình đúng và AWS IoT Core có thể xử lý các bản tin một cách chính xác.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Mở AWS IoT MQTT Test Client.
- Subscribe một MQTT Topic.
- Publish một MQTT Message.
- Kiểm tra việc truyền nhận dữ liệu.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Mở MQTT Test Client

Trong AWS IoT Core, chọn:

```text
Test

↓

MQTT test client
```

![](/fcj-workshop-template/images/workshop/5.4.4/mqtt-test-client.png)

---

# Bước 2 - Subscribe Topic

Thực hiện Subscribe tới topic telemetry của hệ thống.

```text
smarthome/esp32-home-01/telemetry
```

Nhấn **Subscribe**.

Topic sẽ xuất hiện trong danh sách theo dõi.

---

# Bước 3 - Publish Message

Chuyển sang mục **Publish to a topic**.

Sử dụng cùng topic:

```text
smarthome/esp32-home-01/telemetry
```

Nhập payload mẫu:

```json
{
  "temperature": 28,
  "humidity": 65,
  "light": 420
}
```

Sau đó nhấn **Publish**.

---

# Bước 4 - Kiểm tra kết quả

Nếu cấu hình đúng, bản tin vừa Publish sẽ xuất hiện ngay tại cửa sổ Subscribe.

Điều này xác nhận:

- Topic MQTT đã được cấu hình đúng.
- AWS IoT Core hoạt động bình thường.
- MQTT Message được truyền nhận thành công.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- MQTT Test Client hoạt động bình thường.
- Subscribe thành công tới topic telemetry.
- Publish thành công MQTT Message.
- Message được nhận ngay trên AWS IoT Core.

{{% notice tip %}}
MQTT Test Client là công cụ hữu ích để kiểm tra MQTT Topic và Payload trước khi triển khai chương trình lên ESP32-S3.
{{% /notice %}}

**Tiếp theo:** [5.5 Phát triển ứng dụng ESP32](../../5.5-esp32-development/)