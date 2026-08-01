---
title: "Dọn dẹp tài nguyên"
date: 2026-08-01
weight: 8
chapter: false
pre: " <b> 5.8 </b> "
---

{{% notice warning %}}
Chương này sẽ xóa vĩnh viễn các tài nguyên AWS được tạo trong workshop. Hãy sao lưu hoặc xuất dữ liệu telemetry cần giữ lại trước khi tiếp tục.
{{% /notice %}}

# Dọn dẹp tài nguyên

Sau khi hoàn thành workshop Smart Home IoT, các tài nguyên AWS không còn sử dụng nên được xóa.

Mặc dù các dịch vụ trong project là dịch vụ được AWS quản lý và chi phí sử dụng thử nghiệm thường không lớn, việc giữ tài nguyên không cần thiết vẫn có thể gây phát sinh chi phí, tăng rủi ro bảo mật hoặc gây nhầm lẫn cho các project sau.

Quá trình dọn dẹp sẽ xử lý:

- AWS IoT Rules.
- Amazon DynamoDB table.
- Amazon SNS email subscription.
- Amazon SNS topic.
- AWS IoT Policy.
- X.509 Device Certificate.
- AWS IoT Thing.
- AWS IAM Role và Policy dùng cho IoT Rule.
- Certificate và file bí mật được lưu trên máy tính.

---

# Mục tiêu

Sau khi hoàn thành chương này, người thực hiện sẽ có thể:

- Xóa AWS IoT Rule.
- Xóa DynamoDB telemetry table.
- Xóa SNS email subscription.
- Xóa SNS Topic.
- Detach và xóa AWS IoT Policy.
- Vô hiệu hóa và xóa Device Certificate.
- Xóa AWS IoT Thing.
- Xóa IAM Role không còn sử dụng.
- Kiểm tra không còn tài nguyên workshop trên AWS.

---

# Thời gian thực hiện

**Khoảng 10–15 phút**

---

# Thứ tự xóa đề xuất

Thực hiện theo thứ tự:

```text
Dừng ESP32-S3
        ↓
Xóa AWS IoT Rules
        ↓
Xóa Amazon DynamoDB table
        ↓
Xóa SNS subscription và topic
        ↓
Detach AWS IoT Policy
        ↓
Detach Thing khỏi Certificate
        ↓
Vô hiệu hóa và xóa Certificate
        ↓
Xóa AWS IoT Thing
        ↓
Xóa IAM Role và Policy
        ↓
Xóa hoặc bảo vệ Local Secrets
```

Thứ tự này giúp hạn chế lỗi do các resource vẫn còn liên kết với nhau.

---

# Bước 1 - Dừng ESP32-S3

Trước khi xóa tài nguyên Cloud, ngắt kết nối hoặc dừng firmware ESP32-S3.

Mục đích là tránh thiết bị liên tục reconnect hoặc publish message trong khi các AWS resource đang bị xóa.

Có thể:

- Rút cáp USB.
- Tắt nguồn bo mạch.
- Ngắt ESP32-S3 khỏi Wi-Fi.
- Nạp firmware tạm thời không kết nối AWS IoT Core.

---

# Bước 2 - Xóa AWS IoT Rules

Mở:

```text
AWS IoT Core
→ Message routing
→ Rules
```

Xóa các rule đã tạo.

Ví dụ:

```text
StoreSmartHomeTelemetry
SmartHomeDoorAlert
```

Với từng rule:

1. Chọn rule.
2. Chọn **Delete**.
3. Xác nhận thao tác.

Sau khi xóa, AWS IoT Rules Engine không còn ghi telemetry vào DynamoDB hoặc publish cảnh báo đến SNS.

---

# Bước 3 - Xóa DynamoDB Table

Mở:

```text
Amazon DynamoDB
→ Tables
→ SmartHomeTelemetry
```

Nếu cần giữ dữ liệu, hãy backup hoặc export trước.

Để xóa:

1. Chọn table.
2. Chọn **Delete**.
3. Nhập tên table nếu được yêu cầu.
4. Xác nhận.

{{% notice warning %}}
Xóa DynamoDB table sẽ xóa vĩnh viễn toàn bộ telemetry nếu chưa tạo backup hoặc export.
{{% /notice %}}

Kiểm tra table không còn xuất hiện trong danh sách DynamoDB.

---

# Bước 4 - Xóa SNS Email Subscription

Mở:

```text
Amazon SNS
→ Subscriptions
```

Tìm subscription thuộc topic:

```text
SmartHomeDoorAlert
```

Chọn subscription và nhấn:

```text
Delete
```

Thao tác này loại bỏ email endpoint khỏi SNS Topic.

---

# Bước 5 - Xóa SNS Topic

Mở:

```text
Amazon SNS
→ Topics
→ SmartHomeDoorAlert
```

Chọn:

```text
Delete
```

Xác nhận tên topic khi AWS yêu cầu.

Sau khi xóa, topic không còn nhận hoặc phân phối notification.

---

# Bước 6 - Detach AWS IoT Policy

Mở:

```text
AWS IoT Core
→ Security
→ Certificates
```

Chọn Certificate của ESP32-S3.

Mở phần Policy được attach và detach Policy.

Ví dụ:

```text
ESP32Policy
```

Policy phải được detach trước khi có thể xóa.

---

# Bước 7 - Detach Certificate khỏi Thing

Tại trang Certificate Details, mở phần Things được liên kết.

Detach certificate khỏi:

```text
esp32-home-01
```

AWS IoT Core không cho phép xóa Certificate nếu nó vẫn đang attach với Thing hoặc Policy.

---

# Bước 8 - Vô hiệu hóa Device Certificate

Mở Certificate Details.

Đổi trạng thái từ:

```text
Active
```

sang:

```text
Inactive
```

Certificate phải ở trạng thái Inactive trước khi xóa.

Sau khi bị vô hiệu hóa, ESP32-S3 không thể tiếp tục xác thực với AWS IoT Core bằng Certificate đó.

---

# Bước 9 - Xóa Device Certificate

Sau khi:

- Detach Thing.
- Detach Policy.
- Chuyển Certificate sang Inactive.

Chọn:

```text
Delete
```

và xác nhận.

{{% notice warning %}}
Xóa Certificate là thao tác vĩnh viễn. Certificate đã xóa không thể được khôi phục.
{{% /notice %}}

---

# Bước 10 - Xóa AWS IoT Policy

Mở:

```text
AWS IoT Core
→ Security
→ Policies
```

Chọn Policy của ESP32-S3.

Ví dụ:

```text
ESP32Policy
```

Kiểm tra Policy không còn attach với Certificate nào.

Chọn:

```text
Delete
```

Nếu Policy có nhiều version, AWS có thể yêu cầu xóa các non-default version trước.

---

# Bước 11 - Xóa AWS IoT Thing

Mở:

```text
AWS IoT Core
→ Manage
→ All devices
→ Things
```

Chọn:

```text
esp32-home-01
```

Kiểm tra không còn Certificate attach.

Chọn:

```text
Delete
```

Xác nhận Thing name nếu được yêu cầu.

Thao tác này xóa danh tính logic của thiết bị khỏi AWS IoT Core.

---

# Bước 12 - Xóa Device Shadow

Vì project có sử dụng AWS IoT Device Shadow để đồng bộ trạng thái relay, nên cần xóa Shadow state không còn sử dụng.

Có thể xóa trong AWS IoT Console hoặc publish đến topic:

```text
$aws/things/esp32-home-01/shadow/delete
```

Payload:

```json
{}
```

Shadow data chứa `desired` và `reported` state sẽ bị xóa.

{{% notice note %}}
Nên thực hiện bước này trước khi xóa AWS IoT Thing để bảo đảm trạng thái thử nghiệm không còn được lưu trên Cloud.
{{% /notice %}}

---

# Bước 13 - Xóa IAM Role

Mở:

```text
AWS IAM
→ Roles
```

Tìm các role được tạo riêng cho workshop.

Ví dụ:

```text
IoTRuleDynamoDBRole
IoTRuleSNSRole
```

Trước khi xóa, kiểm tra role không được dùng bởi project khác.

Với từng role:

1. Mở role.
2. Kiểm tra các Policy đang attach.
3. Detach hoặc xóa custom policy nếu cần.
4. Chọn **Delete**.
5. Xác nhận tên role.

{{% notice warning %}}
Không xóa IAM Role hoặc Policy dùng chung với các workload AWS khác.
{{% /notice %}}

---

# Bước 14 - Xóa Custom IAM Policy

Nếu đã tạo managed policy riêng cho IoT Rule, mở:

```text
AWS IAM
→ Policies
```

Xóa các policy không còn sử dụng.

Các quyền có thể gồm:

```text
dynamodb:PutItem
sns:Publish
```

Managed Policy không thể bị xóa khi vẫn còn attach với role, user hoặc group.

---

# Bước 15 - Xử lý Local Secrets

Project PlatformIO chứa thông tin nhạy cảm tại:

```text
include/secrets.h
certificates/
```

Các file có thể chứa:

- Wi-Fi SSID.
- Wi-Fi password.
- AWS IoT Endpoint.
- Device Certificate.
- Device Private Key.
- Amazon Root CA.

Nếu không còn sử dụng, có thể xóa bản local:

```bash
rm -rf certificates
rm -f include/secrets.h
```

Không chạy các lệnh này nếu project vẫn cần các file trên.

Đảm bảo `.gitignore` có:

```gitignore
include/secrets.h
certificates/
*.pem
*.key
*.crt
```

---

# Bước 16 - Kiểm tra Git

Chạy:

```bash
git status
```

Đảm bảo file bí mật không được Git theo dõi.

Kiểm tra rule ignore:

```bash
git check-ignore -v include/secrets.h
git check-ignore -v certificates/*
```

Nếu file từng được `git add`, chỉ thêm `.gitignore` sẽ không loại file khỏi Git index.

Sử dụng:

```bash
git rm --cached include/secrets.h
git rm -r --cached certificates
```

Sau đó commit thay đổi.

{{% notice danger %}}
Nếu Private Key đã được push lên remote repository, chỉ xóa file là chưa đủ. Cần vô hiệu hóa và xóa Certificate bị lộ, sau đó tạo Certificate và Private Key mới.
{{% /notice %}}

---

# Bước 17 - Kiểm tra tài nguyên đã xóa

| Tài nguyên | Trạng thái mong đợi |
|---|---|
| Telemetry IoT Rule | Đã xóa |
| Door Alert IoT Rule | Đã xóa |
| DynamoDB telemetry table | Đã xóa |
| SNS email subscription | Đã xóa |
| SNS topic | Đã xóa |
| AWS IoT Policy | Đã xóa |
| Device Certificate | Inactive và đã xóa |
| AWS IoT Thing | Đã xóa |
| Device Shadow | Đã xóa |
| IoT Rule IAM Role | Đã xóa nếu không còn sử dụng |
| Local Private Key | Đã xóa hoặc lưu an toàn |
| Secret bị Git theo dõi | Không có |

---

# Kiểm tra chi phí

Mở:

```text
AWS Billing and Cost Management
→ Cost Explorer
```

Kiểm tra usage của:

- AWS IoT Core.
- Amazon DynamoDB.
- Amazon SNS.
- Amazon CloudWatch.

Dữ liệu chi phí có thể cập nhật trễ.

Có thể kiểm tra thêm:

```text
AWS Billing
→ Bills
```

để tìm dịch vụ vẫn còn phát sinh usage.

---

# Xử lý lỗi

## Không xóa được Certificate

Kiểm tra:

- Certificate đã ở trạng thái `Inactive`.
- Không còn attach Policy.
- Không còn attach Thing.

---

## Không xóa được IoT Policy

Kiểm tra:

- Policy không còn attach với Certificate.
- Các non-default Policy version đã được xóa nếu AWS yêu cầu.

---

## Không xóa được Thing

Kiểm tra:

- Certificate đã được detach.
- Thing không còn association khác.

---

## Không xóa được IAM Role

Kiểm tra:

- Role không còn được IoT Rule sử dụng.
- Attached Policy đã được detach.
- Không có resource AWS khác đang sử dụng role.

---

## DynamoDB vẫn ở trạng thái Deleting

Việc xóa DynamoDB table có thể cần một khoảng thời gian ngắn.

Chờ và refresh danh sách table.

---

# Kết quả đạt được

Sau khi hoàn thành chương này:

- Các tài nguyên AWS của workshop đã được xóa.
- Telemetry đã được backup hoặc xóa.
- Notification resource không còn hoạt động.
- Credential xác thực thiết bị đã bị vô hiệu hóa.
- IAM permission không sử dụng đã được loại bỏ.
- Local secrets được bảo vệ.
- Giảm nguy cơ phát sinh chi phí AWS không cần thiết.

{{% notice tip %}}
Workshop Smart Home IoT đã hoàn thành. Có thể giữ lại source code, sơ đồ kiến trúc, ảnh kiểm thử và tài liệu để tiếp tục phát triển, nhưng tuyệt đối không đưa Private Key hoặc mật khẩu thật vào project nộp hoặc kho mã nguồn công khai.
{{% /notice %}}

# Hoàn thành Workshop

Toàn bộ quy trình đã thực hiện:

```text
Chuẩn bị phần cứng và phần mềm
        ↓
Cấu hình AWS IoT Core
        ↓
Phát triển firmware ESP32-S3
        ↓
Gửi telemetry bảo mật
        ↓
Điều khiển relay từ xa
        ↓
Lưu dữ liệu vào DynamoDB
        ↓
Gửi cảnh báo qua SNS
        ↓
Kiểm thử end-to-end
        ↓
Dọn dẹp tài nguyên AWS
```