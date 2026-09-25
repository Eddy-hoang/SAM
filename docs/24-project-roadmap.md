# 24 - Lộ trình Dự án & Định nghĩa MVP (Project Roadmap)

> **Trạng thái Tài liệu:** `[DECISION]` Lộ trình Thực thi Phân giai đoạn  
> **Cột mốc Mục tiêu:** Bài Trình diễn Trực tiếp tại Cuộc thi Danang AI4Life  

---

## 1. Định nghĩa Phạm vi MVP vs. Mục tiêu Mở rộng (Stretch Goals)

### Phạm vi MVP (Minimum Viable Product) `[BẮT BUỘC HOÀN THÀNH CHO DEMO]`
```text
ESP32 CAM (Person Detect) ──> ESP-NOW Frame ──> Nút Còi Báo động (<50ms Kêu Còi)
                                  │
                                  └───> Gateway (Risk Score 75) ──> Web UI Red Alert
```
* **Phần cứng:** 1 ESP32-S3 CAM, 1 Nút Cảm biến Khói, 1 Nút Còi Local, 1 Gateway (Raspberry Pi/Laptop).
* **AI:** Mô hình MobileNet-V2 INT8 phát hiện sự xuất hiện của con người chạy local 5 FPS.
* **Mạng:** Kênh khẩn cấp ESP-NOW chạy hoàn toàn offline không cần Internet.
* **Gateway & UI:** Deterministic Risk Engine tính điểm đe đọa, WebSocket stream alert đỏ lên React dashboard.

### Mục tiêu Mở rộng (Stretch Goals) `[TƯƠNG LAI / TÙY CHỌN]`
* Tích hợp LLM tạo âm thanh cảnh báo bằng ngôn ngữ tự nhiên ("Phát hiện người lạ gần cửa sau lúc 2:15 sáng").
* Theo dõi không gian đa camera trên 4+ phòng.
* Thông báo Mobile Push Notifications (qua Apple APNS / Firebase FCM).

---

## 2. Lộ trình Triển khai Phân giai đoạn (Phase 0 đến Phase 12)

### Phase 0: Tài liệu Kỹ thuật & Blueprint `[ĐÃ HOÀN THÀNH]`
* **Mục tiêu:** Tạo bộ tài liệu kiến trúc hệ thống hoàn chỉnh mà chưa cần viết code ứng dụng.
* **Kết quả:** 28 file Markdown tài liệu, sơ đồ Mermaid, schemas, ADRs.
* **Tiêu chí Nghiệm thu:** Đạt yêu cầu đánh giá kiến trúc không còn điểm mơ hồ.

### Phase 1: Hardware Bring-Up & Kiểm tra Nguồn
* **Mục tiêu:** Kiểm tra độ ổn định điện áp và nạp bootloader lên chip ESP32.
* **Nhiệm vụ:** Kiểm tra tụ lọc nguồn; xác nhận nhận diện 8MB PSRAM.
* **Kết quả:** Code mẫu ESP-IDF hello-world chạy kèm log chẩn đoán PSRAM thành công.

### Phase 2: Bắt hình Camera ESP32-S3
* **Mục tiêu:** Cấu hình driver camera OV2640 qua giao tiếp DVP với bộ đệm kép DMA double-buffering.
* **Kết quả:** Bắt mượt khung hình $320 \times 240$ RGB565 lưu trong PSRAM.

### Phase 3: Kênh Mạng Khẩn cấp ESP-NOW
* **Mục tiêu:** Lập trình phát/nhận gói tin nhị phân mã hóa ESP-NOW Peer-to-Peer giữa các nút.
* **Kết quả:** Gói tin khẩn cấp gửi từ Cảm biến tới Nút Còi ($<20\text{ ms}$ độ trễ).

### Phase 4: Suy luận TFLite Micro trên Thiết bị & Temporal Filter
* **Mục tiêu:** Nạp mô hình MobileNet INT8 và tích hợp bộ lọc thời gian 3 frame.
* **Kết quả:** Nút phát sự kiện JSON `HUMAN_DETECTION` chỉ khi có 3 frame liên tiếp khớp.

### Phase 5: Ingest & Lưu trữ dữ liệu tại Edge Gateway
* **Mục tiêu:** Xây dựng Node.js Gateway receiver, bộ khử trùng lặp và lưu trữ CSDL SQLite.
* **Kết quả:** Gateway nhận sự kiện, xác thực schema, lưu SQLite trong vòng $<10\text{ ms}$.

### Phase 6: Deterministic Risk Engine & Tường lửa Safety Policy
* **Mục tiêu:** Lập trình quy tắc gom nhóm sự kiện có trọng số và tầng kiểm duyệt Safety Policy.
* **Kết quả:** Risk Engine chuyển cấp đe dọa từ NORMAL lên HIGH khi có kết hợp sự kiện.

### Phase 7: REST API & Server WebSocket Realtime
* **Mục tiêu:** Xây dựng các API endpoint và WebSocket broker broadcast tin nhắn.
* **Kết quả:** Gateway broadcast JSON update lên channel WebSocket `system:risk`.

### Phase 8: Giao diện Web Dashboard Realtime
* **Mục tiêu:** Dựng dashboard React + Vite hiển thị đồng hồ rủi ro và nhật ký sự kiện sống.
* **Kết quả:** UI cập nhật trong vòng $<50\text{ ms}$ tính từ khi WebSocket push.

### Phase 9: Tích hợp Hệ thống & Kiểm thử End-to-End
* **Mục tiêu:** Kết nối tất cả các nút phần cứng và Gateway thành mạng trình diễn hoàn chỉnh.
* **Kết quả:** Chạy thành công toàn bộ ma trận kiểm thử (`TEST-E2E-01`).

### Phase 10: Kiểm thử Kháng lỗi & Thắt chặt An ninh
* **Mục tiêu:** Thử nghiệm rút cáp Wi-Fi, mất nguồn điện và kiểm tra sequence number chống replay.
* **Kết quả:** Gói tin replay bị hủy; hệ thống chạy 100% offline khi Wi-Fi sập.

### Phase 11: Chuẩn bị Kịch bản Trình diễn & Thuyết minh
* **Mục tiêu:** Luyện tập các kịch bản demo trực tiếp trước Ban giám khảo.
* **Kết quả:** Thực thi mượt mà 4 kịch bản trình diễn.

### Phase 12: Thuyết minh Cuộc thi Danang AI4Life
* **Mục tiêu:** Trình diễn dự án SafeHome AI Mesh trực tiếp trước Ban giám khảo.
