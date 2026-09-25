# 06 - Đặc tả AI Pipeline (AI Pipeline Specification)

> **Trạng thái Tài liệu:** `[DECISION]` Blueprint Mô hình Vision AI & Thực thi  
> **Framework:** TensorFlow Lite for Microcontrollers (TFLite Micro) + ESP-NN  

---

## 1. Thuật ngữ & Phân định Khái niệm Kỹ thuật

Để tránh mơ hồ trong kiến trúc, pipeline AI phân định ranh giới nghiêm ngặt giữa 7 thuật ngữ vận hành:

```text
  1. Model ──> 2. Inference ──> 3. Detection ──> 4. Observation ──> 5. Event ──> 6. Decision ──> 7. Action
```

| Thuật ngữ | Định nghĩa Kỹ thuật | Ví dụ Minh họa |
| :--- | :--- | :--- |
| **Model** | Đồ thị file binary chứa các trọng số đã huấn luyện. | `person_detect_v2_int8.tflite` (2.1 MB) |
| **Inference** | Việc thực thi một lượt tính toán forward pass trên 1 ma trận ảnh. | Tính toán tensor đầu ra trong 140ms trên Core 1 |
| **Detection** | Giá trị xác suất thô trích xuất từ tensor đầu ra. | `Score: 0.88` cho lớp `Person` tại Frame #104 |
| **Observation**| Nhận diện đã lọc đạt ngưỡng tin cậy trên 1 frame. | Quan sát dương tính trên 1 frame ($Score \ge 0.75$) |
| **Event** | Sự thay đổi trạng thái theo thời gian được xác minh qua cửa sổ mẫu.| Trạng thái chuyển từ `CLEAR` sang `PERSON_PRESENT` |
| **Decision** | Quyết định tính toán từ Risk Engine dựa trên các sự kiện active. | Cấp độ Đe dọa (Threat Level) tăng lên `HIGH` |
| **Action** | Lệnh kích hoạt phát ra từ tầng Safety Policy. | Lệnh `SOUND_ALARM_SIREN` gửi qua ESP-NOW |

---

## 2. Lựa chọn Mô hình & Chiến lược Lượng hóa (Quantization)

### Kiến trúc Lựa chọn
* **Mô hình Chính (Primary Model):** Quantized MobileNet-V2 (Alpha 0.35, đầu vào $96 \times 96$ grayscale hoặc RGB). `[DECISION]`
* **Mục đích Mô hình:** Phát hiện sự xuất hiện của con người (Person vs. Background) và xác minh mẫu màu khói/cháy.
* **Phương pháp Lượng hóa:** Full INT8 Quantization (post-training quantization sử dụng tập dữ liệu hiệu chuẩn đại diện).

### Chỉ số Benchmark Mục tiêu

| Chỉ số (Metric) | Giá trị Mục tiêu | Trạng thái Kiến trúc |
| :--- | :--- | :--- |
| **Kích thước Binary Model** | 2.1 MB (Lưu trong Flash memory) | `[DECISION]` |
| **Kích thước Tensor Arena** | 150 KB (Cấp phát trong internal SRAM) | `[DECISION]` |
| **Độ trễ Suy luận (Inference Latency)**| 120 ms – 160 ms mỗi frame | `[VERIFY]` |
| **Tốc độ Khung hình (Target FPS)**| 5 FPS | `[DECISION]` |
| **Tỷ lệ Dương tính Đúng (TPR)**| $\ge 92\%$ ở khoảng cách 3 mét | `[VERIFY]` |
| **Tỷ lệ Dương tính Giả (FPR)**| $\le 2\%$ sau bộ lọc temporal filter | `[DECISION]` |

---

## 3. Tiền xử lý & Quy trình Cấp phát Tensor

```text
Khung hình Camera Đầu vào (QVGA 320x240 RGB565)
          │
          ▼  [Bilinear Downsampling]
Ma trận Ảnh Resize (96x96 Grayscale hoặc RGB)
          │
          ▼  [Int8 Normalization: (Pixel - 128)]
Mảng Input Tensor Quantized
          │
          ▼  [Tập lệnh Tăng tốc Convolutions ESP-NN]
Engine Thực thi TFLite Micro
          │
          ▼  [Trích xuất Output Softmax]
Vector Xác suất Đầu ra: [p_background, p_person]
```

---

## 4. Đánh giá Mô hình & Xử lý Ngoại lệ

1. **Xử lý Báo động giả (False Positive Handling):** Các nhận diện sai 1 frame do thay đổi ánh sáng đột ngột sẽ bị loại bỏ hoàn toàn bằng việc yêu cầu $N=3$ frame khớp liên tiếp ở Temporal Engine phía sau (xem [`07-event-generation.md`](file:///d:/SAM/docs/07-event-generation.md)).
2. **Xử lý Âm tính giả (False Negative Handling):** Khi trạng thái `PERSON_PRESENT` đã được thiết lập, state engine áp dụng **bộ đếm thời gian trễ Cooldown Hysteresis** (ví dụ: cần 5 giây liên tiếp không nhận diện được mới chuyển về `CLEAR`), đảm bảo hiện tượng che khuất thoáng qua không làm rơi trạng thái báo động.
