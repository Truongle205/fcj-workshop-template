---
title: "Kiểm thử hệ thống"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---

{{% notice info %}}
Trong chương này, toàn bộ hệ thống Smart Home IoT sẽ được kiểm thử end-to-end. Các bài kiểm thử xác nhận kết nối Wi-Fi, MQTT over TLS, telemetry, lưu dữ liệu, gửi email, điều khiển relay và khả năng phục hồi sau khi kết nối bị gián đoạn.
{{% /notice %}}

# Kiểm thử hệ thống

Sau khi hoàn thành firmware và tích hợp các dịch vụ Cloud, hệ thống cần được kiểm thử như một giải pháp hoàn chỉnh.

Việc kiểm thử từng thành phần riêng lẻ hữu ích trong quá trình phát triển, nhưng chưa đủ để xác nhận toàn bộ kiến trúc hoạt động chính xác.

Luồng end-to-end của hệ thống:

```text
Cảm biến
   ↓
ESP32-S3
   ↓
MQTT over TLS 1.2
   ↓
AWS IoT Core
   ↓
AWS IoT Rules Engine
   ├── Amazon DynamoDB
   └── Amazon SNS
          ↓
     Email Subscriber
```

Luồng điều khiển ngược:

```text
AWS IoT MQTT Test Client
            ↓
MQTT command topic
            ↓
ESP32-S3
            ↓
Relay output
            ↓
AWS IoT Device Shadow
```

---

# Mục tiêu

Sau khi hoàn thành chương này, người thực hiện sẽ có thể:

- Kiểm tra quá trình khởi động ESP32-S3.
- Kiểm tra kết nối Wi-Fi.
- Kiểm tra đồng bộ thời gian NTP.
- Kiểm tra xác thực MQTT over TLS.
- Kiểm tra publish telemetry.
- Kiểm tra lưu dữ liệu vào Amazon DynamoDB.
- Kiểm tra cảnh báo mở cửa qua Amazon SNS.
- Kiểm tra điều khiển relay từ xa.
- Kiểm tra đồng bộ AWS IoT Device Shadow.
- Kiểm tra khả năng reconnect Wi-Fi và MQTT.
- Ghi nhận kết quả kiểm thử cuối cùng.

---

# Thời gian thực hiện

**Khoảng 20–30 phút**

---

# Môi trường kiểm thử

| Thành phần | Chức năng |
|---|---|
| ESP32-S3 | Chạy firmware Smart Home. |
| DHT11 | Cung cấp dữ liệu nhiệt độ và độ ẩm. |
| LDR | Cung cấp giá trị ánh sáng. |
| Cảm biến cửa | Cung cấp trạng thái mở và đóng. |
| Relay module | Đại diện cho thiết bị điều khiển từ xa. |
| Mạng Wi-Fi | Cung cấp kết nối Internet. |
| AWS IoT Core | Xử lý MQTT communication. |
| Amazon DynamoDB | Lưu telemetry. |
| Amazon SNS | Gửi email cảnh báo. |
| PlatformIO Serial Monitor | Theo dõi log firmware. |

---

# Danh sách bài kiểm thử

| Mã | Bài kiểm thử | Kết quả mong đợi |
|---|---|---|
| T01 | Khởi động ESP32-S3 | Firmware khởi động bình thường. |
| T02 | Kết nối Wi-Fi | Thiết bị nhận địa chỉ IP hợp lệ. |
| T03 | Đồng bộ NTP | Thiết bị nhận timestamp hợp lệ. |
| T04 | MQTT over TLS | Kết nối AWS IoT thành công. |
| T05 | Publish telemetry | AWS IoT Core nhận JSON message. |
| T06 | Lưu DynamoDB | Item xuất hiện trong bảng. |
| T07 | Door Alert | Email được gửi khi `door_open = true`. |
| T08 | Lệnh Relay ON | Relay chuyển sang ON. |
| T09 | Lệnh Relay OFF | Relay chuyển sang OFF. |
| T10 | Device Shadow | Reported state khớp trạng thái thật. |
| T11 | Phục hồi Wi-Fi | Thiết bị reconnect sau khi mạng trở lại. |
| T12 | Phục hồi MQTT | Thiết bị reconnect và subscribe lại. |
| T13 | DHT11 không hợp lệ | Các telemetry field khác vẫn được gửi. |

---

# Kiểm thử 1 - Khởi động thiết bị

Kết nối ESP32-S3 với máy tính và mở Serial Monitor:

```bash
pio device monitor
```

Reset thiết bị.

Kết quả mong đợi:

```text
ESP32 Smart Home starting
Relay initialized: OFF
DHT11 initialized
Connecting WiFi...
```

Xác nhận:

- Firmware khởi động đúng.
- Relay được đặt ở trạng thái OFF.
- Sensor module được khởi tạo.
- Thiết bị không liên tục reset.

---

# Kiểm thử 2 - Kết nối Wi-Fi

Chờ ESP32-S3 kết nối Access Point.

Kết quả:

```text
WiFi Connected
IP: 172.20.10.2
```

Xác nhận:

- `WiFi.status()` đạt `WL_CONNECTED`.
- Thiết bị nhận IP hợp lệ.
- DNS hoạt động.
- ESP32-S3 có thể truy cập Internet.

---

# Kiểm thử 3 - Đồng bộ thời gian NTP

Sau khi Wi-Fi connected, firmware đồng bộ thời gian.

Kết quả mong đợi:

```text
Waiting for NTP...
Current UTC: 2026-07-24 09:54:43
```

Xác nhận:

- Timestamp hợp lệ.
- Ngày giờ không còn ở epoch mặc định.
- NTP hoàn thành trước khi kết nối MQTT.

{{% notice note %}}
Thời gian hệ thống sai có thể làm TLS certificate validation thất bại dù Certificate và Private Key đúng.
{{% /notice %}}

---

# Kiểm thử 4 - Kết nối MQTT over TLS

Firmware nạp Root CA, Device Certificate và Private Key trước khi kết nối AWS IoT Core.

Kết quả:

```text
Connecting MQTT...
AWS IoT connected
Command subscribe: OK
MQTT connected successfully
```

Xác nhận:

- AWS IoT Endpoint đúng.
- Sử dụng cổng `8883`.
- TLS authentication thành công.
- Command topic được subscribe.
- Shadow topic được subscribe nếu Device Shadow đang sử dụng.

---

# Kiểm thử 5 - Publish Telemetry

Mở:

```text
AWS IoT Core
→ Test
→ MQTT test client
```

Subscribe:

```text
smarthome/esp32-home-01/telemetry
```

Chờ ESP32-S3 gửi message.

Payload mong đợi:

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

Xác nhận:

- Topic đúng.
- JSON hợp lệ.
- Có `device_id`.
- Timestamp hợp lệ.
- Có các trường sensor và trạng thái thiết bị.
- Message được gửi đúng chu kỳ.

Có thể tái sử dụng ảnh:

```md
![ESP32-S3 gửi telemetry](/fcj-workshop-template/images/workshop/5.5.4/telemetry-published.png)

![AWS IoT Core nhận telemetry](/fcj-workshop-template/images/workshop/5.5.4/telemetry-received.png)
```

---

# Kiểm thử 6 - Lưu dữ liệu DynamoDB

Mở:

```text
Amazon DynamoDB
→ Tables
→ SmartHomeTelemetry
→ Explore table items
```

Xác nhận:

- Item mới được thêm tự động.
- `device_id` chính xác.
- `timestamp` khác nhau giữa các bản ghi.
- Sensor field khớp MQTT payload.
- Door và relay state được lưu đúng.

![Telemetry trong Amazon DynamoDB](/fcj-workshop-template/images/workshop/5.6.2/dynamodb-items.png)

Luồng mong đợi:

```text
ESP32-S3
→ AWS IoT Core
→ AWS IoT Rule
→ Amazon DynamoDB
```

---

# Kiểm thử 7 - Cảnh báo mở cửa

Publish telemetry:

```json
{
  "device_id": "esp32-home-01",
  "timestamp": 1784900000,
  "door_open": true,
  "relay_on": false
}
```

Hoặc mở cảm biến cửa vật lý.

Xác nhận:

- Door Alert Rule khớp message.
- Amazon SNS nhận notification.
- Email Subscriber nhận email.
- Email chứa dữ liệu mong đợi.

![Email cảnh báo mở cửa](/fcj-workshop-template/images/workshop/5.6.4/sns-email-notification.png)

Sau đó publish:

```json
{
  "door_open": false
}
```

Message này không được kích hoạt Door Alert Rule.

---

# Kiểm thử 8 - Lệnh Relay ON

Mở AWS IoT MQTT Test Client.

Publish đến:

```text
smarthome/esp32-home-01/command
```

Payload:

```text
ON
```

Serial Monitor:

```text
Message arrived on topic:
smarthome/esp32-home-01/command

Payload: [ON]
Relay: ON
```

Kiểm tra vật lý:

- Đèn báo relay thay đổi.
- Tải thử thay đổi trạng thái.
- Telemetry tiếp theo có `"relay_on": true`.

---

# Kiểm thử 9 - Lệnh Relay OFF

Publish:

```text
OFF
```

Kết quả:

```text
Payload: [OFF]
Relay: OFF
```

Xác nhận:

- Relay trở về OFF.
- Telemetry có `"relay_on": false`.
- Reported Shadow state được cập nhật.

Có thể dùng lại ảnh:

```md
![Publish command điều khiển relay](/fcj-workshop-template/images/workshop/5.5.5/publish-relay-command.png)

![ESP32-S3 nhận command](/fcj-workshop-template/images/workshop/5.5.5/relay-command-received.png)
```

---

# Kiểm thử 10 - Device Shadow

Publish vào:

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

Xác nhận:

1. Device Shadow tạo delta.
2. ESP32-S3 nhận delta.
3. Relay chuyển sang ON.
4. ESP32-S3 publish:

```json
{
  "state": {
    "reported": {
      "relay_on": true
    }
  }
}
```

5. Desired và reported state đồng bộ.

Lặp lại với `relay_on = false`.

---

# Kiểm thử 11 - Phục hồi Wi-Fi

Tạm thời tắt Wi-Fi Access Point hoặc hotspot.

Hành vi mong đợi:

- MQTT bị ngắt.
- Telemetry tạm dừng.
- Firmware vẫn tiếp tục chạy.
- Thiết bị thử khôi phục kết nối.

Bật lại mạng.

Xác nhận:

- ESP32-S3 reconnect Wi-Fi.
- MQTT connection được thiết lập lại.
- Các topic được subscribe lại.
- Telemetry tiếp tục gửi.

{{% notice warning %}}
Không nên reconnect trong vòng lặp quá nhanh. TLS handshake liên tục có thể làm tăng memory pressure và gây khó khăn khi debug.
{{% /notice %}}

---

# Kiểm thử 12 - Phục hồi MQTT

Giữ Wi-Fi hoạt động nhưng tạo gián đoạn MQTT.

Có thể mô phỏng bằng:

- Tạm thời dùng sai endpoint.
- Tạm thời deactivate Certificate.
- Ngắt mạng trong thời gian ngắn.
- Restart Access Point.

Sau khi khôi phục cấu hình đúng, xác nhận:

```text
Attempting MQTT reconnect...
AWS IoT connected
Command subscribe: OK
```

Firmware cần đóng secure socket cũ trước khi tạo kết nối TLS mới.

---

# Kiểm thử 13 - DHT11 không hợp lệ

Tháo DHT11 hoặc chạy hệ thống khi chưa kết nối sensor.

Kết quả Serial:

```text
Failed to read DHT11
Telemetry warning: invalid DHT11 data
```

Payload mong đợi:

```json
{
  "dht_valid": false,
  "temperature": null,
  "humidity": null,
  "light": 870,
  "door_open": false,
  "relay_on": false
}
```

Xác nhận:

- Firmware không crash.
- MQTT vẫn connected.
- LDR, door và relay state vẫn được gửi.
- Nhiệt độ và độ ẩm lỗi không bị lưu như dữ liệu thật.

---

# Bảng kết quả kiểm thử

| Mã | Bài kiểm thử | Kết quả | Ghi chú |
|---|---|---|---|
| T01 | Khởi động thiết bị | Pass | Firmware khởi tạo bình thường. |
| T02 | Kết nối Wi-Fi | Pass | Nhận IP hợp lệ. |
| T03 | Đồng bộ NTP | Pass | Nhận thời gian chính xác. |
| T04 | MQTT over TLS | Pass | Kết nối AWS IoT thành công. |
| T05 | Publish telemetry | Pass | MQTT Test Client nhận JSON. |
| T06 | Lưu DynamoDB | Pass | Telemetry được lưu. |
| T07 | Door Alert | Pass | Email được gửi. |
| T08 | Relay ON | Pass | Relay được bật từ xa. |
| T09 | Relay OFF | Pass | Relay được tắt từ xa. |
| T10 | Device Shadow | Pass | Desired và reported đồng bộ. |
| T11 | Phục hồi Wi-Fi | Pass | Kết nối trở lại thành công. |
| T12 | Phục hồi MQTT | Pass | MQTT reconnect và subscribe lại. |
| T13 | DHT11 không hợp lệ | Pass | Các field còn lại vẫn được gửi. |

{{% notice warning %}}
Chỉ ghi `Pass` sau khi đã thực hiện và xác nhận kết quả thực tế. Cần thay các giá trị mẫu trong bảng bằng kết quả kiểm thử thật của project.
{{% /notice %}}

---

# Tiêu chí nghiệm thu

Hệ thống được xem là hoạt động khi:

- ESP32-S3 kết nối Wi-Fi ổn định.
- NTP synchronization thành công.
- MQTT over TLS authentication hoạt động.
- Telemetry đến AWS IoT Core.
- DynamoDB lưu dữ liệu tự động.
- Sự kiện mở cửa tạo email cảnh báo.
- Relay điều khiển được từ AWS.
- Device Shadow phản ánh trạng thái relay thật.
- Firmware phục hồi sau gián đoạn kết nối.
- Dữ liệu sensor lỗi không làm hệ thống dừng.

---

# Hạn chế hiện tại

- Door alert có thể lặp khi cửa vẫn mở.
- DHT11 có độ chính xác và tốc độ cập nhật hạn chế.
- Direct command sử dụng plain text `ON` và `OFF`.
- Chưa có local persistent queue khi mất mạng.
- Khả năng khôi phục Device Shadow phụ thuộc vào workflow subscribe và lấy trạng thái Shadow đã triển khai.
- Prototype không được dùng để đóng cắt tải điện áp cao nếu chưa có bảo vệ điện phù hợp.

---

# Kết quả đạt được

Sau khi hoàn thành chương này:

- Luồng Smart Home end-to-end đã được kiểm tra.
- Telemetry từ thiết bị lên Cloud hoạt động.
- Điều khiển relay từ Cloud xuống thiết bị hoạt động.
- DynamoDB và SNS hoạt động đúng.
- Hệ thống phục hồi được sau gián đoạn tạm thời.
- Có đủ bằng chứng kiểm thử cho báo cáo cuối kỳ.

{{% notice tip %}}
Trong chương cuối, các tài nguyên AWS được tạo trong workshop sẽ được xóa để tránh phát sinh chi phí và giảm tài nguyên không còn sử dụng.
{{% /notice %}}

**Tiếp theo:** [5.8 Dọn dẹp tài nguyên](../5.8-cleanup/)