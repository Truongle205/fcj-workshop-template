---
title: "System Testing"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---

{{% notice info %}}
In this chapter, the complete Smart Home IoT system is tested from end to end. The tests verify Wi-Fi connectivity, MQTT over TLS, telemetry publishing, cloud storage, email notification, relay control, and recovery after connection failures.
{{% /notice %}}

# System Testing

After completing the device firmware and cloud integration, the final system must be tested as a complete workflow.

Testing each component independently is useful during development, but successful component tests do not guarantee that the complete architecture operates correctly.

The end-to-end system contains the following path:

```text
Sensors
   ↓
ESP32-S3
   ↓
MQTT over TLS 1.2
   ↓
AWS IoT Core
   ↓
AWS IoT Rules Engine
   ├── Amazon DynamoDB
   └── Amazon SNS
          ↓
     Subscriber Email
```

The reverse control path is:

```text
AWS IoT MQTT Test Client
            ↓
MQTT command topic
            ↓
ESP32-S3
            ↓
Relay output
            ↓
AWS IoT Device Shadow
```

---

# Objectives

After completing this chapter, you will be able to:

- Verify ESP32-S3 startup and Wi-Fi connectivity.
- Verify NTP time synchronization.
- Verify MQTT over TLS authentication.
- Verify telemetry publishing.
- Verify telemetry storage in Amazon DynamoDB.
- Verify door-open notification through Amazon SNS.
- Verify remote relay control.
- Verify AWS IoT Device Shadow synchronization.
- Test reconnection after Wi-Fi or MQTT interruption.
- Record the final test results.

---

# Estimated Time

**Approximately 20–30 minutes**

---

# Test Environment

The test environment includes:

| Component | Purpose |
|---|---|
| ESP32-S3 | Executes the Smart Home firmware. |
| DHT11 | Provides temperature and humidity data. |
| LDR | Provides ambient light readings. |
| Door sensor | Provides open and closed states. |
| Relay module | Represents a remotely controlled device. |
| Wi-Fi network | Provides Internet connectivity. |
| AWS IoT Core | Handles MQTT communication. |
| Amazon DynamoDB | Stores telemetry records. |
| Amazon SNS | Sends email notifications. |
| PlatformIO Serial Monitor | Displays firmware logs. |

---

# Test Checklist

The complete test plan includes:

| Test ID | Test case | Expected result |
|---|---|---|
| T01 | ESP32-S3 startup | Firmware starts without resetting unexpectedly. |
| T02 | Wi-Fi connection | Device receives a valid IP address. |
| T03 | NTP synchronization | Device obtains a valid timestamp. |
| T04 | MQTT over TLS | AWS IoT connection succeeds. |
| T05 | Telemetry publish | JSON message reaches AWS IoT Core. |
| T06 | DynamoDB storage | Telemetry item appears in the table. |
| T07 | Door alert | Email is sent when `door_open = true`. |
| T08 | Relay ON command | Relay changes to ON. |
| T09 | Relay OFF command | Relay changes to OFF. |
| T10 | Shadow update | Reported state matches actual relay state. |
| T11 | Wi-Fi recovery | Device reconnects after network restoration. |
| T12 | MQTT recovery | Device reconnects after MQTT interruption. |
| T13 | Invalid DHT11 data | Other telemetry fields continue to publish. |

---

# Test 1 – Device Startup

Connect the ESP32-S3 to the computer and open the Serial Monitor:

```bash
pio device monitor
```

Reset the device.

Expected output:

```text
ESP32 Smart Home starting
Relay initialized: OFF
DHT11 initialized
Connecting WiFi...
```

Verify:

- The firmware starts correctly.
- The relay is initialized as OFF.
- The sensors are initialized.
- No continuous boot loop occurs.

---

# Test 2 – Wi-Fi Connection

Wait for the ESP32-S3 to connect to the configured access point.

Expected output:

```text
WiFi Connected
IP: 172.20.10.2
```

Verify:

- `WiFi.status()` reaches `WL_CONNECTED`.
- A valid local IP address is assigned.
- DNS resolution is available.
- The device can reach the Internet.

---

# Test 3 – NTP Time Synchronization

After Wi-Fi is connected, the firmware synchronizes time.

Expected output:

```text
Waiting for NTP...
Current UTC: 2026-07-24 09:54:43
```

Verify:

- The timestamp is valid.
- The displayed date is not the ESP32 default epoch.
- Time synchronization completes before MQTT connection.

{{% notice note %}}
An incorrect system time can cause TLS certificate validation to fail even when the certificate and private key are correct.
{{% /notice %}}

---

# Test 4 – MQTT over TLS Connection

The firmware loads the Root CA, device certificate, and private key before connecting to AWS IoT Core.

Expected output:

```text
Connecting MQTT...
AWS IoT connected
Command subscribe: OK
MQTT connected successfully
```

Verify:

- The AWS IoT endpoint is correct.
- MQTT port `8883` is used.
- TLS authentication succeeds.
- The command topic is subscribed successfully.
- Shadow subscriptions succeed when Device Shadow is enabled.

---

# Test 5 – Telemetry Publishing

Open:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Subscribe to:

```text
smarthome/esp32-home-01/telemetry
```

Wait for the ESP32-S3 to publish a telemetry message.

Expected payload:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "dht_valid": true,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": false,
  "relay_on": false
}
```

Verify:

- The topic name is correct.
- JSON syntax is valid.
- `device_id` is present.
- `timestamp` is valid.
- Sensor and device-state fields are present.
- Messages arrive at the configured interval.

The existing telemetry screenshots may be reused here:

```md
![ESP32-S3 publishes telemetry](/fcj-workshop-template/images/workshop/5.5.4/telemetry-published.png)

![AWS IoT Core receives telemetry](/fcj-workshop-template/images/workshop/5.5.4/telemetry-received.png)
```

---

# Test 6 – DynamoDB Storage

Open:

```text
Amazon DynamoDB
→ Tables
→ SmartHomeTelemetry
→ Explore table items
```

Wait for telemetry records to appear.

Verify:

- New items are inserted automatically.
- `device_id` is stored correctly.
- `timestamp` is unique for each record.
- Sensor fields match the MQTT payload.
- Door and relay states are stored correctly.

![Telemetry records in Amazon DynamoDB](/fcj-workshop-template/images/workshop/5.6.2/dynamodb-items.png)

The expected path is:

```text
ESP32-S3
→ AWS IoT Core
→ AWS IoT Rule
→ Amazon DynamoDB
```

---

# Test 7 – Door-Open Notification

Publish telemetry with:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "door_open": true,
  "relay_on": false
}
```

Alternatively, open the physical door sensor.

Verify:

- The Door Alert Rule matches the message.
- Amazon SNS receives the notification.
- The confirmed subscriber receives an email.
- The email contains the expected telemetry information.

![Door-open email notification](/fcj-workshop-template/images/workshop/5.6.4/sns-email-notification.png)

Then publish:

```json
{
  "door_open": false
}
```

Verify that this message does not trigger the Door Alert Rule.

---

# Test 8 – Relay ON Command

Open the AWS IoT MQTT Test Client.

Publish to:

```text
smarthome/esp32-home-01/command
```

Payload:

```text
ON
```

Verify the Serial Monitor:

```text
Message arrived on topic:
smarthome/esp32-home-01/command

Payload: [ON]
Relay: ON
```

Verify physically:

- The relay indicator changes.
- The connected test load changes state.
- Telemetry later reports `"relay_on": true`.

---

# Test 9 – Relay OFF Command

Publish:

```text
OFF
```

Expected Serial output:

```text
Payload: [OFF]
Relay: OFF
```

Verify:

- The relay returns to OFF.
- Telemetry reports `"relay_on": false`.
- The reported Shadow state is updated.

The existing command screenshots may be reused:

```md
![Publish a relay command](/fcj-workshop-template/images/workshop/5.5.5/publish-relay-command.png)

![ESP32-S3 receives the relay command](/fcj-workshop-template/images/workshop/5.5.5/relay-command-received.png)
```

---

# Test 10 – Device Shadow Synchronization

Publish the following payload to:

```text
$aws/things/esp32-home-01/shadow/update
```

```json
{
  "state": {
    "desired": {
      "relay_on": true
    }
  }
}
```

Verify:

1. AWS IoT Device Shadow creates a delta.
2. ESP32-S3 receives the delta message.
3. The relay changes to ON.
4. ESP32-S3 publishes:

```json
{
  "state": {
    "reported": {
      "relay_on": true
    }
  }
}
```

5. Desired and reported states become consistent.

Repeat the test with:

```json
{
  "state": {
    "desired": {
      "relay_on": false
    }
  }
}
```

---

# Test 11 – Wi-Fi Recovery

Disconnect the Wi-Fi access point or disable the hotspot temporarily.

Expected behavior:

- MQTT connection is lost.
- Telemetry publishing stops.
- Firmware continues running.
- The device attempts to restore connectivity.

Restore the network.

Verify:

- ESP32-S3 reconnects to Wi-Fi.
- MQTT connection is re-established.
- MQTT topics are subscribed again.
- Telemetry publishing resumes.

{{% notice warning %}}
Do not create a tight reconnect loop. Repeated TLS handshakes without delay can increase memory pressure and make connection failures harder to diagnose.
{{% /notice %}}

---

# Test 12 – MQTT Recovery

Keep Wi-Fi available but interrupt the MQTT connection.

This can be simulated by:

- Temporarily using an incorrect endpoint.
- Temporarily deactivating the certificate.
- Disconnecting the network for a short period.
- Restarting the access point.

After restoring the valid configuration, verify:

```text
Attempting MQTT reconnect...
AWS IoT connected
Command subscribe: OK
```

The firmware should close the old secure socket before attempting a new TLS connection.

---

# Test 13 – Invalid DHT11 Reading

Disconnect the DHT11 or test the system without the sensor attached.

Expected Serial output:

```text
Failed to read DHT11
Telemetry warning: invalid DHT11 data
```

Expected telemetry:

```json
{
  "dht_valid": false,
  "temperature": null,
  "humidity": null,
  "light": 870,
  "door_open": false,
  "relay_on": false
}
```

Verify:

- The firmware does not crash.
- MQTT remains connected.
- LDR, door, and relay states continue to publish.
- Invalid temperature and humidity are not stored as valid measurements.

---

# Test Result Summary

Record the final results in a table.

| Test ID | Test case | Result | Notes |
|---|---|---|---|
| T01 | Device startup | Pass | Firmware initialized normally. |
| T02 | Wi-Fi connection | Pass | Valid IP address assigned. |
| T03 | NTP synchronization | Pass | Valid time obtained. |
| T04 | MQTT over TLS | Pass | AWS IoT connected successfully. |
| T05 | Telemetry publish | Pass | JSON received by MQTT Test Client. |
| T06 | DynamoDB storage | Pass | Telemetry items stored. |
| T07 | Door alert | Pass | Email notification received. |
| T08 | Relay ON | Pass | Relay turned on remotely. |
| T09 | Relay OFF | Pass | Relay turned off remotely. |
| T10 | Device Shadow | Pass | Desired and reported states synchronized. |
| T11 | Wi-Fi recovery | Pass | Connection recovered after restoration. |
| T12 | MQTT recovery | Pass | MQTT reconnected and resubscribed. |
| T13 | Invalid DHT11 | Pass | Remaining telemetry continued. |

{{% notice warning %}}
Only mark a test as `Pass` after reproducing the expected result. Replace the sample values in the table with the actual results obtained during testing.
{{% /notice %}}

---

# Acceptance Criteria

The Smart Home system is considered operational when:

- ESP32-S3 connects to Wi-Fi reliably.
- NTP synchronization completes successfully.
- MQTT over TLS authentication succeeds.
- Telemetry reaches AWS IoT Core.
- DynamoDB stores telemetry automatically.
- Door-open events generate email notifications.
- Relay commands work in both directions.
- Device Shadow reports the actual relay state.
- The firmware recovers from temporary connection failures.
- Invalid sensor data does not stop the system.

---

# Known Limitations

The current implementation has several limitations:

- Door alerts may repeat while the door remains open.
- DHT11 provides limited accuracy and update frequency.
- Direct command messages use plain text `ON` and `OFF`.
- No local persistent queue is implemented for telemetry during outages.
- Device Shadow recovery depends on the implemented Shadow subscription and retrieval behavior.
- The prototype is not intended to switch high-voltage loads without appropriate electrical protection.

---

# Expected Result

After completing this chapter:

- The complete Smart Home workflow has been verified.
- Device-to-cloud telemetry works correctly.
- Cloud-to-device relay control works correctly.
- Cloud storage and email notification operate as expected.
- Temporary network failures can be recovered.
- Test evidence is available for the final report.

{{% notice tip %}}
The final chapter removes the AWS resources created during the workshop to prevent unnecessary charges and reduce unused cloud resources.
{{% /notice %}}

**Next:** [5.8 Cleanup](../5.8-cleanup/)