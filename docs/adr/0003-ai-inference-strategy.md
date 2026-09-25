# ADR-0003: On-Device INT8 Quantized Model Execution

## Context
Deploying vision AI to microcontrollers requires balancing inference accuracy against strict SRAM/PSRAM boundaries and CPU thermal limits.

## Problem
Which model quantization and execution runtime strategy should be adopted on the ESP32-S3?

## Considered Options
1. **FP32 Floating Point Model:** Native 32-bit floating point model execution.
2. **INT8 Quantized Model with TFLite Micro:** Post-training INT8 quantization executed via TensorFlow Lite for Microcontrollers with ESP-NN SIMD vector acceleration.

## Decision
We select **Option 2: INT8 Quantized TFLite Micro**.

## Why
* INT8 quantization reduces binary model size by $75\%$ (from 8.4MB to 2.1MB), fitting into Flash memory.
* ESP-NN SIMD vector instructions reduce inference latency from $>500\text{ ms}$ to $<150\text{ ms}$.

## Status
`[DECISION]` Accepted & Approved.
