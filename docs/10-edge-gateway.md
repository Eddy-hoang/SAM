# 10 - Edge Gateway Subsystem Specification

> **Document Status:** `[DECISION]` Gateway Software Architecture  
> **Deployment Target:** Raspberry Pi 4B (4GB RAM) / Intel N100 Mini PC running Ubuntu 22.04 LTS  

---

## 1. Modular Subsystem Architecture

The Edge Gateway is designed as an isolated, containerized application with 8 core micro-modules:

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Edge Gateway System                              │
├──────────────────────┬──────────────────────┬───────────────────────────────┤
│ 1. Device Manager    │ 2. Message Receiver  │ 3. Event Processing Engine    │
│  (MAC/IP Registry)   (MQTT / Serial Bridge)  (Validation & Normalization)   │
├──────────────────────┼──────────────────────┼───────────────────────────────┤
│ 4. Deterministic     │ 5. Storage Adapter   │ 6. API Server & Auth          │
│    Risk Engine       │  (SQLite/TimescaleDB)|  (REST HTTP Endpoints)        │
├──────────────────────┼──────────────────────┼───────────────────────────────┤
│ 7. Realtime Server   │ 8. Notification Mgr  │ 9. Safety Policy Firewall     │
│  (WebSocket Push)    │  (Local Siren/Push)  │  (ACTUATOR VALIDATION LAYER)  │
└──────────────────────┴──────────────────────┴───────────────────────────────┘
```

---

## 2. Ingest, Validation & Normalization Pipeline

```mermaid
graph TD
    IN1["ESP-NOW Frame (Serial)"] --> RX["Message Receiver Module"]
    IN2["MQTT Telemetry Topic"] --> RX
    
    RX --> VAL{"Validate Payload Signature & Schema"}
    VAL -- Invalid / Bad HMAC --> DROP["Log Security Warning & Drop Frame"]
    VAL -- Valid --> NORM["Normalize into Universal SystemEvent"]
    
    NORM --> DEDUP{"Check Event UUID in LRU Cache"}
    DEDUP -- Duplicate --> DISCARD["Discard Duplicate Event"]
    DEDUP -- Unique --> PROC["Forward to Event Processor Queue"]
    
    PROC --> PERSIST["Storage Adapter -> SQLite DB"]
    PROC --> RISK["Deterministic Risk Engine"]
    RISK --> WS["Realtime WS Broadcast to Dashboard"]
```

---

## 3. Resilience & Recovery Capabilities

1. **Automatic Startup Recovery:** Gateway operates under `systemd` process supervision with auto-restart on panic/crash within 2 seconds.
2. **Offline Buffer Queuing:** If the local database locks or disk writes fail, incoming normalized events are temporarily appended to an in-memory Ring Buffer (capacity: 10,000 events).
3. **Hardware Watchdog:** Raspberry Pi hardware watchdog timer enabled (`bcm2835_wdt`) to trigger physical reboot if system locks up completely for $>15$ seconds.
