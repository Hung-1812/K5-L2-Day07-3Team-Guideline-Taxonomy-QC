# Thông tin nhóm

- Tên nhóm:3Team
- Mã thành viên (ba chữ số):301, 302, 303
- Người điều phối:302
- Người review bản cuối:302
- Commit chốt bài (điền ở kênh nộp sau khi commit):



## Kết quả tóm tắt

- Lượt gán hoàn tất / được giao: 80/80 lượt trên 40 ảnh; mỗi ảnh có 2 người gán.
- IAA trước calibration: 182 cặp box ghép; IoU trung bình 0.856; đồng thuận lớp 95.6%. Kết quả tổng thể đạt ngưỡng IAA đã đặt ra (IoU >= 0.85, đồng thuận lớp >= 95%). Cặp 301|303 chưa đạt ngưỡng IoU (0.824).
- Số ảnh cần calibration: 36/40; có 429 ca được tự động đánh dấu trong `calibration_candidates.csv`.
- Calibration / Pha 5: Đã có guideline v1.1 và `guideline_change.json`. Chưa xác nhận hoàn tất adjudication: `quyet_dinh` trong `calibration_decisions.csv` còn trống; một số nội dung sửa vẫn là nháp. Chưa thấy `labels_calibrated.*` và `batch_manifest.json` trong workspace hiện tại.
- Vòng QC và số ảnh đã duyệt / cả lô: Chưa có bằng chứng vòng QC toàn lô; cập nhật sau khi hoàn tất Pha 6.
- Tỉ lệ lỗi theo ảnh / theo đối tượng: Chưa đo. Không ghi 0% trước khi có phiếu review và kết quả `qc_report.py`.
- Quyết định bàn giao hiện tại: **Hold tạm thời** — cần hoàn tất chốt calibration, tạo nhãn cuối và duyệt QC toàn bộ 40 ảnh.