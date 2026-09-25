# 16 - Kiến trúc An ninh & Threat Model (Security Architecture)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả An ninh & Mật mã học  
> **Phương pháp Mô hình hóa Mô đe dọa:** Mô hình STRIDE Framework  

---

## 1. Ma trận Mô hình Mối đe dọa STRIDE & Biện pháp Giảm thiểu

| Phân loại STRIDE | Kịch bản Tấn công Cụ thể | Mức độ Tác hại | Biện pháp Giảm thiểu / Countermeasure | Trạng thái |
| :--- | :--- | :--- | :--- | :--- |
| **Giả mạo (Spoofing)** | Kẻ tấn công giả mạo MAC cảm biến để gửi tin nhắn khói giả. | High (Hoảng loạn giả) | Mã hóa ESP-NOW AES-128 LMK + Pre-shared key xác thực nút. | `[DECISION]` |
| **Can thiệp (Tampering)** | Trộm chặn và sửa đổi metadata sự kiện truyền qua Wi-Fi. | High (Hỏng dữ liệu) | TLS 1.3 / HTTPS cho API; xác minh chữ ký HMAC-SHA256 trên MQTT. | `[DECISION]` |
| **Chối bỏ (Repudiation)**| Người dùng cố tình tắt còi rồi chối bỏ hành vi. | Medium (Lỗi Audit) | Bảng `audit_logs` bất biến lưu User ID, IP và timestamp cho mọi thao tác. | `[DECISION]` |
| **Rò rỉ (Info Leak)** | Nghe lén không dây bắt luồng video thô trên không gian mạng. | High (Vi phạm Riêng tư)| Không stream video thô; AI trên thiết bị chỉ tạo ra metadata JSON. | `[DECISION]` |
| **Từ chối (Denial of Svc)**| Tấn công flood request API hoặc dùng jammer phá sóng 2.4GHz. | Critical (Mù hệ thống) | IP Rate Limiting tại Gateway; Còi khẩn cấp ESP-NOW offline độc lập. | `[DECISION]` |
| **Nâng quyền (Elevation)**| Kẻ tấn công bỏ qua UI để phát lệnh điều khiển phần cứng trực tiếp.| Critical (Ghi đè Phần cứng)| Tường lửa Safety Policy cứng kiểm duyệt mọi yêu cầu kích hoạt API. | `[DECISION]` |

---

## 2. Vòng đời Quản lý Khóa Mật mã (Key Management Lifecycle)

```text
                  ┌─────────────────────────────────────────┐
                  │       Khởi tạo Khóa tại Nhà máy         │
                  │ Hardware RNG tạo PMK/LMK 128-bit        │
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │       Lưu trữ trong ESP32 eFuse NVS     │
                  │ Được bảo vệ bởi Flash Encryption (AES)  │
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │     Xoay Khóa Định kỳ (Mỗi 30 Ngày)     │
                  │ Gateway đàm phán lại LMK qua ESP-NOW    │
                  └─────────────────────────────────────────┘
```

---

## 3. Cân nhắc Firmware Bảo mật (Secure Firmware)

1. **ESP32-S3 Secure Boot v2:** Bật tính năng xác minh chữ ký RSA-3072 khi bootloader khởi động để ngăn chặn nạp các file binary firmware độc hại không có chữ ký.
2. **Mã hóa Flash (Flash Encryption):** Bật mã hóa phần cứng AES-128-XTS cho SPI Flash internal. Firmware binary và khóa mã hóa NVS không thể bị dump ra ngoài qua các chân nạp vật lý JTAG/UART.
