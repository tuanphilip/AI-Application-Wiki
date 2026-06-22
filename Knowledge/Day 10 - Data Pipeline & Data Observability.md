# Day 10 - Data Pipeline & Data Observability

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 10: Data Pipeline & Data Observability**. Trọng tâm bài học phân tích cách tự động hóa chuỗi xử lý dữ liệu (ETL/ELT), 6 tiêu chuẩn đánh giá chất lượng dữ liệu (Data Quality), thiết kế các cổng kiểm soát chất lượng (Quality Gates) và cách xây dựng cơ chế giám sát (Data Observability) để khắc phục lỗi hệ thống AI.

---

## 1. Data Pipeline Là Nền Tảng Của Mọi AI Product

> **Nguyên lý thực tế:**
> *"60% - 80% thời gian thực hiện dự án AI thực tế là dành cho công việc xử lý dữ liệu (Data Work) — không phải xây dựng mô hình. Nếu vector database bị nạp dữ liệu bẩn thì dù RAG Agent có xuất sắc đến mấy vẫn sẽ trả ra kết quả ảo giác. **Garbage In $\rightarrow$ Garbage Out**."*

### Sự Khác Biệt Giữa Data Pipeline Cho BI Và AI:
*   **Hệ thống BI (Business Intelligence) lỗi:** Dẫn đến số liệu hiển thị sai lệch trên các báo cáo/dashboard của nhà quản lý.
*   **Hệ thống AI (RAG/Agent) lỗi:** AI trực tiếp đưa ra thông tin sai lệch, khuyên sai luật, hoặc tự thực hiện hành động sai (giao dịch sai) trực tiếp với người dùng cuối.
*   *Do đó:* Pipeline cho AI đòi hỏi các quy trình phức tạp hơn: Chunking, Metadata enrichment, Embeddings, Retrieval validation, và Trace logging.

---

## 2. Các Mô Hình Xử Lý: ETL vs. ELT & Batch vs. Streaming

### 2.1. ETL vs. ELT Trong Hệ Thống AI

```
[ETL Flow]
Sources ──> Transform (Clean + Mask PII) ──> Load (Warehouse / Vector Store)

[ELT Flow]
Sources ──> Load Raw (Bronze Lakehouse)  ──> Transform (Chunk + Embed)
```

*   **ETL (Extract $\rightarrow$ Transform $\rightarrow$ Load):** Xử lý dữ liệu trước khi nạp vào kho lưu trữ.
    *   *Phù hợp cho:* Dữ liệu nhạy cảm cần ẩn danh (redact PII) hoặc loại bỏ mật khẩu/bí mật trước khi lưu trữ và embedding.
*   **ELT (Extract $\rightarrow$ Load $\rightarrow$ Transform):** Nạp dữ liệu thô vào kho trước, thực hiện xử lý sau.
    *   *Phù hợp cho:* Khi hệ thống có nhiều nguồn dữ liệu đa định dạng, cần lưu trữ thô để thử nghiệm nhiều thuật toán chunking hoặc mô hình embedding khác nhau.
*   **Xu hướng thực tế:** Nhiều đội ngũ chọn giải pháp **Hybrid** — Nạp thô dữ liệu trước (ELT), nhưng bắt buộc phải có bước lọc (ETL) các thông tin nhạy cảm trước khi embed hoặc truy vấn cho Agent.

### 2.2. Batch vs. Streaming
*   **Batch Processing (Xử lý theo lô):** Chạy theo lịch trình cố định (ví dụ: mỗi tiếng, mỗi ngày). Đơn giản, chi phí thấp, dễ debug, thích hợp cho dữ liệu huấn luyện, SOP nội bộ.
*   **Streaming Processing (Xử lý dòng dữ liệu):** Xử lý tức thì khi dữ liệu xuất hiện. Phức tạp, chi phí cao, độ trễ cực thấp (ms), thích hợp cho phát hiện gian lận (fraud detection), ngữ cảnh trực tiếp.

---

## 3. Ingestion & Transform Cho AI

### 3.1. Thiết kế Ingestion tốt cần đạt:
*   **Incremental Sync (Đồng bộ tăng trưởng):** Chỉ quét và lấy dữ liệu thay đổi kể từ lần chạy cuối (sử dụng CDC - Change Data Capture cho Database) thay vì quét toàn bộ dữ liệu.
*   **Idempotent Upsert (Ghi đè trùng lặp):** Chạy lại pipeline nhiều lần phải cho ra kết quả giống hệt 1 lần chạy, không tạo ra các chunk trùng lặp trong Vector DB.
*   **Xử lý Rate Limit & Backpressure:** Áp dụng cơ chế Exponential Backoff khi API của nguồn giới hạn tần suất gọi.

### 3.2. Chuẩn hóa văn bản (Text Normalization):
*   Khắc phục các lỗi Unicode dựng sẵn/tổ hợp (NFC/NFD) đặc trưng của tiếng Việt để mô hình Embedding không bị hiểu sai nghĩa.
*   Loại bỏ các thẻ HTML, ký tự rác và nhiễu OCR trong PDF trước khi chia nhỏ.

---

## 4. 6 Chiều Chất Lượng Dữ Liệu & Quality Gates

### 4.1. 6 Dimensions of Data Quality

```
             ┌─────────────────── Completeness ───────────────────┐
             │ - Không thiếu records/fields quan trọng            │
             ├─────────────────── Accuracy ───────────────────────┤
             │ - Dữ liệu đúng với thực tế và quy tắc              │
             ├─────────────────── Consistency ────────────────────┤
             │ - Định dạng đồng nhất giữa các hệ thống            │
             ├─────────────────── Timeliness ─────────────────────┤
             │ - Dữ liệu đủ mới (freshness)                       │
             ├─────────────────── Validity ───────────────────────┤
             │ - Định dạng tuân thủ đúng regex/domain            │
             ├─────────────────── Uniqueness ─────────────────────┤
             │ - Không bị trùng lặp (duplicates)                  │
             └────────────────────────────────────────────────────┘
```

### 4.2. Cấu Hình Các Cổng Kiểm Soát Chất Lượng (Quality Gates)
Thiết lập các chốt chặn tự động tại code pipeline để loại bỏ dữ liệu hỏng trước khi đưa tới Agent:
*   *Schema Gate:* Đảm bảo record có đầy đủ `content`, `doc_id`, `updated_at`.
*   *Freshness Gate:* Cảnh báo hoặc từ chối các tài liệu chính sách quá cũ (ví dụ: đã quá 2 năm chưa cập nhật).
*   *Content Gate:* Từ chối các chunk quá ngắn (ví dụ < 80 ký tự) hoặc có độ tin cậy OCR quá thấp.
*   *PII Gate:* Chặn đứng các record chứa số thẻ, token bảo mật hoặc thông tin cá nhân chưa được che dấu.

---

## 5. Data Observability: Theo Dõi & Gỡ Lỗi

**Data Observability** là khả năng theo dõi, cảnh báo và gỡ lỗi các vấn đề về dữ liệu trước khi chúng ảnh hưởng đến câu trả lời của Agent.

### 5.1. 5 Trụ Cột Của Data Observability
1.  **Freshness (Độ tươi):** Dữ liệu có được cập nhật đúng lịch trình không?
2.  **Distribution (Phân phối):** Dữ liệu có xuất hiện giá trị bất thường (null đột biến, khoảng giá trị lệch)?
3.  **Volume (Thể tích):** Số lượng bản ghi tăng/giảm bất thường hay không (ví dụ: số bản ghi sync đột ngột giảm về 0)?
4.  **Schema (Sơ đồ):** Cấu trúc bảng bị thay đổi (thêm/xóa cột) đột ngột không?
5.  **Lineage (Nguồn gốc):** Dữ liệu đi từ nguồn nào, qua những transform nào để đến được Vector DB?

### 5.2. Debug AI Từ Output Ngược Về Dữ Liệu (Lineage Trace)
Khi Agent trả lời sai, quy trình gỡ lỗi bắt buộc phải trace ngược theo 5 lớp:

```
[1. Output Layer]
Agent trả lời gì? Cite nguồn nào?
       │
       v
[2. Retrieval Layer]
Top-K chunks nào được lấy ra? Có chunk nào bị zero-hit?
       │
       v
[3. Index Layer]
Chunk đó được embed bằng model và version nào?
       │
       v
[4. Pipeline Layer]
Lượt chạy (Run ID) nào sinh ra chunk đó? Có vượt qua các Quality Gates không?
       │
       v
[5. Source Layer]
Tài liệu gốc trên hệ thống nguồn (Sharepoint, Notion) có đúng và mới nhất chưa?
```

*Để làm được điều này, mỗi lượt chạy pipeline và yêu cầu chat phải ghi log các trường:* `request_id`, `pipeline_run_id`, `retrieved_chunk_ids`, `source_doc_ids`, `source_version`, `embedding_model`, `latency_ms`.

---

## 6. Công Cụ Orchestration

Khi hệ thống phát triển lớn, việc sử dụng các script Python chạy bằng cron-job sẽ trở nên khó kiểm soát. Ta cần nâng cấp lên các công cụ Orchestration chuyên nghiệp:
*   **Apache Airflow:** Sử dụng đồ thị DAG (Directed Acyclic Graph) để quản lý thứ tự các task. Trực quan qua giao diện web, trưởng thành, thích hợp cho các tác vụ ETL lớn có lịch trình cố định.
*   **Prefect:** Hướng Python-native, lập trình nhanh, ít boilerplate hơn Airflow, phù hợp cho các luồng xử lý động.
*   **Dagster:** Tập trung vào tài sản dữ liệu (Asset-centric) thay vì quản lý task. Rất mạnh về vẽ sơ đồ nguồn gốc dữ liệu (Lineage) và tích hợp sẵn công cụ theo dõi chất lượng.
