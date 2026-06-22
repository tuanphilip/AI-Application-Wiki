# Day 05 - AI Product Kickoff Sprint

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 05: AI Product Kickoff Sprint**. Trọng tâm là phân tích sự khác biệt bản chất giữa phần mềm truyền thống và sản phẩm AI, cách thiết kế hệ thống chấp nhận sự bất định (uncertainty), quản trị phân phối lỗi và quy hoạch dự án MVP cho ngày hackathon (Day 06).

---

## 1. AI Product ≠ Software Product: Quản Trị Phân Phối Lỗi

Sự khác biệt cốt lõi về bản chất vận hành giữa phần mềm truyền thống và sản phẩm tích hợp AI:

*   **Phần mềm truyền thống (Deterministic):**
    *   *Cơ chế:* Input ổn định $\rightarrow$ Logic xác định $\rightarrow$ Output đúng/sai rõ ràng.
    *   *Khắc phục:* Khi tìm ra lỗi (bug), lập trình viên sửa code và lỗi đó sẽ biến mất hoàn toàn.
*   **Sản phẩm AI (Probabilistic):**
    *   *Cơ chế:* Đầu vào, đầu ra và quá trình xử lý đều có phương sai (variance). Cùng một yêu cầu có thể cho kết quả khác nhau tùy theo mô hình, ngữ cảnh và dữ liệu tại thời điểm chạy.
    *   *Khắc phục:* Không có khái niệm "sửa triệt để bug". **AI Product là cuộc chơi về quản trị phân phối lỗi.**
    *   *Triết lý thiết kế:* AI product không hứa *"không bao giờ sai"*; nó hứa *"khi không chắc hoặc sai, hệ thống vẫn có cơ chế dẫn dắt người dùng đi đúng hướng"*.

---

## 2. Ba Lớp Bất Định (Uncertainty Layers) Trong Hệ Thống AI

Hệ thống AI đối mặt với sự bất định ở mọi cấp độ của luồng dữ liệu:

```
                  ┌─────────────────────────────────┐
                  │       Input Uncertainty         │
                  │ - Người dùng hỏi mơ hồ, dùng    │
                  │   slang, đổi ý giữa chừng.      │
                  │ - Nguy cơ Prompt Injection.     │
                  └────────────────┬────────────────┘
                                   │
                                   v
                  ┌─────────────────────────────────┐
                  │       Process Uncertainty       │
                  │ - Tool chain nhiều bước phức tạp│
                  │ - Khó nhìn thấy quá trình suy   │
                  │   luận bên trong của mô hình.   │
                  └────────────────┬────────────────┘
                                   │
                                   v
                  ┌─────────────────────────────────┐
                  │       Output Uncertainty        │
                  │ - Câu trả lời không cố định.    │
                  │ - RAG/Tool trả về dữ liệu lệch. │
                  │ - Khó kiểm chứng tính đúng đắn. │
                  └─────────────────────────────────┘
```

---

## 3. Hiện Tượng Trôi Lệch Hành Vi (Production Drift)

Ngay cả khi đội ngũ phát triển không thay đổi một dòng code nào, hành vi của sản phẩm AI vẫn có thể bị thay đổi (drift) theo thời gian do các yếu tố:

1.  **Model Update Drift (Lệch do cập nhật mô hình):** Nhà cung cấp cập nhật phiên bản mô hình mới. Bản mới có thể thông minh hơn về trung bình nhưng lại làm giảm hiệu suất ở một tác vụ hẹp cụ thể mà bạn đang sử dụng.
2.  **Context Drift (Lệch do dữ liệu nền):** Các chính sách, biểu phí, tài liệu hướng dẫn, hoặc thông tin API bên ngoài thay đổi khiến dữ liệu RAG nạp vào bị lệch.
3.  **User Drift (Lệch do hành vi người dùng):** Người dùng thật đặt câu hỏi lệch khỏi phạm vi thử nghiệm ban đầu (out of scope), sử dụng từ lóng hoặc cách diễn đạt khác lạ.
4.  **Prompt Drift (Lệch do chắp vá):** Trong quá trình sửa lỗi, đội ngũ liên tục chèn thêm các quy tắc nhỏ vào Prompt (patching) khiến Prompt gốc trở nên cồng kềnh và hành vi của mô hình trở nên cực kỳ khó đoán.

---

## 4. Các Sự Cố Thực Tế (Failure Case Studies)

Khi mô hình AI đúng "gần đủ" nhưng sản phẩm vẫn thất bại do lỗi thiết kế hệ thống kiểm soát:
*   **Google Bard Demo (Rủi ro thương hiệu):** Mô hình đưa ra một factual error (thông tin sai sự thật) ngay trong buổi demo giới thiệu công chúng, gây sụt giảm giá trị vốn hóa thị trường của tập đoàn. *Bài học:* Demo AI tạo niềm tin rất nhanh nhưng một lỗi sai cơ bản trước công chúng sẽ phá hủy lòng tin ngay lập tức.
*   **Gamma / Slide AI (Nỗi đau sửa đổi):** AI sinh slide tự động đạt 80% chất lượng rất nhanh. Nhưng 20% cuối cùng để tinh chỉnh layout, thay hình ảnh và sửa số liệu lại quá khó khăn cho người dùng $\rightarrow$ Người dùng bỏ cuộc và quay lại công cụ truyền thống (Powerpoint/Google Slides). *Bài học:* Sản phẩm AI phải thiết kế giao diện sao cho việc sửa lỗi (correction) cực kỳ dễ dàng.
*   **Air Canada Chatbot (Chịu trách nhiệm pháp lý):** Chatbot hỗ trợ khách hàng tự ý "bịa" ra chính sách giảm giá vé cho người đi dự đám tang. Tòa án phán quyết hãng bay phải bồi thường theo đúng lời của chatbot. *Bài học:* Bot trên website là một phần diện mạo pháp lý của doanh nghiệp. Phải có ranh giới và cơ chế kiểm soát nghiêm ngặt.

---

## 5. Thiết Kế UX Khắc Phục Lỗi (UX Recovery) & Lựa Chọn Đánh Đổi

### 5.1. Phân biệt Loại Lỗi: False Positive vs. False Negative
Trong thiết kế sản phẩm AI, ta luôn phải cân nhắc giữa hai loại sai sót:

*   **False Positive (Báo nhầm / Vu oan):** Hệ thống nhận diện sai một trạng thái lành tính thành độc hại.
    *   *Ví dụ:* Bộ lọc thư rác (Spam Filter) đánh dấu nhầm một email giao dịch quan trọng của khách hàng là Spam $\rightarrow$ **Hậu quả cực kỳ nghiêm trọng (mất khách hàng, đứt gãy công việc).**
*   **False Negative (Bỏ sót / Lọt lưới):** Hệ thống bỏ qua một trạng thái độc hại.
    *   *Ví dụ:* Bộ lọc spam để lọt một thư quảng cáo rác vào hộp thư chính $\rightarrow$ **Hậu quả nhẹ hơn (gây phiền toái nhỏ cho người dùng).**
*   *Quy tắc thiết kế:* Luôn tối ưu hóa hệ thống để giảm thiểu loại lỗi có **chi phí tổn thất cao hơn (Failure Cost)** đối với bài toán kinh doanh cụ thể.

### 5.2. Quyết định Product: Augment hay Automate?
*   **Augment (Hỗ trợ con người):** AI gợi ý bản nháp, con người duyệt và chỉnh sửa. Dành cho tác vụ rủi ro cao, cần tính chính xác lớn.
*   **Automate (Tự động hóa hoàn toàn):** AI tự ra quyết định và thực thi. Dành cho tác vụ volume lớn, rủi ro thấp, dung sai lỗi rộng.

---

## 6. Lộ Trình Khởi Động Dự Án MVP (Day 05 & Day 06)

Lộ trình chuyển dịch từ bài học lý thuyết sang sản phẩm thực tế thông qua Mini-Hackathon:

*   **Day 05: Find / Decide / Prep (Tìm kiếm, Quyết định và Chuẩn bị):**
    *   *Nhiệm vụ:* Tìm vấn đề thật của người dùng thật $\rightarrow$ Thiết kế luồng xử lý khi AI sai (failure path) $\rightarrow$ Cắt lát một bài toán nhỏ nhất đủ dùng (Build Slice) $\rightarrow$ Soạn thảo tài liệu đặc tả mỏng (**thin SPEC**).
    *   *Sản phẩm bàn giao:* Evidence pack (bằng chứng nỗi đau người dùng) + Thin SPEC draft.
*   **Day 06: Build / Test / Show (Xây dựng, Kiểm thử và Demo):**
    *   *M1 (Evidence gate):* Đảm bảo pain point thật và scope nhỏ.
    *   *M2 (Chạy được luồng chính):* Input $\rightarrow$ AI $\rightarrow$ Output hoạt động.
    *   *M3 (Freeze scope):* Khóa cứng các tính năng, hoàn thiện specs.
    *   *M4 (Dry run):* Chạy thử và chuẩn bị video backup demo.
    *   *M5/M6 (Demo zone):* Thuyết trình sản phẩm thực tế và nhận peer feedback.
