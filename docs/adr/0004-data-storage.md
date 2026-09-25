# ADR-0004: Lựa chọn Cơ sở Dữ liệu Gateway Nhúng (SQLite WAL Mode)

## Bối cảnh (Context)
Edge Gateway cần một hệ lưu trữ local persistent, có cấu trúc, độ trễ thấp cho nhật ký sự kiện, chỉ số sức khỏe thiết bị và các cảnh báo active trên phần cứng Raspberry Pi.

## Vấn đề (Problem)
Hệ quản trị cơ sở dữ liệu nào nên được chọn cho lưu trữ cục bộ tại edge?

## Các Phương án Đánh giá (Considered Options)
1. **RDBMS Đầy đủ (PostgreSQL / MySQL):** Hệ CSDL server nặng.
2. **Time-Series Nhúng (SQLite ở chế độ WAL Mode):** CSDL quan hệ nhúng lưu file duy nhất, cấu hình bằng 0 với tính năng Write-Ahead Logging.

## Quyết định (Decision)
Lựa chọn **Phương án 2: SQLite ở chế độ WAL Mode**.

## Lý do (Why)
* **Dung lượng Bộ nhớ:** Chiếm $<5\text{MB}$ RAM so với $>200\text{MB}$ của PostgreSQL.
* **Bảo trì:** Không cần quản trị CSDL hay dịch vụ server phức tạp; giao dịch ACID tin cậy trên bộ nhớ flash.

## Trạng thái (Status)
`[DECISION]` Đã phê duyệt & Thông qua.
