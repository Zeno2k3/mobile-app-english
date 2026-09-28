# English101 — Hướng dẫn cho Agent (Claude Code)

> Ngày: 2026-09-28 · Claude Code tự nạp file này mỗi phiên làm việc trong repo. Mục đích: biết đọc tài liệu nào trước khi làm việc gì, và luật nào không được phá — thay vì đoán hoặc đọc lại toàn bộ tài liệu mỗi lần. Tài liệu gốc: [PRD](../docs/PRD/PRD.md) · [UI/UX Flow](../docs/PRD/UI-UX-FLOWS.md) · [Kiến trúc](../docs/spec/ARCHITECTURE.md) · [Quy chuẩn code](../docs/spec/CODING-STANDARDS.md) · [Kế hoạch Sprint](../docs/PRD/SPRINT-PLAN.md)

## 0. Bối cảnh hiện tại (cập nhật khi sprint đổi)

- **Trạng thái:** trước Sprint 0 (Sprint 0 chạy 2026-10-05 → 2026-10-16). Repo hiện **chỉ có tài liệu** trong `docs/`; chưa có `apps/`, `packages/`, `pubspec.yaml` gốc, CI, hay bất kỳ dòng Dart nào. Cấu trúc monorepo mô tả ở [ARCHITECTURE.md §3](../docs/spec/ARCHITECTURE.md#3-monorepo-và-module) là đích cần dựng ở Sprint 0, **không phải hiện trạng** — luôn kiểm tra (`ls`, `find`) trước khi giả định một file/thư mục đã tồn tại.
- **Chủ dự án:** một người, kiêm product owner kiêm dev duy nhất, dùng Linux, giao tiếp bằng tiếng Việt. Không có team để hỏi — khi cần quyết định nghiệp vụ hoặc điều PRD chưa nói rõ, hỏi thẳng chủ dự án thay vì đoán.
- **Ngôn ngữ:** code, tên định danh, comment, thông điệp commit → **tiếng Anh**. Tài liệu sản phẩm (PRD, spec), chuỗi hiển thị trong app, và hội thoại với chủ dự án → **tiếng Việt**. Chi tiết: [CODING-STANDARDS.md §3](../docs/spec/CODING-STANDARDS.md#3-quy-ước-đặt-tên).

## 1. Đọc gì trước khi làm gì

Không cần đọc cả 5 tài liệu cho mọi việc — chọn đúng cái theo bảng, đọc phần liên quan, rồi làm:

| Định làm gì | Đọc trước |
| --- | --- |
| Đổi/thêm use case, quy tắc nghiệp vụ (BR-*), phạm vi bản phát hành, câu hỏi pháp lý | [docs/PRD/PRD.md](../docs/PRD/PRD.md) |
| Đổi/thêm màn hình, luồng điều hướng, copy, design token | [docs/PRD/UI-UX-FLOWS.md](../docs/PRD/UI-UX-FLOWS.md) |
| Tạo package, đổi phụ thuộc, đổi mô hình dữ liệu/bảng, đổi luồng runtime | [docs/spec/ARCHITECTURE.md](../docs/spec/ARCHITECTURE.md) |
| Viết/sửa code Dart/Flutter, đặt tên, viết test, mở PR, sửa CI | [docs/spec/CODING-STANDARDS.md](../docs/spec/CODING-STANDARDS.md) |
| Không rõ đang ở sprint nào, DoD sprint hiện tại là gì | [docs/PRD/SPRINT-PLAN.md](../docs/PRD/SPRINT-PLAN.md) |

5 file trên là **nguồn chính thức**. Bản "English101 PRD" trên Claude Doc chỉ là bản mirror để bình luận/chia sẻ — sửa xong ở đâu thì đồng bộ lại chỗ kia ([docs/README.md](../docs/README.md)).

## 2. Luật cứng — không có ngoại lệ

Áp dụng ngay khi bắt đầu có code (Sprint 0 trở đi). Đây là tóm tắt phần CI sẽ chặn; xem số mục trong [CODING-STANDARDS.md](../docs/spec/CODING-STANDARDS.md) để biết đầy đủ lý do trước khi cãi luật.

- **Phụ thuộc (§2):** import qua barrel (`package:domain/domain.dart`), không đụng vào `src/` của package khác. `domain` không phụ thuộc package nào khác trong repo; `design_system` không phụ thuộc `domain`; không package nào phụ thuộc `apps/mobile`.
- **Domain sạch (§4.1):** `domain` không import `flutter`, `dart:io`, `dart:ui`, hay bất kỳ adapter nào. Use case = 1 class, 1 method `call()`, không giữ state. Số luật chơi lấy từ `GameRules` ([PRD §7](../docs/PRD/PRD.md#7-quy-tắc-nghiệp-vụ)), không hard-code.
- **Lỗi (§4.2):** không ném exception qua ranh giới lớp — adapter bắt exception và đổi thành `Failure`; use case trả `Result`. Không catch rỗng; mọi `// ignore:` phải kèm lý do trên cùng dòng.
- **Bất đồng bộ & thời gian (§4.4):** việc nặng (parse sách, sinh bài, OCR) chạy trong isolate; ghi nhiều bảng cùng lúc luôn qua `UnitOfWork`; giờ chỉ lấy qua `Clock` và lưu UTC; cấm `DateTime.now()` ngoài adapter `SystemClock`.
- **Test (§5):** mọi use case có test đường chính + test lỗi; mọi quy tắc nghiệp vụ có test mang mã BR trong tên; mọi bug fix kèm test tái hiện lỗi đó. Test không phụ thuộc thời gian thật, số ngẫu nhiên thật, hay mạng.
- **Pháp lý ([PRD §9](../docs/PRD/PRD.md#9-pháp-lý-bản-quyền-và-dữ-liệu-mở)):** **tuyệt đối không** commit nội dung sách có bản quyền, kể cả để làm fixture/test — chỉ sách phạm vi công cộng (đã bỏ header/footer thương hiệu) hoặc nội dung tự viết, luôn ghi nguồn và giấy phép. Với Duolingo (quyết định 2026-09-28): được phép mô phỏng bố cục, luồng màn hình, tương tác, tông màu và tinh thần âm thanh **rất gần** — nhưng không bao giờ dùng tên "Duolingo", logo, nhân vật Duo, hay lấy nguyên file (font/âm thanh/hình) gốc của họ, và không copy nguyên văn câu chữ. Mascot riêng "Mọt" là ranh giới nhận diện chính còn lại. Bảng chi tiết: [PRD §9.2](../docs/PRD/PRD.md#92-ranh-giới-khi-mô-phỏng-duolingo).

## 3. Trước khi commit / mở PR

- Nhánh: `loại/số-issue-mô-tả-ngắn` (vd. `feat/123-match-pairs`). Commit: Conventional Commits + `Signed-off-by` (`git commit -s`, luật DCO) + scope là tên package/feature. Không force-push `main`.
- PR tối đa khoảng 400 dòng thay đổi (không tính file sinh); lớn hơn thì tách theo lớp hoặc theo bước.
- Trước khi báo "xong": chạy format + analyze + test cho package đã đổi. Khi Melos đã tồn tại, dùng đúng script khai báo trong [CODING-STANDARDS.md §1](../docs/spec/CODING-STANDARDS.md#1-công-cụ-và-môi-trường) (`melos run analyze`, `melos run test:dart`, …) — đừng bịa lệnh khác.
- Đổi hành vi sản phẩm hoặc kiến trúc → cập nhật PRD/spec/ADR liên quan trong cùng PR, không để tài liệu trôi khỏi code. Checklist review đầy đủ: [CODING-STANDARDS.md §6](../docs/spec/CODING-STANDARDS.md#6-git-và-pull-request); Definition of Done: [CODING-STANDARDS.md §8](../docs/spec/CODING-STANDARDS.md#8-definition-of-ready-và-definition-of-done).

## 4. Câu hỏi mở — không tự quyết thay chủ dự án

[PRD §11](../docs/PRD/PRD.md#11-câu-hỏi-mở) có 10 câu chưa chốt (Q1–Q10, hạn chốt trải từ 2026-10-05 đến 2027-03-12): giấy phép mã nguồn, tên sản phẩm/mascot, Android-only hay có iOS ở v0.1, sách mẫu đưa vào repo, nguồn từ điển Anh–Việt, tim mặc định bật/tắt, giờ bắt đầu ngày học, kênh phát hành beta, dịch vụ báo crash, nhà cung cấp AI cho v0.2. Mỗi câu đã có sẵn "đề xuất" — dùng đề xuất đó làm mặc định để không bị chặn việc, nhưng nếu một task đụng trực tiếp vào quyết định chưa chốt thì **nói rõ điều đó với chủ dự án**, đừng âm thầm coi như đã chốt.

## 5. Mở rộng file này

File này cố tình ngắn để không tốn nhiều ngữ cảnh mỗi phiên. Khi Sprint 0 dựng xong khung monorepo, cân nhắc tách luật riêng theo package (vd. domain purity chỉ áp cho `packages/domain/**`) sang `.claude/rules/*.md` thay vì làm file này phình to.
