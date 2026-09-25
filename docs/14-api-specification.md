# 14 - API Specification

> **Document Status:** `[DECISION]` RESTful Interface Specification  
> **Base URL:** `http://<gateway-ip>:8080/api/v1`  
> **Auth Header:** `Authorization: Bearer <JWT-TOKEN>`  

---

## 1. Endpoints Overview

| Method | Endpoint Path | Access Level | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/events` | Device / Node | Ingest raw event payload from network nodes |
| `GET` | `/api/events` | User / Operator | Query historical system event log with filters |
| `GET` | `/api/devices` | User / Operator | List registered mesh nodes and status |
| `GET` | `/api/devices/{id}` | User / Operator | Get detailed node health and metadata |
| `GET` | `/api/alerts` | User / Operator | List active security alerts & risk state |
| `GET` | `/api/system/status` | User / Operator | Get Gateway operational health and metrics |
| `POST` | `/api/system/actuate`| Admin User | Manually trigger/silence physical sirens |

---

## 2. Endpoint Specifications

### 2.1 Ingest System Event
* **Method & Path:** `POST /api/events`
* **Purpose:** Node API to submit a normalized event payload over HTTP/Wi-Fi.
* **Authentication:** Device Pre-Shared Key Header `X-Device-Token`.
* **Idempotency:** Enforced via `event_id` UUID in request body.
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
* **Success Response (201 Created):**
```json
{
  "status": "ACCEPTED",
  "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "processed_at": 1727280000015
}
```
* **Error Responses:** `400 Bad Request` (Invalid Schema), `409 Conflict` (Duplicate Event ID).

---

### 2.2 Get Active Alerts
* **Method & Path:** `GET /api/alerts`
* **Purpose:** Retrieves list of unresolved security alerts.
* **Request Query Params:** `status=ACTIVE&min_severity=HIGH`
* **Success Response (200 OK):**
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

### 2.3 Manual Safety Actuation Command
* **Method & Path:** `POST /api/system/actuate`
* **Purpose:** Admin command to manually trigger or silence sirens.
* **Authentication:** Admin JWT Token Required.
* **Request Body:**
```json
{
  "target_device_id": "ALARM-NODE-HALLWAY",
  "action": "SILENCE_SIREN",
  "override_reason": "False alarm confirmed by user",
  "user_pin": "1234"
}
```
* **Success Response (200 OK):**
```json
{
  "status": "EXECUTED",
  "command_id": "cmd_991204",
  "validated_by_safety_policy": true
}
```
* **Error Responses:** `403 Forbidden` (Safety Policy Rejected Command / Invalid PIN).
