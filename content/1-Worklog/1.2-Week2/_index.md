---
title: "Week 2 Worklog"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Week 2 Worklog

## Week 2 Objectives

The objective of this week was to prepare the embedded development environment and establish the initial communication between the ESP32-S3 development board and AWS IoT Core.

## Tasks Completed

| Day | Task |
|---|---|
| Monday | Installed PlatformIO, configured the ESP32-S3 development environment, and verified board connectivity. |
| Tuesday | Created the PlatformIO project and configured the required project libraries. |
| Wednesday | Implemented Wi-Fi connection and verified Internet access on the ESP32-S3. |
| Thursday | Configured Network Time Protocol (NTP) synchronization to support TLS certificate validation. |
| Friday | Imported the Root CA certificate, device certificate, and private key into the firmware project for future MQTT over TLS communication. |

---

## Weekly Outcome

The embedded development environment was successfully prepared. The ESP32-S3 was able to connect to the Wi-Fi network, synchronize the system time using NTP, and was ready for secure MQTT communication with AWS IoT Core.

---

## Skills Acquired

- PlatformIO
- ESP32-S3 Development
- Wi-Fi Programming
- Network Time Protocol (NTP)
- TLS Certificate Management
- Embedded Development Environment