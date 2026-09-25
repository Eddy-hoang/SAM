# ADR-0002: Dual-Path Hybrid Network Strategy (ESP-NOW + Wi-Fi)

## Context
Emergency alarms require microsecond-level latency and zero router dependency, whereas administrative monitoring dashboards require rich state telemetry streaming.

## Problem
What network protocols should be selected for device-to-device emergency signaling versus device-to-gateway telemetry reporting?

## Considered Options
1. **Wi-Fi Only (MQTT/HTTP):** All nodes communicate exclusively over standard 802.11 Wi-Fi.
2. **ESP-NOW Only:** All communications (including telemetry and images) use ESP-NOW.
3. **Dual-Path Hybrid Architecture:** ESP-NOW for emergency alarm channels; Wi-Fi (MQTT + WebSockets) for telemetry and UI streaming.

## Decision
We select **Option 3: Dual-Path Hybrid Architecture**.

## Why
* ESP-NOW bypasses Wi-Fi AP association and DHCP overhead, delivering sub-15ms emergency frames.
* MQTT over local Wi-Fi handles structured telemetry JSON objects efficiently.
* If Wi-Fi AP crashes, ESP-NOW emergency channels remain 100% operational.

## Trade-offs
* All ESP32 Wi-Fi radios must be locked to a fixed 2.4GHz Wi-Fi channel (Channel 6).

## Status
`[DECISION]` Accepted & Approved.
