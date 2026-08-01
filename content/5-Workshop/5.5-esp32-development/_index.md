---
title: "ESP32 Development"
date: 2026-08-01
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

{{% notice info %}}
In this chapter, you will develop the firmware for the ESP32-S3. The device will connect to Wi-Fi, establish a secure MQTT connection with AWS IoT Core, publish telemetry data, and receive remote control commands.
{{% /notice %}}

# ESP32 Development

After configuring AWS IoT Core, the next step is to implement the firmware running on the ESP32-S3 development board.

The firmware is responsible for communicating with AWS Cloud and controlling the Smart Home hardware.

Throughout this chapter, the ESP32-S3 will gradually gain the ability to:

- Connect to a Wi-Fi network.
- Synchronize system time using NTP.
- Authenticate with AWS IoT Core using an X.509 certificate.
- Publish telemetry messages in JSON format.
- Subscribe to MQTT command topics.
- Control the relay remotely.

---

# Objectives

After completing this chapter, you will be able to:

- Create a PlatformIO project.
- Configure the ESP32-S3 development environment.
- Connect the ESP32-S3 to Wi-Fi.
- Establish a secure MQTT over TLS connection.
- Publish Smart Home telemetry.
- Receive MQTT commands from AWS IoT Core.
- Control a relay remotely.

---

# Development Environment

The firmware is developed using the following tools.

| Tool | Purpose |
|------|---------|
| Visual Studio Code | Source code editor |
| PlatformIO IDE | ESP32 development platform |
| ESP32 Arduino Framework | Firmware framework |
| ArduinoJson | JSON serialization |
| PubSubClient | MQTT client |
| WiFiClientSecure | TLS communication |

---

# Project Structure

The PlatformIO project is organized as follows.

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

---

# Firmware Features

The ESP32 firmware implements several independent modules.

| Module | Description |
|---------|-------------|
| Wi-Fi Manager | Connects to the wireless network |
| NTP Client | Synchronizes system time |
| MQTT Client | Connects to AWS IoT Core |
| Telemetry Publisher | Sends sensor data |
| Command Subscriber | Receives MQTT commands |
| Relay Controller | Controls relay output |

---

# Development Workflow

The firmware development process is divided into the following sections.

```text
PlatformIO Project

↓

Wi-Fi Connection

↓

MQTT over TLS

↓

Telemetry Publishing

↓

Relay Control
```

Each feature is implemented and verified individually before moving to the next step.

---

# Estimated Time

| Section | Estimated Time |
|----------|---------------:|
| 5.5.1 Create PlatformIO Project | 5 minutes |
| 5.5.2 Wi-Fi Connection | 5 minutes |
| 5.5.3 MQTT over TLS | 10 minutes |
| 5.5.4 Publish Telemetry | 10 minutes |
| 5.5.5 Relay Control | 10 minutes |

**Estimated completion time: 40 minutes**

---

{{% notice tip %}}
At the end of this chapter, the ESP32-S3 will be fully connected to AWS IoT Core and capable of exchanging MQTT messages securely.
{{% /notice %}}

**Next:** [5.5.1 Create PlatformIO Project](5.5.1-platformio/)