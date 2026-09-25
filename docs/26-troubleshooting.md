# 26 - Troubleshooting & Field Diagnostics Guide

> **Document Status:** `[DECISION]` Field Diagnostic Manual  

---

## 1. Quick Diagnostic Decision Tree

```text
Problem Encountered?
 ├── ESP32-S3 Camera fails to boot? ──────> [Check 5V PSU decoupling capacitor & PSRAM config]
 ├── ESP-NOW messages dropped? ─────────> [Verify Wi-Fi channel is LOCKED to Channel 6 on all nodes]
 ├── High false alerts on camera? ────────> [Increase temporal consecutive frame parameter N from 3 to 5]
 └── Dashboard displays disconnected? ───> [Inspect Gateway WebSocket port 8080 & Mosquitto service]
```

---

## 2. Issue Resolution Matrix

| Symptom / Error | Root Cause | Resolution Step |
| :--- | :--- | :--- |
| `Brownout detector was triggered` | ESP32-S3 power rail dips during Wi-Fi transmission spikes. | Connect dedicated 5V/2A power supply; solder 1000uF capacitor across 5V/GND pins. |
| `ESP_ERR_NO_MEM` during inference | TFLite Tensor Arena size exceeds internal SRAM heap. | Allocate Tensor Arena buffer in PSRAM using `heap_caps_malloc(..., MALLOC_CAP_SPIRAM)`. |
| `ESP-NOW peer not found` | Transmitter and Receiver operate on different Wi-Fi channels. | Call `esp_wifi_set_channel(6, WIFI_SECOND_CHAN_NONE)` before initializing ESP-NOW. |
| Gateway `409 Conflict` Error | Node re-sent duplicate event with identical `event_id`. | Normal idempotency defense working correctly; check if node retry interval is too aggressive. |
