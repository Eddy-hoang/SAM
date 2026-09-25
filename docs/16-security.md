# 16 - Security Architecture & Threat Model

> **Document Status:** `[DECISION]` Security & Cryptographic Specification  
> **Threat Modeling Methodology:** STRIDE Framework  

---

## 1. STRIDE Threat Model & Countermeasures Matrix

| Threat Category | Specific Attack Scenario | Impact | System Mitigation / Countermeasure | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Spoofing** | Attacker spoofs MAC address of sensor node to send fake smoke alerts. | High (False Panic) | ESP-NOW AES-128 LMK encryption + Pre-shared node authentication key. | `[DECISION]` |
| **Tampering** | Attacker intercepts and modifies event metadata in transit over Wi-Fi. | High (Data Corruption) | TLS 1.3 / HTTPS for API; HMAC-SHA256 signature verification on MQTT topics. | `[DECISION]` |
| **Repudiation**| Malicious user disarms system and denies action. | Medium (Audit Failure) | Immutable `audit_logs` table recording User ID, IP, and timestamp for all actions. | `[DECISION]` |
| **Info Leak** | Wireless eavesdropping captures raw video streams over air. | High (Privacy Violation)| Zero raw video streaming over network; on-device AI generates metadata only. | `[DECISION]` |
| **Denial of Svc**| Attacker floods Gateway API with HTTP requests or RF jammer blocks 2.4GHz. | Critical (System Blindness) | Gateway IP Rate Limiting; ESP-NOW direct offline siren trigger bypasses Gateway. | `[DECISION]` |
| **Elevation** | Attacker bypasses UI to execute actuator commands directly. | Critical (Hardware Override)| Hardcoded Safety Policy firewall validates all API actuation requests. | `[DECISION]` |

---

## 2. Cryptographic Key Management Lifecycle

```text
                  ┌─────────────────────────────────────────┐
                  │      Factory Key Generation             │
                  │ Hardware RNG generates 128-bit PMK/LMK   │
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │       ESP32 eFuse NVS Storage           │
                  │ Protected via Flash Encryption (AES-XTS)│
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │     Periodic Key Rotation (30 Days)     │
                  │ Gateway re-negotiates LMK over ESP-NOW  │
                  └─────────────────────────────────────────┘
```

---

## 3. Secure Firmware Considerations

1. **ESP32-S3 Secure Boot v2:** Enables RSA-3072 signature verification during bootloader execution to prevent loading un-signed malicious firmware binaries.
2. **Flash Encryption:** Enables AES-128-XTS hardware encryption of internal SPI Flash. Firmware binaries and NVS encryption keys cannot be dumped via physical JTAG/UART pins.
