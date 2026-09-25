# 04 - Component Architecture

> **Document Status:** `[DECISION]` Baseline Component Specifications  

---

## 1. Component Specification Template

Each system component is defined strictly using the 8-point architectural template:
1. **Purpose**
2. **Responsibilities**
3. **Inputs**
4. **Outputs**
5. **Dependencies**
6. **Failure Modes**
7. **Security Considerations**
8. **Scaling Considerations**

---

## 2. Edge Layer Components

### 2.1 ESP32-S3 Edge Vision Node
* **Purpose:** Performs localized camera frame acquisition, Edge AI inference (person/hazard detection), temporal filtering, and emergency broadcast.
* **Responsibilities:** Frame buffer management, model execution, debounce state machine, AES-128 ESP-NOW framing, heartbeats.
* **Inputs:** Raw CMOS sensor signals (OV2640 over DVP interface), power, ESP-NOW configuration.
* **Outputs:** Encrypted ESP-NOW emergency frames, Wi-Fi MQTT telemetry packets, diagnostic serial logs.
* **Dependencies:** ESP-IDF v5.x, TensorFlow Lite Micro, ESP-NOW drivers.
* **Failure Modes:** Camera lens occlusion, PSRAM memory fragmentation, thermal throttling under direct sunlight.
* **Security Considerations:** Firmware readout protection (Flash Encryption), AES-128 key stored in NVS, sequence number generation.
* **Scaling Considerations:** Up to 8 camera nodes per Gateway area without Wi-Fi channel saturation.

### 2.2 Environmental & Perimeter Sensor Node
* **Purpose:** Captures discrete binary (PIR, Reed) or analog (MQ-2 smoke/gas) environmental safety data.
* **Responsibilities:** Sensor sampling, threshold crossing detection, direct ESP-NOW alerting, power management (sleep cycles).
* **Inputs:** GPIO pin changes (reed switch interrupt), analog ADC values (smoke density).
* **Outputs:** ESP-NOW emergency frames, periodic battery health status.
* **Dependencies:** ESP32-C3 / ESP8266 silicon, ADC calibration tables.
* **Failure Modes:** Sensor element drift, depleted battery, false contact bounce on mechanical switches.
* **Security Considerations:** Sequence counter anti-replay protection, pre-shared ESP-NOW network key.
* **Scaling Considerations:** Up to 32 low-power sensor nodes per ESP-NOW mesh sector.

---

## 3. Communication & Gateway Components

### 3.1 Edge Gateway Event Processor
* **Purpose:** Ingests raw serial/MQTT/ESP-NOW messages, validates schemas, performs deduplication, and normalizes into formal system Events.
* **Responsibilities:** Message validation, sequence order check, deduplication window management, standard JSON event generation.
* **Inputs:** ESP-NOW frames (via Serial bridge), MQTT payload topics (`safehome/device/+/telemetry`).
* **Outputs:** Normalized `SystemEvent` instances routed to Risk Engine and DB.
* **Dependencies:** Gateway runtime (Node.js/Go/Python), MQTT broker (Mosquitto).
* **Failure Modes:** Serial buffer overflow, invalid JSON payload injection, clock drift on ingest timestamping.
* **Security Considerations:** Schema validation prevents injection; HMAC verification drops unauthorized messages.
* **Scaling Considerations:** Single instance handles up to 500 events/sec (far exceeding standard home loads).

### 3.2 Deterministic Risk Engine
* **Purpose:** Evaluates normalized system events against rule matrices to determine real-time home risk score and threat state.
* **Responsibilities:** Multi-event correlation (e.g. PIR + Motion), risk score calculation (0–100), state transition management (NORMAL $\rightarrow$ CRITICAL).
* **Inputs:** Normalized `SystemEvent` objects, stored safety rules, hysteresis timers.
* **Outputs:** System Risk State updates, actuation commands to Safety Policy.
* **Dependencies:** Local Database, In-Memory State Storage (Redis / Map).
* **Failure Modes:** Corrupted rule configuration file, state deadlocks.
* **Security Considerations:** Hardcoded deterministic bounds prevent rule override via external API without administrative password.
* **Scaling Considerations:** In-memory rule table execution completes in $<1\text{ ms}$.

### 3.3 Safety Policy & Command Validator
* **Purpose:** Acts as a strictly isolated firewall between high-level intelligence (AI/LLM/Risk Engine) and physical actuators.
* **Responsibilities:** Validating whether an actuation request (e.g. unlock door, activate siren) complies with hardcoded safety invariants.
* **Inputs:** Proposed commands from Risk Engine or AI Service.
* **Outputs:** Approved Actuation Commands sent to Gateway Hardware Bridge, or Rejected Command Logs.
* **Dependencies:** Hardcoded Policy Module (read-only binary logic).
* **Failure Modes:** Rejection of valid commands due to misconfigured policy bounds.
* **Security Considerations:** AI services CANNOT bypass this validator under any operational mode.
* **Scaling Considerations:** Zero external dependencies; execution is microsecond-level.

---

## 4. Application & Interface Components

### 4.1 Realtime WebSocket Broker & API Server
* **Purpose:** Exposes RESTful management endpoints and pushes instantaneous WebSocket telemetry to operator dashboards.
* **Responsibilities:** Client connection management, JWT auth, WebSocket topic subscription filtering, HTTP REST routing.
* **Inputs:** Operator HTTP requests, Risk Engine state change broadcasts.
* **Outputs:** JSON REST responses, live WebSocket frame feeds.
* **Dependencies:** Node.js Express/Fastify or Python FastAPI, WS library.
* **Failure Modes:** WebSocket client disconnects during network handover, API port exhaustion.
* **Security Considerations:** TLS/WSS encryption, JWT token validation, IP rate limiting.
* **Scaling Considerations:** Accommodates up to 50 concurrent dashboard client sessions natively.
