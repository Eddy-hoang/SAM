# 20 - Chiến lược Kiểm thử Hệ thống (Testing Strategy)

> **Trạng thái Tài liệu:** `[DECISION]` Kế hoạch Kiểm thử Toàn diện  

---

## 1. Kiến trúc Kiểm thử Đa tầng (Multi-Layer Testing)

```text
  ┌────────────────────────────────────────────────────────┐
  │ 10. Chạy Kịch bản Demo Cuộc thi End-to-End             │
  ├────────────────────────────────────────────────────────┤
  │ 9. Kiểm thử Độ trễ & Tải Hardware-in-the-Loop (HIL)   │
  ├────────────────────────────────────────────────────────┤
  │ 8. Kiểm thử An ninh & Thâm nhập (Replay / Spoofing)    │
  ├────────────────────────────────────────────────────────┤
  │ 7. Kiểm thử Tiêm Lỗi Mạng & Sự cố Gián đoạn (Outage)   │
  ├────────────────────────────────────────────────────────┤
  │ 6. Benchmark Độ chính xác Mô hình AI & Lượng hóa INT8  │
  ├────────────────────────────────────────────────────────┤
  │ 5. Kiểm thử Tích hợp API & Realtime WebSocket Stream   │
  ├────────────────────────────────────────────────────────┤
  │ 4. Kiểm thử Ma trận Quy tắc Deterministic Risk Engine  │
  ├────────────────────────────────────────────────────────┤
  │ 3. Kiểm thử Temporal State Machine & Engine Debounce   │
  ├────────────────────────────────────────────────────────┤
  │ 2. Kiểm thử Unit Đóng gói Frame Nhị phân ESP-NOW       │
  ├────────────────────────────────────────────────────────┤
  │ 1. Kiểm thử Unit Data Model & Schema Validation        │
  └────────────────────────────────────────────────────────┘
```

---

## 2. Ma trận Thực thi Kiểm thử (Test Execution Matrix)

| Mã Test ID | Thể loại Test | Thành phần Mục tiêu | Mô tả & Tiêu chí Nghiệm thu | Tham chiếu Yêu cầu |
| :--- | :--- | :--- | :--- | :--- |
| `TEST-AI-01` | AI Model Test | `ESP32-S3 CAM` | Kiểm tra suy luận TFLite INT8 trả về score $>0.85$ khi có người; thời gian suy luận $<150\text{ ms}$. | REQ-001, REQ-021 |
| `TEST-STATE-01`| Unit Test | `Temporal Engine` | Kiểm tra temporal state yêu cầu đủ $N=3$ frame liên tiếp trước khi phát ra `SYSTEM_EVENT`. | REQ-012 |
| `TEST-NET-01` | Protocol Test | `ESP-NOW Mesh` | Đo độ trễ truyền P2P ESP-NOW từ kích hoạt đến còi nhận ($<30\text{ ms}$ trên 100 lần thử). | REQ-002, REQ-010 |
| `TEST-SEC-01` | Security Test | `Gateway / Node` | Tiêm gói tin ESP-NOW bị replay chứa sequence number cũ; xác nhận nút hủy gói. | REQ-030 |
| `TEST-GW-01` | Integration | `Risk Engine` | Kết hợp PIR chuyển động + Vision event trong 15s; xác nhận Risk Score nhảy lên 65 (`HIGH`). | REQ-003 |
| `TEST-FAIL-01` | Resilience | `System Mesh` | Rút cáp Wi-Fi Router khi đang có alert; xác nhận còi báo động local vẫn bật qua ESP-NOW. | REQ-011 |
| `TEST-E2E-01` | End-to-End | `Toàn bộ Hệ thống`| Kích hoạt cảm biến khói $\rightarrow$ xác nhận còi kêu $<50\text{ ms}$ và UI update dashboard $<100\text{ ms}$. | Tất cả REQs |

---

## 3. Vết Thực thi Kiểm thử End-to-End Trace

```text
[Đầu vào Frame Camera] 
       │
       ▼ (TEST-AI-01: Score = 0.88)
[Temporal State Engine] 
       │
       ▼ (TEST-STATE-01: 3 Inferences Liên tiếp -> Phát Event)
[Vận chuyển Khẩn cấp ESP-NOW] 
       │
       ▼ (TEST-NET-01: Độ trễ = 14ms -> Còi Local Bật)
[Engine Xử lý Gateway] 
       │
       ▼ (TEST-GW-01: Đánh giá Rules -> Risk = HIGH -> Lưu CSDL)
[WebSocket Server] 
       │
       ▼ (TEST-UI-01: Push JSON -> Hiển thị Banner Cảnh báo Đỏ trên UI)
```
