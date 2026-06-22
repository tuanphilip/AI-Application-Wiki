# Day 11 - Guardrails & AI Safety

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 11: Guardrails & AI Safety**. Trọng tâm là phân tích các mối đe dọa an ninh phổ biến trên mô hình ngôn ngữ lớn (LLM), danh mục lỗ hổng OWASP Top 10 (2025), cơ chế an toàn thiết kế (CaMeL, The Lethal Trifecta), thiết kế cổng bảo vệ (Input/Output Guardrails), mô hình Human-in-the-Loop (HITL) và thiết kế trải nghiệm người dùng tin cậy (Trust UX).

---

## 1. Tại Sao Cần Guardrails?

> **Triết lý an toàn:**
> *"Mất 10 ngày để xây dựng một Agent cực mạnh, có khả năng tự động hóa và kết nối rộng. Nhưng một Agent mạnh mà không kiểm soát được thì nguy hiểm gấp trăm lần một Agent yếu. **Agent không có Guardrails giống như một chiếc xe đua không phanh.**"*

### Các Sự Cố Thực Tế Điển Hình:
*   **DPD Chatbot:** Chatbot chửi chính công ty vận chuyển của mình khi bị người dùng kích động $\rightarrow$ Khủng hoảng truyền thông mạng xã hội.
*   **Air Canada Bot:** Chatbot tự bịa ra chính sách hoàn tiền sai lệch $\rightarrow$ Tòa án tuyên hãng bay phải bồi thường tiền mặt cho khách hàng theo cam kết của chatbot.
*   **Chevrolet Car Dealer Bot:** Người dùng thực hiện Prompt Injection ép chatbot đồng ý bán chiếc xe SUV trị giá hàng chục nghìn USD với giá \$1 $\rightarrow$ Gây thiệt hại tài chính và uy tín.
*   **EchoLeak & GeminiJack (2025):** Tấn công tiêm lệnh gián tiếp (Indirect Injection) không cần click từ email/tài liệu ngoài làm lộ lọt toàn bộ file, lịch làm việc và email của người dùng.

---

## 2. Các Mối Đe Dọa Phổ Biến & OWASP Top 10 (2025)

### 2.1. 4 Mối Đe Dọa Phổ Biến Nhất
1.  **Direct Prompt Injection (Tiêm lệnh trực tiếp):** Người dùng nhập câu lệnh để ghi đè lên System Prompt gốc (ví dụ: *"Ignore all previous instructions and..."*, *"You are now DAN..."*).
2.  **Indirect Prompt Injection (Tiêm lệnh gián tiếp):** Kẻ tấn công ẩn câu lệnh độc hại vào các nguồn dữ liệu ngoài (trang web, tài liệu PDF, email) mà Agent sẽ đọc. Khi Agent dùng RAG để đọc, nó sẽ thực thi chỉ dẫn ẩn đó mà người dùng không hề biết.
3.  **Jailbreaking (Vượt rào bảo mật):** Sử dụng các kỹ thuật tinh vi để vượt qua bộ lọc an toàn của LLM như Base64/ROT13 encoding, dịch sang ngôn ngữ ít phổ biến (Khmer, Zulu), roleplay (đóng vai), hoặc leo thang dần dần qua nhiều lượt chat (multi-turn escalation).
4.  **Data Leakage (Lộ lọt dữ liệu):** Agent vô tình tiết lộ system prompt, mã API keys, thông tin cá nhân (PII), hoặc lịch sử chat của phiên làm việc trước.

### 2.2. OWASP Top 10 for LLM Applications (Cập nhật 2025)
*   **LLM01:** Prompt Injection (Tiêm lệnh)
*   **LLM02:** Sensitive Info Disclosure (Lộ thông tin nhạy cảm)
*   **LLM03:** Supply Chain Vulnerabilities (Lỗ hổng chuỗi cung ứng mô hình/plugin)
*   **LLM04:** Data & Model Poisoning (Đầu độc dữ liệu/mô hình)
*   **LLM05:** Improper Output Handling (Xử lý đầu ra không an toàn $\rightarrow$ XSS/SQLi)
*   **LLM06:** Excessive Agency (Agent có quá nhiều quyền tự trị vượt mức cần thiết)
*   **LLM07:** System Prompt Leakage [MỚI] (Lộ prompt hệ thống)
*   **LLM08:** Vector & Embedding Weaknesses [MỚI] (Điểm yếu ở tầng Vector/Embedding)
*   **LLM09:** Misinformation (Cung cấp thông tin sai lệch)
*   **LLM10:** Unbounded Consumption (Tấn công từ chối dịch vụ DoS chi phí token)

---

## 3. Cơ Chế Phòng Thủ Bằng Thiết Kế (Defense-by-Design)

### 3.1. The Lethal Trifecta (Simon Willison 2025)
Một Agent sẽ cực kỳ nguy hại và dễ bị khai thác rò rỉ dữ liệu vô điều kiện nếu sở hữu đồng thời **cả 3 yếu tố sau**:
1.  **Private Data:** Quyền đọc dữ liệu riêng tư/nhạy cảm.
2.  **Untrusted Content:** Đọc nội dung không tin cậy từ bên ngoài (web, email, file).
3.  **External Comms:** Quyền gửi dữ liệu ra ngoài (gọi Webhook, gửi email, API).
*   *Nguyên tắc phòng thủ:* **Tuyệt đối không cấp cả 3 quyền này cho cùng một Agent.** Hãy tách vai trò thành nhiều Agent riêng biệt (ví dụ: Agent đọc email riêng, Agent gửi email riêng và không có quyền đọc dữ liệu nhạy cảm).

### 3.2. CaMeL Architecture (DeepMind 2025)
Mô hình an toàn phân quyền sử dụng hai thực thể LLM tách biệt:
*   **Privileged LLM (Đặc quyền):** Chuyên xử lý các lệnh đáng tin cậy của hệ thống, được quyền gọi các công cụ ngoài (tools).
*   **Quarantined LLM (Cách ly):** Chuyên xử lý dữ liệu không tin cậy từ bên ngoài (nội dung web, email). Mô hình này hoàn toàn bị cách ly, không được quyền gọi bất kỳ công cụ nào.
*   *Kết quả:* Giúp giảm tỷ lệ tấn công gián tiếp (indirect injection) từ mức >50% xuống dưới <2%.

---

## 4. Input & Output Guardrails

Hệ thống bảo vệ cần được áp dụng ở cả hai đầu của lượt gọi mô hình (Defense in Depth).

```
User Input ──> [Input Guardrails] ──> Model (LLM) ──> [Output Guardrails] ──> User / System
```

### 4.1. Input Guardrails (Lọc Trước Khi Xử Lý)
Ngăn chặn các input độc hại trước khi gửi tới LLM để tiết kiệm token và tránh kích hoạt hành vi sai:
*   *Validation Layer:* Kiểm tra độ dài, định dạng, bảng mã của input.
*   *Injection Detection:* Sử dụng Regex quét mẫu hoặc dùng mô hình phân loại chuyên dụng như **Llama Prompt Guard 2** (mô hình nhẹ 86M phát hiện injection đa ngôn ngữ).
*   *Topic Filter:* Từ chối các chủ đề nằm ngoài phạm vi hoạt động của Agent (ví dụ: bot tuyển dụng từ chối tư vấn đầu tư tiền số).
*   *Spotlighting:* Sử dụng định dạng hoặc ký hiệu đặc biệt để phân biệt rõ ràng đâu là câu lệnh đáng tin cậy của hệ thống và đâu là dữ liệu đầu vào chưa kiểm chứng từ bên ngoài.

### 4.2. Output Guardrails (Kiểm Tra Trước Khi Trả Lời)
Đảm bảo câu trả lời sinh ra an toàn và đúng định dạng:
*   *Content Filter:* Sử dụng Moderation API (ví dụ: OpenAI omni-moderation) để quét và chặn nội dung toxic, thù địch, bạo lực.
*   *Format Validation:* Ép cấu trúc đầu ra (JSON schema) và tự động loại bỏ các link ảo tưởng hoặc cấu trúc malformed trước khi gửi đi.
*   *Grounding Check:* Sử dụng các thư viện như **Guardrails AI Hub** để đối chiếu xem câu trả lời có khớp hoàn toàn với context lấy từ RAG hay không (phát hiện ảo giác).
*   *HITL Queue:* Đẩy câu trả lời vào hàng đợi duyệt thủ công nếu độ tin cậy (confidence score) của mô hình nằm dưới ngưỡng cho phép.

---

## 5. Mô Hình Phối Hợp Human-in-the-Loop (HITL)

Lựa chọn mô hình HITL phù hợp dựa trên mức độ rủi ro và khả năng hoàn tác của hành động:

1.  **Human-on-the-loop (Giám sát sau):** Agent tự động thực hiện hành động, con người xem xét nhật ký sau. Áp dụng cho: Tác vụ rủi ro thấp, dễ dàng hoàn tác (ví dụ: tạo bản nháp email, lưu log).
2.  **Human-in-the-loop (Duyệt trước):** Agent soạn đề xuất hành động (ví dụ: chuyển tiền, đặt chỗ), hệ thống tạm dừng và hiển thị chi tiết chờ con người bấm nút phê duyệt (**Approve**) trước khi gọi API thực thi. Áp dụng cho: Tác vụ rủi ro trung bình, khó hoàn tác.
3.  **Human-as-tiebreaker (Con người quyết định):** Con người trực tiếp đưa ra quyết định, Agent chỉ hỗ trợ tìm kiếm tài liệu và cung cấp thông tin tham khảo. Áp dụng cho: Tác vụ rủi ro cực cao, ảnh hưởng trực tiếp tới tính mạng hoặc pháp lý (ví dụ: chẩn đoán y khoa, phán quyết pháp lý).

---

## 6. Thiết Kế Trải Nghiệm Tin Cậy (Trust UX)

Xây dựng lòng tin với người dùng không chỉ bằng công nghệ, mà bằng thiết kế giao diện minh bạch qua **4 trụ cột Trust UX**:
1.  **Thinking Display (Hiển thị quá trình suy nghĩ):** Hiển thị rõ ràng Agent đang thực hiện bước nào, tìm được bao nhiêu tài liệu và trích nguồn (citations) cụ thể cho từng câu nói.
2.  **Calibrated Confidence (Định lượng độ tự tin):** Gắn các nhãn độ tin cậy `High/Medium/Low confidence` kèm theo lý giải trực quan tại sao mô hình tự tin ở mức đó.
3.  **Reversible Actions (Khả năng hoàn tác):** Mọi hành động có tác động bên ngoài đều phải có nút bấm khẩn cấp hoặc thời gian chờ để hoàn tác (ví dụ: *"Email đã được gửi đi. [Hủy gửi trong 30 giây]"*).
4.  **User Control (Quyền kiểm soát):** Cung cấp màn hình thiết lập cho phép người dùng bật/tắt các quyền cụ thể của Agent (ví dụ: được đọc danh bạ nhưng không được tự gửi tin nhắn).
