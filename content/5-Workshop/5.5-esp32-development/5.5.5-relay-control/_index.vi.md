---
title: "Điều khiển Relay từ xa"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5.5 </b> "
---

{{% notice info %}}
Trong mục này, ESP32-S3 sẽ nhận MQTT command từ AWS IoT Core và điều khiển relay từ xa. Firmware đồng thời cập nhật trạng thái relay thực tế lên AWS IoT Device Shadow.
{{% /notice %}}

# Tổng quan

Ở mục trước, ESP32-S3 đã có khả năng gửi telemetry một chiều lên AWS IoT Core.

Mục này bổ sung giao tiếp hai chiều.

AWS IoT Core gửi command đến ESP32-S3 thông qua một MQTT topic. Sau khi nhận message, thiết bị thay đổi đầu ra relay và cập nhật trạng thái mới lên AWS IoT Device Shadow.

Luồng điều khiển:

```text
AWS IoT Core
      ↓
MQTT command topic
      ↓
MQTT callback trên ESP32-S3
      ↓
Cập nhật trạng thái relay
      ↓
Publish reported state
      ↓
AWS IoT Device Shadow
```

Chức năng được triển khai qua ba module:

```text
mqtt_manager.cpp
        ↓
relay.cpp
        ↓
shadow_manager.cpp
```

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Khởi tạo GPIO điều khiển relay.
- Xử lý relay active-low.
- Subscribe MQTT command topic.
- Nhận command `ON` và `OFF`.
- Thay đổi trạng thái relay vật lý.
- Báo trạng thái hiện tại lên AWS IoT Device Shadow.
- Xử lý Shadow delta message.
- Kiểm tra điều khiển từ xa bằng AWS IoT MQTT Test Client.

---

# Thời gian thực hiện

**Khoảng 10–15 phút**

---

# Bước 1 - Cấu hình chân Relay

GPIO relay được khai báo trong:

```text
include/config.h
```

Ví dụ:

```cpp
constexpr uint8_t RELAY_PIN = 5;

constexpr uint8_t RELAY_ON_LEVEL = LOW;
constexpr uint8_t RELAY_OFF_LEVEL = HIGH;
```

Project sử dụng relay active-low.

Đối với relay active-low:

```text
GPIO LOW  → Relay ON
GPIO HIGH → Relay OFF
```

{{% notice warning %}}
Relay module có thể sử dụng logic active-low hoặc active-high. Cần kiểm tra thông số phần cứng trước khi kết nối tải điện.
{{% /notice %}}

---

# Bước 2 - Khai báo Relay Interface

Các hàm relay được khai báo trong:

```text
include/relay.h
```

```cpp
#pragma once

void initRelay();
void setRelayState(bool state);
bool getRelayState();

void setRelay(bool enabled);
void toggleRelay();
bool isRelayOn();
```

Interface cung cấp các chức năng:

- Khởi tạo chân output.
- Đặt trạng thái relay.
- Đọc trạng thái relay hiện tại.
- Đảo trạng thái relay.

---

# Bước 3 - Khởi tạo Relay

Relay module được triển khai trong:

```text
src/relay.cpp
```

Trạng thái relay được lưu nội bộ:

```cpp
namespace
{
bool relayState = false;
}

constexpr bool RELAY_ACTIVE_LOW = true;
```

Hàm khởi tạo cấu hình GPIO và đảm bảo relay bắt đầu ở trạng thái OFF:

```cpp
void initRelay()
{
    pinMode(RELAY_PIN, OUTPUT);

    digitalWrite(
        RELAY_PIN,
        RELAY_OFF_LEVEL
    );

    relayState = false;

    Serial.println(
        "Relay initialized: OFF"
    );
}
```

Khởi tạo relay ở trạng thái OFF giúp tránh thiết bị điện bị bật ngoài ý muốn khi ESP32-S3 khởi động.

---

# Bước 4 - Cập nhật trạng thái Relay

Hàm `setRelayState()` hỗ trợ cả relay active-low và active-high:

```cpp
void setRelayState(bool state)
{
    relayState = state;

    if (RELAY_ACTIVE_LOW)
    {
        digitalWrite(
            RELAY_PIN,
            state ? LOW : HIGH
        );
    }
    else
    {
        digitalWrite(
            RELAY_PIN,
            state ? HIGH : LOW
        );
    }

    Serial.print("Relay: ");
    Serial.println(
        state ? "ON" : "OFF"
    );
}
```

Boolean state biểu diễn trạng thái logic:

```text
true  → Relay ON
false → Relay OFF
```

Hàm chuyển trạng thái logic thành mức điện áp GPIO phù hợp với relay.

---

# Bước 5 - Cấu hình Command Topic

Command topic được khai báo trong `include/secrets.h`:

```cpp
constexpr const char *AWS_TOPIC_COMMAND =
    "smarthome/esp32-home-01/command";
```

Sau khi kết nối AWS IoT Core, ESP32-S3 subscribe topic này:

```cpp
bool commandOk =
    mqttClient.subscribe(
        AWS_TOPIC_COMMAND
    );

Serial.print("Command topic: ");
Serial.println(AWS_TOPIC_COMMAND);

Serial.print("Command subscribe: ");
Serial.println(
    commandOk ? "OK" : "FAILED"
);
```

AWS IoT Policy cần cho phép:

```text
iot:Subscribe
iot:Receive
```

đối với command topic.

---

# Bước 6 - Nhận MQTT Command

MQTT message được xử lý trong `mqttCallback()`:

```cpp
void mqttCallback(
    char *topic,
    byte *payload,
    unsigned int length
)
{
    Serial.print(
        "Message arrived on topic: "
    );

    Serial.println(topic);

    String message;

    for (
        unsigned int i = 0;
        i < length;
        i++
    )
    {
        message +=
            static_cast<char>(
                payload[i]
            );
    }

    message.trim();

    Serial.print("Payload: [");
    Serial.print(message);
    Serial.println("]");
```

Callback chuyển raw payload thành Arduino `String`.

`trim()` loại bỏ khoảng trắng và ký tự xuống dòng ở đầu hoặc cuối message.

---

# Bước 7 - Xử lý lệnh ON và OFF

Callback xác định message có đến từ command topic hay không:

```cpp
if (
    strcmp(
        topic,
        AWS_TOPIC_COMMAND
    ) == 0
)
{
    if (
        message.equalsIgnoreCase("ON")
    )
    {
        setRelayState(true);
        publishShadowReportedState();
    }
    else if (
        message.equalsIgnoreCase("OFF")
    )
    {
        setRelayState(false);
        publishShadowReportedState();
    }
    else
    {
        Serial.println(
            "Unknown relay command"
        );
    }
}
```

`equalsIgnoreCase()` cho phép sử dụng:

```text
ON
on
OFF
off
```

Sau khi thay đổi relay, firmware publish trạng thái mới lên Device Shadow.

---

# Bước 8 - Publish Reported State

Logic cập nhật Shadow được triển khai trong:

```text
src/shadow_manager.cpp
```

Payload có cấu trúc:

```json
{
  "state": {
    "reported": {
      "relay_on": true
    }
  }
}
```

Code trong project:

```cpp
void publishShadowReportedState()
{
    JsonDocument doc;

    doc["state"]["reported"]
       ["relay_on"] =
        getRelayState();

    char payload[256];

    size_t payloadLength =
        serializeJson(
            doc,
            payload,
            sizeof(payload)
        );

    bool result =
        mqttClient.publish(
            AWS_SHADOW_UPDATE_TOPIC,
            reinterpret_cast<
                const uint8_t *
            >(payload),
            payloadLength,
            false
        );

    Serial.print(
        "Shadow reported payload: "
    );

    Serial.println(payload);

    Serial.print(
        "Shadow publish result: "
    );

    Serial.println(
        result
            ? "SUCCESS"
            : "FAILED"
    );
}
```

Shadow update topic:

```text
$aws/things/esp32-home-01/shadow/update
```

Giá trị `reported` thể hiện trạng thái thực tế mà ESP32-S3 đã áp dụng.

---

# Bước 9 - Subscribe các Shadow Topic

Sau khi MQTT connected, project subscribe ba Shadow topic:

```cpp
bool shadowDeltaOk =
    mqttClient.subscribe(
        AWS_SHADOW_DELTA_TOPIC
    );

bool shadowAcceptedOk =
    mqttClient.subscribe(
        AWS_SHADOW_UPDATE_ACCEPTED_TOPIC
    );

bool shadowRejectedOk =
    mqttClient.subscribe(
        AWS_SHADOW_UPDATE_REJECTED_TOPIC
    );
```

Các topic được khai báo:

```cpp
#define AWS_SHADOW_UPDATE_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update"

#define AWS_SHADOW_DELTA_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/delta"

#define AWS_SHADOW_UPDATE_ACCEPTED_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/accepted"

#define AWS_SHADOW_UPDATE_REJECTED_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/rejected"
```

Ý nghĩa:

| Topic | Chức năng |
|---|---|
| `/shadow/update` | Publish desired hoặc reported state. |
| `/shadow/update/delta` | Gửi chênh lệch giữa desired và reported state. |
| `/shadow/update/accepted` | Xác nhận Shadow update thành công. |
| `/shadow/update/rejected` | Trả về lỗi Shadow update. |

---

# Bước 10 - Xử lý Shadow Delta

MQTT callback chuyển các message đến Shadow handler trước:

```cpp
if (
    handleShadowMessage(
        topic,
        payload,
        length
    )
)
{
    return;
}
```

Khi topic là Shadow delta, firmware parse JSON:

```cpp
if (
    strcmp(
        topic,
        AWS_SHADOW_DELTA_TOPIC
    ) == 0
)
{
    JsonDocument doc;

    DeserializationError error =
        deserializeJson(
            doc,
            payload,
            length
        );

    if (error)
    {
        Serial.print(
            "Shadow delta JSON error: "
        );

        Serial.println(
            error.c_str()
        );

        return true;
    }
```

Delta payload dự kiến:

```json
{
  "state": {
    "relay_on": true
  }
}
```

Firmware đọc và áp dụng desired state:

```cpp
if (
    doc["state"]["relay_on"]
        .is<bool>()
)
{
    bool desiredRelay =
        doc["state"]["relay_on"]
            .as<bool>();

    setRelayState(
        desiredRelay
    );

    publishShadowReportedState();
}
```

Sau khi thay đổi relay, ESP32-S3 báo trạng thái thực tế lên Shadow.

Kết quả:

```text
desired.relay_on
        =
reported.relay_on
```

---

# Bước 11 - Gửi Direct MQTT Command

Mở:

```text
AWS IoT Core
→ Test
→ MQTT test client
→ Publish to a topic
```

Topic:

```text
smarthome/esp32-home-01/command
```

Bật relay:

```text
ON
```

Tắt relay:

```text
OFF
```

![Publish lệnh điều khiển relay](/fcj-workshop-template/images/workshop/5.5.5/publish-relay-command.png)

{{% notice note %}}
Firmware hiện tại nhận command dạng plain text `ON` hoặc `OFF`, không nhận JSON trên direct command topic.
{{% /notice %}}

---

# Bước 12 - Kiểm tra phản hồi trên ESP32-S3

Mở Serial Monitor:

```bash
pio device monitor
```

Kết quả khi nhận ON:

```text
Message arrived on topic:
smarthome/esp32-home-01/command

Payload: [ON]
Relay: ON
Shadow reported payload:
{"state":{"reported":{"relay_on":true}}}
Shadow publish result: SUCCESS
```

Kết quả khi nhận OFF:

```text
Payload: [OFF]
Relay: OFF
Shadow reported payload:
{"state":{"reported":{"relay_on":false}}}
Shadow publish result: SUCCESS
```

![ESP32-S3 nhận lệnh Relay](/fcj-workshop-template/images/workshop/5.5.5/relay-command-received.png)

---

# Bước 13 - Điều khiển bằng Device Shadow

Có thể điều khiển relay bằng cách thay đổi desired state.

Publish vào:

```text
$aws/things/esp32-home-01/shadow/update
```

Payload:

```json
{
  "state": {
    "desired": {
      "relay_on": true
    }
  }
}
```

Khi desired khác reported, AWS IoT Device Shadow gửi delta:

```json
{
  "state": {
    "relay_on": true
  }
}
```

ESP32-S3 thực hiện:

1. Bật relay.
2. Publish `reported.relay_on = true`.
3. Nhận update accepted.
4. Đồng bộ desired và reported state.

---

# So sánh Direct Command và Device Shadow

| Phương thức | Payload | Mục đích |
|---|---|---|
| MQTT command topic | `ON` hoặc `OFF` | Điều khiển tức thời khi thiết bị đang online. |
| Device Shadow | JSON desired state | Đồng bộ trạng thái thiết bị với Cloud. |

Direct command chỉ được nhận khi ESP32-S3 đang kết nối MQTT.

Device Shadow lưu desired state trên AWS, phù hợp cho việc đồng bộ trạng thái sau khi thiết bị reconnect khi có triển khai đầy đủ quy trình lấy Shadow hiện tại.

---

# Quyền bảo mật cần thiết

AWS IoT Policy phải cho phép direct command topic và các Shadow topic.

Các action thường dùng:

```text
iot:Connect
iot:Publish
iot:Subscribe
iot:Receive
```

Resource ARN phải sử dụng đúng loại:

```text
client/
topic/
topicfilter/
```

Ví dụ:

```text
topic/smarthome/esp32-home-01/command
topicfilter/smarthome/esp32-home-01/command
topic/$aws/things/esp32-home-01/shadow/update
topicfilter/$aws/things/esp32-home-01/shadow/update/delta
```

{{% notice warning %}}
Trong môi trường production, cần giới hạn resource ARN theo nguyên tắc Least Privilege. Policy wildcard chỉ nên dùng tạm thời trong quá trình kiểm thử.
{{% /notice %}}

---

# Xử lý lỗi

## Subscribe command thất bại

Kiểm tra:

- MQTT đã connected.
- Topic chính xác.
- IoT Policy có `iot:Subscribe`.
- Subscribe resource dùng `topicfilter/`.
- Certificate đã attach đúng Policy.

---

## Nhận message nhưng Relay không đổi

Kiểm tra:

- Command chính xác là `ON` hoặc `OFF`.
- `RELAY_PIN` khớp với mạch.
- Loại relay active-low hoặc active-high.
- Nguồn relay phù hợp.
- ESP32-S3 và relay module dùng chung GND.

---

## Trạng thái Relay bị đảo

Kiểm tra:

```cpp
constexpr bool RELAY_ACTIVE_LOW =
    true;
```

Chỉ đổi thành `false` nếu relay sử dụng active-high.

---

## Shadow update bị rejected

Kiểm tra:

- IoT Policy cấp quyền cho Shadow topic.
- `AWS_THING_NAME` khớp Thing thật trên AWS.
- Topic Shadow đúng định dạng.
- JSON payload hợp lệ.

Rejected payload được in qua:

```cpp
Serial.print(
    "Shadow update rejected: "
);
```

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- ESP32-S3 subscribe command topic thành công.
- AWS IoT Core có thể bật và tắt relay.
- Command được xử lý không phân biệt chữ hoa/thường.
- Trạng thái relay được đưa vào telemetry.
- Trạng thái thực tế được publish lên AWS IoT Device Shadow.
- Shadow delta có thể điều khiển relay.
- Firmware xử lý accepted và rejected Shadow update.

{{% notice tip %}}
Firmware ESP32-S3 đã hoàn thành giao tiếp hai chiều với AWS IoT Core. Ở chương tiếp theo, AWS IoT Rules Engine sẽ chuyển telemetry sang Amazon DynamoDB và Amazon SNS.
{{% /notice %}}

**Tiếp theo:** [5.6 Tích hợp dịch vụ Cloud](../../5.6-cloud-integration/)