# Day 07 - Data Foundations - Embedding & Vector Store

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 07: Data Foundations - Embedding & Vector Store**. Đây là bước khởi đầu của tầng dữ liệu cho AI (Data Strategy), làm rõ cách tổ chức bộ nhớ cho Agent, cơ chế hoạt động của Embedding, Vector Store và các chiến lược chia nhỏ tài liệu (Chunking).

---

## 1. Chiến Lược Dữ Liệu Cho Sản Phẩm AI (Data Strategy)

> **Triết lý cốt lõi:**
> *"Mô hình AI mạnh đến đâu cũng vô dụng nếu dữ liệu đầu vào bị bẩn, cũ hoặc thiếu. Việc nâng cấp lên mô hình LLM đắt đỏ hơn không giải quyết được vấn đề dữ liệu bẩn. **Hãy sửa dữ liệu trước khi sửa mô hình.**"*

### 1.1. Ba Loại Dữ Liệu Agent Cần

| Loại Dữ Liệu | Đặc Điểm | Ví Dụ | Retrieval Fit (Mức Độ Phù Hợp Tìm Kiếm) | Phương Pháp Xử Lý |
| :--- | :--- | :--- | :--- | :--- |
| **Knowledge Data** | Ít thay đổi, dạng văn bản dài, chứa tri thức nền. | FAQ, chính sách công ty, tài liệu kỹ thuật, hợp đồng. | **Rất Cao** | Chia nhỏ (Chunking) + Embedding $\rightarrow$ Lưu Vector Database. |
| **Operational Data** | Thay đổi liên tục theo thời gian thực, dạng cấu trúc. | Trạng thái đơn hàng, logs giao dịch, số dư tài khoản, tồn kho. | **Thấp** | Gọi API, truy vấn SQL trực tiếp qua **Function Calling** thay vì embed. |
| **Contextual Data** | Gắn liền với phiên làm việc (session) hoặc hồ sơ người dùng. | Tên user, lịch sử chat gần đây, trang web đang xem, thiết bị. | **Trung Bình** | Điền trực tiếp (Inject) vào prompt hệ thống ở mỗi lượt gọi. |

### 1.2. Kim Tự Tháp Chất Lượng Dữ Liệu (Data Quality Pyramid)
*   **Enriched (Lớp 4):** Dữ liệu được gán thẻ tag, phân quyền truy cập (ACL), gán nhãn chất lượng.
*   **Structured (Lớp 3):** Dữ liệu được chia nhỏ hợp lý, gắn nguồn trích dẫn và thời gian cập nhật.
*   **Cleaned (Lớp 2):** Dữ liệu đã loại bỏ nhiễu, chuẩn hóa định dạng và metadata cơ bản.
*   **Raw (Lớp 1):** Tài liệu gốc hỗn loạn (PDF scan lỗi, HTML chứa thẻ rác, OCR nhiễu).
*   *Sai lầm phổ biến:* Đưa thẳng dữ liệu Raw vào Vector DB rồi kỳ vọng AI sẽ tự suy luận đúng.

### 1.3. Bảo Mật Dữ Liệu & Masking PII
Trước khi thực hiện embedding và lưu vào vector database, các dữ liệu định danh cá nhân nhạy cảm (PII - Personally Identifiable Information) bắt buộc phải được lọc/che dấu (masking):
*   *Tên cá nhân:* Thay bằng nhãn chung như `[PERSON]`.
*   *Số điện thoại / Email:* Sử dụng Regex để thay thế hoặc hash bảo mật.
*   *CMND / CCCD / Số thẻ tín dụng:* Xóa hoàn toàn khỏi văn bản.

---

## 2. Kiến Trúc Bộ Nhớ Của Agent (Memory Architecture)

Bộ nhớ của Agent được thiết kế chia làm hai tầng tương tự như não bộ con người:
1.  **Short-term Memory (Bộ nhớ ngắn hạn):**
    *   *Bản chất:* Nằm trực tiếp trong cửa sổ ngữ cảnh (**Context Window**) của mô hình tại lượt gọi hiện tại.
    *   *Đặc điểm:* Tốc độ truy xuất tức thì, chi phí rẻ, nhưng dung lượng bị giới hạn cứng và dễ đầy token.
2.  **Long-term Memory (Bộ nhớ dài hạn):**
    *   *Bản chất:* Nằm ngoài cửa sổ ngữ cảnh, được lưu trữ trong các cơ sở dữ liệu bên ngoài (Vector Store, Relational DB, Profile Store).
    *   *Đặc điểm:* Chỉ được truy xuất chọn lọc (retrieve) khi cần thiết, giúp lưu trữ tri thức tích lũy qua nhiều phiên làm việc.

### Phân Bổ Ngân Sách Token (Context Window Budget)
Để tránh việc context quá dài gây trôi lệch chỉ thị hoặc hết bộ nhớ sinh kết quả, cần thiết kế phân bổ token hợp lý:
*   *System Prompt & Persona:* Chiếm khoảng **15%**.
*   *Retrieved Context (Tài liệu RAG tìm được):* Chiếm nhiều nhất, khoảng **35%**.
*   *Conversation History (Lịch sử chat gần nhất):* Chiếm khoảng **25%** (cần cắt bớt/tóm tắt khi quá dài).
*   *Generation Budget (Token dành cho output của LLM):* Chiếm khoảng **25%** (nếu quá ít, câu trả lời sẽ bị cắt giữa chừng).

---

## 3. Bản Chất Của Embedding & Mô Hình Thống Kê

*   **Embedding** là quá trình chuyển đổi ngôn ngữ tự nhiên thành một chuỗi các vector số (không gian toán học đa chiều) biểu diễn ý nghĩa ngữ nghĩa của văn bản.
*   **Cosine Similarity:** Metric phổ biến nhất để đo khoảng cách (độ tương đồng về nghĩa) giữa hai vector $\vec{A}$ và $\vec{B}$.
    $$\text{Cosine Similarity}(\vec{A}, \vec{B}) = \frac{\vec{A} \cdot \vec{B}}{\|\vec{A}\| \|\vec{B}\|}$$
    *   `1.0`: Hoàn toàn cùng hướng (đồng nghĩa).
    *   `0.0`: Vuông góc (không liên quan).
    *   `-1.0`: Ngược hướng (đối lập hoàn toàn).
*   **Các mô hình Embedding phổ biến (2025 - 2026):**
    *   *text-embedding-3-small (OpenAI):* 1536 dims, giá rẻ, nhanh, là baseline tốt để bắt đầu.
    *   *text-embedding-3-large (OpenAI):* 3072 dims, chất lượng cao nhất của OpenAI.
    *   *voyage-3 (Voyage AI):* 1024 dims, cực mạnh cho code và dữ liệu đa ngôn ngữ.
    *   *Cohere embed-v4:* 1024 dims, thiết kế tối ưu cho tìm kiếm (search-tuned).
    *   *bge-m3 (BAAI):* Mô hình mã nguồn mở hàng đầu, hỗ trợ đa ngôn ngữ và ngữ cảnh dài.

---

## 4. Vector Store & Phân Loại Vector Databases

Vector Store lưu trữ 3 thành phần cốt lõi: **Original Text Chunk (Văn bản gốc)** + **Embedding Vector (Số hóa)** + **Metadata (Dữ liệu đặc tả)**.

### 4.1. Metadata Filtering: Yếu Tố Quyết Định Chất Lượng
Tìm kiếm tương đồng (semantic search) thuần túy là chưa đủ. Hệ thống cần kết hợp lọc thuộc tính (metadata filtering) để loại bỏ nhiễu:
*   *Source / File name:* Dùng để truy vết và hiển thị nguồn (citation).
*   *Category / Domain:* Lọc giới hạn vùng tìm kiếm (ví dụ: chỉ tìm trong thư mục `finance`).
*   *Freshness (Thời gian):* Tránh truy xuất các tài liệu cũ đã hết hiệu lực.
*   *Access Level:* Kiểm soát quyền truy cập của người dùng đối với tài liệu nhạy cảm.

### 4.2. Bản Đồ Phân Loại Vector Database (Vector DB Landscape)

| Database | Vị Trí Lưu Trữ | Ngôn Ngữ Hỗ Trợ | Lọc Metadata | Quy Mô (Scale) | Sử Dụng Tốt Nhất Cho |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Chroma** | Local (file) | Python / JS | Cơ bản | Nhỏ (Small) | Thử nghiệm, làm prototype nhanh, học tập. |
| **Pinecone** | Cloud SaaS | Đa ngôn ngữ | Tốt | Rất lớn (Large) | Dự án SaaS doanh nghiệp, không muốn quản trị hạ tầng. |
| **Qdrant** | Hybrid (Self-host/Cloud) | Rust-based | Rất tốt | Lớn (Large) | Tác vụ cần tốc độ cao và lọc metadata phức tạp. |
| **pgvector** | Database Add-on | SQL | SQL Where | Trung bình | Tận dụng hệ cơ sở dữ liệu PostgreSQL có sẵn của dự án. |
| **FAISS** | Local Library | C++ / Python | Không hỗ trợ | Mọi quy mô | Nghiên cứu học thuật, benchmark vector ngoại tuyến. |

---

## 5. Chiến Lược Chia Nhỏ Tài Liệu (Chunking Strategies)

Chia nhỏ tài liệu giúp LLM đọc đúng ngữ cảnh trọng tâm mà không làm loãng Token.

```
       [Document] ──> [Chunking] ──> [Embedding] ──> [Vector Database]
```

### 5.1. Các Phương Pháp Chunking

| Phương Pháp | Cơ Chế Hoạt Động | Ưu Điểm | Nhược Điểm |
| :--- | :--- | :--- | :--- |
| **Fixed-size token** | Cắt cứng nhắc theo đúng số lượng N tokens (ví dụ: 500 tokens). | Cực kỳ đơn giản, dễ cài đặt code. | Dễ bị cắt ngang câu, làm mất ý nghĩa ngữ nghĩa ở ranh giới. |
| **Sentence-based** | Tách tài liệu theo các dấu chấm kết thúc câu hoàn chỉnh. | Giữ nguyên vẹn ý nghĩa của từng câu. | Độ dài các chunk không đều, câu quá dài hoặc quá ngắn gây nhiễu. |
| **Section / Heading** | Tách dựa trên cấu trúc tiêu đề (H1, H2, H3). | Chunk trọn vẹn theo từng chủ đề nhỏ trong văn bản. | Đòi hỏi tài liệu gốc phải được format chuẩn Markdown/HTML. |
| **Recursive** | Cắt theo danh sách ký tự ưu tiên từ lớn đến nhỏ (`\n\n`, `\n`, ` `, `""`). | Linh hoạt, cố gắng giữ các đoạn văn gần nhau trong một chunk. | Cần tinh chỉnh tham số tùy theo loại tài liệu. |
| **Semantic** | Đo khoảng cách vector giữa các câu liên tiếp, cắt khi nghĩa thay đổi đột ngột. | Chất lượng ngữ nghĩa cực cao, gom đúng chủ đề. | Tốc độ xử lý chậm, tốn chi phí gọi mô hình embedding liên tục. |

### 5.2. Vai Trò Của Chunk Overlap
*   **Chunk Overlap** là phần nội dung trùng lặp giữa hai chunk kế tiếp nhau (thường cấu hình từ **10% - 20%** kích thước chunk, ví dụ overlap 50 tokens cho chunk 512 tokens).
*   *Tác dụng:* Đảm bảo thông tin ngữ cảnh liên tục, tránh hiện tượng mất nghĩa ở các từ nằm ở ranh giới biên giữa hai chunk.
*   *Lưu ý:* Overlap quá nhiều sẽ gây trùng lặp thông tin, tốn không gian lưu trữ và gây loãng kết quả tìm kiếm.
