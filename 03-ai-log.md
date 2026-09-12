# AI Log & Reflection — Lost & Found tại Vinpearl / VinWonders

**Vai trò:** AI Engineer tại Vin Smart Future  
**Bài toán:** Tìm kiếm và khớp hồ sơ đồ thất lạc  
**Mục đích nhật ký:** Ghi lại trung thực cách tôi dùng AI như một thought-partner trong quá trình scoping, thay vì xem output của AI là sự thật vận hành.

> Những con số về số vụ/ngày, phút xử lý, tỷ lệ khớp và chi phí trong các file bài làm là **giả định để scoping**. Tôi không trình bày chúng như số liệu chính thức của Vinpearl/VinWonders.

## 1. AI đã hỗ trợ tôi những gì?

### 1.1. Mở rộng danh sách pain point

Tôi bắt đầu với yêu cầu: “Tôi là AI Engineer tại Vin Smart Future; hãy gợi ý các pain point vận hành cho Vinpearl/VinWonders”. AI giúp tôi mở rộng từ một ý tưởng chung về đồ thất lạc thành các điểm nghẽn cụ thể hơn:

- Tiếp nhận báo mất đồ từ cuộc gọi, email và form.
- Chuẩn hóa mô tả đồ vật bằng tiếng Việt/Anh.
- Tìm kiếm trong Excel, sổ bàn giao và thư mục ảnh.
- Chuyển giao trạng thái giữa lễ tân, an ninh, Lost & Found và kho.
- Soạn tin xác minh và hẹn trả đồ.

Sau đó tôi dùng bốn lenses trong worksheet để loại bỏ các ý tưởng quá rộng. Tôi giữ lại các vấn đề có actor rõ, có workflow thủ công và có thể đo bằng thời gian hoặc độ chính xác.

### 1.2. Cấu trúc hóa Quick Problem Cards

AI giúp chuyển mô tả tự do thành các trường bắt buộc: Actor, workflow, bottleneck, vị trí AI, metric và quick architecture. Việc này giúp tôi nhận ra rằng “dùng AI để tìm đồ” chưa đủ cụ thể; cần nói rõ AI chỉ xếp hạng ứng viên, còn nhân viên xác minh và bàn giao.

### 1.3. Phản biện lựa chọn kiến trúc

Tôi yêu cầu AI đóng vai CFO và trưởng vận hành để phản biện. Kết quả hữu ích nhất là nhận xét rằng lọc theo thời gian, địa điểm và trạng thái kho nên dùng rule/database trước; LLM phù hợp hơn với mô tả tự do, từ đồng nghĩa và đa ngôn ngữ. Vì vậy kiến trúc cuối cùng là hybrid retrieval assistant, không phải agent tự trị.

### 1.4. Hỗ trợ kiểm tra tính đầy đủ của deliverables

Tôi dùng AI để đối chiếu nội dung với worksheet và autograder: 5 bài toán, 3 cards, Problem Statement 6-field, future flow, HITL, fallback và metric. AI cũng giúp tôi rà tên file và nhận ra file workflow phải có một trong các đuôi `.png`, `.jpg`, `.jpeg` hoặc `.pdf`.

### 1.5. Hỗ trợ tạo workflow diagram

Tôi cung cấp cho công cụ tạo ảnh các bước và thời gian chính xác từ báo cáo. Sau khi ảnh được tạo, tôi kiểm tra trực quan lại tiêu đề, sáu bước, nhánh quyết định, tổng thời gian và bottleneck; chỉ sau đó mới lưu thành `04-workflow-diagram.png`.

## 2. AI đã sai hoặc có nguy cơ sai ở đâu?

### 2.1. Số liệu có vẻ cụ thể nhưng chưa có nguồn

AI dễ sinh ra các số như “25 vụ/ngày”, “15 phút/vụ”, “Recall@5 80%” hoặc “60.000 VND/giờ”. Những số này hữu ích để minh họa cách đặt baseline nhưng không chứng minh được quy mô thật của Vinpearl/VinWonders. Tôi đã gắn nhãn **ước tính scoping**, dùng phép tính nhất quán và thêm yêu cầu phải xác thực bằng log trước pilot.

### 2.2. Suy diễn sai bối cảnh doanh nghiệp

Trong các bản scan của thành viên, có nội dung về Xanh SM, VinFast và thậm chí VinUni, không phù hợp với chủ đề nhóm. Tôi không gộp máy móc các nội dung đó. Tôi chỉ dùng các bản có liên quan để kiểm tra format, sau đó thống nhất lại toàn bộ bài theo Lost & Found của Vinpearl/VinWonders.

### 2.3. Lẫn lộn “tìm thấy” với “xác định chủ sở hữu”

Một mô hình ngôn ngữ có thể viết câu khẳng định kiểu “đây chắc chắn là đồ của khách”. Đây là ranh giới nguy hiểm vì matching hình ảnh/mô tả chỉ tạo ứng viên. Tôi sửa yêu cầu thành “xếp hạng ứng viên kèm bằng chứng”, cấm kết luận chủ sở hữu và bắt buộc nhân viên hỏi thông tin không công khai.

### 2.4. Nguy cơ đề xuất tự động hóa quá mức

Nếu prompt chỉ yêu cầu “tự động xử lý Lost & Found”, AI có thể đề xuất tự gửi tin, tự đổi trạng thái kho hoặc tự bàn giao. Tôi thu hẹp scope: AI chỉ chuẩn hóa, tìm kiếm, xếp hạng và tạo draft; mọi hành động ghi/gửi/bàn giao đều do con người xác nhận.

### 2.5. Hạn chế của ảnh sinh tự động

Ảnh sinh bằng AI có thể viết sai chữ, thiếu dấu tiếng Việt hoặc làm thay đổi thứ tự mũi tên. Tôi đã yêu cầu văn bản nguyên văn, bố cục rõ và kiểm tra ảnh sau khi tạo. Nếu chữ trong ảnh không đọc được, phương án dự phòng là dùng sơ đồ Mermaid hoặc vẽ lại bằng công cụ sơ đồ xác định.

## 3. Tôi đã sửa prompt và ranh giới như thế nào?

### Prompt ban đầu

> “Gợi ý các pain point vận hành có thể tối ưu bằng AI cho Vinpearl/VinWonders.”

Prompt này cho nhiều ý tưởng nhưng thiếu quy mô, actor, rủi ro và tiêu chí loại bỏ.

### Prompt đã cải thiện

> “Đóng vai CFO và Trưởng phòng Vận hành Lost & Found. Với từng pain point, hãy nêu actor, workflow thủ công 3–5 bước, bước bottleneck, giả định số liệu (ghi rõ là ước tính), metric có baseline/target, dữ liệu cần có, phương án Rule vs LLM vs Agent và rủi ro nếu AI sai. Không khẳng định số liệu là dữ kiện của Vinpearl/VinWonders.”

Prompt cải thiện tạo ra đầu ra dễ kiểm chứng hơn và buộc AI phân biệt giả định với dữ kiện.

### Ranh giới tôi giữ trong thiết kế sản phẩm

- AI không phải system of record; biên bản gốc và trạng thái kho là nguồn sự thật.
- Mỗi ứng viên phải có `record_id`, nguồn, trạng thái và lý do khớp.
- Không tự xác định chủ sở hữu, không tự gửi thông báo, không sửa/xóa hồ sơ, không tự bàn giao.
- Không hiển thị đặc điểm bí mật của món đồ cho người báo mất; dùng chúng làm câu hỏi xác minh.
- Tài sản giá trị cao, giấy tờ tùy thân, tranh chấp hoặc điểm tin cậy thấp phải chuyển nhân viên/quản lý.
- Khi model/search lỗi hoặc thiếu dữ liệu, quay về quy trình tìm kiếm thủ công.

## 4. Kiểm chứng và trách nhiệm con người

Tôi không dùng câu trả lời AI làm bằng chứng về vận hành thực tế. Các bước kiểm chứng của tôi là:

1. Đối chiếu yêu cầu trong worksheet và tiêu chí autograder.
2. Kiểm tra phép tính số học của các ước tính thời gian/chi phí.
3. Gắn nhãn rõ giả định và liệt kê dữ liệu cần thu thập.
4. So sánh Rule, LLM và Agent thay vì mặc định chọn LLM.
5. Đặt Human-in-the-loop ở các điểm có hậu quả thật: xác minh, liên hệ và bàn giao.
6. Đề xuất pilot shadow mode trên dữ liệu ẩn danh trước khi tác động quy trình thật.

AI giúp tôi đi nhanh hơn trong việc brainstorm và cấu trúc hóa, nhưng quyết định cuối cùng về scope, metric, quyền riêng tư và ranh giới vận hành vẫn là trách nhiệm của nhóm và stakeholder.

## 5. Bài học cá nhân

Điều quan trọng nhất tôi học được là một ý tưởng AI tốt không bắt đầu từ model. Nó bắt đầu từ workflow đủ cụ thể, bottleneck có thể đo và ranh giới khi model sai. Với Lost & Found, giá trị thực tế không phải để AI “quyết định đồ thuộc về ai”, mà là giảm thời gian nhân viên dò dữ liệu trong khi vẫn giữ quyền xác minh và bàn giao cho con người.

Trong bước tiếp theo, tôi sẽ ưu tiên lấy dữ liệu đã ẩn danh, xây bộ case có nhãn đúng/sai và đo baseline tìm kiếm từ khóa trước. Chỉ khi có bằng chứng rằng semantic search cải thiện Recall@5 và không phá vỡ guardrail, nhóm mới cân nhắc pilot có nhân viên.
