# Ghép UI theo demo — quy tắc chung

## 0. Ba quy tắc cứng

1. **Không bao giờ để ảnh bẹp, méo.** Simple giữ đúng tỉ lệ sprite; Sliced thì khung ≥ tổng border ÷ PPUM; sprite không có border thì không được Sliced rồi kéo méo. Chỉ ảnh màu trơn (nền tối, vùng bấm) mới được kéo giãn.
2. **Không chắc thì hỏi người làm trước.** Layout hoặc chức năng đặc biệt (component mô tả/chức năng riêng, cơ chế cuộn lạ, trạng thái ẩn hiện, ô đặc biệt kiểu EMPTY/NEW, hiệu ứng, nút demo không có…) phải hỏi hướng gắn, đưa 2–3 phương án kèm khuyến nghị. Không tự quyết.
3. **Khớp demo sát nhất có thể.** Không ước lượng bằng mắt: đo từ ảnh demo. Chỉ du di khi bất khả kháng (font khác glyph, ảnh chưa có).

---

## 1. Chuẩn bị trước khi ghép

- [ ] Đọc mọi ảnh demo của màn (`SpriteUI/<Feature>/Demo/`), kể cả các trạng thái (mở rộng, có tin nhắn, vừa tạo…).
- [ ] Quét **toàn bộ sprite** của bộ UI / project: loại import (Sprite?), kích thước, border 9-slice.
  - Sprite chưa là Sprite hoặc chưa cắt 9-slice mà màn cần → **báo người dùng, được đồng ý mới sửa**; cắt theo sprite cùng loại đã có border.
  - Sprite thiếu hẳn → giữ chỗ bằng Image sprite trống **alpha 0** (không để ô trắng), không bịa ảnh khác loại.
- [ ] **Tìm màn đã có cùng cấu trúc** để tái dùng (popup + tiêu đề + nút X, cột tab, list cuộn, panel thông tin, nút giá…). Màn cũ đã có logic thì **ghép lại tại chỗ** để giữ script và ref.
- [ ] Quét **scale ≠ 1** trong prefab cũ: đưa về 1 (giữ vị trí pivot) trước khi đặt kích thước, nếu không mọi số đo bị nhân theo scale cha.
- [ ] Hỏi trước những điểm chưa rõ (xem quy tắc cứng 2): cấu trúc tab hay cuộn, nút thừa/thiếu so với demo, logic khôi phục hay viết mới…
- [ ] Người dùng đang mở prefab với thay đổi chưa lưu → **chờ họ lưu**, không ghi đè.

## 2. Khung màn (popup)

- Cấu trúc gốc thống nhất:
  ```
  <Screen> (stretch, CanvasGroup, script màn)
  ├─ Opacity        nền tối, stretch, ảnh màu trơn đen ~60–80%, chặn bấm
  └─ Root           stretch
     └─ Panel       neo giữa, size cố định theo demo (popup không đổi size giữa các tỉ lệ màn)
        ├─ TitleHolder  prefab tiêu đề dùng chung (icon + chữ), neo trên-trái
        ├─ <Page/Body>  nội dung
        └─ CloseBtn     neo trên-phải, đặt sau cùng (luôn nằm trên)
  ```
- **Neo theo cạnh mà phần tử bám** khi đổi tỉ lệ: tiêu đề trái → neo trái; nút X → neo phải; dòng hint đáy → neo đáy; cột tab nằm ngoài mép trái popup → neo trái, pivot (1, 0.5).
- **Không dùng scale để thu nhỏ**; đặt size thật.
- Vùng nội dung dưới header: anchor tỉ lệ hoặc offset theo số đo, không hardcode lệch.
- Tên object khác nhau nhưng cùng nghĩa (Opacity/Opcity, btnClose/CloseBtn) không phải lỗi; tên trùng hoặc sai nghĩa (hai nút cùng tên, "Slider" cho scroll view) thì nên sửa khi tạo mới.

## 3. Cha-con và prefab

- **Cha-con theo nhóm hiển thị**: icon, nhãn, ô nhập, nút của một hàng/khung là **con của chính khung đó** (vd `NameRow` chứa nhãn + icon + ô nhập) → dời khung là cả cụm đi theo.
- Thành phần **tái sử dụng được** (thẻ gói, nút giá, nhãn %, thẻ item, tiêu đề popup, hotbar…) → **tạo prefab**, không nhân bản tay.
  - Cùng cấu trúc khác art → **prefab variant**.
  - Phần lồng trong thẻ (nút giá, tag) → **nested prefab**.
  - Phần tuỳ chọn trong prefab (nhãn %, NEW…) **mặc định tắt**; instance cần thì bật bằng override.
  - Đặt theo nhóm: `Prefabs/UI/UiElement/<Feature>/`. Đổi tên/di chuyển bằng move asset (giữ GUID).
- Chuyển thẻ đã ghép thành prefab mà vẫn giữ logic và dữ liệu riêng: lưu 1 thẻ mẫu thành prefab (connect), các thẻ còn lại **convert thành instance** (match theo hierarchy, ghi override) thay vì tạo mới.
- Người dùng sửa bên trong nested prefab ngay trên màn (override) → nhắc **Apply** vào prefab gốc nếu thay đổi phải đúng cho mọi chỗ dùng.
- Lặp trong cùng một màn: tách prefab khi hợp lý, không bắt buộc.

## 4. Đo và khớp demo

- **Tham chiếu tọa độ**: Canvas 1920×1080, demo 1920×1080 → 1 px demo = 1 đơn vị UI. Đặt theo **world rect** (tính cả scale cha), không theo `sizeDelta` thô.
- **Ảnh không bị đè** (icon, khung thẻ, ảnh thưởng…): so pixel ảnh sprite với demo (thử scale + vị trí, tính cả màu tô) → lấy sai số thấp nhất. Ra ~0.98–1.02 thì đặt **Native Size**. Nhiều sprite ứng viên (vd 6 cỡ đống vàng) → chọn theo sai số, **không gán theo thứ tự**.
- **Ảnh bị đè** (chữ/icon nằm trên khung): đo theo viền đen hoặc biên màu trong demo; hoặc lấy kích thước từ phần tử cùng loại đã đo chuẩn.
- **Nút 9-slice**: khung bao **cả phần đáy 3D / bóng**, không chỉ tới viền đen.
- **Màu chữ, màu khung**: chấm trực tiếp từ demo.
- **Chọn sprite**: tìm sprite hợp nhất trong **cả bộ UI / project**, so hình với demo; không dừng ở sprite đầu tiên.
- **Kiểm tra bằng overlay**: đặt ảnh demo (RawImage, alpha ~40–50%) phủ toàn màn, chụp Scene/Game view, so cả **những thứ demo không có** (object thừa, trạng thái bật/tắt sai). Ảnh demo có thể để sẵn trong prefab nhưng tắt.

## 5. Ảnh (Image)

| Loại | Quy tắc |
|---|---|
| Simple | Tỉ lệ rect = tỉ lệ sprite (hoặc bật Preserve Aspect). Đổi cỡ thì nhân đều hai chiều. |
| Sliced | PPUM mặc định **1**; chỉnh **size rect** cho khớp. Khung nhỏ hơn tổng border (vạch kẻ 2–4 px, ô nhập thấp, nút nhỏ) → **tăng PPUM** cho border vừa, không để góc đè. Không tăng PPUM chỉ để làm viền mỏng (góc bo sẽ lệch demo). |
| Không border | Không được Sliced rồi kéo méo; dùng Simple đúng cỡ hoặc nhờ cắt 9-slice. |
| Màu trơn | Nền tối / vùng bấm: được kéo giãn. |
| Sprite trống | Giữ Image, alpha 0. |

- Thanh nền tối, vạch kẻ: dùng sprite hình chữ nhật pixel của bộ UI + màu tô, không để Image sprite null hiện ô trắng.
- **Rà soát tự động sau mỗi màn**: mọi Image Simple lệch tỉ lệ > 3%, mọi Sliced nhỏ hơn border → sửa trước khi báo xong.

## 6. Chữ (TMP)

- **Font theo từng dòng chữ**: so kiểu chữ trong demo với các font trong thư mục Font của project; không dùng một font cho tất cả. Không chắc → hỏi.
- **Viền / bóng** (vd UIFX Outline + DropShadow) chỉ thêm ở chỗ demo có; chỗ không có để chữ trơn.
- **Khớp kích thước**: đo khung nét chữ trong demo theo màu → cỡ chữ theo chiều cao, khoảng cách chữ theo chiều rộng, rồi đặt tâm. Đo phía Unity bằng khung glyph thật (characterInfo), không dùng khung dòng.
- **Chữ trong nút**: stretch hết nút, căn giữa, character spacing 0, rồi **khớp theo hình**, không tin vào căn giữa bằng thông số (đặt giữa và căn giữa vẫn có thể lệch, vì ảnh nút có bóng/dải đậm ở đáy và font có khoảng thừa):
  - Đo trong demo: mép trên mặt nút → đỉnh chữ (gapTop) và đáy chữ → mép dưới mặt nút (gapBottom). Mặt nút không tính viền đen và bóng. Độ lệch = (gapBottom − gapTop) ÷ 2, dương là chữ nằm cao hơn tâm mặt nút.
  - Đo lại trong Unity trên ảnh chụp theo đúng cách đó; lệch demo quá 1–2 px thì dời chữ.
  - **Độ lệch chuẩn**: khi nhiều nút cùng bộ ảnh cho cùng một số thì ghi lại làm số khởi đầu cho màn sau. Bộ nút không có độ lệch ổn định thì không dùng; demo của màn đang ghép luôn thắng.
- **Chữ tĩnh vs chữ động**:
  - Tĩnh (PLAY, MAX, nhãn tab): cỡ cố định theo demo.
  - Động (tên, giá, số lượng, ngày, tên người chơi): **Auto Size** (max = cỡ demo, min đủ đọc), khung giới hạn rõ; khung cao ≥ 1.3 × cỡ chữ để autosize không tự thu nhỏ. Cho phép xuống dòng thì đặt độ rộng vùng chữ sao cho xuống dòng **đúng như demo**, căn trên, chỉnh line spacing theo demo. Thử với chuỗi dài nhất.
  - Nhận biết chữ động: field TMP được script gán (txtName, txtPrice…) hoặc hỏi.
- Chữ hiển thị đúng demo là được (gõ sẵn in hoa hay style UpperCase đều được).

## 7. Danh sách cuộn

- Viewport: RectMask2D. Content: neo theo hướng cuộn, **pivot ở mép bắt đầu**, vị trí 0:
  - Cuộn dọc: neo trên-stretch ngang, pivot (0.5, 1).
  - Cuộn ngang: neo trái-stretch dọc, pivot (0, 0.5) → thẻ đầu sát mép, thẻ thêm vào dài ra bên phải.
- Kiểm tra `anchoredPosition` cũ của Content (prefab cũ có thể lệch sẵn).
- LayoutGroup + ContentSizeFitter chỉ cho danh sách **thực sự cuộn**; phần **cố định không cuộn** (2 thẻ, cột tab) → xếp xong **xoá LayoutGroup/ContentSizeFitter**.
- Item mẫu: child **tắt sẵn**, tên `*_Template`.
- **Thanh cuộn**: luôn dùng thanh cuộn tự viết của project (track + tay nắm cỡ cố định, Simple), cho cả danh sách tái chế lẫn thường; không dùng Scrollbar mặc định.

## 8. Trạng thái, nút, logic

- **Trạng thái mặc định của prefab = demo**: object do code bật/tắt lúc chạy (nút bay, nút tấn công, nút nhảy, panel phụ) → đọc code xác nhận runtime tự toggle, rồi đặt mặc định đúng demo.
- Trạng thái chọn: hai ảnh chồng nhau (`Bg` + `SelectBg`), code chỉ bật/tắt, không đổi sprite.
- Nút dùng component nút chuẩn của project (có hiệu ứng bấm/âm thanh) nếu có.
- Trang trí không nhận raycast; vùng bấm thật mới bật.
- Code nhỏ cho một tác vụ (vd thanh cuộn) được tự viết thay vì ép component có sẵn — nhưng thay đổi logic màn thì giao người/phiên phụ trách logic, kèm danh sách object mới, ref, việc cần nối.
- Cái gì **thừa thì xoá hẳn** (chữ cũ sót dưới chữ mới, Image cũ chỉ bị tắt), không giữ "cho chắc" — trừ object có script/ref trỏ tới (khi đó báo người dùng).

## 9. Quy trình làm việc an toàn (khi agent AI ghép bằng Unity MCP hoặc Editor script)

1. Đọc trạng thái editor; prefab đang mở có thay đổi chưa lưu → dừng, hỏi.
2. Chia việc thành bước nhỏ (khung → từng cụm/trang → chữ), **mỗi bước lưu một lần**; script quá nặng (quét toàn bộ ref cho từng chữ) có thể làm Unity crash.
3. Thay chữ/ảnh nhưng giữ ref: sửa ngay trên object cũ; nếu phải thay object thì trỏ lại ref chỉ trong phạm vi nhỏ.
4. Sau khi lưu bằng script trong Prefab Mode, xoá cờ "chưa lưu" để mở prefab khác không bị hộp thoại modal làm treo editor.
5. Kiểm tra: không mất script/ref, không sót object tạm, rà soát ảnh méo, overlay demo.
6. Trước khi giao lại cho người dùng chỉnh, ghi lại số đo/snapshot để lần sau so được họ đã đổi gì.
7. Báo lại: đã làm gì, chỗ còn lệch và lý do, câu hỏi mở. Không commit nếu chưa được yêu cầu.


## 10. Animation show / hide

Áp dụng khi chưa cài skill `unity-ui-animation`. Đã cài thì theo skill đó; phần "Quy tắc khi làm" và "Kiểm tra" dưới đây vẫn dùng.

**Luôn đi theo luồng UI có sẵn của project**, không viết hệ animation mới song song.

- [ ] **Tìm luồng hiện có trước khi làm.**
  - Hàm ảo show/hide của lớp UI gốc (vd `PlayShowAnimation(done)` / `PlayHideAnimation(done)` / `StopAnimation()`).
  - Lớp popup đã có animation.
  - Helper tween và nền mờ dùng chung.
  - Màn mới cùng loại thì **kế thừa hoặc tái dùng** lớp đó.
- [ ] **Hỏi ý tưởng trước khi làm.** Hỏi người làm muốn animation thế nào và đề xuất 2–3 phương án kèm khuyến nghị. Hướng đề xuất:
  - **Giống màn cùng loại đang có**: đồng bộ, rẻ nhất. Thường là khuyến nghị mặc định.
  - **Đơn giản**: fade + scale nhẹ (0.9 → 1), 0.15–0.25 s, ease-out.
  - **Từng phần**: khung trước, rồi tiêu đề trượt vào, tab/thẻ xuất hiện so le (stagger 0.03–0.06 s), nút pop.
  - **Theo vị trí**: panel trượt từ cạnh nó bám (HUD, sidebar, thanh dưới).
  - Mỗi phương án nói rõ: thời lượng, phần nào chuyển động, hide có đảo ngược show không, có nền mờ không.
- [ ] **Quy tắc khi làm** (xem thêm mục 11 của `ui-standard.md`):
  - Animate CanvasGroup hoặc child hiển thị; không animate rect do LayoutGroup điều khiển.
  - Lưu baseline từ prefab, chạy xong trả về đúng baseline; không hardcode vị trí cuối.
  - Show đợi layout dựng xong một frame rồi mới đo hoặc chạy; trước đó để alpha 0 cho khỏi nháy.
  - Gọi callback hoàn tất (`done`) đúng một lần ở mọi nhánh. Mở hoặc đóng giữa chừng thì dừng animation cũ và trả trạng thái hợp lệ (alpha, interactable, blocksRaycasts).
  - Chặn bấm khi đang chạy. UI chạy lúc pause thì dùng unscaled time.
  - Có chế độ `instant` bỏ qua animation.
- [ ] **Kiểm tra**:
  - Mở/đóng nhanh liên tục, Back giữa chừng, đổi tab trong lúc chạy.
  - Đóng rồi mở lại không lệch vị trí, không phình dần.
