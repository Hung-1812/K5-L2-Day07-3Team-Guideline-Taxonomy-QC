# Guideline gán nhãn bounding box: ảnh giao thông đường phố

- **Phiên bản:** v1.1 (hiệu chỉnh sau calibration vòng 1, từ v1.0 gồm bản 302, 301, 303)
- **Trạng thái:** bản nháp v1.1. Các mục đánh dấu **(chờ adjudicate)** phải được reviewer chốt trên ảnh gốc trước khi khóa (xem Mục 5.7).
- **Thay đổi so với v1.0:** thay đổi **semantic** (xem Phụ lục B); **taxonomy giữ nguyên v1.0**.
- **Loại nhãn:** bounding box (hộp chữ nhật) trên ảnh đường phố
- **Công cụ:** CVAT, xuất COCO 1.0
- **Khách hàng / mục đích:** nhận diện xe cộ và người trên đường phố để xe tự lái tránh va chạm. Khách hàng không quy định lớp nhãn; nhóm tự quyết định.
- **Cách dùng nhanh:** đọc Mục 2.2 (cây phân lớp) và Mục 3.1 (cây quyết định) trước khi gán. Mã `R-xx` trong tài liệu này là **mã mới**, thay cho mã trong các bản cá nhân. Dùng mã này trong `calibration_decisions.csv` và phiếu QC.

---

## 1. Mục tiêu

### 1.1 Mô hình dùng nhãn
Nhãn huấn luyện và đánh giá mô hình **phát hiện vật thể 2D**: với mỗi **người thật** và **phương tiện thật** trong ảnh, mô hình xuất ra một hộp và một lớp. Kết quả dùng cho hệ thống xe tự lái **tránh va chạm**.

### 1.2 Nguyên tắc chung
| Nguyên tắc | Ý nghĩa khi gán |
|---|---|
| **Gán những gì nhìn thấy được** | Không đoán phần bị che, không đoán thứ ta nghĩ là có. |
| **Không bỏ sót vật thật** | Bỏ sót nguy hiểm hơn vẽ thừa. Phương tiện nào cũng phải có lớp: không gán sẽ dạy mô hình coi nó là nền. |
| **Không tạo "vật ma"** | Tượng, hình in, phản chiếu mà gán thì mô hình học sai. |
| **Nhất quán hơn cầu toàn** | Cùng một tình huống phải luôn được gán giống nhau, bởi bất kỳ ai. |

### 1.3 Phép thử chọn *lớp* hay *thuộc tính*
> Mô hình phát hiện 2D có phải **đoán ra** thông tin này khi suy luận không?
> **Có** → **lớp** (ví dụ `bus`, `tuk_tuk`, `motorcycle`). **Không**, chỉ dùng để lọc mẫu khó hoặc kiểm QC → **thuộc tính** (ví dụ `occluded_level`, `truncated`, `can_review`).

### 1.4 Ngoài phạm vi
- Biển số, màu xe, hướng xe, hành vi (đi sai làn, không đội mũ), ý định của người đi đường.
- Nhà cửa, đường, vỉa hè, cây, cột và dây điện, biển báo, đèn giao thông, vạch kẻ đường, nón giao thông, chim và động vật (xem R-15).
- Ước lượng 3D và theo dõi (tracking).

---

## 2. Lớp và thuộc tính

### 2.1 Lớp (mỗi vật thuộc đúng **một** lớp)

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ (trông giống nhưng KHÔNG thuộc lớp này) |
|---|---|---|---|
| `person` | **Người thật** nhìn thấy **trực tiếp** trong cảnh: đi bộ, đứng, ngồi, hoặc lái / ngồi trên xe nếu cơ thể thấy được không qua kính. Người hóa trang thấy rõ cơ thể người cũng tính. | Người đi bộ; người lái và người ngồi sau xe máy; người đạp xe và xích lô; người đẩy xe; khán giả; tài xế tuk-tuk thấy qua phía hở (`img_004`). | Tượng (`img_017`); đầu điêu khắc (`img_011`); ma-nơ-canh, ảnh in (`img_033`); mascot không thấy người bên trong (`img_019`); phản chiếu (`img_020`); người sau kính ô tô, van, bus. |
| `car` | Ô tô con chở người, khoang kín: sedan, hatchback, wagon, coupe, SUV, taxi; kể cả xe cổ, xe diễu hành đủ 3 dấu hiệu của R-06, xe đang được chở trên car carrier. | Sedan (`img_023`); xe diễu hành (`img_019`); xe cổ trưng bày (`img_024`); từng xe trên sàn car carrier (`img_002`). | Bán tải (→ `truck`); xe thùng kín (→ `van`); xe khách (→ `bus`); xe mô hình (`img_024`, `img_038`). |
| `truck` | Xe có cấu trúc chính để **chở hàng** hoặc **chở / kéo phương tiện khác**: **cabin tách rời thùng hoặc sàn**. Gồm cả bán tải (pickup), kể cả có nắp thùng cao. | Xe tải thùng, xe ben, xe bồn, xe cẩu, đầu kéo + rơ-moóc; car carrier (`img_002`); bán tải (`img_001`, `img_016`, `img_037`). | Ô tô con; van; bia mộ tạc hình xe bán tải (`img_027`). |
| `van` | Xe cơ giới thân kín **liền khối** (khoang người / hàng không có vách tách cabin và thùng), từ 16 chỗ trở xuống. | Xe giao hàng thùng kín, minivan, minibus nhỏ (`img_016`: van trắng). | Bán tải (cabin tách thùng → `truck`); xe khách cỡ lớn (→ `bus`). |
| `bus` | Xe buýt, xe khách **cỡ lớn**, hàng cửa sổ liên tiếp, trên 16 chỗ. | `img_031`, `img_032`, `img_037`, `img_040`. | Minibus 16 chỗ trở xuống (→ `van`); xe buýt đồ chơi (R-02). |
| `motorcycle` | Xe **hai bánh có động cơ**: xe máy, xe tay ga, mô tô, xe máy điện dáng scooter. Xe gắn sidecar là **một** instance. **Box chỉ ôm xe**, không ôm người (R-05). | Xe máy có người lái và xe đỗ (`img_016`, `img_023`); xe sidecar (`img_022`). | Xe đạp (→ `bicycle`); xe ba bánh (→ `tuk_tuk` / `cycle_rickshaw`); xe điêu khắc trưng bày (`img_011`). |
| `bicycle` | Xe **hai bánh không động cơ**, chạy bằng sức người: xe đạp thường, xe đạp có trợ lực dáng xe đạp (có bàn đạp). | Xe đạp trên đường. | Xe máy (→ `motorcycle`); xích lô ba bánh (→ `cycle_rickshaw`). |
| `tuk_tuk` | Xe **ba bánh có động cơ**, có khoang hoặc mái, chở người hoặc hàng (tuk-tuk, auto-rickshaw, xe lam). | `img_004`, `img_016`, `img_035`. | Xích lô đạp (→ `cycle_rickshaw`); xe máy hai bánh. |
| `cycle_rickshaw` | Xe **ba bánh không động cơ**, người đạp chở khách hoặc hàng: xích lô, cyclo, pedicab. | Xích lô (`img_023`; cùng loại ở `img_006`, `img_034`, `img_039`). | Tuk-tuk (có động cơ); xe đạp hai bánh; xe đẩy tay. |
| `handcart` | Xe **đẩy / kéo tay có bánh**, dùng chở hoặc bán hàng trên đường. | Xe đẩy trái cây, xe hàng rong (`img_003`, `img_010`). | Xe nôi, xe lăn, xe đẩy siêu thị (R-15); hàng hóa trên xe; người bán hàng không có xe. |
| `ignore_region` | Vùng chữ nhật cho khu vực có nhiều vật **thật** quá nhỏ (cao < 20 px), quá dày hoặc quá nhòe để tách đáng tin cậy (R-12). | Cụm xe máy đỗ dày ở xa; lõi đám đông không tách được. | Vùng phủ lên vật rõ đã có box; vật không thật (R-02). |

### 2.2 Cây phân lớp nhanh (kiểm tra theo thứ tự)
1. **Người thật** → `person` (độc lập với box phương tiện).
2. **Hai bánh:** có động cơ → `motorcycle`; không động cơ → `bicycle`.
3. **Ba bánh:** có động cơ → `tuk_tuk`; đạp bằng sức người → `cycle_rickshaw`.
4. **Xe đẩy / kéo tay có bánh** chở hoặc bán hàng → `handcart`.
5. **Bốn bánh trở lên có động cơ:** khách cỡ lớn → `bus`; thân kín liền khối ≤ 16 chỗ → `van`; cabin tách thùng hoặc xe chuyên chở hàng / xe → `truck`; còn lại → `car`.
6. **Không đủ bằng chứng phân lớp:** không đoán (R-14).

### 2.2.1 Bảng phân biệt cặp dễ nhầm (mới ở v1.1)

Kiểm các dấu hiệu theo thứ tự. **Hàng hóa chở trên xe không làm đổi lớp xe.** Cột "Ca tham chiếu" là các ca calibration cần reviewer chốt trước khi chọn ảnh minh họa (**chờ adjudicate**).

| Cặp dễ nhầm | Dấu hiệu quyết định | Ca tham chiếu |
|---|---|---|
| `tuk_tuk` / `cycle_rickshaw` / `bicycle` (số bánh và động cơ) | Hai bánh không động cơ → `bicycle`. Ba bánh, người ngồi trên yên đạp bằng chân, có ghế khách gắn liền → `cycle_rickshaw`. Ba bánh có động cơ (đầu máy, tay ga, người lái không đạp), có khoang hoặc mái → `tuk_tuk`. Chỉ thấy hai bánh nhưng có ghế khách hoặc thùng chở gắn liền → nhiều khả năng `cycle_rickshaw`, không chắc thì `can_review` (`LOP:`). | C253 (bicycle / cycle_rickshaw) |
| `truck` / `tuk_tuk` (xe chở hàng nhỏ) | Ba bánh có động cơ → `tuk_tuk`, dù đang chở hàng. Từ bốn bánh trở lên, hoặc cabin tách thùng / sàn → `truck`. | C389 (truck / tuk_tuk) |
| `truck` / `car` (bán tải, SUV) | Có cabin và thùng tách rời (kể cả thùng hở, có bạt, có nắp thùng cao) → `truck`. Khoang kín liền khối từ đầu đến đuôi (SUV, wagon, hatchback) → `car`. | C097 (truck / car) |
| `car` / `van` (wagon, xe thùng kín) | Dáng ô tô con (có đầu xe nhô, đuôi như ô tô con, từ 4–7 chỗ) → `car`. Thân khối hộp cao, khoang người / hàng liền khối kiểu xe giao hàng, minibus nhỏ → `van`. Không quyết định chỉ dựa vào có hay không có cửa sổ sau. | C060 (car / van) |
| `person` / phương tiện | Box `person` chỉ ôm cơ thể người; yên, thân xe, mui, hàng không thuộc `person`. Xe hay phần xe lộ ra không được gán thành `person`. Xem R-05. | C119 (motorcycle / person), C144, C264 (car / person) |

### 2.3 Thuộc tính (áp dụng cho mọi lớp, trừ `ignore_region`)

| Thuộc tính | Giá trị | Định nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc che không đáng kể: **0–10%** thân vật. |
| | `1` | Bị **vật khác trong cảnh** che **> 10–30%**. |
| | `2` | Bị che **> 30–70%**. |
| | `3` | Bị che **> 70%** nhưng vẫn còn đủ dấu hiệu để nhận ra lớp và tách được instance. |
| `truncated` | `no`, `yes` | `yes` khi vật **thực sự bị khung ảnh cắt cụt**: thân vật kéo dài ra ngoài ảnh và thấy mặt cắt ngang ở mép. `no` khi vật nhìn trọn, kể cả khi box chạm biên (sai số ±2 px). Bị vật khác che là `occluded_level`, **không** phải `truncated`. Một vật có thể vừa `truncated=yes` vừa `occluded_level > 0`. |
| `can_review` | tick (`true`) / không (`false`) | Tick khi nghi vấn về lớp, số instance, ranh giới box, mức che, hoặc gặp tình huống chưa có trong guideline. **Bắt buộc ghi lý do ngắn** ở ô ghi chú của CVAT. Không tick "cho chắc" trên vật rõ. |

Lưu ý:
- **Phân vân giữa hai mức `occluded_level` liền kề: chọn mức cao hơn.** Phân vân cách nhau từ hai mức trở lên: chọn mức bạn nghĩ gần nhất và tick `can_review`.
- **Không dùng `occluded_level = 3` để biểu thị "không rõ".** Nếu không còn đủ dấu hiệu nhận dạng thì không ép: xem R-10 và R-14.
- Chỉ dùng thuộc tính `occluded_level`. Cờ `occluded` mặc định của CVAT (phím `Q`) bị script bỏ qua; có thể bật cờ đó nhưng nó **không thay** `occluded_level`.


### 2.4 Cách ước lượng `occluded_level` (mới ở v1.1; **giữ nguyên các ngưỡng ở 2.3**)

1. Hình dung hình dạng **đầy đủ** của vật, rồi ước lượng phần diện tích bị **vật khác trong cảnh** che (xe khác, người, cột, cây, biển).
2. **Không tính** vào mức che: mờ chuyển động, thiếu sáng hoặc lóa, và phần vật nằm ngoài khung ảnh (đó là `truncated`).
3. Đối chiếu với bảng mô tả dưới đây, rồi chọn mức. Phân vân giữa hai mức liền kề: chọn mức **cao hơn**.

| Mức | Tỷ lệ che | Hình ảnh điển hình |
|---|---|---|
| `0` | 0–10% | Thấy trọn vật; chỉ bị che chút ít (một phần bánh, một góc, cột mảnh đi ngang). |
| `1` | > 10–30% | Còn thấy rõ hơn 2/3 vật; bị che một góc (đuôi xe sau xe khác, một chân người). |
| `2` | > 30–70% | Chỉ thấy khoảng nửa vật (xe bị xe đứng trước che gần một nửa; người chỉ thấy từ ngực trở lên). |
| `3` | > 70% | Chỉ lộ một phần nhỏ (mui xe, đầu và vai người) nhưng vẫn đủ để nhận ra lớp. |

Mẹo: nếu hai người gán chênh nhau từ hai mức trở lên, thường là một người đã tính cả mờ hoặc phần ngoài khung. Kiểm lại trước khi chốt. Ảnh tham chiếu cho từng mức sẽ lấy từ các ca đã adjudicate: **(chờ adjudicate)**.

---

## 3. Quy tắc vẽ box

### 3.1 Cây quyết định nhanh (theo thứ tự)
1. Đây có phải **vật thật** hiện diện trong cảnh không? Tượng, mô hình, đồ chơi, hình in, phản chiếu → **không gán** (R-02, R-03). Xe chủ → không gán (R-03).
2. Vật có cao **< 20 px**, hoặc thuộc cụm quá dày / nhòe không tách được? → `ignore_region` (R-12, R-13).
3. Chọn **lớp** theo cây 2.2. Chưa có lớp phù hợp → không ép, xem R-15.
4. **Vẽ box** ôm sát phần nhìn thấy (R-09), chọn `occluded_level` và `truncated`.
5. Còn lăn tăn → tick `can_review` hoặc ghi `image_issues.csv` (R-14). **Không hỏi người gán cùng ảnh.**

### 3.2 Danh sách quy tắc

| Mã | Quy tắc |
|---|---|
| **R-01** | **Gán mọi vật thật; một vật = một box = một lớp.** Kể cả xe đang đỗ, xe trưng bày trong nhà, xe ở xa (cao ≥ 20 px), xe bị che một phần. Không vẽ hai box cho một vật. Không gộp nhiều vật vào một box chỉ vì chúng sát nhau. |
| **R-02** | **Không gán vật không thật:** tượng, điêu khắc, bia mộ hình xe, ma-nơ-canh, hình nộm, đồ chơi, xe mô hình, mascot không thấy người thật bên trong, người / xe in trên poster, billboard, tranh, màn hình. Ảnh chỉ có các vật này vẫn giữ trong Task và xuất 0 annotation. **Không vẽ `ignore_region` lên các vật này.** |
| **R-03** | **Không gán phản chiếu và không gán xe chủ.** Vật chỉ xuất hiện trong gương, kính, mặt nước thì không gán (kể cả cầu vồng, cảnh vật phản chiếu). **Xe chủ** (xe mang camera: cửa, gương, kính chắn gió, nắp capô thấy từ trong xe) cũng không gán. |
| **R-04** | **Phân lớp theo cấu tạo, không theo công dụng**, theo cây ở Mục 2.2 và bảng phân biệt cặp dễ nhầm ở Mục 2.2.1. Không ép vật vào lớp "gần giống" khi không đủ dấu hiệu. |
| **R-05** | **Người và phương tiện.** (a) *Xe hai bánh, xích lô, handcart:* vẽ box xe **và** box `person` riêng cho từng người; hai box **được chồng lấn**. **Tách bạch (v1.1):** box `person` chỉ ôm **cơ thể người nhìn thấy** (đầu, thân, tay, chân); yên, thân xe, mui, hàng hóa **không** thuộc box `person`. Box xe chỉ ôm xe, không kéo lên đầu người. Người lái **không thay thế** box xe, và xe hoặc phần xe lộ ra **không** được gán nhầm thành `person`. Một vùng ảnh là người hoặc là xe, không vẽ hai box cho cùng một vật. (b) *Người trong xe:* chỉ gán `person` khi cơ thể thấy **trực tiếp** (phía hở của tuk-tuk, xe mui trần, thùng xe). Người sau kính chắn gió hoặc kính cửa của ô tô, van, bus, hoặc không nhìn rõ → **không gán**, không suy đoán qua kính tối. Thấy một phần qua kính, phần qua chỗ hở → tick `can_review`. (c) *Nhiều người trên một xe:* một box xe, mỗi người phân biệt được một box `person`. (d) *Người hóa trang:* gán `person` nếu thấy cơ thể người rõ; chỉ thấy hình nhân vật thì theo R-02. |
| **R-06** | **Xe thật hay vật hình xe? Phép thử 3 dấu hiệu.** Vật hình phương tiện chỉ được gán khi đủ cả 3: (a) kích thước xe thật, đủ chỗ cho người lái; (b) có bánh và có thể di chuyển trên đường (biển số, đèn, cửa); (c) không phải đồ trang trí cố định hay tác phẩm trưng bày. Đủ 3 → gán theo lớp (kể cả xe diễu hành, xe cổ, xe cách điệu, xe đang đỗ hoặc trưng bày). Thiếu một → R-02. |
| **R-07** | **Xe chở xe, xe kéo xe, rơ-moóc.** *Car carrier:* **1 box `truck`** ôm toàn bộ khung và các xe trên sàn (không box riêng cho giá đỡ) **và 1 box `car` cho mỗi xe nhìn thấy đủ để nhận ra**; box lồng nhau là bình thường. *Đầu kéo + rơ-moóc:* **1 box `truck`** bao cả hai. *Xe cứu hộ kéo xe khác:* hai box riêng, mỗi xe theo lớp của nó. |
| **R-08** | **Hàng hóa và bộ phận gắn liền.** Hàng hóa không có lớp riêng và **không tính vào box**: box chỉ tới mép ngoài của thân, thùng, khung xe (mui, thành thùng, giá nóc, khung tuk-tuk), **không kéo lên đống hàng xếp cao** (`img_035`, xe máy chở hàng quá khổ). Ngoại lệ: các xe nằm trên sàn car carrier (R-07). *Sidecar* gắn liền: **một** box `motorcycle` bao cả cụm. Hàng che xe thì ghi bằng `occluded_level`. |
| **R-09** | **Ôm sát phần nhìn thấy.** Bốn cạnh box chạm sát điểm ngoài cùng nhìn thấy của vật (thân, bánh, gương, cản, tay lái); không gồm bóng đổ, mặt đường, vật bên cạnh. Sai số 1–2 px với vật lớn; vật nhỏ thì box không lớn hơn vật quá ~20%. Phóng to khi vẽ vật nhỏ. **Không đoán phần bị che.** Phần nhìn thấy bị chia đôi bởi vật che mỏng (cột điện): vẽ một box bao cả hai phần. Các box **được chồng lấn** khi vật thật chồng lấn. |
| **R-10** | **Che khuất.** Vẫn gán nếu nhận ra chắc chắn lớp và tách được instance; chọn `occluded_level` theo Mục 2.3 và cách ước lượng ở Mục 2.4. Chỉ tính phần bị **vật khác trong cảnh** che; mờ, thiếu sáng, lóa và phần nằm ngoài khung ảnh **không phải** che khuất. Nếu không đủ dấu hiệu: có box tạm hợp lý thì tick `can_review`; không thể tạo box đáng tin cậy thì ghi `image_issues.csv`. |
| **R-11** | **Cắt mép ảnh.** Vẫn gán nếu phần nhìn thấy đủ để nhận diện. Box dừng đúng tại biên ảnh (không vượt canvas, không ước lượng phần ngoài ảnh). Đặt `truncated = yes` **khi vật thực sự bị khung ảnh cắt cụt** (thân vật kéo dài ra ngoài ảnh, thấy mặt cắt ngang ở mép). Vật vẫn nhìn trọn mà chỉ nằm sát biên (box chạm biên ±2 px) thì `truncated = no`: **box chạm biên không đồng nghĩa `truncated`**. Chỉ còn mảnh quá nhỏ không nhận ra → không gán, ghi `image_issues.csv`. |
| **R-12** | **Vật nhỏ và đám đông.** Vật cao **< 20 px** hoặc cụm vật chồng dày không tách được → `ignore_region` thay cho từng box. Vật cao ≥ 20 px, **tách được** (nhận ra là một vật riêng, không bị che > 70%) → **vẫn gán từng box**, kể cả người ở rìa đám đông. Đám đông: không bao giờ gộp thành một `person`. **Chỉ vẽ `ignore_region` sau khi đã rà lượt 2 (R-16).** `ignore_region` bao **chặt** quanh cụm, **không** phủ lên vật rõ đã gán, **không** dùng một hộp khổng lồ cho nửa ảnh, **không** dùng như đường tắt cho vật khó nhưng gán được. `ignore_region` không có thuộc tính. |
| **R-13** | **Mờ chuyển động, thiếu sáng, lóa.** Vẫn gán nếu nhận ra lớp và ranh giới; box ôm vùng lõi nhìn thấy. Chỉ là vệt nhòe không xác định được vật thì không gán; nếu thuộc cụm không phân giải được, dùng `ignore_region` gọn quanh cụm, ngược lại ghi `image_issues.csv`. Nhòe không phải che khuất. |
| **R-14** | **Ca không chắc.** (a) Đã vẽ được box hợp lý → tick `can_review` và **ghi lý do cụ thể** theo mẫu `NHOM: mô tả ngắn; vùng cần xem`, với `NHOM` là `LOP` (phân vân lớp), `BIEN` (ranh giới box), `SO` (số instance), `CHE` (mức che), `CAT` (cắt mép) hoặc `KHAC`. Ví dụ: `LOP: car hay van? wagon thân dài; vùng giữa ảnh`. Không ghi chung chung kiểu "ca mơ hồ". (b) Chưa thể vẽ box đáng tin cậy → ghi `submission/qc/image_issues.csv` (cột `image,annotator,region,note`). Không tự đoán, không hỏi người gán cùng ảnh, không tự sửa taxonomy trong lúc gán độc lập. |
| **R-15** | **Vật ngoài taxonomy.** Không gán nhà, đường, cây, cột, biển báo, đèn, vạch kẻ, nón giao thông, động vật, hàng hóa riêng lẻ, xe nôi, xe lăn, xe đẩy siêu thị. Không ép vào lớp gần nhất. **Người** đẩy xe nôi hoặc ngồi xe lăn vẫn gán `person`. Gặp vật ngoài taxonomy xuất hiện nhiều → ghi `image_issues.csv` để nhóm xét thêm lớp. |
| **R-16** | **Rà soát hai lượt, đặc biệt cảnh đông.** *Lượt 1:* nhìn toàn ảnh, vẽ từng người và xe nhận ra được, từ lớn đến nhỏ. *Lượt 2:* phóng to (khoảng 200–300%) rà lần lượt vật nhỏ, vùng chồng lấn và mép ảnh, rồi đối chiếu với những gì đã vẽ. **Tách trước, bỏ qua sau:** chỉ vẽ `ignore_region` khi lượt 2 xác nhận cụm không tách được (R-12), vẽ gọn quanh cụm và ghi số vật đã gán trong cụm vào ghi chú. Không dùng `ignore_region` thay cho việc rà các vật còn phân biệt được. |

### 3.3 Quy trình khi gán một ảnh (hai lượt, R-16)
1. **Lượt 1, toàn ảnh:** vẽ từng người và xe nhận ra được, từ vật lớn, rõ đến vật nhỏ, bị che. Ghi nhận vùng đông, vùng khó.
2. **Lượt 2, phóng to:** rà vật nhỏ, vùng chồng lấn và mép ảnh. Chỉ sau lượt này mới vẽ `ignore_region` cho cụm thực sự không tách được.
3. Đặt `occluded_level` và `truncated` **ngay khi vẽ** từng box.
4. **Tự kiểm trước khi chốt ảnh:**
   - Còn vật thật nào chưa có box? Với ảnh đông, đã rà lượt 2 chưa?
   - Có box thừa, trùng, hoặc vẽ lên phản chiếu, hình in, tượng?
   - Box ôm sát chưa? Box `person` không chứa xe, box xe không chứa người (R-05)?
   - `truncated = yes` chỉ cho vật bị khung ảnh cắt cụt (R-11)?
   - Mọi `can_review` đã có lý do theo mẫu `NHOM: mô tả; vùng` chưa (R-14)?
   - Ảnh 0 box vẫn còn trong Task chưa?
5. Không sửa nhãn người khác trực tiếp; ghi chú và báo lại.

---

## 4. Ca khó

Mỗi ca trả lời đủ 4 câu hỏi: **(1) Cần gán?** · **(2) Lớp có sẵn?** · **(3) Hay gặp?** · **(4) Quyết định kèm ảnh ví dụ**. Đây là **quyết định hướng dẫn**, không phải nhãn ground truth. Luôn mở ảnh gốc trong CVAT để xác định từng instance.

### 4.1 Nhóm A: ca đã xác minh trên ảnh (12 ảnh)

| Ảnh | Tình huống | (1) Cần gán? | (2) Lớp có sẵn? | (3) Hay gặp? | (4) Quyết định (quy tắc) |
|---|---|---|---|---|---|
| `img_002` | Xe đầu kéo chở 8 ô tô con trên hai sàn; công nhân đứng cạnh | **Có**: xe chở, từng xe con thấy được, công nhân | `truck` (xe chở); `car` (xe con); `person` | Ít (1–2 ảnh) nhưng bất đồng lớn nhất giữa các bản | **1 `truck`** ôm cả khung và xe trên sàn + **1 `car` cho mỗi xe nhìn thấy** + 1 `person`; `occluded_level` xe con theo khung che (R-07, R-08) |
| `img_004` | Hai tuk-tuk taxi đỗ ban đêm, che nhau; tài xế trong khoang; nón giao thông ở góc | **Có**: 2 tuk-tuk; tài xế **thấy trực tiếp**; **Không**: nón | Chưa có trong v0 → `tuk_tuk` | **Có** (`img_004`, `016`, `035`) → lớp mới | 2 `tuk_tuk`; xe phải chọn `occluded_level` theo %; tài xế thấy qua phía hở → `person`, chỉ thấy qua kính chắn gió → không gán, phân vân → `can_review` (R-05, R-15) |
| `img_011` | Hội chợ: đầu điêu khắc khổng lồ, xe điêu khắc ghép từ phụ tùng, đám đông, 1 xe van bị che | **Có**: người thật, van. **Không**: đầu điêu khắc, xe điêu khắc | `person`, `van`; điêu khắc không thuộc lớp nào | Tượng / điêu khắc ≥ 5 ảnh | `person` từng người rõ; `van` + `occluded_level`; đám đông xa → `ignore_region`; điêu khắc **không gán**; xe điêu khắc ghi `image_issues.csv` chờ adjudicator (R-02, R-06, R-12) |
| `img_016` | Phố đông: tuk-tuk, bán tải nắp thùng cao bị che gần hết cabin, xe máy, van, ô tô, người | **Có**: mọi phương tiện và người ngoài | `tuk_tuk`, `truck` (bán tải), `van`, `car`, `motorcycle`, `person` | Tuk-tuk ≥ 3 ảnh; bán tải nắp cao ≥ 2 ảnh | Vẽ đủ theo lớp. Bán tải không thấy cabin: `truck` + `can_review`. Đầu người nhỏ ở góc: `person` + `truncated` + `can_review` nếu chắc, không thì `image_issues.csv` (R-04, R-05, R-10, R-11) |
| `img_017` | Tượng đồng người cưỡi ngựa đội nón giao thông, có chim | **Không**: tượng là vật không thật; nón và chim ngoài phạm vi | `person` giống nhưng không áp dụng | Có (tượng ≥ 4 ảnh); dễ "cùng sai" | **0 box** (R-02, R-15) |
| `img_019` | Xe diễu hành hình ô tô hoạt hình; người lái; nhân vật chuột trắng; khán giả sau hàng rào | **Có**: xe, người lái thấy trực tiếp, khán giả. **Không**: nhân vật chuột | `car`; `person` | Ít (1 ảnh) nhưng bản nào cũng diễn giải khác | 1 `car` (đủ 3 dấu hiệu); `person` người lái nếu thấy trực tiếp; khán giả hàng đầu từng `person` (hàng rào che → `occluded_level`), hàng sau dày → `ignore_region`; mascot không gán, ghi `image_issues.csv`; đầu mờ ở góc: `person` + `truncated` + `can_review` (R-05, R-06, R-12) |
| `img_020` | Gương chiếu hậu ướt; xe xám xa, cầu vồng trong gương; thân xe chủ | **Không**: phản chiếu và xe chủ | `car` giống nhưng không áp dụng | Ít (1–2 ảnh) | **0 box** (R-03) |
| `img_023` | Phố VN: sedan, xích lô, xe máy có người, xe máy đỗ dày | **Có**; xe đỗ dày: gán từng xe nếu tách được | `car`, `cycle_rickshaw`, `motorcycle`, `person` | Xích lô ≥ 5 ảnh | `car`; `cycle_rickshaw` + `person` người đạp; `motorcycle` + `person` người lái; xe máy đỗ cao ≥ 20 px tách được → từng box, còn lại → `ignore_region` (R-05, R-12) |
| `img_024` | Xe đen thật có mô hình xe vàng trên nóc; nhiều xe cổ trưng bày trong nhà | **Có**: xe thật (đen và xe cổ nền). **Không**: mô hình trên nóc | `car` | Ít (1–2 ảnh) | `car` cho xe đen, box tới nóc và giá nóc, **không kéo lên mô hình**; xe cổ nền tách rõ → box, hàng xa dày → `ignore_region` (R-01, R-02, R-06, R-08) |
| `img_027` | Bia mộ granit tạc hình xe bán tải | **Không**: tác phẩm tạc | `truck` giống nhưng thiếu dấu hiệu xe thật | Ít (1 ảnh) | **0 box**; phản ví dụ chuẩn của `truck` (R-02, R-06) |
| `img_033` | Kính cửa hàng: ma-nơ-canh, ảnh in người mẫu, hình xe trắng như phản chiếu | **Không**: ma-nơ-canh, ảnh in, phản chiếu | `person`, `van` giống nhưng không áp dụng | Có (ma-nơ-canh, ảnh in, phản chiếu ≥ 4 ảnh) | **0 box** (R-02, R-03). Nếu hóa ra xe thật sau kính: `image_issues.csv` |
| `img_038` | Tủ kính đồ chơi: vịt cao su, Mickey, ~20 xe mô hình | **Không**: đồ chơi, mô hình | `car`... giống nhưng là mô hình | Có (`img_024`, `img_038`) | **0 box**; **không** vẽ `ignore_region` lên tủ kính (R-02) |

### 4.2 Nhóm B: ca khó theo mô tả của bản 303 và tài liệu lab (**chưa xác minh trên ảnh**; phải mở ảnh kiểm tra)

| Ảnh | Tình huống | (1) Cần gán? | (2) Lớp có sẵn? | (3) Hay gặp? | (4) Quyết định (quy tắc) |
|---|---|---|---|---|---|
| `img_001` | Bán tải cạnh người tại quán ven đường | Có | `truck`, `person` | Có (bán tải ≥ 3 ảnh) | Hai box tách riêng (R-04, R-05) |
| `img_003`, `img_010` | Người bán hàng rong, xe đẩy chở trái cây, xe máy | Có: người, xe đẩy, xe máy. Không: trái cây | `person`, `handcart`, `motorcycle` | 2 ảnh | `handcart` nếu nhận ra cấu trúc xe đẩy có bánh; hàng không gán (R-05, R-08, R-15) |
| `img_005`, `img_008`, `img_021`, `img_030` | Nhiều xe máy và người chồng lấn; chợ đêm; xe máy đỗ sát nhau | Có: từng xe, từng người tách được | `motorcycle`, `person` | **Rất hay gặp** (≥ 4 ảnh) | Mỗi xe một box, mỗi người một box, chồng lấn bình thường; lõi quá dày hoặc nhỏ < 20 px → `ignore_region`; không gộp cả hàng xe (R-01, R-05, R-12) |
| `img_006`, `img_034`, `img_039` | Giao thông có xích lô, xe máy, người | Có | `cycle_rickshaw`, `motorcycle`, `person` | Có (xích lô ≥ 5 ảnh) | Xích lô không phải `motorcycle` hay `tuk_tuk`; người đạp `person` riêng (R-04, R-05) |
| `img_009` | **Hai nguồn mô tả khác nhau:** bản 303 ghi xích lô và người; tài liệu lab ghi xe buýt | Có (theo vật thật nhìn thấy) | `cycle_rickshaw` hoặc `bus` | ? | **Mở ảnh xác minh** rồi áp dụng theo lớp của vật nhìn thấy |
| `img_013`, `img_025` | Phố đêm, cảnh dạo ven biển: người xa, cây, đèn, ghế | Chỉ người / xe đủ rõ | `person`, các lớp xe | Có (cảnh nền phức tạp) | Chỉ gán vật đủ rõ, cao ≥ 20 px; bỏ cây, đèn, ghế, nhà (R-12, R-13, R-15) |
| `img_014` | Người đi bộ qua đường, motion blur | Có nếu còn nhận ra cơ thể | `person` | Có (nhòe `img_014`, `img_015`) | Gán nếu nhận ra cơ thể và biên; chỉ là vệt nhòe thì không gán, ghi `image_issues.csv` (R-13) |
| `img_015` | Ô tô trên đường ướt, bị cắt ở hai mép ảnh | Có | `car` | Có (cắt mép) | `car` + `truncated = yes`, box dừng đúng biên ảnh (R-11) |
| `img_018` | Billboard có hình người và chữ | **Không** | `person` giống nhưng là ảnh in | Có | **0 box** cho hình in (R-02) |
| `img_022` | Xe máy gắn sidecar | Có | `motorcycle` | Ít | **Một** box `motorcycle` bao cả sidecar; người ngồi → `person` riêng (R-05, R-08) |
| `img_029` | **Hai nguồn mô tả khác nhau:** bản 303 ghi xe máy chở hàng quá khổ; tài liệu lab ghi xe đạp / xích lô | Có (theo vật nhìn thấy) | `motorcycle` / `bicycle` / `cycle_rickshaw` | ? | **Mở ảnh xác minh**; hàng quá khổ không tính vào box (R-08) |
| `img_031`, `img_032`, `img_037`, `img_040` | Xe buýt, bến xe buýt; `img_037` còn có bán tải nắp thùng cao | Có: bus, ô tô, người. Không: cọc tiêu, biển | `bus`, `car`, `truck`, `person` | Có (bus ≥ 4 ảnh) | `bus` riêng (không ép vào `truck` / `van`); bán tải nắp cao → `truck` (R-04, R-15) |
| `img_035` | Tuk-tuk chở hàng xếp cao, quá tải | Có: xe, người nhìn thấy. Không: hàng | `tuk_tuk`, `person` | Có (tuk-tuk ≥ 3 ảnh) | Box chỉ tới **mép trên khung xe**, không kéo lên bao hàng (R-08) |

### 4.3 Nhóm C: tình huống chung (từ bản 301, chưa gắn ảnh cụ thể)

| Tình huống | (1) Cần gán? | (2) Lớp có sẵn? | (3) Hay gặp? | (4) Quyết định (quy tắc) |
|---|---|---|---|---|
| Xe cứu hộ kéo xe khác | Có cả hai | `truck` + lớp của xe bị kéo | Ít | Hai box riêng (R-07) |
| Người đẩy xe nôi, xe lăn, xe đẩy siêu thị | Chỉ người | `person`; xe không có lớp | Ít | `person`; xe không gán (R-15) |
| Xe đạp điện, xe máy điện khó phân biệt | Có | `bicycle` hoặc `motorcycle` | Có | Dáng xe đạp có bàn đạp → `bicycle`; dáng scooter → `motorcycle`; khó phân biệt → `can_review` (R-04, R-14) |
| Xe máy chở hàng cồng kềnh che xe | Có | `motorcycle`, `person` | Có | Box ôm thân xe, không gồm hàng; `occluded_level` theo phần xe bị che (R-08, R-10) |
| Vật rất nhỏ hoặc rất mờ sát ngưỡng | Có thể | Theo lớp | Có | Cao < 20 px → `ignore_region`; sát ngưỡng và lưỡng lự: gán + `can_review` (R-12, R-14) |
| Phát hiện box thừa, hai box cho một vật | - | - | Có (lỗi gán) | Xóa box thừa, giữ một box ôm sát phần nhìn thấy (R-01) |


### 4.4 Nhóm D: ca calibration vòng 1 (**chờ adjudicate**)

Các ca dưới đây lấy từ `calibration_candidates.csv`. Số box chưa ghép chỉ cho biết hai người gán khác nhau, **không chứng minh bên nào bỏ sót hay vẽ thừa**: reviewer phải đối chiếu ảnh gốc và box trước khi chốt. Cột (4) ghi quy tắc áp dụng; **kết luận cho từng ca cụ thể** điền vào `calibration_decisions.csv` rồi cập nhật lại đây.

| Ca | Bất đồng | (1) Cần gán? | (2) Lớp có sẵn? | (3) Hay gặp? | (4) Quy tắc áp dụng và kết luận |
|---|---|---|---|---|---|
| C060 | `car` / `van` | Có | `car` hoặc `van` | 8 bất đồng lớp trên 429 ca | Bảng 2.2.1 (car / van). Kết luận ca: **chờ adjudicate** |
| C097 | `truck` / `car` | Có | `truck` hoặc `car` | như trên | Bảng 2.2.1 (truck / car), R-04. Kết luận ca: **chờ adjudicate** |
| C253 | `bicycle` / `cycle_rickshaw` | Có | `bicycle` hoặc `cycle_rickshaw` | như trên | Bảng 2.2.1 (số bánh, ghế khách). Kết luận ca: **chờ adjudicate** |
| C389 | `truck` / `tuk_tuk` | Có | `truck` hoặc `tuk_tuk` | như trên | Bảng 2.2.1 (số bánh, động cơ). Kết luận ca: **chờ adjudicate** |
| C119, C144, C264 | `motorcycle` / `person`; `car` / `person` | Có (người và xe đều là vật thật) | `person` và `motorcycle` hoặc `car`, **hai box riêng** | Hay gặp (xe máy ở nhiều ảnh) | R-05: tách bạch box người và box xe. Kết luận ca: **chờ adjudicate** |
| C044, C427 | `truncated` | Có | Theo lớp của vật | 14 dòng bất đồng `truncated` | R-11: `yes` chỉ khi khung ảnh cắt cụt vật. Kết luận ca: **chờ adjudicate** |
| `img_003`, `img_006`, `img_008`, `img_032` | Cảnh đông: nhiều box chỉ có ở một người gán (31, 31, 28, 31 ca) | Có: từng người, từng xe tách được | Theo lớp; cụm không tách → `ignore_region` | 4 ảnh, 121 ca | R-12, R-16: rà hai lượt, tách trước, bỏ qua sau. Ví dụ box đã adjudicate: **chờ adjudicate** |
| `img_006`, `img_008`, `img_010` | Các box `can_review` | Tùy ca | Tùy ca | 12 box `can_review` | R-14: ghi lý do có cấu trúc; reviewer chốt từng box. **Chờ adjudicate** |


### 4.4.1. Nguyên tắc chốt các ca calibration

`calibration_candidates.csv` liệt kê các khác biệt do đối chiếu tự động phát hiện. Đây là **danh sách ca cần xem**, không phải kết luận rằng box, lớp hoặc thuộc tính của người gán nào đúng. Reviewer phải mở ảnh gốc và đối chiếu các box trước khi ghi quyết định vào `calibration_decisions.csv`.

Trong 429 ca, các nhóm được phát hiện gồm: 276 ca chỉ có box ở một người gán, 66 ca IoU thấp, 66 ca bất đồng thuộc tính, 8 ca bất đồng lớp, 12 ca `can_review` và 1 ca bất đồng `ignore_region`. Các nhóm có thể cùng liên quan đến một vấn đề trong ảnh; không suy ra kết luận chỉ từ số liệu tự động.

| Nhóm bất đồng | Cách chốt sau khi xem ảnh | Quy tắc tham chiếu |
|---|---|---|
| Box chỉ có ở một người gán (`bo_sot_thua`) | Xác định vật có thuộc taxonomy và nhìn thấy đủ rõ để gán không. Nếu có, chốt box/lớp cần giữ hoặc bổ sung; nếu không, ghi rõ lý do loại box. Ghi chú “bỏ sót hoặc vẽ thừa” tự nó chưa phân biệt được hai trường hợp. | R-01; tùy ca R-02, R-03, R-04, R-05, R-12, R-15 hoặc R-16 |
| IoU thấp (`iou_thap`) | So ảnh với cả hai box và chốt/chỉnh box ôm sát phần nhìn thấy. IoU thấp chỉ báo hiệu cần kiểm tra, không cho biết box nào đúng. | R-09; nếu có che khuất, xem thêm R-10 |
| Bất đồng thuộc tính (`sai_thuoc_tinh`) | Chốt từng thuộc tính theo ảnh. `occluded_level` chỉ phản ánh phần bị vật khác trong cảnh che; `truncated = yes` chỉ khi vật thực sự bị khung ảnh cắt. Box chạm biên không tự động đồng nghĩa với `truncated = yes`. | R-10, R-11 |
| Bất đồng lớp (`sai_lop`) | Phân lớp theo cấu tạo và bảng phân biệt lớp; không chọn lớp chỉ dựa trên tên do một người gán đưa ra. Với người và phương tiện, xác định có một hay hai vật riêng biệt trong ảnh. | R-04; R-05 với người/phương tiện |
| Ca được gắn cờ `can_review` | Reviewer chốt theo ảnh và ghi lý do cụ thể. Không để lý do chung chung như “ca mơ hồ”; dùng mẫu `NHOM: mô tả ngắn; vùng cần xem`. | R-14 |
| Bất đồng `ignore_region` | Rà ảnh lượt hai. Chỉ dùng `ignore_region` cho cụm vật thật nhỏ, quá dày hoặc không thể tách đáng tin cậy; vùng phải bao gọn cụm và không che vật rõ đã gán. | R-12, R-16 |

Với các ca bất đồng lớp, các cặp cần kiểm tra gồm: C060 (`car`/`van`), C097 (`truck`/`car`), C119 (`motorcycle`/`person`), C144 (`person`/`motorcycle`), C253 (`bicycle`/`cycle_rickshaw`), C264 (`car`/`person`), C322 (`person`/`car`) và C389 (`truck`/`tuk_tuk`). Danh sách này nêu ca cần xem, không thay thế kết luận của reviewer.

Khi điền `calibration_decisions.csv`:

- `quyet_dinh`: ghi kết quả cụ thể sau khi xem ảnh — box nào được giữ, thêm, xóa hoặc chỉnh; lớp và thuộc tính cuối cùng nếu có.
- `quy_tac_ghi_vao_guideline`: ghi mã R-xx áp dụng. Nếu quy tắc hiện tại chưa đủ, nêu nội dung cần làm rõ thay vì tự tạo kết luận cho ca.
- `phien_ban_guideline`: ghi phiên bản quy tắc được áp dụng; dùng `v1.1` khi quyết định dựa trên guideline v1.1.
- `phieu_sua`: ghi sửa đổi annotation cần thực hiện. Với quyết định thay đổi quy tắc chung, ghi rõ nhóm ảnh bị ảnh hưởng và phạm vi cần kiểm lại.
- `pham_vi`: dùng `anh` cho kết luận chỉ áp dụng cho ca/ảnh đó; dùng `lo` khi quyết định làm thay đổi cách gán trên nhiều ảnh. Không đánh dấu `lo` chỉ vì nhiều ca có cùng loại bất đồng.
- `nguoi_chot` và `kiem_lai`: chỉ điền tên người thật đã xem ảnh và tham gia chốt/kiểm lại. Không sao chép hàng loạt thông tin giữa các ca khi chưa kiểm chứng.

Riêng C416, ghi chú tự động hiển thị IoU `0.85` nhưng lại mô tả là thấp hơn ngưỡng `0.85`. Cần kiểm tra giá trị chưa làm tròn và cách áp dụng ngưỡng; không kết luận từ con số đã làm tròn.


---

## 5. QC và phiên bản

### 5.1 Ngưỡng QC (từ `taxonomy.json` mục `rules`, **không được sửa**)

| Chỉ số | Cách tính | Ngưỡng |
|---|---|---:|
| Độ phủ lượt gán | Lượt gán hợp lệ / tổng lượt theo `assignment.csv` | **100%** (ví dụ 80/80) |
| Ghép cặp box | Ghép 1–1 không phân biệt lớp | IoU ≥ **0.50** |
| IAA: IoU trung bình | Chỉ trên các cặp ghép 1–1 | ≥ **0.85** |
| Đồng thuận lớp | Cùng lớp trên các cặp ghép | ≥ **95%** |
| IoU từng box: cao ≥ 50 px / nhỏ hơn | Box cao ≥ 50 px / box cao < 50 px (và ≥ 20 px) | ≥ **0.85** / ≥ **0.70** |
| Chiều cao box tối thiểu | Thấp hơn xử lý theo R-12 | **20 px** |
| Tỷ lệ ảnh có lỗi (duyệt toàn bộ 40 ảnh) | Số ảnh có ít nhất một lỗi / số ảnh đã duyệt | ≤ **2%** (0/40) |
| Số người gán mỗi ảnh (K) | Nhóm 2–4 người / nhóm 5 người | **2 / 3** |

### 5.2 Quy trình và quyết định
1. **Gán độc lập**, không xem nhãn người khác, không hỏi nhau khi gán. Ca lưỡng lự dùng `can_review` hoặc `image_issues.csv`.
2. **So sánh chéo** bằng `qc_compare.py`. IAA cao chưa đủ: hai người có thể **cùng sai** (ví dụ cùng vẽ `person` lên tượng), nên phải đối chiếu thêm ca tham chiếu của coach.
3. **Calibration:** adjudicator (reviewer hoặc người nhóm chỉ định) chốt từng ca bất đồng, ghi `calibration_decisions.csv`: `case_id`, ảnh / vùng, hai cách gán, mã `R-xx`, quyết định, người chốt, phiên bản guideline, kết quả kiểm lại, phạm vi (`anh` hoặc `lo`).
4. **QC toàn lô 40/40 ảnh**, người duyệt khác người sửa, điền `qc_review.csv`, chạy `qc_report.py`.

| Quyết định | Điều kiện |
|---|---|
| **Release** | Đủ 100% độ phủ, IAA đạt, ≤ 2% ảnh lỗi (0/40), không có lỗi lặp lại, không còn `can_review` hay issue chưa xử lý, reviewer xác nhận. |
| **Rework** | > 2% ảnh lỗi, hoặc lỗi lặp lại (≥ 3 lỗi và ≥ 50% tổng lỗi, cùng một mã `R-xx`). Sửa guideline / nhãn trong phạm vi ảnh hưởng, mở `--round 2`, **giữ nguyên** thư mục `round-01`. |
| **Hold** | Thiếu lượt gán hoặc bằng chứng, taxonomy chưa thống nhất, còn issue chưa giải quyết, hoặc chỉ là vòng demo lấy mẫu (`--n`). |

### 5.3 Quy tắc phiên bản
- Mọi thay đổi **làm đổi nhãn** thì tăng phiên bản guideline.
- **`editorial`:** chỉ sửa câu chữ, giữ nguyên cách chọn vật, lớp, box, thuộc tính.
- **`semantic`:** đổi cách chọn vật, phân lớp, ranh giới box hoặc thuộc tính → ít nhất một dòng `pham_vi = lo` và **kiểm lại cả 40 ảnh**.
- Khi đổi lớp hoặc thuộc tính: tăng `version` trong `taxonomy.json`, **giữ nguyên mục `rules`**, chạy `make_cvat_labels.py`, rồi kiểm cấu hình CVAT.

### 5.4 Điều kiện khóa v1.0 (cả nhóm xác nhận trước Pha 3)
- [ ] Taxonomy mở rộng thành **10 lớp + `ignore_region`**: `person`, `car`, `truck`, `van`, `bus`, `motorcycle`, `bicycle`, `tuk_tuk`, `cycle_rickshaw`, `handcart`. Cập nhật `submission/taxonomy.json` (`"version": "v1.0"`), chạy `python scripts/make_cvat_labels.py`, dán `cvat/labels_generated.json` vào CVAT.
- [ ] Nhóm xác nhận các điểm trong Phụ lục A đánh dấu **(cần xác nhận)**.
- [ ] Mở ảnh xác minh các ca Nhóm B (đặc biệt `img_009`, `img_029`) và bổ sung ca mới nếu có.
- [ ] Mọi thành viên đọc lại Mục 3.1 và làm thử trên một ảnh có người đi xe máy: cùng số box, cùng lớp.

### 5.5 Lịch sử phiên bản

| Phiên bản | Thay đổi | Quy tắc / ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu: 5 lớp, thuộc tính định tính, chưa có ca khó. | - |
| **v1.0** | Hợp nhất 3 bản cá nhân. Mở rộng lớp (`bus`, `bicycle`, `tuk_tuk`, `cycle_rickshaw`, `handcart`); định lượng thuộc tính; R-01…R-15; ca khó nhóm A, B, C theo 4 câu hỏi; chốt các điểm bất đồng ở Phụ lục A. | R-01 … R-15; `img_002`, `img_004`, `img_011`, `img_016`, `img_019`, `img_020`, `img_023`, `img_024`, `img_035` … |
| **v1.1** | Hiệu chỉnh sau calibration vòng 1 (thay đổi semantic, taxonomy giữ v1.0): thêm bảng phân biệt cặp dễ nhầm (2.2.1), cách ước lượng `occluded_level` (2.4), làm rõ tách người / xe (R-05), `truncated` (R-11), cảnh đông (R-12, R-16), lý do `can_review` có cấu trúc (R-14), quy trình gán hai lượt (3.3), ca calibration (4.4). | R-04, R-05, R-10, R-11, R-12, R-14, R-16; Phụ lục B |


### 5.6 Kết quả IAA và calibration vòng 1 (tham khảo)
- Tổng **429 ca trên 36 ảnh**: 276 box chỉ có ở một người gán, 66 box IoU thấp, 66 bất đồng thuộc tính, 8 bất đồng lớp, 12 box `can_review`, 1 bất đồng vùng `ignore_region`.
- Cảnh đông thiếu nhãn nhiều nhất: `img_003`, `img_006`, `img_032` (31 ca mỗi ảnh), `img_008` (28 ca).
- Cặp 301|303: 148 và 13 box chưa ghép, **IoU trung bình 0.824**, dưới ngưỡng 0.85.
- Đồng thuận `occluded_level` chỉ **68.7%** (57 dòng bất đồng thuộc tính có nhắc mức che); 14 dòng bất đồng `truncated`.
- Các hướng sửa trong v1.1 là **hướng xử lý**, chưa phải quyết định đã adjudicate; không đổi taxonomy chỉ dựa trên các bất đồng này.

### 5.7 Điều kiện khóa v1.1 (cả nhóm xác nhận)
- [ ] Reviewer đối chiếu **ảnh gốc** cho các ca ở Mục 4.4, điền `calibration_decisions.csv` (cột quyết định, `quy_tac_ghi_vao_guideline`, người chốt, kết quả kiểm lại, `pham_vi`).
- [ ] Mã quy tắc trong `calibration_decisions.csv` khớp mã `R-xx` trong guideline này; nội dung quy tắc dán nguyên văn.
- [ ] Thay các ô **(chờ adjudicate)** ở 2.2.1, 2.4 và 4.4 bằng ca đã chốt (kèm mã ảnh minh họa).
- [ ] `guideline_change.json` giữ `kind = "semantic"` và cập nhật `summary` cho khớp Phụ lục B.
- [ ] Có ít nhất một dòng `pham_vi = lo`; kiểm lại **cả 40 ảnh** theo v1.1.
- [ ] Chạy `prepare_calibration.py`, ghi lại đường dẫn có hash, export COCO 1.0 vào đúng đường dẫn đó, rồi chạy lại `build_final.py`.

---

## Phụ lục A. Nhật ký thống nhất (các điểm khác nhau giữa 3 bản cá nhân)

| # | Chủ đề | Bản gốc | Bản 301 | Bản 303 | **Quyết định thống nhất** | Lý do |
|---|---|---|---|---|---|---|
| 1 | Taxonomy | 9 lớp (thêm `bus`, `tuk_tuk`, `bicycle`) | 5 lớp; bus, xe đạp, tuk-tuk, xích lô **không gán** | 10 lớp (thêm `auto_rickshaw`, `cycle_rickshaw`, `handcart`) | **10 lớp** (xem 2.1); tên `tuk_tuk` | Không gán phương tiện sẽ dạy mô hình coi chúng là nền, nguy hiểm cho xe tự lái. Tên `tuk_tuk` theo mẫu của lab. **(cần xác nhận)** |
| 2 | Xích lô | Gộp vào `bicycle` | Không gán xe, chỉ gán người | Lớp riêng `cycle_rickshaw` | **Lớp riêng `cycle_rickshaw`** | Ba bánh, kích thước và hình dáng khác xe đạp; xuất hiện ≥ 5 ảnh |
| 3 | `handcart` | Không có | Không gán xe đẩy | Có lớp | **Có** | Vật cản di chuyển trên đường; chỉ 2 ảnh nên **(cần xác nhận)** |
| 4 | Bán tải | `truck` | `car` | `truck` | **`truck`** | Đa số; cabin tách thùng là dấu hiệu đo được; tài liệu lab cũng tranh luận `van` / `truck`, không phải `car` |
| 5 | Xe chở xe (`img_002`) | 1 box `truck`, không gán xe con | `truck` + `car` riêng | `truck` + `car` riêng | **`truck` + `car` riêng** (R-07) | Đa số; "mỗi vật thật một box"; xe trên sàn vẫn trông như xe thường. **(cần xác nhận)** |
| 6 | Người trong xe | Không gán nếu trong khung / mái | Không gán sau kính; gán nếu nhìn rõ | Gán nếu thấy trực tiếp | **Tiêu chí "thấy trực tiếp, không qua kính"** (R-05) | Đa số; dễ kiểm hơn "khung / mái" |
| 7 | Xe chủ | Không gán | Không đề cập | Gán nếu thấy trực tiếp | **Không gán** (R-03) | Tài liệu lab liệt kê "không gán xe chủ" là quy tắc hay dùng; xe chủ không phải đối tượng cần tránh. **(cần xác nhận)** |
| 8 | Ngưỡng `occluded_level` | 0 / 1–30 / 31–70 / > 70 | 0–10 / 10–40 / 40–80 / > 80 | 0–10 / 10–30 / 30–60 / > 60 | **0–10 / > 10–30 / > 30–70 / > 70** | Gần ngưỡng ví dụ 30 và 70 của lab; mốc 0–10% đồng nhất của 301 và 303 |
| 9 | Mức `3` | Chỉ khi vẫn nhận ra chắc chắn | Dùng cả cho "chỉ đoán bằng ngữ cảnh" | Không dùng để biểu thị "không rõ" | **Không dùng 3 để biểu thị "không rõ"** | Đa số; tránh gán vật không nhận ra |
| 10 | `truncated` | Từ ~5% trở lên | Bất kỳ phần nào | Bất kỳ phần nào | **Bất kỳ phần nào** | Đơn giản, không ước lượng % |
| 11 | Ngưỡng vật nhỏ | < 20 px → `ignore_region` | < 10×10 px thì bỏ, không ghi | < 20 px → `ignore_region` | **Cao < 20 px → `ignore_region`** | 20 px là ngưỡng bất biến của lab (`rules`) |
| 12 | Hàng hóa trong box | Không tính | Không tính | Tính nếu là phần ngoài cùng | **Không tính** (ngoại lệ car carrier) | Đa số; ví dụ mẫu `img_035` của lab loại hàng quá khổ khỏi box |
| 13 | Mascot, nhân vật | Không gán | Không đề cập | Gán `person` nếu xác nhận có người | **Gán nếu thấy rõ cơ thể người; không thì không gán** + `image_issues.csv` (R-05d, R-02) | Hợp nhất bản gốc và 303 |
| 14 | `ignore_region` | Vật thật quá nhỏ / dày | Thêm mờ, tối, lóa, vật ngoài lớp | Chỉ lõi không tách; vẫn gán người ở rìa | **Hợp nhất** (R-12, R-13) | Giữ ràng buộc "không dùng làm đường tắt" của cả hai |
| 15 | Xe điêu khắc ở `img_011` | Không gán | Không gán vật thể mô phỏng | Gán `motorcycle` | **Mặc định không gán** + `image_issues.csv` chờ adjudicator | Ghép từ phụ tùng, đặt khu trưng bày, không người lái (R-06). **(cần xác nhận)** |
| 16 | Ghi chú cá nhân | - | - | **Có họ tên người viết trong file** | **Bỏ họ tên khỏi file** | Quy ước của lab: chỉ dùng mã 3 chữ số trong file nộp |
| 17 | Cấu trúc ca khó | 4 câu hỏi | Bảng quyết định nhanh | Bảng 5 cột, không có 4 câu hỏi | **4 câu hỏi cho mọi ca** | Yêu cầu của bài |

---

## Phụ lục B. Nhật ký calibration v1.0 → v1.1

Số liệu lấy từ `calibration_candidates.csv`, `iaa_summary.md`, `image_scores.csv`. Phạm vi `lo` nghĩa là phải kiểm lại cả 40 ảnh (thay đổi semantic).

| # | Vấn đề (số liệu) | Thay đổi trong v1.1 | Quy tắc / mục | Phạm vi | Trạng thái |
|---|---|---|---|---|---|
| 1 | Cảnh đông thiếu nhãn: `img_003`, `img_006`, `img_032` (31 ca), `img_008` (28 ca); 301\|303 có 148 / 13 box chưa ghép, IoU TB 0.824 | Quy trình gán hai lượt; chỉ dùng `ignore_region` sau lượt 2; ghi số vật đã gán trong cụm | R-12, R-16, 3.3 | `lo` | Đã soạn; ví dụ box adjudicate: chờ reviewer |
| 2 | 8 bất đồng lớp (C060, C097, C119, C253, C264, C389...) | Bảng phân biệt cặp dễ nhầm theo số bánh, động cơ, cấu trúc thân; hàng chở không đổi lớp | R-04, 2.2.1 | `lo` | Đã soạn; kết luận từng ca: chờ adjudicate |
| 3 | Nhầm người / xe (C119, C144, C264; `can_review` ở `img_006`, `img_008`, `img_010`) | Tách bạch box `person` và box xe, người lái không thay thế box xe | R-05 | `lo` | Đã soạn; chờ adjudicate |
| 4 | Đồng thuận `occluded_level` 68.7%; 57 dòng bất đồng thuộc tính | Cách ước lượng và bảng mô tả theo mức; phân biệt che khuất với mờ, thiếu sáng, ngoài khung; giữ nguyên ngưỡng | R-10, 2.4 | `lo` | Đã soạn; ảnh tham chiếu: chờ adjudicate |
| 5 | 14 dòng bất đồng `truncated` (C044, C427) | `truncated = yes` chỉ khi vật bị khung ảnh cắt cụt; box chạm biên không đồng nghĩa `truncated` | R-11, 2.3 | `lo` | Đã soạn; chờ adjudicate |
| 6 | Lý do `can_review` chung chung ("ca mơ hồ") | Bắt buộc ghi lý do theo mẫu `NHOM: mô tả; vùng` | R-14 | Quy trình, không đổi nhãn | Đã soạn |
| 7 | 1 bất đồng vùng `ignore_region` | Siết điều kiện dùng `ignore_region` | R-12, R-16 | `lo` | Đã soạn; chờ adjudicate |

