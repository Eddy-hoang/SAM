# 19 - Observability & Diagnostics

> **Document Status:** `[DECISION]` Telemetry, Metrics & Logging Specification  

---

## 1. System Metrics & Telemetry Specification

The system collects real-time operational metrics exposed via Gateway endpoint `GET /api/system/status`:

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

## 2. Structural Structured Log Format (JSON Lines)

All Gateway micro-services emit JSON logs to standard output for processing:

```json
{
  "timestamp": "2026-09-25T15:45:00.123Z",
  "level": "INFO",
  "component": "RISK_ENGINE",
  "trace_id": "tr_8812a0f49",
  "event_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "message": "Risk Score evaluated: ELEVATED -> HIGH",
  "context": {
    "previous_score": 35,
    "new_score": 65,
    "trigger_rules": ["RULE_PIR_MOTION", "RULE_VISION_PERSON_DETECTED"]
  }
}
```

---

## 3. Demo Monitoring Metrics Panel

For the Danang AI4Life competition demo, the UI dashboard features a live **System Health Gauge**:
1. **ESP-NOW Latency Gauge:** Real-time round-trip latency graph ($\text{Target } <20\text{ ms}$).
2. **Edge AI Inference Latency:** Continuous ESP32-S3 inference loop execution speed ($\text{Target } 130-150\text{ ms}$).
3. **Mesh Node Topology Map:** Live node online/offline status with battery levels and RF RSSI signal strength indicators.
