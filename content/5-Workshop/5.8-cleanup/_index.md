---
title: "Cleanup"
date: 2026-08-01
weight: 8
chapter: false
pre: " <b> 5.8 </b> "
---

{{% notice warning %}}
This chapter permanently deletes the AWS resources created during the workshop. Export or back up any telemetry data that must be retained before continuing.
{{% /notice %}}

# Cleanup

After completing the Smart Home IoT workshop, unused AWS resources should be removed.

Although the services used in this project are managed services and may have low usage costs, leaving resources active can still result in unnecessary charges, security exposure, or confusion during future projects.

The cleanup process removes the following resources:

- AWS IoT Rules.
- Amazon DynamoDB table.
- Amazon SNS email subscription.
- Amazon SNS topic.
- AWS IoT Policy.
- X.509 device certificate.
- AWS IoT Thing.
- AWS IAM roles and policies created for IoT Rule actions.
- Local certificate and secret files, when no longer required.

---

# Objectives

After completing this chapter, you will be able to:

- Disable and delete AWS IoT Rules.
- Delete the DynamoDB telemetry table.
- Remove the SNS email subscription.
- Delete the SNS topic.
- Detach and delete AWS IoT policies.
- Deactivate and delete the device certificate.
- Delete the AWS IoT Thing.
- Remove unused AWS IAM roles.
- Verify that no workshop resources remain.

---

# Estimated Time

**Approximately 10–15 minutes**

---

# Recommended Deletion Order

Delete resources in the following order:

```text
Disable ESP32-S3
        ↓
Delete AWS IoT Rules
        ↓
Delete Amazon DynamoDB table
        ↓
Delete SNS subscription and topic
        ↓
Detach AWS IoT Policy
        ↓
Detach Thing from Certificate
        ↓
Deactivate and delete Certificate
        ↓
Delete AWS IoT Thing
        ↓
Delete IAM roles and policies
        ↓
Remove local secrets
```

The order avoids dependency errors when deleting attached resources.

---

# Step 1 – Stop the ESP32-S3

Before deleting cloud resources, disconnect the ESP32-S3 or stop its firmware.

This prevents the device from repeatedly attempting to reconnect or publish messages while AWS resources are being deleted.

You can:

- Disconnect the USB cable.
- Turn off the development board.
- Disconnect the board from Wi-Fi.
- Upload temporary firmware that does not connect to AWS IoT Core.

---

# Step 2 – Delete AWS IoT Rules

Open:

```text
AWS IoT Core
→ Message routing
→ Rules
```

Delete the rules created during the workshop.

Example rules:

```text
StoreSmartHomeTelemetry
SmartHomeDoorAlert
```

Before deletion, verify that the rules are no longer required.

For each rule:

1. Select the rule.
2. Choose **Delete**.
3. Confirm the deletion.

Deleting a rule stops AWS IoT Rules Engine from writing telemetry to DynamoDB or publishing notifications to SNS.

---

# Step 3 – Delete the DynamoDB Table

Open:

```text
Amazon DynamoDB
→ Tables
→ SmartHomeTelemetry
```

Before deleting the table, export any required data.

To delete:

1. Select the table.
2. Choose **Delete**.
3. Enter the table name if confirmation is required.
4. Confirm deletion.

{{% notice warning %}}
Deleting a DynamoDB table permanently removes all telemetry records unless a backup or export has been created.
{{% /notice %}}

Verify that the table no longer appears in the DynamoDB table list.

---

# Step 4 – Delete the SNS Email Subscription

Open:

```text
Amazon SNS
→ Subscriptions
```

Locate the email subscription associated with:

```text
SmartHomeDoorAlert
```

Select the subscription and choose:

```text
Delete
```

This removes the subscriber endpoint from the SNS topic.

---

# Step 5 – Delete the SNS Topic

Open:

```text
Amazon SNS
→ Topics
→ SmartHomeDoorAlert
```

Choose:

```text
Delete
```

Confirm the topic name when prompted.

Deleting the topic prevents any future messages from being published to its subscribers.

---

# Step 6 – Detach the AWS IoT Policy

Open:

```text
AWS IoT Core
→ Security
→ Certificates
```

Select the certificate used by the ESP32-S3.

Open the attached policies section and detach the IoT Policy.

Example policy:

```text
ESP32Policy
```

The policy must be detached before it can be deleted.

---

# Step 7 – Detach the Certificate from the Thing

From the certificate details page, open the attached Things section.

Detach the certificate from:

```text
esp32-home-01
```

AWS IoT Core does not allow deletion of a certificate while it remains attached to a Thing or policy.

---

# Step 8 – Deactivate the Device Certificate

Open the certificate details page.

Change the certificate status from:

```text
Active
```

to:

```text
Inactive
```

A certificate must be inactive before it can be deleted.

After deactivation, the ESP32-S3 can no longer authenticate with AWS IoT Core using that certificate.

---

# Step 9 – Delete the Device Certificate

After detaching the Thing and policy and setting the certificate to inactive, choose:

```text
Delete
```

Confirm the deletion.

{{% notice warning %}}
Certificate deletion is permanent. The same certificate cannot be restored after deletion.
{{% /notice %}}

---

# Step 10 – Delete the AWS IoT Policy

Open:

```text
AWS IoT Core
→ Security
→ Policies
```

Select the policy created for the ESP32-S3.

Example:

```text
ESP32Policy
```

Verify that the policy is not attached to any certificate.

Choose:

```text
Delete
```

If the policy has multiple versions, AWS may require deletion of non-default policy versions before deleting the policy.

---

# Step 11 – Delete the AWS IoT Thing

Open:

```text
AWS IoT Core
→ Manage
→ All devices
→ Things
```

Select:

```text
esp32-home-01
```

Verify that no certificates remain attached.

Choose:

```text
Delete
```

Confirm the Thing name if requested.

Deleting the Thing removes its logical identity from AWS IoT Core.

---

# Step 12 – Delete Device Shadow Data

Deleting a Thing does not always represent the same action as explicitly clearing all application state used during testing.

Before deleting the Thing, the classic unnamed Shadow can be removed through the AWS IoT Console or by publishing to the Shadow delete topic:

```text
$aws/things/esp32-home-01/shadow/delete
```

Publish an empty payload:

```json
{}
```

This removes the stored desired and reported state associated with the Shadow.

{{% notice note %}}
This step is relevant because the project uses AWS IoT Device Shadow to synchronize the relay state.
{{% /notice %}}

---

# Step 13 – Delete IAM Roles

Open:

```text
AWS IAM
→ Roles
```

Locate roles created specifically for the workshop.

Examples:

```text
IoTRuleDynamoDBRole
IoTRuleSNSRole
```

Before deletion, verify that the roles are not used by any other project.

For each role:

1. Open the role.
2. Review attached policies.
3. Detach or delete custom policies if required.
4. Choose **Delete**.
5. Confirm the role name.

{{% notice warning %}}
Do not delete shared roles or policies used by other AWS workloads.
{{% /notice %}}

---

# Step 14 – Delete Custom IAM Policies

If custom managed policies were created for the IoT Rules, open:

```text
AWS IAM
→ Policies
```

Delete policies that are no longer used.

Example permissions include:

```text
dynamodb:PutItem
sns:Publish
```

A managed policy cannot be deleted while it remains attached to a role, user, or group.

---

# Step 15 – Remove Local Secrets

The PlatformIO project contains sensitive information in:

```text
include/secrets.h
certificates/
```

Files may include:

- Wi-Fi SSID.
- Wi-Fi password.
- AWS IoT endpoint.
- Device certificate.
- Device private key.
- Amazon Root CA.

When the credentials are no longer required, remove local copies securely.

Example:

```bash
rm -rf certificates
rm -f include/secrets.h
```

Do not run these commands if the project still requires the files.

For source control, ensure `.gitignore` contains:

```gitignore
include/secrets.h
certificates/
*.pem
*.key
*.crt
```

---

# Step 16 – Verify Git Status

Run:

```bash
git status
```

Ensure that sensitive files are not tracked.

To verify ignored files:

```bash
git check-ignore -v include/secrets.h
git check-ignore -v certificates/*
```

If a secret file was previously added to Git, adding it to `.gitignore` does not automatically remove it from Git history.

Remove it from the current index using:

```bash
git rm --cached include/secrets.h
git rm -r --cached certificates
```

Then commit the removal.

{{% notice danger %}}
If a private key was pushed to a remote repository, deleting the file is not sufficient. Deactivate and delete the exposed certificate immediately, then create a new certificate and private key.
{{% /notice %}}

---

# Step 17 – Verify Resource Cleanup

Review the following checklist:

| Resource | Expected status |
|---|---|
| AWS IoT telemetry rule | Deleted |
| AWS IoT door alert rule | Deleted |
| DynamoDB telemetry table | Deleted |
| SNS email subscription | Deleted |
| SNS topic | Deleted |
| AWS IoT Policy | Deleted |
| Device certificate | Inactive and deleted |
| AWS IoT Thing | Deleted |
| Device Shadow | Deleted |
| IoT Rule IAM roles | Deleted if unused |
| Local private key | Removed or securely stored |
| Secrets tracked by Git | None |

---

# Optional Cost Verification

Open:

```text
AWS Billing and Cost Management
→ Cost Explorer
```

Review recent usage for:

- AWS IoT Core.
- Amazon DynamoDB.
- Amazon SNS.
- Amazon CloudWatch.

Some metrics or billing data may take time to appear.

You can also review:

```text
AWS Billing
→ Bills
```

to identify active service usage.

---

# Troubleshooting

## Certificate cannot be deleted

Verify:

- Certificate status is `Inactive`.
- No AWS IoT Policy is attached.
- No AWS IoT Thing is attached.

---

## IoT Policy cannot be deleted

Verify:

- Policy is not attached to a certificate.
- Non-default policy versions have been removed if required.

---

## Thing cannot be deleted

Verify:

- Certificates are detached.
- Thing groups or other associations are removed if applicable.

---

## IAM role cannot be deleted

Verify:

- The role is no longer used by an AWS IoT Rule.
- Attached policies have been removed.
- No instance profile or other AWS resource uses the role.

---

## DynamoDB table remains in deleting state

DynamoDB table deletion may take a short period.

Refresh the table list after waiting.

---

# Cleanup Result

After completing this chapter:

- All workshop AWS resources have been removed.
- Telemetry storage has been deleted or backed up.
- Notification resources are no longer active.
- Device authentication credentials are invalidated.
- Unused IAM permissions are removed.
- Sensitive local files are protected.
- The risk of unnecessary AWS charges is reduced.

{{% notice tip %}}
The Smart Home IoT workshop is now complete. Keep the source code, architecture diagram, test evidence, and documentation for future improvement, but never include active private keys or passwords in the submitted project.
{{% /notice %}}

# Workshop Completion

You have completed the complete Smart Home IoT workflow:

```text
Prepare hardware and software
        ↓
Configure AWS IoT Core
        ↓
Develop ESP32-S3 firmware
        ↓
Publish telemetry securely
        ↓
Control the relay remotely
        ↓
Store data in DynamoDB
        ↓
Send door alerts through SNS
        ↓
Test the complete system
        ↓
Clean up AWS resources
```