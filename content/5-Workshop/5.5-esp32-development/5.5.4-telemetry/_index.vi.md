---
title: "Gửi dữ liệu Telemetry"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.5.4 </b> "
---

{{% notice info %}}
Trong mục này, ESP32-S3 sẽ đọc dữ liệu cảm biến, tạo JSON telemetry payload và gửi message bảo mật lên AWS IoT Core.
{{% /notice %}}

# Tổng quan

Sau khi thiết lập thành công MQTT connection, ESP32-S3 có thể bắt đầu gửi dữ liệu Smart Home lên AWS IoT Core.

Firmware thu thập các thông tin:

- Nhiệt độ.
- Độ ẩm.
- Cường độ ánh sáng.
- Trạng thái cửa.
- Trạng thái relay.
- Timestamp của thiết bị.

Các giá trị được đóng gói thành JSON và publish đến topic:

```text
smarthome/esp32-home-01/telemetry
```

Luồng telemetry được triển khai qua ba thành phần chính:

```text
sensors.cpp
     ↓
telemetry.cpp
     ↓
mqtt_manager.cpp
     ↓
AWS IoT Core
```

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Khởi tạo các cảm biến Smart Home.
- Đọc nhiệt độ và độ ẩm từ DHT11.
- Đọc giá trị analog từ LDR.
- Đọc trạng thái cảm biến cửa.
- Lấy trạng thái relay hiện tại.
- Tạo telemetry bằng ArduinoJson.
- Publish JSON payload lên AWS IoT Core.
- Xử lý trường hợp cảm biến trả về dữ liệu không hợp lệ.

---

# Thời gian thực hiện

**Khoảng 10–15 phút**

---

# Bước 1 - Khai báo cấu trúc dữ liệu cảm biến

Project sử dụng cấu trúc `SensorData` để nhóm toàn bộ giá trị được trả về từ sensor module.

File:

```text
include/sensors.h
```

```cpp
#pragma once

struct SensorData
{
    float temperature;
    float humidity;
    int light;
    bool doorOpen;
    bool dhtValid;
};

void initSensors();
SensorData readSensors();
```

Cấu trúc này giúp truyền toàn bộ kết quả đo từ `sensors.cpp` sang `telemetry.cpp` một cách rõ ràng.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `temperature` | `float` | Nhiệt độ đọc từ DHT11. |
| `humidity` | `float` | Độ ẩm tương đối đọc từ DHT11. |
| `light` | `int` | Giá trị analog từ cảm biến LDR. |
| `doorOpen` | `bool` | Trạng thái hiện tại của cửa. |
| `dhtValid` | `bool` | Cho biết dữ liệu DHT11 có hợp lệ hay không. |

---

# Bước 2 - Cấu hình chân cảm biến

Các chân GPIO được khai báo trong `include/config.h`.

```cpp
constexpr uint8_t LDR_PIN = 1;
constexpr uint8_t DHT_PIN = 4;
constexpr uint8_t DHT_TYPE = 11;
constexpr uint8_t DOOR_SENSOR_PIN = 2;
constexpr uint8_t RELAY_PIN = 5;
```

Các giá trị trên có thể được thay đổi để phù hợp với bo mạch ESP32-S3 và cách đấu nối thực tế.

{{% notice warning %}}
Cần kiểm tra pinout của bo mạch ESP32-S3 trước khi đấu cảm biến. Một số GPIO có thể được sử dụng cho flash, PSRAM, USB hoặc chức năng đặc biệt khác.
{{% /notice %}}

---

# Bước 3 - Khởi tạo cảm biến

Sensor module được triển khai trong `src/sensors.cpp`.

```cpp
namespace
{
DHT dht(DHT_PIN, DHT11);
}

void initSensors()
{
    dht.begin();

    pinMode(
        DOOR_SENSOR_PIN,
        INPUT_PULLUP
    );

    Serial.println("DHT11 initialized");

    analogReadResolution(12);
}
```

Quá trình khởi tạo gồm:

| Code | Chức năng |
|---|---|
| `dht.begin()` | Khởi động giao tiếp với DHT11. |
| `INPUT_PULLUP` | Bật điện trở kéo lên nội bộ cho cảm biến cửa. |
| `analogReadResolution(12)` | Cấu hình ADC ở độ phân giải 12 bit. |

Với ADC 12 bit, giá trị LDR thường nằm trong khoảng:

```text
0 đến 4095
```

---

# Bước 4 - Đọc dữ liệu cảm biến

Hàm `readSensors()` thu thập toàn bộ dữ liệu.

```cpp
SensorData readSensors()
{
    SensorData data {};

    data.temperature =
        dht.readTemperature();

    data.humidity =
        dht.readHumidity();

    data.light =
        analogRead(LDR_PIN);

    data.doorOpen =
        digitalRead(DOOR_SENSOR_PIN) == HIGH;

    data.dhtValid =
        !isnan(data.temperature) &&
        !isnan(data.humidity);

    return data;
}
```

Hàm thực hiện bốn phép đọc chính:

1. Nhiệt độ từ DHT11.
2. Độ ẩm từ DHT11.
3. Ánh sáng từ ADC.
4. Trạng thái cửa từ digital input.

---

# Bước 5 - Kiểm tra dữ liệu DHT11

Thư viện DHT trả về `NaN` khi quá trình đọc thất bại.

Project kiểm tra cả hai giá trị:

```cpp
data.dhtValid =
    !isnan(data.temperature) &&
    !isnan(data.humidity);
```

Điều này giúp ngăn giá trị không hợp lệ bị xem như dữ liệu nhiệt độ hoặc độ ẩm bình thường.

Các giá trị cảm biến cũng được in ra Serial Monitor:

```cpp
Serial.print("Temperature: ");
Serial.println(data.temperature);

Serial.print("Humidity: ");
Serial.println(data.humidity);

Serial.print("Light: ");
Serial.println(data.light);

Serial.print("Door: ");
Serial.println(
    data.doorOpen ? "OPEN" : "CLOSED"
);
```

Ví dụ:

```text
Temperature: 29.40
Humidity: 71.00
Light: 870
Door: CLOSED
```

---

# Bước 6 - Tạo JSON Telemetry

Logic telemetry được triển khai trong:

```text
src/telemetry.cpp
```

Hàm trước tiên gọi sensor module:

```cpp
void publishTelemetry()
{
    SensorData sensorData =
        readSensors();

    JsonDocument document;
```

Sau đó thêm định danh thiết bị và Unix timestamp:

```cpp
document["device_id"] =
    AWS_THING_NAME;

document["timestamp"] =
    static_cast<int64_t>(time(nullptr));
```

Timestamp được tạo từ thời gian hệ thống đã đồng bộ bằng NTP.

---

# Bước 7 - Thêm dữ liệu cảm biến và trạng thái thiết bị

Firmware ghi trạng thái hợp lệ của DHT11:

```cpp
document["dht_valid"] =
    sensorData.dhtValid;
```

Khi dữ liệu hợp lệ:

```cpp
if (sensorData.dhtValid)
{
    document["temperature"] =
        sensorData.temperature;

    document["humidity"] =
        sensorData.humidity;
}
```

Khi DHT11 đọc thất bại, payload sử dụng giá trị JSON `null`:

```cpp
else
{
    Serial.println(
        "Telemetry warning: invalid DHT11 data"
    );

    document["temperature"] = nullptr;
    document["humidity"] = nullptr;
}
```

Các giá trị còn lại được thêm độc lập:

```cpp
document["light"] =
    sensorData.light;

document["door_open"] =
    sensorData.doorOpen;

document["relay_on"] =
    isRelayOn();
```

Việc dùng `null` giúp tránh lưu dữ liệu DHT11 lỗi thành một phép đo thật.

---

# Bước 8 - Chuyển JSON thành payload

JSON document được serialize vào một buffer cố định:

```cpp
char payload[320];

size_t length = serializeJson(
    document,
    payload,
    sizeof(payload)
);
```

Chương trình kiểm tra kết quả:

```cpp
if (length == 0)
{
    Serial.println(
        "JSON serialization failed"
    );

    return;
}
```

Buffer cố định giúp hạn chế việc cấp phát chuỗi động trong mỗi chu kỳ telemetry.

---

# Bước 9 - Publish telemetry

Payload được in ra trước khi gửi:

```cpp
Serial.print(
    "Publishing telemetry: "
);

Serial.println(payload);
```

Sau đó sử dụng MQTT manager:

```cpp
if (!publishMessage(
        AWS_TOPIC_TELEMETRY,
        payload
    ))
{
    Serial.println(
        "Telemetry publish failed"
    );
}
```

Hàm `publishMessage()` kiểm tra trạng thái MQTT trước khi publish:

```cpp
bool publishMessage(
    const char *topic,
    const char *payload
)
{
    if (!mqttClient.connected())
    {
        Serial.println(
            "MQTT is not connected"
        );

        return false;
    }

    bool success =
        mqttClient.publish(topic, payload);

    return success;
}
```

Việc tách hàm publish giúp telemetry module không phụ thuộc trực tiếp vào cách MQTT client được cấu hình.

---

# Bước 10 - Gửi dữ liệu theo chu kỳ

Project publish telemetry từ `main.cpp`.

```cpp
if (
    now - lastTelemetryTime >= 10000
)
{
    lastTelemetryTime = now;

    Serial.println(
        "Publishing telemetry..."
    );

    publishTelemetry();
}
```

Chu kỳ hiện tại là:

```text
10 giây
```

Sử dụng `millis()` giúp chương trình không block MQTT loop trong thời gian dài.

---

# Ví dụ Telemetry Payload

Khi cảm biến hoạt động bình thường:

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

Khi DHT11 không hợp lệ:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "dht_valid": false,
  "temperature": null,
  "humidity": null,
  "light": 870,
  "door_open": false,
  "relay_on": false
}
```

---

# Bước 11 - Nạp và theo dõi chương trình

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
Publishing telemetry...
Temperature: 29.40
Humidity: 71.00
Light: 870
Door: CLOSED
Publishing telemetry: {"device_id":"esp32-home-01",...}
Published: {"device_id":"esp32-home-01",...}
```

![ESP32-S3 gửi Telemetry](/images/workshop/5.5.4/telemetry-published.png)

---

# Bước 12 - Kiểm tra trên AWS IoT Core

Mở:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Subscribe topic:

```text
smarthome/esp32-home-01/telemetry
```

Telemetry do ESP32-S3 publish sẽ xuất hiện trên MQTT Test Client.

![AWS IoT Core nhận Telemetry](/images/workshop/5.5.4/telemetry-received.png)

Luồng hoàn chỉnh:

```text
Cảm biến
   ↓
ESP32-S3
   ↓
JSON serialization
   ↓
MQTT over TLS
   ↓
AWS IoT Core
```

---

# Xử lý lỗi

## DHT11 trả về dữ liệu lỗi

Nguyên nhân có thể gồm:

- Chưa kết nối cảm biến.
- Sai GPIO.
- Data line thiếu pull-up.
- Đọc cảm biến quá nhanh.
- Nguồn cấp không ổn định.

Telemetry vẫn gửi các trường còn lại, trong khi nhiệt độ và độ ẩm được đặt thành `null`.

---

## Không thấy telemetry trên AWS

Kiểm tra:

- MQTT vẫn đang connected.
- Topic telemetry trùng khớp hoàn toàn.
- AWS IoT Policy có quyền `iot:Publish`.
- MQTT Test Client subscribe đúng topic.
- Payload không vượt quá MQTT buffer.

---

## Giá trị ánh sáng không thay đổi

Kiểm tra:

- LDR được nối vào GPIO hỗ trợ ADC.
- Mạch chia áp được đấu đúng.
- GPIO không bị sử dụng bởi chức năng khác của bo mạch.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- ESP32-S3 đọc được dữ liệu cảm biến.
- JSON telemetry được tạo thành công.
- Dữ liệu DHT11 lỗi được xử lý an toàn.
- Telemetry được gửi theo chu kỳ.
- AWS IoT Core nhận được message.
- Trạng thái cửa và relay được đưa vào payload.

{{% notice tip %}}
Trong mục tiếp theo, hệ thống sẽ bổ sung giao tiếp hai chiều để AWS IoT Core gửi command xuống ESP32-S3 và điều khiển relay từ xa.
{{% /notice %}}

**Tiếp theo:** [5.5.5 Điều khiển Relay từ xa](../5.5.5-relay-control/)