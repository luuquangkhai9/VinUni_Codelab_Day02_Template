# Vin Smart Future — Problem Deep-Dive Report

**Đơn vị:** Vinpearl / VinWonders  
**Bài toán:** AI hỗ trợ tìm kiếm và khớp hồ sơ đồ thất lạc  
**Phạm vi:** Prototype hỗ trợ nhân viên Lost & Found; không tự xác nhận chủ sở hữu hoặc bàn giao tài sản

> Các số liệu trong báo cáo là giả định scoping cho một cơ sở quy mô trung bình, kế thừa từ `01-problem-scan.md`. Chúng chưa phải số liệu vận hành được Vinpearl/VinWonders xác nhận và phải được đo lại trong pilot.

## 1. Tóm tắt điều hành

Hiện nay, báo mất đồ và biên bản đồ nhặt được có thể nằm rải rác trong sổ, Excel, nhóm chat và thư mục ảnh. Nhân viên phải diễn giải các mô tả không đồng nhất rồi dò từng bản ghi. Bước tìm kiếm mất khoảng **15 phút/vụ**, tương đương **6,25 giờ công/ngày** nếu có 25 yêu cầu.

Nhóm đề xuất một công cụ hỗ trợ kết hợp **metadata rules + tìm kiếm ngữ nghĩa/ảnh + LLM**. Hệ thống chuẩn hóa mô tả, lọc theo thời gian/địa điểm, xếp hạng 3–5 món đồ có khả năng khớp và nêu bằng chứng. Nhân viên vẫn chịu trách nhiệm kiểm tra biên bản, xác minh người nhận và quyết định bàn giao.

Mục tiêu pilot là giảm thời gian tìm kiếm từ 15 xuống dưới 5 phút/vụ và đạt **Recall@5 ≥ 80%**, trong khi không phát sinh trường hợp AI tự xác nhận hoặc tự bàn giao.

## 2. Current-State Workflow Mapping

### 2.1. Luồng vận hành hiện tại

```text
Khách báo mất đồ
      │
      ▼
┌──────────────────────────────┐
│ 1. Tiếp nhận yêu cầu         │
│ Actor: Lễ tân/CSKH           │
│ Input: gọi điện, email, form │
│ Output: ghi chú ban đầu      │
│ Thời gian: 3 phút            │
└──────────────┬───────────────┘
               │ 🔄 Handoff: khách → lễ tân/CSKH
               ▼
┌──────────────────────────────┐
│ 2. Hỏi và chuẩn hóa mô tả    │
│ Loại, màu, đặc điểm, vị trí, │
│ thời gian, thông tin liên hệ │
│ Thời gian: 8 phút            │
└──────────────┬───────────────┘
               │ 🔄 Handoff: CSKH → Lost & Found
               ▼
┌──────────────────────────────┐
│ 3. Nhập/chuyển dữ liệu       │
│ Excel, sổ, chat, thư mục ảnh │
│ Thời gian: 4 phút            │
└──────────────┬───────────────┘
               │ 🔄 Handoff: Lost & Found ↔ An ninh/Kho
               ▼
┌──────────────────────────────┐
│ 4. Dò bản ghi và ảnh         │
│ Tìm theo từ khóa thủ công    │
│ Thời gian: 15 phút 🔴        │
└──────────────┬───────────────┘
               │
        ┌──────┴──────┐
        │ Có ứng viên?│
        └───┬─────┬───┘
          Có│     │Không
            ▼     └──────────────► Ghi pending và tìm thủ công tiếp
┌──────────────────────────────┐
│ 5. Gọi khách xác minh        │
│ Kiểm tra chi tiết không công │
│ khai và hướng dẫn nhận đồ    │
│ Thời gian: 7 phút            │
└──────────────┬───────────────┘
               │ 🔄 Handoff: Lost & Found → An ninh/Kho
               ▼
┌──────────────────────────────┐
│ 6. Duyệt và bàn giao         │
│ Kiểm tra giấy tờ, ký biên bản│
│ Thời gian: 5 phút            │
└──────────────────────────────┘
```

**Tổng thời gian xử lý khi tìm thấy ứng viên:** khoảng **42 phút/vụ**, chưa tính thời gian chờ giữa các handoff.  
**Bottleneck chính:** Bước 4 — dò dữ liệu và ảnh thủ công, khoảng **15 phút/vụ**.  
**Bottleneck phụ:** Dữ liệu đầu vào thiếu chuẩn ở bước 2–3 khiến từ khóa không khớp và phải hỏi lại khách.

### 2.2. Nguyên nhân gốc

- Một món đồ có nhiều cách mô tả: “túi đeo chéo đen”, “ví nhỏ màu tối”, “black pouch”.
- Vị trí được ghi theo tên nội bộ hoặc cách gọi của khách, ví dụ “gần trò tàu lượn” và “khu cảm giác mạnh”.
- Dữ liệu văn bản và ảnh không có một mã hồ sơ thống nhất xuyên suốt các bộ phận.
- Tìm kiếm từ khóa chính xác không hiểu từ đồng nghĩa, lỗi chính tả hoặc mô tả đa ngôn ngữ.
- Trạng thái bàn giao cập nhật chậm làm nhân viên tìm cả những món đã được trả.

## 3. Problem Statement — 6 Fields

| Field | Nội dung thống nhất |
|---|---|
| **1. Actor / Operator** | Nhân viên Lost & Found là người tìm và đối chiếu chính; lễ tân/CSKH tiếp nhận báo mất; an ninh và nhân viên kho quản lý biên bản, ảnh và bàn giao. |
| **2. Current Workflow** | Nhân viên nhận mô tả qua cuộc gọi/email/form, hỏi lại thông tin, nhập vào Excel/sổ, tìm theo từ khóa trong nhiều nguồn, xem ảnh rồi gọi khách xác minh. Quy trình khi tìm thấy ứng viên mất khoảng 42 phút/vụ; riêng bước tìm kiếm khoảng 15 phút. |
| **3. Bottleneck** | Dò thủ công các bản ghi có mô tả không thống nhất giữa Excel, sổ, chat và ảnh. Tìm kiếm exact-keyword bỏ sót từ đồng nghĩa, lỗi chính tả và mô tả Việt/Anh. |
| **4. Business Impact** | Với giả định 25 yêu cầu/ngày, riêng việc tìm kiếm tiêu tốn khoảng 6,25 giờ công/ngày. Nếu giảm 10 phút/vụ, có thể tiết kiệm khoảng 4,17 giờ/ngày. Với giả định 60.000 VND/giờ và 26 ngày vận hành/tháng, giá trị thời gian giải phóng khoảng **6,5 triệu VND/tháng/cơ sở**, chưa tính lợi ích trải nghiệm khách. |
| **5. Success Metric** | (1) Median search time ≤ **5 phút/vụ**; (2) Recall@5 ≥ **80%** trên bộ test đã gán nhãn; (3) ≥ **90%** kết quả có lý do và link bản ghi nguồn; (4) **0** trường hợp hệ thống tự xác nhận chủ sở hữu/tự bàn giao; (5) ≥ **80%** nhân viên pilot đánh giá gợi ý hữu ích. |
| **6. Operational Boundary** | AI được phép chuẩn hóa thuộc tính, lọc, xếp hạng ứng viên và soạn câu hỏi xác minh dạng draft. AI không được tự kết luận món đồ thuộc về ai, tiết lộ đặc điểm bí mật dùng để xác minh, tự gửi thông báo cuối cùng, sửa/xóa hồ sơ gốc, thay đổi trạng thái kho hoặc phê duyệt bàn giao. Các quyết định này bắt buộc có nhân viên. |

## 4. AI Fit

### 4.1. So sánh phương án

| Phương án | Phù hợp với | Điểm mạnh | Hạn chế | Kết luận |
|---|---|---|---|---|
| **Rule / State Machine** | Lọc khoảng thời gian, khu vực, trạng thái kho; kiểm tra trường bắt buộc | Dễ audit, nhanh, chi phí thấp | Không hiểu mô tả tự do, từ đồng nghĩa hoặc đa ngôn ngữ | **Bắt buộc dùng** làm lớp lọc và bảo vệ |
| **LLM + Embedding/Multimodal Search** | Chuẩn hóa mô tả, tìm ngữ nghĩa, so sánh mô tả/ảnh, giải thích lý do khớp | Xử lý ngôn ngữ không đồng nhất tốt hơn exact-keyword | Có thể hallucinate, cần đánh giá retrieval và bảo vệ dữ liệu | **Chọn** cho bước hỗ trợ tìm kiếm |
| **Agentic Loop** | Tự gọi nhiều hệ thống, cập nhật trạng thái, gửi tin và điều phối workflow | Tự động hóa cao | Scope và rủi ro vượt nhu cầu; khó kiểm soát hành động ghi/gửi | **Không chọn** cho pilot |

### 4.2. Kiến trúc được chọn

Chọn kiến trúc **hybrid retrieval assistant**, không phải agent tự trị:

1. Rule kiểm tra trường bắt buộc và giới hạn thời gian/địa điểm.
2. Bộ tìm kiếm metadata lọc ứng viên đang còn trong kho.
3. Embedding/multimodal search tính tương đồng văn bản và ảnh.
4. LLM chuẩn hóa mô tả, tổng hợp bằng chứng và xếp hạng 3–5 ứng viên.
5. Nhân viên xem bản ghi nguồn, xác minh và quyết định.

LLM không phải nguồn sự thật. **Biên bản gốc và trạng thái kho** là system of record.

## 5. Future-State Flow

```text
1. Nhân viên nhận báo mất
        │
        ▼
2. 🔵 AI trích xuất thuộc tính
   loại/màu/thời gian/vị trí/đặc điểm
        │
        ▼
3. Rule kiểm tra dữ liệu bắt buộc
        │
   Thiếu├──────────────► 🟢 Nhân viên hỏi bổ sung
        │Đủ
        ▼
4. Rule lọc thời gian, địa điểm,
   trạng thái "còn trong kho"
        │
        ▼
5. 🔵 Semantic/image search xếp hạng
   3–5 ứng viên kèm score + evidence
        │
        ▼
6. 🟢 Nhân viên mở bản ghi nguồn,
   kiểm tra ảnh/biên bản và chọn ứng viên
        │
   Không chắc ──────────► ↩️ Tìm thủ công / chuyển quản lý
        │Có ứng viên
        ▼
7. 🔵 AI soạn câu hỏi xác minh dạng DRAFT
        │
        ▼
8. 🟢 Nhân viên duyệt, liên hệ khách,
   kiểm tra thông tin không công khai
        │
        ▼
9. 🟢 An ninh/Kho kiểm tra giấy tờ,
   ký biên bản và bàn giao
```

### Phân chia trách nhiệm

| Bước | AI được làm | Con người phải làm |
|---|---|---|
| Tiếp nhận | Trích xuất và đề nghị bổ sung trường thiếu | Xác nhận dữ liệu với khách |
| Tìm kiếm | Lọc, tính tương đồng, xếp hạng và dẫn nguồn | Xem hồ sơ/ảnh gốc, chọn ứng viên |
| Xác minh | Soạn draft câu hỏi theo policy | Liên hệ khách và đánh giá câu trả lời |
| Bàn giao | Không tham gia quyết định | Kiểm tra giấy tờ, phê duyệt, ký nhận |

## 6. Guardrails và Fallback

### Guardrails

- Mỗi gợi ý phải có `record_id`, nguồn, trạng thái kho, điểm tương đồng và lý do khớp.
- Chỉ truy xuất hồ sơ trong phạm vi cơ sở và vai trò của nhân viên đang đăng nhập.
- Không hiển thị toàn bộ đặc điểm bí mật cho người báo mất; dùng chúng làm câu hỏi xác minh.
- Output luôn là gợi ý/draft, không dùng ngôn ngữ khẳng định như “đây chắc chắn là đồ của khách”.
- Mọi thay đổi trạng thái và thao tác gửi phải qua giao diện xác nhận của nhân viên.
- Lưu audit log gồm query, ứng viên trả về, lựa chọn của nhân viên và phiên bản model.

### Điều kiện fallback

Hệ thống trả về **“Cần nhân viên kiểm tra thủ công”** khi:

- Thiếu loại đồ, khoảng thời gian hoặc địa điểm.
- Không có ứng viên đạt ngưỡng tương đồng pilot.
- Hai ứng viên có điểm quá gần nhau và không đủ bằng chứng phân biệt.
- Hồ sơ liên quan tài sản giá trị cao, giấy tờ tùy thân, tranh chấp hoặc nghi ngờ gian lận.
- Dịch vụ model/search lỗi, timeout hoặc không truy cập được system of record.

Fallback là quy trình tìm kiếm thủ công hiện tại; hồ sơ không bị mất và AI không được tự bỏ qua bước xác minh.

## 7. Kế hoạch đo lường Pilot

### Dữ liệu cần chuẩn bị

- 200–500 cặp báo mất–đồ nhặt được đã ẩn danh và được nhân viên xác nhận nhãn đúng.
- Các case không có đồ khớp để đo false positive.
- Dữ liệu gồm tiếng Việt, tiếng Anh, lỗi chính tả, mô tả thiếu và nhiều loại địa điểm.
- Ảnh đã loại metadata nhạy cảm; quyền truy cập theo cơ sở.

### Thiết kế đánh giá

1. Chia dữ liệu theo thời gian thành tập phát triển và tập test khóa kín.
2. So sánh ba baseline: tìm từ khóa, metadata rules, hybrid AI.
3. Đo Recall@1, Recall@5, median search time và tỷ lệ nhân viên chấp nhận gợi ý.
4. Chạy shadow mode: AI chỉ đề xuất, không tác động workflow thật.
5. Chỉ mở pilot có nhân viên sau khi đạt Recall@5 ≥ 80% và kiểm thử guardrail đạt 100% trên bộ test an toàn.

## 8. AI Readiness Checklist

| Câu hỏi | Trạng thái | Bằng chứng / việc cần làm |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test? | **Chưa xác nhận** | Cần xuất và ẩn danh 200–500 hồ sơ, chuẩn hóa `record_id`, timestamp, location và trạng thái bàn giao. |
| Rủi ro khi AI sai có kiểm soát được? | **Có, trong scope pilot** | AI chỉ xếp hạng; nhân viên xem nguồn, xác minh và quyết định; có fallback thủ công. |
| Stakeholder sẵn sàng đổi workflow? | **Chưa xác nhận** | Cần workshop với Lost & Found, an ninh, lễ tân và Data Protection; pilot shadow mode trước. |

## 9. Quyết định cuối cùng

### **GO — xây dựng prototype phạm vi hẹp, chưa tự động hóa production**

**Lý do:**

- Pain point lặp lại và đo được: giả định 15 phút tìm kiếm/vụ, khoảng 6,25 giờ công/ngày.
- AI phù hợp với phần hiểu mô tả không đồng nhất; rule/search truyền thống vẫn giữ vai trò lọc và bảo vệ.
- Có thể đánh giá offline trên hồ sơ đã gán nhãn bằng Recall@5 và thời gian tìm kiếm.
- Sai sót được giới hạn vì AI không có quyền xác nhận, gửi, sửa trạng thái hoặc bàn giao.

**Điều kiện trước khi pilot với dữ liệu thật:** hoàn tất đánh giá quyền riêng tư, có bộ dữ liệu ẩn danh đủ chất lượng, thống nhất policy xác minh và đạt các ngưỡng test nêu trên. Nếu không thu thập được dữ liệu gán nhãn hoặc Recall@5 dưới 80%, quyết định chuyển thành **NOT YET** và ưu tiên chuẩn hóa dữ liệu + metadata search trước.

## 10. Ngoài phạm vi

- Nhận diện khuôn mặt hoặc theo dõi khách qua camera.
- Tự động kết luận quyền sở hữu.
- Tự động liên hệ khách, sửa hồ sơ, thay đổi trạng thái kho hoặc bàn giao đồ.
- Dùng dữ liệu giữa các cơ sở khi chưa có quyền và chính sách lưu trữ phù hợp.
- Thay thế vai trò xác minh của lễ tân, an ninh hoặc Lost & Found.
