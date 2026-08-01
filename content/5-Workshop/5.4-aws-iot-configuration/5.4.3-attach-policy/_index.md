---
title: "Create and Attach AWS IoT Policy"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

{{% notice info %}}
In this section, you will create an AWS IoT Policy and attach it to the device certificate. The policy defines which MQTT operations the ESP32-S3 is permitted to perform when communicating with AWS IoT Core.
{{% /notice %}}

# Overview

Authentication and authorization are two separate security mechanisms in AWS IoT Core.

The X.509 device certificate authenticates the identity of the ESP32-S3, while the AWS IoT Policy determines what actions the authenticated device is allowed to perform.

Without an attached policy, AWS IoT Core rejects all MQTT operations even if the device certificate is valid.

---

# Objectives

After completing this section, you will be able to:

- Create an AWS IoT Policy.
- Grant MQTT permissions.
- Attach the policy to the device certificate.
- Verify the policy attachment.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Create an AWS IoT Policy

Navigate to:

```
AWS IoT Core

↓

Security

↓

Policies
```

Create a new policy.

Assign an appropriate name, for example:

```text
esp32-home-policy
```

---

# Step 2 – Configure Permissions

Grant the following MQTT permissions.

| Action | Description |
|---------|-------------|
| iot:Connect | Connect to AWS IoT Core |
| iot:Publish | Publish MQTT messages |
| iot:Subscribe | Subscribe to MQTT topics |
| iot:Receive | Receive MQTT messages |

For simplicity, this workshop allows all IoT resources.

```text
Resource

*

Effect

Allow
```

{{% notice note %}}
For production systems, permissions should follow the principle of least privilege instead of using wildcard resources.
{{% /notice %}}

---

# Step 3 – Attach the Policy

Return to the certificate created in the previous section.

Choose **Attach Policy**.

![](/fcj-workshop-template/images/workshop/5.4.3/attach-iot-policy.png)

Select the newly created policy and confirm the attachment.

---

# Verification

Open the certificate details page.

Verify that:

- The policy is attached.
- Certificate status remains **Active**.
- No warning messages are displayed.

---

# Security Discussion

The device certificate proves **who the device is**.

The AWS IoT Policy defines **what the device can do**.

Both mechanisms are required before an MQTT connection can be established successfully.

---

# Expected Result

After completing this section:

- AWS IoT Policy has been created.
- Required MQTT permissions have been granted.
- The policy has been attached to the device certificate.
- The ESP32-S3 is authorized to communicate with AWS IoT Core.

{{% notice tip %}}
The next section verifies the AWS IoT configuration by using the built-in MQTT Test Client before connecting the ESP32-S3 firmware.
{{% /notice %}}

**Next:** [MQTT Test Client](../5.4.4-mqtt-test-client/)