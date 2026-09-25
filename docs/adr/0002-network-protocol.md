# ADR-0002: Chiến lược Mạng Hybrid Phân luồng Kép (ESP-NOW + Wi-Fi)

## Bối cảnh (Context)
Các cảnh báo khẩn cấp yêu cầu độ trễ ở mức microsecond và không phụ thuộc vào router, trong khi các dashboard quản lý lại cần luồng telemetry dữ liệu trạng thái dồi dào.

## Vấn đề (Problem)
Nên chọn các giao thức mạng nào cho việc truyền phát cảnh báo khẩn cấp giữa các thiết bị so với báo cáo telemetry về gateway?

## Các Phương án Đánh giá (Considered Options)
1. **Chỉ dùng Wi-Fi (MQTT/HTTP):** Tất cả các nút giao tiếp duy nhất qua mạng Wi-Fi 802.11 tiêu chuẩn.
2. **Chỉ dùng ESP-NOW:** Tất cả truyền thông (bao gồm telemetry và hình ảnh) dùng ESP-NOW.
3. **Kiến trúc Hybrid Phân luồng Kép:** ESP-NOW cho kênh còi báo động khẩn cấp; Wi-Fi (MQTT + WebSockets) cho telemetry và streaming giao diện UI.

## Quyết định (Decision)
Lựa chọn **Phương án 3: Kiến trúc Hybrid Phân luồng Kép**.

## Lý do (Why)
* ESP-NOW bỏ qua các thủ tục bắt tay Wi-Fi AP và cấp phát DHCP, truyền frame khẩn cấp dưới 15ms.
* MQTT qua Wi-Fi local xử lý các đối tượng telemetry JSON cấu trúc hiệu quả.
* Nếu router Wi-Fi sập, kênh còi khẩn cấp ESP-NOW vẫn hoạt động 100%.

## Đánh đổi (Trade-offs)
* Tất cả chip radio Wi-Fi của ESP32 phải khóa cố định ở một kênh Wi-Fi 2.4GHz (Channel 6).

## Trạng thái (Status)
`[DECISION]` Đã phê duyệt & Thông qua.
