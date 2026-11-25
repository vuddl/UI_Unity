# Ví dụ áp dụng: project AS92 (MineCraff)

Chỉ để minh hoạ cách điền các quy tắc chung bằng tài nguyên của một project cụ thể. Project khác thay bằng font, sprite, prefab của mình.


- Canvas 1920×1080, CanvasScaler **Expand**. Demo đặt ở `Assets/_Game/SpriteUI/<Feature>/Demo/`.
- Sprite dùng chung: `SpriteUI/Base/{Popup, Button, Frame, Icon, Currencies, Title, Font}`; riêng từng màn: `SpriteUI/<Feature>/`.
  - Popup: `Popup_blue/yellow/purple` (9-slice), nút X `button_close` (Simple).
  - Nút: `Button_green/blue/yellow/gray` (9-slice 116/45/116/43; nút thấp cần PPUM ≥ 88/chiều cao).
  - **Độ lệch chuẩn chữ trong nút `Button_*`** (đo 71 nhãn một dòng trong demo): chữ cao hơn tâm **mặt nút** 3 px (trung vị; dao động 0,5–5,5), tức là chữ nằm giữa phần sáng, không tính dải đậm 6 px ở đáy. Ảnh 88 px có viền đen trên 3, bóng dưới 9 nên tâm mặt nút đã cao hơn tâm rect 3. Cộng lại: **dời chữ lên +6 ÷ PPUM so với tâm rect** (PPUM 1 → +6; 1,1 → +5,5; 0,8 → +7,5). Nút mặt thấp hơn 70 px: khoảng +5 ÷ PPUM. Nút hai dòng hoặc có icon: đo riêng theo demo.
  - CutePixel lệch ngang: tâm nét chữ nằm trái 3,1% cỡ chữ (cỡ 70 → 2,2 px), dời phải bù lại. Dọc không lệch. Jersey25 không lệch cả hai chiều. Chiều cao chữ hoa = 0,5 × cỡ chữ (CutePixel), 0,61 × cỡ chữ (Jersey25).
  - Khung tối viền sáng: `frame_dark_alpha`; khung xanh nhạt: `frame_green`; thanh/vạch: `frame_retangle_pixel` + màu tô; nhãn %/NEW: `Tag_offers`.
  - Thanh cuộn: track `scroll_bg` / `frame_orange` + tay nắm `scroll_button` / `frame_green`, component `ScrollTrackBar` (field `scroll` cho LoopScrollRect, `plainScroll` cho ScrollRect).
- Font (`SpriteUI/Base/Font`): **CutePixel SDF** cho tiêu đề, nút, nhãn (trắng + UIFX `OutlineFilter` **size 3** (không dùng 1.5, người dùng chê quá bé) + `DropShadowFilter` + `FilterStackTextMeshPro`; hoặc đen trơn trên nền sáng); **Jersey25-Regular SDF** cho số, chú thích, nội dung (vd `EQUIPPED`, `Limited: 0/1`, tin nhắn chat).
- Màu hay dùng: chữ tối #232632 / #1E2430, tên vàng #FFDC64, xanh "World Created!" #60CC06, nền tối 60–80% đen.
- Prefab dùng chung: `Prefabs/UI/UiElement/TitleHolder` (tiêu đề popup), `HotBar`, `EquipmentItem` (+ variant `MountItem`), `Shop/{ShopPackCard, ShopGemCard (variant), ShopOfferCard, PriceButton, ShopTag}`, `CreativeInventoryShop/InventoryTab`.
- Popup nhiều tab: `UITabService` + mỗi tab một page (CanvasGroup), cột tab trái dùng `InventoryTab`.
- Danh sách tái chế: `LoopVerticalScrollRect`; danh sách thường: `ScrollRect` + `ScrollTrackBar.plainScroll`.

## Animation show / hide (AS92)

- Lớp gốc `Framework/UI/UIBase.cs` có các hàm ảo `PlayShowAnimation(Action)`, `PlayHideAnimation(Action)` và `StopAnimation()`.
- Popup nhiều tab kế thừa `Inventory/AnimatedTabPopup.cs` (CharacterScreen, ShopScreen, MountScreen…). Lớp này làm các việc sau:
  - Show: chờ layout một frame, đặt alpha 0, rồi chạy `InventoryPopupMotion`. Thứ tự: khung → tiêu đề trượt từ trái → tab so le → item → nút pop.
  - Hide: chạy ngược lại, sau đó ẩn nền `PixelBackdrop`.
  - Đổi tab: tiêu đề cũ trượt ra, ảnh của tab mới chạy animation.
- Màn mới cùng kiểu popup nên kế thừa `AnimatedTabPopup`. Màn có bố cục khác thì thêm nhánh `Add…` cho bố cục đó vào `InventoryPopupMotion`.
