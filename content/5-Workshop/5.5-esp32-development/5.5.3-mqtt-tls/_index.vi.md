---
title: "Kết nối AWS IoT Core bằng MQTT over TLS"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.5.3 </b> "
---

{{% notice info %}}
Trong mục này, ESP32-S3 sẽ đồng bộ thời gian hệ thống và thiết lập kết nối MQTT bảo mật với AWS IoT Core thông qua cơ chế xác thực Mutual TLS.
{{% /notice %}}

# Tổng quan

AWS IoT Core yêu cầu thiết bị giao tiếp thông qua một kết nối được mã hóa và xác thực.

ESP32-S3 kết nối đến AWS IoT Endpoint bằng MQTT over TLS thông qua cổng TCP `8883`. Cơ chế Mutual TLS sử dụng ba thành phần:

- Amazon Root CA 1.
- X.509 Device Certificate.
- Device Private Key.

Trước khi bắt đầu TLS handshake, ESP32-S3 phải có thời gian hệ thống hợp lệ. Quá trình kiểm tra certificate có thể thất bại nếu đồng hồ của thiết bị không chính xác.

Firmware thực hiện tuần tự như sau:

```text
Kết nối Wi-Fi
        ↓
Đồng bộ thời gian bằng NTP
        ↓
Nạp Root CA, Certificate và Private Key
        ↓
Cấu hình MQTT Client
        ↓
Kết nối AWS IoT Core
        ↓
Subscribe command topic
```

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Đồng bộ thời gian ESP32-S3 bằng NTP.
- Cấu hình `WiFiClientSecure`.
- Nạp AWS Root CA và thông tin xác thực thiết bị.
- Cấu hình `PubSubClient`.
- Kết nối bảo mật với AWS IoT Core.
- Subscribe MQTT command topic.
- Xử lý việc kết nối lại MQTT.

---

# Thời gian thực hiện

**Khoảng 10–15 phút**

---

# Bước 1 - Cài đặt thư viện

Project sử dụng các thư viện sau:

```ini
lib_deps =
    knolleary/PubSubClient@2.8
    bblanchon/ArduinoJson
    adafruit/DHT sensor library
    adafruit/Adafruit Unified Sensor
```

`PubSubClient` cung cấp chức năng MQTT, trong khi `WiFiClientSecure` đảm nhiệm lớp truyền tải TLS.

Buffer MQTT được tăng kích thước vì telemetry và Device Shadow payload có thể lớn hơn kích thước packet mặc định.

---

# Bước 2 - Khai báo cấu hình AWS IoT

Lưu AWS IoT Endpoint, Client ID, MQTT topic và certificate trong file `include/secrets.h`.

Trong mã nguồn chia sẻ hoặc đưa lên GitHub, sử dụng placeholder:

```cpp
#pragma once

#define WIFI_SSID       "YOUR_WIFI_SSID"
#define WIFI_PASSWORD   "YOUR_WIFI_PASSWORD"

#define AWS_IOT_ENDPOINT \
    "YOUR_ENDPOINT-ats.iot.us-east-1.amazonaws.com"

constexpr uint16_t MQTT_PORT = 8883;

#define AWS_CLIENT_ID "esp32-home-01"

#define AWS_TOPIC_TELEMETRY \
    "smarthome/esp32-home-01/telemetry"

constexpr const char *AWS_TOPIC_COMMAND =
    "smarthome/esp32-home-01/command";
```

Các chứng chỉ được lưu dưới dạng chuỗi PEM:

```cpp
static const char AWS_ROOT_CA[] = R"EOF(
-----BEGIN CERTIFICATE-----
YOUR_AMAZON_ROOT_CA_1
-----END CERTIFICATE-----
)EOF";

static const char AWS_DEVICE_CERT[] = R"KEY(
-----BEGIN CERTIFICATE-----
YOUR_DEVICE_CERTIFICATE
-----END CERTIFICATE-----
)KEY";

static const char AWS_PRIVATE_KEY[] = R"KEY(
-----BEGIN PRIVATE KEY-----
YOUR_DEVICE_PRIVATE_KEY
-----END PRIVATE KEY-----
)KEY";
```

{{% notice warning %}}
Không công khai Wi-Fi password, Private Key hoặc Device Certificate thật. Cần đưa `include/secrets.h` và thư mục certificate vào `.gitignore`.
{{% /notice %}}

Ví dụ:

```gitignore
include/secrets.h
certificates/
*.pem
*.key
*.crt
```

---

# Bước 3 - Đồng bộ thời gian bằng NTP

Project tách chức năng đồng bộ thời gian vào `time_manager.cpp`.

Thiết bị lấy thời gian từ nhiều NTP server:

```cpp
bool syncTime()
{
    configTime(
        0,
        0,
        "pool.ntp.org",
        "time.google.com",
        "time.cloudflare.com"
    );

    Serial.print("Waiting for NTP");

    time_t now = time(nullptr);
    unsigned long started = millis();

    while (
        now < 1700000000 &&
        millis() - started < 30000
    )
    {
        delay(500);
        Serial.print(".");
        now = time(nullptr);
    }

    Serial.println();

    if (now < 1700000000)
    {
        Serial.println("NTP FAILED");
        return false;
    }

    struct tm timeInfo;

    if (!getLocalTime(&timeInfo))
    {
        Serial.println("getLocalTime FAILED");
        return false;
    }

    Serial.print("Current UTC: ");
    Serial.println(&timeInfo, "%Y-%m-%d %H:%M:%S");

    return true;
}
```

Firmware chờ tối đa 30 giây để nhận Unix timestamp hợp lệ.

Thời gian chính xác giúp ESP32-S3 kiểm tra thời hạn hiệu lực của server certificate do AWS IoT Core cung cấp.

---

# Bước 4 - Tạo TLS Client và MQTT Client

Project tạo hai đối tượng global trong `mqtt_manager.cpp`:

```cpp
WiFiClientSecure secureClient;
PubSubClient mqttClient(secureClient);
```

`secureClient` chịu trách nhiệm tạo kết nối TLS được mã hóa.

`mqttClient` sử dụng kết nối đó để gửi và nhận MQTT packet với AWS IoT Core.

Các đối tượng được khai báo global để duy trì trong toàn bộ vòng đời của chương trình.

---

# Bước 5 - Khởi tạo TLS và MQTT

Hàm `initMQTT()` cấu hình toàn bộ kết nối:

```cpp
void initMQTT()
{
    secureClient.setCACert(AWS_ROOT_CA);
    secureClient.setCertificate(AWS_DEVICE_CERT);
    secureClient.setPrivateKey(AWS_PRIVATE_KEY);

    secureClient.setHandshakeTimeout(15);
    secureClient.setTimeout(15000);

    mqttClient.setServer(
        AWS_IOT_ENDPOINT,
        MQTT_PORT
    );

    mqttClient.setCallback(mqttCallback);
    mqttClient.setSocketTimeout(15);
    mqttClient.setKeepAlive(60);
    mqttClient.setBufferSize(1024);
}
```

Ý nghĩa của các cấu hình:

| Cấu hình | Chức năng |
|---|---|
| `setCACert()` | Xác thực certificate của AWS IoT Core. |
| `setCertificate()` | Cung cấp Device Certificate của ESP32-S3. |
| `setPrivateKey()` | Chứng minh quyền sở hữu Device Certificate. |
| `setServer()` | Cấu hình AWS IoT Endpoint và MQTT port. |
| `setCallback()` | Đăng ký hàm xử lý message nhận được. |
| `setKeepAlive()` | Cấu hình chu kỳ MQTT keep-alive. |
| `setBufferSize()` | Tăng MQTT packet buffer lên 1024 byte. |

---

# Bước 6 - Kết nối AWS IoT Core

Logic kết nối được triển khai trong `connectMQTT()`.

Đoạn code rút gọn từ project:

```cpp
bool connectMQTT()
{
    if (mqttClient.connected())
    {
        return true;
    }

    Serial.print("Free heap before MQTT: ");
    Serial.println(ESP.getFreeHeap());

    Serial.print("Client ID: ");
    Serial.println(AWS_CLIENT_ID);

    unsigned long started = millis();

    bool connected =
        mqttClient.connect(AWS_CLIENT_ID);

    Serial.print("MQTT connection time: ");
    Serial.print(millis() - started);
    Serial.println(" ms");

    Serial.print("MQTT state: ");
    Serial.println(mqttClient.state());

    if (!connected)
    {
        Serial.println("MQTT connection failed");
        return false;
    }

    Serial.println("AWS IoT connected");

    return true;
}
```

MQTT Client ID phải phù hợp với resource được cấp quyền trong AWS IoT Policy nếu policy giới hạn theo client ARN.

Ví dụ:

```text
arn:aws:iot:us-east-1:ACCOUNT_ID:client/esp32-home-01
```

---

# Bước 7 - Subscribe Command Topic

Sau khi kết nối thành công, ESP32-S3 subscribe command topic:

```cpp
bool commandSubscribed =
    mqttClient.subscribe(AWS_TOPIC_COMMAND);

Serial.print("Command topic: ");
Serial.println(AWS_TOPIC_COMMAND);

Serial.print("Command subscribe: ");
Serial.println(
    commandSubscribed ? "OK" : "FAILED"
);
```

Command topic:

```text
smarthome/esp32-home-01/command
```

AWS IoT Policy phải cấp cả hai quyền:

```text
iot:Subscribe
iot:Receive
```

để thiết bị có thể đăng ký và nhận message.

---

# Bước 8 - Khởi tạo kết nối trong setup()

Project thực hiện trình tự khởi động trong `main.cpp`:

```cpp
connectWiFi();

while (WiFi.status() != WL_CONNECTED)
{
    Serial.println("Waiting for WiFi...");
    delay(500);
}

if (!syncTime())
{
    Serial.println("NTP synchronization failed");
    Serial.println("Setup stopped");
    return;
}

initMQTT();

Serial.println("Connecting MQTT...");
connectMQTT();
```

Thứ tự này cần được giữ chính xác:

1. Wi-Fi phải kết nối thành công.
2. Thời gian hệ thống phải được đồng bộ.
3. Thông tin TLS phải được nạp.
4. Sau đó mới thiết lập MQTT connection.

---

# Bước 9 - Duy trì kết nối MQTT

MQTT client cần gọi `loop()` thường xuyên.

Chương trình kiểm tra trạng thái và kết nối lại khi cần:

```cpp
if (!mqttClient.connected())
{
    if (now - lastReconnectAttempt >= 10000)
    {
        lastReconnectAttempt = now;

        Serial.println(
            "Attempting MQTT reconnect..."
        );

        secureClient.stop();
        delay(200);

        connectMQTT();
    }

    delay(10);
    return;
}

mqttClient.loop();
```

Chu kỳ reconnect được đặt là 10 giây để tránh ESP32-S3 thực hiện TLS handshake liên tục trong vòng lặp quá nhanh.

`secureClient.stop()` đóng socket TLS cũ trước khi tạo kết nối mới.

---

# Bước 10 - Nạp và kiểm tra chương trình

Biên dịch và nạp firmware:

```bash
pio run -t upload
```

Mở Serial Monitor:

```bash
pio device monitor
```

Kết quả mong đợi:

```text
ESP32 Smart Home starting
WiFi connected
Waiting for NTP...
Current UTC: 2026-07-24 09:54:43
Connecting MQTT...
AWS IoT connected
Command subscribe: OK
MQTT connected successfully
System ready
```

![ESP32-S3 kết nối MQTT over TLS](/fcj-workshop-template/images/workshop/5.5.3/mqtt-tls-connected.png)

---

# Trạng thái lỗi MQTT

Khi kết nối thất bại, `mqttClient.state()` giúp xác định nhóm lỗi.

| State | Ý nghĩa |
|---:|---|
| `0` | Đã kết nối |
| `-1` | Đã ngắt kết nối |
| `-2` | Lỗi mạng hoặc kết nối TCP |
| `-3` | Mất kết nối |
| `-4` | Hết thời gian chờ |
| `-5` | Kết nối bị từ chối hoặc lỗi liên quan TLS |

Khi xử lý lỗi, cần kiểm tra:

- Trạng thái Wi-Fi.
- Thời gian hệ thống.
- AWS IoT Endpoint.
- Trạng thái Certificate.
- Device Certificate và Private Key có khớp nhau hay không.
- AWS IoT Policy đã được attach chưa.
- MQTT Client ID.
- Cổng TCP `8883`.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- ESP32-S3 đồng bộ thời gian bằng NTP.
- Thông tin TLS được nạp thành công.
- Mutual TLS authentication hoạt động.
- ESP32-S3 kết nối thành công với AWS IoT Core.
- Command topic được subscribe thành công.
- Firmware có khả năng kết nối lại khi MQTT bị gián đoạn.

{{% notice tip %}}
Trong mục tiếp theo, ESP32-S3 sẽ sử dụng kết nối MQTT vừa thiết lập để gửi JSON telemetry lên AWS IoT Core.
{{% /notice %}}

**Tiếp theo:** [5.5.4 Gửi dữ liệu Telemetry](../5.5.4-telemetry/)