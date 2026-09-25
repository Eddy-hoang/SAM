# 22 - Hướng dẫn Phát triển Cục bộ & Setup (Local Development)

> **Trạng thái Tài liệu:** `[DECISION]` Cấu hình Môi trường Lập trình viên  

---

## 1. Công cụ Yêu cầu & Setup Toolchain

Để thiết lập máy tính phát triển local phục vụ giả lập phần cứng và chạy gateway, cài đặt:
* **Hệ điều hành:** Windows 11 / Linux (Ubuntu 22.04) / macOS
* **Embedded Toolchain:** ESP-IDF v5.1+ hoặc VS Code kết hợp extension PlatformIO
* **Công cụ Backend:** Node.js v20+ LTS, Docker Desktop, Mosquitto MQTT Client (`mosquitto_pub` / `mosquitto_sub`)
* **Công cụ Python:** Python 3.10+ (phục vụ script chuyển đổi mô hình AI & lượng hóa TFLite)

---

## 2. Kiến trúc Môi trường & Giả lập (Mocking Environment)

Khi chưa có sẵn phần cứng ESP32 vật lý, lập trình viên sử dụng **Node Telemetry Simulator**:

```text
  ┌────────────────────────────────────────────────────────┐
  │              Máy tính Phát triển Local                 │
  │                                                        │
  │  ┌────────────────────┐      ┌──────────────────────┐  │
  │  │  ESP32 Node Mock   │─────>│ Local Edge Gateway   │  │
  │  │  (Python Script)   │ MQTT │ (Node.js REST / WS)  │  │
  │  └────────────────────┘      └──────────┬───────────┘  │
  │                                         │ WebSockets   │
  │                              ┌──────────▼───────────┐  │
  │                              │ Vite React Dashboard │  │
  │                              └──────────────────────┘  │
  └────────────────────────────────────────────────────────┘
```

---

## 3. Cấu trúc Workspace Khuyến nghị

```text
/ (Project Root)
├── README.md                  # Tài liệu Tổng quan Dự án
├── docs/                      # Bộ Tài liệu Kỹ thuật
├── config/                    # Template Cấu hình Gateway & Network
│   ├── mosquitto.conf
│   └── gateway.config.json
└── tools/                     # Code Giả lập Phần cứng & Script Test (Non-code mocks)
    └── mock_event_publisher.py
```
