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

| Tình huống | Cách xử lý |
|---|---|
| Phân vân `car` hay `van` (hatchback/wagon, SUV, minivan, minibus nhỏ) | Dựa vào kiểu thân xe: ô tô con/SUV/wagon → `car`; xe giao hàng thân thùng kín hoặc minibus nhỏ → `van`. Không dùng riêng dấu hiệu có/không có cửa sổ sau để quyết định. Xe khách cỡ lớn không thuộc hai lớp này (xem mục 2). Còn phân vân → chọn lớp gần nhất và tick `can_review`. |
| Pickup có hàng, nắp thùng hoặc chở vật cồng kềnh | Pickup dân dụng nhận ra được qua cabin riêng và thùng phía sau → `car`, dù đang chở hàng, ngựa hoặc có nắp phủ thùng. Không kéo box xe sang hàng hóa. Xe tải có thân chở hàng chuyên dụng (ví dụ xe tải nhỏ cabin-over) → `truck`; nếu dáng xe không đủ rõ để phân biệt, tick `can_review`. |
| Xe đầu kéo chở ô tô | Gán một box `truck` cho đầu kéo và kết cấu rơ-moóc/chở xe; mỗi ô tô nhìn rõ trên sàn chở được gán box `car` riêng, không gộp vào box xe tải. |
| Xe buýt / xe khách cỡ lớn | Không có lớp tương ứng: **không** gán box xe, không ép vào `truck`/`van`. Người nhìn thấy bên ngoài vẫn gán `person`. Minibus nhỏ có kiểu thân xe thuộc `van` thì gán `van`. |
| Tuk-tuk, xe lam, auto-rickshaw, xích lô / cyclo | Đây là phương tiện ba bánh ngoài taxonomy: **không** gán box phương tiện. Gán `person` cho người đi bộ, người lái và hành khách nhìn thấy được; không gộp người vào box xe. |
| Xe đạp / xe đạp điện | Xe đạp không có lớp: **không** gán box xe, nhưng gán `person` cho người đạp. Chỉ gán `motorcycle` khi phương tiện thực sự có kiểu dáng xe máy; nếu xe điện khó phân biệt, tick `can_review`. |
| Xe máy có sidecar | Gán một box `motorcycle` bao phần nhìn thấy của cả xe máy và sidecar gắn liền; người ngồi trên xe có box `person` riêng. |
| Xe máy chở hàng hoặc bị che bởi hàng | Gán box `motorcycle` quanh phần xe nhìn thấy, không bao hàng hóa; gán người riêng. Đặt `occluded_level` theo phần xe bị che như mục 3.1. |
| Người lái / ngồi sau xe máy, kể cả trẻ em | Mỗi người nhìn thấy được có box `person` riêng; xe có box `motorcycle` riêng. Hai box được phép chồng lên nhau. |
| Nhiều xe máy hoặc người đứng sát, che nhau | Tách từng vật nhận ra được thành box riêng, dù box chồng nhau. Chỉ dùng `ignore_region` cho cụm không thể tách đáng tin cậy; không phủ một vùng lớn lên các vật đã gán riêng. |
| Người đẩy xe, xe nôi hoặc xe lăn | Gán `person`; xe đẩy/xe nôi/xe lăn không có lớp nên không gán box. |
| Chuyển động nhòe, thiếu sáng hoặc lóa | Chỉ gán người/xe còn nhận ra được. Vệt nhòe không xác định được vật thì không gán; dùng `ignore_region` gọn quanh cụm không thể phân biệt, không phủ cả ảnh nếu vẫn có vật rõ. |
| Phản chiếu trên gương/kính/vũng nước; người hoặc xe trong áp phích, màn hình, tranh | **Không** gán đối tượng chỉ xuất hiện dưới dạng phản chiếu hoặc hình ảnh. |
| Tượng, ma-nơ-canh, mô hình xe trong tủ kính hoặc xe làm tượng/đài tưởng niệm | **Không** gán vật thể mô phỏng, kể cả có hình dáng người/xe thật. Ngược lại, xe thật đang đỗ hoặc tham gia diễu hành vẫn gán theo lớp, dù được trang trí hoặc trưng bày. |
| Chỉ thấy một phần xe/người sau vật che hoặc ở mép ảnh | Gán phần nhìn thấy nếu vẫn nhận ra được lớp và đạt ngưỡng 4.5; đặt `occluded_level` và `truncated` đúng trường hợp. Không đoán phần khuất. |
| Vật quá xa/nhỏ hoặc không chắc có nên gán | Áp dụng ngưỡng 4.5; nếu nằm trong cụm vật nhỏ không thể tách thì dùng `ignore_region` theo mục 5. |
| Một vật bị vẽ box hai lần | Xóa box thừa, giữ một box ôm sát phần nhìn thấy của vật. |

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