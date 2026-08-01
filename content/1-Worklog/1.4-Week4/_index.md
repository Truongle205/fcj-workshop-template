---
title: "Week 4 Worklog"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4 Worklog

## Week 4 Objectives

The objective of this week was to integrate the ESP32-S3 firmware with AWS IoT Core using MQTT over TLS and implement bidirectional communication.

## Tasks Completed

| Day | Task |
|---|---|
| Monday | Implemented MQTT over TLS connection using the AWS IoT endpoint and X.509 certificate. |
| Tuesday | Published Smart Home telemetry periodically to the AWS IoT Core MQTT topic. |
| Wednesday | Implemented MQTT callback functions for receiving cloud messages. |
| Thursday | Developed remote relay control using MQTT command topics. |
| Friday | Verified bidirectional communication between the ESP32-S3 and AWS IoT Core through publish and subscribe operations. |

---

## Weekly Outcome

The ESP32-S3 successfully established a secure MQTT connection with AWS IoT Core. Telemetry messages were published periodically, and relay commands sent from AWS IoT Core were received and executed correctly.

---

## Skills Acquired

- MQTT over TLS
- MQTT Publish/Subscribe
- ESP32 MQTT Client
- Remote Relay Control
- TLS Authentication
- IoT Device Integration