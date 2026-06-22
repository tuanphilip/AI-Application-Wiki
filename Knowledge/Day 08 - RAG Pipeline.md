# Day 08 - RAG Pipeline

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 08: RAG Pipeline**. Trọng tâm bài học làm rõ kiến trúc RAG nâng cao, cách phân loại các hệ thống RAG, sự khác biệt giữa RAG và Fine-tuning, các lỗi thường gặp trong vận hành thực tế và bộ chỉ số đánh giá hiệu năng (Evaluation Metrics).

---

## 1. RAG Là Gì & Vì Sao RAG Bắt Buộc Trong Doanh Nghiệp?

**RAG (Retrieval-Augmented Generation - Tạo sinh tăng cường truy xuất)** là kỹ thuật tìm kiếm thông tin liên quan từ nguồn dữ liệu bên ngoài trước, sau đó đưa thông tin này vào ngữ cảnh (context) để yêu cầu LLM trả lời.

$$\text{RAG} = \text{Search} + \text{LLM}$$

### 1.1. Giải quyết các giới hạn của LLM thuần túy (Vanilla LLM):
1.  **Dữ liệu riêng tư (Private Data):** LLM không được huấn luyện trên tài liệu nội bộ, SOP, code hoặc cơ sở dữ liệu của công ty.
2.  **Thông tin lỗi thời:** Kiến thức của LLM bị đóng băng tại thời điểm cutoff. RAG giúp LLM cập nhật thông tin mới nhất theo thời gian thực.
3.  **Giảm thiểu Ảo giác (Hallucination):** Việc ép LLM trả lời dựa trên tài liệu được cung cấp (grounded context) giúp giảm tỷ lệ bịa đặt thông tin.
4.  **Cung cấp trích dẫn (Citations):** Trong môi trường doanh nghiệp, câu trả lời bắt buộc phải hiển thị nguồn tài liệu tham khảo rõ ràng để người dùng tự xác thực.

---

## 2. Kiến Trúc RAG Toàn Diện (End-to-End Pipeline)

Kiến trúc RAG bao gồm hai nhánh chính chạy song song hoặc bất đồng bộ:

```
[Nhánh 1: Ingestion Pipeline]
Tài liệu thô ──> Làm sạch ──> Chia nhỏ (Chunking) ──> Embedding ──> Vector DB

[Nhánh 2: Retrieval & Generation Pipeline]
User Question ──> Rewrite Question ──> Hybrid Search (Vector + BM25) ──> Retrieved Chunks
                                                                             │
                                                                             v
Final Answer <── LLM Generation <── Prompt + Context <── Reranking (Top-K) <┘
```

1.  **Ingestion Pipeline (Nhánh nạp liệu):** Thu thập tài liệu thô $\rightarrow$ Làm sạch $\rightarrow$ Chia nhỏ (Chunking) $\rightarrow$ Tạo vector (Embedding) $\rightarrow$ Lưu trữ trong Vector DB kèm Metadata.
2.  **Retrieval Pipeline (Nhánh truy xuất):** Nhận câu hỏi từ user $\rightarrow$ Tạo query embedding $\rightarrow$ Thực hiện tìm kiếm trong Vector DB.
3.  **Reranking (Chấm điểm lại):** Sử dụng mô hình Cross-Encoder để sắp xếp lại độ liên quan của các chunk tìm được. *Ví dụ:* Tìm kiếm vector lấy ra 20 chunks, Reranker lọc ra 5 chunks tốt nhất để nạp vào prompt.
4.  **Generation Pipeline (Nhánh sinh kết quả):** Ghép câu hỏi và các chunks sạch vào Prompt Template $\rightarrow$ LLM đọc và sinh câu trả lời cuối cùng có kèm trích dẫn nguồn.

---

## 3. Các Loại Kiến Trúc RAG Quan Trọng

*   **Basic RAG:** Tìm kiếm tài liệu liên quan một lần duy nhất rồi trả lời. Thích hợp cho: FAQ, chatbot tài liệu đơn giản.
*   **Conversational RAG (RAG hội thoại):** Có khả năng nhớ ngữ cảnh chat. Hệ thống sử dụng một bước trung gian để viết lại câu hỏi mơ hồ của user dựa trên lịch sử chat trước khi tìm kiếm.
    *   *Ví dụ:* User hỏi: *"Payment service dùng DB nào?"* $\rightarrow$ AI: *"Postgres"*. User hỏi tiếp: *"Nó deploy ở đâu?"* $\rightarrow$ Hệ thống rewrite thành: *"Payment service deploy ở đâu?"* trước khi thực hiện RAG search.
*   **Hybrid Search RAG:** Kết hợp giữa **Vector Search** (tìm theo ý nghĩa ngữ nghĩa) và **Lexical Search/BM25** (tìm theo từ khóa chính xác). Cực kỳ quan trọng trong ngành phần mềm để tìm chính xác mã lỗi (`ERR_500`), tên biến (`UserServiceImpl`), hoặc mã sản phẩm.
*   **Graph RAG:** Kết hợp Knowledge Graph (đồ thị tri thức) để hiểu mối quan hệ thực thể phức tạp. Phù hợp để hỏi đáp trên codebase lớn hoặc cấu trúc dependency hệ thống.
*   **Agentic RAG:** Agent tự trị chạy vòng lặp: Tìm kiếm $\rightarrow$ Nhận diện thiếu thông tin $\rightarrow$ Gọi tiếp tool/SQL hoặc tìm kiếm sâu hơn $\rightarrow$ Tổng hợp trả lời.

---

## 4. So Sánh RAG Với Fine-Tuning

| Tiêu Chí | RAG (Truy xuất) | Fine-Tuning (Tinh chỉnh mô hình) |
| :--- | :--- | :--- |
| **Bản chất** | Cung cấp tài liệu ngoài cho LLM "đọc trước khi thi" | Dạy lại trọng số mô hình để đổi "kiến thức và hành vi" |
| **Dữ liệu động / thay đổi liên tục** | **Rất phù hợp** (chỉ cần cập nhật DB) | Không phù hợp (tốn chi phí và thời gian retraining) |
| **Yêu cầu trích nguồn** | **Có** (bắt buộc thông qua metadata) | Rất khó (mô hình trả lời dựa trên tham số hóa) |
| **Khả năng giảm Ảo giác** | Tốt (có căn cứ thực tế) | Hạn chế (chỉ làm quen style, vẫn bịa dữ liệu) |
| **Độ phức tạp và chi phí** | Thấp hơn | Cao hơn nhiều (cần hạ tầng GPU, chuyên gia ML) |
| **Trường hợp áp dụng tốt nhất** | Tra cứu tài liệu, chính sách công ty, codebase | Thay đổi phong cách phản hồi, ép định dạng đầu ra phức tạp |

---

## 5. Những Lỗi Thường Gặp Khi Triển Khai RAG

1.  **Lựa chọn kích thước chunk sai:** Chunk quá to gây loãng ngữ cảnh, tốn token. Chunk quá nhỏ làm mất thông tin biên.
2.  **Thiếu Metadata:** Không lưu nguồn, ngày hiệu lực hoặc quyền hạn, dẫn đến việc agent trả lời thông tin cũ hoặc vi phạm bảo mật.
3.  **Chỉ dùng Vector Search:** Không tìm được các keyword chính xác hoặc mã lỗi viết tắt do khoảng cách vector của chúng nằm xa nhau.
4.  **Không tích hợp Reranking:** Đưa nhầm các chunk có độ tương đồng vector cao nhưng không liên quan thực tế vào prompt làm LLM bị đánh lạc hướng.
5.  **Không cập nhật dữ liệu (Stale index):** Vector database không có chính sách quét và làm mới tự động khi tài liệu gốc thay đổi.

---

## 6. Bộ Chỉ Số Đánh Giá Chất Lượng RAG (RAG Evaluation)

Để đánh giá một hệ thống RAG, ta cần đánh giá độc lập hai cấu phần: **Retrieval (Truy xuất)** và **Generation (Tạo sinh)**.

```
                  ┌───────────────────────────────┐
                  │         User Query            │
                  └───────┬───────────────┬───────┘
                          │               │
                 [Retrieval]             [Generation]
                          │               │
                 - Precision / Recall    - Faithfulness (Không bịa)
                 - Hit Rate / MRR        - Answer Relevance (Đúng trọng tâm)
```

### 6.1. Đánh giá phần Retrieval (Tìm kiếm):
*   **Hit Rate:** Tỷ lệ số lần tìm kiếm chứa ít nhất một chunk đúng trong Top-K kết quả.
*   **Mean Reciprocal Rank (MRR):** Đánh giá vị trí xuất hiện của chunk đúng nhất. Chunk đúng nằm càng cao trên danh sách kết quả thì điểm MRR càng lớn.
*   **Precision & Recall:** Tỷ lệ chunks tìm thấy thực sự liên quan và tỷ lệ không bị bỏ sót tài liệu đúng.

### 6.2. Đánh giá phần Generation (Tạo sinh):
*   **Faithfulness (Độ trung thực):** Đo lường xem câu trả lời của LLM có hoàn toàn dựa trên context được cung cấp hay tự bịa ra thông tin ngoài (hallucination check).
*   **Answer Relevance (Độ liên quan câu trả lời):** Câu trả lời có đi đúng vào trọng tâm câu hỏi của người dùng không.
*   **Context Relevance (Độ liên quan ngữ cảnh):** Các chunks tìm được có thực sự phục vụ cho việc trả lời câu hỏi hay chứa quá nhiều thông tin rác.
