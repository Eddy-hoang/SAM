# ADR-0001: Kiến trúc Mạng Mesh Ưu tiên Edge Độc lập (Decoupled Edge-First Mesh)

## Bối cảnh (Context)
Các hệ thống an ninh nhà ở truyền thống liên tục stream dữ liệu cảm biến lên máy chủ cloud để xử lý. Điều này tạo ra rủi ro nghiêm trọng khi mất mạng Internet, gây độ trễ từ 2–5 giây và tiềm ẩn rò rỉ quyền riêng tư.

## Vấn đề (Problem)
SafeHome AI Mesh nên tổ chức phân tầng xử lý như thế nào để đảm bảo phản ứng khẩn cấp dưới 1 giây và khả năng hoạt động offline độc lập 100%?

## Các Phương án Đánh giá (Considered Options)
1. **Kiến trúc Tập trung Cloud (Cloud-Centric):** ESP32 stream video lên cloud backend (AWS/GCP) để phân tích AI và đánh giá rủi ro.
2. **Kiến trúc Tập trung Gateway (Gateway-Centric):** Stream video thô qua mạng Wi-Fi local tới một PC Gateway local để xử lý.
3. **Kiến trúc Mesh Ưu tiên Edge (Edge-First Mesh):** ESP32-S3 thực hiện vision AI local, phát ra metadata sự kiện đã lọc nhiễu, dùng ESP-NOW cho còi báo động khẩn cấp và gửi telemetry về Gateway.

## Quyết định (Decision)
Lựa chọn **Phương án 3: Kiến trúc Mesh Ưu tiên Edge (Edge-First Mesh)**. Suy luận vision AI diễn ra trực tiếp trên ESP32-S3. Tín hiệu cảnh báo khẩn cấp dùng truyền phát Peer-to-Peer qua ESP-NOW, hoàn toàn độc lập với router Wi-Fi và kết nối Internet.

## Lý do (Why)
* **Độ trễ:** ESP-NOW đạt độ trễ truyền phát trực tiếp tới còi local $<15\text{ ms}$.
* **Khả năng Chịu lỗi:** Hệ thống tiếp tục vận hành mượt mà ngay cả khi ngắt kết nối mạng WAN/Internet.
* **Quyền riêng tư:** Zero-video-streaming trên không gian mạng local hay máy chủ bên ngoài.

## Đánh đổi (Trade-offs)
* Phức tạp lập trình nhúng cao hơn trên ESP32-S3 (TFLite Micro, quản lý bộ nhớ PSRAM).
* Giới hạn vi điều khiển làm giảm độ phức tạp của mô hình vision AI xuống dạng mô hình nhẹ (MobileNet-V2).

## Trạng thái (Status)
`[DECISION]` Đã phê duyệt & Thông qua.
