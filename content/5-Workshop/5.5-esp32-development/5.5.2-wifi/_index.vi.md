---
title: "Kết nối Wi-Fi"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

{{% notice info %}}
Trong mục này, ESP32-S3 sẽ được cấu hình để kết nối với mạng Wi-Fi. Đây là bước bắt buộc trước khi đồng bộ thời gian bằng NTP và thiết lập kết nối MQTT bảo mật với AWS IoT Core.
{{% /notice %}}

# Tổng quan

Để giao tiếp với AWS Cloud, ESP32-S3 cần có kết nối Internet thông qua mạng Wi-Fi.

Firmware sẽ khởi tạo giao diện mạng, kết nối đến Access Point đã cấu hình và nhận địa chỉ IP thông qua giao thức DHCP.

Sau khi kết nối thành công, thiết bị sẽ sẵn sàng cho các bước đồng bộ thời gian và kết nối đến AWS IoT Core.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Cấu hình tên mạng Wi-Fi (SSID).
- Cấu hình mật khẩu Wi-Fi.
- Kết nối ESP32-S3 với mạng không dây.
- Kiểm tra địa chỉ IP được cấp.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Cấu hình thông tin Wi-Fi

Mở mã nguồn và khai báo thông tin mạng Wi-Fi.

```cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
```

Thay thế bằng tên mạng và mật khẩu Wi-Fi của bạn.

{{% notice warning %}}
Không nên đưa thông tin Wi-Fi lên các kho mã nguồn công khai như GitHub.
{{% /notice %}}

---

# Bước 2 - Kết nối Wi-Fi

Khởi tạo giao diện Wi-Fi và bắt đầu kết nối đến Access Point.

Trong thời gian kết nối, trạng thái sẽ được hiển thị trên Serial Monitor.

![](/fcj-workshop-template/images/workshop/5.5.2/wifi-connecting.png)

---

# Bước 3 - Kiểm tra kết nối

Sau khi kết nối thành công, Serial Monitor sẽ hiển thị địa chỉ IP của thiết bị.

Ví dụ:

```text
WiFi Connected
IP: 192.168.1.100
```

![](/fcj-workshop-template/images/workshop/5.5.2/wifi-connected.png)

Địa chỉ IP này cho thấy ESP32-S3 đã kết nối thành công với mạng và có thể truy cập Internet.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- ESP32-S3 kết nối thành công với Wi-Fi.
- Thiết bị được cấp địa chỉ IP hợp lệ.
- Sẵn sàng thực hiện các bước tiếp theo.

{{% notice tip %}}
Ở mục tiếp theo, ESP32-S3 sẽ đồng bộ thời gian bằng NTP và thiết lập kết nối MQTT over TLS với AWS IoT Core.
{{% /notice %}}

**Tiếp theo:** [5.5.3 Kết nối MQTT over TLS](../5.5.3-mqtt-tls/)