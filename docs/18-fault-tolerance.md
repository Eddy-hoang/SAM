# 18 - Khả năng Kháng lỗi & Ma trận Phục hồi (Fault Tolerance)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Khả năng Chịu lỗi & Kịch bản Thất bại  

---

## 1. Ma trận Kịch bản Thất bại & Hành động Phục hồi Hệ thống

| Kịch bản Thất bại (Failure Mode) | Cơ chế Phát hiện (Detection) | Hành động Tự động Phục hồi | Tác động tới Người dùng / Hệ thống |
| :--- | :--- | :--- | :--- |
| **Sự cố Router Wi-Fi AP Sập** | Nút ngắt kết nối MQTT ping timeout (10s) | Nút cảm biến tự động chuyển sang kênh khẩn cấp ESP-NOW P2P trực tiếp. | UI Dashboard mất telemetry realtime, NHƯNG còi báo động khẩn cấp vẫn bật trong $<50\text{ ms}$ khi kích hoạt. |
| **Edge Gateway bị Treo/Sập** | Nút nhận lỗi ESP-NOW ACK failure | Nút cảm biến tự động điều hướng lại gói tin khẩn cấp tới địa chỉ MAC Nút Còi Local. | Lịch sử log tạm ngừng ghi; hệ thống báo động vật lý cục bộ vẫn hoạt động 100%. |
| **Lỗi Ổ đĩa CSDL Gateway** | Lỗi I/O Write SQLite trên Gateway | Pipeline Ingest chuyển các sự kiện chuẩn hóa vào Ring Buffer in-memory (sức chứa 10k). | Dashboard hiển thị cảnh báo; hệ thống tự thử nghiệm repair/vacuum CSDL SQLite. |
| **Dịch vụ AI Service Sập** | Subprocess health ping thất bại | Risk Engine hạ trọng số AI về 0.0 và tin tưởng 100% vào các quy tắc định tính cứng. | Tóm tắt VLM nâng cao bị tắt; các quy tắc an ninh cốt lõi vẫn hoạt động bình thường. |
| **Nút Cảm biến Mất Nguồn** | Mất 3 lần Heartbeat liên tiếp (90s) | Gateway cập nhật trạng thái thiết bị thành `OFFLINE` và tạo sự kiện `WARNING_DEVICE_DISCONNECTED`. | Dashboard báo người dùng kiểm tra nguồn/pin của nút cảm biến. |
| **Phá sóng RF Jamming ESP-NOW** | Tỷ lệ rớt gói tin cao ($>80\%$) | Gateway báo `RF_INTERFERENCE_HIGH` và đẩy tần suất lấy mẫu của nút lên mức tối đa. | Hệ thống ghi log nghi vấn có can thiệp phá sóng RF. |
| **Ngắt kết nối WebSocket** | Sự kiện `onclose` trên Browser WS | Giao diện UI dashboard chạy vòng lặp reconnect với exponential backoff ($1\text{s} \rightarrow 30\text{s}$). | Dashboard tạm hiển thị banner "Đang kết nối lại..."; tự đồng bộ khi nối lại thành công. |
| **Mất Điện Nguồn AC Đột ngột** | Ngắt ngắt nguồn trên Gateway UPS | Gateway thực thi script Clean Shutdown; các nút ESP32 chạy bằng bộ pin dự phòng local. | Hệ thống duy trì vận hành bằng pin dự phòng local tối đa 6 giờ. |
| **Lệch Xung nhịp Gateway Clock**| Sync NTP thất bại | Bộ xử lý sự kiện dùng timestamp tương đối Monotonic (`uptime_ms`) để xếp thứ tự. | Giờ log có thể lệch nhẹ, nhưng thứ tự sự kiện vẫn giữ tính định tính chính xác. |
