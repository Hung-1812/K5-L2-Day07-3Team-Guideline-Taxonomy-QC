# Guideline gán nhãn bounding box: ảnh giao thông đường phố

- **Phiên bản:** v1.0 (bản nháp để nhóm chốt ở Pha 2)
- **Loại nhãn:** bounding box (hộp chữ nhật) trên ảnh đường phố
- **Công cụ:** CVAT, xuất COCO 1.0
- **Khách hàng / mục đích:** nhận diện xe cộ trên đường phố để xe tự lái tránh va chạm
- **Quy ước:** mỗi quy tắc có mã `R-xx` để dẫn chiếu trong phiếu quyết định và phiếu QC. Ví dụ ảnh dùng tên `img_0xx.jpg`.

---

## 1. Mục tiêu

### 1.1 Mô hình dùng nhãn
Nhãn huấn luyện mô hình **phát hiện vật thể 2D** (object detection). Mô hình xuất ra *hộp + lớp* cho mỗi **người tham gia giao thông** và **phương tiện thật** trong ảnh. Kết quả được hệ thống xe tự lái dùng để **tránh va chạm**.

### 1.2 Hệ quả cho việc gán nhãn
| Nguyên tắc | Ý nghĩa khi gán |
|---|---|
| **Không bỏ sót vật thật** | Bỏ sót nguy hiểm hơn vẽ thừa một chút. Vật thật thì gán, kể cả xe đang đỗ. |
| **Không tạo "vật ma"** | Tượng, hình in, phản chiếu mà gán thì mô hình sẽ phanh gấp vì vật không tồn tại. |
| **Người và xe nhỏ hai bánh là ưu tiên** | Đây là nhóm dễ bị va chạm nhất, nên phải tách thành lớp riêng, không gộp. |
| **Nhất quán hơn cầu toàn** | Hai người gán cùng một cách thì mô hình học được. Mỗi người một kiểu thì mô hình không học được. |

### 1.3 Phép thử chọn *lớp* hay *thuộc tính*
> Mô hình phát hiện 2D có phải **đoán ra** thông tin này khi suy luận không?
> - **Có** → đó là **lớp** (ví dụ: bus, tuk_tuk, motorcycle).
> - **Không**, chỉ dùng để lọc mẫu khó hoặc kiểm QC → đó là **thuộc tính** (ví dụ: occluded_level, truncated, can_review).

### 1.4 Ngoài phạm vi
Không gán: biển báo, đèn giao thông, vạch kẻ đường, động vật, vật cản tĩnh (cột, rào). Nhãn này **không** dùng cho ước lượng 3D hay theo dõi (tracking).

---

## 2. Lớp và thuộc tính

### 2.1 Lớp (mỗi vật thuộc đúng **một** lớp)

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ (trông giống nhưng KHÔNG thuộc lớp này) |
|---|---|---|---|
| `car` | Ô tô con chở người, khoang kín: sedan, hatchback, SUV, taxi; kể cả xe cổ và xe diễu hành cách điệu nếu đủ 3 dấu hiệu của R-15. | Sedan trên đường (`img_023`); xe diễu hành (`img_019`); xe cổ trưng bày (`img_024`). | Xe mô hình trên nóc và trong tủ kính (`img_024`, `img_038`); bán tải (→ `truck`); xe thùng kín giao hàng (→ `van`). |
| `truck` | Xe tải và xe bán tải: **cabin tách rời thùng hàng** (thùng hở, có bạt, có nắp, hoặc sàn chở xe). | Xe đầu kéo chở ô tô (`img_002`); bán tải nắp thùng cao (`img_016`, `img_037`). | Bia mộ tạc hình xe bán tải (`img_027`); xe tải đồ chơi. |
| `van` | Xe thùng kín cỡ nhỏ đến vừa, thân **liền khối** (không có vách tách cabin và thùng), từ 16 chỗ trở xuống. | Xe van trắng (`img_011`, `img_016`); van giao hàng; minivan. | Bán tải có nắp thùng (cabin vẫn tách → `truck`); xe khách lớn (→ `bus`). |
| `bus` | Xe buýt, xe khách cỡ lớn: dài hơn rõ rệt so với van, hàng cửa sổ liên tiếp. | `img_009`, `img_031`, `img_037`, `img_040`. | Xe 16 chỗ trở xuống (→ `van`); xe buýt đồ chơi (R-02). |
| `tuk_tuk` | Xe **ba bánh có động cơ**, có khoang hoặc mái. | `img_004`, `img_016`, `img_035`. | Xích lô đạp (→ `bicycle`); xe máy ba bánh không khoang (xét theo R-05). |
| `motorcycle` | Xe **hai bánh có động cơ**: xe máy, xe tay ga, mô tô, xe máy điện. **Box chỉ ôm xe**, người ngồi trên là `person` riêng (R-06a). | Xe máy có người lái (`img_016`, `img_023`). | Xe đạp (→ `bicycle`); xe điêu khắc ghép từ phụ tùng (`img_011`). |
| `bicycle` *(đề xuất lớp mới)* | Xe **không động cơ** chở người: xe đạp, xích lô đạp. Box chỉ ôm xe, người đi là `person` riêng (R-06a). | Xích lô (`img_023`); `img_006`, `img_029`, `img_039`. | Xe đạp mô hình hoặc tượng (R-02). |
| `person` | **Người thật** đi bộ, đứng, ngồi bên ngoài phương tiện, hoặc điều khiển / ngồi trên xe hai bánh và xích lô (R-06). | Khán giả (`img_019`); người lái xe máy (`img_016`, `img_023`); người đạp xích lô (`img_023`). | Tượng (`img_017`); đầu điêu khắc (`img_011`); ma-nơ-canh, ảnh in (`img_033`); nhân vật mascot (`img_019`); phản chiếu (`img_020`); người ngồi trong ô tô, tuk-tuk, van (R-06b). |
| `ignore_region` | Vùng không gán từng vật: đám đông dày, cụm vật có chiều cao **< 20 px** (R-12). | Nhóm người xa chỉ còn vài pixel. | Vùng phủ lên vật đã có box (gây lệch với người gán kia). |

> Ghi chú: `bicycle` là đề xuất của bản nháp này, vì xích lô và xe đạp là đối tượng dễ va chạm và xuất hiện lặp lại (xem `img_023`). Nếu nhóm không thêm lớp này thì phải ghi rõ cách xử lý xe đạp và xích lô ở Mục 4.

### 2.2 Thuộc tính

| Thuộc tính | Giá trị | Ngưỡng đo được | Ghi chú |
|---|---|---|---|
| `occluded_level` | `0` | Không bị che (0%). | Chỉ tính phần bị **vật khác** che, không tính phần bị mép ảnh cắt. |
| | `1` | Bị che **1–30%** diện tích vật. | |
| | `2` | Bị che **31–70%**. | |
| | `3` | Bị che **> 70%** nhưng vẫn nhận dạng **chắc chắn** lớp. | Nếu không chắc lớp: không vẽ box, ghi `image_issues.csv` (R-14). |
| `truncated` | không / có (tick) | **Có** khi vật bị mép ảnh cắt mất phần thân thấy rõ (từ khoảng 5% trở lên). | Box vẫn kéo sát tới mép ảnh (R-11). |
| `can_review` | tick / không | Tick khi bạn **không chắc** lớp, ranh giới box hoặc có ca khó chưa có quy tắc. | Không tick tùy tiện vì mỗi lần tick là một ca phải họp xem lại. |

> Lưu ý CVAT: chỉ dùng thuộc tính `occluded_level`. Cờ `occluded` mặc định của CVAT (phím `Q`) bị bỏ qua.

---

## 3. Quy tắc vẽ box

### 3.1 Cây quyết định nhanh (làm theo thứ tự)
1. Đây có phải **vật thật** đang hiện diện trong cảnh không? Nếu là tượng, mô hình, hình in hoặc phản chiếu → **không gán** (R-02, R-03).
2. Vật có cao **< 20 px** hoặc nằm trong đám đông dày? → vẽ `ignore_region` (R-12).
3. Chọn **lớp** theo cấu tạo (R-05). Chưa có lớp phù hợp thì xem Mục 4 và ghi `can_review` hoặc `image_issues.csv`.
4. **Vẽ box** ôm sát phần nhìn thấy (R-09), rồi chọn `occluded_level`, `truncated`.
5. Còn lăn tăn → tick `can_review` (R-14). **Không hỏi người gán cùng ảnh.**

### 3.2 Danh sách quy tắc

| Mã | Quy tắc |
|---|---|
| **R-01** | **Gán mọi vật thật** thuộc các lớp ở Mục 2, kể cả xe đang đỗ, xe đứng yên, xe thật trưng bày trong nhà, xe ở xa (nếu cao ≥ 20 px). |
| **R-02** | **Không gán vật không thật:** tượng, điêu khắc, bia mộ hình xe, ma-nơ-canh, đồ chơi, xe mô hình, nhân vật mascot, hình người hoặc xe in trên biển quảng cáo. Ảnh chỉ có các vật này vẫn giữ trong Task và xuất 0 annotation. **Không vẽ `ignore_region` lên các vật này** (chỉ dùng cho vật *thật* quá nhỏ hoặc quá dày). |
| **R-03** | **Không gán phản chiếu** (gương chiếu hậu, kính cửa hàng, mặt nước) và **không gán xe chủ** (phần thân xe mang camera: cửa, gương, kính chắn gió). Chỉ gán vật thật nhìn thấy trực tiếp. |
| **R-04** | **Một vật = một lớp = một box.** Không vẽ hai box cho cùng một vật; không vẽ box lồng cùng lớp lên cùng vật. |
| **R-05** | **Phân lớp theo cấu tạo, không theo công dụng:** 2 bánh có động cơ → `motorcycle`; 2 bánh không động cơ → `bicycle`; 3 bánh có động cơ có khoang → `tuk_tuk`; chở khách cỡ lớn → `bus`; cabin tách rời thùng → `truck`; thân kín liền khối, ≤ 16 chỗ → `van`; còn lại ô tô con kín → `car`. |
| **R-06** | **Người và phương tiện.** (a) *Xe hai bánh và xích lô:* vẽ box xe (`motorcycle` hoặc `bicycle`, chỉ ôm xe) **và** box `person` riêng cho từng người ngồi bên ngoài (người lái, người ngồi sau); box người ôm phần thân nhìn thấy, **được chồng lấn** box xe. (b) *Phương tiện có khung / mái / khoang* (ô tô, van, bus, truck, tuk-tuk, xe diễu hành): người bên trong, kể cả tài xế, **không gán riêng** vì đã nằm trong box phương tiện. (c) *Người đứng, đi, ngồi bên ngoài phương tiện:* gán `person`. |
| **R-07** | **Xe được chở trên phương tiện chuyên chở xe không gán riêng.** Box `truck` ôm cả cabin, rơ-moóc và các xe nằm trên sàn (ngoại lệ của R-08, vì sàn chở là một phần của xe chuyên dụng). Lý do: xe được chở di chuyển cùng xe chở, không phải đối tượng độc lập, và box lồng nhau làm lệch IoU (`img_002`). |
| **R-08** | **Hàng rời xếp vượt khung xe:** box chỉ lên tới mép trên của thùng / khung xe, **không kéo lên bao hàng** (`img_035`). |
| **R-09** | **Ôm sát phần nhìn thấy:** 4 cạnh box chạm mép ngoài của phần nhìn thấy, gồm thân xe và bánh; không gồm ăng-ten, bóng đổ, gương rời. **Không đoán** phần bị che. |
| **R-10** | **Che khuất:** vẫn gán nếu nhận dạng chắc chắn; chọn `occluded_level` theo ngưỡng Mục 2.2. Không nhận dạng chắc → không vẽ, ghi `image_issues.csv`. |
| **R-11** | **Cắt mép ảnh:** kéo box tới sát mép ảnh (không vượt biên) và đánh dấu `truncated` = có. |
| **R-12** | **Vật nhỏ và đám đông:** vật cao **< 20 px**, hoặc cụm vật chồng dày không tách được từng cái → một `ignore_region` thay cho từng box. Người hoặc xe **tách được** (nhận ra là một vật riêng, thấy rõ đầu và vai trở lên, không bị che > 70%) vẫn gán từng box. `ignore_region` **không phủ lên** box của vật đã gán. |
| **R-13** | **Mờ chuyển động / chất lượng kém:** vẫn gán nếu nhận ra lớp; box ôm vùng lõi nhìn thấy (`img_014`, `img_015`). Không chắc lớp → tick `can_review`. |
| **R-14** | **Ca không chắc:** (a) đã vẽ được box → tick `can_review`; (b) chưa vẽ được box → ghi `submission/qc/image_issues.csv` (cột `image,annotator,region,note`). Không tự đoán, không hỏi người gán cùng ảnh. |
| **R-15** | **Xe thật hay đồ chơi? Phép thử 3 dấu hiệu.** Vật có hình phương tiện chỉ được gán khi đủ cả 3: (a) kích thước xe thật, đủ chỗ cho người lái; (b) có bánh và có thể di chuyển trên đường (biển số, đèn, cửa); (c) không phải đồ trang trí cố định hoặc vật trưng bày dạng tác phẩm. Đủ 3 → gán theo lớp tương ứng (kể cả xe diễu hành, xe cổ, xe cách điệu: `img_019`, `img_024`). Thiếu một dấu hiệu → R-02. |

---

## 4. Ca khó

Mã ca = tên file ảnh. Mỗi ca trả lời đủ 4 câu hỏi: **(1) Cần gán?** · **(2) Lớp có sẵn?** · **(3) Hay gặp?** · **(4) Ghi lại quyết định kèm ảnh ví dụ**.

> Giá trị `occluded_level`, `truncated` trong các bảng là **ước lượng bằng mắt** từ ảnh. Khi gán thật, hãy đo lại trong CVAT theo ngưỡng ở Mục 2.2.

### 4.0 Tổng hợp nhanh

| Mã ca | Tình huống | Kết quả vẽ | Quy tắc |
|---|---|---|---|
| `img_002` | Xe đầu kéo chở 8 ô tô con trên hai sàn, công nhân đứng cạnh | 1 `truck` + 1 `person` | R-07, R-06 |
| `img_004` | Hai tuk-tuk taxi đỗ ban đêm, che nhau, tài xế ngồi trong khoang | 2 `tuk_tuk` | R-05, R-06, R-10 |
| `img_011` | Hội chợ: đầu điêu khắc khổng lồ, xe điêu khắc từ phế liệu, đám đông, 1 xe van bị che | Vài `person`, 1 `van`, `ignore_region` | R-02, R-10, R-12 |
| `img_016` | Phố đông: tuk-tuk, bán tải nắp thùng cao bị che, xe máy, van, ô tô, người | Nhiều box | R-05, R-06, R-10, R-11 |
| `img_017` | Tượng người cưỡi ngựa đội nón giao thông, có chim đậu | **0 box** | R-02 |
| `img_019` | Xe diễu hành hình ô tô hoạt hình, khán giả sau hàng rào | 1 `car` + nhiều `person` + `ignore_region` | R-15, R-06, R-12 |
| `img_020` | Cảnh phản chiếu trong gương chiếu hậu | **0 box** | R-03 |
| `img_023` | Phố Việt Nam: xích lô, ô tô, xe máy, hàng xe máy đỗ dày | Nhiều box + `ignore_region` | R-05, R-06, R-12 |
| `img_024` | Xe trưng bày trong nhà, mô hình xe vàng đặt trên nóc | `car` thật; không gán mô hình | R-02, R-08, R-15 |
| `img_027` | Bia mộ hình xe bán tải trong nghĩa trang | **0 box** | R-02 |
| `img_033` | Kính cửa hàng: ma-nơ-canh, ảnh người in, phản chiếu xe | **0 box** | R-02, R-03 |
| `img_038` | Tủ kính đồ chơi: xe mô hình, thú nhồi | **0 box** | R-02 |

Ảnh **0 box** vẫn phải giữ trong Task CVAT và xuất COCO với 0 annotation.

---

### img_002: Xe chở ô tô hai tầng

*Mô tả:* xe đầu kéo cabin trắng kéo rơ-moóc chở xe hai sàn, trên sàn có khoảng 8 ô tô con (khoảng 5 xe tầng trên, 3 xe tầng dưới). Một công nhân đứng cạnh đuôi rơ-moóc.

- **(1) Cần gán?** Xe đầu kéo: **có**. Các xe con được chở: **không gán riêng**, vì chúng là hàng chuyên chở và di chuyển cùng xe. Công nhân: **có**.
- **(2) Lớp có sẵn?** `truck` (cabin tách rời sàn chở). Về hình dáng, các xe con thuộc `car`, nhưng lúc này chúng là hàng hóa chứ không phải đối tượng độc lập.
- **(3) Hay gặp?** Ít (1–2 ảnh) nhưng gây **bất đồng lớn**: một người vẽ 1 box, người khác vẽ 9 box. Cần một quy tắc duy nhất.
- **(4) Quyết định:** 1 box `truck` ôm từ phần trước của cabin đến đuôi rơ-moóc, **gồm cả các xe nằm trên sàn** (R-07, ngoại lệ của R-08). 1 box `person` cho công nhân. *Phương án bị loại:* gán thêm từng `car` trên sàn, vì box lồng nhau làm lệch IoU. Nhóm cần chốt lại ở họp Pha 2 nếu có người phản đối.

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Xe đầu kéo + rơ-moóc (cả đoàn xe) | `truck` | 0 | không | không |
| Công nhân đứng cạnh đuôi rơ-moóc | `person` | 0 hoặc 1 (đo lại) | không | không |

---

### img_004: Hai tuk-tuk taxi đỗ ban đêm

*Mô tả:* hai xe ba bánh có mái, mỗi xe có một tài xế ngồi trong khoang. Xe bên phải bị xe bên trái che một phần. Một nón giao thông cam ở góc dưới trái. Ảnh chụp ban đêm.

- **(1) Cần gán?** Hai tuk-tuk: **có**. Tài xế trong khoang: **không gán riêng** (R-06b: người nằm trong khung/mái xe thuộc box phương tiện). Nón giao thông: **không** (vật cản tĩnh, ngoài phạm vi Mục 1.4).
- **(2) Lớp có sẵn?** **Không hợp**: `car` (4 bánh, kín), `motorcycle` (2 bánh). v0 không có lớp phù hợp.
- **(3) Hay gặp?** **Có**: tuk-tuk xuất hiện ít nhất ở `img_004`, `img_016`, `img_035` → **đề xuất lớp mới `tuk_tuk`**. Nón giao thông chỉ thấy ở 2 ảnh (`img_004`, `img_017`), chưa đủ để thêm lớp.
- **(4) Quyết định:** 2 box `tuk_tuk`, mỗi box ôm từ đầu xe đến đuôi xe thấy được, gồm mái và bánh. Xe phải bị che bởi xe trái nên chọn `occluded_level` theo %. Không vẽ gì cho tài xế và nón giao thông. R-05, R-06, R-10.

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Tuk-tuk bên trái (ở trước) | `tuk_tuk` | 0 | không | không |
| Tuk-tuk bên phải (bị che một phần) | `tuk_tuk` | 1 (đo lại) | không | không |

---

### img_011: Hội chợ có điêu khắc, đám đông và xe van bị che

*Mô tả:* tiền cảnh bên phải là đầu điêu khắc khổng lồ (mặt giống nhân vật người ngoài hành tinh). Giữa ảnh là tác phẩm điêu khắc hình xe máy ghép từ phụ tùng kim loại. Nền có người dự hội chợ (vài người gần, nhiều người xa) và một xe van trắng bị đầu tượng che một phần.

- **(1) Cần gán?** Người thật: **có**. Xe van thật: **có**. Đầu điêu khắc và xe điêu khắc: **không** (vật không thật, R-02, R-15). Đám đông xa, nhỏ: không vẽ từng người, dùng `ignore_region`.
- **(2) Lớp có sẵn?** Đầu điêu khắc có mặt như người nhưng **không phải `person`**. Xe điêu khắc có bánh nhưng ghép từ phế liệu, đặt ở khu trưng bày, không có người lái → thiếu dấu hiệu (c) của R-15.
- **(3) Hay gặp?** Tượng và điêu khắc xuất hiện ở ít nhất 5 ảnh (`img_011`, `img_017`, `img_027`…) → đã đủ thường xuyên để thành một quy tắc cố định (R-02).
- **(4) Quyết định:** vẽ `person` cho từng người đứng gần, thấy rõ; vẽ `van` cho xe trắng (phần bị đầu tượng che chọn `occluded_level` theo %); đám đông xa vẽ 1–2 `ignore_region`. Không vẽ gì cho điêu khắc. Nếu nhóm không chắc xe điêu khắc có phải xe thật hay không, ghi vào `image_issues.csv` (R-14).

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Người gần (nữ áo đen, nam áo trắng đeo máy ảnh) | `person` | 0 (đo lại) | không | không |
| Người cỡ trung ở giữa ảnh (nếu cao ≥ 20 px và tách được) | `person` | 0 hoặc 1 | không | không |
| Xe van trắng bị đầu tượng che | `van` | 1 hoặc 2 (đo lại) | không | không |
| Đám đông xa, chồng dày | `ignore_region` | - | - | - |

---

### img_016: Phố đông, nhiều loại phương tiện và che khuất

*Mô tả:* phố có tuk-tuk bên trái (xanh–vàng) và bên phải (xanh lá), xe máy tay ga xanh có người lái áo xanh, xe van trắng có biển taxi trên nóc, ô tô đen phía sau, xe bán tải màu kem nắp thùng cao bị tuk-tuk trái che gần hết cabin, một phụ nữ áo đỏ ở tiền cảnh bên phải và một phần đầu người ở góc dưới trái.

- **(1) Cần gán?** Tất cả phương tiện và người ngoài: **có**. Tài xế trong tuk-tuk: **không** (R-06b).
- **(2) Lớp có sẵn?** Bán tải nắp thùng cao: không thấy cabin nên dễ nhầm `van`. Dùng dấu hiệu còn thấy được (cửa thùng sau, đèn hậu, chữ trên cửa thùng, kiểu thân xe bán tải) → **`truck`** theo R-05, và tick `can_review`.
- **(3) Hay gặp?** Cả hai đều lặp lại: tuk-tuk (≥ 3 ảnh), bán tải nắp thùng cao (≥ 2 ảnh, `img_016`, `img_037`).
- **(4) Quyết định:** vẽ theo bảng dưới. Mỗi vật bị che hay bị cắt mép chọn thuộc tính theo ngưỡng (R-10, R-11). Phần đầu người ở góc dưới trái chỉ thấy một phần rất nhỏ: nếu nhận ra chắc chắn là đầu người thì vẽ `person` với `truncated` và `can_review`, nếu không thì ghi `image_issues.csv`.

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Tuk-tuk trái (xanh–vàng) | `tuk_tuk` | 0 hoặc 1 | có (sát mép trái) | không |
| Bán tải kem nắp thùng cao sau tuk-tuk trái | `truck` | 2 (đo lại) | có (nếu chạm mép trái) | **có** |
| Xe máy xanh | `motorcycle` | 0 | không | không |
| Người lái xe máy (áo xanh) | `person` | 0 hoặc 1 | không | không |
| Xe van trắng (biển taxi trên nóc) | `van` | 1 | không | không |
| Ô tô đen phía sau xe máy và van | `car` | 2 (đo lại) | không | không |
| Tuk-tuk phải (xanh lá) | `tuk_tuk` | 1 | có (sát mép phải) | không |
| Phụ nữ áo đỏ ở tiền cảnh | `person` | 0 | có (cạnh dưới, cạnh phải) | không |
| Đầu người ở góc dưới trái | `person` | 3 (đo lại) | có | **có** |

---

### img_017: Tượng cưỡi ngựa đội nón giao thông

*Mô tả:* tượng đồng người cưỡi ngựa trước cột đá, có một nón giao thông đội trên đầu tượng và một con chim đậu trên lưng ngựa.

- **(1) Cần gán?** **Không.** Tượng người và ngựa là vật không thật (R-02). Nón giao thông và chim ngoài phạm vi Mục 1.4.
- **(2) Lớp có sẵn?** `person` trông giống nhưng đây là tượng → **không dùng**. Nón giao thông và chim không có lớp.
- **(3) Hay gặp?** Có (tượng ở ≥ 4 ảnh). Rủi ro: cả hai người gán **cùng vẽ `person`** cho tượng và IAA vẫn cao, đây là kiểu lỗi "giống nhau nhưng cùng sai" (Mục 5.3).
- **(4) Quyết định:** **0 box.** R-02. Giữ ảnh trong Task, xuất COCO 0 annotation.

---

### img_019: Xe diễu hành hình ô tô hoạt hình

*Mô tả:* xe màu vàng có hình dáng cách điệu, logo "DM" ở đầu xe, có biển số, bánh xe thật. Một người đàn ông ngồi lái bên trong, bên cạnh là một nhân vật chuột trắng cỡ lớn (mô hình nhân vật). Phía sau là hàng rào chắn và đông khán giả (người lớn, trẻ em, nhân viên áo vàng). Góc dưới phải có phần đầu một người bị mờ và bị cắt mép.

- **(1) Cần gán?** Xe: **có**, vì là phương tiện thật đang di chuyển trên đường (đủ 3 dấu hiệu R-15). Khán giả thật: **có**. Người lái bên trong xe: **không gán riêng** (R-06b). Nhân vật chuột trắng: **không** (mô hình nhân vật, R-02).
- **(2) Lớp có sẵn?** `car` (xe 4 bánh, khoang kín một phần, đi trên đường). Không cần tạo lớp mới.
- **(3) Hay gặp?** Xe diễu hành, xe cách điệu ít gặp (1 ảnh), nhưng cần quy tắc rõ để không nhầm với xe đồ chơi.
- **(4) Quyết định:** 1 box `car` ôm thân xe, kể cả phần kính. Khán giả hàng đầu tách được thì vẽ từng `person` (hàng rào che chân → `occluded_level` theo %). Phần đông ở hàng sau chồng lấn dày chỉ thấy đầu → `ignore_region` (R-12). Phần đầu mờ ở góc dưới phải: vẽ `person` với `truncated` và `can_review`.

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Xe diễu hành vàng | `car` | 0 | không | không |
| Khán giả hàng đầu, thấy rõ | `person` | 1 hoặc 2 (hàng rào che) | không | không |
| Nhân viên áo vàng ở mép phải | `person` | 0 hoặc 1 | có (nếu chạm mép) | không |
| Đầu người mờ ở góc dưới phải | `person` | 0 | có | **có** |
| Khán giả hàng sau, chồng dày | `ignore_region` | - | - | - |

---

### img_020: Phản chiếu trong gương chiếu hậu

*Mô tả:* ảnh chụp qua cửa kính xe, thấy gương chiếu hậu ướt nước mưa. Trong gương có đường, một chiếc ô tô xám nhỏ ở xa và cầu vồng; thân xe đang chụp cũng hiện trong gương.

- **(1) Cần gán?** **Không.** Chiếc xe trong gương chỉ là phản chiếu (R-03). Phần thân xe, gương, cửa kính là **xe chủ** (xe mang camera), cũng không gán.
- **(2) Lớp có sẵn?** `car` trông giống nhưng không áp dụng, vì ảnh phản chiếu không phải vật thật nhìn trực tiếp.
- **(3) Hay gặp?** Ít (1 ảnh trong gương + 1 ảnh trong kính cửa hàng ở `img_033`), nhưng nếu gán sẽ tạo "vật ma" cho xe tự lái.
- **(4) Quyết định:** **0 box.** R-03. Mẹo nhận biết: vật nằm trong khung gương, hình bị cong hoặc ngược, có giọt nước trên bề mặt gương.

---

### img_023: Phố Việt Nam, xích lô và xe máy đỗ dày

*Mô tả:* ô tô sedan đen bên trái; xích lô ba bánh không động cơ ở tiền cảnh bên phải, người đạp ngồi trên yên (nhìn từ phía sau); xe máy có người lái áo nâu ở xa hơn; một người đứng cạnh ô dù bên trái; một người ngồi sát mép phải; dọc vỉa hè có hàng xe máy đỗ sát nhau; xa hơn nữa có xe máy chở người và xe khác.

- **(1) Cần gán?** Ô tô, xích lô, xe máy đang chạy, người: **có**. Xe máy đỗ dày đặc: tách được và cao ≥ 20 px thì vẽ từng box, ngược lại dùng `ignore_region`.
- **(2) Lớp có sẵn?** Xích lô là xe **không động cơ** ba bánh, không hợp `motorcycle` (có động cơ) và `tuk_tuk` (có động cơ). Gần nhất là **`bicycle`** (lớp đề xuất, định nghĩa gồm xe đạp và xích lô đạp).
- **(3) Hay gặp?** **Có**: xe đạp và xích lô xuất hiện ít nhất ở `img_006`, `img_023`, `img_029`, `img_039` (4 ảnh) → xác nhận nên có lớp riêng. Xe dễ va chạm với xe tự lái nên không gộp vào `motorcycle`.
- **(4) Quyết định:** box `bicycle` chỉ ôm xích lô; người đạp xích lô là `person` riêng (R-06a). Xe máy có người lái: 1 `motorcycle` + 1 `person` (R-06a).

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Ô tô sedan đen | `car` | 0 | không | không |
| Xích lô | `bicycle` | 0 (đo lại) | không | không |
| Người đạp xích lô (áo xanh nhạt) | `person` | 0 hoặc 1 | không | không |
| Xe máy phía xa | `motorcycle` | 0 | không | không |
| Người lái xe máy (áo nâu, mũ bảo hiểm) | `person` | 0 | không | không |
| Người đứng cạnh ô dù (áo trắng) | `person` | 1 (đo lại) | không | không |
| Người ngồi sát mép phải (áo xanh) | `person` | 1 | có | không |
| Hàng xe máy đỗ sát nhau, nhỏ | `ignore_region` (hoặc từng `motorcycle` nếu cao ≥ 20 px và tách được) | - | - | - |

---

### img_024: Xe trưng bày trong nhà, mô hình xe trên nóc

*Mô tả:* ô tô đen (kiểu AMC Pacer) nhìn từ phía sau, bên trong gian trưng bày xe cổ. Trên nóc xe đặt một chiếc xe màu vàng cỡ nhỏ (mô hình). Xung quanh là nhiều xe cổ khác (kem gỗ, nâu, cam, xanh bạc), hàng xa dày đặc, có biển Texaco.

- **(1) Cần gán?** Xe đen: **có**, đây là xe thật kích thước thật (R-01, R-15). Các xe cổ trong nền: **có** nếu thấy rõ và cao ≥ 20 px. Mô hình xe vàng trên nóc: **không** (R-02).
- **(2) Lớp có sẵn?** `car` cho các xe cổ. Mô hình xe vàng trông giống `car` nhưng **không đủ dấu hiệu (a)** của R-15 (cỡ nhỏ, không chở người lái được, đặt trên giá nóc).
- **(3) Hay gặp?** Xe đặt trên xe khác và xe trưng bày trong nhà đều ít gặp (1–2 ảnh), nhưng là ví dụ điển hình của việc "xe thật" so với "vật hình xe".
- **(4) Quyết định:** box `car` cho xe đen **chỉ ôm thân xe tới nóc và giá nóc**, không kéo lên mô hình xe vàng (R-08). Xe cổ ở nền: vẽ từng box nếu tách rõ; hàng xa chồng dày hoặc cao < 20 px → `ignore_region` (R-12). *Lưu ý:* ảnh trong nhà, nếu cả nhóm quyết định ngoài phạm vi thì đổi thành 0 box; cần chốt ở Pha 2.

| Vật | Lớp | occluded | truncated | can_review |
|---|---|---|---|---|
| Xe đen AMC Pacer (không gồm mô hình) | `car` | 0 | không | không |
| Xe cổ kem gỗ (bên trái) | `car` | 1 (đo lại) | có (nếu sát mép) | không |
| Xe cổ màu nâu (giữa nền) | `car` | 2 hoặc 3 | không | không |
| Xe cam (bên phải) | `car` | 1 | không | không |
| Xe cam ở góc dưới trái (chỉ thấy một phần) | `car` | 0 | có | không |
| Hàng xe xa, chồng dày | `ignore_region` | - | - | - |

---

### img_027: Bia mộ hình xe bán tải

*Mô tả:* bia mộ bằng đá granit đen tạc hình xe bán tải cổ, đặt trên bệ trong nghĩa trang, có bảng tưởng niệm ở bên trái. Không thấy người hay xe thật nào.

- **(1) Cần gán?** **Không.** Đây là tác phẩm tạc, không phải phương tiện (R-02).
- **(2) Lớp có sẵn?** `truck` trông giống nhất, nhưng bánh xe gắn cố định vào đá, không di chuyển được (thiếu dấu hiệu (b), (c) của R-15). Đây là **phản ví dụ chuẩn** của `truck`.
- **(3) Hay gặp?** Vật tạc hình phương tiện ít gặp (1 ảnh), nhưng gây nhầm lẫn mạnh vì hình dáng rất giống xe thật.
- **(4) Quyết định:** **0 box.** R-02.

---

### img_033: Kính cửa hàng có ma-nơ-canh, ảnh in và phản chiếu

*Mô tả:* ảnh đơn sắc, chụp qua kính cửa hàng. Ở giữa là ma-nơ-canh trắng; bên trái là ảnh in người mẫu mặc đồ chấm bi trên nền xương rồng cùng bóng phản chiếu của công trình; bên phải là hình một xe trắng có gương chiếu hậu, trông như bóng phản chiếu trên mặt kính.

- **(1) Cần gán?** **Không.** Ma-nơ-canh và ảnh in người mẫu là vật không thật (R-02). Hình xe trắng trông như lớp phản chiếu trên kính (R-03).
- **(2) Lớp có sẵn?** `person` (ma-nơ-canh, ảnh in) và `van`/`truck` (xe trắng) trông giống nhưng không áp dụng.
- **(3) Hay gặp?** Ma-nơ-canh, ảnh in, phản chiếu qua kính xuất hiện ở nhiều ảnh (`img_018`, `img_020`, `img_033`, `img_038`).
- **(4) Quyết định:** **0 box.** R-02, R-03. Nếu bạn xác nhận xe trắng là xe thật đứng sau kính chứ không phải phản chiếu, tick `can_review` và ghi vào `image_issues.csv` để nhóm chốt.

---

### img_038: Tủ kính đồ chơi

*Mô tả:* tủ kính ba tầng chứa vịt cao su, nhân vật Mickey, hộp đồ chơi, xe ba bánh đồ chơi và khoảng hai chục xe mô hình đủ màu (xe thể thao, Jeep, xe cổ).

- **(1) Cần gán?** **Không.** Toàn bộ là đồ chơi và mô hình (R-02).
- **(2) Lớp có sẵn?** `car` và `motorcycle`/`bicycle` trông giống nhưng đây là xe **mô hình** (thiếu dấu hiệu (a) của R-15).
- **(3) Hay gặp?** Có (ít nhất 2 ảnh có xe mô hình/đồ chơi: `img_024`, `img_038`).
- **(4) Quyết định:** **0 box.** R-02. **Không vẽ `ignore_region`** lên tủ kính, vì `ignore_region` chỉ dành cho vật **thật** quá nhỏ hoặc quá dày; vật không thật thì không cần vùng bỏ qua.

---

## 5. QC và phiên bản

### 5.1 Quy trình
1. **Gán độc lập**: mỗi ảnh gán bởi **K = 2** người (nhóm 5 người: K = 3). Không trao đổi lúc gán; ca lưỡng lự dùng `can_review` hoặc `image_issues.csv`.
2. **So sánh chéo (IAA)** bằng `qc_compare.py`:

   | Chỉ số | Ngưỡng đạt |
   |---|---|
   | Độ phủ lượt gán | **100%** (ví dụ 80/80) |
   | IoU trung bình (cặp ghép 1–1, IoU ≥ 0.5) | **≥ 0.85** |
   | Đồng thuận lớp | **≥ 95%** |
   | IoU từng box: vật cao ≥ 50 px / nhỏ hơn | **0.85 / 0.70** |

3. **Calibration**: họp từng ca bất đồng, ghi `calibration_decisions.csv` (người chốt, quy tắc `R-xx`, phiên bản, phạm vi).
4. **QC toàn lô 40/40 ảnh**: người duyệt khác người sửa, điền `qc_review.csv`, chạy `qc_report.py`.

| Quyết định | Điều kiện |
|---|---|
| **Release** | ≤ 2% ảnh lỗi (với 40 ảnh nghĩa là 0 ảnh lỗi), không có lỗi lặp lại, đủ bằng chứng. |
| **Rework** | > 2% ảnh lỗi, hoặc lỗi lặp lại (≥ 3 lỗi và ≥ 50% tổng lỗi). Mở vòng QC mới và giữ nguyên thư mục vòng cũ. |
| **Hold** | Thiếu bằng chứng, còn box `can_review`, còn vấn đề guideline/dữ liệu, hoặc chỉ là vòng demo lấy mẫu. |

### 5.2 Quy tắc phiên bản
- **Mọi thay đổi làm đổi nhãn** thì tăng phiên bản guideline.
- **`editorial`**: chỉ sửa câu chữ, giữ nguyên cách chọn vật, lớp, box, thuộc tính.
- **`semantic`**: đổi cách chọn vật, phân lớp, ranh giới box hoặc thuộc tính. **Bắt buộc kiểm lại toàn bộ 40 ảnh** (`pham_vi = lo`), vì sửa chỉ các ảnh bị cờ sẽ bỏ sót ảnh chưa bị gắn cờ.
- Khi đổi lớp hoặc thuộc tính: tăng `version` trong `taxonomy.json`, **giữ nguyên mục `rules`**, chạy `make_cvat_labels.py` để đồng bộ CVAT.
- Mỗi lần sửa quy tắc phải ghi vào bảng lịch sử và dẫn mã `R-xx`.

### 5.3 Lưu ý của cả nhóm
- IAA cao chưa đủ: hai người có thể **cùng hiểu sai** một quy tắc. Vì vậy cần đối chiếu thêm ca tham chiếu của coach.
- Nguyên nhân lệch được phân loại: quy tắc mơ hồ → sửa guideline; một người lệch → huấn luyện lại người đó; cả hai cùng sai → rà cả lô.

### 5.4 Lịch sử phiên bản

| Phiên bản | Thay đổi | Quy tắc / ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu: 5 lớp, thuộc tính định tính, chưa có ca khó. | - |
| v1.0 | Thêm lớp `bus`, `tuk_tuk`, `bicycle`; định lượng `occluded_level`; thêm R-01…R-15; thêm 12 ca khó theo ảnh (`img_002` … `img_038`). | R-01 … R-15 |
| v1.1 | *(điền sau calibration)* | *(điền)* |
