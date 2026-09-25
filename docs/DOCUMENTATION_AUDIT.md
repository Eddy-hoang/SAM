# SafeHome AI Mesh: Báo cáo Đánh giá Tài liệu & Đánh giá Kiến trúc (Documentation Audit)

> **Trạng thái Tài liệu:** `[DECISION]` Báo cáo Đánh giá Kiến trúc & Tóm tắt Audit  
> **Vai trò Kiểm toán:** Senior System Architect & Technical Lead  
> **Ngày:** 25 tháng 9 năm 2026  

---

## 1. Phân tích Trạng thái Đặc tả Hệ thống

### 1.1 Các Phân hệ Đã Đặc tả Hoàn chỉnh (`READY`)
* **Kiến trúc Hệ thống & Thành phần:** Các sơ đồ C4, phân tách 6 tầng kiến trúc và đặc tả 8 điểm cho các thành phần được định nghĩa hoàn chỉnh trong [`03-system-architecture.md`](file:///d:/SAM/docs/03-system-architecture.md) và [`04-component-architecture.md`](file:///d:/SAM/docs/04-component-architecture.md).
* **Kênh Khẩn cấp ESP-NOW:** Cấu trúc gói tin C-struct nhị phân (64 bytes), cơ chế sequence counter chống phát lại (anti-replay), quản lý khóa mã hóa LMK và luồng dự phòng unicast/broadcast được định nghĩa 100% trong [`09-esp-now-emergency-path.md`](file:///d:/SAM/docs/09-esp-now-emergency-path.md).
* **Temporal State Engine & Event Schema:** Chuyển trạng thái Frame $\rightarrow$ Observation $\rightarrow$ Event, ngưỡng tin cậy ($0.75$), số frame liên tiếp ($N=3$), bộ đếm cooldown ($5000\text{ ms}$) và toàn bộ JSON schema được đặc tả trong [`07-event-generation.md`](file:///d:/SAM/docs/07-event-generation.md).
* **Deterministic Risk Engine & Safety Policy:** Bảng ánh xạ cấp độ đe dọa (NORMAL đến CRITICAL), công thức gom nhóm sự kiện có trọng số và các Bất biến An toàn (Safety Invariants) được định nghĩa trong [`12-risk-engine.md`](file:///d:/SAM/docs/12-risk-engine.md).
* **Mô hình Dữ liệu & API Specification:** Sơ đồ quan hệ ERD, các chỉ mục bảng CSDL, JSON schemas, REST endpoints và định dạng push WebSocket được đặc tả trong [`13-data-model.md`](file:///d:/SAM/docs/13-data-model.md), [`14-api-specification.md`](file:///d:/SAM/docs/14-api-specification.md), và [`15-realtime-communication.md`](file:///d:/SAM/docs/15-realtime-communication.md).
* **An ninh & Quyền riêng tư Blueprint:** Ma trận mối đe dọa STRIDE, Flash Encryption, Secure Boot và các quy tắc quyền riêng tư zero-video-streaming được định nghĩa trong [`16-security.md`](file:///d:/SAM/docs/16-security.md) và [`17-privacy.md`](file:///d:/SAM/docs/17-privacy.md).
* **Kịch bản Demo Cuộc thi:** 4 kịch bản trình diễn chi tiết được thiết lập trong [`25-demo-scenario.md`](file:///d:/SAM/docs/25-demo-scenario.md).

### 1.2 Các Phân hệ Đang Đặc tả Một phần (`IN DESIGN`)
* **Tranh chấp Bus Bộ nhớ PSRAM ESP32-S3:** Phân chia truy cập bộ nhớ của 2 lõi LX7 (Core 0 Wi-Fi vs Core 1 DMA Camera) đã được lập sơ đồ target SRAM/PSRAM, nhưng cần chạy micro-benchmark thực tế trên phần cứng (`[VERIFY]`).
* **Trọng số AI & Tập dữ liệu Huấn luyện:** Các chỉ số mục tiêu cho mô hình MobileNet-V2 INT8 đã được chốt, nhưng việc chọn tập dữ liệu representative cho mẫu cháy/khói indoor vẫn cần thử nghiệm (`[EXPERIMENT]`).

### 1.3 Các Mục Cần Kiểm chứng Phần cứng (`[VERIFY]`)
1. `[VERIFY-01]` Đo độ trễ truyền gói tin unicast ESP-NOW bằng miligiây từ ESP32-S3 tới Nút Còi trong môi trường có nhiễu sóng 2.4GHz mạnh.
2. `[VERIFY-02]` Benchmark tốc độ FPS suy luận TFLite Micro INT8 trên ESP32-S3 khi bật tăng tốc tập lệnh SIMD ESP-NN.
3. `[VERIFY-03]` Kiểm tra cân bằng nhiệt lượng của chip ESP32-S3 N16R8 khi chạy liên tục 240MHz lõi kép trong hộp nhựa in 3D.

### 1.4 Các Giả định Cơ sở (`[ASSUMPTION]`)
1. `[ASSUMPTION-01]` Ánh sáng tự nhiên trong phòng đủ cho cảm biến OV2640 bắt hình ban ngày mà không cần bật thêm LED hồng ngoại.
2. `[ASSUMPTION-02]` Phạm vi triển khai mục tiêu (nhà 2-3 tầng) nằm trong tầm truyền sóng không dây 30 mét line-of-sight của ESP-NOW 2.4GHz.

---

## 2. Top 3 Rủi ro Kiến trúc & Chiến lược Giảm thiểu

1. **Rủi ro 1: Nhiễu Sóng RF 2.4GHz & Tự động Nhảy Kênh Wi-Fi**
   * *Mô tả:* Nếu router Wi-Fi nhà tự đổi kênh RF, gói tin ESP-NOW truyền trên Channel 6 cố định sẽ bị ngắt.
   * *Giảm thiểu:* Khóa cứng chip Wi-Fi PHY ESP32 ở Kênh 6 cố định (`WIFI_SECOND_CHAN_NONE`) và duy trì đường còi báo động local backup.
2. **Rủi ro 2: Tranh chấp Bus PSRAM khi Camera DMA Bắt hình Tải cao**
   * *Mô tả:* Việc ghi DMA camera và đọc tensor TFLite đồng thời trên Octal PSRAM bus có thể gây rớt khung hình.
   * *Giảm thiểu:* Cấp phát Tensor Arena 150KB của TFLite trong bộ nhớ internal SRAM tốc độ cao, dành riêng PSRAM cho bộ đệm frame.
3. **Rủi ro 3: Báo động Giả Mô hình AI do Ánh sáng Mặt trời Đột ngột**
   * *Mô tả:* Bóng đổ di chuyển hoặc vệt nắng gắt bất ngờ gây ra nhận diện sai trên 1 frame.
   * *Giảm thiểu:* Bắt buộc áp dụng bộ lọc thời gian $N=3$ frame liên tiếp và bộ đếm trễ Cooldown Hysteresis ($5000\text{ ms}$).

---

## 3. Tóm tắt Độ sẵn sàng Triển khai (Implementation Readiness Summary)

```text
================================================================================
                    TÓM TẮT ĐỘ SẴN SÀNG TRIỂN KHAI
================================================================================
  Mảng Phân hệ              Trạng thái Đánh giá   Tài liệu Tham chiếu Chính
--------------------------------------------------------------------------------
  Architecture Readiness   READY (100%)          docs/03-system-architecture.md
  Hardware Readiness       READY (95%)           docs/23-hardware-bom.md
  AI Readiness             READY (90%)           docs/06-ai-pipeline.md
  Network Readiness        READY (100%)          docs/08-network-architecture.md
  Backend Readiness        READY (100%)          docs/10-edge-gateway.md
  Frontend Readiness       READY (100%)          docs/15-realtime-communication.md
  Deployment Readiness     READY (95%)           docs/21-deployment.md
  Testing Readiness        READY (100%)          docs/20-testing-strategy.md
  Demo Readiness           READY (100%)          docs/25-demo-scenario.md
================================================================================
```

---

## 4. 3 Việc Cần Làm Đầu Tiên Khi Bắt Đầu Giai đoạn Lập trình

Khi chuyển từ Giai đoạn Tài liệu Kỹ thuật sang Thực thi Source Code, team kỹ thuật BẮT BUỘC thực hiện 3 nhiệm vụ đầu tiên theo đúng thứ tự:

1. **Hành động 1 (Hardware Bring-up Nhúng):**  
   Nạp template firmware ESP-IDF v5.1 lên chip ESP32-S3 N16R8, bật Octal PSRAM trong `sdkconfig`, xác nhận cấp phát đủ 8MB PSRAM và đo bộ nhớ free heap cơ sở.
2. **Hành động 2 (Kiểm chứng ESP-NOW Fast-Path):**  
   Lập trình gói tin C-struct nhị phân phát và nhận trên 2 nút ESP32, khóa kênh Wi-Fi ở Kênh 6, và đo độ trễ kích hoạt còi báo động từ lúc nhấn nút bằng Dao động ký.
3. **Hành động 3 (Khung Gateway Ingest Engine & CSDL):**  
   Tạo khung dự án Node.js TypeScript cho Gateway với Mosquitto MQTT subscriber, script khởi tạo CSDL SQLite (bảng `events` & `alerts`), và REST endpoint `/api/events`.
