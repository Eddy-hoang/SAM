# 00 - Tổng quan Dự án (Project Overview)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Cơ sở  
> **Đối tượng Hướng tới:** Tất cả các vai trò kỹ thuật, Ban giám khảo Cuộc thi, Technical Leads  

---

## 1. Tầm nhìn & Mục đích (Vision & Purpose)

**SafeHome AI Mesh** là một hệ thống an toàn nhà ở ưu tiên xử lý tại Edge, tôn trọng quyền riêng tư và hoạt động thông minh. Hệ thống được thiết kế để xử lý các thảm họa vật lý và an ninh môi trường nghiêm trọng (như người lạ đột nhập trái phép, phát hiện khói/cháy, rò rỉ khí gas, xâm nhập vi phạm ranh giới) với độ tin cậy cao và độ trễ cực thấp.

Hệ thống được thiết kế đặc biệt cho các điều kiện thực tế khắt khe: nơi kết nối Internet có thể bị gián đoạn, mạng Wi-Fi nội bộ bị nghẽn, và các dịch vụ Cloud mang lại độ trễ không thể chấp nhận được cùng rủi ro rò rỉ quyền riêng tư.

---

## 2. Đối tượng Sử dụng & Môi trường Vận hành

### Đối tượng Sử dụng (Target User Persona)
* **Chính (Primary):** Chủ nhà và người cư trú mong muốn nhận cảnh báo an toàn tức thì, độ tin cậy cao mà không cần trả phí thuê bao định kỳ hoặc stream video liên tục lên Cloud.
* **Phụ (Secondary):** Quản lý văn phòng nhỏ / SOHO cần giám sát an ninh và an toàn truy cập cục bộ.
* **Ban Đánh giá (Evaluators):** Ban giám khảo cuộc thi **Danang AI4Life** đánh giá tính đổi mới sáng tạo, khả năng thực thi Edge AI, kiến trúc phân tán và tính rõ ràng của bản thuyết minh kỹ thuật.

### Môi trường Vận hành (Operational Environment)
* **Không gian Vật lý:** Nhà ở từ 2–3 tầng hoặc căn hộ chung cư điển hình.
* **Bối cảnh Mạng:** Môi trường indoor phức tạp có nhiễu sóng Wi-Fi, tường bê tông dày và rủi ro mất điện nguồn AC.
* **Dấu chân Phần cứng (Hardware Footprint):** Các nút vi điều khiển công suất thấp (ESP32-S3, ESP32-C3) kết hợp với một Edge Gateway (Raspberry Pi / Mini PC).

---

## 3. Phạm vi Dự án: Scope vs. Non-Scope

### Trong Phạm vi (In-Scope - Những gì SafeHome AI Mesh Xây dựng)
* Vision AI trực tiếp trên thiết bị để phát hiện người & thảm họa bằng ESP32-S3 CAM.
* Kênh truyền phát cảnh báo khẩn cấp độ trễ thấp (<50ms) qua ESP-NOW mã hóa Peer-to-Peer.
* Pipeline tạo sự kiện theo thời gian (Temporal Event Generation) để lọc nhiễu camera và loại bỏ thông báo rác.
* Edge Gateway cục bộ chứa Deterministic Risk Engine và CSDL Time-Series local.
* Tầng kiểm duyệt Safety Policy ngăn chặn tuyệt đối các đầu ra bất định của AI trực tiếp kích hoạt phần cứng GPIO.
* Web Dashboard giám sát realtime hiển thị luồng alert sống, chỉ số sức khỏe hệ thống và bản đồ topology thiết bị.

### Ngoài Phạm vi (Out-of-Scope - Những gì Dự án KHÔNG Làm)
* Hệ thống ghi hình/stream video 4K 24/7 liên tục lên Cloud (Thay thế NVR).
* Tự động đấu nối/gọi điện trực tiếp tới các dịch vụ cứu hộ công cộng (như tự động gọi 114/113).
* Phụ thuộc vào các chip chứng thực phần cứng độc quyền (như Apple HomeKit MFi).
* Nhận diện khuôn mặt chi tiết hoặc theo dõi sinh trắc học (do giới hạn RAM trên ESP32-S3 và rủi ro quyền riêng tư).

---

## 4. Sơ đồ Chuyển đổi từ Vấn đề đến Triển khai

```text
    ┌────────────────────────────────────────────────────────┐
    │                      Vấn đề                            │
    │  Độ trễ Cloud, rủi ro mất mạng Wi-Fi, báo động giả AI  │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                     Yêu cầu                            │
    │ Phản ứng dưới 1s offline, 0 báo động giả, đè nén an    │
    │ toàn định tính (deterministic safety override)         │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                 Khả năng Hệ thống                      │
    │ Mạng kết hợp kép (ESP-NOW + Wi-Fi), Edge AI state      │
    │ engine, Gateway Risk Engine                            │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                    Triển khai                          │
    │ ESP32-S3 TFLite-Micro model, ESP-NOW MAC peer relay,   │
    │ Deterministic Rule Evaluator, Gateway WS server        │
    └────────────────────────────────────────────────────────┘
```

---

## 5. Hướng dẫn Onboarding: "Tôi Nên Bắt đầu từ Đâu?"

Nếu bạn mới tham gia dự án, hãy đi theo lộ trình theo chuyên môn dưới đây:

| Vai trò | Bước 1 | Bước 2 | Bước 3 |
| :--- | :--- | :--- | :--- |
| **Embedded Dev** | Đọc [`05-esp32-edge-device.md`](file:///d:/SAM/docs/05-esp32-edge-device.md) | Nghiên cứu [`09-esp-now-emergency-path.md`](file:///d:/SAM/docs/09-esp-now-emergency-path.md) | Kiểm tra [`23-hardware-bom.md`](file:///d:/SAM/docs/23-hardware-bom.md) |
| **AI / ML Dev** | Đọc [`06-ai-pipeline.md`](file:///d:/SAM/docs/06-ai-pipeline.md) | Nghiên cứu [`07-event-generation.md`](file:///d:/SAM/docs/07-event-generation.md) | Xem [`adr/0003-ai-inference-strategy.md`](file:///d:/SAM/docs/adr/0003-ai-inference-strategy.md) |
| **Backend Dev** | Đọc [`03-system-architecture.md`](file:///d:/SAM/docs/03-system-architecture.md) | Nghiên cứu [`10-edge-gateway.md`](file:///d:/SAM/docs/10-edge-gateway.md) & [`12-risk-engine.md`](file:///d:/SAM/docs/12-risk-engine.md) | Xem [`13-data-model.md`](file:///d:/SAM/docs/13-data-model.md) & [`14-api-specification.md`](file:///d:/SAM/docs/14-api-specification.md) |
| **Frontend Dev** | Đọc [`15-realtime-communication.md`](file:///d:/SAM/docs/15-realtime-communication.md) | Nghiên cứu [`14-api-specification.md`](file:///d:/SAM/docs/14-api-specification.md) | Xem [`25-demo-scenario.md`](file:///d:/SAM/docs/25-demo-scenario.md) |
