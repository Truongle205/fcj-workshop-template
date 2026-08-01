---
title: "Cấu hình AWS IoT"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---

{{% notice info %}}
Trong chương này, người thực hiện sẽ cấu hình các tài nguyên AWS IoT cần thiết cho hệ thống Smart Home. Các bước bao gồm tạo AWS IoT Thing, sinh Device Certificate, cấu hình AWS IoT Policy và kiểm tra kết nối MQTT bằng AWS IoT MQTT Test Client.
{{% /notice %}}

# Cấu hình AWS IoT

AWS IoT Core là dịch vụ trung tâm chịu trách nhiệm quản lý thiết bị IoT và cung cấp kênh giao tiếp bảo mật giữa ESP32-S3 và AWS Cloud.

Trước khi ESP32-S3 có thể kết nối đến AWS IoT Core, cần tạo và cấu hình một số tài nguyên trên AWS. Các tài nguyên này giúp định danh thiết bị, xác thực kết nối, phân quyền MQTT và kiểm tra khả năng truyền nhận dữ liệu.

Chương này sẽ hướng dẫn từng bước cấu hình AWS IoT Core phục vụ cho hệ thống Smart Home.

---

# Mục tiêu

Sau khi hoàn thành chương này, người thực hiện sẽ có thể:

- Tạo AWS IoT Thing.
- Sinh X.509 Device Certificate.
- Gắn Certificate với Thing.
- Tạo AWS IoT Policy.
- Gắn Policy cho Certificate.
- Kiểm tra kết nối MQTT bằng AWS IoT MQTT Test Client.

---

# Các tài nguyên sẽ tạo

Trong chương này sẽ tạo các tài nguyên sau.

| Tài nguyên | Chức năng |
|------------|-----------|
| AWS IoT Thing | Đại diện cho thiết bị ESP32-S3 trên AWS. |
| X.509 Device Certificate | Xác thực thiết bị. |
| AWS IoT Policy | Cấp quyền giao tiếp MQTT. |
| Thing Attachment | Liên kết Certificate với Thing. |
| MQTT Test Client | Kiểm tra Publish/Subscribe. |

---

# Quy trình cấu hình

Toàn bộ quá trình cấu hình được thực hiện theo trình tự sau.

```text
Tạo Thing

↓

Sinh Device Certificate

↓

Kích hoạt Certificate

↓

Gắn Certificate vào Thing

↓

Tạo IoT Policy

↓

Gắn Policy

↓

Kiểm tra MQTT
```

Các bước trên sẽ được trình bày chi tiết trong các mục tiếp theo.

---

# Thời gian thực hiện

| Nội dung | Thời gian |
|-----------|----------:|
| 5.4.1 Tạo AWS IoT Thing | 5 phút |
| 5.4.2 Tạo Device Certificate | 5 phút |
| 5.4.3 Tạo và gắn IoT Policy | 5 phút |
| 5.4.4 MQTT Test Client | 5 phút |

**Tổng thời gian thực hiện khoảng 20 phút.**

---

{{% notice tip %}}
Trước khi bắt đầu, hãy đăng nhập AWS Management Console và đảm bảo đang sử dụng Region **US East (N. Virginia) – us-east-1**.
{{% /notice %}}

**Tiếp theo:** [5.4.1 Tạo AWS IoT Thing](5.4.1-create-thing/)