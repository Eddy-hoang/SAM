# ADR-0001: Decoupled Edge-First Mesh System Architecture

## Context
Traditional home security systems stream raw sensor data to cloud servers for processing. This creates severe vulnerabilities during Internet outages, introduces 2–5 second latencies, and poses major user privacy risks.

## Problem
How should SafeHome AI Mesh structure its processing hierarchy to guarantee sub-second emergency response and 100% offline operational resilience?

## Considered Options
1. **Cloud-Centric Architecture:** ESP32 streams video to AWS/GCP cloud backend for AI and risk evaluation.
2. **Gateway-Centric Architecture:** Raw video streamed over local Wi-Fi to a local Gateway PC for processing.
3. **Edge-First Mesh Architecture:** ESP32-S3 performs local vision AI, emits debounced event metadata, uses ESP-NOW for emergency sirens, and sends telemetry to Gateway.

## Decision
We select **Option 3: Edge-First Mesh Architecture**. Edge vision inference occurs directly on ESP32-S3. Emergency alerting uses direct ESP-NOW peer-to-peer transmission, completely independent of Wi-Fi routers and internet access.

## Why
* **Latency:** ESP-NOW achieves $<15\text{ ms}$ direct transmission to local sirens.
* **Resilience:** System functions continuously during WAN/Internet outages.
* **Privacy:** Zero video streaming over local network or external servers.

## Trade-offs
* Higher embedded software complexity on ESP32-S3 (TFLite Micro, memory management).
* Microcontroller hardware constraints limit computer vision model complexity to lightweight models (MobileNet-V2).

## Consequences
All safety-critical features must be designed to run offline on local embedded hardware.

## Status
`[DECISION]` Accepted & Approved.
