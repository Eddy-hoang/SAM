# 26 - Cẩm nang Chẩn đoán Sự cố Thực địa (Troubleshooting)

> **Trạng thái Tài liệu:** `[DECISION]` Sổ tay Chẩn đoán Thực địa  

---

## 1. Cây Quyết định Chẩn đoán Nhanh (Diagnostic Decision Tree)

```text
Gặp Sự cố Kỹ thuật?
 ├── ESP32-S3 Camera không khởi động được? ─> [Kiểm tra tụ lọc nguồn 5V & cấu hình nhận diện PSRAM]
 ├── Tin nhắn ESP-NOW bị rơi gói tin? ──────> [Xác nhận kênh Wi-Fi đã KHÓA cố định Channel 6 trên tất cả nút]
 ├── Camera bị báo động giả nhiều? ──────────> [Tăng tham số frame liên tiếp N từ 3 lên 5 trong Temporal Filter]
 └── Dashboard hiển thị mất kết nối? ────────> [Kiểm tra port 8080 WebSocket Gateway & dịch vụ Mosquitto]
```

---

## 2. Ma trận Xử lý Sự cố (Issue Resolution Matrix)

| Hiện tượng / Mã Lỗi | Nguyên nhân Gốc rễ | Bước Xử lý Phục hồi |
| :--- | :--- | :--- |
| `Brownout detector was triggered` | Điện áp đường 5V ESP32-S3 bị sụt khi Wi-Fi phát sóng RF đỉnh. | Nối củ nguồn 5V/2A riêng; hàn tụ hóa 1000uF song song với chân 5V/GND. |
| Lỗi `ESP_ERR_NO_MEM` khi suy luận | Kích thước TFLite Tensor Arena vượt quá bộ nhớ internal SRAM. | Cấp phát bộ đệm Tensor Arena trong PSRAM dùng `heap_caps_malloc(..., MALLOC_CAP_SPIRAM)`. |
| `ESP-NOW peer not found` | Thiết bị phát và Thiết bị nhận chạy trên 2 kênh Wi-Fi khác nhau. | Gọi lệnh `esp_wifi_set_channel(6, WIFI_SECOND_CHAN_NONE)` trước khi khởi tạo ESP-NOW. |
| Lỗi `409 Conflict` tại Gateway | Nút gửi lại sự kiện trùng lặp chứa mã `event_id` giống hệt. | Cơ chế chống trùng Idempotency hoạt động bình thường; kiểm tra lại khoảng thời gian retry nút. |
