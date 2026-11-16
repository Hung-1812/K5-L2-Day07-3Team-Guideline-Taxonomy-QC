# Guideline gán nhãn — v1 (bản bổ sung)

- Phiên bản: v1 (dựa trên v0)
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

> Các phần đánh dấu **[QUYẾT ĐỊNH]** là chỗ v0 còn trống nên v1 chọn một cách làm cụ thể.
> Nếu nhóm/giảng viên chọn khác, hãy sửa và tăng phiên bản.

---

## 1. Mục tiêu và phạm vi

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố (camera giao thông, xe tự lái).

**Nguyên tắc chung:** gán nhãn để mô hình học được thứ *người nhìn thấy được trong ảnh*, không phải thứ ta đoán. Khi không chắc, ưu tiên nhất quán hơn là hoàn hảo: cùng một tình huống phải luôn được gán giống nhau.

**Không thuộc phạm vi:** biển số, màu xe, hướng xe, hành vi (đi sai làn, không đội mũ), ý định của người đi đường.

## 2. Lớp

Mỗi vật thuộc **đúng một lớp**. Thứ tự kiểm tra khi phân vân: `ignore_region` → lớp cụ thể → `can_review`.

| Lớp | Gồm | Không gồm |
|---|---|---|
| `car` | Ô tô con 4–7 chỗ: sedan, hatchback, SUV, bán tải cỡ nhỏ (pickup) chở người, taxi | Xe tải, xe thùng kín cỡ nhỏ (→ `van`), xe buýt |
| `truck` | Xe tải mọi cỡ, xe đầu kéo, xe ben, xe chở container, xe bồn, xe cẩu | Pickup dân dụng (→ `car`) |
| `van` | Xe thùng kín cỡ nhỏ: xe giao hàng, minivan, xe 16 chỗ kiểu minibus nhỏ | Ô tô con (→ `car`) |
| `motorcycle` | Xe máy, mô tô, xe tay ga, xe máy điện; **chỉ phần xe** | Người ngồi trên xe (→ `person` riêng), xe đạp |
| `person` | Người đi bộ, người lái và người ngồi sau xe máy, người đạp xe, người đứng/ngồi bên đường | Hình người trên áp phích/biển quảng cáo, ma-nơ-canh, tượng, bóng phản chiếu trên kính |
| `ignore_region` | Xem mục 5 | |

**[QUYẾT ĐỊNH] Vật không có lớp tương ứng** (xe buýt, xe đạp, xe ba gác, xích lô, xe lam…): v0 không có lớp cho các vật này.
- **Không** ép vào lớp gần giống.
- Nếu nó đứng riêng và rõ: **không gán box**, nhưng ghi vào danh sách "vật ngoài lớp" (mục 8) để giảng viên cân nhắc thêm lớp.
- Nếu nó nằm trong đám đông phương tiện khó tách: có thể gộp vào `ignore_region`.
- Người đang đạp xe đạp vẫn gán `person` bình thường.

## 3. Thuộc tính

### 3.1 `occluded_level` (mức bị che)

Đánh giá theo **phần thân vật bị che bởi vật khác** (xe khác, cột, cây, người), không tính phần bị cắt ở mép ảnh.

| Mức | Tỉ lệ bị che (ước lượng bằng mắt) | Mô tả dễ nhớ |
|---|---|---|
| `0` | 0 – ~10% | Thấy rõ toàn bộ |
| `1` | ~10 – 40% | Bị che ít, vẫn dễ nhận ra hình dạng |
| `2` | ~40 – 80% | Bị che nhiều, chỉ thấy một phần lớn |
| `3` | > ~80%, hoặc chỉ đoán được bằng ngữ cảnh | Gần như không thấy, hoặc không thể ước lượng |

Lưu ý:
- Chỉ ước lượng bằng mắt, không cần đo chính xác. Giữa hai mức thì chọn mức **cao hơn** nếu phần bị che chạm vào bộ phận nhận dạng chính (bánh, đèn, đầu người).
- Mức `3` và vật gần như không thấy: xem mục 4.5 (ngưỡng gán/không gán).

### 3.2 `truncated`

- `yes` khi vật **bị cắt bởi mép ảnh** (một phần vật nằm ngoài khung hình).
- `no` ở mọi trường hợp còn lại, kể cả khi bị che bởi vật khác (đó là `occluded_level`, không phải `truncated`).
- Một vật có thể vừa `truncated=yes` vừa `occluded_level>0`.

### 3.3 `can_review`

Tick khi:
- Không chắc lớp (ví dụ `car` hay `van`).
- Không chắc có nên gán hay không (vật rất nhỏ, rất mờ).
- Không chắc `occluded_level` giữa hai mức cách xa nhau (ví dụ 1 hay 3).
- Gặp tình huống chưa có trong guideline.

**Không** tick chỉ để "cho chắc" trên vật rõ ràng. Tick quá nhiều làm người review quá tải và mất tác dụng. Khi tick, ghi ngắn lý do trong ô mô tả/ghi chú của CVAT (ví dụ "van hay car?").

## 4. Quy tắc vẽ box

### 4.1 Gán box cho cái gì
1. Gán box cho mọi **người** và **phương tiện thuộc 5 lớp** nhìn thấy được trong ảnh, kể cả xe đang đỗ, xe ở xa, xe bị che một phần.
2. **Không** gán box cho người **bên trong** ô tô (không nhìn rõ qua kính). Chỉ gán `person` cho người ngồi trên xe máy, hoặc người nhìn rõ (ví dụ thò đầu ra ngoài cửa sổ).
3. Mỗi vật thật chỉ có **một** box. Không vẽ hai box chồng lên cùng một vật.

### 4.2 Box ôm sát
- Cạnh box chạm sát điểm ngoài cùng của vật ở bốn hướng: không chừa khoảng trắng dư, không cắt mất phần vật.
- Sai số chấp nhận được: khoảng **1–2 pixel** với vật lớn; với vật nhỏ, không để box lớn hơn vật quá ~20%.
- Phóng to ảnh khi vẽ vật nhỏ. Không vẽ vội ở mức zoom thấp.

### 4.3 Vật bị che: vẽ phần nhìn thấy hay cả vật?
**[QUYẾT ĐỊNH] Chỉ vẽ box quanh phần *nhìn thấy được*** (visible-only). Không đoán phần bị che.
- Lý do: dễ nhất quán giữa nhiều người gán; không phải đoán.
- Phần nhìn thấy bị chia đôi bởi vật che (ví dụ cột điện ở giữa xe): vẽ **một** box bao cả hai phần.

### 4.4 Vật bị cắt ở mép ảnh
- Vẽ box đến sát mép ảnh (box có thể chạm mép).
- Đặt `truncated=yes`.
- Nếu chỉ còn một mảnh rất nhỏ (xem 4.5): không gán.

### 4.5 Ngưỡng tối thiểu: khi nào không gán
**[QUYẾT ĐỊNH]** Không gán box (và không cần tick) khi:
- Vật nhỏ hơn khoảng **10×10 pixel** trên ảnh gốc, hoặc
- Phần nhìn thấy quá ít để một người nhìn ảnh bình thường (không zoom quá mức) nhận ra đó là gì.

Nếu vật nằm trong vùng đông nhiều vật nhỏ như vậy, dùng `ignore_region` (mục 5). Nếu **lưỡng lự** ngay sát ngưỡng: gán và tick `can_review`.

### 4.6 Người và xe máy
- Người lái / người ngồi sau: **mỗi người một box `person`**, riêng với box `motorcycle`.
- Box `person` và box `motorcycle` được phép chồng lên nhau (điều này bình thường).
- Xe máy không có người (đỗ): chỉ có box `motorcycle`.
- Hàng hóa cồng kềnh trên xe máy: không đưa hàng vào box `motorcycle` trừ khi nó che dính liền với thân xe và khó tách; mặc định box ôm **thân xe**.

### 4.7 Xe kéo và vật mang theo
- Xe đầu kéo + rơ-moóc: **một** box `truck` bao cả hai.
- Xe kéo xe khác (cứu hộ): xe cứu hộ là `truck`, xe bị kéo gán lớp riêng của nó, hai box riêng.

### 4.8 Hai vật sát nhau
- Hai người/xe đứng sát, che nhau: hai box riêng, được phép chồng.
- Nếu không thể tách được hai vật (đám người dày đặc): xem `ignore_region`.

## 5. `ignore_region`

Dùng cho vùng mà **việc gán từng vật là không thực tế hoặc không đáng tin**. Mô hình sẽ không bị phạt khi đoán trong vùng này.

Dùng khi:
- **Đám đông** hoặc dãy xe máy dày đặc, không tách được từng vật.
- Nhiều vật rất nhỏ/rất xa (dưới ngưỡng 4.5) tụ thành cụm.
- Vùng quá mờ, nhòe chuyển động, quá tối hoặc lóa sáng, không phân biệt được vật.
- Vật ngoài lớp nằm lẫn trong đám phương tiện (mục 2).

Quy tắc vẽ:
- Vẽ hộp bao **đủ chặt** quanh cả cụm; không dùng một hộp khổng lồ cho nửa ảnh.
- **Không** vẽ `ignore_region` đè lên vật đã gán box riêng rõ ràng, trừ khi cả vùng thật sự không tách được.
- Không dùng `ignore_region` như đường tắt cho vật khó nhưng gán được. Chỉ dùng khi thật sự không thể gán từng vật.
- `ignore_region` không có `occluded_level` / `truncated`; bỏ qua các thuộc tính đó.

## 6. Ca khó (bảng quyết định nhanh)

| Ảnh | v0 chưa trả lời rõ | Quyết định |
|---|---|---|
| `img_002.jpg` | Xe đầu kéo chở ô tô: gộp tất cả thành một box hay tách từng xe? | Một box `truck` cho đầu kéo và rơ-moóc; mỗi ô tô nhìn rõ trên sàn chở có box `car` riêng. |
| `img_004.jpg`, `img_006.jpg`, `img_035.jpg` | Tuk-tuk, cyclo và auto-rickshaw trông giống phương tiện có động cơ nhưng không có lớp riêng. | Không gán box phương tiện ngoài taxonomy; gán `person` riêng cho người nhìn thấy được. Không ép xe ba bánh vào `motorcycle`, `car` hoặc `truck`. |
| `img_005.jpg`, `img_021.jpg`, `img_030.jpg` | Giao thông đông: gán từng xe/người chồng lấp hay gom cả vùng? | Tách box cho từng người và xe nhận ra được, cho phép box chồng. Chỉ dùng `ignore_region` cho cụm thực sự không thể tách đáng tin cậy. |
| `img_009.jpg`, `img_032.jpg` | Xe khách/minibus nằm giữa lớp `van` và vật ngoài taxonomy. | Minibus nhỏ có kiểu thân xe thuộc `van` thì gán `van`; xe buýt/xe khách cỡ lớn không có lớp thì không gán box xe. Người nhìn rõ vẫn gán `person`. |
| `img_012.jpg`, `img_027.jpg`, `img_028.jpg`, `img_037.jpg` | Pickup chở hàng, có đồ trên thùng hoặc gắn nắp thùng: `car` hay `truck`, box có ôm hàng không? | Pickup dân dụng có cabin riêng và thùng phía sau là `car`, không phụ thuộc hàng đang chở; box không bao hàng. Xe tải thân chở hàng chuyên dụng là `truck`; nếu không phân biệt chắc chắn thì tick `can_review`. |
| `img_014.jpg` | Người và xe bị nhòe do chuyển động: có đoán box theo vệt mờ không? | Chỉ gán vật còn nhận ra được. Không đoán vật từ vệt nhòe; nếu một cụm không thể phân biệt thì dùng `ignore_region` gọn quanh cụm. |
| `img_017.jpg`, `img_019.jpg`, `img_027.jpg`, `img_033.jpg`, `img_038.jpg` | Tượng, ma-nơ-canh, xe mô hình và xe thật được trang trí/trưng bày dễ bị nhầm. | Không gán tượng, ma-nơ-canh, mô hình xe hoặc xe làm tượng/đài tưởng niệm. Gán xe thật đang đỗ hoặc diễu hành theo lớp tương ứng. |
| `img_018.jpg`, `img_020.jpg` | Người/xe xuất hiện trên biển quảng cáo hoặc trong gương có phải đối tượng cần gán? | Không gán hình in/hiển thị trên biển, áp phích hay ảnh phản chiếu; chỉ gán người/xe thật trong cảnh. |
| `img_022.jpg`, `img_029.jpg` | Sidecar hoặc hàng cồng kềnh làm thay đổi hình dạng xe máy: box `motorcycle` bao phần nào? | Sidecar gắn liền được bao cùng xe trong một box `motorcycle`; hàng hóa không thuộc box. Gán người riêng và đặt `occluded_level` nếu hàng che xe. |
| `img_007.jpg`, `img_036.jpg` | Xe tải nhỏ trong kho/đường phố có thể trông như `van`, nhất là khi thiếu sáng hoặc thân xe bị che. | Xe tải có cabin-over/thân chở hàng chuyên dụng là `truck`; xe thùng kín cỡ nhỏ/minivan là `van`. Chỉ dựa vào phần nhìn thấy; nếu không phân biệt chắc chắn thì tick `can_review`. |

## 7. Quy trình làm việc

1. **Lướt cả ảnh** một lượt, ghi nhận các vùng đông/khó trước.
2. Vẽ box cho vật **lớn, rõ** trước; sau đó vật nhỏ, bị che; cuối cùng `ignore_region`.
3. Đặt `occluded_level` và `truncated` ngay khi vẽ từng box (không để cuối).
4. **Tự kiểm tra** trước khi nộp ảnh:
   - Còn vật nào thuộc 5 lớp chưa có box?
   - Có box nào thừa/trùng/vẽ lên phản chiếu hoặc áp phích?
   - Box có ôm sát không?
   - Mọi vật chạm mép ảnh đã `truncated=yes` chưa?
   - Mọi `can_review` đã có ghi chú lý do chưa?
5. Không sửa nhãn người khác trực tiếp nếu chưa thống nhất; ghi chú và báo lại.