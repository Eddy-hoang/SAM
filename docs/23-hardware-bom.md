# 23 - Hardware Bill of Materials (BOM) & Pinout Specifications

> **Document Status:** `[DECISION]` Hardware Sourcing & Wiring Specification  

---

## 1. Categorized Hardware Bill of Materials

### Category A: Required Hardware (MVP Prototype Baseline)

| Component Name | Primary Function / Purpose | Qty | Interface | Estimated Cost (VND) | Architectural Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ESP32-S3-WROOM-1-N16R8 CAM** | Edge Vision AI Node + ESP-NOW sender | 2 | DVP / QSPI / USB-C | ~180,000 VND ($7.50) | `[DECISION]` |
| **OV2640 Camera Module** | 2MP CMOS Image Sensor | 2 | 24-Pin DVP Ribbon | Included with CAM | `[DECISION]` |
| **ESP32-C3 / ESP8266 Node MCU** | Environmental Sensor Node | 2 | GPIO / ADC / I2C | ~60,000 VND ($2.50) | `[DECISION]` |
| **MQ-2 Gas / Smoke Sensor** | Combustible gas & smoke detection | 1 | Analog ADC (Pin A0) | ~35,000 VND ($1.50) | `[DECISION]` |
| **HC-SR501 PIR Motion Sensor** | Infra-red human motion detection | 2 | Digital GPIO | ~25,000 VND ($1.00) | `[DECISION]` |
| **Magnetic Door Reed Switch** | Perimeter door/window breach detection| 2 | Digital GPIO Interrupt | ~15,000 VND ($0.60) | `[DECISION]` |
| **Active Electronic Buzzer / Siren**| Local high-decibel audible alarm sounder| 2 | Digital GPIO Relay/Transistor | ~20,000 VND ($0.80) | `[DECISION]` |
| **Raspberry Pi 4B (4GB) or N100** | Edge Gateway Host & DB Server | 1 | Ethernet / USB 3.0 | ~1,400,000 VND ($55.00)| `[DECISION]` |

### Category B: Recommended Hardware (Demo Enhancements)

| Component Name | Purpose | Qty | Interface | Estimated Cost (VND) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **5V 2A Power Adapter + USB-C Cable**| Stable power delivery for ESP32 CAM nodes | 3 | Type-C | ~100,000 VND ($4.00) | `[DECISION]` |
| **1000uF 10V Electrolytic Capacitors**| Filtering Wi-Fi transmit voltage dips | 4 | Direct 5V/GND rail | ~10,000 VND ($0.40) | `[DECISION]` |

---

## 2. ESP32-S3 CAM Pinout Mapping Table

```text
       ┌─────────────────────────────────────────────────────────────┐
       │             ESP32-S3 CAM Pin Assignment Table               │
       ├──────────────────────────┬──────────────────────────────────┤
       │ Function                 │ ESP32-S3 GPIO Pin Number         │
       ├──────────────────────────┼──────────────────────────────────┤
       │ Camera DVP SIOC (SCL)    │ GPIO 39                          │
       │ Camera DVP SIOD (SDA)    │ GPIO 40                          │
       │ Camera VSYNC             │ GPIO 38                          │
       │ Camera HREF              │ GPIO 47                          │
       │ Camera PCLK              │ GPIO 13                          │
       │ Camera XCLK              │ GPIO 10                          │
       │ Camera Data D0 - D7      │ GPIO 11, 9, 8, 6, 4, 2, 3, 15    │
       │ Status LED (Onboard)     │ GPIO 21                          │
       │ ESP-NOW Serial Tx Bridge │ GPIO 43 (TXD0)                   │
       │ ESP-NOW Serial Rx Bridge │ GPIO 44 (RXD0)                   │
       └──────────────────────────┴──────────────────────────────────┘
```
