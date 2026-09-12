# Problem Scan & Quick Problem Cards

**Đơn vị lựa chọn:** VinUni
**Mảng:** Giáo dục — vận hành và phản hồi bài lab lập trình
**Bài toán được chọn để deep-dive:** AI hỗ trợ soạn phản hồi dựa trên kết quả autograder


---

## Phase 1 — SCAN: Danh sách cơ hội

Tôi dùng đủ bốn lenses trong worksheet để quét một workflow quen thuộc: sinh viên nộp
bài lab, hệ thống chấm tự động, TA đọc lỗi và phản hồi cho sinh viên.

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---:|---|---|---|
| 1 | **VinUni** | Lặp lại (Repetitive) | Sau mỗi lần autograder báo lỗi, TA phải đọc log, đối chiếu rubric và viết lại lời giải thích tương tự cho nhiều sinh viên. |
| 2 | **VinUni** | Tốn thời gian (Time-consuming) | TA phải tái hiện lỗi môi trường Python, dependency hoặc đường dẫn trước khi có thể giải thích nguyên nhân cho sinh viên. |
| 3 | **VinUni** | AI có thể tốt hơn (AI-upgrade) | Thông báo kiểu `AssertionError` hoặc `ModuleNotFoundError` chính xác về máy nhưng khó hiểu; sinh viên cần phản hồi có ngữ cảnh và bước tự kiểm tra. |
| 4 | **VinUni** | Pain từ người khác (Stakeholder Pain) | Sinh viên nhận phản hồi muộn hoặc quá ngắn nên lặp lại cùng lỗi ở lần nộp sau; TA cũng bị dồn câu hỏi gần deadline. |
| 5 | **VinUni** | Tốn thời gian (Time-consuming) | Giảng viên phải rà soát thủ công các submission có dấu hiệu bất thường trước khi quyết định có cần kiểm tra học thuật thêm hay không. |
| 6 | **VinUni** | Lặp lại (Repetitive) | Trợ giảng trả lời nhiều câu hỏi giống nhau về cấu trúc thư mục, cách chạy test và quy tắc nộp bài. |

### Tiêu chí shortlist

Tôi ưu tiên vấn đề có đầu vào số hóa sẵn, kết quả có thể kiểm chứng, rủi ro có thể chặn
bằng human-in-the-loop, và không trao quyền quyết định điểm cho mô hình. Ba cơ hội được
đưa vào Quick Cards là #1, #2 và #5.

---

## Phase 2 — QUICK-ASSESS

## Quick Problem Card #1 — Giải thích lỗi autograder

```text
┌──────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                        │
│                                                              │
│ Bài toán: TA mất nhiều thời gian chuyển log autograder thành │
│ phản hồi dễ hiểu, có căn cứ và có bước tự sửa cho sinh viên. │
│ Công ty thành viên: [x] Khác — VinUni                        │
│                                                              │
│ Ai đang đau? TA/giảng viên và sinh viên học lập trình.       │
│                                                              │
│ Workflow hiện tại:                                           │
│ 1. Nhận submission → 2. Chạy autograder → 3. Đọc/reproduce   │
│ lỗi → 4. Đối chiếu rubric → 5. Viết và gửi phản hồi.         │
│                                                              │
│ Bước tốn nhất: Bước 3-5 (giả định 17 phút/submission).       │
│ AI hỗ trợ: Tóm tắt log, trích căn cứ, draft giải thích và    │
│ gợi ý bước kiểm tra; TA bắt buộc duyệt trước khi gửi.        │
│                                                              │
│ Metric pilot: giảm median 22 → ≤8 phút/submission; 100%      │
│ điểm số giữ nguyên; ≥90% draft được TA đánh giá có căn cứ.   │
│                                                              │
│ Quick Architecture: [x] LLM Feature + rules + HITL           │
└──────────────────────────────────────────────────────────────┘
```

### Stress-test Card #1

- **Điểm yếu logic:** Không phải mọi lỗi đều cần LLM; lỗi cú pháp hoặc thiếu file có
  thể ánh xạ bằng rule/template nhanh hơn và ổn định hơn.
- **Điểm yếu metric:** Baseline 22 phút mới là giả định; phải đo timestamp thật thay
  vì tuyên bố đó là hiệu suất hiện tại của VinUni.
- **Rủi ro:** Mô hình có thể suy diễn nguyên nhân không xuất hiện trong log hoặc vô tình
  tiết lộ hidden test. Vì vậy output chỉ là draft, phải kèm evidence lines và không được
  thay đổi điểm.

---

## Quick Problem Card #2 — Hỗ trợ chẩn đoán môi trường lab

```text
┌──────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                        │
│                                                              │
│ Bài toán: Sinh viên và TA tốn thời gian xử lý lỗi dependency,│
│ Python version, biến môi trường và sai working directory.    │
│ Công ty thành viên: [x] Khác — VinUni                        │
│                                                              │
│ Ai đang đau? Sinh viên mới học Python và TA trực lab.        │
│                                                              │
│ Workflow hiện tại:                                           │
│ 1. Sinh viên gửi screenshot → 2. TA hỏi lại context →        │
│ 3. Thu thập version/cwd → 4. Thử lệnh chẩn đoán →            │
│ 5. Gửi hướng dẫn và chờ xác nhận.                            │
│                                                              │
│ Bước tốn nhất: Bước 2-4 (giả định 15 phút/ticket).           │
│ AI hỗ trợ: Phân loại lỗi và draft checklist chẩn đoán.       │
│                                                              │
│ Metric pilot: ≥80% ticket được phân loại đúng nhóm; median   │
│ first-response ≤3 phút; không tự chạy lệnh phá hủy.          │
│                                                              │
│ Quick Architecture: [x] Rule trước, [x] LLM Feature sau      │
└──────────────────────────────────────────────────────────────┘
```

### Stress-test Card #2

- Phần thu thập `python --version`, `pwd`, file tồn tại hay không phù hợp với script
  deterministic hơn LLM.
- LLM chỉ có ích khi giải thích log không đồng nhất và điều chỉnh hướng dẫn theo trình
  độ người học.
- Hệ thống tuyệt đối không được đề xuất xóa môi trường, cache hoặc file nếu chưa có xác
  nhận và phạm vi cụ thể.

---

## Quick Problem Card #3 — Triage dấu hiệu vi phạm học thuật

```text
┌──────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                        │
│                                                              │
│ Bài toán: Giảng viên cần ưu tiên những submission nên được   │
│ kiểm tra thêm, thay vì đọc toàn bộ bài theo cách thủ công.   │
│ Công ty thành viên: [x] Khác — VinUni                        │
│                                                              │
│ Ai đang đau? Giảng viên, TA và sinh viên bị gắn cờ nhầm.     │
│                                                              │
│ Workflow hiện tại:                                           │
│ 1. Thu bài → 2. So sánh dấu hiệu → 3. Đọc code/report →      │
│ 4. Trao đổi với sinh viên → 5. Giảng viên quyết định.        │
│                                                              │
│ Bước tốn nhất: Bước 2-3 (giả định 20 phút/case).             │
│ AI hỗ trợ: Xếp ưu tiên review và nêu bằng chứng, không kết tội│
│                                                              │
│ Metric pilot: recall ≥95% trên bộ case đã được gán nhãn;     │
│ false-positive ≤10%; 100% case có người quyết định.          │
│                                                              │
│ Quick Architecture: [x] Rules + similarity, LLM chỉ giải thích│
└──────────────────────────────────────────────────────────────┘
```

### Stress-test Card #3

- Đây là bài toán rủi ro cao: false positive có thể ảnh hưởng uy tín và quyền lợi của
  sinh viên.
- Rule/similarity có khả năng audit tốt hơn LLM cho bước phát hiện ban đầu.
- Không nên chọn làm prototype đầu tiên khi chưa có policy, dữ liệu gán nhãn và quy
  trình khiếu nại rõ ràng.

---

## Quyết định lựa chọn

Tôi chọn **Card #1 — AI hỗ trợ soạn phản hồi dựa trên kết quả autograder** để deep-dive.

| Card | Giá trị | Khả năng kiểm chứng | Rủi ro | Quyết định |
|---|---|---|---|---|
| #1 Phản hồi autograder | Cao, tác vụ lặp lại | Cao vì có log/rubric làm nguồn | Trung bình, chặn bằng HITL | **Chọn** |
| #2 Chẩn đoán môi trường | Trung bình | Cao | Trung bình nếu đề xuất lệnh sai | Giữ làm phase sau |
| #3 Triage học thuật | Cao | Thấp khi chưa có labels/policy | Rất cao | Không chọn lúc này |

Card #1 có scope hẹp nhất: autograder vẫn là nguồn kết quả chính thức; LLM không chấm
điểm mà chỉ chuyển evidence có cấu trúc thành phản hồi dễ hiểu. Điều này tạo giá trị
ngôn ngữ phù hợp với LLM nhưng vẫn giữ quyết định học thuật cho con người.
