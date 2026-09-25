# 00 - Project Overview

> **Document Status:** `[DECISION]` Baseline Specification  
> **Target Audience:** All Engineering Roles, Competition Jury, Technical Leads  

---

## 1. Vision & Purpose

**SafeHome AI Mesh** is an edge-first, privacy-respecting, intelligent home safety system engineered to handle critical physical security and environmental hazards (e.g., intruding unauthorized persons, fire/smoke detection, gas leaks, perimeter breaches) with high reliability and low latency.

The system is specifically designed for real-world deployment constraints where Internet connectivity may fail, local Wi-Fi networks may become congested, and cloud services add unacceptable latency or privacy risks.

---

## 2. Target User & Operational Environment

### Target User Persona
* **Primary:** Homeowners and residential occupants seeking reliable, high-speed safety alerts without subscription fees or continuous cloud video streaming.
* **Secondary:** Small office / home office (SOHO) managers requiring localized physical access and safety monitoring.
* **Evaluators:** Competition judges (Danang AI4Life) evaluating technical innovation, edge AI feasibility, distributed architecture, and presentation clarity.

### Operational Environment
* **Physical Space:** Typical 2–3 story residential house or apartment unit.
* **Network Context:** Heterogeneous indoor environment with Wi-Fi interference, thick concrete walls, and potential AC mains power disruptions.
* **Hardware Footprint:** Low-power microcontroller nodes (ESP32-S3, ESP32-C3) paired with an edge gateway (Raspberry Pi / Mini PC).

---

## 3. System Scope vs. Non-Scope

### In-Scope (What SafeHome AI Mesh Build)
* On-device vision AI for person & hazard detection using ESP32-S3 CAM.
* Low-latency (<50ms) direct peer-to-peer emergency alerting over ESP-NOW.
* Temporal event generation pipeline to debounce camera noise and eliminate duplicate alert spam.
* Local Edge Gateway housing a deterministic Risk Engine and local time-series database.
* Safety Policy validation layer that strictly prevents non-deterministic AI outputs from triggering physical actuators directly.
* Real-time monitoring dashboard with live alert feeds, system health metrics, and device topology maps.

### Out-of-Scope (Explicit Non-Goals)
* Continuous 24/7 4K cloud video streaming/recording (NVR replacement).
* Direct integration with public emergency services (e.g., automated 911 / 114 dialing).
* Proprietary smart home ecosystem lock-in (e.g., Apple HomeKit MFi hardware chips).
* Full facial recognition or biometric identity tracking (due to memory limitations on ESP32-S3 and high privacy risks).

---

## 4. Problem-to-Implementation Mapping Flow

```text
    ┌────────────────────────────────────────────────────────┐
    │                       Problem                          │
    │  Cloud latency, Wi-Fi outage risks, AI false alarms    │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                     Requirement                        │
    │ Sub-second offline response, zero false-alarm spam,    │
    │ deterministic safety override                          │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                  System Capability                     │
    │ Dual-path networking (ESP-NOW + Wi-Fi), Edge AI state  │
    │ engine, Gateway Risk Engine                            │
    └───────────────────────────┬────────────────────────────┘
                                │
                                ▼
    ┌────────────────────────────────────────────────────────┐
    │                     Implementation                     │
    │ ESP32-S3 TFLite-Micro model, ESP-NOW MAC peer relay,   │
    │ Deterministic Rule Evaluator, Gateway WS server        │
    └────────────────────────────────────────────────────────┘
```

---

## 5. Beginner Onboarding: "Where Should I Start?"

If you are joining the project as a new team member, use the role-based quick start roadmap below:

| Role | Step 1 | Step 2 | Step 3 |
| :--- | :--- | :--- | :--- |
| **Embedded Dev** | Read [`05-esp32-edge-device.md`](file:///d:/SAM/docs/05-esp32-edge-device.md) | Study [`09-esp-now-emergency-path.md`](file:///d:/SAM/docs/09-esp-now-emergency-path.md) | Check [`23-hardware-bom.md`](file:///d:/SAM/docs/23-hardware-bom.md) |
| **AI / ML Dev** | Read [`06-ai-pipeline.md`](file:///d:/SAM/docs/06-ai-pipeline.md) | Study [`07-event-generation.md`](file:///d:/SAM/docs/07-event-generation.md) | Review [`adr/0003-ai-inference-strategy.md`](file:///d:/SAM/docs/adr/0003-ai-inference-strategy.md) |
| **Backend Dev** | Read [`03-system-architecture.md`](file:///d:/SAM/docs/03-system-architecture.md) | Study [`10-edge-gateway.md`](file:///d:/SAM/docs/10-edge-gateway.md) & [`12-risk-engine.md`](file:///d:/SAM/docs/12-risk-engine.md) | Inspect [`13-data-model.md`](file:///d:/SAM/docs/13-data-model.md) & [`14-api-specification.md`](file:///d:/SAM/docs/14-api-specification.md) |
| **Frontend Dev** | Read [`15-realtime-communication.md`](file:///d:/SAM/docs/15-realtime-communication.md) | Study [`14-api-specification.md`](file:///d:/SAM/docs/14-api-specification.md) | Review [`25-demo-scenario.md`](file:///d:/SAM/docs/25-demo-scenario.md) |
