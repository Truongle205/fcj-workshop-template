---
title: "Publish Smart Home Telemetry"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.5.4 </b> "
---

{{% notice info %}}
In this section, the ESP32-S3 reads sensor values, creates a JSON telemetry payload, and publishes the message securely to AWS IoT Core.
{{% /notice %}}

# Overview

After establishing the MQTT connection, the ESP32-S3 can begin sending Smart Home telemetry to AWS IoT Core.

The firmware collects:

- Temperature.
- Humidity.
- Ambient light.
- Door state.
- Relay state.
- Device timestamp.

The collected values are placed into a JSON document and published to:

```text
smarthome/esp32-home-01/telemetry
```

The telemetry workflow is implemented by three main components:

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

# Objectives

After completing this section, you will be able to:

- Initialize the Smart Home sensors.
- Read temperature and humidity from the DHT11.
- Read an analog light value from the LDR.
- Read the digital door sensor state.
- Read the current relay state.
- Create a telemetry message using ArduinoJson.
- Publish the JSON payload to AWS IoT Core.
- Handle invalid sensor readings.

---

# Estimated Time

**Approximately 10–15 minutes**

---

# Step 1 – Define the Sensor Data Structure

The project uses a structure named `SensorData` to group the values returned by the sensor module.

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

Using a structure keeps sensor-related values together and makes it easier to pass the complete measurement result to the telemetry module.

| Field | Type | Description |
|---|---|---|
| `temperature` | `float` | Temperature measured by the DHT11. |
| `humidity` | `float` | Relative humidity measured by the DHT11. |
| `light` | `int` | Analog value read from the LDR. |
| `doorOpen` | `bool` | Current door state. |
| `dhtValid` | `bool` | Indicates whether the DHT11 values are valid. |

---

# Step 2 – Configure Sensor Pins

The project defines sensor pins in `include/config.h`.

```cpp
constexpr uint8_t LDR_PIN = 1;
constexpr uint8_t DHT_PIN = 4;
constexpr uint8_t DHT_TYPE = 11;
constexpr uint8_t DOOR_SENSOR_PIN = 2;
constexpr uint8_t RELAY_PIN = 5;
```

The actual pin numbers may be changed to match the selected ESP32-S3 board and wiring configuration.

{{% notice warning %}}
Verify the pinout of the selected ESP32-S3 board before connecting sensors. Some pins may be reserved for flash, PSRAM, USB, or other board functions.
{{% /notice %}}

---

# Step 3 – Initialize the Sensors

The sensor initialization logic is implemented in `src/sensors.cpp`.

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

The initialization performs the following operations:

| Code | Purpose |
|---|---|
| `dht.begin()` | Starts communication with the DHT11 sensor. |
| `INPUT_PULLUP` | Enables the internal pull-up resistor for the door input. |
| `analogReadResolution(12)` | Configures the ESP32 ADC for 12-bit readings. |

With 12-bit ADC resolution, the LDR reading normally ranges from:

```text
0 to 4095
```

---

# Step 4 – Read Sensor Values

The `readSensors()` function collects all measurements.

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

The function performs four main readings:

1. Temperature from the DHT11.
2. Humidity from the DHT11.
3. Light level from the ADC.
4. Door state from the digital input.

---

# Step 5 – Validate DHT11 Data

The DHT11 library returns `NaN` when a sensor reading fails.

The project checks both values:

```cpp
data.dhtValid =
    !isnan(data.temperature) &&
    !isnan(data.humidity);
```

This validation prevents invalid floating-point data from being treated as a valid temperature or humidity measurement.

The project also prints sensor values to the Serial Monitor:

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

Example output:

```text
Temperature: 29.40
Humidity: 71.00
Light: 870
Door: CLOSED
```

---

# Step 6 – Create the JSON Telemetry Document

The telemetry publishing logic is implemented in:

```text
src/telemetry.cpp
```

The function first retrieves the current sensor values:

```cpp
void publishTelemetry()
{
    SensorData sensorData =
        readSensors();

    JsonDocument document;
```

The device identifier and Unix timestamp are then added:

```cpp
document["device_id"] =
    AWS_THING_NAME;

document["timestamp"] =
    static_cast<int64_t>(time(nullptr));
```

The timestamp is generated from the system time synchronized through NTP in the previous section.

---

# Step 7 – Add Sensor and Device State

The firmware records whether the DHT11 reading is valid:

```cpp
document["dht_valid"] =
    sensorData.dhtValid;
```

When the reading is valid:

```cpp
if (sensorData.dhtValid)
{
    document["temperature"] =
        sensorData.temperature;

    document["humidity"] =
        sensorData.humidity;
}
```

When the DHT11 reading fails, the payload uses JSON `null` values:

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

The remaining values are added independently:

```cpp
document["light"] =
    sensorData.light;

document["door_open"] =
    sensorData.doorOpen;

document["relay_on"] =
    isRelayOn();
```

The use of `null` prevents invalid DHT11 values from being stored as real measurements.

---

# Step 8 – Serialize the JSON Payload

The JSON document is serialized into a fixed-size character buffer:

```cpp
char payload[320];

size_t length = serializeJson(
    document,
    payload,
    sizeof(payload)
);
```

The code checks that serialization succeeded:

```cpp
if (length == 0)
{
    Serial.println(
        "JSON serialization failed"
    );

    return;
}
```

A fixed buffer avoids dynamic string allocation during every telemetry cycle.

---

# Step 9 – Publish the Telemetry Message

Before publishing, the payload is printed to the Serial Monitor:

```cpp
Serial.print(
    "Publishing telemetry: "
);

Serial.println(payload);
```

The message is then published through the shared MQTT manager:

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

The `publishMessage()` function first verifies that MQTT is connected:

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

This separation keeps the telemetry module independent from the internal MQTT client implementation.

---

# Step 10 – Publish Periodically

The project publishes telemetry periodically from `main.cpp`.

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

The current interval is:

```text
10 seconds
```

Using `millis()` avoids blocking the MQTT loop for long periods.

---

# Example Telemetry Payload

When all sensors are available, the message may look like:

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

When the DHT11 reading is invalid:

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

# Step 11 – Upload and Monitor

Build and upload the firmware:

```bash
pio run -t upload
```

Open the Serial Monitor:

```bash
pio device monitor
```

Expected Serial output:

```text
Publishing telemetry...
Temperature: 29.40
Humidity: 71.00
Light: 870
Door: CLOSED
Publishing telemetry: {"device_id":"esp32-home-01",...}
Published: {"device_id":"esp32-home-01",...}
```

![Telemetry published by ESP32-S3](/fcj-workshop-template/images/workshop/5.5.4/telemetry-published.png)

---

# Step 12 – Verify Telemetry in AWS IoT Core

Open:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Subscribe to:

```text
smarthome/esp32-home-01/telemetry
```

The telemetry message published by the ESP32-S3 should appear in the MQTT Test Client.

![Telemetry received in AWS IoT Core](/fcj-workshop-template/images/workshop/5.5.4/telemetry-received.png)

This verifies the complete path:

```text
Sensors
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

# Troubleshooting

## DHT11 returns invalid data

Possible causes:

- Sensor is not connected.
- Incorrect GPIO pin.
- Data line has no pull-up resistor.
- Sensor is read too frequently.
- Power supply is unstable.

The telemetry module continues publishing other values and sets temperature and humidity to `null`.

---

## Telemetry is not visible in AWS

Verify:

- MQTT connection is active.
- The telemetry topic matches exactly.
- AWS IoT Policy allows `iot:Publish`.
- The MQTT Test Client subscribes to the correct topic.
- The payload size is smaller than the MQTT buffer.

---

## Light value does not change

Verify:

- The LDR is connected to an ADC-capable pin.
- The voltage divider is wired correctly.
- The analog pin is not used by another board function.

---

# Expected Result

After completing this section:

- All sensor values are read by the ESP32-S3.
- A JSON telemetry message is created.
- Invalid DHT11 readings are represented safely.
- Telemetry is published periodically.
- AWS IoT Core receives the telemetry message.
- Relay and door states are included in every payload.

{{% notice tip %}}
The next section adds bidirectional communication by allowing AWS IoT Core to send commands to the ESP32-S3 and control the relay remotely.
{{% /notice %}}

**Next:** [5.5.5 Remote Relay Control](../5.5.5-relay-control/)