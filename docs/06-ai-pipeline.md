# 06 - AI Pipeline Specification

> **Document Status:** `[DECISION]` Vision AI Model & Execution Blueprint  
> **Framework:** TensorFlow Lite for Microcontrollers (TFLite Micro) + ESP-NN  

---

## 1. Domain Taxonomy & Formal Terminology

To prevent architectural ambiguity, the AI pipeline strictly distinguishes between 7 operational terms:

```text
  1. Model ──> 2. Inference ──> 3. Detection ──> 4. Observation ──> 5. Event ──> 6. Decision ──> 7. Action
```

| Term | Technical Definition | Example |
| :--- | :--- | :--- |
| **Model** | Static binary graph containing trained weights. | `person_detect_v2_int8.tflite` (2.1 MB) |
| **Inference** | Execution of forward pass over 1 image matrix. | Compute output tensor in 140ms on Core 1 |
| **Detection** | Raw probability score from output tensor. | `Score: 0.88` for class `Person` in Frame #104 |
| **Observation**| Filtered detection meeting single-frame threshold. | Single frame positive observation ($Score \ge 0.75$) |
| **Event** | Temporal state change validated over time window. | State transitioned from `CLEAR` to `PERSON_PRESENT` |
| **Decision** | Risk Engine calculation based on active events. | Threat Level escalated to `HIGH` |
| **Action** | Actuation command emitted by Safety Policy. | `SOUND_ALARM_SIREN` sent via ESP-NOW |

---

## 2. Model Selection & Quantization Strategy

### Selected Architecture
* **Primary Model:** Quantized MobileNet-V2 (Alpha 0.35, $96 \times 96$ input grayscale or RGB). `[DECISION]`
* **Model Purpose:** Binary Human Presence Detection (Person vs. Background) and Fire/Smoke color-blob pattern validation.
* **Quantization Method:** Full INT8 Quantization (post-training quantization using representative calibration dataset).

### Resource Benchmark Targets

| Metric | Target Value | Architectural Status |
| :--- | :--- | :--- |
| **Model Binary Size** | 2.1 MB (Stored in Flash memory) | `[DECISION]` |
| **Tensor Arena Size** | 150 KB (Allocated in internal SRAM) | `[DECISION]` |
| **Inference Latency** | 120 ms – 160 ms per frame | `[VERIFY]` |
| **Target Frame Rate** | 5 FPS | `[DECISION]` |
| **True Positive Rate (TPR)**| $\ge 92\%$ at 3-meter distance | `[VERIFY]` |
| **False Positive Rate (FPR)**| $\le 2\%$ post-temporal filter | `[DECISION]` |

---

## 3. Preprocessing & Tensor Allocation Flow

```text
Input Camera Frame (QVGA 320x240 RGB565)
          │
          ▼  [Bilinear Downsampling]
Resized Matrix (96x96 Grayscale or RGB)
          │
          ▼  [Int8 Normalization: (Pixel - 128)]
Quantized Input Tensor Array
          │
          ▼  [ESP-NN Accelerated Convolution Ops]
TFLite Micro Execution Engine
          │
          ▼  [Output Softmax Extraction]
Detection Probability Vector: [p_background, p_person]
```

---

## 4. Model Evaluation & Failure Recovery

1. **False Positive Handling:** Single-frame false positives caused by sudden light changes are ignored by requiring $N=3$ consecutive frame matches in the downstream Temporal Engine (see [`07-event-generation.md`](file:///d:/SAM/docs/07-event-generation.md)).
2. **False Negative Handling:** Once an active `PERSON_PRESENT` state is established, the state engine uses a **cooldown hysteresis timer** (e.g., 5 seconds of consecutive zero detections required to revert to `CLEAR`), ensuring brief occlusion does not drop the alarm state.
