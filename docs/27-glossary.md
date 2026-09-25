# 27 - Thuật ngữ Kỹ thuật (Glossary)

> **Trạng thái Tài liệu:** `[DECISION]` Từ điển Thuật ngữ Kỹ thuật & Khái niệm  

---

## 1. Danh mục Thuật ngữ

### ESP-NOW
Giao thức truyền thông không dây không kết nối (connectionless), công suất thấp do Espressif phát triển, sử dụng các khung tin IEEE 802.11 action frames tùy biến. Trong SafeHome, ESP-NOW được dùng cho kênh cảnh báo khẩn cấp sub-50ms không phụ thuộc router Wi-Fi.

### Edge AI
Phương thức thực thi các mô hình trí tuệ nhân tạo machine learning trực tiếp trên vi điều khiển cục bộ hoặc thiết bị edge gateway mà không gửi dữ liệu thô về máy chủ cloud. Được dùng trong SafeHome để nhận diện hình ảnh trực tiếp trên chip ESP32-S3.

### PSRAM (Pseudo-Static RAM)
Bộ nhớ RAM mở rộng bên ngoài kết nối với vi điều khiển ESP32 qua giao tiếp SPI/QSPI. SafeHome sử dụng 8MB Octal PSRAM trên ESP32-S3 để chứa các bộ đệm khung hình camera và trọng số mô hình TensorFlow Lite.

### Temporal Engine & Debounce
Bộ máy trạng thái phần mềm lọc bỏ nhiễu và tín hiệu thoáng qua bằng cách yêu cầu nhiều khung hình nhận diện khớp liên tiếp theo thời gian trước khi chuyển trạng thái. Giúp triệt tiêu báo động giả.

### Hysteresis
Đặc tính của một hệ thống khi trạng thái phụ thuộc vào lịch sử vận hành trước đó. Được sử dụng trong Risk Engine của SafeHome để ngăn hiện tượng bật/tắt liên tục giữa các cấp độ cảnh báo `NORMAL` và `CRITICAL`.

### Idempotency
Tính chất của một API khi thực thi một thao tác nhiều lần vẫn trả về kết quả giống hệt như thực thi một lần. Được đảm bảo trong SafeHome bằng cơ chế khử trùng lặp dựa trên V4 UUID.

### Risk Engine
Bộ đánh giá định tính trên Gateway có nhiệm vụ tổng hợp các sự kiện từ nhiều cảm biến/camera, tính toán điểm rủi ro hệ thống (0-100) và quản lý cấp độ đe dọa.

### Safety Policy Firewall
Mô-đun quyết định đọc mã nguồn độc lập, có nhiệm vụ kiểm duyệt các lệnh phần mềm cấp cao dựa trên các bất biến an toàn (Safety Invariants) trước khi cho phép kích hoạt phần cứng vật lý.
