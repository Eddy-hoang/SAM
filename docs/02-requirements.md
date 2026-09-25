# 02 - System Requirements

> **Document Status:** `[DECISION]` Baseline Specification  
> **Prioritization Standard:** RFC 2119 (MUST, SHOULD, COULD, FUTURE)  

---

## 1. Functional Requirements (FR)

### REQ-001: Edge Vision Inference
* **Description:** The ESP32-S3 CAM node MUST perform local computer vision inference to detect unauthorized humans or hazard signatures without streaming video to external servers.
* **Priority:** `MUST`
* **Source:** Core Safety Requirement
* **Acceptance Criteria:** Frame processing rate $\ge$ 5 FPS at QVGA resolution with confidence score output per frame.
* **Related Component:** `ESP32-S3 CAM Firmware`, `AI Pipeline`

### REQ-002: Direct ESP-NOW Emergency Triggering
* **Description:** Upon detecting a critical hazard (e.g. smoke or verified intrusion), the sensor or camera node MUST transmit an encrypted ESP-NOW emergency frame directly to the Local Alarm Node and Gateway.
* **Priority:** `MUST`
* **Source:** Emergency Safety Path Specification
* **Acceptance Criteria:** Alarm node receives and sounds siren within $<50\text{ ms}$ of local event generation, independent of Wi-Fi router status.
* **Related Component:** `Sensor Node`, `Local Alarm Node`, `ESP-NOW Mesh`

### REQ-003: Gateway Risk Scoring & Event Aggregation
* **Description:** The Edge Gateway MUST correlate incoming temporal events from multiple sensors/cameras, calculate a dynamic risk score (0 to 100), and determine the system threat level (NORMAL, ELEVATED, HIGH, CRITICAL).
* **Priority:** `MUST`
* **Source:** Gateway Architectural Specification
* **Acceptance Criteria:** Multiple correlated events (e.g. PIR motion + Camera human detection within 10s) escalate threat level from ELEVATED to HIGH within $<100\text{ ms}$.
* **Related Component:** `Edge Gateway`, `Risk Engine`

### REQ-004: Real-Time Monitoring Dashboard
* **Description:** The system MUST provide a local web-based dashboard displaying live device status, event logs, risk level gauges, and manual safety override controls.
* **Priority:** `MUST`
* **Source:** User Interface Requirement
* **Acceptance Criteria:** UI updates via WebSockets within $<100\text{ ms}$ of gateway event emission; dashboard works entirely offline.
* **Related Component:** `Realtime Server`, `Web Dashboard`

### REQ-005: AI-Assisted Risk Analysis (LLM Advisory)
* **Description:** The Gateway MAY query an optional local or cloud LLM/VLM service to generate natural language safety summaries and recommended actions for human operators.
* **Priority:** `SHOULD`
* **Source:** Advisory Intelligence Concept
* **Acceptance Criteria:** LLM output is marked strictly as "ADVISORY ONLY" and cannot trigger hardware GPIO actuation directly.
* **Related Component:** `AI Service`, `Risk Engine`

---

## 2. Non-Functional Requirements (NFR)

### REQ-010: Emergency Response Latency
* **Description:** End-to-end latency from physical sensor trigger (e.g., reed switch open or smoke detected) to local siren actuation MUST NOT exceed 100 milliseconds.
* **Priority:** `MUST`
* **Source:** Critical Safety Constraint
* **Acceptance Criteria:** Tested and verified via oscilloscope/logic analyzer over 100 consecutive trials.
* **Related Component:** `ESP-NOW Network`, `Local Alarm Node`

### REQ-011: Offline Operational Autonomy
* **Description:** All primary security, detection, logging, and siren alert functions MUST operate continuously during total loss of WAN/Internet connectivity.
* **Priority:** `MUST`
* **Source:** Resilience Constraint
* **Acceptance Criteria:** System maintains full local alert loop when WAN Ethernet cable is physically disconnected from Gateway.
* **Related Component:** `Edge Gateway`, `ESP-NOW Mesh`, `Local DB`

### REQ-012: False Alarm Mitigation Rate
* **Description:** The system MUST reduce camera false alarms caused by temporary occlusion, shadow change, or single-frame inference anomalies by at least 95% compared to raw frame detection.
* **Priority:** `MUST`
* **Source:** User Experience / System Quality
* **Acceptance Criteria:** Temporal filter engine requires $N=3$ consecutive positive detections before transitioning to state `OBSERVED`.
* **Related Component:** `Event Generator`, `AI Pipeline`

---

## 3. AI & Hardware Requirements

### REQ-020: ESP32-S3 Memory & Processing Budget
* **Description:** The vision inference pipeline MUST operate within 8MB external PSRAM and 512KB internal SRAM, reserving at least 150KB SRAM for Wi-Fi/ESP-NOW network stacks.
* **Priority:** `MUST`
* **Source:** Hardware Limitation Specs
* **Acceptance Criteria:** No heap allocation failure or PSRAM bus panic during 24-hour continuous inference.
* **Related Component:** `ESP32-S3 Edge Device`

### REQ-021: Model Quantization & Size
* **Description:** Vision models deployed to ESP32-S3 MUST be INT8 quantized TensorFlow Lite Micro models, with binary file size not exceeding 2.5MB.
* **Priority:** `MUST`
* **Source:** Edge AI Budget
* **Acceptance Criteria:** Quantized model accuracy within 3% of FP32 baseline on evaluation dataset.
* **Related Component:** `AI Pipeline`

---

## 4. Security & Privacy Requirements

### REQ-030: Replay Attack Protection over ESP-NOW
* **Description:** Emergency frames transmitted via ESP-NOW MUST include a monotonically increasing 32-bit sequence counter and AES-128 cryptographic payload tag to prevent replay attacks.
* **Priority:** `MUST`
* **Source:** Security Threat Model
* **Acceptance Criteria:** Replayed emergency frames with old sequence numbers are immediately dropped by Gateway/Alarm nodes.
* **Related Component:** `ESP-NOW Emergency Path`, `Security Module`

### REQ-031: On-Device Camera Privacy (No Unsanctioned Streaming)
* **Description:** Raw camera frame buffers MUST be processed in local RAM and discarded immediately after inference. Raw images MUST NOT be saved to local storage or transmitted over network unless explicitly requested during an active ALARM state.
* **Priority:** `MUST`
* **Source:** Privacy Policy
* **Acceptance Criteria:** Network packet analysis verifies zero video payload transmission during normal state.
* **Related Component:** `ESP32-S3 CAM Firmware`, `Privacy Module`

---

## 5. Traceability Matrix Summary

| Requirement ID | Primary Component | Verification Test ID | Competition Demo Step |
| :--- | :--- | :--- | :--- |
| **REQ-001** | `ESP32-S3 CAM Firmware` | `TEST-AI-01` | `DEMO-STEP-02` |
| **REQ-002** | `ESP-NOW Mesh` | `TEST-NET-02` | `DEMO-STEP-03` |
| **REQ-003** | `Gateway Risk Engine` | `TEST-GW-01` | `DEMO-STEP-04` |
| **REQ-004** | `Web Dashboard` | `TEST-UI-01` | `DEMO-STEP-05` |
| **REQ-010** | `Local Alarm Node` | `TEST-PERF-01` | `DEMO-STEP-03` |
| **REQ-011** | `Edge Gateway` | `TEST-FAIL-01` | `DEMO-STEP-06` |
| **REQ-030** | `Security Module` | `TEST-SEC-01` | `DEMO-STEP-07` |
