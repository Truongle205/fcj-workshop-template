---
title: "Tạo dự án PlatformIO"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.5.1 </b> "
---

{{% notice info %}}
Trong mục này, người thực hiện sẽ tạo một dự án PlatformIO dành cho bo mạch ESP32-S3. Đây sẽ là nền tảng để phát triển toàn bộ firmware của hệ thống Smart Home IoT trong các phần tiếp theo.
{{% /notice %}}

# Tổng quan

PlatformIO là một môi trường phát triển nhúng hiện đại, hỗ trợ nhiều dòng vi điều khiển, trong đó có ESP32.

So với Arduino IDE, PlatformIO cung cấp cấu trúc dự án rõ ràng, quản lý thư viện tự động, hỗ trợ nhiều môi trường biên dịch và tích hợp trực tiếp với Visual Studio Code.

Trong workshop này, toàn bộ chương trình điều khiển ESP32-S3 sẽ được phát triển bằng PlatformIO.

---

# Mục tiêu

Sau khi hoàn thành mục này, người thực hiện sẽ có thể:

- Cài đặt PlatformIO.
- Tạo dự án mới cho ESP32-S3.
- Cấu hình môi trường phát triển.
- Hiểu cấu trúc thư mục của dự án.
- Biên dịch và nạp chương trình lên ESP32-S3.

---

# Thời gian thực hiện

**Khoảng 5 phút**

---

# Bước 1 - Mở Visual Studio Code

Khởi động **Visual Studio Code** và đảm bảo tiện ích mở rộng **PlatformIO IDE** đã được cài đặt.

Từ thanh công cụ bên trái, chọn biểu tượng **PlatformIO**.

---

# Bước 2 - Tạo dự án mới

Tạo một dự án PlatformIO với các thông số sau.

| Thiết lập | Giá trị |
|------------|----------|
| Project Name | awsprj |
| Board | ESP32-S3 Dev Module |
| Framework | Arduino |

Sau đó nhấn **Finish** và chờ PlatformIO tạo dự án.

![](/images/workshop/5.5.1/platformio-project.png)

---

# Bước 3 - Tìm hiểu cấu trúc dự án

Sau khi hoàn tất, PlatformIO sẽ tạo cấu trúc thư mục như sau.

```text
awsprj/
├── include/
├── lib/
├── src/
│   └── main.cpp
├── test/
├── platformio.ini
└── .pio/
```

Ý nghĩa của từng thư mục:

| Thư mục | Chức năng |
|----------|-----------|
| include | Chứa các file header |
| lib | Chứa thư viện do người dùng bổ sung |
| src | Chứa mã nguồn chính |
| test | Chứa các bài kiểm thử |
| platformio.ini | Cấu hình của dự án |

---

# Bước 4 - Kiểm tra file platformio.ini

Mở file **platformio.ini** và kiểm tra cấu hình của dự án.

Ví dụ:

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
```

Các thư viện cần thiết sẽ được bổ sung ở các mục tiếp theo.

---

# Bước 5 - Biên dịch dự án

Nhấn **Build** trên thanh công cụ của PlatformIO.

Nếu cấu hình chính xác, quá trình biên dịch sẽ hoàn thành mà không xuất hiện lỗi.

---

# Bước 6 - Nạp chương trình

Kết nối ESP32-S3 với máy tính bằng cáp USB.

Nhấn **Upload** để nạp chương trình.

PlatformIO sẽ tự động biên dịch và ghi firmware xuống bo mạch.

---

# Kết quả đạt được

Sau khi hoàn thành mục này:

- Dự án PlatformIO đã được tạo.
- ESP32-S3 được cấu hình đúng.
- Dự án biên dịch thành công.
- Firmware được nạp thành công lên bo mạch.

{{% notice tip %}}
Dự án PlatformIO được tạo trong mục này sẽ được sử dụng xuyên suốt các chương tiếp theo của workshop để phát triển và kiểm thử các chức năng của hệ thống Smart Home IoT.
{{% /notice %}}

**Tiếp theo:** [5.5.2 Kết nối Wi-Fi](../5.5.2-wifi/)