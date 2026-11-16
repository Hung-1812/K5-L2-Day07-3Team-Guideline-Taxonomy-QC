# Guideline gán nhãn — v1.0 — K5-L2-Day07

- **Phiên bản:** v1.0 – bản hoàn thiện đề xuất để nhóm review và khóa trước khi gán độc lập
- **Ngày:** 16/11/2026
- **Người viết / duyệt:** Người viết: Nguyễn Ngọc Đức / Trần Thành Hùng

## 1. Mục tiêu

Tạo bộ nhãn Bounding Box nhất quán để huấn luyện và đánh giá mô hình phát hiện **người thật và phương tiện giao thông thực** trong ảnh đường phố, bến xe, khu vực giao thông và hoạt động vận chuyển.

Mỗi đối tượng thuộc phạm vi, đủ khả năng nhận diện phải có **một box cho một instance** và **một class duy nhất**. 

## 2. Lớp và thuộc tính
### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Một cơ thể người thật nhìn thấy trực tiếp trong cảnh, bất kể đang đi bộ, lái xe, ngồi trên xe hay làm việc. | Người đi bộ, người lái/khách xe máy, người bán hàng, người đẩy xe. | Tượng, ma-nơ-canh, hình in trên poster, hình phản chiếu; mascot/nhân vật hóa trang không thấy được cơ thể người rõ ràng. |
| `car` | Ô tô chở người dạng xe con, không có cấu trúc chính là xe tải hoặc xe buýt. | Sedan, hatchback, wagon, coupe, SUV, taxi; ô tô thật đang đỗ/trưng bày. | Pickup gán `truck`; van gán `van`; bus gán `bus`; xe mô hình/đồ chơi không gán. |
| `truck` | Phương tiện cơ giới có cấu trúc/chức năng chính để chở hàng hoặc kéo/chở phương tiện khác. | Xe tải thùng, pickup, xe giao hàng dạng tải, xe trộn bê tông, xe chở ô tô/car carrier. | Ô tô con gán `car`; xe kín cỡ nhỏ dạng van gán `van`; xe buýt gán `bus`. |
| `van` | Xe cơ giới thân kín cỡ nhỏ/trung bình, khoang người hoặc hàng liền thân, không có thùng pickup hở. | Delivery van, minivan, xe tải van nhỏ. | Pickup có thùng hở gán `truck`; xe bus có cấu trúc chở khách gán `bus`. |
| `bus` | Phương tiện cơ giới thân lớn được thiết kế chủ yếu để chở nhiều hành khách. | City bus, coach, xe buýt công cộng. | Van/minivan; truck. |
| `motorcycle` | Phương tiện hai bánh có động cơ, gồm motorbike và scooter. | Xe máy, mô tô, scooter; xe máy gắn sidecar (một instance tổ hợp). | Xe đạp `bicycle`; xe ba bánh có động cơ `auto_rickshaw`; Xích lô đạp `cycle_rickshaw`. |
| `bicycle` | Phương tiện hai bánh không có động cơ, vận hành bằng sức người. | Xe đạp thường. | Motorcycle; Xích lô ba bánh; xe đẩy hàng. |
| `auto_rickshaw` | Phương tiện ba bánh có động cơ dùng chở người hoặc hàng. | Tuk-tuk, auto-rickshaw, xe ba bánh máy. | Xích lô/xích lô đạp `cycle_rickshaw`; motorcycle hai bánh. |
| `cycle_rickshaw` | Phương tiện ba bánh không có động cơ, được đạp để chở khách hoặc hàng. | Xích lô, xích lô, pedicab. | Tuk-tuk/auto-rickshaw có động cơ; bicycle hai bánh. |
| `handcart` | Xe đẩy/kéo không động cơ có bánh, dùng để vận chuyển hàng hóa hoặc bán hàng trên đường. | Xe đẩy trái cây, xe hàng đẩy tay, hand trolley lớn. | Người bán hàng không có xe đẩy; hàng hóa riêng; xe đẩy siêu thị không tham gia cảnh giao thông. |
| `ignore_region` | Vùng chữ nhật không phải instance, dùng cho khu vực có nhiều vật quá nhỏ/dày/mờ đến mức không thể tách và nhận diện đáng tin cậy. | Cụm xe/người rất xa hoặc vùng đám đông không thể tách từng người. | Không dùng để thay thế box của đối tượng rõ; không gán ignore chỉ vì annotator chưa muốn vẽ. |

### Quy tắc phân lớp

1. Người thật → `person`, độc lập với box phương tiện.
2. Phương tiện 2 bánh: có động cơ → `motorcycle`; không động cơ → `bicycle`.
3. Phương tiện 3 bánh: có động cơ → `auto_rickshaw`; đạp bằng sức người → `cycle_rickshaw`.
4. Xe đẩy tay có bánh để chở hàng → `handcart`.
5. Xe chở khách lớn → `bus`; xe kín cỡ nhỏ → `van`; ô tô con → `car`; xe chở hàng/pickup/car carrier → `truck`.
6. Nếu không đủ bằng chứng phân lớp: không đoán. Đặt `can_review=true` trên box nếu đã có box hợp lý; nếu không thể tạo box đáng tin cậy, ghi `image_issues.csv` và chỉ dùng `ignore_region` khi đúng định nghĩa vùng bất khả phân.

### Thuộc tính

Áp dụng cho mọi instance class ngoại trừ `ignore_region`. Tên và giá trị phải khớp taxonomy/CVAT.

| Thuộc tính | Mức cố định | Định nghĩa / ngưỡng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng tỷ lệ phần thân dự kiến bị **vật khác trong cảnh** che khuất: `0` = 0–10% (không/không đáng kể); `1` = >10–30% (nhẹ); `2` = >30–60% (vừa/nhiều); `3` = >60% nhưng vẫn còn đủ dấu hiệu để nhận diện class và instance. Nếu không còn đủ dấu hiệu, không ép mức 3; chuyển reviewer/ignore theo quy tắc. |
| `truncated` | `no`, `yes` | `yes` chỉ khi biên ảnh cắt mất một phần vật thể; `no` nếu vật nằm trọn trong ảnh. Bị vật khác che không phải truncation. |
| `can_review` | `false`, `true` | `true` khi còn nghi vấn về class, số instance, ranh giới box, mức che hoặc ca đặc biệt; phải ghi lý do/vùng trong nhật ký. `false` khi annotator tự tin theo rule. |

`occluded_level` là thuộc tính của nhóm; không nhầm với cờ `occluded` mặc định của CVAT. Nếu cấu hình CVAT có cờ `occluded`, bật cờ khi có phần vật thể bị vật khác che; cờ này không thay thế `occluded_level`. `ignore_region` không cần ba thuộc tính trên.

## 3. Quy tắc vẽ box

### R-01 — Một box cho một instance

- Dùng **Rectangle**. Box là hình chữ nhật nhỏ nhất bao quanh toàn bộ phần nhìn thấy của đúng một vật thể.
- Bao gồm các phần thuộc về vật thể: thân, bánh, gương, cản, tay lái, tay cầm xe đẩy và hàng hóa gắn/chở trên phương tiện nếu chúng tạo thành phần ngoài cùng của tổ hợp.
- Không lấy bóng đổ, khoảng trống, mặt đường, người/xe bên cạnh hoặc vật không thuộc instance.
- Các box được phép chồng lấn khi các vật thể thật chồng lấn trong ảnh. Không gộp các vật chỉ vì chúng sát nhau.

### R-02 — Chỉ vẽ phần nhìn thấy

- Vật bị che: vẽ box theo phần vật thể còn nhìn thấy; không tự suy diễn phần khuất hoặc vẽ box xuyên qua vật che.
- Nếu vẫn nhận diện được vật và xác định được vùng nhìn thấy, vẫn gán box và đặt `occluded_level` theo tỷ lệ ước lượng.
- Nếu không thể xác định class/instance hoặc không thể tách box đáng tin cậy, không đoán; đặt `can_review=true` nếu có box tạm hợp lý, nếu không ghi issue cấp ảnh.
- Không đặt `occluded_level=3` chỉ để biểu thị “không rõ”.

### R-03 — Cắt mép ảnh

- Vẫn gán nếu phần nhìn thấy đủ để nhận diện.
- Box dừng đúng tại biên ảnh; không kéo tọa độ ra ngoài canvas và không ước lượng phần nằm ngoài.
- Đặt `truncated=yes`. Không bật occluded chỉ vì vật bị biên ảnh cắt.

### R-04 — Người và phương tiện

- Mỗi người nhìn thấy rõ có một box `person`; mỗi phương tiện có một box class tương ứng.
- Người lái/ngồi trên xe máy: vẽ `person` riêng và `motorcycle` riêng. Không đưa đầu, thân, tay/chân người vào box xe; không đưa thân xe vào box người.
- Người đẩy/kéo handcart: `person` và `handcart` là hai box độc lập.
- Người ngồi trong ô tô: chỉ gán `person` khi cơ thể người nhìn thấy trực tiếp đủ để vẽ box đáng tin cậy; không suy đoán người qua kính tối.
- Nhiều người trên một xe: một box phương tiện và một box `person` cho từng người phân biệt được.

### R-05 — Tổ hợp và hàng hóa

- Xe máy chở hàng: một box `motorcycle`; hàng hóa không tạo class riêng.
- Sidecar gắn liền xe máy: một box `motorcycle` bao cả cụm sidecar và xe.
- Car carrier: box `truck` cho xe chở; box `car` riêng cho từng xe được chở nếu nhìn thấy đủ. Không box cho giá đỡ.
- Pickup chở thùng/túi/trái cây: một box `truck`; hàng hóa không gán riêng.
- Rơ-moóc gắn với đầu kéo: trong phiên bản này coi toàn tổ hợp kéo là một `truck` nếu là một đơn vị vận tải liên kết; không tạo class rơ-moóc.

### R-06 — Người/xe giả, ảnh in và phản chiếu

- Không gán người/xe xuất hiện chỉ trong poster, billboard, tranh, màn hình, gương hoặc bề mặt phản chiếu.
- Không gán tượng, tượng sáp, ma-nơ-canh, hình nộm thành `person`; không gán xe mô hình/đồ chơi thành phương tiện.
- Người thật mặc trang phục hóa trang vẫn là `person` nếu có đủ dấu hiệu cơ thể người/performer trực tiếp. Nếu chỉ thấy hình dáng mascot không thể xác nhận cơ thể người, đặt `can_review=true` hoặc bỏ qua theo quyết định reviewer; không mặc định mọi nhân vật hóa trang là người.
- Một chiếc ô tô thật được tạo hình trang trí/nhân vật nhưng có thân xe, bánh và đang được vận hành trong đoàn diễu hành vẫn là `car`; không nhầm với mô hình đồ chơi tĩnh.

### R-07 — Vật nhỏ, mờ và nhòe

- Theo ngưỡng lab, box có chiều cao **dưới 20 px** không tạo instance thường; đưa vào `ignore_region` nếu nó thuộc vùng nhỏ/dày không thể gán riêng. Không dùng ignore để phủ vật lớn, rõ.
- Box cao từ 20 px trở lên: gán nếu class và biên nhìn thấy đủ rõ.
- Motion blur không tự động làm vật thành ignore. Nếu vẫn nhận diện được class và biên, vẽ box; nếu chỉ là vệt nhòe không xác định được instance, không đoán và ghi issue/ignore vùng không thể phân giải.
- Với xe/người rất xa, không gộp nhiều instance thành một box class thông thường.

### R-08 — Đám đông

- Tách từng `person` nếu phân biệt được cơ thể và box hợp lý.
- Không gộp đám đông thành một `person`.
- Chỉ dùng `ignore_region` cho phần lõi quá dày không thể tách; vẫn phải gán những người rõ ở rìa vùng đó.
- Không dùng một ignore_region lớn che toàn bộ ảnh chỉ vì ảnh đông.

### R-09 — Xe chủ (ego) và vật trong gương

- Dataset là ảnh tĩnh, không có nhãn riêng cho ego vehicle. Nếu một phần xe chủ nhìn thấy **trực tiếp** trong ảnh và đủ để nhận diện là phương tiện thuộc taxonomy, gán theo class tương ứng và đặt `truncated=yes` nếu bị mép ảnh cắt.
- Vật chỉ xuất hiện trong gương/kính là hình phản chiếu, không gán. Không gán cầu vồng/cảnh vật phản chiếu trong gương.

### R-10 — Vật ngoài taxonomy

Không gán nhà cửa, đường, vỉa hè, cây, cột/dây điện, biển báo, đèn giao thông, billboard, vạch kẻ đường, hàng hóa riêng lẻ, chim/động vật hoặc vật thể không thuộc taxonomy. Không map một vật ngoài taxonomy vào class gần giống nhất.

## 4. Ca khó

Bảng này được lập từ việc xem ảnh gốc `img_001`–`img_040` trong file người dùng tải lên và đối chiếu mô tả nguồn/attribution của repository. Đây là **quyết định hướng dẫn đề xuất**, không phải nhãn ground truth hay kết quả adjudication. Annotator phải mở ảnh gốc trong CVAT để xác định chính xác từng instance và ghi `can_review`/issue khi không đủ bằng chứng.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_001` | Pickup cạnh người tại quán ven đường | R-01, R-04 | Pickup `truck`; người thật `person`; box tách riêng. | Pickup là xe chở hàng; người không nhập vào box xe. |
| `img_002` | Xe chở nhiều ô tô (car carrier) | R-05 | Xe chuyên chở `truck`; từng ô tô được chở là `car` nếu nhận diện được. | Xe vận chuyển và các xe trên sàn là các instance vật lý riêng. |
| `img_003` | Người bán hàng và xe/giỏ hàng chở trái cây | R-04, R-05 | Người là `person`; xe đẩy có bánh là `handcart` nếu cấu trúc xe đẩy nhận diện được; trái cây không gán. | Tách người với phương tiện vận chuyển; không gán hàng hóa. |
| `img_005` | Nhiều người đi xe máy, người/xe chồng lấn | R-02, R-04 | Từng `person` và `motorcycle` riêng nếu phân biệt được; dùng occlusion khi bị che thật. | Không gộp người lái vào xe hoặc ngược lại. |
| `img_006` | Giao thông có Xích lô, xe máy và người | R-04, R-05 | Xích lô `cycle_rickshaw`; xe máy `motorcycle`; người nhìn thấy `person`. | Xích lô ba bánh đạp khác tuk-tuk có động cơ. |
| `img_008` | Chợ đêm, nhiều xe máy đỗ/di chuyển và người | R-07, R-08 | Tách từng `motorcycle` và `person` đủ rõ; vùng quá dày/nhỏ dùng `ignore_region`. | Tránh gộp xe hoặc tạo box đoán trong cảnh tối. |
| `img_009` | Xích lô và người trong phố | R-04, R-05 | Xích lô `cycle_rickshaw`; người lái/khách là `person` riêng khi nhìn thấy. | Người và phương tiện là hai instance. |
| `img_010` | Người bán hàng rong, người đi xe máy và hàng hóa | R-04, R-05 | Người thật `person`; xe máy `motorcycle`; xe đẩy có bánh `handcart` nếu thấy rõ; hàng hóa không gán. | Chỉ gán các class thuộc taxonomy. |
| `img_011` | Nhân vật Predator hóa trang và các xe máy trang trí | R-06 | Xe máy thật `motorcycle`; người xung quanh `person`. Nhân vật hóa trang chỉ gán `person` nếu reviewer xác nhận performer/cơ thể người đủ rõ; nếu không, `can_review`/bỏ qua. | Không đồng nhất mascot với người khi không thấy được cơ thể người. |
| `img_013` | Phố đêm, cây, cửa hàng và người/xe ở xa | R-07, R-10 | Chỉ gán người/phương tiện đủ rõ; bỏ qua cây, nhà, đèn và cảnh nền; vật xa dưới 20 px xử lý theo ignore. | Tránh nhãn thiếu tin cậy trong cảnh thiếu sáng. |
| `img_014` | Người đi bộ qua đường, có motion blur | R-02, R-07 | Gán `person` nếu còn nhận diện được cơ thể; không vẽ box theo vệt nhòe không xác định. | Blur không phải occlusion; không suy đoán ranh giới. |
| `img_015` | Ô tô trong bãi/đường ướt, bị cắt ở hai mép ảnh | R-03, R-09 | Mỗi ô tô thật là `car`; xe bị khung ảnh cắt đặt `truncated=yes`. | Xe thật vẫn thuộc taxonomy dù chỉ thấy một phần. |
| `img_016` | Phố Bangkok với ô tô, xe máy và người | R-01, R-04 | Gán từng xe `car`/`motorcycle` và người `person`; không gán bảng hiệu/cửa hàng. | Nhiều loại phương tiện cùng xuất hiện. |
| `img_018` | Billboard có hình người và chữ | R-06 | Không gán hình người/xe in trên billboard. | Đối tượng trong ảnh in không phải instance thật trong cảnh. |
| `img_019` | Xe diễu hành tạo hình nhân vật hoạt hình, có người mặc mascot/ngồi trên xe | R-04, R-06 | Xe có thân xe, bánh và đang được vận hành: `car`; người lái nếu cơ thể nhìn thấy: `person`; mascot chỉ gán `person` khi xác nhận có performer/cơ thể người rõ. | Phân biệt phương tiện thật được trang trí với xe mô hình; mascot không tự động là người. |
| `img_021` | Cảnh giao thông đông, nhiều xe máy/người | R-07, R-08 | Tách từng xe/người nhận diện được; vùng đám đông không thể tách dùng `ignore_region`. | Không gộp cả nhóm người hoặc cụm xe. |
| `img_022` | Motorcycle gắn sidecar | R-05 | Một box `motorcycle` bao toàn bộ tổ hợp xe + sidecar; người là box `person` riêng. | Sidecar là bộ phận gắn liền của tổ hợp xe máy. |
| `img_023` | Xích lô trên phố, người lái và khách | R-04, R-05 | `cycle_rickshaw` cho Xích lô; `person` riêng cho người nhìn thấy. | Không gán Xích lô thành motorcycle. |
| `img_024` | Ô tô cổ trưng bày/đỗ trong khu vực lưu giữ xe | R-01, R-10 | Ô tô thật `car` nếu thân xe/bánh xe là phương tiện thực; không gán nội thất/ghế thành object riêng. | Xe thật không biến thành đồ chơi chỉ vì đang trưng bày. |
| `img_025` | Người đi bộ trên lối dạo ven biển; cảnh phố ở xa | R-07, R-10 | Gán từng người đủ rõ `person`; không gán cây, cột đèn, ghế hoặc vật ngoài taxonomy. | Chỉ các lớp người/phương tiện nằm trong phạm vi. |
| `img_029` | Xe máy chở hàng quá khổ | R-05 | Một box `motorcycle`; hàng hóa không gán riêng. | Hàng hóa là tải, không phải class phương tiện. |
| `img_030` | Nhiều xe máy đỗ sát nhau | R-01, R-07 | Một box `motorcycle` cho từng xe tách biệt; không gộp cả hàng xe. | Mỗi xe là một instance dù các box gần/chạm nhau. |
| `img_032` | Bến xe buýt/giờ cao điểm, bus, ô tô, người và cọc giao thông | R-01, R-04, R-10 | Bus `bus`; ô tô `car`; người `person`; không gán cọc tiêu/biển/cảnh nền. | Phân lớp bus riêng và loại vật ngoài taxonomy. |
| `img_034` | Người qua đường, xe máy và Xích lô trong cảnh giao thông | R-04, R-05, R-07 | Gán từng `person`, `motorcycle`, `cycle_rickshaw` nếu đủ rõ; box tách riêng. | Cần phân biệt Xích lô và xe máy trong cảnh giao cắt. |
| `img_035` | Auto-rickshaw chở quá tải | R-04, R-05 | Xe `auto_rickshaw`; người thật nhìn thấy là `person` riêng; hàng hóa không gán. | Phương tiện ba bánh có động cơ; không gộp người vào xe. |
| `img_039` | Xích lô trên đường phố, người và xe khác ở xa | R-04, R-05, R-07 | Xích lô `cycle_rickshaw`; người `person`; phương tiện khác chỉ gán khi nhận diện và đạt ngưỡng kích thước. | Tách phương tiện ba bánh đạp và xử lý vật nhỏ. |


Nếu chưa chốt được ca: đặt `can_review=true` trên box có thể vẽ hợp lý hoặc ghi `image, region, annotator, note` vào `image_issues.csv`. Dùng Rule ID ở trên trong nhật ký QC. Không tự sửa taxonomy trong lúc đang gán độc lập.

## 5. QC và phiên bản

### Ngưỡng QC của repository

| Chỉ số | Cách tính | Ngưỡng |
|---|---|---:|
| Chiều cao box tối thiểu | Box cao dưới ngưỡng không tạo instance thường; xử lý theo `ignore_region` nếu không thể phân giải | `20 px` |
| Ghép cặp box | Ghép 1–1 không phân biệt class khi IoU đạt ngưỡng | `IoU ≥ 0.50` |
| IAA – IoU trung bình | Chỉ tính trên cặp box ghép 1–1 | `≥ 0.85` |
| Đồng thuận class | Cùng class trên các cặp box ghép | `≥ 95%` |
| IoU kiểm box lớn | Box cao `≥ 50 px` | `≥ 0.85` |
| IoU kiểm box nhỏ | Box cao `< 50 px` (nhưng không dưới ngưỡng tối thiểu) | `≥ 0.70` |
| Tỷ lệ ảnh có lỗi | Số ảnh có ít nhất một lỗi / số ảnh đã duyệt | `≤ 2%` |
| Độ phủ gán nhãn | Lượt gán hợp lệ / tổng lượt theo phân công | `100%` |


### Người chốt và quyết định QC

- Annotator tự gán độc lập, không xem nhãn người khác trước khi khóa lượt gán.
- Reviewer/coach hoặc người được nhóm chỉ định làm adjudicator quyết định ca bất đồng; ghi `case_id`, ảnh/vùng, hai cách gán, Rule ID, quyết định, người chốt, version guideline và kết quả kiểm lại.
- **Hold:** thiếu lượt gán/bằng chứng, taxonomy chưa được thống nhất, mới chỉ kiểm mẫu hoặc còn issue chưa giải quyết.
- **Rework:** tỷ lệ ảnh lỗi >2% hoặc có lỗi hệ thống/lặp lại cùng một Rule ID; sửa guideline/nhãn trong phạm vi ảnh hưởng và kiểm lại.
- **Release:** đủ 100% độ phủ, IAA đạt ngưỡng, QC toàn bộ 40 ảnh đạt, không còn issue chưa xử lý và reviewer xác nhận.

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v1.0` | Chuẩn hóa theo khung 5 phần; đề xuất mở rộng `bus`, `bicycle`, `auto_rickshaw`, `cycle_rickshaw`, `handcart`; định nghĩa ngưỡng thuộc tính, quy tắc R-01–R-10 và ca khó theo 40 ảnh. | Các ảnh `img_002`–`img_040` có car carrier, Xích lô, tuk-tuk, bus, xe đạp/xe đẩy, tượng, poster, phản chiếu, mascot, mô hình và cảnh đông. |

**Điều kiện khóa v1.0:** nhóm phải xác nhận taxonomy mở rộng, đặc biệt `bus`, `bicycle`, `auto_rickshaw`, `cycle_rickshaw`, `handcart`; sau đó cập nhật `submission/taxonomy.json`, giữ nguyên các ngưỡng `rules` do repository quy định, chạy `make_cvat_labels.py` và kiểm tra cấu hình CVAT trước khi bắt đầu gán độc lập.
