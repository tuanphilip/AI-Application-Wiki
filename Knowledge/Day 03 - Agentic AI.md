# Day 03 - Agentic AI

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 03: Agentic AI**. Trọng tâm bài học làm rõ sự khác biệt giữa Chatbot thông thường và Agent, tiêu chí đánh giá mức độ phù hợp (Agentic Fit), cơ chế hoạt động ReAct và các trường hợp không nên lạm dụng Agent.

---

## 1. Phổ Tiến Hóa: Từ Rule-Based Đến Autonomous Agent

Trong thực tế phát triển sản phẩm, các giải pháp tự động hóa ngôn ngữ được chia làm 4 cấp độ với khả năng thích nghi và độ phức tạp tăng dần:

$$\text{Rule-based Bot} \longrightarrow \text{LLM Chatbot} \longrightarrow \text{ReAct Agent} \longrightarrow \text{Autonomous Agent}$$

*   **Rule-based Bot:** Chạy theo các khối điều kiện `if/else` cứng nhắc, dễ đoán nhưng không thể xử lý đầu vào tự do hoặc ngoài kịch bản.
*   **LLM Chatbot:** Trả lời thông minh dựa trên ngữ cảnh được cung cấp nhưng chủ yếu xử lý một lượt (reactive/single-turn), không tự động gọi công cụ hay thực hiện hành động chuỗi.
*   **ReAct Agent (Reasoning + Acting):** Sử dụng các công cụ ngoài (tools) và chạy vòng lặp quan sát-hành động từng bước để giải quyết vấn đề.
*   **Autonomous Agent:** Tự trị hoàn toàn, giải quyết các mục tiêu dài hạn (long-horizon goals) thông qua chuỗi quyết định phức tạp và cơ chế tự sửa sai.

---

## 2. Bảng So Sánh Các Cấp Độ Giải Pháp

| Tiêu Chí | Rule-based Bot | LLM Chatbot | Agent (ReAct) |
| :--- | :--- | :--- | :--- |
| **Cơ chế xử lý** | Quy tắc `if/else` cố định | Sinh câu trả lời theo ngữ cảnh | Vòng lặp: Lập kế hoạch $\rightarrow$ Hành động $\rightarrow$ Quan sát |
| **Độ linh hoạt** | Thấp | Trung bình | Cao |
| **Bộ nhớ (Memory)** | Gần như không có | Ngắn hạn (trong context window) | Ngắn hạn + Bộ nhớ dài hạn (Vector store/DB) |
| **Sử dụng công cụ** | Cấu hình cứng (hard-coded API) | Gọi tool theo chỉ định của code | Chủ động chọn tool và tham số dựa trên trạng thái |
| **Chi phí (Cost)** | Thấp nhất | Trung bình | Cao (do nhiều lượt gọi LLM và duy trì vòng lặp) |
| **Rủi ro (Risk)** | Dễ kiểm soát, ít lỗi | Ảo giác (hallucination), sai format | Ảo giác + Gọi sai tool + Vòng lặp vô hạn (infinite loop) |
| **Use case điển hình** | Menu IVR, biểu mẫu đăng ký, validation | Hỏi đáp FAQ, CSKH cơ bản | Trợ lý đặt chỗ, coding assistant, nghiên cứu thị trường |

---

## 3. Tiêu Chí Đánh Giá Agentic Fit (Có Cần Agent Không?)

Không phải hệ thống nào sử dụng LLM cũng cần thiết kế theo kiến trúc Agent. Ta chấm điểm bài toán từ **1 đến 5** dựa trên 3 tiêu chí cốt lõi sau để đánh giá:

1.  **Multi-step Reasoning (Suy luận nhiều bước):** Bài toán có cần phân rã thành nhiều bước phụ thuộc nhau, bước sau kế thừa kết quả bước trước không?
2.  **Tool Interaction (Tương tác công cụ):** Hệ thống có cần kết nối với Search, APIs, Database, Trình duyệt hoặc File system để lấy thông tin khách quan không?
3.  **Dynamic Decision (Quyết định động):** Mỗi hành động tiếp theo có phụ thuộc trực tiếp vào kết quả vừa quan sát được ở bước trước không?

### Gợi ý thang điểm quyết định:
*   **0 - 5 điểm (Chatbot hoặc Rule):** Nên xây dựng hệ thống rule-based hoặc chatbot thông thường.
*   **6 - 10 điểm (Augmented Chatbot):** Tích hợp chatbot kết hợp với 1-2 công cụ gọi trực tiếp cố định (không cần loop suy luận phức tạp).
*   **11 - 15 điểm (Agent):** Rất đáng đầu tư xây dựng Agent dạng ReAct để tối ưu hiệu quả.

---

## 4. Cơ Chế Hoạt Động ReAct (Reasoning & Acting)

ReAct là khung tư duy nền tảng giúp LLM kết hợp khả năng suy luận và hành động trong thế giới thực thông qua vòng lặp:

```
┌────────────────────────────────────────────────────────┐
│                        Thought                         │
│   (LLM tự phân tích trạng thái hiện tại và lập kế hoạch)│
└──────────────────────────┬─────────────────────────────┘
                           │
                           v
┌────────────────────────────────────────────────────────┐
│                         Action                         │
│       (LLM quyết định gọi Tool nào với tham số gì)       │
└──────────────────────────┬─────────────────────────────┘
                           │
                           v
┌────────────────────────────────────────────────────────┐
│                      Observation                       │
│     (Hệ thống thực thi Tool và trả về kết quả cho LLM)  │
└──────────────────────────┬─────────────────────────────┘
                           │
                           v
              (Lặp lại hoặc trả lời cuối cùng)
```

### Các yếu tố ảnh hưởng đến khả năng gọi Tool đúng của Agent:
*   **Cách viết mô tả (Tool description):** Cần rõ ràng, ghi rõ khi nào nên dùng và khi nào không nên dùng. LLM đọc mô tả để chọn tool giống như con người đọc hướng dẫn sử dụng.
*   **Số lượng công cụ:** Quá nhiều công cụ trong context window sẽ làm loãng sự tập trung của LLM và tăng tỷ lệ chọn sai tool.
*   **Độ tương quan giữa các công cụ:** Các công cụ trùng lặp tính năng dễ khiến LLM bối rối.

---

## 5. Khi Nào Sử Dụng AI Agent Là Sai Bài Toán?

Đội ngũ phát triển cần tránh lạm dụng Agent trong các trường hợp sau (Anti-patterns):
1.  **Tác vụ 1 bước đơn giản:** Chỉ hỏi đáp thông thường hoặc phân loại văn bản cơ bản (FAQ).
2.  **Không có công cụ hỗ trợ:** Agent chỉ "suy nghĩ" suông mà không có API hay database nào để hành động thực tế.
3.  **Yêu cầu tính deterministic tuyệt đối (100%):** Các tác vụ như tính lãi suất ngân hàng, tính lương bắt buộc dùng code truyền thống. Sai sót của AI ở các mảng này là cực kỳ đắt đỏ.
4.  **Khắt khe về thời gian phản hồi (Latency):** Việc chạy vòng lặp ReAct từ 3-5 bước sẽ kéo dài độ trễ hệ thống, không phù hợp cho các tác vụ cần phản hồi tức thì dưới 1 giây.
