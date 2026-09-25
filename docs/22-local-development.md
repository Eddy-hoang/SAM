# 22 - Local Development & Setup Guide

> **Document Status:** `[DECISION]` Developer Environment Setup  

---

## 1. Prerequisites & Toolchain Setup

To set up a local development PC for hardware simulation and gateway execution, install:
* **Operating System:** Windows 11 / Linux (Ubuntu 22.04) / macOS
* **Embedded Toolchain:** ESP-IDF v5.1+ or VS Code with PlatformIO extension
* **Backend Tools:** Node.js v20+ LTS, Docker Desktop, Mosquitto MQTT Client (`mosquitto_pub` / `mosquitto_sub`)
* **Python Utilities:** Python 3.10+ (for AI model conversion & TFLite quantizer scripts)

---

## 2. Environment Architecture & Mocking

When physical ESP32 hardware is unavailable, developers use the **Node Telemetry Simulator**:

```text
  ┌────────────────────────────────────────────────────────┐
  │              Local Developer PC                        │
  │                                                        │
  │  ┌────────────────────┐      ┌──────────────────────┐  │
  │  │  ESP32 Node Mock   │─────>│ Local Edge Gateway   │  │
  │  │  (Python Script)   │ MQTT │ (Node.js REST / WS)  │  │
  │  └────────────────────┘      └──────────┬───────────┘  │
  │                                         │ WebSockets   │
  │                              ┌──────────▼───────────┐  │
  │                              │ Vite React Dashboard │  │
  │                              └──────────────────────┘  │
  └────────────────────────────────────────────────────────┘
```

---

## 3. Recommended Workspace Layout

```text
/ (Project Root)
├── README.md                  # Project Entry Blueprint
├── docs/                      # Technical Documentation Suite
├── config/                    # Gateway & Network Config Templates
│   ├── mosquitto.conf
│   └── gateway.config.json
└── tools/                     # Hardware Simulators & Test Scripts (Non-code mocks)
    └── mock_event_publisher.py
```
