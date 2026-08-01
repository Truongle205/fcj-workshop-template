---
title: "Create AWS IoT Thing"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

{{% notice info %}}
In this section, you will create an AWS IoT Thing representing the ESP32-S3 device. During the creation process, AWS also generates an X.509 device certificate that will later be used for mutual TLS authentication.
{{% /notice %}}

# Overview

AWS IoT Core identifies every physical IoT device as an **AWS IoT Thing**.

A Thing is a logical representation of a device inside AWS IoT Core. It stores device identity, certificates, policies, and other metadata required for secure communication.

In this workshop, the ESP32-S3 development board will be registered as a Thing before connecting to AWS IoT Core.

---

# Objectives

After completing this section, you will:

- Access AWS IoT Core.
- Create an AWS IoT Thing.
- Generate an X.509 device certificate.
- Download security credentials.
- Activate the certificate.
- Attach the certificate to the Thing.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Open AWS IoT Core

Sign in to the AWS Management Console.

From the AWS Console home page, search for **IoT Core**.

![AWS Console](/images/workshop/5.4.1/aws-console-home.png)

Open the AWS IoT Core service.

---

# Step 2 – Navigate to Manage

After opening AWS IoT Core, select **Manage** from the left navigation pane.

Choose **All devices** → **Things**.

![IoT Dashboard](/images/workshop/5.4.1/iot-core-dashboard.png)

Click **Create Thing**.

---

# Step 3 – Create a Thing

Choose:

- Create a single Thing

Click **Next**.

![Create Thing](/images/workshop/5.4.1/create-thing.png)

---

# Step 4 – Configure the Thing

Enter the Thing name.

Example:

```text
esp32-home-01
```

Leave all remaining settings as default.

![Thing Name](/images/workshop/5.4.1/thing-name.png)

Click **Next**.

---

# Step 5 – Generate Device Certificate

Select:

```
Auto-generate a new certificate
```

AWS will automatically create:

- Device Certificate
- Public Key
- Private Key

Click **Create Thing**.

![Create Certificate](/images/workshop/5.4.1/create-certificate.png)

---

# Step 6 – Download Certificates

After the Thing is created, AWS displays the generated security credentials.

Download the following files.

- Device Certificate
- Private Key
- Public Key
- Amazon Root CA 1

These files will later be copied into the PlatformIO project.

![Download Certificates](/images/workshop/5.4.1/download-certificates.png)

{{% notice warning %}}
The private key can only be downloaded once. Store it securely before leaving this page.
{{% /notice %}}

---

# Step 7 – Activate Certificate

Before the device can connect to AWS IoT Core, activate the generated certificate.

Verify that the certificate status changes to:

```text
Active
```

![Activate Certificate](/images/workshop/5.4.1/activate-certificate.png)

---

# Step 8 – Attach Certificate

Attach the certificate to the newly created Thing.

![Attach Thing](/images/workshop/5.4.1/attach-thing.png)

After the attachment is complete, the certificate becomes associated with the ESP32-S3 device.

---

# Verification

Open the Thing details page.

Verify that:

- Thing Name is correct.
- Certificate is attached.
- Certificate status is **Active**.

![Thing Summary](/images/workshop/5.4.1/thing-summary.png)

---

# Expected Result

After completing this section, the following resources should exist in AWS IoT Core.

| Resource | Status |
|-----------|--------|
| AWS IoT Thing | Created |
| X.509 Device Certificate | Active |
| Public Key | Downloaded |
| Private Key | Downloaded |
| Amazon Root CA 1 | Downloaded |
| Certificate attached to Thing | Completed |

{{% notice tip %}}
Keep the downloaded certificates in a secure location. They will be used later when configuring the ESP32-S3 firmware to establish a secure MQTT connection with AWS IoT Core.
{{% /notice %}}

**Next:** [Generate Device Certificate](../5.4.2-create-certificate/)