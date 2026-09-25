# 17 - Kiến trúc Quyền riêng tư & Quản trị Dữ liệu (Privacy Architecture)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Quyền riêng tư từ Thiết kế (Privacy-by-Design)  

---

## 1. Kiến trúc Metadata trên Thiết bị (On-Device Metadata)

SafeHome AI Mesh thực thi nghiêm ngặt **Kiến trúc Quyền riêng tư Zero-Video-Streaming**. Các cảm biến camera hoạt động hoàn toàn như các bộ phát hiện sự kiện quang học thông minh:

```text
Khung cảnh Vật lý ──> Frame RAM (Bay hơi) ──> AI Inference ──> Metadata JSON ──> RAM bị xóa sạch
                             │
                             └─── Frame thô KHÔNG BAO GIỜ rời khỏi RAM trong Trạng thái Bình thường
```

---

## 2. Quy tắc Lưu trữ Ảnh Bằng chứng khi có Alert (Evidentiary Snapshot)

1. **Trạng thái Vận hành Bình thường:** Các bộ đệm frame camera thô chỉ tồn tại trong bộ nhớ tạm PSRAM ngắn hơn $< 200\text{ ms}$ trong thời gian xử lý suy luận và lập tức bị ghi đè bởi frame tiếp theo. Không lưu vào đĩa đĩa.
2. **Snapshot Bằng chứng khi có Alert:**  
   * *Điều kiện Kích hoạt:* CHỈ KHI có cảnh báo rủi ro mức `CRITICAL` hoặc `HIGH` đã được kiểm duyệt bởi Risk Engine.
   * *Hành động:* Nút camera chụp đúng 1 tấm ảnh JPEG ($640 \times 480$), mã hóa bằng AES-128 và gửi về CSDL local trên Gateway.
   * *Thời gian Lưu trữ (Retention Lifetime):* Lưu cục bộ trên Gateway trong **7 ngày**, sau đó một cron job tự động thực thi xóa file bảo mật (`shred / wipe`).

---

## 3. Ma trận Tối thiểu hóa Dữ liệu & Cách ly Cloud

| Loại Dữ liệu | Lưu trên Nút ESP32? | Truyền về Gateway? | Truyền lên Cloud? | Cấp độ Truy cập Người dùng |
| :--- | :--- | :--- | :--- | :--- |
| **Luồng Video Thô** | KHÔNG | KHÔNG | **KHÔNG BAO GIỜ** | Không có |
| **Ảnh JPEG Bằng chứng**| KHÔNG (Chỉ RAM) | CÓ (CSDL Local Mã hóa) | TÙY CHỌN (Chỉ khi Opt-in) | Chỉ Admin Owner |
| **Detection Metadata** | KHÔNG | CÓ (CSDL Local) | KHÔNG | Tất cả Dashboard Users |
| **Telemetry Cảm biến** | KHÔNG | CÓ (CSDL Local) | KHÔNG | Tất cả Dashboard Users |
