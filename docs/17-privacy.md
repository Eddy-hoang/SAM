# 17 - Privacy Architecture & Data Governance

> **Document Status:** `[DECISION]` Privacy-by-Design Specification  

---

## 1. On-Device Metadata Architecture

SafeHome AI Mesh strictly enforces a **Zero-Video-Streaming Privacy Architecture**. Camera sensors function exclusively as intelligent optical event detectors:

```text
Physical Scene ──> Frame RAM (Volatile) ──> AI Inference ──> Metadata JSON ──> RAM Wiped
                           │
                           └─── Raw Frame NEVER leaves RAM during Normal State
```

---

## 2. Image Retention & Evidentiary Snapshot Rules

1. **Normal Operational State:** Raw camera frame buffers reside in volatile PSRAM for $< 200\text{ ms}$ during inference processing and are immediately overwritten by the next frame DMA buffer. Zero disk retention.
2. **Alert Evidentiary Snapshot:**  
   * *Trigger:* ONLY when a `CRITICAL` or `HIGH` risk alert is validated by the Risk Engine.
   * *Action:* The camera node captures a single JPEG snapshot ($640 \times 480$), encrypts it with AES-128, and transmits it to the local Gateway database.
   * *Retention Lifetime:* Stored locally on Gateway for **7 days**, after which an automated cron job performs cryptographically secure file deletion (`shred / wipe`).

---

## 3. Data Minimization & Cloud Insulation Matrix

| Data Type | Stored on ESP32 Node? | Transmitted to Gateway? | Transmitted to Cloud? | User Access Level |
| :--- | :--- | :--- | :--- | :--- |
| **Raw Video Stream** | NO | NO | **NEVER** | None |
| **Alert JPEG Frame** | NO (In Memory Only) | YES (Encrypted Local DB) | OPTIONAL (User Opt-in Only) | Admin Owner Only |
| **Detection Metadata**| NO | YES (Local DB) | NO | All Dashboard Users |
| **Sensor Telemetry** | NO | YES (Local DB) | NO | All Dashboard Users |
