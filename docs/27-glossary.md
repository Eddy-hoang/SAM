# 27 - Technical Glossary

> **Document Status:** `[DECISION]` Technical Terms & Domain References  

---

## 1. Glossary Terms

### ESP-NOW
A low-power, peer-to-peer connectionless wireless communication protocol developed by Espressif that uses vendor-specific IEEE 802.11 action frames. Used in SafeHome for sub-50ms emergency alert transmission bypassing Wi-Fi routers.

### Edge AI
The execution of artificial intelligence machine learning models directly on localized microcontroller units or edge gateway hardware without streaming raw data to cloud servers. Used in SafeHome for on-device vision detection on ESP32-S3.

### PSRAM (Pseudo-Static RAM)
External RAM connected to ESP32 microcontrollers via SPI/QSPI interfaces. SafeHome uses 8MB Octal PSRAM on ESP32-S3 to hold camera frame buffers and TensorFlow Lite model weights.

### Temporal Engine & Debounce
A software state machine that filters transient noise and glitches by requiring multiple consecutive frame matches over time before transitioning states. Prevents false alarm spam.

### Hysteresis
The dependence of a system state on its operational history. Used in SafeHome's Risk Engine to prevent rapid bouncing between `NORMAL` and `CRITICAL` alert levels during marginal sensor readings.

### Idempotency
An API property where executing an operation multiple times produces identical results as executing it once. Enforced in SafeHome using UUID-based event deduplication.

### Risk Engine
A deterministic software evaluator on the Gateway that correlates events from multiple sensors/cameras, calculates a system risk score (0-100), and maintains system threat levels.

### Safety Policy Firewall
An isolated, read-only decision module that validates high-level software commands against hardcoded safety invariants before allowing physical hardware actuation.
