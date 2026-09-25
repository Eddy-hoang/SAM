# ADR-0003: Thực thi Mô hình Lượng hóa INT8 trên Thiết bị (On-Device AI)

## Bối cảnh (Context)
Triển khai vision AI lên vi điều khiển đòi hỏi cân bằng giữa độ chính xác suy luận với giới hạn bộ nhớ SRAM/PSRAM khắt khe và ngưỡng tản nhiệt CPU.

## Vấn đề (Problem)
Chiến lược lượng hóa mô hình và runtime thực thi nào nên được áp dụng trên chip ESP32-S3?

## Các Phương án Đánh giá (Considered Options)
1. **Mô hình Dấu phẩy động FP32 (FP32 Floating Point):** Thực thi mô hình dạng dấu phẩy động 32-bit gốc.
2. **Mô hình Lượng hóa INT8 với TFLite Micro:** Post-training INT8 quantization thực thi qua TensorFlow Lite for Microcontrollers kết hợp tăng tốc tập lệnh SIMD vector ESP-NN.

## Quyết định (Decision)
Lựa chọn **Phương án 2: Mô hình Lượng hóa INT8 với TFLite Micro**.

## Lý do (Why)
* Lượng hóa INT8 giảm kích thước binary mô hình đi $75\%$ (từ 8.4MB xuống 2.1MB), vừa vặn vào bộ nhớ Flash.
* Tập lệnh SIMD vector ESP-NN giảm độ trễ suy luận từ $>500\text{ ms}$ xuống $<150\text{ ms}$.

## Trạng thái (Status)
`[DECISION]` Đã phê duyệt & Thông qua.
