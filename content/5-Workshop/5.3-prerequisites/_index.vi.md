---
title: "Điều kiện chuẩn bị"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

{{% notice warning %}}
Trước khi bắt đầu workshop, hãy đảm bảo rằng tất cả phần cứng, phần mềm và tài nguyên AWS đã được chuẩn bị đầy đủ. Việc hoàn thành các điều kiện chuẩn bị sẽ giúp quá trình triển khai diễn ra thuận lợi và hạn chế các lỗi cấu hình.
{{% /notice %}}

# Điều kiện chuẩn bị

Chương này trình bày các yêu cầu về phần cứng, phần mềm, tài khoản AWS và kiến thức nền cần có trước khi triển khai hệ thống Smart Home IoT.

---

# Yêu cầu phần cứng

Workshop sử dụng các thiết bị phần cứng sau.

| Phần cứng | Mô tả |
|-----------|------|
| ESP32-S3 Development Board | Bộ điều khiển trung tâm của hệ thống |
| Cáp USB Type-C | Nạp chương trình và Serial Monitor |
| Cảm biến DHT11 | Đo nhiệt độ và độ ẩm |
| Cảm biến LDR | Đo cường độ ánh sáng |
| Cảm biến cửa từ | Phát hiện trạng thái cửa |
| Relay Module | Điều khiển thiết bị điện từ xa |
| Breadboard và dây nối | Lắp ráp mạch thử nghiệm |
| Mạng Wi-Fi | Kết nối Internet |

ESP32-S3 chịu trách nhiệm đọc dữ liệu cảm biến, gửi telemetry lên AWS IoT Core và nhận các lệnh điều khiển relay thông qua MQTT.

---

# Yêu cầu phần mềm

Cần cài đặt các phần mềm sau trước khi bắt đầu workshop.

| Phần mềm | Mục đích |
|-----------|----------|
| Visual Studio Code | Môi trường lập trình |
| PlatformIO IDE | Phát triển firmware cho ESP32 |
| Git | Quản lý mã nguồn |
| Python 3 | Phụ thuộc của PlatformIO |
| Serial Monitor | Theo dõi và gỡ lỗi chương trình |

Kiểm tra PlatformIO đã được cài đặt thành công bằng lệnh:

```bash
pio --version
```

Kết quả mong đợi:

```text
PlatformIO Core, version 6.x.x
```

---

# Yêu cầu tài khoản AWS

Người thực hiện cần có một tài khoản AWS và quyền truy cập các dịch vụ sau:

- AWS IoT Core
- Amazon DynamoDB
- Amazon SNS
- AWS IAM
- Amazon CloudWatch

Trong workshop này, tất cả tài nguyên sẽ được triển khai tại Region:

```text
US East (N. Virginia)
us-east-1
```

Việc sử dụng cùng một Region giúp đảm bảo tất cả dịch vụ có thể kết nối và hoạt động chính xác.

---

# Các tài nguyên AWS sẽ tạo

Trong quá trình thực hiện workshop, các tài nguyên AWS sau sẽ được tạo.

| Tài nguyên | Số lượng |
|------------|---------:|
| AWS IoT Thing | 1 |
| X.509 Device Certificate | 1 |
| AWS IoT Policy | 1 |
| Amazon DynamoDB Table | 1 |
| Amazon SNS Topic | 1 |
| Email Subscription | 1 |
| AWS IoT Rule | 2 |
| AWS IAM Role | 1 |

---

# Yêu cầu kết nối mạng

ESP32-S3 cần kết nối Internet thông qua mạng Wi-Fi.

Thiết bị hoạt động ở chế độ **Station (STA Mode)** để kết nối với một điểm truy cập (Access Point) có sẵn.

Mạng Wi-Fi cần đáp ứng các điều kiện sau:

- Có kết nối Internet.
- Hỗ trợ phân giải tên miền (DNS).
- Cho phép kết nối HTTPS.
- Cho phép MQTT over TLS thông qua cổng TCP 8883.

---

# Môi trường phát triển

Mã nguồn của dự án được tổ chức theo cấu trúc PlatformIO.

```text
awsprj/
├── include/
├── src/
├── certificates/
├── platformio.ini
└── lib/
```

Trong đó:

- **include/** chứa các file header của dự án.
- **src/** chứa toàn bộ mã nguồn C++.
- **certificates/** lưu Root CA, Device Certificate và Private Key dùng để xác thực với AWS IoT Core.
- **platformio.ini** là file cấu hình PlatformIO.
- **lib/** chứa các thư viện bổ sung (nếu có).

Toàn bộ firmware được phát triển bằng ngôn ngữ C++ và biên dịch bằng PlatformIO.

---

# Kiến thức nền

Để hoàn thành workshop, người đọc nên có kiến thức cơ bản về:

- Lập trình C/C++.
- Vi điều khiển ESP32.
- Giao thức MQTT.
- Định dạng dữ liệu JSON.
- AWS Management Console.
- Kiến thức mạng máy tính cơ bản.

Không yêu cầu người thực hiện phải có kinh nghiệm trước với AWS IoT Core vì tất cả các bước sẽ được hướng dẫn chi tiết trong workshop.

---

# Kiểm tra trước khi bắt đầu

Trước khi chuyển sang chương tiếp theo, hãy xác nhận các nội dung sau:

- ✅ PlatformIO đã được cài đặt thành công.
- ✅ ESP32-S3 được máy tính nhận diện.
- ✅ Có tài khoản AWS hoạt động.
- ✅ Mạng Wi-Fi có kết nối Internet.
- ✅ Đã chuẩn bị đầy đủ các thiết bị phần cứng.

{{% notice tip %}}
Sau khi hoàn thành các điều kiện chuẩn bị, người thực hiện có thể chuyển sang chương tiếp theo để tạo các tài nguyên AWS IoT cần thiết cho hệ thống Smart Home.
{{% /notice %}}

**Tiếp theo:** [Cấu hình AWS IoT](../5.4-aws-iot-configuration/)