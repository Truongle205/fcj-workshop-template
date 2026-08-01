---
title: "Cloud Integration"
date: 2026-08-01
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---

{{% notice info %}}
In this chapter, AWS IoT Rules Engine processes telemetry messages received from the ESP32-S3 and routes them to Amazon DynamoDB for storage and Amazon SNS for door-open email notifications.
{{% /notice %}}

# Cloud Integration

At this stage, the ESP32-S3 can securely communicate with AWS IoT Core, publish telemetry, receive commands, and update the relay state.

The next step is to integrate AWS IoT Core with additional AWS services.

The system uses AWS IoT Rules Engine to evaluate incoming MQTT messages and trigger actions automatically.

Two main cloud workflows are implemented:

```text
Telemetry message
      ↓
AWS IoT Rules Engine
      ├── Store data in Amazon DynamoDB
      └── Publish alert to Amazon SNS
```

This event-driven approach removes the need for a dedicated application server.

---

# Objectives

After completing this chapter, you will be able to:

- Understand how AWS IoT Rules Engine processes MQTT messages.
- Create an IoT SQL rule.
- Store Smart Home telemetry in Amazon DynamoDB.
- Create an Amazon SNS topic.
- Create and confirm an email subscription.
- Send door-open notifications automatically.
- Configure AWS IAM permissions for IoT Rule actions.
- Verify the complete cloud data flow.

---

# Services Used

The following AWS services are used in this chapter.

| AWS Service | Purpose |
|---|---|
| AWS IoT Rules Engine | Evaluates MQTT messages and invokes rule actions. |
| Amazon DynamoDB | Stores telemetry records. |
| Amazon SNS | Delivers door-open notifications. |
| AWS IAM | Grants AWS IoT Rules permission to access DynamoDB and SNS. |
| Amazon CloudWatch | Provides metrics and error information for rule execution. |

---

# MQTT Telemetry Topic

The rules in this chapter process messages published to:

```text
smarthome/esp32-home-01/telemetry
```

A typical telemetry message is:

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

---

# AWS IoT Rules Engine

AWS IoT Rules Engine evaluates messages by using an SQL-like syntax called AWS IoT SQL.

A rule contains:

- A rule name.
- An SQL statement.
- One or more actions.
- An IAM role.
- Optional error handling.

The basic telemetry query is:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

This statement selects every field from each telemetry message.

For the door alert workflow, a condition can be added:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

Only messages where `door_open` is `true` will trigger the alert action.

---

# Cloud Integration Workflow

The integration is divided into four sections:

```text
5.6.1 Create AWS IoT Rules
        ↓
5.6.2 Store Telemetry in Amazon DynamoDB
        ↓
5.6.3 Create Amazon SNS Topic
        ↓
5.6.4 Configure Email Notification
```

Each section focuses on one cloud-side responsibility.

---

# Telemetry Storage Flow

The telemetry storage workflow is:

```text
ESP32-S3
   ↓
MQTT telemetry topic
   ↓
AWS IoT Core
   ↓
Telemetry IoT Rule
   ↓
Amazon DynamoDB table
```

The DynamoDB table stores values such as:

- Device identifier.
- Timestamp.
- Temperature.
- Humidity.
- Light level.
- Door state.
- Relay state.

---

# Door Notification Flow

The notification workflow is:

```text
Door sensor detects open state
              ↓
ESP32-S3 publishes door_open = true
              ↓
AWS IoT Rules Engine evaluates the message
              ↓
Door alert rule condition matches
              ↓
Amazon SNS topic
              ↓
Confirmed email subscriber
```

The alert is generated automatically without requiring the ESP32-S3 to send email directly.

---

# AWS IAM Permissions

AWS IoT Rules Engine requires permission to invoke actions in other AWS services.

An AWS IAM role is associated with the IoT Rule.

The role grants permission to:

```text
dynamodb:PutItem
sns:Publish
```

The trust relationship must allow AWS IoT to assume the role.

Example trust principal:

```json
{
  "Principal": {
    "Service": "iot.amazonaws.com"
  }
}
```

{{% notice note %}}
AWS IAM is not part of the telemetry data path. It provides the authorization required for AWS IoT Rules Engine to invoke DynamoDB and SNS actions.
{{% /notice %}}

---

# Error Handling

AWS IoT Rules can include an error action.

An error action is triggered when the main action fails, for example:

- DynamoDB table does not exist.
- IAM permission is missing.
- SNS topic ARN is incorrect.
- Payload fields do not match the action configuration.

During troubleshooting, verify:

- Rule status is enabled.
- MQTT topic matches exactly.
- SQL statement is valid.
- IAM role contains the required permissions.
- Target resource exists in the same Region.
- Amazon CloudWatch reports no rule execution error.

---

# Estimated Time

| Section | Estimated Time |
|---|---:|
| 5.6.1 Create AWS IoT Rules | 10 minutes |
| 5.6.2 Configure Amazon DynamoDB | 10 minutes |
| 5.6.3 Configure Amazon SNS | 5 minutes |
| 5.6.4 Configure Email Notification | 5 minutes |

**Estimated completion time: approximately 30 minutes**

---

# Expected Result

After completing this chapter:

- AWS IoT Rules Engine processes telemetry messages.
- Telemetry records are stored in Amazon DynamoDB.
- Door-open events are filtered by an IoT SQL condition.
- Amazon SNS receives alert messages.
- A confirmed subscriber receives notification emails.
- IAM permissions allow each rule action to operate successfully.

{{% notice tip %}}
The next section creates the AWS IoT Rules that route telemetry messages to the required AWS services.
{{% /notice %}}

**Next:** [5.6.1 Create AWS IoT Rules](5.6.1-iot-rules/)