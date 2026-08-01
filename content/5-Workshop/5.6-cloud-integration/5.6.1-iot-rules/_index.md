---
title: "Create AWS IoT Rules"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.6.1 </b> "
---

{{% notice info %}}
In this section, you will create AWS IoT Rules that process telemetry messages published by the ESP32-S3. One rule stores telemetry data, while another detects door-open events and triggers a notification workflow.
{{% /notice %}}

# Overview

AWS IoT Rules Engine allows MQTT messages to be processed without deploying a dedicated backend application.

Each rule contains an AWS IoT SQL statement that selects messages from an MQTT topic. When a message satisfies the SQL statement, AWS IoT Core invokes one or more configured actions.

The Smart Home system uses two rules:

| Rule | Purpose |
|---|---|
| Telemetry storage rule | Processes every telemetry message for storage in Amazon DynamoDB. |
| Door alert rule | Processes only messages where `door_open` is `true`. |

Both rules listen to:

```text
smarthome/esp32-home-01/telemetry
```

---

# Objectives

After completing this section, you will be able to:

- Open the AWS IoT Rules interface.
- Create an IoT SQL rule.
- Select telemetry messages from an MQTT topic.
- Add a condition to filter door-open events.
- Configure rule actions.
- Associate an AWS IAM role with an action.
- Verify that the rules are enabled.

---

# Estimated Time

**Approximately 10–15 minutes**

---

# Rule Processing Flow

The rule processing workflow is:

```text
ESP32-S3 publishes telemetry
             ↓
AWS IoT Message Broker
             ↓
AWS IoT Rules Engine
             ↓
Evaluate AWS IoT SQL
       ┌─────┴─────┐
       │           │
       ▼           ▼
Telemetry Rule   Door Alert Rule
       │           │
       ▼           ▼
DynamoDB          SNS
```

AWS IoT Rules Engine processes the message independently for each matching rule.

---

# Step 1 – Open AWS IoT Rules

Sign in to the AWS Management Console and open **AWS IoT Core**.

From the left navigation pane, choose:

```text
Message routing
→ Rules
```

The Rules page displays all configured AWS IoT Rules.

![AWS IoT Rules dashboard](/fcj-workshop-template/images/workshop/5.6.1/iot-rules-dashboard.png)

Choose **Create rule**.

---

# Step 2 – Configure the Telemetry Rule

Enter a descriptive rule name.

Example:

```text
StoreSmartHomeTelemetry
```

Optionally add a description:

```text
Stores ESP32-S3 Smart Home telemetry in Amazon DynamoDB.
```

{{% notice note %}}
AWS IoT Rule names cannot contain spaces. Use letters, numbers, hyphens, or underscores according to the restrictions shown in the AWS Console.
{{% /notice %}}

---

# Step 3 – Define the Telemetry SQL Statement

Use the following SQL statement:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

This query selects every attribute from every message published to the telemetry topic.

The `*` wildcard includes fields such as:

```text
device_id
timestamp
dht_valid
temperature
humidity
light
door_open
relay_on
```

Select a supported AWS IoT SQL version from the console. Unless the project requires an older version, use the current recommended version shown by AWS.

---

# Step 4 – Configure the Telemetry Action

Choose an action that writes the message to Amazon DynamoDB.

Depending on the AWS Console interface, the action may be displayed as:

```text
DynamoDB
```

or:

```text
Split message into multiple columns of a DynamoDB table
```

Configure the action using the DynamoDB table created for telemetry storage.

The exact action configuration is completed in the next section.

![Telemetry storage rule](/fcj-workshop-template/images/workshop/5.6.1/telemetry-rule.png)

---

# Step 5 – Configure the IAM Role

AWS IoT Rules Engine needs permission to write to DynamoDB.

Choose an existing role or create a new role through the AWS Console.

The role should allow an action such as:

```text
dynamodb:PutItem
```

The resource should point to the telemetry table ARN.

Example:

```text
arn:aws:dynamodb:us-east-1:ACCOUNT_ID:table/SmartHomeTelemetry
```

The trust relationship must allow AWS IoT to assume the role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

{{% notice warning %}}
Do not grant administrator access to the rule role. Grant only the permissions required by the configured IoT Rule action.
{{% /notice %}}

---

# Step 6 – Create the Telemetry Rule

Review the configuration:

| Setting | Value |
|---|---|
| Rule name | `StoreSmartHomeTelemetry` |
| MQTT topic | `smarthome/esp32-home-01/telemetry` |
| SQL condition | None |
| Action | Write to Amazon DynamoDB |
| IAM role | IoT Rule DynamoDB role |
| Status | Enabled |

Choose **Create**.

The rule should appear in the Rules dashboard.

---

# Step 7 – Create the Door Alert Rule

Create another rule.

Example name:

```text
SmartHomeDoorAlert
```

Description:

```text
Publishes an Amazon SNS notification when the door is open.
```

---

# Step 8 – Define the Door Alert SQL Statement

Use the following statement:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

This query filters incoming telemetry.

The rule is triggered only when the payload contains:

```json
{
  "door_open": true
}
```

Messages containing:

```json
{
  "door_open": false
}
```

do not trigger the SNS action.

---

# Step 9 – Configure the SNS Action

Choose the action that publishes a message to Amazon SNS.

Select the SNS topic used for Smart Home door alerts.

Example topic name:

```text
SmartHomeDoorAlert
```

The SNS topic and email subscription are configured in later sections.

AWS IoT Rules Engine also requires an IAM role with permission to perform:

```text
sns:Publish
```

Example resource:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

![Door alert rule](/fcj-workshop-template/images/workshop/5.6.1/door-alert-rule.png)

---

# Step 10 – Create the Door Alert Rule

Review the configuration:

| Setting | Value |
|---|---|
| Rule name | `SmartHomeDoorAlert` |
| MQTT topic | `smarthome/esp32-home-01/telemetry` |
| SQL condition | `door_open = true` |
| Action | Publish to Amazon SNS |
| IAM role | IoT Rule SNS role |
| Status | Enabled |

Choose **Create**.

---

# Step 11 – Verify the Rules

Return to:

```text
AWS IoT Core
→ Message routing
→ Rules
```

Verify that both rules are present and enabled.

Expected rules:

```text
StoreSmartHomeTelemetry
SmartHomeDoorAlert
```

Open each rule and verify:

- The MQTT topic is correct.
- The SQL statement is valid.
- The action points to the correct AWS resource.
- The IAM role is configured.
- The rule status is enabled.

---

# Understanding AWS IoT SQL

The telemetry rule uses:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
```

This means:

| SQL part | Meaning |
|---|---|
| `SELECT *` | Select every field from the JSON payload. |
| `FROM` | Read messages from the specified MQTT topic. |
| Topic in quotes | Exact MQTT source topic. |

The door alert rule adds:

```sql
WHERE door_open = true
```

This means that only matching messages trigger the action.

---

# Test the Rule Input

Open the AWS IoT MQTT Test Client and publish a test message to:

```text
smarthome/esp32-home-01/telemetry
```

Example telemetry:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "dht_valid": true,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

This message should match:

- The telemetry storage rule.
- The door alert rule.

Change the value to:

```json
{
  "door_open": false
}
```

The message should match only the telemetry storage rule.

---

# Monitoring Rule Execution

Rule activity can be reviewed through AWS IoT and Amazon CloudWatch metrics.

Useful indicators include:

- Rule matched.
- Action succeeded.
- Action failed.
- Authorization error.
- Target service error.

When no data reaches the destination, verify:

1. The ESP32-S3 publishes to the correct topic.
2. The SQL topic matches exactly.
3. The rule is enabled.
4. The IAM role has the required permission.
5. The target resource exists in `us-east-1`.
6. The JSON payload contains the expected field names.

---

# Common Errors

## Rule is not triggered

Possible causes:

- Topic name mismatch.
- Rule is disabled.
- ESP32-S3 is not publishing telemetry.
- SQL statement contains a syntax error.

---

## Door alert rule triggers incorrectly

Verify that the payload contains a Boolean:

```json
{
  "door_open": true
}
```

Avoid publishing the value as a string:

```json
{
  "door_open": "true"
}
```

A JSON Boolean and a JSON string are different data types.

---

## Action fails

Possible causes:

- IAM role is missing.
- IAM permission is incorrect.
- DynamoDB table or SNS topic does not exist.
- The resource is in another Region.
- The action configuration references an incorrect ARN.

---

# Expected Result

After completing this section:

- The telemetry storage rule is created and enabled.
- The door alert rule is created and enabled.
- Both rules read from the Smart Home telemetry topic.
- The telemetry rule processes every message.
- The door alert rule processes only `door_open = true`.
- IAM roles authorize the configured rule actions.
- Test telemetry can trigger the expected rules.

{{% notice tip %}}
The next section creates and verifies the Amazon DynamoDB table used to store telemetry records processed by the telemetry rule.
{{% /notice %}}

**Next:** [5.6.2 Store Telemetry in Amazon DynamoDB](../5.6.2-dynamodb/)