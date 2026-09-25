# 24 - Project Roadmap & MVP Definition

> **Document Status:** `[DECISION]` Phased Execution Roadmap  
> **Target Milestone:** Danang AI4Life Competition Presentation  

---

## 1. MVP Definition vs. Stretch Goals

### Minimum Viable Product (MVP) Scope `[MUST COMPLETE FOR DEMO]`
```text
ESP32 CAM (Person Detect) ──> ESP-NOW Frame ──> Siren Node (<50ms Siren Sound)
                                  │
                                  └───> Gateway (Risk Score 75) ──> Web UI Red Alert
```
* **Hardware:** 1 ESP32-S3 CAM, 1 Smoke Sensor Node, 1 Local Siren Node, 1 Gateway (Raspberry Pi/Laptop).
* **AI:** INT8 MobileNet-V2 human presence detection running locally at 5 FPS.
* **Network:** ESP-NOW emergency fast-path working completely offline without internet.
* **Gateway & UI:** Local Risk Engine calculating threat scores, WebSocket streaming red alert gauges to React dashboard.

### Stretch Goals `[FUTURE / OPTIONAL]`
* LLM integration for natural language audio alerts ("An unknown person was detected near the back door at 2:15 AM").
* Multi-camera spatial tracking across 4+ rooms.
* Mobile Push Notifications (via Apple APNS / Firebase FCM).

---

## 2. Phased Development Roadmap (Phase 0 to Phase 12)

### Phase 0: Technical Blueprint & Documentation `[COMPLETED]`
* **Goal:** Create complete system architecture specs without writing application code.
* **Deliverables:** 28 markdown document files, Mermaid diagrams, schemas, ADRs.
* **Acceptance Criteria:** Architectural review passes with zero unaddressed ambiguities.

### Phase 1: Hardware Bring-Up & Power Verification
* **Goal:** Verify electrical stability and flash firmware bootloaders on ESP32 silicon.
* **Tasks:** Test power supply decoupling caps; verify PSRAM 8MB detection.
* **Deliverables:** Working ESP-IDF hello-world with PSRAM diagnostic logs.

### Phase 2: ESP32-S3 Camera & Frame Acquisition
* **Goal:** Configure OV2640 camera driver over DVP interface with DMA double-buffering.
* **Deliverables:** Clean $320 \times 240$ RGB565 frame capture in PSRAM.

### Phase 3: ESP-NOW Emergency Mesh Channel
* **Goal:** Implement encrypted ESP-NOW peer-to-peer sending between nodes.
* **Deliverables:** Binary emergency frame sent from Sensor to Siren Node ($<20\text{ ms}$ latency).

### Phase 4: On-Device TFLite Micro Inference & Temporal Filter
* **Goal:** Deploy INT8 quantized MobileNet model and integrate 3-frame temporal filter.
* **Deliverables:** Node emits `HUMAN_DETECTION` JSON event only after 3 consecutive frames.

### Phase 5: Edge Gateway Ingest & Persistence
* **Goal:** Build Node.js Gateway receiver, deduplicator, and SQLite database storage.
* **Deliverables:** Gateway ingests events, validates schemas, saves to SQLite within $<10\text{ ms}$.

### Phase 6: Deterministic Risk Engine & Safety Policy Firewall
* **Goal:** Implement weighted correlation rules and Safety Policy validator.
* **Deliverables:** Risk Engine transitions threat level from NORMAL to HIGH on correlated events.

### Phase 7: REST API & Realtime WebSocket Server
* **Goal:** Build API endpoints and WebSocket broadcast broker.
* **Deliverables:** Gateway broadcasts JSON updates over WebSocket channel `system:risk`.

### Phase 8: Real-Time Web Dashboard UI
* **Goal:** Build React + Vite dashboard displaying risk gauges and live event logs.
* **Deliverables:** Dashboard updates UI within $<50\text{ ms}$ of WebSocket push.

### Phase 9: System Integration & End-to-End Testing
* **Goal:** Connect all hardware nodes and Gateway into a unified physical demonstration network.
* **Deliverables:** Full end-to-end execution of test matrix (`TEST-E2E-01`).

### Phase 10: Fault Tolerance & Security Hardening
* **Goal:** Test Wi-Fi disconnect, power failure, and anti-replay sequence number checks.
* **Deliverables:** Replayed frames dropped; system operates 100% offline during Wi-Fi outage.

### Phase 11: Demo Script & Jury Presentation Preparation
* **Goal:** Rehearse live competition demo scenarios with judges.
* **Deliverables:** Flawless live execution of 3 competition scenarios.

### Phase 12: Danang AI4Life Competition Presentation
* **Goal:** Present SafeHome AI Mesh live to jury members.
