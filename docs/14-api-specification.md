# 14 - Đặc tả Giao diện API (API Specification)

> **Trạng thái Tài liệu:** `[DECISION]` Đặc tả Giao diện RESTful  
> **Base URL:** `http://<gateway-ip>:8080/api/v1`  
> **Header Xác thực:** `Authorization: Bearer <JWT-TOKEN>`  

---

## 1. Tổng quan Các Endpoints

| Phương thức | Đường dẫn Endpoint | Cấp độ Truy cập | Mô tả Chức năng |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/events` | Device / Node | Ingest payload sự kiện thô từ các nút mạng |
| `GET` | `/api/events` | User / Operator | Truy vấn nhật ký sự kiện hệ thống kèm bộ lọc |
| `GET` | `/api/devices` | User / Operator | Danh sách các nút mesh đã đăng ký và trạng thái |
| `GET` | `/api/devices/{id}` | User / Operator | Xem thông số sức khỏe và metadata chi tiết nút |
| `GET` | `/api/alerts` | User / Operator | Danh sách cảnh báo an ninh active & cấp độ rủi ro |
| `GET` | `/api/system/status` | User / Operator | Xem chỉ số sức khỏe vận hành và metrics Gateway |
| `POST` | `/api/system/actuate`| Admin User | Lệnh bật/tắt còi báo động vật lý thủ công |

---

## 2. Chi tiết Đặc tả Endpoint

### 2.1 Ingest Sự kiện Hệ thống (Ingest Event)
* **Phương thức & Đường dẫn:** `POST /api/events`
* **Mục đích:** API cho nút gửi payload sự kiện chuẩn hóa qua HTTP/Wi-Fi.
* **Xác thực:** Header Token của Thiết bị `X-Device-Token`.
* **Tính Idempotency:** Bắt buộc áp dụng qua mã `event_id` UUID trong body.
* **Request Body:**
```json
{
  "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "schema_version": "1.0.0",
  "device_id": "ESP32CAM-ZONE1-FRONTDOOR",
  "timestamp_ms": 1727280000000,
  "sequence_number": 1042,
  "event_type": "HUMAN_DETECTION",
  "severity": "CRITICAL",
  "confidence": 0.92,
  "location_zone": "FRONT_PORCH",
  "metadata": { "consecutive_frames": 3 }
}
```
* **Phản hồi Thành công (201 Created):**
```json
{
  "status": "ACCEPTED",
  "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "processed_at": 1727280000015
}
```
* **Phản hồi Lỗi:** `400 Bad Request` (Invalid Schema), `409 Conflict` (Duplicate Event ID).

---

### 2.2 Lấy Danh sách Alert Đang Active
* **Phương thức & Đường dẫn:** `GET /api/alerts`
* **Mục đích:** Lấy danh sách các cảnh báo an ninh chưa được xử lý.
* **Query Params:** `status=ACTIVE&min_severity=HIGH`
* **Phản hồi Thành công (200 OK):**
```json
{
  "system_risk_score": 75,
  "system_risk_level": "CRITICAL",
  "active_alerts": [
    {
      "alert_id": "alt_8823f01a",
      "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "device_id": "ESP32CAM-ZONE1-FRONTDOOR",
      "risk_level": "CRITICAL",
      "risk_score": 75,
      "status": "ACTIVE",
      "created_at": "2026-09-25T15:30:00Z"
    }
  ]
}
```

---

### 2.3 Lệnh Kích hoạt An toàn Thủ công (Manual Safety Actuation)
* **Phương thức & Đường dẫn:** `POST /api/system/actuate`
* **Mục đích:** Lệnh Admin để bật hoặc tắt còi báo động thủ công.
* **Xác thực:** Yêu cầu Admin JWT Token.
* **Request Body:**
```json
{
  "target_device_id": "ALARM-NODE-HALLWAY",
  "action": "SILENCE_SIREN",
  "override_reason": "Xác nhận báo động giả bởi người dùng",
  "user_pin": "1234"
}
```
* **Phản hồi Thành công (200 OK):**
```json
{
  "status": "EXECUTED",
  "command_id": "cmd_991204",
  "validated_by_safety_policy": true
}
```
* **Phản hồi Lỗi:** `403 Forbidden` (Safety Policy từ chối lệnh / Mã PIN sai).
