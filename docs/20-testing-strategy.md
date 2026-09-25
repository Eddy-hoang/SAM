# 20 - System Testing Strategy & Validation Matrix

> **Document Status:** `[DECISION]` Comprehensive Test Plan  

---

## 1. Multi-Layer Testing Architecture

```text
  ┌────────────────────────────────────────────────────────┐
  │ 10. Competition Demo Scenario Validation Run (End-to-End)│
  ├────────────────────────────────────────────────────────┤
  │ 9. Hardware-in-the-Loop (HIL) Latency & Load Testing   │
  ├────────────────────────────────────────────────────────┤
  │ 8. Security & Penetration Testing (Replay / Spoofing)   │
  ├────────────────────────────────────────────────────────┤
  │ 7. Network Fault-Injection & Outage Tests              │
  ├────────────────────────────────────────────────────────┤
  │ 6. AI Model Precision & Quantization Benchmark Tests   │
  ├────────────────────────────────────────────────────────┤
  │ 5. API & WebSocket Realtime Integration Tests          │
  ├────────────────────────────────────────────────────────┤
  │ 4. Deterministic Risk Engine Rule Matrix Tests         │
  ├────────────────────────────────────────────────────────┤
  │ 3. Temporal State Machine & Debounce Engine Tests      │
  ├────────────────────────────────────────────────────────┤
  │ 2. ESP-NOW Binary Protocol Framing Unit Tests          │
  ├────────────────────────────────────────────────────────┤
  │ 1. Core Data Model & Schema Validation Unit Tests       │
  └────────────────────────────────────────────────────────┘
```

---

## 2. Test Execution Matrix

| Test ID | Test Category | Target Component | Description & Acceptance Criteria | Requirement Ref |
| :--- | :--- | :--- | :--- | :--- |
| `TEST-AI-01` | AI Model Test | `ESP32-S3 CAM` | Verify TFLite INT8 inference returns $>0.85$ score for human in frame; execution time $<150\text{ ms}$. | REQ-001, REQ-021 |
| `TEST-STATE-01`| Unit Test | `Temporal Engine` | Verify temporal state requires $N=3$ consecutive frames before generating `SYSTEM_EVENT`. | REQ-012 |
| `TEST-NET-01` | Protocol Test | `ESP-NOW Mesh` | Measure ESP-NOW P2P transmission latency from trigger to alarm receive ($<30\text{ ms}$ over 100 trials). | REQ-002, REQ-010 |
| `TEST-SEC-01` | Security Test | `Gateway / Node` | Inject replayed ESP-NOW frame with old sequence number; verify node drops packet. | REQ-030 |
| `TEST-GW-01` | Integration | `Risk Engine` | Correlate PIR motion + Vision event within 10s; verify Risk Score transitions to 65 (`HIGH`). | REQ-003 |
| `TEST-FAIL-01` | Resilience | `System Mesh` | Disconnect Wi-Fi Router during active alert; verify local alarm siren activates normally via ESP-NOW. | REQ-011 |
| `TEST-E2E-01` | End-to-End | `Complete System` | Trigger smoke sensor $\rightarrow$ verify siren sounds $<50\text{ ms}$ and UI updates dashboard $<100\text{ ms}$. | All REQs |

---

## 3. End-to-End Test Execution Trace

```text
[Camera Frame Input] 
       │
       ▼ (TEST-AI-01: Score = 0.88)
[Temporal State Engine] 
       │
       ▼ (TEST-STATE-01: 3 Consecutive Inferences -> Emit Event)
[ESP-NOW Emergency Transport] 
       │
       ▼ (TEST-NET-01: Latency = 14ms -> Local Siren Triggers)
[Gateway Processing Engine] 
       │
       ▼ (TEST-GW-01: Evaluate Rules -> Risk = HIGH -> Save DB)
[WebSocket Server] 
       │
       ▼ (TEST-UI-01: Push JSON -> Render Visual Red Alert Banner)
```
