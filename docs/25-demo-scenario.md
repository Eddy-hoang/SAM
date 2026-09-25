# 25 - Kịch bản Trình diễn Cuộc thi & Walkthrough (Demo Scenarios)

> **Trạng thái Tài liệu:** `[DECISION]` Kịch bản Trình diễn Trực tiếp  
> **Sự kiện Mục tiêu:** Cuộc thi Danang AI4Life  

---

## 1. Tổng quan Kịch bản Trình diễn Demo

Để chứng minh khả năng của hệ thống trước Ban giám khảo cuộc thi trong dưới 7 phút, bài thuyết trình thực thi 4 kịch bản chạy live thực tế:

```text
  [Scenario 1: Idle Bình thường] ──> [Scenario 2: Người lạ Đột nhập] ──> [Scenario 3: Sự cố Cháy/Khói] ──> [Scenario 4: Giả lập Sập Router Wi-Fi]
```

---

## 2. Kịch bản Chi tiết (Scenario Scripts)

### Scenario 1: Trạng thái Hệ thống Idle Bình thường
* **Trạng thái Ban đầu:** Tất cả các nút được bật nguồn, kết nối Wi-Fi, Điểm Rủi ro Hệ thống = `0` (`NORMAL`).
* **Kích hoạt (Trigger):** Người thuyết minh di chuyển tự nhiên ở khu vực không bị hạn chế (Background).
* **Sự kiện Kỳ vọng:** Gói tin `DEVICE_HEARTBEAT` phát định kỳ; điểm rủi ro giữ nguyên `NORMAL`.
* **Hành động Kỳ vọng:** Không có.
* **Giao diện UI Kỳ vọng:** Dashboard hiển thị đồng hồ màu xanh; icon các nút sáng xanh.
* **Log Kỳ vọng:** `[INFO] Heartbeat received from ESP32CAM-ZONE1`.

---

### Scenario 2: Phát hiện Người lạ Đột nhập (Intrusion Flow)
* **Trạng thái Ban đầu:** Risk Level = `NORMAL` ($S=0$).
* **Kích hoạt (Trigger):** Người thuyết minh bước vào khu vực giới hạn của camera.
* **Sự kiện Kỳ vọng:**
  * Frame 1 & 2: Score = 0.88 (Observation được giữ trong RAM).
  * Frame 3: Temporal Engine xác nhận đủ số frame $\rightarrow$ Phát `HUMAN_DETECTION` ($Confidence = 0.91$).
* **Rủi ro Kỳ vọng:** Risk Engine gom nhóm sự kiện $\rightarrow$ Điểm Rủi ro nhảy lên `65` (`HIGH`).
* **Hành động Kỳ vọng:** Gateway phát tín hiệu chiming; Dashboard phát tiếng kêu beep.
* **Giao diện UI Kỳ vọng:** Đồng hồ Dashboard quét sang màu Cam `HIGH`; thẻ snapshot camera hiện lên.
* **Log Kỳ vọng:** `[INFO] EVENT_TRIGGERED: HUMAN_DETECTION (Confidence: 0.91)`.

---

### Scenario 3: Sự cố Khẩn cấp Khói/Cháy (<50ms Fast-Path)
* **Trạng thái Ban đầu:** Risk Level = `HIGH` ($S=65$).
* **Kích hoạt (Trigger):** Đưa nguồn khói thử nghiệm lại gần Nút Cảm biến Khói MQ-2.
* **Sự kiện Kỳ vọng:** Cảm biến đọc ADC vượt ngưỡng 2500 $\rightarrow$ Tạo `SMOKE_DETECTION`.
* **Rủi ro Kỳ vọng:** Điểm Rủi ro lập tức bị ghi đè lên `100` (`CRITICAL`).
* **Hành động Kỳ vọng:**
  * **Luồng Khẩn cấp ESP-NOW (<50ms):** Gói tin P2P trực tiếp gửi tới Nút Còi Local $\rightarrow$ **Còi Báo động 105dB Kêu Tức thì**.
  * **Luồng Báo cáo Gateway (<100ms):** Sự kiện được lưu CSDL; kích hoạt trạng thái UI khẩn cấp.
* **Giao diện UI Kỳ vọng:** Màn hình chớp Đỏ; Banner Khẩn cấp Toàn màn hình hiện "CRITICAL SMOKE DETECTED".
* **Log Kỳ vọng:** `[CRITICAL] ESP-NOW Direct Emergency Frame Received -> Siren Actuated (14ms)`.

---

### Scenario 4: Sự cố Sập Router Wi-Fi ("Kịch bản Chứng minh Thực chiến")
* **Trạng thái Ban đầu:** Còi Báo động Vật lý đang kêu trong Scenario 3.
* **Kích hoạt (Trigger):** Người thuyết minh rút trực tiếp cáp nguồn của Router Wi-Fi AP (Sập Wi-Fi hoàn toàn).
* **Hành động Kỳ vọng:**
  * Người thuyết minh kích hoạt tiếp cảm biến công tắc cửa phụ.
  * Nút cảm biến không thể nối Wi-Fi, lập tức chuyển đổi fallback sang chế độ phát unicast ESP-NOW.
  * **Còi Báo động Vật lý vẫn tiếp tục kêu liên tục** mặc dù router Wi-Fi đã bị ngắt điện hoàn toàn.
* **Giao diện UI Kỳ vọng:** Dashboard báo "Gateway Offline - Direct Mesh Alarm Active".
* **Tác động tới Ban Giám khảo:** Chứng minh thuyết phục hệ thống là **Edge-First** và không bị phụ thuộc vào độ ổn định của router Wi-Fi hay Cloud đối với các tính năng an toàn tính mạng.
