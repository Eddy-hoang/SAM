# 03 - System Architecture

> **Document Status:** `[DECISION]` Primary System Architecture Specification  
> **Modeling Standard:** C4 Model (Context, Container, Component, Deployment)  

---

## 1. Multi-Layer Functional Breakdown

The SafeHome AI Mesh architecture is organized into 6 distinct, decoupled functional layers:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ 6. Application Layer (Web Monitoring Dashboard, Mobile Alert UI)        │
├──────────────────────────────────────────────────────────────────────────┤
│ 5. Data & Storage Layer (TimescaleDB / SQLite, Event Store, Config DB)   │
├──────────────────────────────────────────────────────────────────────────┤
│ 4. Gateway Processing Layer (Event Normalizer, Risk Engine, Safety Policy)│
├──────────────────────────────────────────────────────────────────────────┤
│ 3. Communication Layer (ESP-NOW Encrypted Fast-Path, MQTT / WebSockets)  │
├──────────────────────────────────────────────────────────────────────────┤
│ 2. Edge Vision & Sensor Layer (ESP32-S3 Edge AI, PIR, Reed, Smoke Nodes)│
├──────────────────────────────────────────────────────────────────────────┤
│ 1. Physical Hardware Layer (ESP32 Silicon, Camera Sensors, Sirens/Relays)│
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 2. C4 Context Diagram (Level 1)

```mermaid
graph TB
    User["Homeowner / Operator"]
    System["SafeHome AI Mesh System\n(Edge-First Local Safety Network)"]
    ExternalAI["Optional External AI Service\n(Cloud VLM / LLM API)"]
    LocalAlarm["Physical Local Sirens / Relays"]

    User -->|Views Alerts, Manages System| System
    System -->|Sounds Emergency Alarm| LocalAlarm
    System -.->|Sends Anonymized Metadata for Deep Analysis| ExternalAI
    ExternalAI -.->|Returns Advisory Insights| System

    classDef primary fill:#2563eb,stroke:#1d4ed8,color:#fff;
    classDef external fill:#475569,stroke:#334155,color:#fff;
    class System primary;
    class ExternalAI,LocalAlarm,User external;
```

---

## 3. C4 Container Diagram (Level 2)

```mermaid
graph TB
    subgraph Edge_Devices ["Edge Device Containers"]
        CAM["ESP32-S3 Vision Node\n(C++/ESP-IDF + TFLite Micro)"]
        SENS["Environmental Sensor Node\n(C++/ESP-IDF)"]
        ALARM["Local Alarm Actuator Node\n(C++/ESP-IDF)"]
    end

    subgraph Edge_Gateway_Node ["Edge Gateway Container Host (Raspberry Pi / Mini PC)"]
        GW_RECV["Message Receiver Service\n(ESP-NOW Serial Bridge & MQTT Broker)"]
        GW_PROC["Event Processing & State Engine"]
        GW_RISK["Deterministic Risk Engine"]
        GW_POL["Safety Policy & Command Validator"]
        GW_API["REST / WebSocket API Server"]
        GW_DB[(Local SQLite / TimeSeries DB)]
    end

    subgraph Client_Applications ["User Interface Containers"]
        DASH["Web Dashboard\n(React / Vite PWA)"]
    end

    %% Network Connections
    CAM -->|ESP-NOW Encrypted Emergency Frame| ALARM
    CAM -->|ESP-NOW / Wi-Fi Telemetry| GW_RECV
    SENS -->|ESP-NOW Emergency Trigger| ALARM
    SENS -->|ESP-NOW Telemetry| GW_RECV

    GW_RECV --> GW_PROC
    GW_PROC --> GW_RISK
    GW_RISK --> GW_POL
    GW_RISK --> GW_DB
    GW_POL -->|Validated Relay Command| GW_RECV
    GW_API --> GW_DB
    GW_PROC -->|WebSocket Event Push| GW_API
    DASH <-->|HTTP REST & WS| GW_API

    classDef edge fill:#1e293b,stroke:#3b82f6,color:#fff;
    classDef gw fill:#0f172a,stroke:#8b5cf6,color:#fff;
    classDef ui fill:#1e1b4b,stroke:#f59e0b,color:#fff;

    class CAM,SENS,ALARM edge;
    class GW_RECV,GW_PROC,GW_RISK,GW_POL,GW_API,GW_DB gw;
    class DASH ui;
```

---

## 4. System Sequence Diagram: End-to-End Emergency Flow

```mermaid
sequenceDiagram
    autonumber
    actor Hazard as Physical Threat / Person
    participant ESP32CAM as ESP32-S3 Vision Node
    participant AlarmNode as Local Alarm Node
    participant Gateway as Edge Gateway
    participant RiskEng as Risk Engine
    participant DB as Local Database
    participant UI as Web Dashboard

    Hazard->>ESP32CAM: Person enters restricted perimeter
    ESP32CAM->>ESP32CAM: Capture Frame -> TFLite Inference (Person > 0.85)
    ESP32CAM->>ESP32CAM: Temporal Filter (3 consecutive matches) -> Generate EVENT
    
    par Priority Emergency Path (<50ms)
        ESP32CAM->>AlarmNode: Send ESP-NOW Emergency Payload (AES-128, Seq# N)
        AlarmNode->>AlarmNode: Verify AES Tag & Sequence Number
        AlarmNode->>AlarmNode: Actuate Siren / Buzzer (GPIO HIGH)
    and Gateway Reporting Path (<100ms)
        ESP32CAM->>Gateway: Send Event Metadata via ESP-NOW/MQTT
        Gateway->>RiskEng: Ingest & Evaluate Event Rules
        RiskEng->>RiskEng: Update Risk Score (ELEVATED -> HIGH)
        RiskEng->>DB: Persist Immutable Event Record
        RiskEng->>UI: Broadcast Realtime Alert via WebSocket
    end
    
    UI->>UI: Play Visual/Audio Alert on Operator Screen
```

---

## 5. Architectural Trade-offs & ADR Summary

1. **Decoupled Edge Vision vs. Direct Video Streaming:**  
   * *Decision:* On-device MobileNet inference generating metadata JSON events instead of streaming 24/7 video.
   * *Trade-off:* Saves network bandwidth and preserves privacy, but prevents remote users from viewing live 4K streams unless an alert state is triggered.
2. **ESP-NOW Mesh vs. Standard Wi-Fi Infrastructure:**  
   * *Decision:* ESP-NOW for emergency path, Wi-Fi for telemetry.
   * *Trade-off:* Requires dedicated MAC peer management on ESP32 nodes, but guarantees sub-50ms transmission even if the home Wi-Fi access point crashes.
