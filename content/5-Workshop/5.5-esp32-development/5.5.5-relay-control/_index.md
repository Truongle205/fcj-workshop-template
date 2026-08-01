---
title: "Remote Relay Control"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5.5 </b> "
---

{{% notice info %}}
In this section, the ESP32-S3 receives MQTT commands from AWS IoT Core and controls a relay remotely. The firmware also reports the current relay state to AWS IoT Device Shadow.
{{% /notice %}}

# Overview

The previous section implemented one-way telemetry communication from the ESP32-S3 to AWS IoT Core.

This section adds bidirectional communication.

AWS IoT Core can send a command to the ESP32-S3 through an MQTT topic. After receiving the command, the device changes the relay output and reports the latest relay state to AWS IoT Device Shadow.

The control flow is:

```text
AWS IoT Core
      ↓
MQTT command topic
      ↓
ESP32-S3 MQTT callback
      ↓
Relay state updated
      ↓
Reported state published
      ↓
AWS IoT Device Shadow
```

The implementation is divided into three modules:

```text
mqtt_manager.cpp
        ↓
relay.cpp
        ↓
shadow_manager.cpp
```

---

# Objectives

After completing this section, you will be able to:

- Initialize the relay output.
- Handle active-low relay modules.
- Subscribe to the MQTT command topic.
- Receive `ON` and `OFF` commands.
- Update the physical relay state.
- Report the current relay state to AWS IoT Device Shadow.
- Process Device Shadow delta messages.
- Verify remote relay control using the AWS IoT MQTT Test Client.

---

# Estimated Time

**Approximately 10–15 minutes**

---

# Step 1 – Configure the Relay Pin

The relay GPIO is defined in:

```text
include/config.h
```

Example:

```cpp
constexpr uint8_t RELAY_PIN = 5;

constexpr uint8_t RELAY_ON_LEVEL = LOW;
constexpr uint8_t RELAY_OFF_LEVEL = HIGH;
```

The project uses an active-low relay module.

For an active-low relay:

```text
GPIO LOW  → Relay ON
GPIO HIGH → Relay OFF
```

{{% notice warning %}}
Relay modules may operate as active-low or active-high devices. Verify the relay module specification before connecting an external load.
{{% /notice %}}

---

# Step 2 – Define the Relay Interface

The relay functions are declared in:

```text
include/relay.h
```

```cpp
#pragma once

void initRelay();
void setRelayState(bool state);
bool getRelayState();

void setRelay(bool enabled);
void toggleRelay();
bool isRelayOn();
```

The interface provides functions for:

- Initializing the output pin.
- Setting a specific relay state.
- Reading the current relay state.
- Toggling the relay state.

---

# Step 3 – Initialize the Relay

The relay implementation is located in:

```text
src/relay.cpp
```

The project stores the current relay state internally:

```cpp
namespace
{
bool relayState = false;
}

constexpr bool RELAY_ACTIVE_LOW = true;
```

The initialization function configures the GPIO and ensures the relay starts in the OFF state:

```cpp
void initRelay()
{
    pinMode(RELAY_PIN, OUTPUT);

    digitalWrite(
        RELAY_PIN,
        RELAY_OFF_LEVEL
    );

    relayState = false;

    Serial.println(
        "Relay initialized: OFF"
    );
}
```

Initializing the relay as OFF prevents the connected appliance from being activated unexpectedly during device startup.

---

# Step 4 – Update the Relay State

The `setRelayState()` function supports both active-low and active-high relay modules:

```cpp
void setRelayState(bool state)
{
    relayState = state;

    if (RELAY_ACTIVE_LOW)
    {
        digitalWrite(
            RELAY_PIN,
            state ? LOW : HIGH
        );
    }
    else
    {
        digitalWrite(
            RELAY_PIN,
            state ? HIGH : LOW
        );
    }

    Serial.print("Relay: ");
    Serial.println(
        state ? "ON" : "OFF"
    );
}
```

The Boolean state represents the logical device state:

```text
true  → Relay ON
false → Relay OFF
```

The function translates the logical state into the correct electrical GPIO level.

---

# Step 5 – Configure the Command Topic

The command topic is defined in `include/secrets.h`:

```cpp
constexpr const char *AWS_TOPIC_COMMAND =
    "smarthome/esp32-home-01/command";
```

The ESP32-S3 subscribes to this topic after connecting successfully to AWS IoT Core:

```cpp
bool commandOk =
    mqttClient.subscribe(
        AWS_TOPIC_COMMAND
    );

Serial.print("Command topic: ");
Serial.println(AWS_TOPIC_COMMAND);

Serial.print("Command subscribe: ");
Serial.println(
    commandOk ? "OK" : "FAILED"
);
```

AWS IoT Policy must allow:

```text
iot:Subscribe
iot:Receive
```

for this topic.

---

# Step 6 – Receive the MQTT Command

Incoming MQTT messages are processed in `mqttCallback()`:

```cpp
void mqttCallback(
    char *topic,
    byte *payload,
    unsigned int length
)
{
    Serial.print(
        "Message arrived on topic: "
    );

    Serial.println(topic);

    String message;

    for (
        unsigned int i = 0;
        i < length;
        i++
    )
    {
        message +=
            static_cast<char>(
                payload[i]
            );
    }

    message.trim();

    Serial.print("Payload: [");
    Serial.print(message);
    Serial.println("]");
```

The callback converts the raw MQTT payload into an Arduino `String`.

`trim()` removes leading and trailing whitespace so that values such as `ON`, `ON\n`, and ` OFF ` can be processed consistently.

---

# Step 7 – Process ON and OFF Commands

The callback checks whether the message was received on the command topic:

```cpp
if (
    strcmp(
        topic,
        AWS_TOPIC_COMMAND
    ) == 0
)
{
    if (
        message.equalsIgnoreCase("ON")
    )
    {
        setRelayState(true);
        publishShadowReportedState();
    }
    else if (
        message.equalsIgnoreCase("OFF")
    )
    {
        setRelayState(false);
        publishShadowReportedState();
    }
    else
    {
        Serial.println(
            "Unknown relay command"
        );
    }
}
```

The use of `equalsIgnoreCase()` allows both uppercase and lowercase command values:

```text
ON
on
OFF
off
```

After changing the relay output, the firmware publishes the updated state to AWS IoT Device Shadow.

---

# Step 8 – Publish the Reported Relay State

The reported-state logic is implemented in:

```text
src/shadow_manager.cpp
```

The firmware creates the following Shadow payload:

```json
{
  "state": {
    "reported": {
      "relay_on": true
    }
  }
}
```

The project code is:

```cpp
void publishShadowReportedState()
{
    JsonDocument doc;

    doc["state"]["reported"]
       ["relay_on"] =
        getRelayState();

    char payload[256];

    size_t payloadLength =
        serializeJson(
            doc,
            payload,
            sizeof(payload)
        );

    bool result =
        mqttClient.publish(
            AWS_SHADOW_UPDATE_TOPIC,
            reinterpret_cast<
                const uint8_t *
            >(payload),
            payloadLength,
            false
        );

    Serial.print(
        "Shadow reported payload: "
    );

    Serial.println(payload);

    Serial.print(
        "Shadow publish result: "
    );

    Serial.println(
        result
            ? "SUCCESS"
            : "FAILED"
    );
}
```

The Shadow update topic is:

```text
$aws/things/esp32-home-01/shadow/update
```

The `reported` value represents the actual relay state currently applied by the ESP32-S3.

---

# Step 9 – Subscribe to Device Shadow Topics

After the MQTT connection is established, the project subscribes to three Device Shadow topics:

```cpp
bool shadowDeltaOk =
    mqttClient.subscribe(
        AWS_SHADOW_DELTA_TOPIC
    );

bool shadowAcceptedOk =
    mqttClient.subscribe(
        AWS_SHADOW_UPDATE_ACCEPTED_TOPIC
    );

bool shadowRejectedOk =
    mqttClient.subscribe(
        AWS_SHADOW_UPDATE_REJECTED_TOPIC
    );
```

The topic definitions are:

```cpp
#define AWS_SHADOW_UPDATE_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update"

#define AWS_SHADOW_DELTA_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/delta"

#define AWS_SHADOW_UPDATE_ACCEPTED_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/accepted"

#define AWS_SHADOW_UPDATE_REJECTED_TOPIC \
    "$aws/things/" AWS_THING_NAME \
    "/shadow/update/rejected"
```

These topics provide:

| Topic | Purpose |
|---|---|
| `/shadow/update` | Publishes desired or reported state. |
| `/shadow/update/delta` | Delivers differences between desired and reported states. |
| `/shadow/update/accepted` | Confirms that a Shadow update was accepted. |
| `/shadow/update/rejected` | Reports a rejected Shadow update. |

---

# Step 10 – Process Shadow Delta Messages

The MQTT callback first passes every incoming message to the Shadow handler:

```cpp
if (
    handleShadowMessage(
        topic,
        payload,
        length
    )
)
{
    return;
}
```

When the topic matches the Shadow delta topic, the payload is parsed:

```cpp
if (
    strcmp(
        topic,
        AWS_SHADOW_DELTA_TOPIC
    ) == 0
)
{
    JsonDocument doc;

    DeserializationError error =
        deserializeJson(
            doc,
            payload,
            length
        );

    if (error)
    {
        Serial.print(
            "Shadow delta JSON error: "
        );

        Serial.println(
            error.c_str()
        );

        return true;
    }
```

The expected delta payload is:

```json
{
  "state": {
    "relay_on": true
  }
}
```

The desired state is then applied:

```cpp
if (
    doc["state"]["relay_on"]
        .is<bool>()
)
{
    bool desiredRelay =
        doc["state"]["relay_on"]
            .as<bool>();

    setRelayState(
        desiredRelay
    );

    publishShadowReportedState();
}
```

After the relay changes, the ESP32-S3 reports the actual state back to the Shadow.

This synchronizes:

```text
desired.relay_on
        =
reported.relay_on
```

---

# Step 11 – Publish a Direct MQTT Command

Open:

```text
AWS IoT Core
→ Test
→ MQTT test client
→ Publish to a topic
```

Use the topic:

```text
smarthome/esp32-home-01/command
```

To turn the relay on, publish:

```text
ON
```

To turn it off, publish:

```text
OFF
```

![Publish relay command from AWS IoT Core](/fcj-workshop-template/images/workshop/5.5.5/publish-relay-command.png)

{{% notice note %}}
The current firmware expects a plain text command, not a JSON payload, on the direct command topic.
{{% /notice %}}

---

# Step 12 – Verify the ESP32-S3 Response

Open the PlatformIO Serial Monitor:

```bash
pio device monitor
```

Expected output for an ON command:

```text
Message arrived on topic:
smarthome/esp32-home-01/command

Payload: [ON]
Relay: ON
Shadow reported payload:
{"state":{"reported":{"relay_on":true}}}
Shadow publish result: SUCCESS
```

Expected output for an OFF command:

```text
Payload: [OFF]
Relay: OFF
Shadow reported payload:
{"state":{"reported":{"relay_on":false}}}
Shadow publish result: SUCCESS
```

![ESP32-S3 receives relay command](/fcj-workshop-template/images/workshop/5.5.5/relay-command-received.png)

---

# Step 13 – Control the Relay using Device Shadow

The relay can also be controlled by changing the Shadow desired state.

Publish the following payload to:

```text
$aws/things/esp32-home-01/shadow/update
```

Payload:

```json
{
  "state": {
    "desired": {
      "relay_on": true
    }
  }
}
```

AWS IoT Device Shadow generates a delta message when the desired state differs from the reported state.

The ESP32-S3 receives:

```json
{
  "state": {
    "relay_on": true
  }
}
```

It then:

1. Turns the relay ON.
2. Publishes `reported.relay_on = true`.
3. Receives an update accepted response.
4. Removes the desired/reported difference.

---

# Direct Command and Device Shadow Comparison

| Method | Payload | Purpose |
|---|---|---|
| MQTT command topic | `ON` or `OFF` | Immediate direct device command. |
| Device Shadow | JSON desired state | Persistent cloud-side device state synchronization. |

A direct MQTT command is only delivered while the device is connected.

Device Shadow retains the desired state in AWS, allowing the device to synchronize after reconnecting when the full Shadow retrieval workflow is implemented.

---

# Security Permissions

The AWS IoT Policy must allow the direct command topic and the required Shadow topics.

Typical actions include:

```text
iot:Connect
iot:Publish
iot:Subscribe
iot:Receive
```

The resource types differ depending on the action:

```text
client/
topic/
topicfilter/
```

For example:

```text
topic/smarthome/esp32-home-01/command
topicfilter/smarthome/esp32-home-01/command
topic/$aws/things/esp32-home-01/shadow/update
topicfilter/$aws/things/esp32-home-01/shadow/update/delta
```

{{% notice warning %}}
Use least-privilege resource ARNs in production. A wildcard policy is acceptable only for temporary testing and troubleshooting.
{{% /notice %}}

---

# Troubleshooting

## Command subscribe fails

Verify:

- MQTT is connected.
- The topic name is correct.
- The IoT Policy allows `iot:Subscribe`.
- The policy resource uses `topicfilter/`.
- The certificate has the correct policy attached.

---

## Message is received but relay does not change

Verify:

- The command is exactly `ON` or `OFF`.
- `RELAY_PIN` matches the wiring.
- The relay is active-low or active-high as expected.
- The relay module has a suitable power supply.
- Ground is shared between ESP32-S3 and the relay module.

---

## Relay state is reversed

The relay module may use active-low logic.

Check:

```cpp
constexpr bool RELAY_ACTIVE_LOW =
    true;
```

Change the value only when the relay hardware uses active-high control.

---

## Shadow update is rejected

Verify:

- The IoT Policy allows Shadow publish, subscribe, and receive operations.
- `AWS_THING_NAME` matches the actual AWS IoT Thing.
- The Shadow topic path is correct.
- The JSON payload has valid syntax.

The rejected message is printed by:

```cpp
Serial.print(
    "Shadow update rejected: "
);
```

---

# Expected Result

After completing this section:

- ESP32-S3 subscribes to the command topic.
- AWS IoT Core can turn the relay ON and OFF.
- Commands are processed case-insensitively.
- Relay state is included in telemetry.
- The actual state is published to AWS IoT Device Shadow.
- Shadow delta messages can control the relay.
- Accepted and rejected Shadow updates are handled.

{{% notice tip %}}
The ESP32 firmware is now capable of bidirectional communication. In the next chapter, AWS IoT Rules Engine will route telemetry to Amazon DynamoDB and Amazon SNS.
{{% /notice %}}

**Next:** [5.6 Cloud Integration](../../5.6-cloud-integration/)