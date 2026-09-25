# 18 - Fault Tolerance & Recovery Matrix

> **Document Status:** `[DECISION]` Resilience & Failure Mode Specification  

---

## 1. System Failure Mode & Recovery Matrix

| Failure Mode | Detection Mechanism | Automated Recovery Action | Impact on User / System |
| :--- | :--- | :--- | :--- |
| **Wi-Fi AP Outage / Crash** | Node MQTT disconnect ping timeout (10s) | Sensor nodes automatically failover to direct ESP-NOW P2P alert channel. | Dashboard UI loses real-time telemetry updates, BUT emergency sirens still sound within $<50\text{ ms}$ on sensor trigger. |
| **Gateway Hardware Crash** | Node ESP-NOW ACK failure | Sensor nodes re-route unicast emergency frames to standalone Local Alarm Siren Node MAC. | Historical logging stops temporarily; emergency physical alerting remains 100% operational. |
| **Database Disk Failure** | Gateway SQLite I/O Write Error | Ingest pipeline redirects normalized events to an in-memory Ring Buffer (10k capacity). | Dashboard shows warning gauge; system attempts sqlite db repair / vacuum. |
| **AI Service Crash** | Subprocess health ping fails | Risk Engine drops AI weight factor to 0.0 and relies 100% on hardcoded deterministic rules. | Advanced VLM summaries disabled; core security rules continue functioning normally. |
| **Sensor Node Power Loss** | Missed 3 consecutive Heartbeats (90s) | Gateway updates device status to `OFFLINE` and generates `WARNING_DEVICE_DISCONNECTED`. | UI alerts user to replace battery / check sensor node wiring. |
| **ESP-NOW RF Jamming** | High packet drop rate ($>80\%$) | Gateway alerts `RF_INTERFERENCE_HIGH` and switches node sampling to maximum frequency. | System logs potential RF tampering incident. |
| **WebSocket Disconnect** | Browser WS `onclose` event listener | UI dashboard executes exponential backoff reconnect loop ($1\text{s} \rightarrow 30\text{s}$). | Dashboard temporarily displays "Reconnecting..." banner; auto-resyncs state on connect. |
| **Sudden AC Power Cut** | Gateway UPS / Power loss interrupt | Gateway executes clean shutdown script; ESP32 nodes powered by battery backup buffers. | System continues running on localized battery power for up to 6 hours. |
| **Gateway Clock Drift** | NTP sync check fails | Event processor uses monotonic relative timestamps (`uptime_ms`) for sequence order. | Log timestamps may diverge slightly, but event ordering remains strictly deterministic. |
