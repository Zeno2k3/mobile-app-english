# Báo cáo: Xây dựng app học tiếng Anh theo giáo trình "Grammar in Use", offline, hình thành thói quen

Có một khoảng trống thật trên thị trường: chưa app nào vừa bám giáo trình Grammar in Use theo lộ trình kiểu Duolingo, vừa cho bôi đen từ để đưa vào kho từ vựng có lặp lại ngắt quãng (FSRS), lại chạy offline trên cả iPad lẫn Android. Tuy vậy, khoảng trống này có một phần là do vướng bản quyền. Vì thế, hãy tự làm bản cá nhân bằng **Flutter + SQLite (Drift) + FSRS**, theo mô hình "tự mang sách của bạn" (bring-your-own-book). Nếu muốn phát hành công khai thì phải đổi sang nội dung tự viết, bám khung ngữ pháp CEFR.

## TL;DR

- **Khoảng trống có thật nhưng hẹp:** App chính thức của Cambridge (English Grammar in Use app, 145 unit) đã bị gỡ khỏi Google Play ngày 6/10/2022. eBook trên Cambridge One cho highlight và ghi chú nhưng không có lộ trình, streak hay SRS từ vựng. LingQ, eJOY, Readlang, Migaku có tính năng "chạm từ → lưu → ôn SRS" nhưng không bám giáo trình ngữ pháp. Chưa sản phẩm nào gộp được cả 7 yêu cầu của bạn.
- **Pháp lý quyết định kiến trúc:** Dùng cho cá nhân với sách bạn đã mua hợp pháp thì rủi ro thấp. Muốn phát hành công khai mà có trang sách hay bài tập Cambridge thì cần giấy phép (qua PLSclear hoặc rights@cambridge.org). Cách thực tế hơn: (1) người dùng tự nhập PDF của họ, app không phân phối nội dung; (2) viết nội dung gốc bám English Grammar Profile / CEFR-J. Nên hỏi luật sư trước khi phát hành.
- **Công nghệ và phương pháp:** Flutter + Drift/SQLite + gói `fsrs` (Dart) + pdfrx/Syncfusion + TTS của hệ điều hành + thông báo cục bộ. Mỗi bài học theo vòng: *đọc giải thích → tự nhớ lại (retrieval) → bài tập có phản hồi → khai thác từ → ôn ngắt quãng*. Mục tiêu thói quen là "5 phút mỗi ngày", có streak freeze. Thời gian dự kiến cho MVP cá nhân khoảng 6–10 tuần làm bán thời gian.

---

## 1. Key Findings (Phát hiện chính)

1. **Cambridge từng làm đúng ý tưởng "sách → app" nhưng đã dừng.** App "English Grammar in Use" chính thức (Raymond Murphy) có gói Starter miễn phí 6 unit, mua thêm từng unit trong app, tổng 145 unit, có chấm điểm tự động và audio.\[1\]\[2\] App bị gỡ khỏi Google Play ngày 6/10/2022 sau khoảng 2,2 triệu lượt tải; bản cuối là 1.11.40, cập nhật 24/1/2020. Nhiều đánh giá phàn nàn giá cao và lỗi mua hàng.\[3\] Hiện Cambridge dồn vào **interactive eBook trên Cambridge One**. eBook này có giải thích, bài tập, audio, bookmark, highlight và ghi chú, nhưng **không có lộ trình kiểu game, streak hay kho từ vựng SRS**.\[4\]\[5\]
2. **Offline trên tablet là điểm yếu của Cambridge One.** Trang trợ giúp Cambridge One nói eBook chỉ dùng offline được qua Cambridge One Desktop App trên Windows, macOS, Linux, và "app can only be used with computers, not with tablets or mobile devices".\[6\] Các trang catalogue cũ hơn (bản 4th edition, Mỹ) lại nói eBook "can be accessed online or downloaded to iPads and Android tablets".\[7\] Hai nguồn này mâu thuẫn, có thể do chính sách thay đổi theo thời gian. Hãy thử thực tế với mã sách của bạn.
3. **Các app "đọc và khai thác từ" (LingQ, eJOY, Readlang, Migaku) làm tốt vế từ vựng nhưng không bám giáo trình.**\[8\] LingQ nhập được EPUB, PDF, DOCX, TXT, MOBI rồi biến thành bài học có từ màu xanh để chạm và lưu.\[9\]\[10\] Tuy nhiên đó là đọc tự do, không có bài tập ngữ pháp theo unit.
4. **Bằng chứng khoa học rất rõ:** kiểm tra/nhớ lại chủ động và lặp lại ngắt quãng là hai kỹ thuật được Dunlosky et al. (2013) xếp "high utility". Highlight và đọc lại bị xếp "low utility".\[11\] Hệ quả thiết kế quan trọng: **bôi đen từ chỉ là bước thu thập; việc học thật diễn ra ở bước ôn chủ động.**
5. **FSRS vượt SM-2 về độ chính xác dự đoán.** Benchmark của open-spaced-repetition cho thấy FSRS-6 dự đoán tốt hơn SM-2 ở 99,6% người dùng (theo log loss), và Anki đã tích hợp FSRS như một tính năng tuỳ chọn từ bản 23.10 (2023). Đã có sẵn gói `fsrs` cho Dart/Flutter.\[12\]
6. **Thói quen mất thời gian hơn bạn nghĩ nhưng lại dễ tha thứ.** Theo Lally et al. (2010), trung vị là 66 ngày để hành vi đạt tự động hoá, dao động 18–254 ngày, và bỏ một ngày không làm hỏng quá trình.\[13\]

---

## 2. Phần A: Các app hiện có so với yêu cầu

### 2.1 Bảng so sánh

Chú thích: ✅ có · ⚠️ một phần hoặc có điều kiện · ❌ không có

| App | Bám giáo trình Grammar in Use | Lộ trình/streak kiểu Duolingo | Bôi đen/chạm từ để lưu | SRS từ vựng | Offline iPad + Android | Ghi chú |
|---|---|---|---|---|---|---|
| **English Grammar in Use app (Cambridge, cũ)** | ✅ (145 unit, B1–B2) | ❌ | ❌ | ❌ | ⚠️ (đã gỡ khỏi Google Play từ 10/2022) | Có audio, chấm tự động; lỗi mua in-app |
| **Advanced Grammar in Use Activities (Cambridge)** | ⚠️ (170 hoạt động, hơn 1.500 câu hỏi, 100 unit; *chỉ là bài tập thêm, không phải cả sách*) | ❌ | ❌ (có glossary chạm từ để xem nghĩa) | ❌ | ⚠️ | Có ghi âm để so phát âm |\[14\]
| **Cambridge One interactive eBook** | ✅ (nội dung đầy đủ) | ❌ | ⚠️ highlight/ghi chú, không xuất ra kho từ | ❌ | ⚠️ offline chính thức chỉ trên máy tính | Đi kèm mã trong sách |
| **LingQ** | ⚠️ (tự nhập PDF/EPUB) | ⚠️ (có streak, thống kê) | ✅ | ✅ (SRS cơ bản) | ⚠️ cần tài khoản/đồng bộ | Gói miễn phí giới hạn số từ lưu |
| **Readlang** | ❌ (chủ yếu đọc web) | ❌ | ✅ | ✅ flashcard | ❌ (web/extension) | Nhẹ, rẻ |\[15\]
| **Migaku** | ❌ | ⚠️ | ✅ | ✅ | ⚠️ | Mạnh về video/Netflix, trả phí |
| **Language Reactor** | ❌ | ❌ | ✅ (phụ đề) | ⚠️ | ❌ (extension Chrome) | Không dùng để đọc sách |
| **Anki / AnkiDroid / AnkiMobile** | ⚠️ (tự làm deck) | ❌ | ❌ | ✅ (FSRS) | ✅ | Cực mạnh ở SRS, UX thô |
| **Kindle Vocabulary Builder / KOReader** | ⚠️ (nếu có ebook) | ❌ | ✅ (tra từ tự lưu) | ❌/⚠️ | ✅ | Kindle không hợp với PDF bố cục phức tạp |
| **Duolingo** | ❌ (giáo trình riêng) | ✅ (chuẩn mực) | ❌ | ⚠️ (tự động, ẩn) | ⚠️ | Mẫu hình tốt nhất về thói quen |
| **Busuu / Memrise / Lingvist / Clozemaster** | ❌ | ⚠️–✅ | ❌ | ✅ | ⚠️ | Clozemaster: cloze theo tần suất |
| **ELSA Speak** | ❌ | ⚠️ | ❌ | ❌ | ❌ | Chuyên phát âm |
| **eJOY (App/Reader)** | ❌ (hơn 60 chủ đề ngữ pháp riêng) | ✅ (XP, streak) | ✅ (lưu từ khi đọc tin, sách, PDF, video) | ✅ | ⚠️ (có từ điển offline, video offline) | Gần ý tưởng nhất ở vế từ vựng |\[16\]\[17\]
| **TFlat, Laban Dictionary** | ❌ | ❌ | ⚠️ (lưu từ khi tra) | ⚠️ | ✅ (từ điển offline) | Từ điển là chính |
| **Monkey, Prep** | ❌ (giáo trình riêng: trẻ em / luyện thi) | ⚠️–✅ | ❌ | ⚠️ | ⚠️ | Không phục vụ việc học theo sách |

*Lưu ý:* Tính năng của nhóm app Việt Nam (TFlat, Laban, Monkey, Prep) tôi đánh giá theo mô tả chung trên cửa hàng ứng dụng và hiểu biết phổ biến. Chỉ eJOY được kiểm chứng trực tiếp qua trang App Store/Google Play. Hãy tự kiểm tra lại trước khi dùng bảng này để định vị sản phẩm.

### 2.2 Kết luận về khoảng trống

- **Khoảng trống có thật:** không sản phẩm nào gộp được *(giáo trình ngữ pháp chuẩn) + (lộ trình và streak) + (khai thác từ ngay trên trang học) + (FSRS) + (offline đa nền tảng)*.
- **Vì sao chưa ai lấp:** nội dung Grammar in Use thuộc bản quyền Cambridge. Chính Cambridge đã thử app và rút lui. Bên thứ ba thì không thể phân phối nội dung đó. Nói cách khác, **khoảng trống ở "sản phẩm" dễ lấp; khoảng trống ở "nội dung" mới là rào cản.**
- **Cơ hội cho bản công khai:** bán *công cụ* (trình đọc PDF + lộ trình + SRS, người dùng tự nhập sách) hoặc *nội dung gốc* bám CEFR cho người Việt mất gốc, chứ không bán lại nội dung của Murphy.

---

## 3. Phần B: Bản quyền và giấy phép

*Tôi không phải luật sư. Phần này chỉ tổng hợp thông tin thực tế; hãy tham khảo luật sư sở hữu trí tuệ trước khi phát hành công khai.*

### 3.1 Tình trạng pháp lý

- **Dùng cá nhân:** Chuyển PDF hoặc sách bạn *đã mua hợp pháp* thành bài học để tự học, không chia sẻ, thường được coi là rủi ro thấp. Luật Sở hữu trí tuệ Việt Nam có ngoại lệ cho việc tự sao chép một bản để nghiên cứu, học tập cá nhân, không nhằm mục đích thương mại. Việt Nam cũng là thành viên Công ước Berne. Tuy nhiên, điều khoản sử dụng của eBook (DRM, cấm trích xuất) có thể hạn chế thêm. Và lấy PDF lậu thì không được bảo vệ bởi ngoại lệ này.
- **Phát hành công khai:** Hiển thị trang sách, chép lại giải thích, hay "tái cấu trúc" bài tập của Murphy thành bài học trong app đều là sao chép hoặc tạo tác phẩm phái sinh. Việc này **cần giấy phép**. Đổi cách diễn đạt nhẹ vẫn chưa đủ; chính Cambridge ghi rõ "if you make minor changes to the original material, you still need to seek permission".\[18\]

### 3.2 Kênh xin phép Cambridge

- **Trích đoạn (permissions):** Yêu cầu tái sử dụng nội dung sách Cambridge University Press được xử lý qua **PLSclear**.\[19\]
- **Toàn bộ tác phẩm (rights licence):** Giấy phép cho "translation, audio, video, broadcast or electronic reproduction of an entire work" được cấp qua form yêu cầu hoặc email **rights@cambridge.org**. Cambridge đặt mục tiêu phản hồi ban đầu "within no more than two working weeks".\[20\]\[21\]
- **Đánh giá thực tế:** Với một nhà phát triển độc lập, khả năng được cấp phép số hoá toàn bộ sản phẩm bán chạy nhất của họ là thấp, vì Cambridge đang tự khai thác kênh eBook. Cứ gửi yêu cầu cho có hồ sơ, nhưng đừng xây kế hoạch kinh doanh dựa vào đó.

### 3.3 Các lựa chọn thay thế (xếp theo mức khuyến nghị)

| Mô hình | Cách làm | Rủi ro | Nhận xét |
|---|---|---|---|
| **1. Bring-your-own-book (BYOB)** | App không chứa nội dung; người dùng tự nhập PDF/EPUB họ sở hữu; mọi xử lý (OCR, tách unit) chạy trên máy, dữ liệu không rời thiết bị | Thấp–trung bình (giống LingQ cho nhập ebook) | Không quảng cáo "app cho Grammar in Use", không phân phối "template" chứa nội dung Cambridge |
| **2. Nội dung gốc bám CEFR** | Tự viết giải thích và bài tập (có AI hỗ trợ và bạn biên tập) theo danh mục ngữ pháp chuẩn | Thấp | Tài sản thật sự để phát hành công khai; khung ngữ pháp *(danh mục chủ điểm)* không được bảo hộ như cách diễn đạt |
| **3. Xin giấy phép Cambridge** | PLSclear hoặc rights@cambridge.org | Pháp lý thấp nếu được cấp, nhưng khả năng được cấp thấp | Chi phí và thời gian khó đoán |
| **4. Liên kết (deep link)** | App chỉ ghi "Unit 12 trong sách" và người dùng mở sách giấy/eBook song song | Rất thấp | Đơn giản, UX kém hơn |

**Nguồn khung ngữ pháp và từ vựng mở hoặc tham khảo:**
- **English Grammar Profile (EGP, English Profile/Cambridge):** hơn 1.200 "grammatical competence statements" theo cấp CEFR, rút từ Cambridge Learner Corpus. Trang web cho "free access" để tra cứu, nhưng tôi không truy cập được nội dung điều khoản sử dụng.\[22\]\[23\] Hãy coi đây là **tài liệu tham khảo để thiết kế syllabus**, đừng nhúng nguyên dữ liệu vào app công khai khi chưa đọc điều khoản.
- **CEFR-J Grammar/Vocabulary Profile (Tono Laboratory, TUFS):** các bộ dữ liệu này dùng được "for research and commercial purposes with no charge, provided that you cite the dataset properly".\[24\] Đây là **lựa chọn an toàn nhất** để làm xương sống cho lộ trình A1–B2 của bản công khai.
- **Wiktionary** (CC BY-SA), **WordNet** (giấy phép mở kiểu BSD), và danh sách tần suất từ (ví dụ NGSL, CC BY-SA) làm dữ liệu từ vựng.

---

## 4. Phần C: Phương pháp học có cơ sở khoa học

### 4.1 Tóm tắt bằng chứng

| Phương pháp | Bằng chứng chính | Áp dụng vào app |
|---|---|---|
| **Retrieval practice / testing effect** | Roediger & Karpicke (2006, *Psychological Science*): sau 1 tuần, nhóm làm bài kiểm tra nhớ **56%** so với **42%** ở nhóm đọc lại. Ở thí nghiệm 2, nhóm học 1 lần + 3 lần kiểm tra nhớ 61% so với 40% ở nhóm học 4 lần, dù nhóm học 4 lần *tự tin hơn*\[25\] | Mỗi bài bắt đầu và kết thúc bằng câu hỏi nhớ lại, không chỉ đọc |
| **Spacing / lặp lại ngắt quãng** | Cepeda et al. (2006, *Psychological Bulletin*), 317 thí nghiệm: nhớ trung bình **47,3%** khi học giãn cách so với **36,7%** khi học dồn; khoảng cách tối ưu tăng theo thời gian cần nhớ\[26\] | FSRS cho cả thẻ từ vựng lẫn thẻ ngữ pháp |
| **Đường cong quên (Ebbinghaus, 1885)** | Trí nhớ suy giảm theo hàm mũ nếu không ôn | Cơ sở của FSRS và HLR |
| **Interleaving** | Dunlosky et al. (2013): "moderate utility"; bằng chứng trong lớp học còn ít\[11\] | Phiên ôn tập trộn unit cũ (ví dụ present simple với present continuous) |
| **Elaborative interrogation / self-explanation** | Dunlosky: "moderate"; Pressley et al. (1987) đạt khoảng 72% so với 37% nhớ có gợi ý\[11\] | Câu hỏi "Vì sao dùng *have been* ở đây?" và nút "giải thích bằng lời của bạn" |
| **Highlight / đọc lại** | Dunlosky: "low utility", "do not consistently boost students' performance"\[11\] | Highlight chỉ để *thu thập*; bắt buộc chuyển thành thẻ ôn chủ động |
| **Desirable difficulties (Bjork)** | Khó vừa phải khi học giúp nhớ lâu hơn\[25\] | Gõ đáp án thay vì chọn; ẩn gợi ý |
| **Số lần gặp từ** | Webb (2007): khoảng **10 lần gặp** mới đạt hiểu biết tương đối sâu; các nghiên cứu khác dao động từ 6 (Rott, 1999) đến hơn 20 (Waring & Takaki, 2003). Nation nhấn mạnh không có ngưỡng tối thiểu chắc chắn\[27\]\[28\] | Mỗi từ xuất hiện trong nhiều ngữ cảnh: câu gốc, câu ví dụ, cloze, nghe |
| **Tần suất và độ phủ từ (Nation)** | 1.000 họ từ đầu phủ **77,86%** văn bản, 1.000 tiếp theo thêm 8,23%, 1.000 tiếp nữa thêm 3,70%. Đọc thoải mái (98% độ phủ) cần **8.000–9.000 họ từ**; nghe cần 6.000–7.000 (Nation, 2006)\[29\] | Ưu tiên từ tần suất cao; gắn nhãn tần suất/CEFR cho mỗi từ bôi đen |
| **Học chủ ý và học ngẫu nhiên (Nation)** | Nên kết hợp: học chủ ý (flashcard) tăng tốc, đọc và nghe nhiều củng cố | App có cả hai: bài đọc (input) và kho từ SRS |
| **Comprehensible input i+1 (Krashen)** | Có ảnh hưởng lớn nhưng khó kiểm chứng trực tiếp; cần hiểu gần hết văn bản mới học được từ mới | Câu ví dụ chỉ chứa 1 điểm mới |
| **Dual coding / audio** | Hình ảnh và âm thanh bổ sung cho chữ | TTS offline cho mọi câu; ảnh tuỳ chọn |
| **Output practice** | Viết và nói buộc xử lý sâu hơn | Bài "viết 1 câu về bản thân dùng cấu trúc vừa học" |

### 4.2 Thuật toán lặp lại ngắt quãng: SM-2, HLR, FSRS

- **SM-2 (Woźniak, 1987):** quy tắc cố định với "ease factor" khởi đầu 2,5.\[30\] Đơn giản nhưng không cá nhân hoá được.
- **Half-Life Regression (Duolingo, Settles & Meeder, ACL 2016):** ước lượng "chu kỳ bán rã" của từ trong trí nhớ. Mô hình giảm sai số dự đoán **hơn 45%** so với các baseline và tăng mức tương tác hằng ngày **12%** trong thử nghiệm thực tế. Mã nguồn và 13 triệu bản ghi học tập đã được công bố.\[31\]\[32\]\[33\]
- **FSRS:** mô hình Difficulty–Stability–Retrievability. Theo README srs-benchmark của open-spaced-repetition, tham số mặc định dựa trên bộ dữ liệu khoảng 727 triệu lượt ôn từ 10 nghìn người dùng Anki; bộ benchmark mới hơn (FSRS-Anki-20k) gồm khoảng 1,7 tỷ lượt ôn từ 20 nghìn người, và đã có FSRS-7. Benchmark cho thấy FSRS-6 tốt hơn SM-2 ở 99,6% người dùng. Có hai lưu ý: SM-2 vốn không được thiết kế để dự đoán xác suất, và con số "giảm 20–30% lượt ôn" mà nhiều blog nhắc lại **đến từ mô phỏng, không phải thử nghiệm có đối chứng**.
- **Khuyến nghị:** dùng **FSRS** với desired retention mặc định 0,9, và cho người mất gốc hạ xuống 0,85 nếu thấy quá tải.\[12\]

### 4.3 Hình thành thói quen

- **Lally et al. (2010, *European Journal of Social Psychology*):** 96 tình nguyện viên lặp một hành vi hằng ngày trong cùng bối cảnh (ví dụ "sau bữa sáng"). Trung vị **66 ngày** để đạt mức tự động hoá, dao động **18–254 ngày**, và bỏ một ngày không ảnh hưởng đáng kể.\[13\]\[34\] Do đó cần kỳ vọng 2–3 tháng và **có cơ chế tha thứ**.
- **Implementation intentions (Gollwitzer & Sheeran, 2006):** kế hoạch "Nếu X thì tôi làm Y" có hiệu ứng **d = 0,65** trên 94 kiểm định.\[35\] Ứng dụng: khi onboarding, bắt người dùng chọn "Sau khi [pha cà phê buổi sáng], tôi sẽ học 1 bài".
- **BJ Fogg, Tiny Habits:** hành vi thật nhỏ, gắn vào thói quen sẵn có, rồi ăn mừng ngay. Ứng dụng: mục tiêu tối thiểu là "1 bài hoặc 5 phút", và streak được tính khi đạt mức tối thiểu.
- **Bằng chứng từ Duolingo:**
  - Tính năng Streak Wager làm tăng giữ chân người dùng ngày 1, 7 và 14 có ý nghĩa thống kê; riêng **ngày 7 tăng 14%**.\[36\]
  - Người học "cày dồn" (binge) dễ bỏ app hơn người học đều đặn.\[37\]
  - Streak Freeze cho phép nghỉ một ngày mà vẫn giữ chuỗi.\[38\]
  - Thuật toán tối ưu nội dung thông báo (Yancey & Settles, KDD 2020) tăng **0,5% DAU** và **2% giữ chân người dùng mới**. Mức tăng nhỏ nhưng có thật; nội dung thông báo cần *xoay vòng* vì thông báo mới có "novelty effect".\[39\]\[40\]
- **Cảnh báo:** streak dựa trên nỗi sợ mất (loss aversion).\[41\] Với người học một mình, nên kết hợp streak freeze và thống kê tiến bộ thật (số từ đã nhớ, unit đã xong) để tránh học "cho giữ streak".

### 4.4 Vòng học khuyến nghị cho mỗi bài (khoảng 10–15 phút)

1. **Khởi động nhớ lại (1–2 phút):** 3 thẻ ôn đến hạn, bao gồm ngữ pháp unit trước.
2. **Câu hỏi dự đoán (pre-test, 1 phút):** 2 câu thuộc unit mới, trước khi học. Làm sai cũng có ích vì bạn thấy ngay điều mình chưa biết.
3. **Đọc giải thích (3–4 phút):** trang sách (bản cá nhân) hoặc giải thích dạng thẻ; có TTS cho câu ví dụ; **bôi đen từ lạ** trong lúc đọc.
4. **Tự giải thích (30 giây):** gõ hoặc nói 1 câu "quy tắc này nghĩa là…".
5. **Bài tập có phản hồi ngay (4–5 phút):** điền từ, chọn, sửa lỗi, sắp xếp. Ưu tiên *gõ đáp án* thay vì chọn.
6. **Output (1–2 phút):** viết 1–2 câu về bản thân dùng cấu trúc mới; tuỳ chọn cho AI nhận xét khi có mạng.
7. **Kết thúc:** những câu làm sai và điểm ngữ pháp mới tự động thành thẻ FSRS; cộng streak/XP.

### 4.5 Phương pháp khuyến nghị cho mục từ vựng: "Sentence mining + cloze + FSRS"

- **Thu thập:** chạm hoặc bôi đen từ, app lưu **từ + câu gốc chứa từ + unit/trang + nghĩa tiếng Việt (từ điển offline) + phát âm TTS + nhãn CEFR/tần suất**.
- **Mỗi từ tạo 2–3 loại thẻ (tăng số lần gặp theo Webb):**
  1. *Nhận diện:* xem từ trong câu gốc, nhớ nghĩa.
  2. *Cloze:* câu gốc bị che từ, gõ lại từ (active recall, khó vừa phải).
  3. *Nghe:* TTS đọc câu, chọn hoặc gõ từ (dual coding).
- **Giới hạn đầu vào:** tối đa 10 từ mới mỗi ngày, ưu tiên từ thuộc nhóm 2.000–3.000 từ tần suất cao. Người mất gốc thường bôi đen quá nhiều rồi bị ngợp.
- **Chấm 4 mức (Again/Hard/Good/Easy)** theo FSRS.\[42\] Từ bị "Again" nhiều lần thì hiện câu ví dụ mới để có thêm ngữ cảnh.
- **Kết hợp input:** bài đọc ngắn dễ (graded) trong mỗi unit để từ đã học xuất hiện lại tự nhiên.

---

## 5. Phần D: Công nghệ

### 5.1 So sánh framework

| Tiêu chí | **Flutter** | React Native (Expo) | Kotlin Multiplatform | Native SwiftUI + Kotlin | PWA |
|---|---|---|---|---|---|
| Một codebase cho iPad + iPhone + Android | ✅ | ✅ | ⚠️ (logic chung; UI chung qua Compose MP — JetBrains công bố Compose for iOS Stable ở bản 1.8.0, 6/5/2025) | ❌ (2 codebase) | ✅ |
| Offline, SQLite | ✅ Drift/sqflite | ✅ op-sqlite/expo-sqlite/WatermelonDB | ✅ SQLDelight/Room | ✅ | ⚠️ IndexedDB; iOS có thể xoá dữ liệu |
| PDF + chọn chữ | ✅ pdfrx (PDFium), Syncfusion | ⚠️ react-native-pdf không chọn chữ tốt; cần Nutrient (trả phí) hoặc WebView pdf.js | ⚠️ phải viết riêng cho từng nền tảng | ✅ PDFKit (iOS) tốt nhất | ✅ pdf.js |
| FSRS | ✅ gói `fsrs` (Dart) | ✅ ts-fsrs | ✅ (port/binding fsrs-rs)\[12\]\[43\] | ✅ | ✅ ts-fsrs |
| iPad layout, Apple Pencil | ⚠️ tốt, Pencil cơ bản | ⚠️ | ⚠️/✅ | ✅ tốt nhất | ❌ |
| Hợp với dev dùng AI | ✅ | ✅✅ (TS phổ biến nhất) | ⚠️ | ⚠️ | ✅ |
| Mức độ phổ biến | Flutter 9,4% người trả lời Stack Overflow 2024 | RN 8,4% | KMP tăng từ 7% (2024) lên 18% (2025) theo khảo sát JetBrains Developer Ecosystem | — | — |

### 5.2 Stack khuyến nghị (một lựa chọn)

**Flutter + Drift (SQLite) + `fsrs` (Dart) + pdfrx + flutter_tts + flutter_local_notifications**

Lý do chọn:
- **PDF là yêu cầu khó nhất.** pdfrx chạy trên PDFium và hỗ trợ Android, iOS, Windows, macOS, Linux, Web, có text selection và tìm kiếm.\[44\] Syncfusion Flutter PDF Viewer có API `getSelectedTextLines` + `HighlightAnnotation`, đúng cho tính năng "bôi đen từ".\[45\] Syncfusion là thư viện thương mại; hãy kiểm tra điều kiện community license. Hệ sinh thái React Native yếu hơn ở mảng này nếu không trả phí.
- **UI tuỳ biến kiểu game** (bản đồ lộ trình, hiệu ứng streak) là thế mạnh của Flutter.\[46\]
- **Một codebase, logic thuần Dart** giúp dễ test thuật toán FSRS, streak và lịch.

Thành phần chi tiết:

| Lớp | Lựa chọn | Ghi chú |
|---|---|---|
| DB cục bộ | **Drift** (SQLite, type-safe, migration) | Nguồn dữ liệu chính (source of truth) là local |
| SRS | **`fsrs` (dart-fsrs, open-spaced-repetition)** | desired_retention 0,9; learning steps 1 phút và 10 phút; lưu thời gian UTC; có bản binding Rust `fsrs-rs-dart` nếu cần tối ưu tham số |\[12\]\[43\]
| PDF | **pdfrx** (MIT, miễn phí); dự phòng Syncfusion | Lưu highlight dạng *(pageIndex, textRange, rects)* trong DB của app, không ghi đè lên file PDF |
| Text có cấu trúc | `SelectableText`/`RichText` với mỗi từ là một `TextSpan` có `TapGestureRecognizer` | Chạm 1 lần = tra, giữ lâu = lưu |
| Từ điển offline | Wiktionary (qua Wiktextract, CC BY-SA), WordNet, dữ liệu Free Vietnamese Dictionary Project (GPL) hoặc từ điển CC BY-SA trên GitHub | **Chú ý giấy phép:** GPL/CC BY-SA có yêu cầu share-alike; tách dữ liệu từ điển thành gói tải riêng, ghi nguồn đầy đủ. VNEDICT (CC BY 3.0) là Việt→Anh, ít hữu ích cho chiều Anh→Việt\[47\] |
| TTS offline | `flutter_tts` gọi AVSpeechSynthesizer (iOS) và Android TTS | Chạy offline nếu giọng đã tải về máy |
| Nhắc nhở | `flutter_local_notifications` | Lịch cục bộ, không cần server; xoay vòng nội dung |
| AI (tuỳ chọn) | API đám mây (Claude, v.v.) chỉ dùng cho *sinh nội dung trước* và *nhận xét câu viết*; kết quả được cache vào DB | Mọi chức năng lõi vẫn chạy offline |
| Sync (giai đoạn công khai) | Supabase (Postgres + Auth) + một lớp sync (PowerSync hoặc tự viết đồng bộ theo log sự kiện) | Review log chỉ ghi thêm (append-only), rất hợp để merge không xung đột |

**Phương án thay thế:**
- **React Native (Expo) + op-sqlite + ts-fsrs:** nếu bạn mạnh TypeScript và chấp nhận dùng *nội dung đã chuyển sang JSON* thay vì hiển thị PDF gốc.
- **SwiftUI + PDFKit:** nếu bạn chấp nhận chỉ dùng iPad/iPhone. Trải nghiệm Pencil tốt nhất, nhưng mất Android.
- **PWA:** chỉ nên làm bản demo, vì lưu trữ offline trên iOS không đáng tin cậy lâu dài.

### 5.3 Pipeline chuyển sách PDF thành bài học (chỉ cho dùng cá nhân)

1. **Trích xuất văn bản:** nếu PDF có lớp chữ thì lấy trực tiếp (pdfrx/PyMuPDF). Nếu là bản scan thì OCR (Tesseract, Apple Vision, hoặc một mô hình OCR thị giác).
2. **Tách theo cấu trúc sách:** Grammar in Use có bố cục cố định, *trang trái là giải thích, trang phải là bài tập*.\[5\]\[48\] Tách theo cặp trang thành từng unit.
3. **LLM trích xuất sang JSON schema** với các trường: `unit {id, title, level, sections[{heading, explanation, examples[]}], exercises[{type, instruction, items[{prompt, answer, alternatives[]}]}]}`. Dùng structured output/JSON mode và kiểm tra bằng schema.
4. **Đối chiếu đáp án:** lấy từ phần Key (Answers) ở cuối sách, rồi kiểm tra chéo bằng script.
5. **Soát tay:** khoảng 10–20 phút mỗi unit ở giai đoạn đầu. Sách A1–A2 có khoảng 114 unit, sách B1–B2 khoảng 145 unit.
6. **Đóng gói:** file `.bookpack` (SQLite + ảnh) để nhập vào app. **Không** phân phối file này cho người khác.

---

## 6. Phần E: Kế hoạch triển khai

### 6.1 Mô hình dữ liệu (rút gọn)

```
Book(id, title, level_cefr, source_type[pdf|structured], file_hash)
Unit(id, book_id, order, title, grammar_tags[], pdf_page_start, pdf_page_end)
Lesson(id, unit_id, order, type[explain|practice|review|reading])
Exercise(id, lesson_id, type[gap|mcq|reorder|transform|free_write], prompt, answer_json, explanation)
Attempt(id, exercise_id, answer, is_correct, duration_ms, created_at_utc)
Highlight(id, book_id, page_index|lesson_id, text, range_json, rects_json, created_at)
VocabItem(id, lemma, surface, pos, meaning_vi, cefr, freq_rank, source_sentence, highlight_id, audio_ref)
Card(id, vocab_item_id|grammar_point_id, card_type[recog|cloze|listen|grammar], fsrs_state_json, due_utc)
ReviewLog(id, card_id, rating, reviewed_at_utc, elapsed_days, scheduled_days)   -- append-only
DailyActivity(date_local, minutes, lessons_done, reviews_done, goal_met, freeze_used)
Streak(current, longest, freezes_available, last_goal_date)
Settings(daily_goal_minutes, reminder_times[], desired_retention, if_then_plan_text)
```

### 6.2 Các màn hình chính

1. **Lộ trình (Path Map):** các unit dạng node theo sách/cấp độ; node khoá, đang học, đã xong; có huy hiệu "đã ôn".
2. **Lesson Reader:** chế độ trang PDF hoặc chế độ thẻ có cấu trúc; thanh công cụ bôi đen từ; TTS; trên iPad chia đôi màn hình giải thích/bài tập, mô phỏng bố cục trái–phải của sách.
3. **Exercise Player:** một câu mỗi lần, phản hồi ngay, nút "vì sao?".
4. **Vocab Bank:** danh sách, bộ lọc CEFR/unit, xem câu gốc, sửa nghĩa.
5. **Review Session:** thẻ đến hạn trộn từ vựng và ngữ pháp; 4 nút FSRS.
6. **Stats/Streak:** lịch nhiệt (heatmap), streak, số freeze, số từ "mature", tiến độ unit.
7. **Onboarding:** chọn mục tiêu phút mỗi ngày, giờ nhắc, câu "Nếu… thì…".

### 6.3 Lộ trình phát triển và ước lượng công sức (một người, bán thời gian khoảng 10–15 giờ/tuần, có AI hỗ trợ)

| Giai đoạn | Phạm vi | Thời gian ước tính |
|---|---|---|
| **0. Spike (thử nghiệm kỹ thuật)** | pdfrx: mở PDF sách, chọn chữ, lưu highlight; Drift; `fsrs` chạy được | 1–2 tuần |
| **1. MVP cá nhân** | Reader PDF + bôi đen → Vocab + Review FSRS + streak/thông báo cục bộ + lộ trình tay cho 10 unit đầu sách Essential | 4–6 tuần |
| **2. Nội dung có cấu trúc** | Pipeline LLM → JSON cho sách đỏ; Exercise Player; ôn ngữ pháp | 4–6 tuần (phần lớn là soát nội dung) |
| **3. Beta kín** | TestFlight (iOS) + Google Play Internal Testing; mô hình BYOB; xuất/nhập sao lưu | 3–4 tuần |
| **4. Công khai** | Nội dung gốc A1–A2 theo CEFR-J; Supabase + sync; tài khoản; chính sách quyền riêng tư; tư vấn luật sư | 3–6 tháng (nội dung là phần tốn nhất) |

**Phân phối:** TestFlight cho phép mời tester nội bộ và bên ngoài (bên ngoài cần Apple duyệt beta). Theo Apple Developer, Apple Developer Program có phí 99 USD mỗi năm thành viên (giá có thể khác theo khu vực). Google Play có kênh Internal testing (nhóm nhỏ, không cần duyệt lâu) rồi đến Closed/Open testing. Theo Google Play Console Help, tài khoản cá nhân tạo sau ngày 13/11/2023 phải chạy closed test với tối thiểu 12 tester tham gia liên tục ít nhất 14 ngày trước khi lên production; hãy kiểm tra quy định hiện hành. Riêng Android dùng cá nhân thì có thể cài APK trực tiếp.

### 6.4 Rủi ro và cách giảm thiểu

| Rủi ro | Mức độ | Giảm thiểu |
|---|---|---|
| **Bản quyền khi công khai** | Cao | Chỉ BYOB và nội dung gốc; không dùng tên/thương hiệu "Grammar in Use" để quảng bá; hỏi luật sư |
| **Công sức trích xuất nội dung** | Cao | Bắt đầu bằng chế độ PDF gốc (không cần trích xuất); chỉ cấu trúc hoá dần 1–2 unit mỗi ngày như một phần "học" |
| **Phình phạm vi (scope creep)** | Cao | Khoá phạm vi MVP gồm 5 tính năng; AI, sync, leaderboard để sau |
| **Động lực: "xây app thay vì học"** | Rất cao | Quy tắc: *học 1 bài trong app trước khi code mỗi ngày*; dùng chính app từ tuần 2 |
| **Chọn chữ trên PDF kém chính xác** (bố cục 2 cột, ô bài tập) | Trung bình | Dự phòng bằng Syncfusion hoặc chế độ văn bản có cấu trúc |
| **Dữ liệu offline bị mất** | Trung bình | Xuất/nhập file sao lưu từ MVP; review log append-only |
| **Streak gây áp lực** | Thấp–trung bình | Streak freeze, mục tiêu tối thiểu nhỏ, hiển thị tiến bộ thật |

---

## 7. Recommendations (Khuyến nghị hành động)

1. **Tuần này:** Làm spike pdfrx với PDF sách đỏ bạn sở hữu, kiểm tra chọn chữ trên trang bài tập. Nếu không ổn, thử Syncfusion. Đây là rủi ro kỹ thuật lớn nhất nên cần xác minh trước.
2. **Chọn Flutter + Drift + `fsrs`.** Cho SRS và DB chạy độc lập với UI để dễ test.
3. **MVP chỉ gồm 5 thứ:** trang sách, bôi đen từ, kho từ, ôn FSRS, streak/nhắc nhở. Chưa làm AI và sync.
4. **Tự dùng từ ngày đầu:** mục tiêu 1 unit sách Essential mỗi ngày, cộng không quá 10 từ mới, cộng ôn đến hạn. Theo Lally, hãy đo sau 66 ngày chứ đừng đánh giá sau 1 tuần.
5. **Trước khi công khai:** chuyển sang nội dung gốc dựa trên CEFR-J (được phép thương mại nếu trích dẫn), dùng EGP làm tham khảo; gửi yêu cầu đến PLSclear/rights@cambridge.org để có hồ sơ; thuê luật sư sở hữu trí tuệ duyệt mô hình BYOB và giấy phép dữ liệu từ điển.
6. **Định vị bản công khai:** "trình học có cấu trúc cho sách ngữ pháp của bạn + khoá A1–A2 miễn phí tự biên soạn cho người Việt mất gốc", chứ không phải "app Grammar in Use".

## 8. Caveats (Lưu ý)

- Thông tin về Cambridge One offline có mâu thuẫn giữa các trang của Cambridge. Tình trạng app Cambridge trên App Store (iOS) hiện tại tôi chưa xác minh được, chỉ xác minh được việc gỡ khỏi Google Play.
- Một số nguồn so sánh app (giá LingQ, Readlang, Migaku; các "báo cáo 2026" trên blog) do đối thủ hoặc trang tiếp thị viết, có thể thiên vị.\[49\] Giá và giới hạn gói miễn phí thay đổi thường xuyên.\[8\]
- Con số "FSRS giảm 20–30% lượt ôn" đến từ mô phỏng.\[50\] Benchmark 99,6% đo độ chính xác dự đoán, không đo kết quả học.
- Điều khoản sử dụng của English Grammar Profile chưa đọc được đầy đủ; đừng giả định là dữ liệu mở.
- Ước lượng thời gian phát triển là ước lượng chủ quan, phụ thuộc lớn vào công sức soát nội dung.
- Đây không phải tư vấn pháp lý.

## Sources

1. [Murphy&#39;s English Grammar in Use - App on Amazon Appstore](https://www.amazon.com/Murphys-English-Grammar-in-Use/dp/B00MX1NML8)
2. [English Grammar in Use 1.11.40 Free Download](https://english-grammar-in-use-47.soft112.com/)
3. [Download English Grammar in Use - Free APK for Android](https://www.appbrain.com/app/english-grammar-in-use/org.cambridge.englishgrammar.egiu)
4. [English Grammar in Use 5th Edition - Digital | Grammar, Vocabulary and Pronunciation | Cambridge Spain](https://www.cambridge.es/en/catalogue/grammar-vocabulary-and-pronunciation/grammar/english-grammar-in-use5/digital)
5. [English Grammar in Use 4th Edition | Cambridge University Press & Assessment](https://www.cambridge.org/us/cambridgeenglish/catalog/grammar-vocabulary-and-pronunciation/english-grammar-use-4th-edition)
6. [Can I access my Cambridge One eBooks offline? – Cambridge One](https://cambridgeonehelp.cambridge.org/hc/en-gb/articles/4410618273298-Can-I-access-my-Cambridge-One-eBooks-offline)
7. [shop.cambridge.org](https://shop.cambridge.org/english/product/2700197219)
8. [Best Anki Alternatives for Language Learners: An Honest Comparison — Word Holder](https://wordholder.com/blog/anki-alternative)
9. [The Complete Guide to Importing on LingQ – LingQ Blog](https://www.lingq.com/blog/complete-guide-importing-lingq/)
10. [LingQ iOS App Support - How to learn languages using LingQ for mobile](https://www.lingq.com/en/ios-app-support/)
11. <https://gwern.net/doc/psychology/spaced-repetition/2013-dunlosky.pdf>
12. [fsrs | Dart package](https://pub.dev/packages/fsrs)
13. [The 21-Day Habit Myth: Lally's Real Answer Is 66 Days](https://taskcoach.ai/blog/habit-formation-real-curve-lally-66-days/)
14. [Advanced Grammar in Use Activities by Cambridge University Press & Assessment (App)](https://advanced-grammar-use-activities.appstor.io/)
15. [Software Alternatives, Accelerators & Startups](https://www.saashub.com/compare-unacademy-languages-app-vs-readlang)
16. [eJOY Learn English with videos - App Store](https://apps.apple.com/vn/app/ejoy-learn-english-with-videos/id1258329729)
17. [eJOY Learn English Videos - App Store - Apple](https://apps.apple.com/us/app/ejoy-learn-english-videos/id1622797145)
18. [Seeking permission to use copyrighted material](https://www.cambridge.org/core/services/aop-file-manager/file/577e80b452035e1b364ad118/Cambridge-Core-author-guide-Seeking-permission-to-use-copyrighted-material.pdf)
19. [Permissions requests | Cambridge University Press & Assessment](https://www.cambridge.org/rights-and-permissions/permissions)
20. [Rights requests | Cambridge University Press & Assessment](https://www.cambridge.org/about-us/rights-permissions/rights)
21. [Rights & Permissions - Cambridge University Press](https://www.cambridge.org/rights/rights_request.htm)
22. [The English Grammar Profile of learner competence: Methodology and key findings](https://benjamins.com/catalog/ijcl.14086.oke)
23. [English Profile -](https://englishprofile.org/)
24. [GitHub - openlanguageprofiles/olp-en-cefrj: Open Language Profiles — English profile datasets from CEFR-J](https://github.com/openlanguageprofiles/olp-en-cefrj)
25. <https://colinallen.dnsalias.org/Readings/2006_Roediger_Karpicke_PsychSci.pdf>
26. <https://augmentingcognition.com/assets/Cepeda2006.pdf>
27. [Word Frequency Levels and L2 Learners' Vocabulary ...](https://files.eric.ed.gov/fulltext/EJ1179006.pdf)
28. [How much input do you need to learn the most frequent ...](https://files.eric.ed.gov/fulltext/EJ1044345.pdf)
29. <https://www.scienceguide.nl/wp-content/uploads/2017/11/nation-2006-vocabulary.pdf>
30. [FSRS vs SM-2: why the modern spaced-repetition algorithm beats the classic](https://www.antiagent.io/blog/fsrs-vs-sm-2)
31. [Replication Data for: A Trainable Spaced Repetition Model for Language Learning - Duolingo Dataverse](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME)
32. [\[PDF\] A Trainable Spaced Repetition Model for Language Learning | Semantic Scholar](https://www.semanticscholar.org/paper/A-Trainable-Spaced-Repetition-Model-for-Language-Settles-Meeder/cb836d2b8e126dc31ded5e674d73021604dcc6e0)
33. [A Trainable Spaced Repetition Model for Language Learning](https://research.duolingo.com/papers/settles.acl16.pdf)
34. [How are habits formed: Modelling habit formation in the real world - Lally - 2010 - European Journal of Social Psychology - Wiley Online Library](https://onlinelibrary.wiley.com/doi/abs/10.1002/ejsp.674)
35. [(PDF) Implementation intentions and goal achievement: A meta-analysis of effects and processes (2006) | Peter M. Gollwitzer | 3248 Citations](https://scispace.com/papers/implementation-intentions-and-goal-achievement-a-meta-4awoi9wm93)
36. [Duolingo's Habit-Forming Reminders: A UX Breakdown](https://www.digia.tech/post/duolingo-habit-forming-reminders-retention-architecture/)
37. [How Streaks keep Duolingo learners committed to their language goals](https://blog.duolingo.com/how-streaks-keep-duolingo-learners-committed-to-their-language-goals/)
38. [how duolingo streak builds habit](https://blog.duolingo.com/how-duolingo-streak-builds-habit)
39. [How I Re-Created Duolingo’s Famous Notification Algorithm | by Jacob Mazurkiewicz | Medium](https://medium.com/@jakemazurkiewicz6/how-i-re-created-duolingos-famous-notification-algorithm-00fce580b84e)
40. [KDD 2020 | A Sleeping, Recovering Bandit Algorithm for Optimizing Recurring Notifications](https://www.kdd.org/kdd2020/accepted-papers/view/a-sleeping-recovering-bandit-algorithm-for-optimizing-recurring-notificatio.html)
41. [The Secret Behind Duolingo Streaks: How They Keep You Hooked (And How You Can Do It Too)](https://darewell.co/en/duolingo-streaks-retention-secret/)
42. [GitHub - open-spaced-repetition/dart-fsrs: Dart Package for FSRS · GitHub](https://github.com/open-spaced-repetition/dart-fsrs)
43. [GitHub - open-spaced-repetition/fsrs-rs-dart: dart binding for fsrs-rs · GitHub](https://github.com/open-spaced-repetition/fsrs-rs-dart)
44. [pdfrx\_fork | Flutter package](https://pub.dev/packages/pdfrx_fork)
45. [Text selection in Flutter PDF Viewer widget | Syncfusion](https://help.syncfusion.com/document-processing/pdf/pdf-viewer/flutter/text-selection)
46. [Flutter vs React Native vs Kotlin Multiplatform (2026) — When to Choose Each](https://syedali.dev/flutter-vs-react-native-vs-kotlin-multiplatform-2026)
47. [VNEDICT (Vietnamese-English Dictionary)](http://www.denisowski.org/Vietnamese/vnedict_readme.htm)
48. [English Grammar in Use APK for Android - Download](https://english-grammar-in-use.en.softonic.com/android)
49. [Best Language Learning App 2026: Independent Research Shows Clear Winner - A Nation of Moms](https://anationofmoms.com/2026/05/best-language-learning-apps.html)
50. [fsrs-vs-sm2-spaced-repetition-algorithm | Mindomax Blog](https://www.mindomax.com/fsrs-vs-sm2-spaced-repetition-algorithm)
