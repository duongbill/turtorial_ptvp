# 📋 Hướng Dẫn Cài Đặt & Kết Nối Phần Cứng AquaSTEM (Chuẩn Production)

Tài liệu này hướng dẫn chi tiết cách thiết lập hệ thống giám sát môi trường nuôi trồng thủy sản **AquaSTEM**. Hệ thống đã được nâng cấp từ kết nối dây Serial truyền thống lên **Kiến trúc Không dây Chuẩn Production** sử dụng Docker và MQTT Broker (Eclipse Mosquitto) trên nền tảng WSL 2 (Windows Subsystem for Linux).

---

## 🏗️ 1. So Sánh Kiến Trúc Hệ Thống

Dưới đây là sự chuyển đổi từ mô hình kết nối có dây truyền thống sang mô hình không dây chuẩn công nghiệp:

### 🔴 Mô hình truyền thống (Cáp USB / Cổng COM)
```text
[Cảm biến] ──▶ [ESP32] ──(Cáp USB)──▶ [Cổng COM3 Máy tính] ──(pyserial)──▶ [Python Backend]
```
* **Hạn chế:** 
  * Giới hạn khoảng cách vật lý của cáp USB (thường dưới 2 mét).
  * Chỉ kết nối được duy nhất 1 thiết bị vào 1 cổng COM vật lý.
  * Cổng COM dễ bị đổi số tự động (COM4, COM5...) khi rút ra cắm lại gây lỗi code.

### 🟢 Mô hình hiện đại (Wi-Fi + MQTT Broker qua Docker)
```text
[Cảm biến] ──▶ [ESP32] ──(Wi-Fi LAN)──▶ [MQTT Broker:1883 (Docker)] ──(TCP/IP)──▶ [Python Backend]
```
* **Ưu điểm vượt trội:**
  * **Không giới hạn khoảng cách:** Thiết bị ESP32 và máy tính chỉ cần kết nối chung một mạng Wi-Fi (hoặc qua Internet).
  * **Khả năng mở rộng (Scalability):** Hỗ trợ nhiều thiết bị ESP32 (đa nút cảm biến) đồng thời gửi dữ liệu về một Broker duy nhất.
  * **Môi trường đồng nhất nhờ Docker:** MQTT Broker chạy độc lập bên trong Docker container, đảm bảo hoạt động ổn định trên mọi hệ điều hành (Windows, macOS, Linux) mà không bị xung đột thư viện.

---

## 🛠️ 2. Hướng Dẫn Thiết Lập Môi Trường (Windows/WSL2/Docker)

Để chạy được MQTT Broker ảo hóa, máy tính cần đáp ứng các bước cấu hình ảo hóa sau:

### Bước 2.1: Kích hoạt ảo hóa (Virtualization) trong BIOS & Windows Features
1. **Kiểm tra ảo hóa:** Nhấn tổ hợp `Ctrl + Shift + Esc` để mở **Task Manager** -> Chọn tab **Performance** -> **CPU**. Đảm bảo dòng **Virtualization** hiển thị là **Enabled**. 
   > *Lưu ý: Nếu hiển thị "Disabled", bạn phải khởi động lại máy tính vào BIOS để bật tính năng **Intel Virtualization Technology (VT-x)** hoặc **SVM Mode (AMD)**.*
2. **Kích hoạt Windows Features:** Tìm kiếm từ khóa `Turn Windows features on or off` trên Windows, tích chọn hai mục sau rồi nhấn **OK** và restart máy:
   * **Virtual Machine Platform**
   * **Windows Subsystem for Linux**

### Bước 2.2: Cập nhật nhân WSL 2 và khởi động lại
Mở **PowerShell bằng quyền Administrator** và thực hiện chạy các lệnh sau để đảm bảo WSL hoạt động chính xác:
```powershell
# Cập nhật WSL lên phiên bản mới nhất
wsl --update

# Tắt hoàn toàn dịch vụ WSL để áp dụng thay đổi
wsl --shutdown
```

### Bước 2.3: Khởi chạy hạ tầng bằng Docker Compose
Dự án đã định nghĩa sẵn cấu hình cho dịch vụ MQTT Broker (Eclipse Mosquitto) tại cổng `1883`. Hãy di chuyển đến thư mục gốc của dự án (nơi chứa file `docker-compose.yml`) bằng Terminal của VS Code và chạy:
```bash
docker compose up -d
```
> [!NOTE]
> Lệnh trên sẽ tải hình ảnh (image) của Mosquitto về máy và khởi chạy ngầm Broker. Cổng kết nối mặc định của Broker là `1883`.

---

## 🔌 3. Hướng Dẫn Cấu Hình Thiết Bị & Kết Nối

### 3.1 Hướng dẫn kết nối phần cứng ESP32
Mạch điều khiển ESP32 trong bộ kit AquaSTEM đã được lập trình sẵn. **Bạn không cần viết code hay cấu hình gì thêm.**

Chỉ cần cấp nguồn cho ESP32 (bằng cách cắm cáp USB vào máy tính hoặc củ sạc), bo mạch sẽ tự động khởi động và gửi tín hiệu cảm biến không dây về hệ thống.

### 3.2 Cấu hình Dịch Vụ Backend Python nhận dữ liệu
Hãy loại bỏ hoàn toàn thư viện đọc cổng COM (`pyserial`). Sử dụng thư viện `paho-mqtt` để đăng ký lắng nghe (Subscribe) trực tiếp từ Broker chạy tại `localhost:1883`:

```bash
# Cài đặt thư viện MQTT Client cho Python
pip install paho-mqtt
```

Tạo file Python (ví dụ: `subscriber.py`) hoặc tích hợp mã nguồn sau vào dịch vụ Backend của bạn:
```python
import json
import paho.mqtt.client as mqtt

# Cấu hình kết nối MQTT Broker
MQTT_BROKER = "localhost"
MQTT_PORT = 1883
MQTT_TOPIC = "aquastem/sensors"

# Callback khi kết nối thành công tới Broker
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("✅ Kết nối MQTT Broker thành công!")
        # Subscribe vào chủ đề dữ liệu của ESP32
        client.subscribe(MQTT_TOPIC)
        print(f"📡 Đang lắng nghe dữ liệu từ Topic: {MQTT_TOPIC}")
    else:
        print(f"❌ Kết nối thất bại, mã lỗi: {rc}")

# Callback khi nhận được dữ liệu (tin nhắn) mới
def on_message(client, userdata, msg):
    try:
        # Giải mã payload JSON nhận được từ ESP32
        payload = msg.payload.decode("utf-8")
        data = json.loads(payload)
        
        print("\n📥 Nhận thông số cảm biến mới:")
        print(f"  - Độ pH: {data.get('ph')}")
        print(f"  - Nhiệt độ: {data.get('temperature')} °C")
        print(f"  - Độ đục: {data.get('turbidity')} NTU")
        
        # Ở đây bạn có thể viết logic xử lý dữ liệu (lưu Database, tính toán vi phân...)
        
    except Exception as e:
        print(f"❌ Lỗi giải mã dữ liệu JSON: {e}")

# Khởi tạo MQTT Client
client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

try:
    print(f"🔄 Đang kết nối tới MQTT Broker tại {MQTT_BROKER}:{MQTT_PORT}...")
    client.connect(MQTT_BROKER, MQTT_PORT, 60)
    
    # Bắt đầu vòng lặp vô hạn để giữ kết nối và xử lý dữ liệu truyền đến
    client.loop_forever()
except KeyboardInterrupt:
    print("\n👋 Đã tắt dịch vụ lắng nghe MQTT.")
except Exception as e:
    print(f"❌ Lỗi kết nối Broker: {e}")
```

---

## ⚡ 4. Các Lệnh Quản Trị Docker Hữu Ích

* **Khởi chạy hệ thống ngầm:** `docker compose up -d`
* **Dừng và giải phóng tài nguyên:** `docker compose down`
* **Xem logs hệ thống thời gian thực:** `docker compose logs -f`
* **Kiểm tra trạng thái các container:** `docker compose ps`
