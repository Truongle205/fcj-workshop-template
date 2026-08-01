---
title: "Week 3 Worklog"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Week 3 Worklog

## Week 3 Objectives

The objective of this week was to develop the embedded firmware for collecting sensor data and generating telemetry messages.

## Tasks Completed

| Day | Task |
|---|---|
| Monday | Implemented the DHT11 driver for temperature and humidity measurement. |
| Tuesday | Implemented the LDR sensor interface for ambient light detection. |
| Wednesday | Developed the magnetic door sensor interface for monitoring door status. |
| Thursday | Implemented relay control functions and verified relay operation. |
| Friday | Generated JSON telemetry containing sensor readings and device status for cloud communication. |

---

## Weekly Outcome

The ESP32-S3 firmware successfully collected environmental data from all sensors and generated JSON telemetry that was ready to be transmitted to AWS IoT Core.

---

## Skills Acquired

- ESP32 GPIO Programming
- DHT11 Sensor Interface
- LDR Sensor Interface
- Door Sensor Interface
- Relay Control
- JSON Data Format