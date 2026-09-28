# English101

App học tiếng Anh mã nguồn mở, chạy offline, biến sách bạn sở hữu (EPUB, PDF, TXT, Markdown) thành một lộ trình học kiểu Duolingo. Từ nào bạn bôi đen khi đọc sẽ vào kho từ cá nhân và được ôn lại đúng lúc bằng FSRS.

- **Mô hình:** tự mang sách (bring-your-own-book) — app không kèm, không bán nội dung sách; mọi xử lý chạy trên máy bạn.
- **Nền tảng:** Flutter, offline-first, Clean Architecture trong một Dart pub workspace.
- **Trạng thái:** đang ở giai đoạn lập kế hoạch/spike (Sprint 0), beta cá nhân v0.1 dự kiến 2027-03-12.

## Tài liệu

Toàn bộ PRD, luồng UI/UX, kiến trúc, quy chuẩn code và kế hoạch sprint nằm trong [docs/](docs/README.md):

| Tài liệu | Nội dung |
| --- | --- |
| [docs/PRD/PRD.md](docs/PRD/PRD.md) | Mục tiêu, persona, use case, quy tắc nghiệp vụ, pháp lý, rủi ro |
| [docs/PRD/UI-UX-FLOWS.md](docs/PRD/UI-UX-FLOWS.md) | Luồng màn hình, design system, microcopy |
| [docs/PRD/SPRINT-PLAN.md](docs/PRD/SPRINT-PLAN.md) | Kế hoạch 11 sprint, DoD từng sprint |
| [docs/spec/ARCHITECTURE.md](docs/spec/ARCHITECTURE.md) | ADR, Clean Architecture, monorepo, tech stack |
| [docs/spec/CODING-STANDARDS.md](docs/spec/CODING-STANDARDS.md) | Quy chuẩn code, Git/PR, CI/CD, đóng góp mã nguồn mở |

## Đóng góp

Dự án chưa mở CONTRIBUTING.md chính thức (đang ở Sprint 0). Trước khi đóng góp, đọc [docs/spec/CODING-STANDARDS.md](docs/spec/CODING-STANDARDS.md) và mở issue để trao đổi trước khi gửi PR lớn.

## Giấy phép

Chưa chốt (xem câu hỏi mở Q1 trong [PRD](docs/PRD/PRD.md#11-câu-hỏi-mở)); đề xuất hiện tại là Apache-2.0.
