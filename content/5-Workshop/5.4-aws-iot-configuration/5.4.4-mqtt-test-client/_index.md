---
title: "Verify MQTT Communication"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

{{% notice info %}}
In this section, you will use the AWS IoT MQTT Test Client to verify that AWS IoT Core can successfully publish and subscribe to MQTT topics before connecting the ESP32-S3 device.
{{% /notice %}}

# Overview

Before deploying the firmware to the ESP32-S3, it is recommended to verify that AWS IoT Core is functioning correctly.

AWS IoT Core provides a built-in **MQTT Test Client** that allows developers to publish and subscribe to MQTT topics directly from the AWS Management Console.

This tool is useful for testing topic names, payload formats, and message routing without requiring any physical device.

---

# Objectives

After completing this section, you will be able to:

- Open the AWS IoT MQTT Test Client.
- Subscribe to an MQTT topic.
- Publish a test message.
- Verify MQTT message delivery.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Open MQTT Test Client

In AWS IoT Core, navigate to:

```text
Test

↓

MQTT test client
```

The MQTT Test Client provides an interactive interface for testing MQTT communication.

![](/fcj-workshop-template/images/workshop/5.4.4/mqtt-test-client.png)

---

# Step 2 – Subscribe to a Topic

Subscribe to the telemetry topic used by the Smart Home device.

```text
smarthome/esp32-home-01/telemetry
```

Click **Subscribe**.

The topic now appears in the subscription list and is ready to receive messages.

---

# Step 3 – Publish a Test Message

Open the **Publish to a topic** tab.

Use the same topic:

```text
smarthome/esp32-home-01/telemetry
```

Publish the following sample payload.

```json
{
  "temperature": 28,
  "humidity": 65,
  "light": 420
}
```

Click **Publish**.

---

# Step 4 – Verify the Result

If the configuration is correct, the published message will immediately appear in the subscribed topic.

This confirms that:

- The MQTT topic is valid.
- AWS IoT Core is functioning correctly.
- Messages can be routed successfully.

---

# Expected Result

After completing this section:

- MQTT Test Client is operational.
- The telemetry topic is subscribed successfully.
- Test messages can be published.
- Published messages are received immediately.

{{% notice tip %}}
The MQTT Test Client is a convenient tool for debugging MQTT topics before deploying firmware to physical devices.
{{% /notice %}}

**Next:** [ESP32 Development](../../5.5-esp32-development/)