---
title: "Tạo Device Certificate"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo X.509 Device Certificate để ESP32-S3 có thể xác thực với AWS IoT Core thông qua cơ chế Mutual TLS Authentication.
{{% /notice %}}

# Tổng quan

AWS IoT Core không sử dụng tên đăng nhập và mật khẩu để xác thực thiết bị.

Thay vào đó, mỗi thiết bị được cấp một **X.509 Device Certificate** đóng vai trò là danh tính số (Digital Identity). Trong quá trình thiết lập kết nối TLS, AWS IoT Core sẽ kiểm tra chứng chỉ này trước khi cho phép ESP32-S3 kết nối MQTT.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Tạo X.509 Device Certificate.
- Tải các tệp xác thực của thiết bị.
- Kích hoạt Device Certificate.
- Hiểu vai trò của từng tệp chứng chỉ.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Sinh Device Certificate

Khi tạo AWS IoT Thing, chọn:

```text
Auto-generate a new certificate
```

AWS sẽ tự động tạo:

- Device Certificate
- Public Key
- Private Key

![](/fcj-workshop-template/images/workshop/5.4.2/create-certificate.png)

Nhấn **Create Thing** để tiếp tục.

---

# Bước 2 - Kiểm tra Certificate

Sau khi tạo thành công, AWS sẽ hiển thị thông tin của Device Certificate.

![](/fcj-workshop-template/images/workshop/5.4.2/certificate-generated.png)

---

# Bước 3 - Tải các tệp chứng chỉ

Tải xuống các tệp sau:

- Device Certificate (`certificate.pem.crt`)
- Private Key (`private.pem.key`)
- Public Key (`public.pem.key`)
- Amazon Root CA 1 (`AmazonRootCA1.pem`)

![](/fcj-workshop-template/images/workshop/5.4.2/download-certificates.png)

{{% notice warning %}}
Private Key chỉ được tải xuống một lần duy nhất. Nếu làm mất, cần tạo Device Certificate mới.
{{% /notice %}}

---

# Bước 4 - Kích hoạt Certificate

Đổi trạng thái Device Certificate sang **Active**.

Chỉ những chứng chỉ ở trạng thái **Active** mới được phép xác thực với AWS IoT Core.

![](/fcj-workshop-template/images/workshop/5.4.2/activate-certificate.png)

---

# Ý nghĩa của các tệp chứng chỉ

| Tệp | Chức năng |
|------|-----------|
| Device Certificate | Xác thực ESP32-S3 với AWS IoT Core. |
| Private Key | Chứng minh quyền sở hữu của thiết bị. |
| Public Key | Ghép cặp với Private Key. |
| Amazon Root CA 1 | Xác thực máy chủ AWS IoT Core. |

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Device Certificate đã được tạo.
- Các tệp chứng chỉ đã được tải về.
- Certificate ở trạng thái **Active**.
- Các tệp đã sẵn sàng để sử dụng trong chương trình ESP32.

{{% notice tip %}}
Các tệp chứng chỉ sẽ được sử dụng trong chương **5.5 ESP32 Development** để thiết lập kết nối MQTT over TLS giữa ESP32-S3 và AWS IoT Core.
{{% /notice %}}

**Tiếp theo:** [5.4.3 Gắn AWS IoT Policy](../5.4.3-attach-policy/)