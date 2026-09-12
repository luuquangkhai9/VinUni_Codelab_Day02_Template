# Vin Smart Future — Phase 1 & Phase 2

> **Vai trò:** AI Engineer tại Vin Smart Future  
> **Phạm vi khảo sát:** Vinpearl / VinWonders  
> **Lưu ý:** Các con số dưới đây là **ước tính để scoping**, dựa trên giả định một khu nghỉ dưỡng/công viên quy mô trung bình. Cần xác thực bằng log vận hành trước khi triển khai.

## Phase 1 — SCAN: Bảng quét cơ hội

| # | Subsidiary | Lens | Quy trình/pain point | Ước tính tổn thất hiện tại |
|---:|---|---|---|---|
| 1 | Vinpearl / VinWonders | Lặp lại + Tốn thời gian | Nhân viên lễ tân và CSKH đọc, phân loại rồi chuyển tiếp phản hồi từ app, email, hotline và mạng xã hội. | Khoảng 120 phản hồi/ngày × 6 phút = **12 giờ công/ngày**; 10–15% ticket bị chuyển sai, làm SLA trễ thêm 4–8 giờ. |
| 2 | Vinpearl | Tốn thời gian + AI-upgrade | Điều phối buồng phòng tổng hợp trạng thái phòng từ PMS, điện thoại và nhóm chat để ưu tiên phòng cần dọn trước giờ check-in. | 60 phòng đến sớm/ngày × 4 phút tra cứu = **4 giờ công/ngày**; 8% phòng sẵn sàng trễ, gây khoảng 20 lượt khách chờ trên 30 phút/ngày. |
| 3 | VinWonders | Stakeholder Pain + Lặp lại | Nhân viên vận hành thủ công đếm khách tại cổng, theo dõi hàng đợi và gọi bộ đàm để điều tiết khu trò chơi/nhà hàng. | Mỗi ca mất khoảng **90 phút** cho việc tổng hợp; thời gian chờ cao điểm tăng 10–15 phút/khách, ước tính 3–5% khách bỏ qua một dịch vụ. |
| 4 | Vinpearl / VinWonders | Tốn thời gian + Stakeholder Pain | Tiếp nhận báo mất đồ từ nhiều điểm, hỏi lại mô tả và dò sổ bàn giao thủ công trước khi phản hồi khách. | 25 vụ/ngày × 15 phút = **6,25 giờ công/ngày**; khoảng 20% hồ sơ thiếu thông tin, làm thời gian tìm kiếm kéo dài thêm 1 ngày. |
| 5 | VinWonders | AI-upgrade + Stakeholder Pain | Dự báo lượng khách theo khung giờ để bố trí nhân sự, xe điện và quầy dịch vụ vẫn dựa chủ yếu vào kinh nghiệm/quy tắc tĩnh. | Sai lệch dự báo khoảng 20%; có thể thừa 10–15 nhân sự ở giờ thấp điểm và thiếu 8–12 người ở giờ cao điểm, tương đương **10–15% chi phí ca** bị phân bổ kém. |

### Ưu tiên sơ bộ

Chọn ba bài toán **#1 (phản hồi đa kênh), #2 (điều phối buồng phòng), #4 (mất đồ)** để làm Quick Problem Cards vì có quy trình lặp lại, dữ liệu đầu vào tương đối rõ và metric đo được. Bài toán #3 cần dữ liệu camera/đếm người theo thời gian thực; #5 cần baseline dự báo và dữ liệu lịch sử đủ dài nên phù hợp giai đoạn sau.

## Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### Card #1 — Phân loại và soạn nháp phản hồi khách hàng đa kênh

| Trường | Nội dung |
|---|---|
| **Bài toán** | Tự động gom, phân loại mức độ ưu tiên và soạn nháp trả lời cho phản hồi của khách tại Vinpearl/VinWonders. |
| **Công ty** | Vinpearl / VinWonders |
| **Actor đang đau** | Nhân viên CSKH, trưởng ca; khách chờ phản hồi. |
| **Workflow hiện tại** | 1) Mở từng kênh → 2) Đọc và chép nội dung vào bảng theo dõi → 3) Gán nhóm (phòng, vé, ăn uống, sự cố) → 4) Chuyển bộ phận → 5) Tự soạn và gửi phản hồi. |
| **Bottleneck** | Đọc–gán nhãn và viết phản hồi (khoảng **6 phút/ticket**), đặc biệt với nội dung tiếng Việt/Anh lẫn nhau. |
| **AI hỗ trợ tại** | Bước 2–4: chuẩn hóa nội dung, phân loại, phát hiện khẩn cấp và tạo **bản nháp** theo chính sách. |
| **Metric thành công** | 90% ticket được phân loại trong **dưới 30 giây**; thời gian soạn từ 6 xuống **dưới 2 phút**; độ chính xác route ≥ **95%**. |
| **Quick Architecture** | **LLM Feature + Rule-based router**, bắt buộc nhân viên duyệt trước khi gửi. |

### Card #2 — Ưu tiên điều phối buồng phòng theo giờ nhận phòng

| Trường | Nội dung |
|---|---|
| **Bài toán** | Giúp điều phối viên chọn phòng cần dọn trước dựa trên giờ check-in, loại khách và trạng thái thực tế. |
| **Công ty** | Vinpearl |
| **Actor đang đau** | Housekeeping dispatcher, trưởng bộ phận buồng phòng, khách đến sớm. |
| **Workflow hiện tại** | 1) Nhận danh sách check-in từ PMS → 2) Gọi/nhắn hỏi tình trạng từng phòng → 3) So sánh giờ đến và yêu cầu đặc biệt → 4) Gọi nhân viên dọn phòng → 5) Cập nhật PMS thủ công. |
| **Bottleneck** | Tổng hợp trạng thái từ nhiều nguồn (khoảng **4 phút/phòng đến sớm**), dễ bỏ sót thay đổi phút cuối. |
| **AI hỗ trợ tại** | Bước 2–3: hợp nhất trạng thái, dự đoán phòng có nguy cơ trễ và đề xuất thứ tự ưu tiên. |
| **Metric thành công** | Giảm thời gian lập danh sách ưu tiên từ 30 xuống **dưới 5 phút/ca**; giảm phòng trễ check-in **8% → dưới 3%**; không tự thay đổi trạng thái PMS. |
| **Quick Architecture** | **Rule/State-machine + ML/LLM tóm tắt**, nhân viên xác nhận lệnh điều phối. |

### Card #3 — Tiếp nhận và tìm kiếm đồ thất lạc

| Trường | Nội dung |
|---|---|
| **Bài toán** | Chuẩn hóa báo mất đồ và gợi ý khớp với kho đồ thất lạc để rút ngắn thời gian tìm kiếm. |
| **Công ty** | VinWonders / Vinpearl |
| **Actor đang đau** | Nhân viên Lost & Found, lễ tân, khách du lịch. |
| **Workflow hiện tại** | 1) Nhận cuộc gọi/form → 2) Hỏi lại địa điểm, thời gian, mô tả → 3) Ghi sổ hoặc file Excel → 4) Tìm thủ công ảnh/biên bản bàn giao → 5) Gọi lại xác minh và hẹn nhận. |
| **Bottleneck** | Nhập lại thông tin và tìm kiếm bằng từ khóa không nhất quán (khoảng **15 phút/vụ**). |
| **AI hỗ trợ tại** | Bước 2–4: trích xuất thuộc tính (màu, loại, thời gian, vị trí), tìm các bản ghi tương đồng và tạo nháp tin xác minh. |
| **Metric thành công** | 85% hồ sơ hoàn chỉnh trong **dưới 3 phút**; giảm thời gian tìm từ 15 xuống **dưới 5 phút/vụ**; tăng tỷ lệ khớp đúng ≥ **80%** sau nhân viên xác nhận. |
| **Quick Architecture** | **LLM Feature + vector/search**, không tự kết luận chủ sở hữu hoặc tự bàn giao tài sản. |

### Ranh giới chung cần ghi nhớ

- Số liệu là giả định scoping, không phải cam kết hiệu quả hay số liệu công bố của Vinpearl/VinWonders.
- AI chỉ phân loại, đề xuất và soạn **draft**; nhân viên chịu trách nhiệm duyệt các phản hồi, điều phối và xác minh tài sản.
- Dữ liệu khách phải được ẩn danh/phân quyền; các trường hợp khẩn cấp, khiếu nại pháp lý hoặc nghi ngờ gian lận phải chuyển người xử lý.
