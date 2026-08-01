---
title: "Connect to AWS IoT Core using MQTT over TLS"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.5.3 </b> "
---

{{% notice info %}}
In this section, the ESP32-S3 synchronizes its system time and establishes a secure MQTT connection with AWS IoT Core using mutual TLS authentication.
{{% /notice %}}

# Overview

AWS IoT Core requires secure device communication.

The ESP32-S3 connects to the AWS IoT endpoint through MQTT over TLS on TCP port `8883`. Mutual TLS authentication is performed using:

- Amazon Root CA 1
- X.509 device certificate
- Device private key

Before starting the TLS handshake, the ESP32-S3 must have a valid system time. Certificate validation may fail when the device clock is incorrect.

The firmware therefore performs the following sequence:

```text
Connect to Wi-Fi
        ↓
Synchronize time using NTP
        ↓
Load CA, certificate, and private key
        ↓
Configure the MQTT client
        ↓
Connect to AWS IoT Core
        ↓
Subscribe to command and Shadow topics
```

---

# Objectives

After completing this section, you will be able to:

- Synchronize ESP32-S3 time using NTP.
- Configure `WiFiClientSecure`.
- Load the AWS Root CA and device credentials.
- Configure `PubSubClient`.
- Connect securely to AWS IoT Core.
- Subscribe to MQTT command topics.
- Implement MQTT reconnection handling.

---

# Estimated Time

**Approximately 10–15 minutes**

---

# Step 1 – Add the Required Libraries

The project uses the following libraries:

```ini
lib_deps =
    knolleary/PubSubClient@2.8
    bblanchon/ArduinoJson
    adafruit/DHT sensor library
    adafruit/Adafruit Unified Sensor
```

`PubSubClient` provides MQTT communication, while `WiFiClientSecure` provides the TLS transport layer.

The project also uses a larger MQTT buffer because Shadow and telemetry payloads may be larger than the default PubSubClient packet size.

---

# Step 2 – Define the AWS IoT Configuration

Store the AWS IoT endpoint, client ID, MQTT topics, and certificates in `include/secrets.h`.

Use placeholders in shared or public source code:

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

The security credentials are stored as PEM strings:

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
Never publish the private key, Wi-Fi password, or production device certificate. Add `include/secrets.h` and the certificate directory to `.gitignore`.
{{% /notice %}}

Example `.gitignore` entries:

```gitignore
include/secrets.h
certificates/
*.pem
*.key
*.crt
```

---

# Step 3 – Synchronize the Device Time

The project separates time synchronization into `time_manager.cpp`.

The device requests the current time from multiple public NTP servers:

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

The firmware waits for a valid Unix timestamp for a maximum of 30 seconds.

A valid system time is required so that the ESP32-S3 can verify the validity period of the AWS IoT server certificate.

---

# Step 4 – Create the Secure MQTT Clients

The project creates one global TLS client and one MQTT client in `mqtt_manager.cpp`.

```cpp
WiFiClientSecure secureClient;
PubSubClient mqttClient(secureClient);
```

`secureClient` handles the encrypted TLS connection.

`mqttClient` uses that secure connection to exchange MQTT packets with AWS IoT Core.

The objects are declared as global objects so they remain available throughout the program lifecycle.

---

# Step 5 – Initialize TLS and MQTT

The project configures the clients through the `initMQTT()` function:

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

The configuration performs the following tasks:

| Configuration | Purpose |
|---|---|
| `setCACert()` | Verifies the AWS IoT Core server certificate. |
| `setCertificate()` | Sends the ESP32-S3 device certificate. |
| `setPrivateKey()` | Proves ownership of the device certificate. |
| `setServer()` | Configures the AWS IoT endpoint and MQTT port. |
| `setCallback()` | Registers the message receive handler. |
| `setKeepAlive()` | Configures the MQTT keep-alive interval. |
| `setBufferSize()` | Increases the MQTT packet buffer to 1024 bytes. |

---

# Step 6 – Connect to AWS IoT Core

The connection logic is implemented in `connectMQTT()`.

A simplified extract from the project is shown below:

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

The MQTT Client ID must match the resource permitted by the AWS IoT Policy when the policy uses a restricted client ARN.

For example:

```text
arn:aws:iot:us-east-1:ACCOUNT_ID:client/esp32-home-01
```

---

# Step 7 – Subscribe to the Command Topic

After the MQTT connection is established, the ESP32-S3 subscribes to the command topic:

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

The command topic is:

```text
smarthome/esp32-home-01/command
```

AWS IoT Policy must grant both:

```text
iot:Subscribe
iot:Receive
```

to allow the device to subscribe and receive messages.

---

# Step 8 – Initialize the Connection in setup()

The project performs the startup sequence in `main.cpp`.

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

The order is important:

1. Wi-Fi must be connected.
2. Time must be synchronized.
3. TLS credentials must be configured.
4. MQTT connection can then be established.

---

# Step 9 – Maintain the MQTT Connection

The MQTT client requires `loop()` to run frequently.

The main program checks the connection and reconnects when necessary:

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

The reconnect interval is set to 10 seconds to prevent repeated TLS connection attempts in a tight loop.

Calling `secureClient.stop()` releases the previous TLS socket before a new connection is attempted.

---

# Step 10 – Upload and Monitor the Firmware

Build and upload the firmware:

```bash
pio run -t upload
```

Open the Serial Monitor:

```bash
pio device monitor
```

Expected output:

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

![MQTT over TLS connected](/images/workshop/5.5.3/mqtt-tls-connected.png)

---

# MQTT Connection States

When the connection fails, `mqttClient.state()` provides diagnostic information.

| State | Meaning |
|---:|---|
| `0` | Connected |
| `-1` | Disconnected |
| `-2` | Network or TCP connection failure |
| `-3` | Connection lost |
| `-4` | Connection timeout |
| `-5` | Connection rejected or TLS-related failure |

When troubleshooting, verify:

- Wi-Fi connection.
- Current system time.
- AWS IoT endpoint.
- Certificate status.
- Device certificate and private key pair.
- AWS IoT Policy attachment.
- MQTT Client ID.
- TCP port `8883`.

---

# Expected Result

After completing this section:

- ESP32-S3 synchronizes its system time using NTP.
- TLS credentials are loaded successfully.
- Mutual TLS authentication succeeds.
- ESP32-S3 connects to AWS IoT Core.
- The MQTT command topic is subscribed successfully.
- The firmware automatically attempts to reconnect after a connection loss.

{{% notice tip %}}
The next section uses the established MQTT connection to publish JSON telemetry from the ESP32-S3 to AWS IoT Core.
{{% /notice %}}

**Next:** [5.5.4 Publish Telemetry](../5.5.4-telemetry/)