# ADR-0005: Tường lửa Tách biệt cho Safety Policy (Safety Policy Firewall)

## Bối cảnh (Context)
Generative AI và các dịch vụ phần mềm phức tạp mang đặc tính đầu ra bất định (non-deterministic), có thể đưa ra các lệnh kích hoạt phần cứng nhầm lẫn nguy hiểm (như vô tình mở khóa cửa hoặc tắt còi báo động khẩn cấp khi đang có hỏa hoạn).

## Vấn đề (Problem)
Làm thế nào kiến trúc có thể đảm bảo chắc chắn rằng các lỗi phần mềm hoặc ảo giác AI không bao giờ thực thi lệnh kích hoạt phần cứng trái phép?

## Các Phương án Đánh giá (Considered Options)
1. **AI Điều khiển Kích hoạt Trực tiếp:** Cho phép dịch vụ AI hoặc mô-đun rủi ro cấp cao trực tiếp phát lệnh GPIO actuator.
2. **Tường lửa Safety Policy Tách biệt:** Đón chặn mọi yêu cầu kích hoạt bằng một bộ kiểm duyệt Safety Policy Validator định tính, mã nguồn đọc, bắt buộc tuân thủ các bất biến an toàn (Safety Invariants).

## Quyết định (Decision)
Lựa chọn **Phương án 2: Tường lửa Safety Policy Tách biệt**.

## Lý do (Why)
* **An toàn Tính mạng Định tính:** Đảm bảo các bất biến an toàn (ví dụ: "Không được tắt còi báo động khẩn cấp khi cảm biến khói đang active") không thể bị vượt qua bởi bất kỳ lỗi phần mềm hay ảo giác LLM nào.

## Trạng thái (Status)
`[DECISION]` Đã phê duyệt & Thông qua.
