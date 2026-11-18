# Báo cáo QC — <<3Team>>

Copy vào `submission/qc/QC_REPORT.md`. Giữ tên trường trong các dòng gạch đầu dòng. Thay các chỗ `<<...>>` và điền vòng QC của bản nhãn muốn bàn giao.


## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Gán và kiểm soát chất lượng bounding box 2D cho người thật và phương tiện thật nhìn thấy trong ảnh đường phố; nhãn hướng tới bài toán phát hiện vật thể 2D. Bộ dữ liệu gồm 40 ảnh nên không đại diện cho mọi điều kiện giao thông và không đủ để khẳng định mô hình an toàn khi triển khai thực tế. Ngoài phạm vi: ước lượng 3D, tracking, biển số, màu xe, hành vi và các vật thể không thuộc taxonomy.

- Nguồn và phép sử dụng dữ liệu: 40 ảnh trong `data/images/`, nguồn, tác giả, giấy phép và URL từng ảnh được ghi trong `data/ATTRIBUTION.csv`. Theo `data/README.md`, ảnh dùng cho mục đích học tập; khi chia sẻ lại cần giữ ghi công và tuân thủ điều kiện giấy phép, bao gồm chia sẻ tương tự với ảnh CC BY-SA.


- Guideline và taxonomy khi gán / khi bàn giao: Khi gán dùng `guideline_v1.0_agreed.md` và `taxonomy.json` phiên bản v1.0. Taxonomy gồm 11 lớp (kể cả `ignore_region`) và các thuộc tính `occluded_level`, `truncated`, `can_review`. 


## 2. Phân công và đồng thuận trước calibration

- Các thành viên tương ứng với mã : 
+, 301 : Lê Duy Bảo
+, 302: Trần Thành Hùng
+, 303 : Nguyễn Đức

- Lượt hoàn thành / được giao: Theo `iaa_summary.md`, hoàn thành 80/80 lượt trên 40 ảnh, với ba thành viên 301, 302, 303; mỗi ảnh được giao cho hai người theo `assignment.csv`. Tuy nhiên do ban đầu 303 gán nhãn chưa được chính xác lên 302 có phụ sửa lại , lần chạy `python scripts/validate_labels.py --all` gần nhất cho thấy 301, 302 và 303 hợp lệ .

- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Tổng 182 cặp box ghép; IoU trung bình 0.856; đồng thuận lớp 95.6%; mẫu số là 182 cặp. Theo ngưỡng IAA IoU >= 0.85 và đồng thuận lớp >= 95%, kết quả tổng thể đạt. Theo cặp: 301|302 — 81 cặp, IoU 0.881, cùng lớp 95.1% (đạt); 301|303 — 53 cặp, IoU 0.824, cùng lớp 96.2% (chưa đạt do IoU); 302|303 — 48 cặp, IoU 0.851, cùng lớp 95.8% (đạt).

- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Tổng box chưa ghép là 247/29 (A/B theo thứ tự cặp trong báo cáo). Theo từng cặp: 301|302 là 60/13; 301|303 là 148/13; 302|303 là 39/3. Có 0 box được bỏ qua do nằm trong vùng ignore; báo cáo ghi nhận 1 ca bất đồng ignore cần kiểm tra riêng.

- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa có ca nào được xác nhận bằng đối chiếu ảnh gốc trong các kết quả hiện có; IAA chỉ cho biết mức độ đồng thuận, không chứng minh nhãn đồng thuận là đúng. Ưu tiên review `img_003.jpg`, `img_006.jpg` và `img_032.jpg` vì có điểm cần calibration cao trong `image_scores.csv`; đối chiếu ảnh gốc với điều khoản guideline liên quan, ghi lại vùng và quyết định trước khi kết luận.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Có 429 ca trong `calibration_candidates.csv` trên 36 ảnh. Chưa ghi nhận đủ quyết định cuối: `quyet_dinh` trong `calibration_decisions.csv` còn trống. Cập nhật sau khi adjudicate và kiểm tra từng ca.
- Tác động v1.1 lên nhãn và ảnh: `guideline_change.json` khai báo `semantic`; theo quy trình, cần kiểm lại cả 40 ảnh. Chưa có bằng chứng nhãn calibrated và manifest được tạo trong workspace hiện tại.

## 4. Kết quả duyệt

- Vòng QC: Chưa có hồ sơ review vòng QC trong workspace hiện tại. <<cập nhật số vòng sau khi chạy `qc_sample.py`>>
- Người review và ngày: <<mã người thực sự review; ngày review>>
- Đã duyệt / cả lô: Chưa có kết quả. <<số ảnh đã duyệt / 40; xác nhận duyệt toàn lô hay không>>
- Ảnh lỗi / ảnh đã duyệt: Chưa đo. <<số ảnh có lỗi / số ảnh đã duyệt; tỷ lệ>>
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: Chưa đo. <<số lỗi / số annotation đã review>>
- Vật bỏ sót, ảnh và vùng: Chưa có kết quả review. <<số lượng, ảnh, vùng cụ thể hoặc “0” nếu đã kiểm chứng>>
- Ca khó / phần còn lại, lỗi lặp và ví dụ: <<dẫn `qc_review.csv`; nêu lỗi lặp theo rule ID và case/ảnh minh họa>>
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: <<nêu một ví dụ có bằng chứng cho mỗi loại>>

## 5. Quyết định bàn giao

 Vòng QC: <<số vòng QC cuối>>
- Người review và ngày: <<mã người thực sự review; ngày review>>
- Đã duyệt / cả lô: <<số ảnh đã duyệt / 40; full hay sample>>
- Quyết định: Hold — trạng thái hiện tại do chưa có bằng chứng hoàn tất calibration, nhãn cuối và QC toàn lô. Thay bằng kết quả thực tế sau khi hoàn tất Pha 6.
- Lý do, người nhận và phạm vi sử dụng: <<lý do theo ngưỡng QC; người nhận; mục đích và giới hạn sử dụng>>
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: <<một case ID, đường dẫn ảnh, mã R-xx, vùng/box trong nhãn cuối và kết quả đối chiếu>>
