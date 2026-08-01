---
title: "Create a PlatformIO Project"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.5.1 </b> "
---

{{% notice info %}}
In this section, you will create a PlatformIO project for the ESP32-S3 development board. This project will serve as the foundation for implementing Wi-Fi connectivity, secure MQTT communication, telemetry publishing, and remote relay control.
{{% /notice %}}

# Overview

PlatformIO is an embedded development ecosystem that supports multiple microcontroller platforms, including the ESP32 series.

Compared with the Arduino IDE, PlatformIO provides a more structured project layout, built-in library management, dependency handling, and seamless integration with Visual Studio Code.

Throughout this workshop, all firmware development will be performed using PlatformIO.

---

# Objectives

After completing this section, you will be able to:

- Install PlatformIO.
- Create a new ESP32-S3 project.
- Configure the project environment.
- Understand the project directory structure.
- Build and upload firmware to the ESP32-S3.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Open Visual Studio Code

Launch **Visual Studio Code** with the PlatformIO extension installed.

From the left navigation bar, select the **PlatformIO** icon.

---

# Step 2 – Create a New Project

Create a new PlatformIO project with the following configuration.

| Setting | Value |
|----------|-------|
| Project Name | awsprj |
| Board | ESP32-S3 Dev Module |
| Framework | Arduino |

Click **Finish** and wait for PlatformIO to generate the project.

![](/fcj-workshop-template/images/workshop/5.5.1/platformio-project.png)

---

# Step 3 – Review the Project Structure

After the project has been created, PlatformIO generates the following directory structure.

```text
awsprj/
├── include/
├── lib/
├── src/
│   └── main.cpp
├── test/
├── platformio.ini
└── .pio/
```

Each directory has a specific purpose.

| Directory | Description |
|------------|-------------|
| include | Header files |
| lib | Custom libraries |
| src | Application source code |
| test | Unit tests |
| platformio.ini | Project configuration |

---

# Step 4 – Configure platformio.ini

Open **platformio.ini** and verify that the correct board and framework are selected.

Example configuration:

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
```

Additional libraries will be installed in later sections.

---

# Step 5 – Build the Project

Click the **Build** button in the PlatformIO toolbar.

If the configuration is correct, PlatformIO compiles the project successfully without any errors.

---

# Step 6 – Upload Firmware

Connect the ESP32-S3 development board using a USB cable.

Click **Upload** to flash the firmware.

PlatformIO automatically compiles and uploads the project to the board.

---

# Expected Result

After completing this section:

- PlatformIO project has been created.
- ESP32-S3 board is configured correctly.
- The project builds successfully.
- Firmware uploads successfully.

{{% notice tip %}}
The PlatformIO project created in this section will be used throughout the remaining chapters of this workshop.
{{% /notice %}}

**Next:** [5.5.2 Connect to Wi-Fi](../5.5.2-wifi/)