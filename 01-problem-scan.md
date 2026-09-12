# 📄 01-problem-scan.md — Problem Scan & Quick Cards (Vin Smart Future)

**Họ và tên:** Vũ Văn Điền  
**Mã sinh viên:** 02418  
**Nhóm:** Vin Smart Future — AI Scoping Team  
**Chi nhánh Git:** `VuVanDien-02418`  
**Lĩnh vực trọng tâm:** 🚕 Xanh SM (GSM) — Vận hành đội xe taxi điện thông minh

---

# 🔍 Phase 1 — SCAN: Danh Sách Bài Toán Vận Hành Vingroup

Quét qua các hoạt động vận hành của các công ty thành viên thuộc Tập đoàn Vingroup bằng **4 Lenses**: *Lặp lại (Repetitive), Tốn thời gian (Time-consuming), AI có thể tốt hơn (AI-upgrade), Pain từ người khác (Stakeholder Pain)*.

| # | Subsidiary (Công ty) | Lens | Mô tả ngắn bài toán / Điểm nghẽn vận hành |
|---|----------------------|------|-------------------------------------------|
| 1 | **Xanh SM (GSM)** | **Tốn thời gian** | Điều phối viên xử lý thủ công cuộc gọi sự cố sạc pin/hết pin của tài xế giữa đường (15 phút/lượt: tra GPS, kiểm tra trụ sạc trống, soạn tin hướng dẫn). |
| 2 | **Xanh SM (GSM)** | **AI-upgrade** | Hệ thống gợi ý điểm đón khách hiện tại không tính đến mật độ khách theo thời gian thực, giờ cao điểm, và lịch sử hủy chuyến — tài xế tốn nhiên liệu/pin vô ích. |
| 3 | **Xanh SM (GSM)** | **Pain từ người khác** | Tài xế phàn nàn: sau mỗi sự cố hủy chuyến từ phía khách, tài xế phải tự báo cáo thủ công qua form giấy — mất 8-10 phút/lần, ảnh hưởng thời gian nhận chuyến tiếp theo. |
| 4 | **Vinhomes** | **Lặp lại** | Phân loại và điều hướng tự động các khiếu nại/phản ánh của cư dân (mất nước, hỏng hóc, ồn ào) từ App Vinhomes Resident đến đúng Ban Quản Lý từng tòa nhà. |
| 5 | **Vinmec** | **Tốn thời gian** | Bác sĩ mất 20-30 phút/bệnh nhân để trích xuất dữ liệu lâm sàng và viết tóm tắt hồ sơ xuất viện bằng ngôn ngữ dễ hiểu cho bệnh nhân. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

> **Lưu ý:** Cả 3 thẻ bài toán đều tập trung vào lĩnh vực **Xanh SM (GSM)** — mảng vận hành xe taxi điện thông minh. Đây là lĩnh vực tôi lựa chọn nghiên cứu sâu vì có cơ hội ứng dụng AI rõ ràng và tác động trực tiếp đến hiệu suất vận hành thực tế.

---

## 📌 QUICK PROBLEM CARD #1 — Xanh SM: Xử Lý Sự Cố Pin & Điều Phối Trạm Sạc Khẩn Cấp

> ⭐ **BÀI TOÁN ĐƯỢC CHỌN ĐỂ DEEP-DIVE** — Bài toán có tác động vận hành cao nhất, đo lường rõ ràng và ranh giới an toàn AI có thể thiết lập cụ thể.

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1 ⭐ [SELECTED FOR DEEP-DIVE]           │
│                                                             │
│ Bài toán (1 câu): Tài xế Xanh SM báo sự cố cạn pin giữa    │
│ đường, Dispatcher phải thủ công tra vị trí, tìm trạm sạc    │
│ VinFast gần nhất còn trụ trống, soạn tin hướng dẫn.         │
│                                                             │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)?                                         │
│   - Tài xế: dừng xe giữa đường, mất chuyến, mất thu nhập   │
│   - Dispatcher: bị overload khi ≥3 sự cố đồng thời         │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                         │
│   1. Tài xế gọi tổng đài điều vận báo nguy cơ hết pin       │
│   ──> 2. Dispatcher tra cứu vị trí GPS xe trên bản đồ nội bộ│
│   ──> 3. Tra cứu danh sách trạm sạc VinFast còn trụ trống   │
│   ──> 4. Soạn tin nhắn hướng dẫn đường đi gửi qua App tài xế│
│   ──> 5. Liên hệ xe sạc di động cứu hộ nếu pin xe < 5%      │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3 & 4 (⏱ 10 phút)    │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4            │
│   - Auto-pull GPS + query trạm sạc phù hợp loại cổng sạc    │
│   - LLM soạn nháp tin hướng dẫn bằng tiếng Việt thân thiện  │
│   - Dispatcher 1-click phê duyệt/gửi (HITL)                 │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - Giảm thời gian xử lý sự cố từ 15 min ──> dưới 3 min     │
│   - Tỉ lệ gợi ý đúng cổng sạc phù hợp loại xe đạt ≥ 98%    │
│   - Giảm tỉ lệ xe cạn pin giữa đường từ ~5%/ngày xuống <1%  │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Draft + HITL approval) │
└─────────────────────────────────────────────────────────────┘
```

---

## 📌 QUICK PROBLEM CARD #2 — Xanh SM: Tối Ưu Gợi Ý Điểm Đón Khách Thông Minh

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán (1 câu): Hệ thống gợi ý điểm đón khách của Xanh    │
│ SM không tính mật độ khách theo thời gian thực và lịch sử   │
│ hủy chuyến, khiến tài xế di chuyển đến điểm đón kém hiệu quả│
│                                                             │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)?                                         │
│   - Tài xế: mất pin/nhiên liệu đến điểm đón sai             │
│   - Khách hàng: chờ taxi lâu, tỉ lệ hủy chuyến cao          │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                         │
│   1. Tài xế bật App nhận điểm đón từ hệ thống dispatch       │
│   ──> 2. Hệ thống gợi ý vị trí gần nhất theo GPS đơn thuần  │
│   ──> 3. Tài xế di chuyển đến điểm đón theo hướng dẫn        │
│   ──> 4. Khách thường hủy / tài xế đến điểm không có khách  │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 4 (⏱ 5-7 phút/lượt)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2               │
│   - LLM + ML model dự đoán mật độ khách theo thời gian thực │
│   - Tính điểm hội tụ dựa trên lịch sử hủy chuyến            │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - Giảm tỉ lệ hủy chuyến từ phía khách từ 18% ──> dưới 8%  │
│   - Tăng tỉ lệ ghép chuyến thành công / xe / giờ lên ≥ 20%  │
│                                                             │
│ Quick Architecture: [x] Agentic Loop (Real-time routing AI) │
└─────────────────────────────────────────────────────────────┘
```

---

## 📌 QUICK PROBLEM CARD #3 — Xanh SM: Tự Động Hoá Báo Cáo Sự Cố Hủy Chuyến Của Tài Xế

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán (1 câu): Tài xế mất 8-10 phút/lần tự báo cáo thủ  │
│ công sự cố hủy chuyến qua form phức tạp, làm giảm thời gian │
│ nhận chuyến tiếp theo trong khung giờ cao điểm.             │
│                                                             │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)?                                         │
│   - Tài xế: mất 8-10 phút làm form, mất chuyến tiếp theo    │
│   - Trung tâm điều vận: dữ liệu báo cáo bị trễ, thiếu chuẩn │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                         │
│   1. Khách hủy chuyến hoặc sự cố xảy ra                     │
│   ──> 2. Tài xế mở App, điền form báo cáo (8-10 trường)     │
│   ──> 3. Submit form, chờ hệ thống xử lý                    │
│   ──> 4. Bộ phận phân tích xem xét và phân loại thủ công     │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 (⏱ 8-10 phút/lần)  │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 & 4            │
│   - Tài xế nói miệng 1 câu mô tả sự cố -> LLM điền form     │
│   - LLM phân loại lý do hủy chuyến tự động                   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - Giảm thời gian báo cáo từ 8 phút ──> dưới 30 giây       │
│   - Tỉ lệ phân loại đúng lý do hủy chuyến bởi AI đạt ≥ 90%  │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Voice/Text -> Form fill)│
└─────────────────────────────────────────────────────────────┘
```
