# Day 17 Submission
**Student:** Hứa Quang Linh
**Date:** 24/4/2026
**Product idea:** Sinh viên trước mỗi kỳ thi cần tổng hợp lại kiến thức và ôn tập, đặc biệt là vá những kiến thức bị hổng do nghỉ học hoặc bỏ lỡ nội dung trên lớp. ViBook.ai tối ưu hóa thời gian ôn tập bằng cách biến toàn bộ tài liệu học kỳ thành kho kiến thức đối thoại được (AI Tutor) — sinh viên hỏi thay vì đọc, nhận câu trả lời chất lượng ngang giảng viên giảng lại, có verification nhiều lớp để đảm bảo độ chính xác.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> ViBook.ai có thể đạt độ chính xác >98% và độ đầy đủ >95% trên tài liệu học thuật 2 môn pilot, đủ để sinh viên chủ động dùng AI thay vì đọc tài liệu gốc, kể cả khi rủi ro trượt môn là có thật.

**In-Scope** (tối đa 3):
- [ ] **AI Tutor Conversational Interface với Multi-layer Verification** — test giả định: SV có dám tin AI đủ để thay thế việc đọc slide gốc khi accuracy đạt >98% trên eval dataset không?
- [ ] **Confidence Scoring + Citation Layer (deep-link slide/timestamp)** — test giả định: Confidence Score minh bạch có giúp SV hiệu chỉnh mức độ tin cậy đúng lúc (tin khi nên tin, verify khi nên verify) không?
- [ ] **Quiz Generator + Knowledge Gap Roadmap** — test giả định: Sau khi dùng ViBook ôn tập thay cho việc đọc slide, điểm thi của SV có **không giảm** (ideally tăng) so với nhóm control không?

**Out-of-Scope:**
- **Flashcards & Thuật toán lặp lại ngắt quãng** — lý do bỏ: Không phục vụ giả định cốt lõi (thay thế đọc); là tính năng "nice-to-have" kéo dài timeline.
- **Mind-map & Slide generator** — lý do bỏ: Chưa chứng minh được nhu cầu ở mức pilot; không liên quan đến test "SV có tin AI đủ để thay thế đọc không".
- **Mở rộng ra nhiều môn học, nhiều trường** — lý do bỏ: Hướng B yêu cầu accuracy >98%, chỉ đạt được khi scope domain đủ hẹp. MVP chỉ target 2 môn tại 1 trường để build eval dataset chất lượng.

**Non-Goals:**
- KHÔNG giải bài tập thay sinh viên (giữ khả năng tư duy).
- KHÔNG trả lời kiến thức ngoài tài liệu giảng viên cung cấp (tránh hallucination và scope creep).
- KHÔNG thay thế việc tương tác trực tiếp với giảng viên.
- KHÔNG xây Dashboard cho giảng viên trong MVP (chỉ phục vụ SV).

---

## 2. PRD Skeleton

### Problem Statement
> Sinh viên đại học Việt Nam vào mỗi kỳ thi cuối kỳ phải xử lý 500+ trang slide, 20-30 giờ recording và nhiều tài liệu phụ; phương pháp đọc tuần tự không còn khả thi về thời gian, dẫn đến ôn tập dàn trải, bỏ sót kiến thức trọng tâm, và gia tăng tỷ lệ thi trượt/điểm thấp — một hậu quả học thuật và tài chính nghiêm trọng (học lại tốn phí, kéo dài thời gian tốt nghiệp).

### Target User
> Sinh viên năm 1-3 đại học kinh tế/kỹ thuật tại Việt Nam, 18-22 tuổi, có thói quen dùng smartphone + laptop song song, đã quen dùng ChatGPT cho việc học nhưng **không tin tưởng** để dùng cho việc ôn thi vì sợ sai lệch so với tài liệu giảng viên. Pilot scope: SV tại 1 trường đại học cụ thể, đang học 2 môn pilot (1 môn lý thuyết + 1 môn kỹ thuật) có tài liệu chuẩn hóa từ giảng viên.

### User Stories

**Story 1:**
> As a sinh viên vào mùa thi cuối kỳ, I want hỏi AI Tutor về bất kỳ khái niệm nào trong tài liệu và nhận câu trả lời đầy đủ + có cấu trúc như giảng viên giảng lại, so that tôi không phải đọc lại toàn bộ 500 trang slide mà vẫn hiểu đúng và đủ kiến thức trọng tâm.

**Story 2:**
> As a sinh viên đã vắng một số buổi học, I want dùng AI học lại nội dung đã bỏ lỡ mà không cần xem lại toàn bộ video ghi âm 2 tiếng, so that tôi bắt kịp kiến thức nhanh chóng và verify qua deep-link khi cảm thấy cần kiểm tra độ tin cậy.

**Story 3:**
> As a sinh viên chuẩn bị thi, I want hệ thống phát hiện lỗ hổng kiến thức qua Quiz và tạo roadmap ôn tập cá nhân hóa, so that tôi ôn tập có trọng tâm thay vì học ngẫu nhiên theo cảm tính.

### AI-Specific

**Model Selection:**
- Model:
  - **Ingestion (OCR + Vision + Chunking):** Claude Opus 4.7 hoặc GPT-4o + Vision
  - **Audio Transcription:** Whisper-large-v3 có fine-tune tiếng Việt
  - **Primary RAG (trả lời chính):** Claude Opus 4.7
  - **Verification Layer:** Claude Sonnet 4.6 (model độc lập check câu trả lời của Primary)
  - **Embedding:** text-embedding-3-large + BGE-M3 (Vietnamese) + cross-encoder re-ranking
  - **Quiz Generation:** GPT-4o-mini
- Lý do chọn: Hướng B yêu cầu độ chính xác >98% và hallucination <1%, nên mọi khâu "rủi ro cao" (ingestion + RAG + verification) đều phải dùng model frontier. Ingestion là nền móng — nếu OCR sai công thức hay chunking sai ngữ nghĩa thì toàn bộ câu trả lời downstream sụp đổ. Verification Layer dùng model khác hoặc model độc lập để tránh "confirmation bias" của một model.
- Trade-offs **chấp nhận**:
  - Chi phí API cao hơn 3-5x so với dùng model mini
  - Latency cao hơn (2-5s/câu trả lời) do có verification step
  - Timeline MVP kéo dài 9-12 tháng thay vì 2-3 tháng
- Trade-offs **không chấp nhận**:
  - KHÔNG hạ cấp model khi quá tải (thà báo "hệ thống bận" còn hơn trả lời bằng model yếu)
  - KHÔNG skip verification để tăng tốc
  - KHÔNG dùng kiến thức ngoài tài liệu gốc để "bổ sung" câu trả lời

**Data Requirements:**
- Nguồn:
  - Tài liệu giảng viên upload (slide PDF/PPTX, transcript, recording audio)
  - Eval dataset: 500+ câu hỏi thi thật từ 2 môn pilot, có đáp án chuẩn do giảng viên xác nhận
- Owner:
  - Tài liệu học: Giảng viên pilot (10-20 người) — cần MoU hợp tác chính thức với trường
  - Eval dataset: Team Data/ML Engineer xây dựng cùng giảng viên
  - Feedback loop: Student success manager thu thập từ pilot users
- Update frequency:
  - Tài liệu học: theo học kỳ (mỗi 4-5 tháng)
  - Eval dataset: mở rộng 10-20% mỗi tháng trong pilot
  - Quality monitoring: 5% câu trả lời/ngày được giảng viên review (human-in-the-loop sampling)

**Fallback UX:**
- Chiến lược: **Expectation Management + Graceful Handover**
- Trigger:
  - Confidence Score < 70% → AI không đủ tự tin
  - Câu hỏi ngoài phạm vi tài liệu gốc (retrieval không match chunk nào với similarity > threshold)
  - Verification Layer phát hiện claim không có trong nguồn (grounding fail)
  - Hệ thống quá tải / API provider down
- Hành động:
  - **Confidence 70-89%:** Show câu trả lời kèm warning + gợi ý đọc nguồn để verify
  - **Confidence < 70%:** Không show câu trả lời tự do, chỉ gợi ý trang slide/timestamp liên quan để SV tự đọc (tạm fallback về "Hướng A mode")
  - **Ngoài tài liệu:** Từ chối rõ ràng, gợi ý hỏi giảng viên hoặc upload thêm tài liệu
  - **Quá tải:** Thông báo nhẹ nhàng + gợi ý chờ, KHÔNG hạ cấp model
- User options:
  - Xem trực tiếp nguồn gốc (slide page/video timestamp)
  - Rephrase câu hỏi
  - Báo lỗi (👎 + lý do) để team cải thiện
  - Escalate: gợi ý hỏi trực tiếp giảng viên

### Success Metrics
- Primary metric: **Exam Performance Delta** — chênh lệch điểm thi giữa nhóm SV dùng ViBook vs nhóm control không dùng, đo qua A/B test
- Ngưỡng thành công:
  - Exam Performance Delta **≥ +0 GPA** (điểm không giảm, ideally tăng)
  - Accuracy trên eval dataset **>98%**
  - Primary Usage Rate **>70%** (% SV mở ViBook trước khi mở slide gốc)
  - Sean Ellis Test **>40%** "Very Disappointed", đo **giữa học kỳ** (không phải mùa thi)
- Timeframe đo lường:
  - Accuracy: liên tục từ giai đoạn internal testing (tháng 7)
  - Exam Performance Delta: cuối kỳ thi đầu tiên sau pilot (~tháng 12)
  - Primary Usage Rate: tuần thứ 4 trở đi sau khi SV onboard
  - Sean Ellis: giữa học kỳ (tuần 7-8 của 15-tuần-kỳ)

### Dependencies & Constraints
- **Phụ thuộc:**
  - MoU với ít nhất 1 trường đại học pilot + 10-20 giảng viên sẵn sàng collaborate (cung cấp tài liệu + review eval)
  - API access tới Claude Opus 4.7, GPT-4o, Whisper (budget dự trù $80-150k cho 9-12 tháng)
  - Team tối thiểu: 1 Product Lead + 2 AI Engineers + 1 Full-stack + 1 Data/ML Engineer + 1 UX Designer (part-time)
- **Ràng buộc:**
  - MVP KHÔNG được release nếu không đạt Release Gate Metrics (Accuracy >98%, Completeness >95%, Hallucination <1%, Citation Precision >99%)
  - Legal: cần disclaimer rõ ràng về giới hạn AI + ToS giảm liability khi SV trượt môn
  - Văn hóa học tập VN: giảng viên có thể phản đối nếu SV không đọc slide → cần co-design với giảng viên làm champion thay vì đối thủ
  - Timeline: 9-12 tháng MVP (không thể rút ngắn nếu muốn giữ chất lượng Hướng B)

---

## 3. Hypothesis Table

### Hypothesis 1 (cho tính năng In-Scope #1: AI Tutor với Multi-layer Verification)
> "Chúng tôi tin rằng cung cấp AI Tutor với độ chính xác >98% và verification pipeline 3 lớp (Grounding + Faithfulness + Completeness) sẽ giúp sinh viên vào mùa thi đạt được trải nghiệm 'hỏi thay vì đọc' mà không phải lo sai lệch so với tài liệu gốc.
> Chúng tôi sẽ biết mình đúng khi thấy Accuracy Rate đạt **>98%** trên eval dataset 500 câu và Primary Usage Rate đạt **>70%** trong **2 tuần đầu của pilot** với 100-200 SV."

Riskiest assumption: Verification Layer (Sonnet 4.6 check lại Opus 4.7) thực sự giảm hallucination xuống dưới 1% — chứ không chỉ cho cảm giác an toàn giả tạo. Nếu verification model bỏ sót cùng loại lỗi với primary model, toàn bộ pipeline sụp đổ.

Cách test cheapest: **Red-team eval** — trước khi đầu tư full engineering, chạy 200 câu eval qua pipeline prototype (Opus + Sonnet verification) trên Jupyter notebook. Đo rate hallucination được verification catch vs bỏ sót. Nếu catch rate <80%, phải thay đổi kiến trúc verification (ví dụ: thêm ensemble, hoặc dùng retrieval-based verification thay vì LLM-based) trước khi build UI.

### Hypothesis 2 (cho tính năng In-Scope #2: Confidence Scoring + Citation)
> "Chúng tôi tin rằng hiển thị Confidence Score minh bạch (% độ tin cậy) kèm deep-link citation sẽ giúp sinh viên hiệu chỉnh mức độ tin cậy đúng lúc — tin khi AI tự tin cao, verify khi AI không chắc chắn — đạt được việc dùng AI 'an toàn' thay cho đọc tài liệu.
> Chúng tôi sẽ biết mình đúng khi thấy **Calibration Error <5%** (sai số giữa confidence hiển thị và accuracy thực tế) và **Source-click Rate giảm dần theo thời gian** (từ >50% tuần đầu xuống <30% tuần thứ 4) trong pilot."

Riskiest assumption: SV hiểu và tin vào Confidence Score — chứ không phớt lờ nó. Nếu SV nhìn thấy "94%" và hiểu là "gần đúng" thay vì "vẫn có 6% rủi ro", họ có thể tin mù quáng. Ngược lại, nếu họ không hiểu con số, họ có thể từ chối dùng.

Cách test cheapest: **Wizard-of-Oz usability test** với 10 SV — giả lập 20 câu trả lời có confidence khác nhau (60%, 75%, 90%, 98%). Quan sát hành vi: SV có click citation khi confidence thấp không? Có hỏi lại khi confidence thấp không? Phỏng vấn sau: họ diễn giải con số thế nào? Từ đó tinh chỉnh UX (ví dụ: thay % bằng "3 mức tự tin" dễ hiểu hơn).

### Hypothesis 3 (cho tính năng In-Scope #3: Quiz + Roadmap)
> "Chúng tôi tin rằng Quiz Generator + Knowledge Gap Roadmap sẽ giúp sinh viên chuẩn bị thi đạt được điểm thi không giảm so với khi ôn tập truyền thống.
> Chúng tôi sẽ biết mình đúng khi thấy **Exam Performance Delta ≥ +0 GPA** trong A/B test kết thúc kỳ thi đầu tiên sau pilot."

Riskiest assumption: SV dùng Quiz + Roadmap một cách nghiêm túc, không phải làm qua loa. Nếu họ chỉ làm Quiz để "cho xong", kết quả sẽ không phản ánh đúng knowledge gap.

Cách test cheapest: Bắt đầu bằng **manual roadmap** — 1 học kỳ đầu, team giảng viên tự tạo roadmap sau khi xem kết quả Quiz của 20 SV champion. Đo xem roadmap thủ công có cải thiện điểm không. Nếu có → mới đầu tư vào automation. Nếu không → Quiz/Roadmap không phải là feature đáng đầu tư trong MVP.

---

## 4. PMF Scorecard

**Aha Moment:**
> Sinh viên đặt một câu hỏi khó về nội dung **họ chưa từng đọc**, ViBook trả lời chính xác + đầy đủ + có cấu trúc như buổi giảng mini (Khái niệm → Giải thích → Ví dụ → Cảnh báo), và sinh viên **đóng app, không mở slide gốc để verify**, sau đó làm Quiz về chủ đề đó với confidence cao và đúng.

Đây là hành vi cụ thể chứ không phải cảm giác — có thể tracking được qua event log: (1) user sends question, (2) user reads answer, (3) user does NOT click citation, (4) user does NOT open source document, (5) user proceeds to next question or takes quiz on same topic within X minutes, (6) user gets quiz question right.

**Actionable Metric:**
> **Primary Usage Rate** = % session học tập mà SV mở ViBook **trước** khi mở slide gốc / PDF viewer / recording player.
>
> Cách đo: Browser extension tracking + mobile app analytics ghi nhận thứ tự hành động của SV trong 1 session ôn tập (session = khoảng thời gian liên tục <30 phút giữa các action học tập). Mỗi session được tag là "ViBook-first" hoặc "Source-first".

**PMF Method:**
> **Hybrid: Aha Moment Tracking (primary) + Sean Ellis Test (secondary) + Exam Performance Delta (validation)**
>
> - Aha Moment Tracking (liên tục): Primary Usage Rate > **70%** trong tuần 4+ của pilot
> - Sean Ellis Test: ≥ **40%** SV trả lời "Very Disappointed" nếu không được dùng ViBook, đo ở **tuần 7-8 (giữa học kỳ)** — không phải mùa thi, để tránh survivorship bias
> - Exam Performance Delta: **≥ +0 GPA** trong A/B test cuối kỳ thi pilot

Lý do dùng hybrid: Sean Ellis một mình dễ bị bias do SV đang stress mùa thi. Primary Usage Rate đo được hành vi thực. Exam Performance Delta là ground truth — nếu SV dùng app nhiều nhưng điểm thi giảm, sản phẩm thất bại về mặt đạo đức dù các metric khác đẹp.

**Vanity Metrics tôi sẽ không dùng:**
- **Số lượng sign-up** — đăng ký rồi bỏ không chứng minh giá trị.
- **Số câu hỏi AI đã trả lời / ngày** — dùng nhiều không bằng dùng hiệu quả.
- **Số lượt click deep-link** — ở Hướng B, click nhiều = SV không tin AI = sản phẩm thất bại (ngược với trực giác và ngược với Hướng A).
- **DAU/MAU trong mùa thi** — mùa thi SV dùng mọi công cụ vì tuyệt vọng; DAU tăng không chứng minh PMF.
- **Review 5 sao trên App Store** — dễ bị thao túng bởi pilot users là champion.

---

## 5. AI Critique Log

**Điểm AI chỉ ra:**

1. **Mâu thuẫn triết lý sản phẩm** giữa Non-Goals ("không thay thế đọc") và Riskiest Assumption ("SV tin AI đủ để thay thế đọc") — **Action: Accept** — Lý do: Đây là lỗi logic nghiêm trọng nhất. Đã quyết định dứt khoát chọn Hướng B (thay thế đọc) và rewrite toàn bộ Non-Goals, Riskiest Assumption, Hypothesis, Aha Moment để đồng bộ. Không thể xây sản phẩm nếu không biết mình đang xây gì.

2. **Model Selection đảo ngược logic rủi ro** — dùng gpt-4o-mini cho Ingestion (khâu rủi ro cao nhất) và gpt-4o cho RAG (khâu chỉ cần tổng hợp) — **Action: Accept** — Lý do: Nguyên tắc "garbage in, garbage out" trong RAG là tuyệt đối. Đã đổi: Opus 4.7 + Vision cho Ingestion, Opus 4.7 cho RAG, Sonnet 4.6 cho Verification Layer (mới thêm). Chấp nhận chi phí API cao hơn 3-5x vì Hướng B không cho phép compromise chất lượng.

3. **PMF Metrics đo sai thứ cần đo** — "5 lần click deep-link / phiên" làm Aha Metric ở Hướng B là ngược logic; thiếu metric đo kết quả học tập thực sự — **Action: Accept** — Lý do: Ở Hướng B, click deep-link nhiều = SV không tin AI = thất bại. Đã đổi sang Primary Usage Rate (>70%) + Exam Performance Delta (≥ +0 GPA) làm metric chính. Sean Ellis đo ở giữa học kỳ thay vì mùa thi để tránh survivorship bias.

4. **Thiếu hoàn toàn AI Quality Assurance** — không có eval dataset, không có release gate, không có kill switch — **Action: Accept** — Lý do: Đây là hole chí tử của Hướng B. Đã thêm Section "AI Quality Assurance" với eval dataset 500+ câu, Release Gate Metrics bắt buộc đạt trước khi launch, và Kill Switch tự động hạ cấp về "Hướng A mode" nếu metric tụt 2 ngày liên tiếp.

5. **Scope MVP quá rộng so với rủi ro kỹ thuật** — tham vọng làm cho mọi môn ngay từ MVP — **Action: Accept** — Lý do: Accuracy >98% chỉ đạt được khi domain đủ hẹp. Đã thu hẹp MVP xuống 2 môn tại 1 trường với 10-20 giảng viên collaborate + 100-200 SV pilot. Expansion roadmap để sau khi QA pipeline chứng minh scale được.

**Thay đổi lớn nhất giữa Version A và Version B:**
> Sự thay đổi lớn nhất không phải ở tính năng mà ở **triết lý và bar chất lượng**.
>
> Version A (PRD cũ) là sản phẩm "chatbot hỗ trợ tra cứu" — bar chất lượng trung bình, timeline 2-3 tháng, chi phí thấp, moat yếu, rủi ro thấp.
>
> Version B (PRD mới) là sản phẩm "AI Tutor thay thế việc đọc" — bar chất lượng phải đạt >98% accuracy, timeline 9-12 tháng, chi phí cao hơn 3-5x, moat mạnh (nếu làm được), rủi ro cực cao (1 case hallucination viral có thể kết liễu sản phẩm).
>
> Ba trụ cột mới được thêm vào Version B mà Version A không có:
> 1. **Verification Layer** — model độc lập check câu trả lời của primary model trước khi hiển thị
> 2. **AI Quality Assurance pipeline** — eval dataset + release gate + kill switch + continuous monitoring
> 3. **Pilot Strategy thu hẹp** — 2 môn tại 1 trường thay vì "mọi SV mọi trường", chấp nhận scale chậm để giữ chất lượng
>
> Về metric, sự đảo ngược quan trọng nhất: **Source-click Rate** từ "càng cao càng tốt" (Version A) thành **"càng thấp càng tốt"** (Version B) — phản ánh thay đổi triết lý từ "dẫn đường đến nguồn" sang "thay thế nguồn".

---

## 6. Self-assessment

**Mắt xích nào trong [MVP Boundary → PRD → Hypothesis → PMF] bạn đang yếu nhất?**
> **Mắt xích Hypothesis → PMF** đang yếu nhất.
>
> Cụ thể: Tôi tự tin về MVP Boundary (đã thu hẹp scope rõ ràng) và PRD Skeleton (đã có model selection, verification, QA pipeline cụ thể). Nhưng khi chuyển sang phần Hypothesis và PMF, tôi nhận ra có 3 điểm chưa chắc:
>
> 1. **Chưa có cách test cheapest thực sự "cheap"** cho Hypothesis 1. Tôi đề xuất "red-team eval 200 câu" nhưng thực tế cần eval dataset có đáp án chuẩn — mà đáp án chuẩn cần giảng viên review, mà giảng viên cần MoU với trường — mà MoU cần 2-3 tháng. Vòng lặp này khiến "cheapest test" vẫn đắt và chậm.
>
> 2. **Exam Performance Delta là metric đúng nhưng khó đo**. A/B test với SV thật + biến kiểm soát (cùng giảng viên, cùng đề thi, cùng trình độ đầu vào) là một thiết kế nghiên cứu phức tạp, không giống A/B test sản phẩm thông thường. Tôi chưa biết rõ cách thiết kế sao cho có ý nghĩa thống kê với n=100-200.
>
> 3. **Primary Usage Rate khó tracking trên mobile**. Để biết SV mở ViBook "trước" slide gốc, cần browser extension + mobile tracking — khả thi trên web nhưng khó trên iOS do sandbox. Có thể phải dùng self-report survey thay cho tracking tự động, nhưng self-report bias.
>
> Nói cách khác: Tôi đã có bar chất lượng đúng cho Hướng B, nhưng **chưa có instrument đủ tốt để đo bar đó**. Nếu không giải quyết, tôi sẽ build sản phẩm mà không biết nó đang PMF hay không.

**Open questions bạn muốn giải đáp tiếp:**

1. **Làm sao thiết kế A/B test Exam Performance Delta có ý nghĩa thống kê khi n chỉ 100-200 SV?** — Có cần dùng paired test (cùng SV, 2 môn khác nhau — 1 dùng ViBook, 1 không)? Làm sao kiểm soát biến "độ khó môn học"? Có cần thuê thống kê gia tư vấn ngay từ giai đoạn thiết kế pilot không?

2. **Verification Layer có bị "correlated error" với Primary Model không?** — Nếu cả Opus 4.7 và Sonnet 4.6 cùng lỗi ở 1 loại câu (ví dụ: câu có thuật ngữ tiếng Anh trong ngữ cảnh tiếng Việt), verification sẽ không catch được. Có cần dùng model khác nhà cung cấp (ví dụ: Gemini làm verifier cho Claude) để đảm bảo error diversity không? Chi phí tăng có đáng không?

3. **Khi nào nên pivot từ Hướng B về Hướng A?** — Nếu sau 6 tháng pilot, Accuracy chỉ đạt 94% thay vì 98%, Primary Usage Rate chỉ đạt 50% thay vì 70% — đó là tín hiệu cần improve thêm hay dấu hiệu Hướng B không khả thi với nguồn lực hiện có? Cần định trước "pivot trigger" rõ ràng để không sunk-cost fallacy.

4. **Làm sao bảo vệ sản phẩm khỏi "một case viral hallucination"?** — Ngoài technical safeguard (verification + kill switch), có cần chuẩn bị crisis playbook, insurance uy tín, hay legal disclaimer mạnh hơn không? Bài học từ các sản phẩm AI khác (Air Canada chatbot thua kiện, NYC MyCity chatbot tư vấn sai luật) cho thấy đây là rủi ro sản phẩm chứ không chỉ rủi ro kỹ thuật.

5. **Co-design với giảng viên ở mức nào là đủ?** — Giảng viên là champion hay là "beta tester"? Họ có được coi là co-founder của feature (có incentive tài chính/academic) hay chỉ là advisor? Answer câu này quyết định tốc độ build eval dataset và chất lượng MoU với trường.
