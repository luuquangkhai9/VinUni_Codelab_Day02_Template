# Vin Smart Future — Problem Scan & Quick Problem Cards

**Vai trò:** AI Engineer tại Vin Smart Future  
**Chủ đề nhóm thống nhất:** Tìm kiếm đồ thất lạc tại Vinpearl / VinWonders

> Các con số là **ước tính scoping** cho một cơ sở quy mô trung bình, dùng để đặt baseline ban đầu; cần đối chiếu bằng log vận hành thực tế trước khi triển khai.

## Phase 1 — SCAN

| # | Đơn vị | Lens | Pain point vận hành | Ước tính tổn thất |
|---:|---|---|---|---|
| 1 | VinWonders | Lặp lại | Nhân viên tại cổng, trò chơi và nhà hàng ghi nhận đồ khách bỏ quên vào sổ/form với mô tả không thống nhất. | 30 vụ/ngày × 8 phút nhập liệu = **4 giờ công/ngày**; khoảng 20% hồ sơ thiếu vị trí hoặc thời gian. |
| 2 | Vinpearl | Tốn thời gian | Lễ tân tiếp nhận cuộc gọi/email báo mất đồ, phải hỏi lại loại đồ, màu sắc, phòng và thời điểm. | 25 yêu cầu/ngày × 12 phút = **5 giờ công/ngày**; thời gian phản hồi đầu tiên thường vượt 30 phút. |
| 3 | Vinpearl / VinWonders | AI-upgrade | Nhân viên tìm kiếm thủ công trong Excel, sổ bàn giao và thư mục ảnh bằng từ khóa khác nhau. | 25 vụ/ngày × 15 phút tìm kiếm = **6,25 giờ công/ngày**; tỷ lệ khớp ban đầu ước tính chỉ 50–60%. |
| 4 | Vinpearl / VinWonders | Stakeholder Pain | Việc chuyển giao đồ giữa bộ phận nhặt được, an ninh, lễ tân và kho Lost & Found qua bộ đàm/nhóm chat dễ bị trễ hoặc thất lạc trạng thái. | 10% biên bản cập nhật trễ trên 2 giờ; khoảng **3–5 vụ/tháng** phải tìm lại lịch sử bàn giao. |
| 5 | Vinpearl / VinWonders | Tốn thời gian + AI-upgrade | Nhân viên soạn thủ công tin xác minh và hẹn trả đồ bằng tiếng Việt/Anh, đồng thời kiểm tra giấy tờ nhận dạng. | 20 lượt xác minh/ngày × 7 phút = **2,3 giờ công/ngày**; 5–8% khách phải liên hệ lại vì thiếu hướng dẫn hoặc sai ngôn ngữ. |

### Lựa chọn để làm Quick Cards

Ba cơ hội ưu tiên là **#2 tiếp nhận báo mất đồ**, **#3 tìm kiếm/khớp hồ sơ** và **#5 xác minh–hẹn trả**. Chúng nằm trong cùng một workflow, có dữ liệu văn bản/hình ảnh để thử nghiệm và có thể giữ người thật ở bước quyết định. Nhóm chọn **Card #3 — tìm kiếm và khớp hồ sơ Lost & Found** để làm deep-dive vì đây là bottleneck lớn nhất và metric dễ đo bằng bộ hồ sơ đã gán nhãn.

## Phase 2 — QUICK-ASSESS

### Card #1 — Chuẩn hóa tiếp nhận báo mất đồ

| Trường | Nội dung |
|---|---|
| **Bài toán** | Biến cuộc gọi/form tự do thành hồ sơ Lost & Found có đủ loại đồ, màu, thời gian, vị trí và thông tin liên hệ. |
| **Công ty** | Vinpearl / VinWonders |
| **Actor** | Lễ tân, nhân viên CSKH, khách du lịch. |
| **Workflow hiện tại** | 1) Nhận cuộc gọi/form → 2) Hỏi lại mô tả → 3) Ghi sổ/Excel → 4) Chuyển bộ phận liên quan → 5) Gọi lại xác nhận. |
| **Bottleneck** | Hỏi lại và nhập liệu không nhất quán, khoảng **12 phút/vụ**. |
| **AI hỗ trợ** | Trích xuất trường dữ liệu từ tiếng Việt/Anh, phát hiện trường còn thiếu và tạo hồ sơ nháp để nhân viên duyệt. |
| **Metric** | 85% hồ sơ hoàn chỉnh trong **dưới 3 phút**; giảm yêu cầu hỏi lại **30%**; không tự xác nhận quyền sở hữu. |
| **Architecture** | Rule kiểm tra trường bắt buộc + **LLM Feature**, Human-in-the-loop. |

### Card #2 — Soạn tin xác minh và hẹn trả đồ

| Trường | Nội dung |
|---|---|
| **Bài toán** | Tạo bản nháp tin nhắn đa ngôn ngữ để xác minh đúng người nhận và hướng dẫn thủ tục nhận đồ. |
| **Công ty** | Vinpearl / VinWonders |
| **Actor** | Nhân viên Lost & Found, lễ tân, khách quốc tế. |
| **Workflow hiện tại** | 1) Nhân viên thấy bản ghi có khả năng khớp → 2) Đọc lại biên bản → 3) Soạn tin Việt/Anh → 4) Hỏi giấy tờ và thời gian nhận → 5) Gửi tin thủ công. |
| **Bottleneck** | Soạn nội dung và kiểm tra thông tin, khoảng **7 phút/lượt**; dễ dùng sai ngôn ngữ hoặc thiếu giấy tờ cần thiết. |
| **AI hỗ trợ** | Tạo draft theo mẫu chính sách, dịch, liệt kê giấy tờ và câu hỏi xác minh; nhân viên duyệt trước khi gửi. |
| **Metric** | Giảm thời gian soạn từ 7 xuống **dưới 2 phút**; 100% tin có checklist giấy tờ; **0** tin tự động gửi. |
| **Architecture** | **LLM Feature + template/rules**, bắt buộc HITL. |

### Card #3 — Tìm kiếm và khớp hồ sơ Lost & Found *(được chọn để deep-dive)*

| Trường | Nội dung |
|---|---|
| **Bài toán** | Gợi ý các món đồ trong kho có đặc điểm tương đồng với báo mất để nhân viên tìm nhanh hơn. |
| **Công ty** | Vinpearl / VinWonders |
| **Actor** | Nhân viên Lost & Found, an ninh, lễ tân; khách đang chờ kết quả. |
| **Workflow hiện tại** | 1) Nhận báo mất → 2) Hỏi loại/màu/vị trí/thời gian → 3) Ghi Excel và lưu ảnh → 4) Tìm thủ công theo từ khóa → 5) Đối chiếu biên bản, gọi khách xác minh. |
| **Bottleneck** | Bước 3–4: dữ liệu rời rạc, từ khóa không nhất quán, khoảng **15 phút/vụ**. |
| **AI hỗ trợ** | Chuẩn hóa thuộc tính, tìm kiếm ngữ nghĩa/ảnh tương đồng và xếp hạng 3–5 ứng viên; nhân viên đối chiếu biên bản và quyết định. |
| **Metric** | Giảm thời gian tìm **15 → dưới 5 phút/vụ**; Recall@5 ≥ **80%** trên bộ case gán nhãn; 100% kết quả có link bằng chứng; không tự kết luận chủ sở hữu. |
| **Architecture** | **LLM/embedding search + metadata rules + Human review**. |

### Ranh giới vận hành chung

- AI chỉ chuẩn hóa, tìm kiếm, xếp hạng và soạn **bản nháp**; nhân viên chịu trách nhiệm xác minh và bàn giao.
- Không tự khẳng định món đồ thuộc về khách, không tự gửi thông báo cuối cùng, không tự thay đổi biên bản hoặc trạng thái kho.
- Ảnh, số điện thoại và giấy tờ phải được phân quyền/ẩn danh; trường hợp tranh chấp, tài sản giá trị cao hoặc nghi ngờ gian lận phải chuyển quản lý/an ninh.
- Khi thiếu dữ liệu hoặc độ tin cậy thấp, hệ thống trả về “Cần nhân viên kiểm tra thủ công” và dùng quy trình hiện tại làm fallback.
