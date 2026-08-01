---
title: "Proposal"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Smart Home IoT System on AWS

## A Secure IoT-Based Home Monitoring System Using AWS Cloud Services

---

# 1. Executive Summary

This project proposes the development of a Smart Home IoT system using an ESP32-S3 development board integrated with AWS cloud services.

The system continuously monitors environmental conditions, including temperature, humidity, ambient light intensity, and door status. Sensor data are securely transmitted to AWS IoT Core using MQTT over TLS 1.2 with X.509 certificate authentication.

AWS IoT Rules Engine processes incoming telemetry data and routes them to Amazon DynamoDB for storage. When the system detects that the door has been opened, Amazon SNS automatically sends an email notification to the user.

The proposed architecture demonstrates a lightweight, secure, and scalable IoT solution suitable for home automation and educational purposes.

---

# 2. Problem Statement

## Current Challenges

Traditional home monitoring systems often rely on local devices without centralized management.

These systems usually lack:

- Real-time remote monitoring.
- Secure device authentication.
- Centralized telemetry storage.
- Automatic event notifications.
- Cloud-based scalability.

As the number of connected devices increases, managing sensor data and monitoring system status becomes increasingly difficult.

---

## Proposed Solution

The proposed Smart Home IoT system utilizes AWS managed services to provide secure communication, centralized data storage, and real-time monitoring.

The ESP32-S3 collects sensor readings from multiple devices and publishes telemetry messages to AWS IoT Core through MQTT over TLS.

AWS IoT Rules Engine automatically routes incoming telemetry to Amazon DynamoDB for long-term storage.

Whenever the door sensor detects an open event, another IoT Rule publishes a notification to Amazon SNS, which immediately sends an email alert to registered subscribers.

The system also supports remote relay control through MQTT command topics.

---

# Benefits

The proposed solution provides:

- Secure communication using MQTT over TLS 1.2.
- Device authentication using X.509 certificates.
- Centralized telemetry storage.
- Automatic email notifications.
- Low operational cost.
- Simple architecture with fully managed AWS services.
- Easy scalability for future Smart Home devices.

---

# 3. Solution Architecture

The Smart Home IoT system consists of an ESP32-S3 device connected to several environmental sensors and AWS cloud services.

The ESP32-S3 periodically collects telemetry data and publishes JSON messages to AWS IoT Core.

AWS IoT Core authenticates the device using an X.509 certificate and AWS IoT Policy before forwarding messages to AWS IoT Rules Engine.

The Rules Engine stores telemetry in Amazon DynamoDB and sends door-open alerts through Amazon SNS.

The proposed architecture is illustrated below.

![Smart Home IoT Architecture](/images/workshop/5.2/architec.jpg)

*Figure: Proposed architecture of the Smart Home IoT system on AWS.*

---

# AWS Services Used

- AWS IoT Core
- AWS IoT Rules Engine
- Amazon DynamoDB
- Amazon Simple Notification Service (Amazon SNS)
- AWS Identity and Access Management (AWS IAM)
- Amazon CloudWatch

---

# Hardware Components

- ESP32-S3 Development Board
- DHT11 Temperature and Humidity Sensor
- LDR Light Sensor
- Magnetic Door Sensor
- Relay Module

---

# 4. Technical Implementation

## Implementation Phases

The project is divided into four implementation phases.

### Phase 1

Requirement analysis and system architecture design.

### Phase 2

AWS IoT Core configuration, including Thing creation, X.509 certificates, IoT Policy, and MQTT testing.

### Phase 3

Embedded firmware development for ESP32-S3, including Wi-Fi connection, MQTT over TLS communication, telemetry generation, and relay control.

### Phase 4

Cloud integration, testing, system validation, and documentation.

---

# Technical Requirements

Software

- Visual Studio Code
- PlatformIO
- AWS Management Console

AWS Services

- AWS IoT Core
- DynamoDB
- Amazon SNS
- AWS IAM

Programming Language

- C++
- Arduino Framework

Communication

- MQTT over TLS 1.2

---

# 5. Timeline

| Week | Activities |
|---|---|
| Week 1 | Requirement analysis and AWS research |
| Week 2 | Configure AWS IoT Core and development environment |
| Week 3 | Develop firmware and configure cloud services |
| Week 4 | Integrate ESP32-S3 with AWS IoT Core |
| Week 5 | Optimize system architecture and firmware |
| Week 6 | Perform system testing |
| Week 7 | Complete documentation and final presentation |

---

# 6. Budget Estimation

The project uses AWS Free Tier services whenever possible.

Estimated operational cost is minimal because:

- AWS IoT Core message volume is low.
- Amazon DynamoDB stores only lightweight telemetry.
- Amazon SNS sends only event-driven notifications.
- CloudWatch usage is limited to monitoring and logs.

Hardware costs include:

- ESP32-S3 Development Board
- DHT11 Sensor
- LDR Sensor
- Door Sensor
- Relay Module

---

# 7. Risk Assessment

## Potential Risks

- Wi-Fi connection failure.
- MQTT communication interruption.
- Sensor malfunction.
- Incorrect AWS configuration.
- Email notification delays.

---

## Mitigation

- Implement automatic Wi-Fi reconnection.
- Implement MQTT reconnection.
- Validate sensor readings.
- Apply AWS IAM least-privilege permissions.
- Test AWS IoT Rules before deployment.

---

# 8. Expected Outcomes

The completed Smart Home IoT system will provide:

- Secure communication between ESP32-S3 and AWS IoT Core.
- Real-time monitoring of environmental conditions.
- Remote relay control through MQTT.
- Automatic email notifications for door-open events.
- Centralized telemetry storage in Amazon DynamoDB.
- A scalable architecture suitable for future Smart Home expansion.

The project also provides a practical example of integrating embedded systems with AWS cloud services for IoT applications.