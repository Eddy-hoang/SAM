# 02 - Yêu cầu Hệ thống (System Requirements)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Cơ sở  
> **Tiêu chuẩn Ưu tiên:** RFC 2119 (MUST, SHOULD, COULD, FUTURE)  

---

## 1. Yêu cầu Chức năng (Functional Requirements - FR)

### REQ-001: Suy luận Vision AI tại Edge (Edge Vision Inference)
* **Mô tả:** Nút ESP32-S3 CAM BẮT BUỘC thực hiện suy luận computer vision cục bộ để phát hiện người lạ hoặc dấu hiệu thảm họa mà không được stream video ra máy chủ bên ngoài.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Yêu cầu An toàn Cốt lõi
* **Tiêu chí Nghiệm thu:** Tốc độ xử lý khung hình $\ge 5\text{ FPS}$ ở độ phân giải QVGA đi kèm score xác suất đầu ra cho mỗi frame.
* **Thành phần Liên quan:** `ESP32-S3 CAM Firmware`, `AI Pipeline`

### REQ-002: Kích hoạt Khẩn cấp Trực tiếp qua ESP-NOW
* **Mô tả:** Khi phát hiện nguy hiểm nghiêm trọng (như có khói hoặc xâm nhập trái phép), nút cảm biến hoặc camera BẮT BUỘC gửi một gói tin khẩn cấp mã hóa ESP-NOW trực tiếp tới Nút Còi báo động Local và Gateway.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Đặc tả Kênh Khẩn cấp (Emergency Safety Path)
* **Tiêu chí Nghiệm thu:** Nút còi báo động nhận tín hiệu và bật còi trong vòng $<50\text{ ms}$ tính từ thời điểm tạo sự kiện cục bộ, không phụ thuộc vào trạng thái router Wi-Fi.
* **Thành phần Liên quan:** `Sensor Node`, `Local Alarm Node`, `ESP-NOW Mesh`

### REQ-003: Đánh giá Rủi ro & Gom nhóm Sự kiện tại Gateway
* **Mô tả:** Edge Gateway BẮT BUỘC tổng hợp các sự kiện theo thời gian từ nhiều cảm biến/camera, tính toán điểm rủi ro động (0 đến 100), và xác định cấp độ đe dọa của hệ thống (NORMAL, ELEVATED, HIGH, CRITICAL).
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Đặc tả Kiến trúc Gateway
* **Tiêu chí Nghiệm thu:** Khi có nhiều sự kiện kết hợp (ví dụ: PIR chuyển động + Camera phát hiện người trong vòng 10 giây), cấp độ đe dọa phải leo thang từ ELEVATED lên HIGH trong vòng $<100\text{ ms}$.
* **Thành phần Liên quan:** `Edge Gateway`, `Risk Engine`

### REQ-004: Dashboard Giám sát Realtime
* **Mô tả:** Hệ thống BẮT BUỘC cung cấp giao diện web dashboard cục bộ hiển thị trạng thái thiết bị realtime, nhật ký sự kiện, đồng hồ đo mức độ rủi ro, và các nút ghi đè an toàn thủ công.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Yêu cầu Giao diện Người dùng
* **Tiêu chí Nghiệm thu:** UI cập nhật qua WebSockets trong vòng $<100\text{ ms}$ tính từ khi Gateway phát sự kiện; dashboard hoạt động hoàn toàn offline.
* **Thành phần Liên quan:** `Realtime Server`, `Web Dashboard`

### REQ-005: Phân tích Rủi ro Hỗ trợ bởi AI (LLM Advisory)
* **Mô tả:** Gateway CÓ THỂ truy vấn dịch vụ LLM/VLM local hoặc cloud tùy chọn để tạo bản tóm tắt an toàn bằng ngôn ngữ tự nhiên và đưa ra khuyến nghị hành động cho người vận hành.
* **Độ ưu tiên:** `SHOULD`
* **Nguồn:** Khái niệm Trí tuệ Cố vấn (Advisory Intelligence)
* **Tiêu chí Nghiệm thu:** Đầu ra của LLM phải được đánh dấu nghiêm ngặt là "ADVISORY ONLY" và không bao giờ được phép trực tiếp kích hoạt GPIO phần cứng.
* **Thành phần Liên quan:** `AI Service`, `Risk Engine`

---

## 2. Yêu cầu Phi Chức năng (Non-Functional Requirements - NFR)

### REQ-010: Độ trễ Phản ứng Khẩn cấp (Emergency Response Latency)
* **Mô tả:** Độ trễ end-to-end từ khi cảm biến vật lý kích hoạt (như công tắc cửa mở hoặc phát hiện khói) đến khi còi báo động local bật BẮT BUỘC KHÔNG VƯỢT QUÁ 100 miligiây.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Ràng buộc An toàn Cốt lõi
* **Tiêu chí Nghiệm thu:** Được kiểm chứng qua Oscilloscope / Logic Analyzer qua 100 lần thử nghiệm liên tiếp.
* **Thành phần Liên quan:** `ESP-NOW Network`, `Local Alarm Node`

### REQ-011: Khả năng Vận hành Offline Độc lập
* **Mô tả:** Tất cả các tính năng an ninh, phát hiện, lưu log và bật còi báo động BẮT BUỘC hoạt động liên tục ngay cả khi mất toàn bộ kết nối WAN/Internet.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Ràng buộc Độ tin cậy (Resilience Constraint)
* **Tiêu chí Nghiệm thu:** Hệ thống duy trì đầy đủ vòng lặp cảnh báo cục bộ khi rút cáp mạng Ethernet WAN khỏi Gateway.
* **Thành phần Liên quan:** `Edge Gateway`, `ESP-NOW Mesh`, `Local DB`

### REQ-012: Tỷ lệ Giảm Báo động Giả (False Alarm Mitigation Rate)
* **Mô tả:** Hệ thống BẮT BUỘC giảm ít nhất 95% báo động giả do vật cản tạm thời, thay đổi ánh sáng bóng đổ hoặc lỗi suy luận 1 frame so với việc phát hiện trực tiếp theo từng frame.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Chất lượng Trải nghiệm Người dùng
* **Tiêu chí Nghiệm thu:** Bộ lọc Temporal Filter Engine yêu cầu $N=3$ frame phát hiện dương tính liên tiếp trước khi chuyển sang trạng thái `OBSERVED`.
* **Thành phần Liên quan:** `Event Generator`, `AI Pipeline`

---

## 3. Yêu cầu AI & Phần cứng (AI & Hardware Requirements)

### REQ-020: Ngân sách Bộ nhớ & Xử lý trên ESP32-S3
* **Mô tả:** Pipeline suy luận vision BẮT BUỘC vận hành trong phạm vi 8MB PSRAM external và 512KB SRAM internal, dành riêng ít nhất 150KB SRAM cho luồng mạng Wi-Fi/ESP-NOW.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Giới hạn Phần cứng
* **Tiêu chí Nghiệm thu:** Không xảy ra lỗi cạn bộ nhớ Heap hoặc vỡ bus PSRAM trong 24 giờ suy luận liên tục.
* **Thành phần Liên quan:** `ESP32-S3 Edge Device`

### REQ-021: Lượng hóa & Kích thước Mô hình AI
* **Mô tả:** Mô hình vision triển khai lên ESP32-S3 BẮT BUỘC là dạng TensorFlow Lite Micro đã được lượng hóa INT8, kích thước file binary không vượt quá 2.5MB.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Ngân sách Edge AI
* **Tiêu chí Nghiệm thu:** Độ chính xác của mô hình lượng hóa nằm trong khoảng 3% so với baseline FP32 trên tập dữ liệu đánh giá.
* **Thành phần Liên quan:** `AI Pipeline`

---

## 4. Yêu cầu An ninh & Quyền riêng tư (Security & Privacy Requirements)

### REQ-030: Bảo vệ Chống Bắt lại Gói tin over ESP-NOW
* **Mô tả:** Các gói tin khẩn cấp truyền qua ESP-NOW BẮT BUỘC bao gồm một sequence counter 32-bit tăng dần và thẻ xác thực payload AES-128 để chống tấn công replay attack.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Threat Model An ninh
* **Tiêu chí Nghiệm thu:** Các gói tin replay có sequence number cũ phải bị Gateway/Alarm node hủy bỏ ngay lập tức.
* **Thành phần Liên quan:** `ESP-NOW Emergency Path`, `Security Module`

### REQ-031: Quyền riêng tư Camera trên Thiết bị (Zero Video Streaming)
* **Mô tả:** Bộ đệm frame camera thô BẮT BUỘC phải được xử lý trong RAM cục bộ và xóa ngay lập tức sau khi suy luận. Hình ảnh thô KHÔNG ĐƯỢC lưu vào ổ đĩa local hoặc truyền qua mạng trừ khi được yêu cầu trong trạng thái ALARM đang kích hoạt.
* **Độ ưu tiên:** `MUST`
* **Nguồn:** Chính sách Quyền riêng tư
* **Tiêu chí Nghiệm thu:** Kiểm tra gói tin mạng (Packet Analysis) xác nhận không có luồng dữ liệu video nào được truyền đi trong trạng thái bình thường.
* **Thành phần Liên quan:** `ESP32-S3 CAM Firmware`, `Privacy Module`

---

## 5. Tóm tắt Ma trận Truy xuất Nguồn gốc (Traceability Matrix)

| Requirement ID | Primary Component | Verification Test ID | Competition Demo Step |
| :--- | :--- | :--- | :--- |
| **REQ-001** | `ESP32-S3 CAM Firmware` | `TEST-AI-01` | `DEMO-STEP-02` |
| **REQ-002** | `ESP-NOW Mesh` | `TEST-NET-02` | `DEMO-STEP-03` |
| **REQ-003** | `Gateway Risk Engine` | `TEST-GW-01` | `DEMO-STEP-04` |
| **REQ-004** | `Web Dashboard` | `TEST-UI-01` | `DEMO-STEP-05` |
| **REQ-010** | `Local Alarm Node` | `TEST-PERF-01` | `DEMO-STEP-03` |
| **REQ-011** | `Edge Gateway` | `TEST-FAIL-01` | `DEMO-STEP-06` |
| **REQ-030** | `Security Module` | `TEST-SEC-01` | `DEMO-STEP-07` |
