# English101 — Quy chuẩn code

> Phiên bản: 1.0 · Ngày: 2026-09-28 · Tham chiếu: [Kiến trúc](ARCHITECTURE.md) · [PRD](../PRD/PRD.md) · [Kế hoạch Sprint](../PRD/SPRINT-PLAN.md)

Mọi PR phải qua format, analyze, test và review theo các quy tắc dưới đây. Quy tắc phụ thuộc giữa các lớp được ép bằng pubspec, không chỉ bằng quy ước.

---

## 1. Công cụ và môi trường

Mọi người dùng cùng phiên bản Flutter (ghim bằng FVM) và cùng bộ lệnh Melos, nên "máy tôi chạy được" cũng có nghĩa là CI chạy được.

| Công cụ | Quy định |
| --- | --- |
| Flutter | Kênh stable, phiên bản ghim trong `.fvmrc`; CI đọc cùng file |
| Dart SDK | ≥ 3.9 (Melos 8 yêu cầu); mọi pubspec khai báo `environment: sdk: ^3.9.0` và `resolution: workspace` |
| Workspace | `pubspec.yaml` gốc liệt kê package dưới khóa `workspace` và cấu hình Melos dưới khóa `melos` |
| Sinh mã | build_runner cho freezed, json_serializable, riverpod_generator, drift_dev, go_router_builder; file sinh ra được commit và đánh dấu `linguist-generated` |
| Lint | very_good_analysis + riverpod_lint; CI chạy analyze với `--fatal-infos` |
| Định dạng | `dart format` mặc định (80 cột); bật format khi lưu |
| Git hook | lefthook: pre-commit chạy format và analyze cho package thay đổi; commit-msg kiểm Conventional Commits |
| Editor | Commit `.vscode/settings.json` và `extensions.json` (Dart, Flutter); Android Studio dùng cùng `analysis_options.yaml` |

Phác thảo `pubspec.yaml` gốc (tên script là quy ước của dự án; kiểm lại cú pháp filter với tài liệu Melos 8 ở S0):

```yaml
name: english101_workspace
publish_to: none
environment:
  sdk: ^3.9.0
workspace:
  - apps/mobile
  - packages/domain
  - packages/nlp
  - packages/course_builder
  - packages/ingestion
  - packages/srs
  - packages/dictionary
  - packages/data
  - packages/platform_services
  - packages/design_system
  - packages/test_support
dev_dependencies:
  melos: ^8.9.0
melos:
  scripts:
    analyze: melos exec -- flutter analyze --fatal-infos
    gen: melos exec --depends-on=build_runner -- dart run build_runner build -d
    test:dart: melos exec --no-flutter --dir-exists=test -- dart test --coverage=coverage
    test:flutter: melos exec --flutter --dir-exists=test -- flutter test --coverage
    check:deps: dart run tools/check_deps.dart
    check:licenses: dart run tools/check_licenses.dart
```

Thiết lập máy mới trong ≤ 30 phút (mục tiêu G5): cài FVM → `fvm install` → `dart pub get` ở thư mục gốc → `melos run gen` → `melos run test:dart` → chạy app trên emulator Android.

---

## 2. Cấu trúc thư mục và quy tắc phụ thuộc

Cây thư mục và bảng phụ thuộc giữa các package nằm ở [Kiến trúc, mục 3](ARCHITECTURE.md#3-monorepo-và-module); mục này là các luật CI sẽ chặn.

1. **Import qua barrel:** chỉ import API công khai của package khác (`package:domain/domain.dart`), không bao giờ import thư mục `src/` của package khác (lint `implementation_imports`).
2. **Không có cạnh cấm:** script `check:deps` đọc mọi pubspec và fail khi `domain` phụ thuộc package trong repo, khi `design_system` phụ thuộc `domain`, hoặc khi package nào phụ thuộc `apps/mobile`.
3. **Feature không dùng ruột của nhau:** feature A không import file của feature B; dùng chung qua domain hoặc `apps/mobile/lib/app/shared`; chuyển màn qua route đã khai báo.
4. **Mỗi file một việc:** một class công khai chính mỗi file, tối đa khoảng 400 dòng; vượt thì tách.
5. **Test đặt song song:** `test/` phản chiếu đúng đường dẫn trong `lib/src/` của cùng package.
6. **Tài nguyên:** font, âm thanh, Rive nằm trong `design_system/assets`; ảnh riêng của một feature nằm trong assets của app theo tên feature.

Ví dụ import đúng và sai trong `apps/mobile`:

```dart
// Đúng
import 'package:domain/domain.dart';
import 'package:design_system/design_system.dart';

// Sai: vào ruột package khác
import 'package:domain/src/vocabulary/rules/new_card_quota.dart';

// Sai: controller gọi thẳng adapter thay vì use case
import 'package:data/src/repositories/drift_vocab_repository.dart';
```

---

## 3. Quy ước đặt tên

Tên theo Effective Dart, cộng hậu tố theo vai trò để đọc tên là biết file thuộc lớp nào. Code, tên và comment viết bằng tiếng Anh; tài liệu sản phẩm và chuỗi giao diện có tiếng Việt.

| Đối tượng | Quy ước | Ví dụ |
| --- | --- | --- |
| File, thư mục | snake_case | `lesson_controller.dart` |
| Class, enum, typedef | UpperCamelCase | `LessonSession` |
| Biến, hàm, tham số, hằng số | lowerCamelCase (không dùng SCREAMING_CASE) | `dueCards`, `maxHearts` |
| Use case | Động từ + danh từ + UseCase, một phương thức `call` | `SaveWordUseCase` |
| Port | Danh từ vai trò, không tiền tố I | `BookRepository`, `TtsService` |
| Adapter | Công nghệ + tên port | `DriftBookRepository`, `FlutterTtsService` |
| Controller | Tên màn hình + Controller | `LessonController` |
| Provider | Sinh tự động từ tên class hoặc hàm `@riverpod` | `lessonControllerProvider` |
| Widget | Hậu tố Screen, View, Sheet, Dialog, Tile, Button | `LessonScreen`, `FeedbackSheet` |
| Sự kiện miền | Thì quá khứ | `LessonCompleted`, `WordSaved` |
| Lỗi | Hậu tố Failure, sealed theo context | `BookParseFailure` |
| Bảng DB | snake_case số nhiều | `vocab_entries` |
| Khóa ARB | lowerCamelCase: feature + vị trí + ý nghĩa | `lessonCheckButton` |
| Test | Cùng đường dẫn với file nguồn + `_test.dart`; tên test mô tả hành vi, test luật ghi mã BR | `'BR-STK-03 uses a freeze when a day is missed'` |
| Nhánh, commit | Xem mục 6 | `feat/123-match-pairs` |

---

## 4. Quy tắc viết code

Mỗi quy tắc dưới đây có lý do kiến trúc đứng sau; reviewer chỉ cần trỏ đúng số mục khi yêu cầu sửa.

### 4.1 Domain và use case

- Domain không import `package:flutter`, `dart:io`, `dart:ui` hay bất kỳ package adapter nào.
- Use case là một class, một phương thức `Future<Result<T>> call(input)`, không giữ state; phụ thuộc nhận qua constructor dưới dạng port.
- Entity và value object bất biến (freezed); value object tự kiểm hợp lệ khi tạo (`Lemma`, `LearningDay`, `Xp`).
- Mọi con số luật chơi lấy từ `GameRules` ([PRD mục 7](../PRD/PRD.md#7-quy-tắc-nghiệp-vụ)), không viết cứng trong code.

### 4.2 Lỗi

- Không ném exception qua ranh giới lớp: adapter bắt exception và đổi thành `Failure` (sealed class); use case trả `Result`.
- UI đổi `Failure` thành thông điệp đã dịch; lỗi không lường trước được log và hiện màn lỗi chung có nút Thử lại.
- Cấm catch rỗng; mọi dòng `// ignore:` phải kèm lý do trên cùng dòng.

### 4.3 State và UI

- Mỗi màn có một controller (`AsyncNotifier`); widget chỉ `ref.watch` hoặc `select`; điều hướng và snackbar chạy trong `ref.listen`.
- Widget mặc định stateless và `const`; `StatefulWidget` chỉ cho animation hoặc controller cục bộ.
- `build()` không quá khoảng 80 dòng; tách widget con thành class, không dùng hàm trả về `Widget`.
- Không viết cứng màu, cỡ chữ, khoảng cách: dùng token của `design_system`.
- Không viết cứng chuỗi hiển thị: dùng `AppLocalizations`; câu có số dùng ICU plural.
- Mọi nút chỉ có icon phải có nhãn Semantics; vùng chạm ≥ 48 dp; tôn trọng `MediaQuery.disableAnimations` và `textScaler`.

### 4.4 Bất đồng bộ, dữ liệu, thời gian

- Việc nặng (parse, sinh bài, OCR) chạy trong isolate; UI isolate không đọc file lớn đồng bộ.
- `unawaited()` chỉ dùng có chủ đích và kèm comment; mọi `StreamSubscription` được hủy.
- Ghi nhiều bảng cùng lúc luôn qua `UnitOfWork` (một transaction); không có SQL thô ngoài package `data`.
- Thời gian chỉ lấy qua `Clock` và lưu UTC; "hôm nay" chỉ tính qua `LearningDay.of`; cấm `DateTime.now()` ngoài adapter `SystemClock`.

### 4.5 Log, phụ thuộc, tài liệu

- Log qua `Logger` port với 4 cấp fine, info, warning, severe; không log nội dung sách, từ người học lưu hay API key.
- Thêm dependency phải nêu lý do trong PR: giấy phép (mã app chỉ dùng MIT, BSD, Apache), cập nhật trong 12 tháng gần nhất, kích thước.
- API công khai của package có dartdoc; comment giải thích vì sao, không lặp lại code làm gì; TODO viết dạng `// TODO(#123): …` và phải có issue.

Use case mẫu theo đúng các quy tắc trên:

```dart
final class SaveWordUseCase {
  SaveWordUseCase(this._vocab, this._dictionary, this._unitOfWork, this._events, this._clock);

  final VocabRepository _vocab;
  final DictionaryRepository _dictionary;
  final UnitOfWork _unitOfWork;
  final EventBus _events;
  final Clock _clock;

  Future<Result<VocabEntry>> call(SaveWordInput input) {
    return _unitOfWork.run(() async {
      final lemma = Lemma.parse(input.surface); // value object tự kiểm hợp lệ
      final senses = await _dictionary.lookup(lemma.value, pos: input.pos);
      final entry = await _vocab.upsertWithContext(
        lemma: lemma,
        meaning: senses.firstOrNull?.gloss,
        context: input.context, // câu gốc + vị trí trong sách
        now: _clock.now(),
      );
      _events.publish(WordSaved(entryId: entry.id, at: _clock.now()));
      return Result.ok(entry);
    });
  }
}
```

---

## 5. Kiểm thử

Code chưa có test thì chưa xong; các tầng test và công cụ nằm ở [Kiến trúc, mục 10](ARCHITECTURE.md#10-chiến-lược-kiểm-thử), còn đây là luật áp cho từng PR.

| Luật | Chi tiết |
| --- | --- |
| Use case | Ít nhất 1 test đường chính và 1 test lỗi |
| Quy tắc nghiệp vụ | Mỗi BR có test mang mã BR trong tên; luật có nhiều trường hợp viết dạng bảng (table-driven) |
| Sửa lỗi | Mỗi bug fix kèm test tái hiện lỗi đó |
| Độ phủ | domain ≥ 90%; nlp, course_builder, srs, ingestion ≥ 80%; toàn repo ≥ 70%; CI chặn khi giảm quá 1 điểm so với main |
| Xác định | Không `DateTime.now`, không chờ thời gian thật, không mạng; dùng `FakeClock`, `SeededRandom`, repository in-memory |
| Golden | Chỉ cập nhật bằng `--update-goldens` khi đổi UI có chủ đích; PR ghi rõ và đính ảnh trước/sau |
| Fixture | Chỉ sách phạm vi công cộng hoặc tự viết; mỗi thư mục fixture có file ghi nguồn và giấy phép |
| Cấu trúc test | Arrange, Act, Assert; mỗi test một hành vi; group theo use case hoặc widget |
| E2E | 5 flow chính bằng Patrol, chạy mỗi đêm và trước mỗi bản phát hành; story chạm flow nào thì cập nhật flow đó |

Ví dụ test bảng cho luật streak:

```dart
void main() {
  group('BR-STK-03 missed day', () {
    final cases = [
      (streak: 5, freezes: 1, missedDays: 1, expectStreak: 5, expectFreezes: 0),
      (streak: 5, freezes: 0, missedDays: 1, expectStreak: 0, expectFreezes: 0),
      (streak: 5, freezes: 2, missedDays: 3, expectStreak: 0, expectFreezes: 0),
    ];
    for (final c in cases) {
      test('streak ${c.streak}, freezes ${c.freezes}, missed ${c.missedDays}', () {
        final result = closeDays(
          StreakState(current: c.streak, freezes: c.freezes),
          missedDays: c.missedDays,
        );
        expect(result.current, c.expectStreak);
        expect(result.freezes, c.expectFreezes);
      });
    }
  });
}
```

---

## 6. Git và Pull Request

Dự án dùng trunk-based development: `main` luôn phát hành được, nhánh sống không quá 3 ngày, merge bằng squash.

- **Tên nhánh:** `loại/số-issue-mô-tả-ngắn`, ví dụ `feat/123-match-pairs`, `fix/140-streak-timezone`, `docs/88-adr-srs`.
- **Commit:** Conventional Commits, scope là tên package hoặc feature: `feat(lesson): add match pairs exercise`; `fix(habit): keep streak across DST`. Đổi schema DB hoặc API công khai thì thêm footer `BREAKING CHANGE`. Mọi commit có dòng `Signed-off-by` (`git commit -s`).
- **Kích thước PR:** tối đa khoảng 400 dòng thay đổi, không tính file sinh; lớn hơn thì tách theo lớp hoặc theo bước.
- **Tiêu đề PR:** theo Conventional Commits vì squash merge dùng tiêu đề làm commit.
- **Mẫu PR:** Mục đích; Issue liên kết (`Closes #123`); Ảnh hoặc video nếu đổi UI; Cách test; Checklist review.
- **Duyệt:** cần 1 approve của maintainer theo CODEOWNERS và CI xanh. Giai đoạn một người: tự review theo checklist và để PR mở ít nhất tới hôm sau rồi mới merge.
- **Cấm:** force-push lên `main`, commit file bí mật, commit sách có bản quyền.
- **Phiên bản:** SemVer, 0.x trong giai đoạn beta; build number lấy từ số lần chạy CI; CHANGELOG sinh bằng release-please từ commit.

Checklist review:

- [ ] Đúng lớp: không import sai chiều, controller chỉ gọi use case.
- [ ] Có test theo mục 5; test BR mang mã BR.
- [ ] Chuỗi giao diện có đủ vi và en; không chuỗi viết cứng.
- [ ] Accessibility: nhãn, vùng chạm, tương phản, giảm chuyển động.
- [ ] Không lộ bí mật, không log dữ liệu người dùng.
- [ ] Không làm chậm các đường nóng (lộ trình, bài học, tra từ).
- [ ] File sinh đã cập nhật; tài liệu hoặc ADR đã cập nhật nếu hành vi hay kiến trúc đổi.

---

## 7. CI/CD

Sáu workflow trên GitHub Actions; workflow `ci` phải xanh mới được merge và chạy xong dưới 10 phút nhờ cache pub và Flutter SDK.

| Workflow | Kích hoạt | Việc chạy | Cổng chặn |
| --- | --- | --- | --- |
| ci | Mọi PR và push lên main | Kiểm format; chạy build_runner rồi `git diff --exit-code`; analyze `--fatal-infos`; test kèm coverage; check:deps; check:licenses; kiểm tiêu đề PR | Mọi bước xanh; coverage không giảm quá 1 điểm |
| e2e | Hằng đêm, hoặc PR có nhãn `run-e2e` | Patrol trên emulator Android API 26 và API mới nhất | 5 flow chính xanh |
| build | Push lên main | Build APK, báo kích thước, đính kèm artifact | Cảnh báo khi APK tăng trên 10% |
| release | Tag `v*` | Build AAB và APK đã ký; tạo GitHub Release; đẩy lên Play Console (internal hoặc closed); iOS qua Codemagic khi có tài khoản Apple | Release DoD ([Kế hoạch Sprint, S10](../PRD/SPRINT-PLAN.md)) |
| release-please | Push lên main | Mở PR tăng version và cập nhật CHANGELOG | Người duyệt merge |
| dependabot | Hằng tuần | Cập nhật package pub và GitHub Actions | CI xanh mới merge |

Keystore Android và tài khoản dịch vụ Play nằm trong GitHub encrypted secrets, không bao giờ nằm trong repo; PR từ fork không được truy cập secrets.

---

## 8. Definition of Ready và Definition of Done

Story chỉ vào sprint khi đạt DoR và chỉ được tính xong khi đạt đủ DoD; DoD riêng của từng sprint nằm trong [Kế hoạch Sprint](../PRD/SPRINT-PLAN.md) và cộng thêm vào danh sách này.

### Definition of Ready (story)

- [ ] Viết theo mẫu "Là …, tôi muốn …, để …" và gắn mã UC.
- [ ] Có tiêu chí chấp nhận Given/When/Then kiểm được.
- [ ] Có tham chiếu màn hình (SCR), flow và mã BR liên quan.
- [ ] Phụ thuộc đã rõ; fixture và dữ liệu test có sẵn.
- [ ] Ước lượng ≤ 8 điểm; lớn hơn thì tách.
- [ ] Không còn câu hỏi mở nào chặn việc làm.

### Definition of Done (story)

- [ ] Code đã merge vào main qua PR, CI xanh.
- [ ] Logic mới có unit test đạt ngưỡng độ phủ; luật mới có test mang mã BR.
- [ ] UI mới có widget test và golden sáng/tối; E2E đã cập nhật nếu chạm flow chính.
- [ ] Tiêu chí chấp nhận đã kiểm trên máy Android thật và emulator.
- [ ] Chuỗi vi và en đầy đủ; accessibility đạt (nhãn, vùng chạm, tương phản).
- [ ] 0 cảnh báo analyzer; không TODO nào thiếu issue.
- [ ] Đạt ngân sách hiệu năng liên quan (NFR-02 đến NFR-06).
- [ ] PRD, spec hoặc ADR đã cập nhật nếu hành vi hay kiến trúc thay đổi.
- [ ] PR có video hoặc ảnh demo.
- [ ] Với tính năng người dùng thấy: chủ dự án đã dùng thử trên máy mình ít nhất 1 ngày.

### Definition of Done (sprint)

- [ ] Mọi story cam kết đạt DoD, hoặc được chuyển sprint kèm lý do ghi lại.
- [ ] Bản APK của sprint được tag `v0.0.N` và cài được trên máy thật.
- [ ] Có video demo khoảng 3 phút và ghi chú retro trong GitHub Discussions.
- [ ] Backlog của sprint sau đã được refine đến mức đạt DoR.

---

## 9. Đóng góp mã nguồn mở

Repo công khai cần 6 file quản trị, một bộ nhãn cố định và cam kết phản hồi để người mới đóng góp lần đầu mà không phải hỏi.

| File | Nội dung |
| --- | --- |
| LICENSE | Apache-2.0 (sau khi chốt Q1); gói dữ liệu có LICENSE riêng |
| README.md | Mục tiêu, ảnh chụp, tải bản beta, cách chạy, liên kết tài liệu |
| CONTRIBUTING.md | Thiết lập ≤ 30 phút, quy trình issue → PR, liên kết tới tài liệu này |
| CODE_OF_CONDUCT.md | Contributor Covenant 2.1 |
| SECURITY.md | Báo lỗ hổng qua email riêng, phản hồi trong 7 ngày; không mở issue công khai |
| GOVERNANCE.md | Chủ dự án quyết định cuối; contributor thành maintainer sau 5 PR được merge và được mời |

- **DCO:** mọi commit có `Signed-off-by`; GitHub App DCO kiểm tự động, không cần ký CLA.
- **Mẫu issue:** Lỗi (bước tái hiện, thiết bị, phiên bản), Đề xuất tính năng (vấn đề người dùng, UC liên quan), Định dạng sách lỗi (chỉ mô tả cấu trúc, không đính kèm sách), Ý tưởng dạng bài.
- **Nhãn:** `type/bug`, `type/feature`, `type/docs`; `area/ingestion`, `area/lesson`, `area/srs`, `area/ui`, `area/habit`; `priority/p0` đến `p3`; `good first issue`; `help wanted`; `status/needs-triage`.
- **Kênh:** GitHub Discussions cho hỏi đáp và ý tưởng; GitHub Projects là bảng sprint công khai.
- **Cam kết phản hồi:** phân loại issue trong 7 ngày; review PR trong 5 ngày làm việc.
- **Nội dung:** không commit sách có bản quyền; fixture và gói dữ liệu phải ghi nguồn và giấy phép; bản dịch giao diện đóng góp theo giấy phép của mã.

Gợi ý good first issue cho cộng đồng: thêm câu nhắc nhở vào kho nội dung, viết test bảng cho một BR, hỗ trợ file TXT mã hóa Windows-1252, thêm `StructureDetector` cho mẫu tiêu đề "Lesson 1.2", dịch chuỗi ARB sang tiếng Anh.
