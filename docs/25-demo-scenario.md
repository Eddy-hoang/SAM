# 25 - Competition Demo Scenarios & Walkthrough

> **Document Status:** `[DECISION]` Live Demonstration Script  
> **Target Event:** Danang AI4Life Competition  

---

## 1. Demo Scenario Overview

To demonstrate system capabilities to competition judges in under 7 minutes, the presentation executes 4 distinct live test scenarios:

```text
  [Scenario 1: Idle Normal] ──> [Scenario 2: Person Incursion] ──> [Scenario 3: Emergency Smoke] ──> [Scenario 4: Wi-Fi Outage Failover]
```

---

## 2. Detailed Scenario Scripts

### Scenario 1: Normal System Idle State
* **Initial State:** All nodes powered on, Wi-Fi connected, System Risk Score = `0` (`NORMAL`).
* **Trigger:** Presenter moves naturally in non-restricted zone (Background).
* **Expected Event:** Periodic `DEVICE_HEARTBEAT` emitted; score remains `NORMAL`.
* **Expected Action:** None.
* **Expected UI:** Dashboard displays green status gauge; node icons glow green.
* **Expected Logs:** `[INFO] Heartbeat received from ESP32CAM-ZONE1`.

---

### Scenario 2: Unauthorized Person Detection (Intrusion Flow)
* **Initial State:** Risk Level = `NORMAL` ($S=0$).
* **Trigger:** Presenter steps into restricted camera area holding a target prop.
* **Expected Event:**
  * Frame 1 & 2: Score = 0.88 (Observation held in RAM).
  * Frame 3: Temporal Engine validates match $\rightarrow$ Emits `HUMAN_DETECTION` ($Confidence = 0.91$).
* **Expected Risk:** Risk Engine correlates event $\rightarrow$ Risk Score jumps to `65` (`HIGH`).
* **Expected Action:** Gateway sends chime signal; Dashboard sounds audio beep.
* **Expected UI:** Dashboard gauge sweeps to Orange `HIGH`; camera snapshot card pops up.
* **Expected Logs:** `[INFO] EVENT_TRIGGERED: HUMAN_DETECTION (Confidence: 0.91)`.

---

### Scenario 3: Critical Emergency Smoke Event (<50ms Fast-Path)
* **Initial State:** Risk Level = `HIGH` ($S=65$).
* **Trigger:** Test smoke source activated near MQ-2 Smoke Sensor Node.
* **Expected Event:** Sensor ADC reading exceeds 2500 threshold $\rightarrow$ Generates `SMOKE_DETECTION`.
* **Expected Risk:** Immediate Risk Score override to `100` (`CRITICAL`).
* **Expected Action:**
  * **ESP-NOW Fast Path (<50ms):** Direct P2P frame sent to Local Alarm Node $\rightarrow$ **105dB Physical Siren Sounds Instantly**.
  * **Gateway Path (<100ms):** Event persisted; emergency UI state triggered.
* **Expected UI:** Screen flashes Red; full-screen Emergency Banner displays "CRITICAL SMOKE DETECTED".
* **Expected Logs:** `[CRITICAL] ESP-NOW Direct Emergency Frame Received -> Siren Actuated (14ms)`.

---

### Scenario 4: Wi-Fi Router Outage Resilience (The "Judge Killer Test")
* **Initial State:** Physical Siren sounding in Scenario 3.
* **Trigger:** Presenter physically pulls out the Wi-Fi Access Point power cable (Total Wi-Fi Crash).
* **Expected Action:**
  * Presenter triggers a secondary door sensor.
  * Sensor node fails to connect to Wi-Fi, immediately fails over to ESP-NOW unicast mode.
  * **Local Alarm Siren sounds continuously** despite complete loss of Wi-Fi router.
* **Expected UI:** Dashboard displays "Gateway Offline - Direct Mesh Alarm Active".
* **Impact on Jury:** Conclusively proves system is **Edge-First** and does not depend on cloud or Wi-Fi router stability for life safety.
