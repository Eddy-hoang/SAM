# 11 - Pipeline Xử lý Sự kiện (Event Processing Pipeline)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Hàng chờ Sự kiện & Chuẩn hóa  

---

## 1. Kiến trúc Hàng chờ Ingest Queue

Để tránh mất sự kiện khi có bùng nổ dữ liệu (ví dụ: nhiều cảm biến bị kích hoạt cùng lúc), các tin nhắn gửi đến được đưa qua một Hàng chờ Bất đồng bộ (Asynchronous Queue) lưu trong bộ nhớ:

```text
Payload Gửi đến ──> Schema Validator ──> LRU Deduplicator ──> In-Memory Queue (FIFO) ──> Event Processor
```

---

## 2. Tham số Hàng chờ Ingest Queue

| Khóa Cấu hình | Giá trị Tham số | Lý do Kiến trúc |
| :--- | :--- | :--- |
| `MAX_QUEUE_DEPTH` | `1000 Events` | Tránh tràn RAM khi xử lý bị trễ |
| `DEDUP_WINDOW_TTL_MS`| `60000 ms` | Thời gian sống của cửa sổ khử trùng lặp cho V4 UUID |
| `MAX_CLOCK_DRIFT_MS`| `30000 ms` | Độ lệch thời gian tối đa cho phép so với giờ Gateway |
| `CONCURRENCY_WORKERS`| `4 Threads` | Pool các luồng công việc xử lý sự kiện đồng thời |

---

## 3. Quy tắc Idempotency & Kiểm tra Thứ tự Sequence Number

1. **Tính Duy nhất của UUID:** Mỗi sự kiện chứa một `event_id` (V4 UUID). Gateway sẽ hủy bỏ bất kỳ sự kiện nào gửi tới trùng với `event_id` đang active trong cache khử trùng lặp.
2. **Phát hiện Lệch Sequence Number:**
   * Gateway theo dõi `last_seen_seq` theo từng `device_id`.
   * Nếu `incoming_seq == last_seen_seq + 1`: Xử lý bình thường.
   * Nếu `incoming_seq > last_seen_seq + 1`: Ghi log `WARNING_PACKET_LOSS` (phát hiện mất gói tin trên không gian mạng).
   * Nếu `incoming_seq <= last_seen_seq`: Ghi log `SECURITY_ALERT_REPLAY_ATTEMPT` và hủy bỏ gói tin ngay lập tức.
