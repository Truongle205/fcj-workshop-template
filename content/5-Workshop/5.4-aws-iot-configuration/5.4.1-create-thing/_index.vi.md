---
title: "Tạo AWS IoT Thing"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo một AWS IoT Thing đại diện cho bo mạch ESP32-S3. Đồng thời, AWS IoT Core sẽ tự động tạo X.509 Device Certificate để phục vụ việc xác thực bảo mật giữa thiết bị và AWS Cloud.
{{% /notice %}}

# Tổng quan

Trong AWS IoT Core, mỗi thiết bị vật lý đều được biểu diễn dưới dạng một **AWS IoT Thing**.

Thing là đối tượng dùng để định danh thiết bị trên nền tảng AWS. Nó lưu trữ các thông tin như tên thiết bị, Device Certificate, AWS IoT Policy và các cấu hình liên quan đến việc giao tiếp bảo mật.

Trong workshop này, bo mạch ESP32-S3 sẽ được đăng ký dưới dạng một AWS IoT Thing trước khi tiến hành kết nối đến AWS IoT Core.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Truy cập AWS IoT Core.
- Tạo một AWS IoT Thing mới.
- Sinh X.509 Device Certificate.
- Tải các tệp chứng chỉ cần thiết.
- Kích hoạt Device Certificate.
- Liên kết Certificate với Thing.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Truy cập AWS IoT Core

Đăng nhập vào **AWS Management Console**.

Tại trang chủ AWS, tìm kiếm dịch vụ **IoT Core** bằng thanh tìm kiếm.

![](/fcj-workshop-template/images/workshop/5.4.1/aws-console-home.png)

Sau đó mở dịch vụ **AWS IoT Core**.

---

# Bước 2 - Mở trang quản lý Thing

Trong giao diện AWS IoT Core, chọn:

**Manage → All devices → Things**

![](/fcj-workshop-template/images/workshop/5.4.1/iot-core-dashboard.png)

Nhấn **Create Thing** để tạo thiết bị mới.

---

# Bước 3 - Tạo Thing

Chọn:

```
Create a single Thing
```

Sau đó nhấn **Next**.

![](/fcj-workshop-template/images/workshop/5.4.1/create-thing.png)

---

# Bước 4 - Cấu hình Thing

Nhập tên thiết bị.

Ví dụ:

```text
esp32-home-01
```

Các thông số khác giữ nguyên mặc định.

![](/fcj-workshop-template/images/workshop/5.4.1/thing-name.png)

Sau khi hoàn tất, nhấn **Next**.

---

# Bước 5 - Sinh Device Certificate

Chọn tùy chọn:

```text
Auto-generate a new certificate
```

AWS sẽ tự động tạo:

- Device Certificate
- Public Key
- Private Key

Nhấn **Create Thing** để tiếp tục.

![](/fcj-workshop-template/images/workshop/5.4.1/create-certificate.png)

---

# Bước 6 - Tải chứng chỉ

Sau khi Thing được tạo thành công, AWS sẽ hiển thị các tệp xác thực của thiết bị.

Tải xuống các tệp sau:

- Device Certificate
- Private Key
- Public Key
- Amazon Root CA 1

Các tệp này sẽ được sử dụng ở các chương sau khi cấu hình chương trình trên ESP32-S3.

![](/fcj-workshop-template/images/workshop/5.4.1/download-certificates.png)

{{% notice warning %}}
Private Key chỉ có thể tải xuống **một lần duy nhất**. Hãy lưu trữ tệp này ở nơi an toàn trước khi rời khỏi trang.
{{% /notice %}}

---

# Bước 7 - Kích hoạt Certificate

Sau khi tải chứng chỉ, cần kích hoạt Device Certificate.

Đảm bảo trạng thái của Certificate chuyển sang:

```text
Active
```

![](/fcj-workshop-template/images/workshop/5.4.1/activate-certificate.png)

---

# Bước 8 - Liên kết Certificate với Thing

Tiếp theo, thực hiện liên kết Device Certificate với AWS IoT Thing vừa tạo.

![](/fcj-workshop-template/images/workshop/5.4.1/attach-thing.png)

Sau khi hoàn tất, Device Certificate sẽ trở thành thông tin xác thực chính thức của ESP32-S3.

---

# Kiểm tra kết quả

Mở trang thông tin của Thing và kiểm tra các nội dung sau:

- Tên Thing chính xác.
- Device Certificate đã được liên kết.
- Trạng thái Certificate là **Active**.

![](/fcj-workshop-template/images/workshop/5.4.1/thing-summary.png)

---

# Kết quả đạt được

Sau khi hoàn thành mục này, AWS IoT Core sẽ có các tài nguyên sau:

| Tài nguyên | Trạng thái |
|------------|------------|
| AWS IoT Thing | Đã tạo |
| X.509 Device Certificate | Active |
| Public Key | Đã tải |
| Private Key | Đã tải |
| Amazon Root CA 1 | Đã tải |
| Certificate gắn với Thing | Hoàn thành |

{{% notice tip %}}
Hãy lưu các tệp Certificate, Private Key và Amazon Root CA vào một thư mục an toàn. Các tệp này sẽ được sử dụng trong chương **ESP32 Development** để thiết lập kết nối MQTT over TLS giữa ESP32-S3 và AWS IoT Core.
{{% /notice %}}

**Tiếp theo:** [5.4.2 Tạo Device Certificate](../5.4.2-create-certificate/)