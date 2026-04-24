# Day 17 Submission
**Student:** Phạm Hoàng Kim Liên
**Date:** 24/4/2026
**Product idea:** Trợ lý AI điều hướng và thủ tục bệnh viện

---
## 1. MVP Boundary Sheet
**Riskiest Assumption:**
> Người bệnh (đặc biệt là người lớn tuổi) sẽ đủ tin tưởng và kiên nhẫn để làm theo hướng dẫn thủ tục/điều hướng của một AI Chatbot thay vì giữ thói quen chạy đi tìm nhân viên y tế để hỏi trực tiếp.

**In-Scope** (tối đa 3):
- [x] **Hỏi đáp thủ tục & Quy trình (RAG-based):** test giả định người dùng sẵn sàng tra cứu thông tin giấy tờ trên bot thay vì hỏi lễ tân.
- [x] **Gợi ý khoa khám sơ bộ (NLP text classification):** test giả định AI có thể ánh xạ (map) đúng triệu chứng phổ thông sang khoa lâm sàng tương ứng với độ chính xác chấp nhận được.
- [x] **Điều hướng tĩnh (Static Routing):** test giả định người dùng có thể tự tìm đường dựa trên chỉ dẫn văn bản/sơ đồ ảnh do AI cung cấp mà không đi lạc.

**Out-of-Scope:**
- Báo thời gian chờ theo thời gian thực — lý do bỏ: Đòi hỏi tích hợp sâu API với hệ thống HIS/hệ thống gọi số của bệnh viện, rào cản kỹ thuật quá lớn cho MVP.
- Đặt lịch khám online — lý do bỏ: Đã có app chuyên dụng, không giải quyết "nỗi đau" bối rối ngay tại sảnh chờ.
- Thanh toán viện phí — lý do bỏ: Phức tạp về bảo mật, đối soát tài chính và pháp lý.

**Non-Goals:**
- KHÔNG chẩn đoán bệnh hay kết luận tình trạng sức khỏe.
- KHÔNG kê đơn, gợi ý tên thuốc hay phác đồ điều trị.

---
## 2. PRD Skeleton
### Problem Statement
> Bệnh nhân đến khám tại các bệnh viện lớn thường xuyên bối rối, đi lòng vòng vì không nắm rõ thủ tục và sơ đồ, dẫn đến sự mệt mỏi cho người bệnh và gây quá tải trầm trọng cho bộ phận tiếp đón, bảo vệ.

### Target User
> Bệnh nhân và người nhà (đặc biệt là người từ tuyến tỉnh lên hoặc đến khám lần đầu) đang có mặt tại sảnh chờ của bệnh viện tuyến đầu, cần định hướng ngay lập tức.

### User Stories
**Story 1:**
> As a người lần đầu đi khám nội trú, I want biết chính xác mình cần chuẩn bị giấy tờ gì (CCCD, BHYT, giấy chuyển tuyến) và nộp ở quầy nào, so that tôi không bị yêu cầu quay lại bổ sung hồ sơ sau khi đã xếp hàng 30 phút.

**Story 2:**
> As a bệnh nhân vừa hoàn tất thủ tục và cầm phiếu khám trên tay, I want được cung cấp ngay chỉ dẫn đường đi chi tiết (kèm sơ đồ) tới chính xác phòng khám đó, so that tôi có thể tự đi đến nơi mà không phải hỏi thăm mù mờ khắp nơi, đồng thời tránh cảm giác e ngại khi phải làm phiền các nhân viên y tế đang bận rộn.

### AI-Specific
**Model Selection:**
- Model: **Gemini 1.5 Flash** (hoặc tương đương).
- Lý do chọn: Cần tốc độ phản hồi (latency) cực nhanh tại điểm chạm kiosk/mobile, chi phí token rẻ, và khả năng hiểu ngữ cảnh/từ vựng tiếng Việt rất tốt để làm RAG. Không cần khả năng reasoning quá nặng đô.
- Trade-offs chấp nhận: AI có thể thỉnh thoảng không hiểu các từ lóng địa phương mô tả bệnh tật (có thể yêu cầu người dùng nhập lại).
- Trade-offs không chấp nhận: Hallucination (tuyệt đối không bịa ra số phòng, tên khoa hoặc quy trình không tồn tại trong bệnh viện).

**Data Requirements:**
- Nguồn: Sổ tay quy trình khám chữa bệnh của BV, sơ đồ tầng (text mô tả), danh sách mapping "Triệu chứng cơ bản $\rightarrow$ Khoa lâm sàng".
- Owner: Phòng Kế hoạch Tổng hợp và Phòng IT của bệnh viện.
- Update frequency: Cập nhật thủ công mỗi khi bệnh viện thay đổi quy trình hoặc dời đổi vị trí phòng ban.

**Fallback UX:**
- Chiến lược: Graceful Handover & Expectation Management.
- Trigger: Khi Confidence score của RAG retrieval thấp, từ khóa phát hiện rủi ro (cấp cứu, chảy máu, tự tử), hoặc AI nhận diện câu hỏi mang tính chất chẩn đoán y khoa chuyên sâu.
- Hành động: Dừng sinh câu trả lời tự động, in ra cảnh báo chuẩn: *"Thông tin này vượt quá khả năng hỗ trợ quy trình của tôi. Đây có thể là vấn đề y khoa cần bác sĩ hoặc tình huống khẩn cấp. Vui lòng đến ngay Quầy Cấp cứu hoặc Quầy Thông tin số 1."*
- User options: Nút bấm hiển thị bản đồ khẩn cấp dẫn đến Quầy Thông tin/Cấp cứu.

### Success Metrics
- Primary metric: **Resolution Rate (Tỷ lệ giải quyết thành công)** - Tỷ lệ phiên chat kết thúc mà người dùng không cần đến quầy hỏi lại nhân viên (đo bằng survey 1 chạm: "Bạn đã tìm được thông tin mình cần chưa?").
- Ngưỡng thành công: > 60% session được rate "Thành công".
- Timeframe đo lường: 4 tuần triển khai Pilot tại 1 sảnh viện.

### Dependencies & Constraints
- Bệnh viện phải có bộ tài liệu quy trình được số hóa và chuẩn hóa (nhiều BV vẫn làm việc theo thói quen truyền miệng).
- Sảnh bệnh viện phải có kết nối Wifi/4G ổn định để thiết bị của user có thể load model.

---
## 3. Hypothesis Table
### Hypothesis 1 (cho tính năng In-Scope #2: Gợi ý khoa khám)
> "Chúng tôi tin rằng **tính năng AI map triệu chứng sang khoa khám** sẽ giúp **bệnh nhân mua đúng loại phiếu khám ngay từ lần đầu**.
> Chúng tôi sẽ biết mình đúng khi thấy **tỷ lệ bệnh nhân bị điều chuyển lại khoa (đăng ký nhầm) tại quầy tiếp đón** giảm **15%** trong **1 tháng test pilot**."
* Riskiest assumption: AI phân loại sai dẫn đến hậu quả bệnh nhân mất tiền mua sai phiếu.
* Cách test cheapest: Chưa cho end-user dùng ngay. Cho AI chạy "ngầm" song song với luồng phân bệnh của nhân viên tiếp đón trong 1 tuần (Shadow Testing). So sánh kết quả của AI với quyết định của nhân viên để đo độ chính xác (Precision/Recall).

### Hypothesis 2 (cho tính năng In-Scope #1: Hỏi đáp thủ tục)
> "Chúng tôi tin rằng **chatbot RAG giải đáp thủ tục** sẽ giúp **nhân viên y tế giảm tải**.
> Chúng tôi sẽ biết mình đúng khi thấy **thời gian trung bình để xử lý 1 bệnh nhân tại quầy tiếp đón** giảm đi **30 giây** trong **tuần đầu tiên triển khai**."

---
## 4. PMF Scorecard
**Aha Moment:**
> Khi bệnh nhân đang hoang mang giữa sảnh đông người, họ nhập một câu hỏi lủng củng ("đau ruột thừa thì đi đâu"), và ngay lập tức (dưới 3 giây) nhận được hướng dẫn rõ ràng: *"Vui lòng chuẩn bị thẻ BHYT, đến Quầy số 3 lấy số, đăng ký khám khoa Ngoại Tiêu Hóa ở Tầng 2, Dãy B"*.

**Actionable Metric:**
> **Tỷ lệ hoàn thành luồng "Gợi ý $\rightarrow$ Tìm đường" (Funnel Completion Rate):** % user hỏi triệu chứng/thủ tục và sau đó click tiếp vào nút "Xem vị trí quầy trên bản đồ/sơ đồ".

**PMF Method:**
> Aha Moment tracking (Đo lường tần suất người dùng đạt được giá trị ngay trong session đầu tiên).
> Ngưỡng thành công: > 40% user bắt đầu chat đi được đến bước nhận thông tin điều hướng cuối cùng.

**Vanity Metrics tôi sẽ không dùng:**
- Tổng số tin nhắn trao đổi trên mỗi session (Nhiều tin nhắn không có nghĩa là tốt, có thể do AI ngốc khiến user phải hỏi đi hỏi lại mãi mới ra vấn đề).

---
## 5. AI Critique Log
**Điểm AI chỉ ra:**
1. Rủi ro Hallucination trong y tế là chí mạng dù chỉ làm thủ tục — Action: **Accept** — Lý do: Cần thiết lập prompt cực kỳ chặt (system prompt cấm tuyệt đối việc tự suy diễn) và có cơ chế Fallback rõ ràng như đã định nghĩa ở phần PRD.
2. Việc làm RAG với tài liệu nội bộ BV sẽ gặp khó vì dữ liệu thường không có cấu trúc chuẩn — Action: **Accept** — Lý do: Đây là thách thức lớn về Data Engineering. Sẽ cần một bước tiền xử lý (parsing) tài liệu thủ công cẩn thận trước khi đưa vào Vector Database.
3. Chỉ text điều hướng là chưa đủ, người dùng sẽ không hình dung được — Action: **Partial** — Lý do: Tạm thời chưa làm bản đồ 3D vì quá tốn kém (Out-of-scope), nhưng sẽ khắc phục bằng cách đính kèm ảnh (Image) sơ đồ tĩnh 2D của bệnh viện vào câu trả lời của AI.

**Thay đổi lớn nhất giữa Version A và Version B:**
> Thu hẹp hoàn toàn từ một "Trợ lý ảo toàn năng cho bệnh viện" (có cả đặt lịch, xem kết quả) xuống thành **"Chuyên gia phân luồng và thủ tục tại sảnh"**. Việc này giúp giảm rủi ro kỹ thuật (không đụng vào HIS của viện) và tập trung giải quyết đúng 1 pain-point khẩn cấp nhất.

---
## 6. Self-assessment
Mắt xích nào trong [MVP Boundary - PRD - Hypothesis - PMF] bạn đang yếu nhất?
> **Phần Hypothesis & PMF Metric.** Việc đo lường chính xác các chỉ số như "giảm thời gian chờ" hay "tỷ lệ đăng ký sai khoa" phụ thuộc vào việc lấy được số liệu đối chiếu từ bệnh viện (baseline data). Nếu bệnh viện không có hệ thống đo lường từ trước, việc chứng minh giá trị của chatbot sẽ rất khó khăn và mang tính cảm tính.

Open questions bạn muốn giải đáp tiếp:
1. Giao diện (Access path) nào là tối ưu nhất tại sảnh? Kiosk màn hình lớn (chi phí cao, dễ ùn tắc), hay dán QR code khắp nơi (phụ thuộc vào kỹ năng dùng smartphone của bệnh nhân lớn tuổi)?
2. Làm thế nào để thu thập bộ ground-truth data chuẩn xác nhất để test tính năng phân loại khoa khám trước khi ra mắt thật? Có nên cào dữ liệu từ các forum y tế?