# Luồng nhiều phiên khi gọi /unity-ui

Chỉ chạy luồng này khi **người dùng gõ `/unity-ui`** (hoặc yêu cầu mở luồng UI_Skill) trong một chat chưa có vai. Agent tự đọc skill để sửa một màn thì bỏ qua file này. Phiên đã được giao vai (Ghép UI, Nối logic, Animation) **không mở thêm phiên**.

```
UI_Skill (chat gõ lệnh: nhận lệnh, chia việc, nhận báo cáo, giữ lượt Unity)
 ├─ Ghép UI      layout, prefab, theo ảnh demo
 ├─ Nối logic    script, gán reference, nối dữ liệu
 └─ Animation    show, hide, hiệu ứng (skill unity-ui-animation)
```

## 1. Mở luồng

1. Xác định project hiện tại. Đổi tên chat này thành **UI_Skill**.
2. Tìm phiên cũ của đúng project (xác minh đường dẫn project và nội dung, không chỉ dựa vào tên). Có thì dùng lại, không mở trùng.
3. Kiểm tra skill `unity-ui-animation`. Chưa cài thì hỏi người dùng có muốn cài từ https://github.com/vuddl/UI_Unity_Animation không; chỉ cài khi họ đồng ý. Họ từ chối thì phiên Animation làm theo mục 10 của `assembly-guide.md`.
4. Mở ngay cả ba phiên con, cùng thư mục project (Unity chỉ mở được một project, không dùng worktree). Brief mỗi phiên gồm: vai, file này, đường dẫn project, phiên nhận báo cáo là UI_Skill.

Claude (app desktop): đổi tên và mở phiên bằng công cụ quản lý phiên, nhắn bằng SendMessage. Người dùng gõ `/unity-ui` nghĩa là họ yêu cầu mở ba phiên này.

## 2. Quét cấu trúc UI (trước khi ghép)

1. **Ghép UI** quét: UI Manager và lớp UI gốc, cách mở/đóng màn, luồng show/hide, backdrop, safe area, prefab dùng chung, font, bộ sprite và 9-slice, quy ước tên, cách code giữ reference. Ghi vào `Docs/UI/ui-project-map.md` của project.
2. **Nối logic** và **Animation** đọc bản đó, đối chiếu với code, bổ sung hoặc phản biện.
3. Ba phiên khớp nhau thì báo UI_Skill; UI_Skill báo người dùng tóm tắt. Chưa khớp thì chưa ghép.
4. Lần sau đọc lại file này và chỉ quét phần đã đổi.

## 2b. Chốt cách làm chữ (phiên Ghép UI hỏi một lần khi cài đặt)

Quét project trước, chỉ hỏi phần chưa rõ. Ghi kết quả vào `Docs/UI/ui-project-map.md`; các màn sau dùng lại, không hỏi lại. Chưa mở luồng nhiều phiên thì vẫn hỏi bước này trước khi ghép màn đầu tiên.

**Câu 1: mặc định dùng chữ gì?** `TMP` hoặc `Text thường (legacy)`. Chọn TMP mà project chưa có (package hoặc TMP Essentials): yêu cầu cài, người dùng đồng ý mới cài, biên dịch sạch rồi mới tiếp.

**Câu 2: chữ có outline, drop shadow thì dùng gì?** Tìm package hiệu ứng chữ/UI (UIFX, UIEffect…) trong `Packages/`, `Assets/`.

- **Tìm thấy:** báo "đã có package X" và dùng nó.
- **Không tìm thấy:** hỏi người dùng có package không.
  - **Có, trong project rồi:** người dùng báo tên hoặc đường dẫn; kiểm tra lại rồi dùng.
  - **Có, kéo package vào:** người dùng tự kéo vào; chờ họ báo xong và biên dịch sạch.
  - **Không có:** chọn một trong ba:
    1. Bỏ, không dùng outline/shadow.
    2. Tự tạo material TMP có outline/underlay (chỉ khi dùng TMP). Tạo material preset mới theo font, không sửa material gốc và không sửa font asset.
    3. Text thường + component Outline / Shadow. Nếu câu 1 chọn TMP thì chỉ các chữ có hiệu ứng mới dùng Text thường.

## 2c. Chốt safe area (phiên Ghép UI, một lần khi cài đặt)

Quét project trước, chỉ hỏi phần chưa rõ. Ghi kết quả vào `Docs/UI/ui-project-map.md`.

1. **Tìm component SafeArea.** Có thì báo tên, đường dẫn, cách cập nhật và dùng nó. Chưa có thì hỏi: tự viết một component nhỏ, dùng package người dùng đưa, hay không dùng.
2. **Hỏi phạm vi nếu chưa rõ:** hướng màn hình (ngang, dọc, xoay được) và áp dụng cho màn nào.
3. **Khi ghép mỗi màn** (chi tiết ở mục 7 `ui-standard.md`):
   - Nền, dim, hiệu ứng phủ màn và tâm ngắm: toàn Canvas, ngoài SafeArea.
   - HUD, tiêu đề, nút góc, thanh tiền, nút đóng sát mép: trong một `SafeContent` có SafeArea. Không lồng hai SafeArea.
   - Popup nằm giữa màn và không chạm mép: không cần SafeArea.
   - Object trong `SafeContent` neo theo mép của nó; chừa vùng banner quảng cáo nếu có.
4. **Kiểm tra bàn giao:** Device Simulator hoặc Game View với tai thỏ bên trái, bên phải và thanh vuốt dưới. Ghi vào gói bàn giao màn nào dùng SafeArea; màn cố tình bỏ qua thì nêu lý do.

## 3. Chia việc và báo cáo

- Người dùng ra lệnh ở UI_Skill. UI_Skill tự giao cho đúng phiên, việc ghép nhiều vai thì tách theo vai.
- Mọi phiên con báo về UI_Skill: bắt đầu, xong, vướng, câu hỏi. Người dùng nói trực tiếp với phiên con thì phiên đó vẫn báo về.
- Câu hỏi cần người dùng quyết: phiên đang nói chuyện với họ hỏi. Một câu chỉ hỏi một nơi.
- Báo cáo của phiên con chưa phải bằng chứng: UI_Skill kiểm tra file hoặc ảnh trước khi báo người dùng là xong.

Thứ tự mỗi màn: **Ghép UI → Nối logic → Animation**. Gói bàn giao: prefab, script, điểm mở/đóng, object và reference mới, chỗ còn lệch demo, việc chưa kiểm tra.

Ghép nhiều màn: chờ **nối xong logic tất cả các màn** rồi mới giao Animation. Animation chụp lại các màn, hỏi show và hide từng màn (Làm như đề xuất / Chỉnh sửa / Không làm), **hỏi hết rồi làm một lượt**.

## 4. Lượt dùng Unity

Mỗi lúc chỉ một phiên giữ lượt. Phải có lượt mới được: vào hoặc thoát Play, biên dịch, sửa prefab, scene hoặc import.

- Xin lượt ở UI_Skill kèm việc sẽ làm. Xong thì trả lượt và báo đã lưu gì.
- Trước khi sửa: kiểm tra Play, biên dịch, prefab/scene đang mở có thay đổi chưa lưu. Có thì hỏi người dùng, không tự thoát Play, không ghi đè.
- Layout chỉ Ghép UI sửa; script chỉ Nối logic sửa (phần trình diễn animation do Animation sửa). Cần sửa phần của vai khác thì nhắn phiên đó.
- Scene báo dirty mà không rõ lý do: lưu bản sao rồi so với file trước khi kết luận có thay đổi.
- Không sửa dữ liệu save của người dùng khi thử. Buộc phải đổi thì sao lưu trước, trả lại và báo.

## 5. Codex và môi trường không mở được phiên con

Dùng đủ 4 chat: UI_Skill, Ghép UI, Nối logic, Animation. Ba chat báo về UI_Skill qua file chung.

1. Người dùng mở 4 chat và gọi `/unity-ui` kèm vai: `tổng`, `ghép`, `logic`, `anim`.
2. Bảng chung: `Docs/UI/ui-skill-board.md` trong project, do chat tổng tạo. Gồm: lượt Unity đang thuộc ai, việc đang giao, báo cáo.
3. Mỗi chat đọc bảng **đầu mỗi lượt** và chỉ **thêm dòng** ở cuối, dạng: `[giờ] [vai] [giao | bắt đầu | xong | vướng | xin lượt | trả lượt] nội dung`. Không sửa dòng của chat khác.
4. Chat tổng không tự nhận được tin: nó đọc bảng mỗi khi người dùng nhắn. Muốn chat khác chạy tiếp thì người dùng sang chat đó nhắn "tiếp".
5. Môi trường chỉ có một chat: làm tuần tự các vai trong chat đó.
