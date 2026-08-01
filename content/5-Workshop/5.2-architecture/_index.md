---
title: "Architecture"
date: 2026-08-01
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---

{{% notice info %}}
This chapter explains the overall architecture of the Smart Home IoT system, the interaction between the ESP32-S3 and AWS managed services, and the complete telemetry data flow from the physical device to cloud services.
{{% /notice %}}

# System Architecture

The Smart Home IoT system follows a **serverless**, **event-driven**, and **cloud-native** architecture built entirely on managed AWS services.

Unlike traditional IoT deployments that require dedicated application servers, databases, and message brokers, this solution relies on AWS managed services to provide secure communication, telemetry processing, data persistence, and event notifications.

The architecture consists of two major environments:

- Smart Home Environment
- AWS Cloud

The following architecture diagram illustrates the complete workflow implemented throughout this workshop.

![Architecture](/images/workshop/5.2/smart-home-architecture.png)

---

# Overall Architecture

The Smart Home system contains a physical device and several cloud services working together.

The ESP32-S3 periodically collects environmental information from connected sensors and publishes telemetry messages to AWS IoT Core using MQTT over TLS 1.2.

AWS IoT Core authenticates the device using an X.509 certificate and authorizes communication through an AWS IoT Policy.

Incoming telemetry messages are processed by AWS IoT Rules Engine before being routed to Amazon DynamoDB for storage or Amazon SNS for event notifications.

This architecture completely eliminates the need to manage backend servers while maintaining secure communication and high scalability.

---

# Architecture Components

The system is divided into the following logical components.

## Smart Home Environment

The Smart Home environment contains all physical devices installed inside the house.

### ESP32-S3

The ESP32-S3 development board serves as the central controller of the Smart Home system.

Its responsibilities include:

- Connecting to the Wi-Fi network.
- Synchronizing time using NTP.
- Reading sensor values.
- Creating telemetry payloads.
- Publishing MQTT messages.
- Receiving MQTT commands.
- Controlling the relay.

---

### DHT11 Sensor

The DHT11 sensor periodically measures:

- Temperature
- Relative humidity

These values are included in every telemetry message sent to AWS IoT Core.

---

### LDR Sensor

The Light Dependent Resistor (LDR) measures ambient light intensity.

The measured value is used to monitor environmental lighting conditions.

---

### Magnetic Door Sensor

The magnetic door sensor detects whether the door is opened or closed.

Whenever the door status changes to **open**, AWS IoT Rules Engine triggers an event notification.

---

### Relay Module

The relay represents a controllable electrical device such as:

- Lamp
- Fan
- Electrical outlet

Instead of being controlled locally, the relay receives commands from AWS IoT Core through MQTT.

---

# AWS Cloud

The cloud side of the architecture consists entirely of managed AWS services.

The workshop is deployed in:

```text
US East (N. Virginia)
us-east-1
```

---

# AWS IoT Core

AWS IoT Core is the central communication service of the architecture.

It provides three major functions.

## Thing Registry

Each ESP32-S3 device is registered as an **AWS IoT Thing**.

The Thing Registry stores the logical identity of every connected IoT device.

---

## AWS IoT Message Broker

The Message Broker provides MQTT communication between devices and cloud applications.

It supports:

- MQTT Publish
- MQTT Subscribe
- MQTT QoS
- Secure TLS communication

The ESP32-S3 publishes telemetry through the topic:

```text
smarthome/esp32-home-01/telemetry
```

and subscribes to

```text
smarthome/esp32-home-01/command
```

for relay control.

---

## AWS IoT Rules Engine

The Rules Engine automatically evaluates every incoming MQTT message.

Instead of requiring application servers, the Rules Engine directly routes telemetry to AWS services.

In this workshop, two logical rules are implemented.

### Telemetry Rule

Stores telemetry records in Amazon DynamoDB.

### Door Alert Rule

Checks whether

```text
door_open == true
```

If the condition is satisfied, the rule publishes a notification to Amazon SNS.

---

# Amazon DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service.

The telemetry table stores:

- Device ID
- Timestamp
- Temperature
- Humidity
- Light level
- Door status
- Relay status

A typical telemetry record is shown below.

| Attribute | Description |
|------------|-------------|
| device_id | Device identifier |
| timestamp | Measurement time |
| temperature | Temperature value |
| humidity | Humidity value |
| light | LDR reading |
| door_open | Door state |
| relay_on | Relay state |

---

# Amazon SNS

Amazon Simple Notification Service (Amazon SNS) provides event-driven notifications.

When the Rules Engine detects an open-door event, SNS immediately sends an email notification to every confirmed subscriber.

The notification flow is:

```text
Door Sensor

↓

AWS IoT Rule

↓

Amazon SNS Topic

↓

Subscriber Email
```

---

# Identity and Security

Security is one of the most important aspects of an IoT system.

The architecture uses multiple AWS security mechanisms.

## X.509 Device Certificate

Every ESP32-S3 owns a unique X.509 certificate.

The certificate authenticates the device before MQTT communication is established.

---

## AWS IoT Policy

The IoT Policy defines which MQTT operations the device is allowed to perform.

Typical permissions include:

- iot:Connect
- iot:Publish
- iot:Subscribe
- iot:Receive

---

## AWS IAM Role

The AWS IAM Role is assigned to AWS IoT Rules Engine.

Instead of authenticating devices, the IAM Role grants cloud-side permissions.

The role allows:

- Writing telemetry to Amazon DynamoDB.
- Publishing notifications to Amazon SNS.

---

# Monitoring

The architecture includes two monitoring services.

## Amazon CloudWatch

Amazon CloudWatch provides:

- Metrics
- Logs
- Operational monitoring
- Troubleshooting

---

## AWS CloudTrail

AWS CloudTrail records:

- AWS API calls
- Configuration changes
- Security events

This helps administrators audit all cloud activities.

---

# Complete Data Flow

The Smart Home system operates according to the following sequence.

1. ESP32-S3 connects to Wi-Fi.
2. The device synchronizes time through NTP.
3. Sensor values are collected.
4. A JSON telemetry payload is generated.
5. MQTT publishes telemetry through TLS.
6. AWS IoT Core authenticates the device.
7. AWS IoT Rules Engine evaluates the telemetry.
8. Telemetry is stored in Amazon DynamoDB.
9. Door-open events trigger Amazon SNS.
10. Amazon SNS sends an email notification.
11. AWS IoT Core can publish MQTT commands.
12. ESP32-S3 receives commands and updates the relay state.

---

# Why This Architecture?

This architecture was selected because it provides several important advantages.

- Fully managed AWS services.
- No application server maintenance.
- Secure MQTT communication using TLS.
- Device authentication with X.509 certificates.
- Event-driven processing.
- High scalability.
- Low operational cost.
- Easy integration with ESP32 devices.
- Native support for AWS IoT services.

The architecture is suitable for educational projects, rapid prototyping, and small to medium Smart Home deployments.

{{% notice tip %}}
The following chapters describe how each component of this architecture is implemented step by step, beginning with the creation of AWS IoT resources and device authentication.
{{% /notice %}}

**Next:** [Prerequisites](../5.3-prerequisites/)