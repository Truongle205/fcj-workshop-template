---
title: "Configure Amazon SNS"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 5.6.3 </b> "
---

{{% notice info %}}
In this section, you will create an Amazon Simple Notification Service topic for door-open alerts. AWS IoT Rules Engine will publish a notification to this topic whenever telemetry contains `door_open = true`.
{{% /notice %}}

# Overview

Amazon Simple Notification Service, commonly referred to as Amazon SNS, is a fully managed messaging service used to distribute notifications to one or more subscribers.

In the Smart Home system, Amazon SNS acts as the notification component.

When the ESP32-S3 detects that the door is open, it publishes telemetry containing:

```json
{
  "door_open": true
}
```

AWS IoT Rules Engine evaluates the message and publishes an alert to the configured SNS topic.

The notification workflow is:

```text
ESP32-S3
     ↓
Telemetry message
     ↓
AWS IoT Rules Engine
     ↓
Door alert rule
     ↓
Amazon SNS topic
     ↓
Email subscription
```

Amazon SNS allows the notification logic to remain entirely in AWS Cloud. The ESP32-S3 does not need to connect directly to an email server.

---

# Objectives

After completing this section, you will be able to:

- Open the Amazon SNS console.
- Create an SNS topic.
- Understand the difference between Standard and FIFO topics.
- Configure the topic for Smart Home alerts.
- Obtain the SNS topic ARN.
- Connect the topic to an AWS IoT Rule action.
- Test message publishing.

---

# Estimated Time

**Approximately 5–10 minutes**

---

# Why Use Amazon SNS?

Amazon SNS was selected because it provides:

- Fully managed notification delivery.
- Event-driven integration with AWS IoT Rules Engine.
- Support for email subscriptions.
- No requirement to manage an SMTP server.
- Automatic message distribution to multiple subscribers.
- Pay-as-you-go pricing.

The ESP32-S3 only publishes telemetry. AWS services handle the notification workflow.

---

# Step 1 – Open Amazon SNS

Sign in to the AWS Management Console.

Search for:

```text
Simple Notification Service
```

Open **Amazon SNS**.

From the left navigation pane, choose:

```text
Topics
```

Then choose:

```text
Create topic
```

---

# Step 2 – Select the Topic Type

Select:

```text
Standard
```

A Standard SNS topic is appropriate for this workshop because it provides:

- High throughput.
- Best-effort message ordering.
- At-least-once message delivery.
- Support for email subscriptions.

A FIFO topic is not required because strict ordering and deduplication are not necessary for simple door alerts.

---

# Step 3 – Configure the Topic

Enter a descriptive topic name.

Example:

```text
SmartHomeDoorAlert
```

Optionally add a display name:

```text
Smart Home Door Alert
```

The display name may be shown by supported delivery protocols.

Leave the remaining settings at their default values unless the project requires additional encryption or access control.

Choose:

```text
Create topic
```

---

# Step 4 – Verify the SNS Topic

After creation, Amazon SNS displays the topic details page.

Verify:

- Topic name.
- Topic type.
- Topic ARN.
- Region.
- Subscription count.

![Amazon SNS topic](/images/workshop/5.6.3/sns-topic.png)

The topic ARN has the following structure:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

The ARN uniquely identifies the SNS topic and is required when configuring the AWS IoT Rule action.

{{% notice warning %}}
Do not expose the AWS account ID unnecessarily in public screenshots. The account ID can be blurred or masked before publishing the workshop.
{{% /notice %}}

---

# Step 5 – Configure the AWS IoT Rule Action

Return to:

```text
AWS IoT Core
→ Message routing
→ Rules
→ SmartHomeDoorAlert
```

Open the rule action configuration.

Choose the action that publishes a message to an SNS topic.

Select:

```text
SmartHomeDoorAlert
```

or provide the topic ARN:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

The IoT Rule uses the SQL statement:

```sql
SELECT *
FROM 'smarthome/esp32-home-01/telemetry'
WHERE door_open = true
```

When the condition matches, the selected telemetry message is published to the SNS topic.

---

# Step 6 – Configure IAM Permission

AWS IoT Rules Engine requires permission to publish to the SNS topic.

The IAM role associated with the rule must allow:

```text
sns:Publish
```

Example permission policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert"
    }
  ]
}
```

The role trust relationship must allow AWS IoT to assume the role:

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

{{% notice note %}}
The AWS IAM role grants permission to AWS IoT Rules Engine. It is not used by the ESP32-S3 and is not part of the MQTT device authentication process.
{{% /notice %}}

---

# Step 7 – Publish a Test Message from Amazon SNS

Before testing the full IoT workflow, the SNS topic can be tested directly.

On the topic details page, choose:

```text
Publish message
```

Enter a subject such as:

```text
Smart Home Door Alert Test
```

Enter a message:

```text
This is a test notification from Amazon SNS.
```

Choose:

```text
Publish message
```

At this stage, the message will only be delivered after a subscription has been created and confirmed.

The subscription process is covered in the next section.

---

# Step 8 – Test the AWS IoT Rule Input

Open:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Publish the following message to:

```text
smarthome/esp32-home-01/telemetry
```

Payload:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

Because `door_open` is `true`, the Door Alert Rule should publish the message to Amazon SNS.

Publish another message with:

```json
{
  "door_open": false
}
```

This message should not trigger the SNS action.

---

# Message Format

By default, the complete message selected by the IoT SQL statement may be published to SNS.

For example:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "temperature": 29.4,
  "humidity": 71,
  "light": 870,
  "door_open": true,
  "relay_on": false
}
```

A simpler notification message can be generated later by changing the IoT SQL statement or adding another processing component.

For this workshop, the complete telemetry payload is sufficient to demonstrate the alert workflow.

---

# Security Considerations

The SNS topic should not be publicly writable.

Only the AWS IoT Rule role should receive permission to publish to the topic.

Recommended controls include:

- Restrict `sns:Publish` to the specific topic ARN.
- Avoid wildcard permissions in production.
- Do not allow unauthenticated public publishing.
- Review the SNS access policy.
- Protect email addresses shown in screenshots.
- Remove unused subscriptions after testing.

---

# Monitoring

Amazon SNS provides metrics through Amazon CloudWatch.

Useful metrics include:

- Number of messages published.
- Number of notifications delivered.
- Number of failed notifications.
- Number of notifications filtered out.

If the rule is triggered but no notification is received, verify:

1. The SNS topic exists in `us-east-1`.
2. The IoT Rule action uses the correct topic ARN.
3. The IAM role allows `sns:Publish`.
4. The subscription status is `Confirmed`.
5. The email message is not in the spam folder.

---

# Troubleshooting

## SNS topic does not receive messages

Verify:

- Door Alert Rule is enabled.
- The SQL condition uses `door_open = true`.
- The payload field is a Boolean.
- The IoT Rule action points to the correct topic.
- The IAM role has `sns:Publish`.

---

## Test message publishes successfully but no email arrives

This is expected when no confirmed email subscription exists.

Create and confirm an email subscription in the next section.

---

## Access denied error

Check the IAM role permission policy.

The resource must match the exact topic ARN:

```text
arn:aws:sns:us-east-1:ACCOUNT_ID:SmartHomeDoorAlert
```

Also verify that the rule is using the correct IAM role.

---

## Duplicate notifications

Possible causes include:

- Multiple subscriptions using the same email address.
- Multiple IoT Rules matching the same message.
- Repeated telemetry with `door_open = true`.
- At-least-once message delivery behavior.

For a production system, additional logic can detect a door state transition instead of alerting on every telemetry message where the door remains open.

---

# Expected Result

After completing this section:

- A Standard Amazon SNS topic has been created.
- The topic ARN is available.
- The AWS IoT Door Alert Rule points to the SNS topic.
- The IoT Rule role can perform `sns:Publish`.
- Test messages can be published to the topic.
- Door-open telemetry can trigger the SNS workflow.

{{% notice tip %}}
The next section creates and confirms an email subscription so that notifications published to the SNS topic are delivered to the user.
{{% /notice %}}

**Next:** [5.6.4 Configure Email Notification](../5.6.4-email-notification/)