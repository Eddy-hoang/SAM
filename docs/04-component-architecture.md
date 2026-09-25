# 04 - Kiến trúc Thành phần (Component Architecture)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Thành phần Cơ sở  

---

## 1. Tiêu chuẩn Mô tả Thành phần (8-Point Template)

Mỗi thành phần trong hệ thống được định nghĩa nghiêm ngặt theo mẫu đặc tả 8 điểm:
1. **Mục đích (Purpose)**
2. **Nhiệm vụ chính (Responsibilities)**
3. **Đầu vào (Inputs)**
4. **Đầu ra (Outputs)**
5. **Thành phần Phụ thuộc (Dependencies)**
6. **Kịch bản Thất bại (Failure Modes)**
7. **Cân nhắc An ninh (Security Considerations)**
8. **Cân nhắc Mở rộng (Scaling Considerations)**

---

## 2. Các Thành phần Tầng Edge (Edge Layer Components)

### 2.1 Nút ESP32-S3 Edge Vision Node
* **Mục đích:** Thực hiện bắt khung hình camera cục bộ, suy luận Edge AI (phát hiện người/thảm họa), lọc nhiễu theo thời gian (temporal filtering) và phát tín hiệu khẩn cấp.
* **Nhiệm vụ chính:** Quản lý frame buffer, thực thi mô hình TFLite Micro, vận hành state machine debounce, đóng gói khung tin ESP-NOW mã hóa AES-128, gửi heartbeat.
* **Đầu vào:** Tín hiệu CMOS sensor thô (OV2640 qua giao tiếp DVP), nguồn điện, cấu hình ESP-NOW.
* **Đầu ra:** Gói tin khẩn cấp ESP-NOW mã hóa, gói tin telemetry MQTT qua Wi-Fi, log chẩn đoán serial.
* **Thành phần Phụ thuộc:** ESP-IDF v5.x, TensorFlow Lite Micro, driver ESP-NOW.
* **Kịch bản Thất bại:** Ống kính camera bị che khuất, phân mảnh bộ nhớ PSRAM, thảm nhiệt CPU khi bị nắng chiếu trực tiếp.
* **Cân nhắc An ninh:** Bảo vệ chống đọc ngược firmware (Flash Encryption), khóa mã hóa AES-128 lưu trong NVS, tạo sequence number chống phát lại.
* **Cân nhắc Mở rộng:** Hỗ trợ tối đa 8 nút camera cho mỗi khu vực Gateway mà không gây nghẽn kênh Wi-Fi.

### 2.2 Nút Cảm biến Môi trường & Vi phạm Ranh giới (Sensor Node)
* **Mục đích:** Thu thập dữ liệu an toàn môi trường dạng nhị phân (PIR, Reed switch) hoặc analog (khói/gas MQ-2).
* **Nhiệm vụ chính:** Đọc mẫu cảm biến, phát hiện vượt ngưỡng, phát cảnh báo ESP-NOW trực tiếp, quản lý năng lượng (tiết kiệm pin).
* **Đầu vào:** Thay đổi chân GPIO (ngắt công tắc cửa), giá trị analog ADC (nồng độ khói/gas).
* **Đầu ra:** Gói tin khẩn cấp ESP-NOW, trạng thái dung lượng pin định kỳ.
* **Thành phần Phụ thuộc:** Vi điều khiển ESP32-C3 / ESP8266, bảng hiệu chuẩn ADC.
* **Kịch bản Thất bại:** Trôi điểm chuẩn cảm biến, kiệt pin, dội tiếp điểm cơ khí (contact bounce).
* **Cân nhắc An ninh:** Chống phát lại bằng sequence counter, khóa mạng mã hóa pre-shared ESP-NOW.
* **Cân nhắc Mở rộng:** Hỗ trợ tối đa 32 nút cảm biến công suất thấp trên mỗi phân đoạn mạng ESP-NOW mesh.

---

## 3. Thành phần Truyền thông & Gateway (Communication & Gateway)

### 3.1 Bộ Xử lý Sự kiện Gateway (Event Processor)
* **Mục đích:** Ingest các tin nhắn thô từ Serial/MQTT/ESP-NOW, xác minh schema, khử trùng lặp (deduplication) và chuẩn hóa thành các `SystemEvent` thống nhất.
* **Nhiệm vụ chính:** Xác thực tin nhắn, kiểm tra thứ tự sequence number, quản lý cửa sổ khử trùng lặp LRU, tạo đối tượng JSON sự kiện chuẩn.
* **Đầu vào:** Gói tin ESP-NOW (qua Serial bridge), topic MQTT telemetry (`safehome/device/+/telemetry`).
* **Đầu ra:** Đối tượng `SystemEvent` đã chuẩn hóa chuyển tới Risk Engine và DB.
* **Thành phần Phụ thuộc:** Gateway runtime (Node.js/Go/Python), MQTT broker (Mosquitto).
* **Kịch bản Thất bại:** Tràn bộ đệm Serial, tiêm payload JSON không hợp lệ, lệch xung nhịp timestamping.
* **Cân nhắc An ninh:** Kiểm duyệt Schema ngăn chặn mã độc tiêm vào; xác minh HMAC loại bỏ tin nhắn giả mạo.
* **Cân nhắc Mở rộng:** Xử lý lên tới 500 events/giây trên 1 instance (vượt xa tải nhà ở thông thường).

### 3.2 Deterministic Risk Engine
* **Mục đích:** Đánh giá các sự kiện hệ thống đã chuẩn hóa theo ma trận quy tắc để tính toán điểm rủi ro và cấp độ đe dọa realtime.
* **Nhiệm vụ chính:** Gom nhóm nhiều sự kiện (PIR + Vision), tính toán điểm rủi ro (0–100), quản lý chuyển trạng thái đe dọa (NORMAL $\rightarrow$ CRITICAL).
* **Đầu vào:** Đối tượng `SystemEvent`, quy tắc an toàn đã lưu, bộ đếm thời gian hysteresis.
* **Đầu ra:** Cập nhật trạng thái Risk State, phát lệnh điều khiển tới Safety Policy.
* **Thành phần Phụ thuộc:** CSDL Local, Bộ nhớ In-Memory State (Redis / Map).
* **Kịch bản Thất bại:** File cấu hình quy tắc bị hỏng, bế tắc trạng thái (state deadlock).
* **Cân nhắc An ninh:** Quy tắc định tính được ghi cứng ngăn chặn việc đè quy tắc từ API bên ngoài nếu không có mật khẩu Admin.
* **Cân nhắc Mở rộng:** Thời gian thực thi bảng quy tắc in-memory $<1\text{ ms}$.

### 3.3 Safety Policy & Command Validator
* **Mục đích:** Đóng vai trò tường lửa cách ly tuyệt đối giữa trí tuệ cấp cao (AI/LLM/Risk Engine) và các thiết bị kích hoạt phần cứng (actuators).
* **Nhiệm vụ chính:** Xác minh xem lệnh kích hoạt (mở cửa, bật còi) có tuân thủ các bất biến an toàn (Safety Invariants) hay không.
* **Đầu vào:** Lệnh đề xuất từ Risk Engine hoặc AI Service.
* **Đầu ra:** Lệnh kích hoạt đã phê duyệt chuyển tới Gateway Hardware Bridge, hoặc Log từ chối lệnh.
* **Thành phần Phụ thuộc:** Policy Module mã nguồn đọc (read-only binary logic).
* **Kịch bản Thất bại:** Từ chối nhầm lệnh hợp lệ do cấu hình sai ngưỡng Policy.
* **Cân nhắc An ninh:** Các dịch vụ AI KHÔNG THỂ vượt qua tầng kiểm duyệt này trong bất kỳ chế độ vận hành nào.
* **Cân nhắc Mở rộng:** Không phụ thuộc bên ngoài; thời gian thực thi ở mức microsecond.

---

## 4. Thành phần Ứng dụng & Giao diện (Application & Interfaces)

### 4.1 Realtime WebSocket Broker & API Server
* **Mục đích:** Cung cấp các RESTful endpoint quản lý và push telemetry WebSocket ngay tức thì tới dashboard người vận hành.
* **Nhiệm vụ chính:** Quản lý kết nối client, xác thực JWT, lọc subscription WebSocket topic, điều hướng HTTP REST.
* **Đầu vào:** HTTP Request từ người dùng, broadcast chuyển trạng thái từ Risk Engine.
* **Đầu ra:** HTTP REST JSON Response, luồng frame WebSocket realtime.
* **Thành phần Phụ thuộc:** Node.js Express/Fastify hoặc Python FastAPI, thư viện WS.
* **Kịch bản Thất bại:** Mất kết nối WebSocket client khi chuyển mạng, cạn kiệt cổng API.
* **Cân nhắc An ninh:** Mã hóa TLS/WSS, xác thực JWT token, giới hạn tần suất IP Rate Limiting.
* **Cân nhắc Mở rộng:** Hỗ trợ đồng thời 50 phiên làm việc dashboard client mượt mà.
