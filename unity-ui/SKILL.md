---
name: unity-ui
description: Quy chuẩn và quy trình dựng / ghép UI Unity uGUI theo ảnh demo (Canvas, prefab, screen, popup, HUD, UIManager, script UI). Dùng khi tạo, sửa, ghép hoặc review UI trong project Unity có Assets/ và ProjectSettings/, nhất là khi ghép màn theo ảnh demo, cắt 9-slice, căn chữ TMP, tạo prefab tái dùng, danh sách cuộn. Gõ /unity-ui để mở phiên tổng UI_Skill cùng ba phiên Ghép UI, Nối logic, Animation. Làm animation thì ưu tiên skill unity-ui-animation nếu đã cài.
---

# Unity UI: chuẩn dựng + ghép theo demo

Dùng cho mọi project Unity uGUI, với Claude Code, Codex hay agent khác. Tài liệu trong `references/`:

| File | Đọc khi |
|---|---|
| `references/ui-standard.md` | Luôn đọc trước khi dựng, sửa hoặc review UI: Canvas, anchor, LayoutGroup, code auto-layout chỉ chạy trong OnEnable, UIManager, checklist bàn giao (mục 15). |
| `references/assembly-guide.md` | Khi ghép màn theo ảnh demo: đo khớp pixel, sprite và 9-slice, font, prefab, danh sách cuộn, quy trình an toàn, animation show/hide (mục 10). |
| `references/session-flow.md` | Khi người dùng gõ `/unity-ui`, hoặc phiên này được giao vai Ghép UI, Nối logic, Animation: mở phiên, chia việc, báo cáo, lượt dùng Unity, cách làm ở Codex. |
| `references/example-as92.md` | Chỉ để tham khảo cách áp dụng vào một project thật; không phải quy tắc. |

## Khi người dùng gõ /unity-ui

Chat chưa có vai: trở thành **UI_Skill** và mở luồng theo `references/session-flow.md`. Chat đã có vai (được giao trong brief hoặc người dùng ghi kèm lệnh): làm đúng vai đó, không mở thêm phiên.

## Ba quy tắc cứng

1. **Không để ảnh bẹp, méo.** Ảnh Simple giữ đúng tỉ lệ sprite. Ảnh Sliced phải có khung ≥ tổng border ÷ PPUM.
2. **Không chắc thì hỏi người làm trước**, đưa ra 2–3 phương án kèm khuyến nghị. Ví dụ những chỗ phải hỏi: layout hoặc chức năng đặc biệt, cơ chế cuộn lạ, nút thừa/thiếu so với demo, logic khôi phục hay viết mới.
3. **Khớp demo sát nhất có thể**: đo từ ảnh demo, không ước lượng bằng mắt.

## Trình tự làm việc

1. Đọc `references/ui-standard.md` (và `assembly-guide.md` nếu ghép theo demo).
2. Đọc mọi ảnh demo. Quét sprite của bộ UI. Tìm màn đã có cùng cấu trúc để tái dùng.
3. Hỏi những điểm chưa rõ trước khi dựng. Chưa chốt loại chữ và cách làm outline/shadow thì hỏi theo mục 2b, chưa chốt safe area thì theo mục 2c `session-flow.md`.
4. Dựng hoặc ghép theo checklist của `assembly-guide.md`, từng bước nhỏ, mỗi bước lưu một lần.
5. Kiểm tra:
   - Phủ ảnh demo lên màn để so.
   - Rà soát ảnh méo.
   - Không mất script hay reference.
   - Không còn object thừa.
   - Safe area đúng mục 2c `session-flow.md`.
   - Đi hết checklist mục 15 của `ui-standard.md`.
6. Animation show/hide: đã cài `unity-ui-animation` thì dùng skill đó (trong luồng nhiều phiên do phiên Animation làm). Chưa cài: dùng luồng UI có sẵn của project. Hỏi người làm muốn animation thế nào và đề xuất 2–3 phương án kèm khuyến nghị trước khi làm (mục 10 `assembly-guide.md`).
7. Báo lại: đã làm gì, chỗ còn lệch và lý do, câu hỏi còn mở.

## Ràng buộc chung

- Không sửa `.prefab`, `.unity`, `.asset` bằng công cụ text. Chỉ sửa qua Unity Editor API hoặc Unity MCP.
- Người dùng đang mở prefab và còn thay đổi chưa lưu: chờ họ lưu rồi mới sửa.
- Logic màn (script, nối dữ liệu) tách khỏi việc ghép layout. Ghi lại danh sách object và reference mới cho người nối logic.
- Không commit nếu chưa được yêu cầu.
