---
title: "Tạo và gắn AWS IoT Policy"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo AWS IoT Policy và gắn Policy vào Device Certificate. AWS IoT Policy quy định các thao tác MQTT mà ESP32-S3 được phép thực hiện sau khi đã xác thực thành công.
{{% /notice %}}

# Tổng quan

Trong AWS IoT Core, xác thực (Authentication) và phân quyền (Authorization) là hai cơ chế hoàn toàn độc lập.

X.509 Device Certificate xác minh danh tính của thiết bị, trong khi AWS IoT Policy quy định các quyền truy cập của thiết bị sau khi đã được xác thực.

Nếu Device Certificate không được gắn với một AWS IoT Policy hợp lệ, mọi thao tác MQTT sẽ bị AWS IoT Core từ chối.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Tạo AWS IoT Policy.
- Cấp quyền cho thiết bị giao tiếp MQTT.
- Gắn Policy vào Device Certificate.
- Kiểm tra việc liên kết Policy.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Tạo AWS IoT Policy

Truy cập:

```text
AWS IoT Core

↓

Security

↓

Policies
```

Tạo một Policy mới.

Ví dụ đặt tên:

```text
esp32-home-policy
```

---

# Bước 2 - Cấu hình quyền truy cập

Cấp các quyền MQTT sau.

| Quyền | Mục đích |
|--------|----------|
| iot:Connect | Kết nối đến AWS IoT Core |
| iot:Publish | Publish MQTT message |
| iot:Subscribe | Subscribe MQTT topic |
| iot:Receive | Nhận MQTT message |

Trong workshop này, Resource được cấu hình là:

```text
*

Effect

Allow
```

{{% notice note %}}
Để đơn giản hóa quá trình thực hành, workshop sử dụng ký tự đại diện (*). Trong môi trường thực tế, nên giới hạn quyền truy cập theo nguyên tắc **Least Privilege** nhằm tăng cường bảo mật.
{{% /notice %}}

---

# Bước 3 - Gắn Policy

Quay trở lại Device Certificate đã tạo ở mục trước.

Chọn **Attach Policy**.

![](/images/workshop/5.4.3/attach-iot-policy.png)

Chọn Policy vừa tạo và xác nhận.

---

# Kiểm tra kết quả

Mở trang thông tin của Device Certificate.

Xác nhận:

- Policy đã được liên kết.
- Device Certificate vẫn ở trạng thái **Active**.
- Không xuất hiện cảnh báo cấu hình.

---

# Giải thích

Device Certificate trả lời câu hỏi:

> Thiết bị là ai?

AWS IoT Policy trả lời câu hỏi:

> Thiết bị được phép làm gì?

Hai cơ chế này phải được cấu hình đầy đủ thì ESP32-S3 mới có thể thiết lập kết nối MQTT với AWS IoT Core.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- AWS IoT Policy đã được tạo.
- Thiết bị được cấp quyền MQTT.
- Policy đã được gắn vào Device Certificate.
- ESP32-S3 đã sẵn sàng kết nối AWS IoT Core.

{{% notice tip %}}
Ở mục tiếp theo, người thực hiện sẽ sử dụng **AWS IoT MQTT Test Client** để kiểm tra Publish và Subscribe trước khi lập trình ESP32-S3.
{{% /notice %}}

**Tiếp theo:** [5.4.4 Kiểm tra MQTT](../5.4.4-mqtt-test-client/)