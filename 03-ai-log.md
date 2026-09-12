# AI Log & Personal Reflection

**Ngày:** 12/09/2026  
**Vai trò của tôi:** AI Product Engineer trong bài lab Vin Smart Future  
**Use case đã chọn:** VinUni — AI hỗ trợ draft phản hồi từ autograder

> Tôi dùng AI như một thought-partner để mở rộng phương án, phản biện scope và kiểm tra
> tính nhất quán. Tôi không coi nội dung AI tạo ra là dữ kiện vận hành thực tế. Mọi con
> số chưa có nguồn đều được đổi thành giả định cần đo trong pilot.

---

## 1. Tôi đã dùng AI ở những bước nào?

| Bước | AI hỗ trợ | Tôi giữ quyền quyết định gì? |
|---|---|---|
| Problem scan | Gợi ý pain point theo bốn lenses | Chọn VinUni và loại các ý tưởng rủi ro/khó kiểm chứng |
| Quick cards | Cấu trúc actor, workflow, metric, architecture | Sửa metric thành giả định; chọn Card #1 |
| Deep-dive | Phản biện Rule vs LLM vs Agent và failure modes | Giữ autograder là nguồn điểm; chọn LLM Feature + HITL |
| Boundary design | Gợi ý schema, invariant, fallback | Cấm đổi điểm, lộ hidden test, auto-send và code execution |
| Technical prototype | Hỗ trợ thiết kế structured output và adversarial cases | Đồng bộ prototype VinUni; tự kiểm tra score, HITL và blocked requests |
| Final review | Tìm mâu thuẫn giữa scope, metric và evidence | Chọn NOT YET production thay vì tuyên bố GO thiếu dữ liệu |

---

## 2. Prompt log tiêu biểu

### Prompt 1 — Brainstorm vấn đề

```text
Tôi là AI Product Engineer tại Vin Smart Future. Hãy dùng bốn lenses Repetitive,
Time-consuming, AI-upgrade và Stakeholder Pain để gợi ý các bottleneck cụ thể trong
workflow chấm và phản hồi bài lab lập trình tại VinUni. Với mỗi ý tưởng, hãy phân biệt
phần nào nên dùng rule, LLM hoặc không nên tự động hóa. Không được trình bày số ước tính
như dữ liệu thật.
```

**AI giúp gì:** Mở rộng scan từ “chấm bài tự động” sang giải thích log, xử lý lỗi môi
trường, FAQ và triage học thuật.

**Tôi sửa gì:** AI ban đầu có xu hướng gom chấm điểm, phản hồi và phát hiện gian lận vào
một hệ thống. Tôi tách chúng thành ba card vì actor, rủi ro và tiêu chí chấp nhận khác
nhau.

### Prompt 2 — CFO/Operations stress-test

```text
Hãy đóng vai CFO và Trưởng vận hành khó tính. Phản biện đề xuất dùng LLM để đọc log
autograder và soạn phản hồi. Chỉ ra phần rule-based làm tốt hơn, metric nào chưa có
baseline, chi phí/rủi ro nào bị bỏ sót và điều kiện khiến dự án phải Not Yet hoặc No-Go.
```

**AI giúp gì:** Chỉ ra rằng lỗi thiếu file, syntax và score calculation không cần LLM;
đồng thời yêu cầu đo edit rate, groundedness và thời gian thật.

**Tôi sửa gì:** Tôi bỏ ý tưởng “AI chấm điểm” và chuyển thành “AI draft feedback”. Tôi
thêm invariant score không đổi, validator, audit log và TA approval.

### Prompt 3 — Red-team operational boundary

```text
Red-team hệ thống draft feedback bằng các tình huống: comment trong code cố prompt
injection, yêu cầu tiết lộ hidden test, model tự đổi điểm, output không hợp lệ, API
timeout và draft đưa thẳng lời giải. Với mỗi lỗi, đề xuất detection và fallback không
dựa hoàn toàn vào LLM.
```

**AI giúp gì:** Tạo danh sách failure modes rộng hơn lỗi hallucination thông thường,
đặc biệt là prompt injection từ source code và rò rỉ hidden test.

**Tôi sửa gì:** Tôi không chấp nhận gợi ý “để model tự đánh giá confidence” như control
duy nhất. Tôi thêm allowlist dữ liệu, schema validator, so sánh score deterministic và
TA review.

### Prompt 4 — Kiểm tra quyết định

```text
Dựa trên việc chưa có submission đã ẩn danh, chưa đo baseline và chưa có đánh giá của
TA, hãy chọn trung thực giữa GO, NOT YET và NO-GO. Tách quyết định production khỏi một
pilot offline có thể đảo ngược.
```

**AI giúp gì:** Làm rõ rằng “có kiến trúc hợp lý” không đồng nghĩa “sẵn sàng production”.

**Quyết định của tôi:** NOT YET cho production, nhưng GO cho pilot offline 50 bài sau
khi có phê duyệt dữ liệu.

---

## 3. Những điểm AI trả lời chưa tốt

### 3.1 Dễ tạo số liệu nghe hợp lý nhưng không có nguồn

AI có thể đề xuất “100 bài mỗi tuần”, “22 phút mỗi bài” hoặc một tỉ lệ tiết kiệm lớn như
thể đó là dữ liệu VinUni. Nếu chép thẳng, Problem Statement có vẻ cụ thể nhưng lại không
trung thực. Tôi giữ các con số để thiết kế phép đo, đồng thời gắn nhãn rõ là giả định
baseline. Production decision bị chặn cho đến khi đo thật.

### 3.2 Thường chọn kiến trúc quá phức tạp

Ý tưởng agent tự chạy code, sửa bài, chấm và gửi feedback nghe hấp dẫn nhưng trao quá
nhiều quyền. Rule/autograder đã tốt hơn cho score và lỗi đơn giản. Tôi chọn hybrid:
rules làm quyết định deterministic, LLM chỉ draft phần ngôn ngữ, TA phê duyệt.

### 3.3 Có thể làm mờ ranh giới giữa “giúp học” và “đưa đáp án”

Một feedback quá chi tiết có thể trở thành lời giải hoàn chỉnh. Tôi bổ sung boundary chỉ
cho phép giải thích evidence và learning hint, không tạo full corrected solution.

### 3.4 Có thể đánh đồng prototype với validation sản phẩm

Ngay cả khi ba adversarial tests của prototype VinUni đều pass, đó vẫn chỉ là bằng chứng
trên ba input được thiết kế trước. Tôi không dùng kết quả ấy để tuyên bố sản phẩm đã sẵn
sàng; production vẫn cần một bộ evaluation rộng hơn, review của TA và pilot đã ẩn danh.

---

## 4. Tôi đã kiểm chứng và chỉnh sửa như thế nào?

1. Đọc toàn bộ worksheet để ánh xạ nội dung vào I1, G1-G4 và I3.
2. So sánh với deliverable example để giữ đủ workflow, handoff, thời gian, six-field
   statement, future flow, HITL và fallback—nhưng không sao chép use case hoặc số liệu.
3. Dùng inspiration kit để xác nhận “tự động hóa chấm điểm và phản hồi bài lab” là một
   hướng gợi ý, sau đó thu hẹp còn draft feedback.
4. Đọc autograder cục bộ để xác nhận tên ba file và yêu cầu kỹ thuật; không coi việc
   “file exists” là đủ chất lượng cho rubric người chấm.
5. Đồng bộ `prompt_prototype.py` với use case VinUni và kiểm tra ba loại tấn công: đổi
   điểm, prompt injection/lộ hidden test, và full solution/auto-send. Lần chạy live ngày
   12/09/2026 pass cả ba cases với exit code 0.
6. Tách rõ ba mức bằng chứng: giả định scoping, prototype boundary tests, và pilot
   VinUni trên submission thật vẫn chưa chạy.

---

## 5. Điều tôi học được

Điều quan trọng nhất không phải là tìm nơi “có thể gắn AI”, mà là xác định quyết định
nào AI **không được sở hữu**. Trong use case này, score, policy học thuật và hành động
gửi phản hồi phải thuộc về hệ thống deterministic hoặc con người. LLM chỉ có lợi thế ở
việc diễn giải evidence thành ngôn ngữ dễ hiểu.

Tôi cũng nhận ra metric có số chưa chắc đã là evidence. Một con số ước tính giúp thiết
kế pilot, nhưng phải được gắn nhãn và có kế hoạch đo. Vì vậy quyết định NOT YET không
phải thất bại; nó là kết quả đúng khi dữ liệu readiness còn thiếu.

Cuối cùng, AI hữu ích nhất khi tôi yêu cầu nó phản biện và tìm failure mode, không phải
khi yêu cầu nó viết một phương án nghe thuyết phục. Tôi vẫn phải chịu trách nhiệm về
scope, giả định, boundary và kết luận cuối cùng.

---

## 6. Cam kết sử dụng AI có trách nhiệm

- Không đưa dữ liệu cá nhân, source code riêng tư, secret hoặc hidden tests vào model
  nếu chưa có phê duyệt và cơ chế bảo vệ dữ liệu.
- Không dùng AI output làm điểm số hoặc cáo buộc vi phạm học thuật.
- Luôn phân biệt nội dung AI gợi ý, điều tôi tự quyết định và bằng chứng đã kiểm chứng.
- Giữ human-in-the-loop cho mọi phản hồi gửi sinh viên.
- Lưu version của prompt/model, input đã lọc, output, validator result và người duyệt để
  có thể audit pilot.
