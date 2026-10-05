# Bài tập về nhà 3 — latency toàn pipeline ONNX trên CPU

- Weights: `yolo26n.pt`, Ultralytics 8.4.171, ONNX Runtime 1.30.0.
- Máy đo: Linux-6.6.122+-x86_64-with-glibc2.39; `device=cpu`, `imgsz=640`, ảnh `bus.jpg`.
- Export: one-to-many `nms=None` → [1, 84, 8400]; one-to-one `nms=False` → [1, 300, 6].
- One-to-many dùng NMS ngoài graph với `iou=0.7`; one-to-one chỉ lọc confidence theo graph đã export.
- Mỗi cấu hình warm-up 3 lần, lấy trung vị 30 lần. `wall p50` đo toàn bộ lệnh `predict`; ba cột còn lại là `Results.speed`. Export/nạp model/warm-up không tính vào thời gian.

| head | conf | preprocess p50 (ms) | inference p50 (ms) | postprocess p50 (ms) | wall p50 (ms) | số box |
| --- | --- | --- | --- | --- | --- | --- |
| one-to-many + NMS | 0.25 | 8.64 | 179.77 | 2.16 | 214.73 | 5 |
| one-to-many + NMS | 0.001 | 9.8 | 158.43 | 3.19 | 181.38 | 186 |
| one-to-one NMS-free | 0.25 | 8.98 | 192.49 | 0.64 | 211.92 | 5 |
| one-to-one NMS-free | 0.001 | 4.94 | 78.13 | 0.5 | 88.85 | 177 |

## Nhận xét từ lần đo

- `conf=0.25`: one-to-one nhanh hơn 2.81 ms theo `wall p50`; postprocess 2.16 → 0.64 ms.
- `conf=0.001`: one-to-one nhanh hơn 92.53 ms theo `wall p50`; postprocess 3.19 → 0.50 ms.

Các số liệu là latency trên CPU của lần chạy này; so sánh tổng pipeline theo `wall p50`, không suy ra từ riêng cột inference.
