# 07 - Event Generation & Temporal State Engine

> **Document Status:** `[DECISION]` Event Pipeline & Data Serialization Specification  

---

## 1. Formal Event Generation Pipeline

To eliminate false-alarm spam, raw frame inferences are processed through a deterministic temporal state machine before creating a network payload:

```mermaid
stateDiagram-v2
    [*] --> CLEAR: System Init / Idle

    CLEAR --> OBSERVING: Detection Score >= 0.75 (Frame 1)
    OBSERVING --> CLEAR: Detection Score < 0.75 (Frame Drop)
    
    OBSERVING --> CONFIRMED: Consecutive Frame Count = 3
    CONFIRMED --> EVENT_TRIGGERED: Emit SYSTEM_EVENT JSON

    EVENT_TRIGGERED --> ACTIVE_HOLD: Cooldown Hysteresis Timer Active (5s)
    
    ACTIVE_HOLD --> ACTIVE_HOLD: Continued Detection
    ACTIVE_HOLD --> CLEAR: Cooldown Expired & No Detection for 5s
```

---

## 2. Temporal Engine Parameters

| Parameter Name | Value | Purpose | Architectural Status |
| :--- | :--- | :--- | :--- |
| `CONFIDENCE_THRESHOLD` | `0.75` | Minimum Softmax score for single-frame detection | `[DECISION]` |
| `CONSECUTIVE_FRAMES` | `3` | Required consecutive positive matches ($N=3$) | `[DECISION]` |
| `COOLDOWN_WINDOW_MS` | `5000` | Delay before allowing state drop or duplicate re-emission | `[DECISION]` |
| `HYSTERESIS_MARGIN` | `0.15` | Drop threshold is $0.75 - 0.15 = 0.60$ during active hold | `[DECISION]` |
| `HEARTBEAT_INTERVAL_MS`| `30000` | Periodic keep-alive ping when state remains unchanged | `[DECISION]` |

---

## 3. Formal System Event JSON Schema

When state transitions to `EVENT_TRIGGERED`, the node generates a strictly typed JSON payload:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "SafeHomeSystemEvent",
  "type": "object",
  "properties": {
    "event_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique V4 UUID generated at event origin for deduplication and idempotency"
    },
    "schema_version": {
      "type": "string",
      "enum": ["1.0.0"],
      "description": "Semantic versioning tag for backwards compatibility"
    },
    "device_id": {
      "type": "string",
      "example": "ESP32CAM-ZONE1-FRONTDOOR",
      "description": "Unique hardware MAC-derived identifier"
    },
    "timestamp_ms": {
      "type": "integer",
      "description": "Epoch timestamp in milliseconds at temporal trigger moment"
    },
    "sequence_number": {
      "type": "integer",
      "minimum": 1,
      "description": "Monotonically increasing counter per device for anti-replay & ordering"
    },
    "event_type": {
      "type": "string",
      "enum": [
        "HUMAN_DETECTION",
        "SMOKE_DETECTION",
        "GAS_LEAK_DETECTION",
        "PERIMETER_BREACH",
        "TAMPER_ALERT",
        "DEVICE_HEARTBEAT"
      ]
    },
    "severity": {
      "type": "string",
      "enum": ["INFO", "WARNING", "CRITICAL", "EMERGENCY"]
    },
    "confidence": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Filtered temporal confidence score"
    },
    "location_zone": {
      "type": "string",
      "example": "PERIMETER_NORTH"
    },
    "metadata": {
      "type": "object",
      "properties": {
        "consecutive_frames": { "type": "integer", "example": 3 },
        "inference_latency_ms": { "type": "integer", "example": 135 },
        "battery_voltage": { "type": "number", "example": 4.12 }
      },
      "additionalProperties": true
    }
  },
  "required": [
    "event_id",
    "schema_version",
    "device_id",
    "timestamp_ms",
    "sequence_number",
    "event_type",
    "severity",
    "confidence",
    "location_zone"
  ]
}
```

---

## 4. Idempotency & Deduplication Rules

1. **Gateway Ingest Deduplication:** The Edge Gateway maintains an in-memory Sliding Window LRU Cache of `event_id` strings (TTL = 60 seconds). Any duplicate `event_id` received within the window is silently logged and dropped.
2. **Sequence Number Verification:** Out-of-order sequence numbers per `device_id` are logged to detect dropped packets or network relay anomalies.
