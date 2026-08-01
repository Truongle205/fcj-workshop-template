---
title: "Generate Device Certificate"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

{{% notice info %}}
In this section, you will generate an X.509 device certificate that enables the ESP32-S3 to authenticate securely with AWS IoT Core using mutual TLS.
{{% /notice %}}

# Overview

AWS IoT Core authenticates every device using an X.509 certificate instead of a traditional username and password.

Each certificate is unique and acts as the digital identity of an IoT device.

During the TLS handshake, AWS IoT Core verifies the certificate before allowing the ESP32-S3 to establish an MQTT connection.

---

# Objectives

After completing this section, you will be able to:

- Generate a new X.509 device certificate.
- Download the device credentials.
- Activate the certificate.
- Understand the purpose of each credential file.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Generate a New Certificate

When creating the AWS IoT Thing, select:

```text
Auto-generate a new certificate
```

AWS automatically creates:

- Device Certificate
- Public Key
- Private Key

![](/fcj-workshop-template/images/workshop/5.4.2/create-certificate.png)

Click **Create Thing** to continue.

---

# Step 2 – Certificate Generated

After the creation process completes successfully, AWS displays the generated certificate information.

The certificate status should initially appear as **Inactive** until it is activated.

![](/fcj-workshop-template/images/workshop/5.4.2/certificate-generated.png)

---

# Step 3 – Download Security Credentials

Download and securely store the following files:

- Device Certificate (`certificate.pem.crt`)
- Private Key (`private.pem.key`)
- Public Key (`public.pem.key`)
- Amazon Root CA 1 (`AmazonRootCA1.pem`)

![](/fcj-workshop-template/images/workshop/5.4.2/download-certificates.png)

{{% notice warning %}}
The private key is only available for download once. If it is lost, a new certificate must be generated.
{{% /notice %}}

---

# Step 4 – Activate the Certificate

Activate the certificate by changing its status to **Active**.

Only active certificates are allowed to authenticate with AWS IoT Core.

![](/fcj-workshop-template/images/workshop/5.4.2/activate-certificate.png)

---

# Certificate Files

The downloaded credentials serve different purposes.

| File | Purpose |
|------|---------|
| Device Certificate | Authenticates the ESP32-S3. |
| Private Key | Proves device ownership. |
| Public Key | Paired with the private key. |
| Amazon Root CA 1 | Verifies the AWS IoT server certificate. |

---

# Expected Result

After completing this section:

- The certificate has been generated.
- All credential files have been downloaded.
- The certificate status is **Active**.
- The credentials are ready for use in the ESP32 firmware.

{{% notice tip %}}
The downloaded certificate files will be embedded into the PlatformIO project in Chapter 5.5 when configuring secure MQTT communication.
{{% /notice %}}

**Next:** [Attach AWS IoT Policy](../5.4.3-attach-policy/)