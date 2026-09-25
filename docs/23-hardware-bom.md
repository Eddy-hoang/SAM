# 23 - Danh mục Vật tư Phần cứng (Hardware BOM) & Đặc tả Sơ đồ Chân Pinout

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Mua sắm Phần cứng & Đấu nối Sơ đồ  

---

## 1. Danh mục Linh kiện Phần cứng Phân loại (Hardware BOM)

### Nhóm A: Linh kiện Bắt buộc (MVP Prototype Baseline)

| Tên Linh kiện | Chức năng / Nhiệm vụ Chính | SL | Giao diện Truyền | Chi phí Ước tính (VND) | Trạng thái Kiến trúc |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ESP32-S3-WROOM-1-N16R8 CAM** | Nút Edge Vision AI + Phát ESP-NOW | 2 | DVP / QSPI / USB-C | ~180,000 VND ($7.50) | `[DECISION]` |
| **Module Camera OV2640** | Cảm biến Ảnh CMOS 2MP | 2 | 24-Pin DVP Ribbon | Đi kèm kit CAM | `[DECISION]` |
| **Node MCU ESP32-C3 / ESP8266** | Nút Cảm biến Môi trường | 2 | GPIO / ADC / I2C | ~60,000 VND ($2.50) | `[DECISION]` |
| **Cảm biến Khói / Gas MQ-2** | Phát hiện khí dễ cháy & nồng độ khói| 1 | Analog ADC (Chân A0) | ~35,000 VND ($1.50) | `[DECISION]` |
| **Cảm biến Chuyển động PIR HC-SR501**| Phát hiện chuyển động hồng ngoại người| 2 | Digital GPIO | ~25,000 VND ($1.00) | `[DECISION]` |
| **Công tắc Từ Cửa Reed Switch**| Phát hiện mở cửa vi phạm ranh giới | 2 | Digital GPIO Interrupt | ~15,000 VND ($0.60) | `[DECISION]` |
| **Còi Báo động Điện tử Active Buzzer**| Còi phát âm thanh cảnh báo local decibel cao| 2 | Digital GPIO Relay/Transistor | ~20,000 VND ($0.80) | `[DECISION]` |
| **Raspberry Pi 4B (4GB) hoặc N100** | Edge Gateway Host & Máy chủ CSDL | 1 | Ethernet / USB 3.0 | ~1,400,000 VND ($55.00)| `[DECISION]` |

### Nhóm B: Linh kiện Khuyến nghị (Nâng cao cho Trình diễn Demo)

| Tên Linh kiện | Mục đích Sử dụng | SL | Giao diện | Chi phí Ước tính (VND) | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Củ nguồn 5V 2A + Cáp USB-C** | Nguồn điện ổn định cho các nút ESP32 CAM | 3 | Type-C | ~100,000 VND ($4.00) | `[DECISION]` |
| **Tụ hóa Lọc nguồn 1000uF 10V** | Lọc sụt áp điện áp khi Wi-Fi phát sóng | 4 | Nối trực tiếp 5V/GND | ~10,000 VND ($0.40) | `[DECISION]` |

---

## 2. Bảng Gán Chân Sơ đồ Pinout ESP32-S3 CAM

```text
       ┌─────────────────────────────────────────────────────────────┐
       │             Bảng Gán Chân Pinout ESP32-S3 CAM               │
       ├──────────────────────────┬──────────────────────────────────┤
       │ Chức năng                │ Số Chân GPIO trên ESP32-S3       │
       ├──────────────────────────┼──────────────────────────────────┤
       │ Camera DVP SIOC (SCL)    │ GPIO 39                          │
       │ Camera DVP SIOD (SDA)    │ GPIO 40                          │
       │ Camera VSYNC             │ GPIO 38                          │
       │ Camera HREF              │ GPIO 47                          │
       │ Camera PCLK              │ GPIO 13                          │
       │ Camera XCLK              │ GPIO 10                          │
       │ Camera Data D0 - D7      │ GPIO 11, 9, 8, 6, 4, 2, 3, 15    │
       │ Status LED (Onboard)     │ GPIO 21                          │
       │ ESP-NOW Serial Tx Bridge │ GPIO 43 (TXD0)                   │
       │ ESP-NOW Serial Rx Bridge │ GPIO 44 (RXD0)                   │
       └──────────────────────────┴──────────────────────────────────┘
```
