# Quy chuẩn UI Unity (uGUI) — v1.0 (10/10/2026)

Nguồn: `Quy_chuan_UI_Unity.docx`. Áp dụng cho mọi project Unity. Mục tiêu: designer chỉnh bố cục trực tiếp trong prefab, UI co giãn đúng trên nhiều màn hình, **code chỉ điều khiển dữ liệu, trạng thái, tương tác**.

Mặc định cho game **ngang** trên PC / điện thoại / iPad. Project dọc hoặc nền tảng khác: chốt lại cấu hình Canvas và bộ tỷ lệ kiểm thử **trước** khi dựng UI.

## 1. Nguyên tắc bắt buộc
- Dùng **uGUI Canvas + RectTransform**. Không dùng IMGUI/NGUI cho screen, popup, HUD.
- Cấu trúc, component, anchor, kích thước, màu, reference **lưu sẵn trong prefab**; prefab xem và sửa được trong Editor.
- Runtime **không** tự tạo cây UI, không AddComponent để chữa thiếu reference, không ghi đè vị trí/kích thước designer đã chỉnh.
- UIManager được instantiate prefab dựng sẵn và quản lý pool (đó là vòng đời view, không phải dựng UI bằng code).
- Ngoại lệ có chủ đích (phải khai báo phạm vi rõ): animation, SafeArea, world indicator, kéo chỉnh HUD, layout component khai báo trong prefab.

## 2. Cấu hình Canvas
| Thuộc tính | Mặc định |
|---|---|
| Render Mode | Screen Space Overlay; chỉ dùng Camera khi có nhu cầu cụ thể |
| Canvas Scaler | Scale With Screen Size, Reference Resolution **1920×1080** |
| Screen Match Mode | Match Width Or Height |
| Điện thoại & PC ngang | Match = **1** (ưu tiên chiều cao) |
| iPad | Match = **0** (ưu tiên chiều rộng) |
| Tablet Android | Chốt chính sách riêng; không mặc định coi là iPad |

1920×1080 là hệ tọa độ tham chiếu, không phải kích thước cố định cho mọi panel. Cấu hình thiết bị chỉ áp dụng khi khởi tạo hoặc khi thông số màn hình đổi.

### 2b. Project màn dọc (Portrait)
Project có `defaultInterfaceOrientation = Portrait` (vd ColorDraw) dùng bảng này thay cho bảng trên:

| Thuộc tính | Màn dọc |
|---|---|
| Reference Resolution | **1080×1920** |
| Screen Match Mode | Match Width Or Height |
| Điện thoại dọc (9:16, 9:19.5…) | Match = **0** (ưu tiên chiều rộng; máy dài thì dư chiều cao, header/footer neo cạnh) |
| iPad / tablet dọc (3:4) | Match = **1** (ưu tiên chiều cao, tránh tràn dọc) |
| Bộ tỷ lệ kiểm thử | 1080×1920, 1080×2400 (máy dài), 1536×2048 (iPad) |

- Header neo trên, footer/nút chính neo dưới; phần giữa stretch hoặc ScrollRect dọc.
- Vùng banner quảng cáo dưới đáy là phần chừa sẵn của Footer/SafeContent, không để nội dung cuộn bị che.

## 3. Quy tắc RectTransform / Anchor
Neo 9 điểm cho phần tử cố định theo góc/cạnh; container phủ vùng dùng **stretch**; pivot chọn theo hướng phần tử mở rộng.

| Thành phần | Anchor & co giãn |
|---|---|
| Screen, nền | Stretch 2 chiều, offset = 0 khi phủ toàn parent |
| Header | Stretch ngang, neo trên; chiều cao khai báo trong prefab |
| Footer | Stretch ngang, neo dưới; nằm ngoài vùng cuộn |
| Sidebar | Neo trái, stretch dọc; chiều rộng khai báo rõ |
| Body / Content | Stretch vùng còn lại, chừa header/footer/sidebar |
| Nút góc, thanh trạng thái | Neo đúng góc hoặc giữa cạnh tương ứng |
| Popup nhỏ | Neo giữa; giới hạn kích thước theo vùng hiển thị |
| Popup lớn | Stretch có lề; nội dung dài dùng ScrollRect |
| Tâm ngắm | Anchor & pivot (0.5, 0.5), giữa full Canvas |
| Nội dung cuộn dọc | Stretch ngang, pivot trên; cao theo nội dung |

**Cấm**: container neo 1 điểm, rộng/cao = 0, rồi bố trí con bằng tọa độ toàn màn hình. `sizeDelta = 0` hợp lệ khi đang stretch — luôn kiểm tra anchor + kích thước thực, không chỉ nhìn Width/Height.

## 4. Layout Group & quyền điều khiển layout
| Nhu cầu | Cách dựng |
|---|---|
| Khối xếp dọc | VerticalLayoutGroup; spacing/padding trên group |
| Lựa chọn ngang | HorizontalLayoutGroup; LayoutElement min/preferred/flexible |
| Ô cùng kích thước | GridLayoutGroup khi cell size đã xác định |
| Nội dung dài | ScrollRect + Mask; ContentSizeFitter chỉ trên trục cần fit |
| Ảnh giữ tỷ lệ | AspectRatioFitter hoặc Preserve Aspect (Fit/Crop) |
| HUD kéo vị trí tự do | RectTransform tự do theo anchor; không để group điều khiển |

- **Mỗi trục chỉ một cơ chế quyết định kích thước** — không để LayoutGroup, ContentSizeFitter và script cùng tranh quyền.
- GridLayoutGroup không tự chia cell vừa chiều rộng → chọn cách chia cột rõ ràng. Nút chia đều hàng → flexible width.
- Phần tử trong group chỉnh bằng LayoutElement và padding/spacing của group. Ẩn nhóm tùy chọn thì bố cục phải thu hồi khoảng trống.

## 4b. Multi-size / auto-layout bằng code (quy tắc bổ sung)
- **Prefab UI phải luôn sửa tay được**: chỉnh vị trí, kích thước, anchor trong prefab xong, vào Play vẫn thấy đúng như đã chỉnh; không có script nào tự đặt lại.
- **Ưu tiên tự căn chỉnh không cần code**: anchor/stretch, LayoutGroup, LayoutElement, ContentSizeFitter, AspectRatioFitter. Chỉ viết code khi các công cụ này không làm được.
- Nếu bắt buộc dùng code để multi-size/auto-layout:
  - Chạy trong **`OnEnable`, đúng một lần** cho mỗi lần bật (có cờ chống chạy lặp). **Không chạy trong `Update`/`LateUpdate`**, không chạy theo frame hay coroutine lặp.
  - **Trước khi tính, đọc giá trị hiện tại** của từng thành phần UI liên quan (anchorMin/Max, pivot, anchoredPosition, sizeDelta, localScale, padding/spacing…) và lấy chúng làm baseline. Không dùng hằng số hardcode thay cho giá trị trong prefab.
  - Kết quả tính từ baseline đó (vd nhân theo tỷ lệ màn hình, cộng safe-area offset), để chỉnh tay trong prefab vẫn có tác dụng và không bị tự chỉnh lại.
  - Lưu baseline lần đầu; lần bật sau tính lại **từ baseline**, không tính chồng lên kết quả cũ (tránh trôi/phình dần).
  - Chỉ chạy lại khi kích thước màn hình hoặc safe rect thực sự đổi, và vẫn tính từ baseline.
  - Script chỉ đụng vào đúng thuộc tính nó chịu trách nhiệm; không ghi vào phần tử do LayoutGroup quản lý (quy tắc một cơ chế cho mỗi trục ở mục 4).

```csharp
// Mẫu: auto-size trong OnEnable, tôn trọng giá trị đã chỉnh tay trong prefab
[SerializeField] RectTransform target;
Vector2 baseSize, basePos; bool hasBaseline, applied;

void OnEnable()
{
    if (applied) return;                       // đúng một lần mỗi lần bật
    if (!hasBaseline)                          // đọc giá trị hiện tại từ prefab
    {
        baseSize = target.sizeDelta;
        basePos  = target.anchoredPosition;
        hasBaseline = true;
    }
    float k = ComputeScale();                  // tỷ lệ theo màn hình
    target.sizeDelta        = baseSize * k;    // tính từ baseline, không hardcode
    target.anchoredPosition = basePos  * k;
    applied = true;
}

void OnDisable() => applied = false;
```

## 5. Phân lớp & vòng đời UIManager
UIManager khởi tạo ở Startup, `DontDestroyOnLoad`, **duy nhất 1 instance**; scene sau không tạo thêm manager hay EventSystem trùng.

| Lớp | Trách nhiệm |
|---|---|
| Screens | Điều hướng màn chính, quản lý screen đang active |
| HUD | Module chung + module riêng từng mode |
| Popups | Stack popup, modal, focus, Back, chặn input |
| Notifications | Hàng đợi, số lượng đồng thời, thời gian hiển thị, pool |
| WorldIndicators | Nhãn nhân vật, damage; chiếu tọa độ hoặc world canvas |
| Overlays | Loading, fade, transition; thứ tự hiển thị quy định |

Đây là lớp trách nhiệm, không bắt buộc mỗi lớp một Canvas. Chỉ tách Canvas khi thứ tự render / tần suất cập nhật đòi hỏi **và đã đo chi phí**. Overlay chuyển cảnh trên cùng; popup modal phải che và chặn đúng các lớp dưới.

## 6. Cây prefab chuẩn
```
Screen/Popup
├─ Background            (full Canvas)
└─ SafeContent
   ├─ Header
   ├─ Body
   │  ├─ Navigation
   │  └─ Content | ScrollView
   └─ Footer
```
- Nút Back, Close, Save, Play để **ngoài vùng cuộn** khi cần luôn truy cập.
- HUD chia module: `SharedHUD`, `<Mode>HUD` (vd DeathmatchHUD, TeamHUD, ZombieHUD…). Chỉ bật module của mode hiện tại; không gom mọi mode vào một view lớn với điều kiện chồng chéo.

## 7. SafeArea & tâm ngắm
- Background, hiệu ứng phủ màn hình, tâm ngắm dùng **toàn Canvas**. Chỉ nội dung cần tránh tai thỏ/vùng hệ thống mới vào SafeContent. Không lồng nhiều lớp SafeArea.
- Tâm ngắm **ngoài SafeContent** để luôn ở tâm màn hình, kể cả SafeArea bất đối xứng.
- SafeArea chỉ cập nhật khi kích thước màn hình hoặc safe rect đổi; không tự đổi reference resolution (vd về 1280×720).

## 8. Reference & dữ liệu
- View giữ **serialized reference** tới Text, Image, Button, RectTransform cần dùng. Thiếu reference → log lỗi có ngữ cảnh để sửa prefab; **không âm thầm tạo UI thay thế**.
- Gameplay cập nhật model/data store → Presenter/View nhận Action/event (máu, đạn, điểm, inventory, trạng thái trận…).
- Subscribe/unsubscribe **đối xứng**; khi mở view, đồng bộ snapshot hiện tại trước khi nhận cập nhật tiếp.

## 9. Text & hình ảnh
- Text của nút stretch trong vùng nút có padding; không giữ width cũ của label khi đổi kích thước nút.
- Quy định font, cỡ chữ, alignment, wrap, overflow. Auto size chỉ trong khoảng dễ đọc; không thu chữ quá nhỏ để che lỗi bố cục.
- Ảnh giữ tỷ lệ gốc: Fit (thấy đủ) hoặc Crop có kiểm soát; không kéo méo.
- Tên map, trạng thái khóa, Coming Soon là các vùng riêng; map mở được xếp trước theo mode; lớp khóa tối vẫn thấy ảnh nền.
- Bảng điểm: cột độ rộng nhất quán (tên, vai trò, kill, death). Khi mở bảng tổng kết, ẩn thanh thông tin trận trùng lặp phía trên.

## 10. Input & mobile multitouch
- Joystick, Aim, Fire, vùng vuốt camera **quản lý pointerId riêng**; một ngón không chiếm input của ngón khác. Aim giữ theo pointer down, kết thúc khi pointer up hoặc cancel.
- Khi đóng view, mất focus hoặc mở modal chặn gameplay → giải phóng trạng thái đang giữ, đưa input về zero.
- Vùng trang trí **không nhận raycast** (Raycast Target = off).
- Popup khai báo rõ có chặn gameplay, camera, nút bên dưới hay không.

## 11. Animation & feeling UI
- Screen, popup, thông báo có animation vào/ra và phản hồi chọn. UI cần chạy khi pause dùng **unscaled time**.
- Animation lưu và khôi phục **baseline từ prefab**; không dùng kích thước hardcode làm điểm kết thúc.
- Ưu tiên animate **CanvasGroup** hoặc một **Visual child** bên trong container. Không animate anchoredPosition/size do LayoutGroup điều khiển.
- Bị ngắt hoặc mở/đóng liên tục: alpha, scale, interactable, blocksRaycasts phải về trạng thái hợp lệ.
- Text đếm ngược chỉ pulse khi số đổi. Thông báo/damage có vòng đời rõ, không reset tween mỗi frame. Hiệu ứng không che tâm ngắm, không làm sai vùng chạm.

## 12. Hiệu năng & code
- Chỉ update mỗi frame cho hành vi liên tục thật (tween, chiếu world indicator). Timer chỉ đổi text khi số giây đổi.
- Cache reference; tránh `Find`, `GetComponent` lặp, nối chuỗi, rebuild layout không cần thiết trong `Update`.
- Pool damage text và notification; giới hạn số lượng hiển thị, trả về pool khi xong.
- Code tách hàm rõ, thụt lề nhất quán, không nhồi nhiều lệnh một dòng. Debug Game tách tab, có cấu hình ẩn khỏi bản release.
- Tool dựng/migrate prefab nằm trong **Editor**, chạy có chủ đích, **không tự ghi đè prefab đã chỉnh tay**.

## 13. Quy trình cho project mới
1. Chốt orientation, thiết bị mục tiêu, reference resolution, chính sách Match (project dọc cấu hình riêng).
2. Dựng Startup, UIManager, EventSystem, phân lớp UI; khai báo luồng screen, stack popup, input blocking.
3. Dựng **một screen mẫu** có header, sidebar, scroll content, footer; kiểm tra co giãn trước khi nhân rộng.
4. Lưu prefab, bind reference; tách module HUD theo mode; nối data store bằng event/Action.
5. Thêm animation, xử lý cancel, pause, scene change; đo hiệu năng trên thiết bị mục tiêu.
6. Chỉ chuyển các màn còn lại khi screen mẫu đạt nghiệm thu và designer sửa prefab độc lập được.

## 14. Chỉnh nhanh trong Editor
| Muốn chỉnh | Chỉnh tại |
|---|---|
| Lề, khoảng cách cả nhóm | Padding, spacing của LayoutGroup |
| Rộng/cao ô trong group | LayoutElement (min, preferred, flexible) |
| Nút HUD độc lập | Anchor, pivot, anchoredPosition, size của RectTransform |
| Nội dung dài bị tràn | Viewport, ScrollRect, Content — không kéo footer xuống |
| Text đè nhau | Vùng label, wrap, overflow, bề rộng cột; thử chuỗi dài |
| Hiệu ứng popup | Visual child + CanvasGroup; giữ container layout ổn định |

## 15. Checklist trước khi bàn giao
- [ ] Đã xem Game View ở 16:9, điện thoại dài, tablet 4:3 (project dọc: 9:16, 9:20, 3:4); thử SafeArea bất đối xứng.
- [ ] Tâm ngắm đúng tâm; không chữ đè, ảnh méo, nút vượt khung.
- [ ] Thử nội dung rỗng, dài, nhiều hàng; header/footer không bị cuộn mất.
- [ ] Đổi tab/mode không để lại khoảng trống hoặc HUD mode cũ.
- [ ] Thử mở/đóng nhanh, pause, Back, chuyển scene, nhiều ngón cùng lúc.
- [ ] Chỉnh vị trí/kích thước trong prefab không bị runtime ghi đè (ngoài ngoại lệ đã khai báo).
- [ ] Script auto-size/multi-size (nếu có) chạy trong OnEnable đúng một lần, không có trong Update, đọc baseline từ prefab trước khi tính; sửa tay prefab rồi Play vẫn giữ đúng.
- [ ] Không manager, EventSystem, listener, object UI bị nhân đôi sau nhiều lần mở.
- [ ] Không lỗi compile/runtime; đã kiểm tra hình ảnh thực tế và chi phí cập nhật UI.

> Compile thành công chỉ xác nhận code hợp lệ. Nghiệm thu UI cần kiểm tra trực quan + tương tác; test trong Editor không thay thế test cảm ứng và hiệu năng trên thiết bị thật.
