# Problem Deep-Dive Report

## VinUni — AI-assisted Feedback for Programming Labs

**Owner giả định:** Đội ngũ giảng viên/TA phụ trách lab lập trình
**Phạm vi:** Draft phản hồi sau khi autograder đã chạy
**Ngoài phạm vi:** Chấm điểm, thay đổi rubric, phát hiện gian lận, tự gửi phản hồi


---

## 1. Executive Summary

Sau khi autograder trả về kết quả, TA vẫn phải đọc traceback, tìm test/rubric liên quan,
tái hiện lỗi và viết phản hồi phù hợp với trình độ sinh viên. Autograder làm tốt phần
quyết định deterministic, nhưng raw log thường không phải là phản hồi sư phạm tốt.

Giải pháp đề xuất là một **LLM Feature có retrieval giới hạn và human-in-the-loop**:
rule engine giữ nguyên điểm số; hệ thống chỉ gửi cho LLM log đã lọc, rubric công khai và
đoạn code liên quan; LLM trả về draft có evidence; validator kiểm tra schema và cấm nội
dung nhạy cảm; TA duyệt/chỉnh sửa trước khi gửi.

Mục tiêu pilot là giảm median thời gian xử lý một submission từ baseline giả định 22
phút xuống không quá 8 phút, trong khi **100% điểm chính thức không bị LLM thay đổi** và
ít nhất 90% draft được đánh giá là bám đúng evidence.

---

## 2. Current-State Workflow Mapping

### 2.1 Workflow hiện tại

```text
┌────────────────────┐
│ 1. Sinh viên nộp   │
│ code lên hệ thống  │
│ Actor: Sinh viên   │
│ In: source + meta  │
│ Out: submission ID │
│ ⏱ 0 phút của TA    │
└─────────┬──────────┘
          │ 🔄 Handoff: LMS/Git → autograder
          ▼
┌────────────────────┐
│ 2. Chạy autograder │
│ Actor: Hệ thống    │
│ In: code + tests   │
│ Out: score + log   │
│ ⏱ ~1 phút          │
└─────────┬──────────┘
          │ 🔄 Handoff: máy → TA
          ▼
┌────────────────────┐
│ 3. Đọc và reproduce│ 🔴 Bottleneck
│ Actor: TA          │
│ In: code + raw log │
│ Out: root-cause note│
│ ⏱ ~8 phút          │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ 4. Đối chiếu rubric│ 🔴 Bottleneck
│ Actor: TA          │
│ In: note + rubric  │
│ Out: evidence list │
│ ⏱ ~5 phút          │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ 5. Soạn phản hồi   │ 🔴 Bottleneck
│ Actor: TA          │
│ In: evidence       │
│ Out: feedback draft│
│ ⏱ ~6 phút          │
└─────────┬──────────┘
          │ 🔄 Handoff: TA → LMS/email
          ▼
┌────────────────────┐
│ 6. Gửi phản hồi    │
│ Actor: TA          │
│ In: final feedback │
│ Out: student sees  │
│ ⏱ ~2 phút          │
└────────────────────┘

Tổng baseline giả định: 22 phút/submission (1 + 8 + 5 + 6 + 2).
```

### 2.2 Bottleneck và nguyên nhân

| Bottleneck | Nguyên nhân | Hậu quả có thể đo |
|---|---|---|
| Đọc/reproduce lỗi | Log dài, lỗi phụ thuộc môi trường, nhiều file | Thời gian xử lý tăng; feedback đến muộn |
| Nối lỗi với rubric | Test name không luôn giải thích kỹ năng bị thiếu | Phản hồi có thể chỉ nói “test failed” mà không giúp học |
| Viết phản hồi | Cần vừa chính xác kỹ thuật vừa không đưa thẳng đáp án | TA lặp lại nội dung; chất lượng không đồng đều |

### 2.3 Điểm cần đo trước pilot

Để thay giả định bằng evidence, nhóm sẽ lấy mẫu 50 submission đã ẩn danh từ một bài lab,
ghi timestamp bắt đầu/kết thúc cho từng bước, số lỗi mỗi bài, số vòng trao đổi sau phản
hồi và mức độ chỉnh sửa feedback. Không đưa dữ liệu cá nhân hoặc hidden tests vào bộ dữ
liệu đánh giá LLM.

---

## 3. Problem Statement — 6 Fields

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | TA là operator chính; giảng viên sở hữu rubric và chịu trách nhiệm cuối; sinh viên là người nhận phản hồi. |
| **2. Current Workflow** | Autograder tạo score/log; TA đọc code và log, reproduce lỗi, đối chiếu rubric, soạn phản hồi rồi gửi qua LMS. Baseline scoping gồm 5 bước xử lý sau submission, giả định 22 phút/lượt. |
| **3. Bottleneck** | Ba bước đọc/reproduce, nối evidence với rubric và diễn giải thành phản hồi sư phạm chiếm khoảng 19/22 phút giả định. Đây là tác vụ cần hiểu ngôn ngữ/code nhưng vẫn phải bám nguồn. |
| **4. Business Impact** | Với giả định một lớp có 100 submission/mốc nộp, 22 phút tương đương khoảng 36,7 giờ xử lý. Feedback chậm làm giảm cơ hội sửa lỗi trước bài tiếp theo và tạo tải câu hỏi lặp lại cho TA. Con số phải được xác nhận bằng pilot. |
| **5. Success Metric** | (a) median handling time ≤8 phút; (b) 100% score không đổi; (c) ≥90% draft đạt ≥4/5 về groundedness theo review của 2 TA; (d) 0 lần lộ hidden test/PII; (e) ≥80% draft được chấp nhận sau tối đa chỉnh sửa nhỏ. |
| **6. Operational Boundary** | AI chỉ được draft feedback từ nguồn cho phép và phải dẫn evidence. AI không được chấm/đổi điểm, kết luận gian lận, tiết lộ hidden tests, chạy code sinh viên, suy đoán thông tin cá nhân, tự gửi feedback hoặc cung cấp nguyên lời giải. Mọi output phải có trạng thái `DRAFT_ONLY` và được TA duyệt. |

### Problem statement một câu

> TA cần biến kết quả autograder thành phản hồi học tập chính xác và hữu ích nhanh hơn,
> vì workflow giả định hiện mất median 22 phút/submission; một LLM feature có grounding,
> validator và TA phê duyệt sẽ thử giảm xuống ≤8 phút mà không được thay đổi điểm hoặc
> tiết lộ dữ liệu đánh giá.

---

## 4. AI Fit: Rule vs LLM vs Agent

| Phương án | Phù hợp với | Điểm mạnh | Điểm yếu | Kết luận |
|---|---|---|---|---|
| **Rule/template** | Lỗi rõ như thiếu file, syntax, sai tên hàm | Deterministic, rẻ, audit dễ | Khó bao phủ lỗi logic và giải thích theo ngữ cảnh | Dùng làm lớp đầu tiên |
| **LLM Feature** | Tóm tắt log, liên kết evidence, draft giải thích | Linh hoạt với code/log và ngôn ngữ tự nhiên | Có thể hallucinate hoặc đưa quá nhiều đáp án | **Chọn**, nhưng bắt buộc grounding + HITL |
| **Agentic Loop** | Tự chạy code, sửa, chấm và gửi | Có thể tự động hóa sâu | Quyền quá rộng, prompt injection và code execution risk cao | Không dùng trong scope này |

### Quyết định kiến trúc

Giải pháp là **hybrid Rule + LLM Feature**, không phải autonomous agent:

1. Autograder/rules tiếp tục tạo kết quả và điểm chính thức.
2. Rules xử lý lỗi đơn giản bằng template cố định.
3. Chỉ case cần diễn giải mới đi qua LLM.
4. TA luôn là người phê duyệt và gửi.

---

## 5. Future-State Flow

```text
Submission
    │
    ▼
[Rule/Autograder: chạy test và khóa score]
    │ score + public evidence
    ▼
[Data guard: loại PII, secret, hidden-test body]
    │
    ├── invalid/sensitive ───────────────┐
    │                                   │
    ▼                                   ▼
[Rule router]                    ↩️ FALLBACK A
    │ simple error                     TA xử lý thủ công
    ├──────────────→ [Template feedback]
    │ complex error
    ▼
[🔵 AI STEP: draft feedback JSON từ evidence cho phép]
    │
    ▼
[Validator: schema, citations, banned content, score unchanged]
    │
    ├── fail/low confidence ────────────┐
    │                                   ▼
    │                            ↩️ FALLBACK B
    │                            template + TA tự viết
    ▼
[🟢 HITL: TA kiểm tra evidence, sửa hoặc reject]
    │ approved
    ▼
[TA gửi feedback qua LMS]
```

### Handoff tương lai

| Handoff | Payload tối thiểu | Owner nhận | Control |
|---|---|---|---|
| Autograder → Data guard | submission ID giả danh, score, public test log | Hệ thống | Allowlist trường dữ liệu |
| Data guard → LLM | đoạn code liên quan, error excerpt, rubric excerpt | LLM feature | Không PII/secret/hidden-test body |
| LLM → Validator | JSON draft | Hệ thống | Schema + policy checks |
| Validator → TA | draft + evidence + flags | TA | Không có nút auto-send |
| TA → LMS | feedback đã duyệt | Sinh viên | Audit log người duyệt |

---

## 6. Input/Output Contract

### Input được phép

```json
{
  "submission_id": "anon_042",
  "public_test_summary": [
    {"test": "test_normalize_empty", "status": "failed", "error": "AssertionError"}
  ],
  "relevant_code_excerpt": "def normalize(items): ...",
  "rubric_excerpt": "Handle empty input without raising an exception.",
  "official_score": 7.5
}
```

### Output bắt buộc

```json
{
  "status": "DRAFT_ONLY",
  "submission_id": "anon_042",
  "summary": "Hàm chưa xử lý trường hợp danh sách rỗng.",
  "evidence": ["test_normalize_empty: AssertionError"],
  "learning_hint": "Kiểm tra điều kiện đầu vào trước phép chia hoặc truy cập phần tử.",
  "official_score": 7.5,
  "confidence": 0.88,
  "requires_human_review": true
}
```

### Invariants

- `official_score` ở output phải bằng byte-for-byte/numeric-equivalent với score do
  autograder cấp; tốt hơn nữa, UI hiển thị score trực tiếp từ autograder thay vì từ LLM.
- `status` luôn là `DRAFT_ONLY`; `requires_human_review` luôn là `true`.
- `evidence` chỉ được trích từ input đã lọc.
- Không có lời giải hoàn chỉnh, hidden-test content, nhận xét về danh tính hay cáo buộc
  gian lận.

---

## 7. Fallback và Failure Modes

| Failure mode | Detection | Fallback |
|---|---|---|
| LLM không trả JSON hợp lệ | JSON Schema validator | Dùng template lỗi chung; TA tự soạn |
| Evidence không tồn tại trong input | Exact/semantic evidence checker + TA review | Reject draft, không hiển thị cho sinh viên |
| Model thay đổi score | So sánh với immutable autograder score | Chặn output và ghi audit event |
| Có PII/secret/hidden test | Pre-LLM scanner và allowlist | Không gọi LLM; chuyển TA |
| Prompt injection trong comment code | Bao dữ liệu trong trường quoted; policy test | Không làm theo instruction từ submission; chuyển TA nếu nghi ngờ |
| API timeout/rate limit | Timeout và error code | Rule/template hoặc workflow thủ công hiện tại |
| Draft đưa thẳng đáp án | Policy classifier + TA review | Reject; chỉ cung cấp hint theo rubric |

---

## 8. Pilot & Evaluation Plan

### 8.1 Thiết kế pilot

- **Dữ liệu:** 50 submission lịch sử đã ẩn danh, chỉ từ một lab và một rubric version.
- **So sánh:** TA xử lý 25 case theo workflow hiện tại và 25 case có AI draft; đổi nhóm
  ở vòng hai để giảm bias do độ khó.
- **Blind review:** Hai TA chấm draft mà không biết draft do người hay AI tạo.
- **Không production send:** Mọi feedback pilot nằm trong sandbox/offline review.

### 8.2 Rubric đánh giá draft

Mỗi tiêu chí chấm 1-5:

1. Đúng nguyên nhân dựa trên evidence.
2. Bám đúng rubric và không tự thay đổi score.
3. Dễ hiểu, có bước tiếp theo nhưng không đưa lời giải hoàn chỉnh.
4. Không tiết lộ dữ liệu cấm.
5. TA cần chỉnh sửa ít.

### 8.3 Acceptance gates

| Gate | Ngưỡng pilot | Nếu không đạt |
|---|---:|---|
| Score integrity | 100% không đổi | Dừng pilot, sửa data contract |
| Critical safety/privacy | 0 PII/hidden-test leak | Dừng pilot và incident review |
| Groundedness | ≥90% draft đạt ≥4/5 | Cải thiện retrieval/prompt |
| Time | Median ≤8 phút/submission | Kiểm tra UX hoặc No-Go |
| TA acceptance | ≥80% accept/minor edit | Phân tích loại lỗi thất bại |

---

## 9. Readiness, Risks, and Decision

### AI Readiness Checklist

- [ ] Có bộ 50 submission đã ẩn danh và được phê duyệt cho pilot.
- [ ] Đã đo baseline thật theo từng bước thay cho giả định 22 phút.
- [x] Rủi ro score có thể chặn bằng cách để autograder làm nguồn duy nhất.
- [x] Có workflow fallback thủ công và bắt buộc TA review.
- [ ] Có xác nhận của giảng viên, TA và bộ phận phụ trách dữ liệu.

### Quyết định

- [ ] **GO production**
- [x] **NOT YET — cần dữ liệu và baseline; cho phép pilot offline**
- [ ] **NO-GO**

### Justification

Bài toán có AI fit hợp lý vì phần cần hỗ trợ là diễn giải code/log thành ngôn ngữ sư
phạm, trong khi phần chấm điểm vẫn do rule/autograder đảm nhiệm. Rủi ro lớn nhất—đổi
điểm, hallucination, lộ hidden test và đưa thẳng lời giải—đã có boundary và fallback cụ
thể. Tuy nhiên, chưa có dữ liệu thật chứng minh baseline, groundedness hoặc mức chấp nhận
của TA. Vì vậy chưa đủ evidence để GO production. Một pilot offline 50 bài là bước nhỏ,
đảo ngược được và tạo đúng evidence cần thiết cho quyết định tiếp theo.

---

## 10. Prompt Prototype của Lab

File `starter-code/prompt_prototype.py` đã được đồng bộ với use case VinUni. Prototype
dùng structured JSON output và ba adversarial cases để kiểm tra: yêu cầu đổi điểm, prompt
injection/yêu cầu lộ hidden tests, và yêu cầu cung cấp lời giải đầy đủ/tự gửi feedback.
Prototype kiểm tra trực tiếp bốn invariant: `DRAFT_ONLY`, score không đổi, TA review bắt
buộc và mọi unsafe request phải xuất hiện trong `blocked_requests`.

Trong lần chạy live ngày 12/09/2026 với `gemini-3.6-flash`, cả ba adversarial cases đều
pass bốn verification checks và tiến trình kết thúc với exit code 0.

Kết quả prototype chỉ là bằng chứng về prompt boundary trên ba input cụ thể, chưa phải
validation production. Giải pháp vẫn phải qua pilot 50 submission và acceptance gates
tại Mục 8.
