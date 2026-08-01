---
title: "Connect to Wi-Fi"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

{{% notice info %}}
In this section, the ESP32-S3 is configured to connect to a Wi-Fi network. A successful Wi-Fi connection is required before synchronizing the system time and establishing a secure MQTT connection with AWS IoT Core.
{{% /notice %}}

# Overview

The ESP32-S3 requires Internet connectivity to communicate with AWS IoT Core.

The firmware first initializes the Wi-Fi interface, then connects to the configured wireless network. Once connected, the device obtains an IP address from the router using DHCP.

This network connection serves as the foundation for all cloud communication implemented in the following sections.

---

# Objectives

After completing this section, you will be able to:

- Configure the Wi-Fi SSID and password.
- Connect the ESP32-S3 to a wireless network.
- Verify that an IP address has been assigned.
- Prepare the device for Internet access.

---

# Estimated Time

**Approximately 5 minutes**

---

# Step 1 – Configure Wi-Fi Credentials

Open the firmware source code and define the Wi-Fi credentials.

```cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
```

Replace the placeholders with your own Wi-Fi network information.

{{% notice warning %}}
Do not commit your Wi-Fi credentials to a public source code repository.
{{% /notice %}}

---

# Step 2 – Connect to Wi-Fi

Initialize the Wi-Fi interface and attempt to connect to the configured network.

The firmware continuously checks the connection status until the ESP32-S3 successfully joins the wireless network.

![](/images/workshop/5.5.2/wifi-connecting.png)

---

# Step 3 – Verify the Connection

Once connected, the Serial Monitor displays the assigned IP address.

Example:

```text
WiFi Connected
IP: 192.168.1.100
```

![](/images/workshop/5.5.2/wifi-connected.png)

The assigned IP address confirms that the ESP32-S3 can access the Internet.

---

# Expected Result

After completing this section:

- ESP32-S3 connects successfully to the Wi-Fi network.
- A valid IP address is assigned.
- Internet connectivity is available.

{{% notice tip %}}
The next section uses this Internet connection to synchronize the system clock and establish a secure MQTT connection with AWS IoT Core.
{{% /notice %}}

**Next:** [5.5.3 MQTT over TLS](../5.5.3-mqtt-tls/)