# 03 - Kiến trúc Hệ thống (System Architecture)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Kiến trúc Hệ thống Chính  
> **Tiêu chuẩn Mô hình hóa:** Mô hình C4 (Context, Container, Component, Deployment)  

---

## 1. Phân chia Chức năng Đa tầng (Multi-Layer Functional Breakdown)

Kiến trúc SafeHome AI Mesh được tổ chức thành 6 tầng chức năng độc lập, giảm thiểu phụ thuộc lẫn nhau:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ 6. Tầng Ứng dụng (Web Monitoring Dashboard, Mobile Alert UI)            │
├──────────────────────────────────────────────────────────────────────────┤
│ 5. Tầng Dữ liệu & Lưu trữ (TimescaleDB / SQLite, Event Store, Config DB) │
├──────────────────────────────────────────────────────────────────────────┤
│ 4. Tầng Xử lý Gateway (Event Normalizer, Risk Engine, Safety Policy)     │
├──────────────────────────────────────────────────────────────────────────┤
│ 3. Tầng Truyền thông (ESP-NOW Encrypted Fast-Path, MQTT / WebSockets)    │
├──────────────────────────────────────────────────────────────────────────┤
│ 2. Tầng Cảm biến & Edge Vision (ESP32-S3 Edge AI, PIR, Reed, Smoke Nodes)│
├──────────────────────────────────────────────────────────────────────────┤
│ 1. Tầng Phần cứng Vật lý (Vi điều khiển ESP32, Camera, Còi/Relay)       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Sơ đồ C4 Context Diagram (Mức 1)

```mermaid
graph TB
    User["Chủ nhà / Người vận hành"]
    System["Hệ thống SafeHome AI Mesh\n(Mạng An toàn Cục bộ Ưu tiên Edge)"]
    ExternalAI["Dịch vụ AI Bên ngoài Tùy chọn\n(Cloud VLM / LLM API)"]
    LocalAlarm["Còi / Relay Báo động Vật lý Local"]

    User -->|Xem Cảnh báo, Quản lý Hệ thống| System
    System -->|Kích hoạt Còi Báo động Khẩn cấp| LocalAlarm
    System -.->|Gửi Metadata Ẩn danh để Phân tích Sâu| ExternalAI
    ExternalAI -.->|Trả về Đánh giá Cố vấn| System

    classDef primary fill:#2563eb,stroke:#1d4ed8,color:#fff;
    classDef external fill:#475569,stroke:#334155,color:#fff;
    class System primary;
    class ExternalAI,LocalAlarm,User external;
```

---

## 3. Sơ đồ C4 Container Diagram (Mức 2)

```mermaid
graph TB
    subgraph Edge_Devices ["Container Thiết bị Edge"]
        CAM["Nút Vision ESP32-S3\n(C++/ESP-IDF + TFLite Micro)"]
        SENS["Nút Cảm biến Môi trường\n(C++/ESP-IDF)"]
        ALARM["Nút Còi Báo động Local\n(C++/ESP-IDF)"]
    end

    subgraph Edge_Gateway_Node ["Host Container Edge Gateway (Raspberry Pi / Mini PC)"]
        GW_RECV["Dịch vụ Nhận Tin nhắn (Message Receiver)\n(ESP-NOW Serial Bridge & MQTT Broker)"]
        GW_PROC["Engine Xử lý Sự kiện & Trạng thái (Event Engine)"]
        GW_RISK["Deterministic Risk Engine"]
        GW_POL["Safety Policy & Command Validator"]
        GW_API["REST / WebSocket API Server"]
        GW_DB[(SQLite / TimeSeries DB Local)]
    end

    subgraph Client_Applications ["Container Ứng dụng Người dùng"]
        DASH["Web Dashboard\n(React / Vite PWA)"]
    end

    %% Network Connections
    CAM -->|Gói tin Khẩn cấp Mã hóa ESP-NOW| ALARM
    CAM -->|ESP-NOW / Wi-Fi Telemetry| GW_RECV
    SENS -->|Kích hoạt Khẩn cấp ESP-NOW| ALARM
    SENS -->|ESP-NOW Telemetry| GW_RECV

    GW_RECV --> GW_PROC
    GW_PROC --> GW_RISK
    GW_RISK --> GW_POL
    GW_RISK --> GW_DB
    GW_POL -->|Lệnh Relay đã Kiểm duyệt| GW_RECV
    GW_API --> GW_DB
    GW_PROC -->|WebSocket Event Push| GW_API
    DASH <-->|HTTP REST & WS| GW_API

    classDef edge fill:#1e293b,stroke:#3b82f6,color:#fff;
    classDef gw fill:#0f172a,stroke:#8b5cf6,color:#fff;
    classDef ui fill:#1e1b4b,stroke:#f59e0b,color:#fff;

    class CAM,SENS,ALARM edge;
    class GW_RECV,GW_PROC,GW_RISK,GW_POL,GW_API,GW_DB gw;
    class DASH ui;
```

---

## 4. Sơ đồ Tuần tự Hệ thống (Sequence Diagram): Luồng Khẩn cấp End-to-End

```mermaid
sequenceDiagram
    autonumber
    actor Hazard as Nguy cơ Vật lý / Người lạ
    participant ESP32CAM as Nút Vision ESP32-S3
    participant AlarmNode as Nút Còi Local
    participant Gateway as Edge Gateway
    participant RiskEng as Risk Engine
    participant DB as Database Local
    participant UI as Web Dashboard

    Hazard->>ESP32CAM: Người xuất hiện ở khu vực hạn chế
    ESP32CAM->>ESP32CAM: Bắt Frame -> TFLite Inference (Person Score > 0.85)
    ESP32CAM->>ESP32CAM: Temporal Filter (3 frames khớp) -> Phát EVENT
    
    par Luồng Ưu tiên Khẩn cấp (<50ms)
        ESP32CAM->>AlarmNode: Gửi Payload Khẩn cấp ESP-NOW (AES-128, Seq# N)
        AlarmNode->>AlarmNode: Xác minh Thẻ AES & Sequence Number
        AlarmNode->>AlarmNode: Kích hoạt Còi / Buzzer (GPIO HIGH)
    and Luồng Báo cáo Gateway (<100ms)
        ESP32CAM->>Gateway: Gửi Event Metadata qua ESP-NOW/MQTT
        Gateway->>RiskEng: Ingest & Đánh giá Quy tắc
        RiskEng->>RiskEng: Cập nhật Điểm Rủi ro (ELEVATED -> HIGH)
        RiskEng->>DB: Lưu Bản ghi Sự kiện Bất biến
        RiskEng->>UI: Broadcast Cảnh báo Realtime qua WebSocket
    end
    
    UI->>UI: Hiển thị Cảnh báo Đỏ & Bật Âm thanh trên Dashboard
```

---

## 5. Tóm tắt Đánh đổi Kiến trúc & Hồ sơ ADR

1. **Edge Vision Độc lập vs. Stream Video Trực tiếp:**  
   * *Quyết định:* Suy luận MobileNet tại thiết bị phát ra metadata JSON sự kiện thay vì stream video 24/7.
   * *Đánh đổi:* Tiết kiệm băng thông mạng và bảo vệ quyền riêng tư, nhưng người dùng không xem được video 4K trực tiếp ngoại trừ khi có cảnh báo.
2. **Mạng ESP-NOW Mesh vs. Mạng Wi-Fi Tiêu chuẩn:**  
   * *Quyết định:* Dùng ESP-NOW cho kênh khẩn cấp, Wi-Fi cho telemetry.
   * *Đánh đổi:* Cần quản lý bảng địa chỉ MAC của peer trên từng nút ESP32, nhưng đảm bảo tín hiệu truyền dưới 50ms ngay cả khi router Wi-Fi bị sập.
