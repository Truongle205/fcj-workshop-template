---
title: "Phát triển ứng dụng ESP32"
date: 2026-08-01
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

{{% notice info %}}
Trong chương này, người thực hiện sẽ xây dựng firmware cho ESP32-S3. Thiết bị sẽ được cấu hình để kết nối Wi-Fi, xác thực với AWS IoT Core thông qua MQTT over TLS, gửi dữ liệu telemetry và nhận lệnh điều khiển từ AWS Cloud.
{{% /notice %}}

# Phát triển ứng dụng ESP32

Sau khi hoàn thành việc cấu hình AWS IoT Core, bước tiếp theo là xây dựng chương trình chạy trên bo mạch ESP32-S3.

Firmware đóng vai trò là cầu nối giữa hệ thống Smart Home và nền tảng AWS Cloud. Chương trình chịu trách nhiệm đọc dữ liệu cảm biến, tạo telemetry theo định dạng JSON, gửi dữ liệu lên AWS IoT Core và nhận các lệnh điều khiển relay từ Cloud.

Trong chương này, firmware sẽ được phát triển theo từng bước nhỏ nhằm đảm bảo mỗi chức năng đều được kiểm tra trước khi tích hợp vào hệ thống hoàn chỉnh.

---

# Mục tiêu

Sau khi hoàn thành chương này, người thực hiện sẽ có thể:

- Tạo dự án PlatformIO.
- Cấu hình môi trường phát triển ESP32-S3.
- Kết nối Wi-Fi.
- Thiết lập MQTT over TLS với AWS IoT Core.
- Publish dữ liệu telemetry.
- Subscribe MQTT command.
- Điều khiển relay từ xa.

---

# Môi trường phát triển

Workshop sử dụng các công cụ và thư viện sau.

| Công cụ | Mục đích |
|----------|----------|
| Visual Studio Code | Soạn thảo mã nguồn |
| PlatformIO IDE | Phát triển ESP32 |
| Arduino Framework | Framework lập trình |
| ArduinoJson | Xử lý dữ liệu JSON |
| PubSubClient | MQTT Client |
| WiFiClientSecure | Kết nối TLS |

---

# Cấu trúc dự án

Mã nguồn được tổ chức theo cấu trúc PlatformIO như sau.

```text
awsprj/
├── include/
├── lib/
├── src/
│   └── main.cpp
├── certificates/
│   ├── AmazonRootCA1.pem
│   ├── device.pem.crt
│   └── private.pem.key
├── platformio.ini
└── README.md
```

Trong đó, thư mục `certificates/` chứa các tệp chứng chỉ đã tải ở chương trước và sẽ được sử dụng để thiết lập kết nối MQTT bảo mật.

---

# Các chức năng của firmware

Firmware được chia thành các module độc lập.

| Module | Chức năng |
|---------|-----------|
| Wi-Fi Manager | Kết nối mạng Wi-Fi |
| NTP Client | Đồng bộ thời gian |
| MQTT Client | Kết nối AWS IoT Core |
| Telemetry Publisher | Gửi dữ liệu cảm biến |
| Command Subscriber | Nhận lệnh MQTT |
| Relay Controller | Điều khiển Relay |

Việc tách firmware thành các module giúp chương trình dễ mở rộng, dễ bảo trì và thuận tiện trong quá trình kiểm thử.

---

# Quy trình phát triển

Toàn bộ quá trình lập trình được chia thành các bước sau.

```text
Tạo Project PlatformIO

↓

Kết nối Wi-Fi

↓

MQTT over TLS

↓

Publish Telemetry

↓

Điều khiển Relay
```

Mỗi bước sẽ được triển khai và kiểm thử riêng trước khi chuyển sang bước tiếp theo.

---

# Thời gian thực hiện

| Nội dung | Thời gian |
|-----------|----------:|
| 5.5.1 Tạo Project PlatformIO | 5 phút |
| 5.5.2 Kết nối Wi-Fi | 5 phút |
| 5.5.3 MQTT over TLS | 10 phút |
| 5.5.4 Publish Telemetry | 10 phút |
| 5.5.5 Điều khiển Relay | 10 phút |

**Tổng thời gian thực hiện khoảng 40 phút.**

---

{{% notice tip %}}
Sau khi hoàn thành chương này, ESP32-S3 sẽ có khả năng kết nối bảo mật với AWS IoT Core, gửi dữ liệu telemetry và nhận lệnh điều khiển relay thông qua MQTT.
{{% /notice %}}

**Tiếp theo:** [5.5.1 Tạo Project PlatformIO](5.5.1-platformio/)