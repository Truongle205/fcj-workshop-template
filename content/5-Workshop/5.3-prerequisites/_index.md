---
title: "Prerequisites"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

{{% notice warning %}}
Before starting this workshop, ensure that all required hardware, software, and AWS resources are prepared. Completing the prerequisites first will help avoid configuration issues during later chapters.
{{% /notice %}}

# Prerequisites

This chapter describes the required hardware, software, AWS resources, and background knowledge needed before implementing the Smart Home IoT system.

---

# Hardware Requirements

The following hardware components are used throughout this workshop.

| Hardware | Description |
|-----------|-------------|
| ESP32-S3 Development Board | Main IoT controller |
| USB Type-C Cable | Programming and Serial Monitor |
| DHT11 Sensor | Temperature and Humidity |
| LDR Sensor | Ambient Light |
| Magnetic Door Sensor | Door Detection |
| Relay Module | Remote Appliance Control |
| Breadboard & Jumper Wires | Circuit Connection |
| Wi-Fi Network | Internet Connectivity |

The ESP32-S3 is responsible for collecting sensor readings, publishing telemetry, and receiving MQTT commands from AWS IoT Core.

---

# Software Requirements

Install the following software before beginning the workshop.

| Software | Purpose |
|-----------|---------|
| Visual Studio Code | Code editor |
| PlatformIO IDE | ESP32 development environment |
| Git | Version control |
| Python 3 | PlatformIO dependency |
| Serial Monitor | Debugging |

Verify that PlatformIO is correctly installed by running:

```bash
pio --version
```

Expected output:

```text
PlatformIO Core, version 6.x.x
```

---

# AWS Requirements

Before starting the implementation, prepare an AWS account with permission to access the following services.

- AWS IoT Core
- Amazon DynamoDB
- Amazon SNS
- AWS IAM
- Amazon CloudWatch

The workshop is deployed in:

```text
US East (N. Virginia)
us-east-1
```

Using the same AWS Region throughout the workshop ensures that all resources are created in the correct location.

---

# AWS Resources

During this workshop, the following AWS resources will be created.

| Resource | Quantity |
|----------|---------:|
| AWS IoT Thing | 1 |
| X.509 Device Certificate | 1 |
| AWS IoT Policy | 1 |
| Amazon DynamoDB Table | 1 |
| Amazon SNS Topic | 1 |
| Email Subscription | 1 |
| AWS IoT Rules | 2 |
| AWS IAM Role | 1 |

---

# Network Requirements

The ESP32-S3 requires Internet connectivity through a Wi-Fi network.

The device operates in **Station (STA) Mode**, allowing it to connect to an existing wireless access point.

The network must provide:

- Internet access
- DNS resolution
- HTTPS connectivity
- MQTT over TLS (TCP Port 8883)

---

# Development Environment

The project source code is organized as a PlatformIO project.

The main directory structure is shown below.

```text
awsprj/
├── include/
├── src/
├── certificates/
├── platformio.ini
└── lib/
```

The firmware source code is implemented in C++ and compiled using PlatformIO.

---

# Required Knowledge

Readers are expected to have basic knowledge of:

- Embedded programming using C/C++
- ESP32 development
- MQTT protocol
- JSON data format
- AWS Management Console
- Basic networking concepts

No prior experience with AWS IoT Core is required because every step will be explained throughout the workshop.

---

# Before You Continue

Before moving to the next chapter, verify the following checklist.

- PlatformIO has been installed successfully.
- ESP32-S3 can be detected by the computer.
- AWS account is available.
- Internet connection is working.
- Required hardware is prepared.

{{% notice tip %}}
After completing all prerequisites, continue to the next chapter to create the AWS IoT resources required for the Smart Home system.
{{% /notice %}}

**Next:** [AWS IoT Configuration](../5.4-aws-iot-configuration/)