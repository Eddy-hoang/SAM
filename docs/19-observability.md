# 19 - Khả năng Giám sát & Chẩn đoán (Observability)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Telemetry, Metrics & Logging  

---

## 1. Đặc tả Chỉ số Metrics & Telemetry Hệ thống

Hệ thống thu thập các chỉ số vận hành realtime được truy vấn qua endpoint Gateway `GET /api/system/status`:

```json
{
  "system_uptime_seconds": 86400,
  "gateway_cpu_usage_percent": 14.2,
  "gateway_memory_free_bytes": 2840192000,
  "metrics": {
    "esp_now_frames_received_total": 1420,
    "events_processed_total": 312,
    "active_alerts_count": 0,
    "average_inference_latency_ms": 138.5,
    "average_network_latency_ms": 12.2,
    "event_ingest_error_rate": 0.001,
    "dropped_duplicate_events_total": 45
  }
}
```

---

## 2. Định dạng Log Cấu trúc (JSON Lines)

Tất cả các vi dịch vụ Gateway phát log cấu trúc JSON ra tiêu chuẩn stdout để phục vụ thu thập:

```json
{
  "timestamp": "2026-09-25T15:45:00.123Z",
  "level": "INFO",
  "component": "RISK_ENGINE",
  "trace_id": "tr_8812a0f49",
  "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "message": "Điểm Rủi ro được đánh giá: ELEVATED -> HIGH",
  "context": {
    "previous_score": 35,
    "new_score": 65,
    "trigger_rules": ["RULE_PIR_MOTION", "RULE_VISION_PERSON_DETECTED"]
  }
}
```

---

## 3. Panel Giám sát Chỉ số cho Bài Trình diễn Demo

Phục vụ bài trình diễn cuộc thi Danang AI4Life, giao diện UI trang bị panel **System Health Gauge** sống:
1. **Đồng hồ Độ trễ ESP-NOW:** Đồ thị độ trễ truyền phát hai chiều khứ hồi realtime ($\text{Mục tiêu } <20\text{ ms}$).
2. **Độ trễ Suy luận Edge AI:** Tốc độ thực thi vòng lặp suy luận ESP32-S3 liên tục ($\text{Mục tiêu } 130-150\text{ ms}$).
3. **Bản đồ Topology Nút Mesh:** Hiển thị trạng thái online/offline, dung lượng pin và cường độ tín hiệu RF RSSI từng nút.
