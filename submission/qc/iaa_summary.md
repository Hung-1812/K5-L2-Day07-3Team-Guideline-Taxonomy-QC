# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 182 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 182 | 0.856 | 95.6% | 68.7% | 92.3% | 247/29 | 0 | ĐẠT |
| 301|302 | 13 | 81 | 0.881 | 95.1% | 55.6% | 96.3% | 60/13 | 0 | ĐẠT |
| 301|303 | 13 | 53 | 0.824 | 96.2% | 71.7% | 90.6% | 148/13 | 0 | CHƯA ĐẠT |
| 302|303 | 14 | 48 | 0.851 | 95.8% | 87.5% | 87.5% | 39/3 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 429 ({'bo_sot_thua': 276, 'iou_thap': 66, 'sai_thuoc_tinh': 66, 'sai_lop': 8, 'co_can_review': 12, 'bat_dong_ignore': 1}), trên 36 ảnh cần calibration.
