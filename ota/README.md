# OTA Server Files

Thư mục này được push lên GitHub và phục vụ như OTA server cho ESP32-CAM.

## Cách sử dụng

### 1. Cấu hình lần đầu

Sửa `Config.h`:
```cpp
static constexpr const char* FIRMWARE_VERSION = "1.2.0";   // version hiện tại
static constexpr const char* OTA_VERSION_URL  = "https://raw.githubusercontent.com/Trung-Nam-2512/bug-bot-esp32cam-ota/main/ota/version.json";
static constexpr const char* OTA_API_KEY      = "bugbot-ota-2026-03-02";
```

Sửa `ota/version.json` trên GitHub — đặt `api_key` khớp với `OTA_API_KEY` trong Config.h.

---

### 2. Quy trình release firmware mới

```
1. Sửa code → build → lấy firmware.bin từ .pio/build/esp32cam/firmware.bin
2. Tăng FIRMWARE_VERSION trong Config.h, ví dụ: "1.2.0" → "1.3.0"
3. Tạo GitHub Release tag v1.3.0, upload firmware.bin vào release
4. Sửa ota/version.json:
     "version": "1.3.0"
     "firmware_url": "https://github.com/Trung-Nam-2512/bug-bot-esp32cam-ota/releases/download/v1.3.0/firmware.bin"
5. Push version.json lên GitHub main branch
6. ESP32-CAM sẽ tự detect và cập nhật trong lần check tiếp theo
```

---

### 3. MQTT commands

| Command | Payload | Mô tả |
|---|---|---|
| Kiểm tra version | `{"cmd":"ota_check"}` | Chỉ check, không update |
| Cập nhật firmware | `{"cmd":"ota_update"}` | Check + flash nếu có version mới |

### 4. MQTT status events

| Event | Ý nghĩa |
|---|---|
| `{"event":"ota_checking"}` | Đang fetch version.json |
| `{"event":"ota_available","current":"1.2.0","latest":"1.3.0"}` | Có bản mới, chờ lệnh update |
| `{"event":"ota_start","from":"1.2.0","to":"1.3.0","notes":"..."}` | Bắt đầu download + flash |
| `{"event":"ota_progress","pct":45}` | Tiến độ download (mỗi 5%) |
| `{"event":"ota_success","version":"1.3.0"}` | Flash xong, đang restart |
| `{"event":"ota_fail","reason":"fail_download"}` | Thất bại + lý do |
| `{"event":"ota_result","result":"up_to_date","current":"1.3.0"}` | Đã là bản mới nhất |

---

### 5. Lưu ý bảo mật

- `api_key` trong `version.json` phải khớp với `OTA_API_KEY` trong firmware đang chạy
- Nếu không khớp → thiết bị từ chối update
- Để đổi key: cập nhật cả firmware (flash qua cáp lần cuối) và version.json
