# 📚 Cẩm Nang Tri Thức AI Application & Hệ Thống Agentic

Chào mừng bạn đến với **AI Application Wiki**! Đây là kho tài liệu tri thức tích lũy toàn diện, hệ thống hóa toàn bộ hành trình thiết kế, phát triển, vận hành và thương mại hóa các ứng dụng Trí Tuệ Nhân Tạo hiện đại (đặc biệt là các hệ thống **Agentic AI & Multi-Agent**).

Dự án được tổ chức khoa học nhằm cung cấp cái nhìn sâu sắc từ các bài toán thực tế để thiết kế, triển khai, vận hành và thương mại hóa các hệ thống AI Application & Multi-Agent phức tạp ở quy mô doanh nghiệp.

---

## 🗺️ Bản Đồ Cấu Trúc Tri Thức

Kho kiến thức này được chia thành các phần chính để bạn dễ dàng tiếp cận tùy thuộc vào nhu cầu:

1. **Hành Trình Phát Triển Ứng Dụng AI (15-Day AI Journey)**: Lộ trình từ nền móng lý thuyết đến sản xuất thực tế.
2. **Bản Đồ Giải Pháp Theo Vai Trò (Playbooks)**: Bộ quy chuẩn thực thi cho các vai trò PM/PO, Kỹ sư Fullstack, Kỹ sư AI và Sales B2B.
3. **Kiến Trúc Hệ Thống AI Application Tiêu Chuẩn**: Mô tả luồng xử lý và các module cấu thành của một hệ thống AI hoàn chỉnh.
4. **Từ Điển & Hệ Thống Hóa**: Các khái niệm cốt lõi, sơ đồ tư duy và thuật ngữ chuyên ngành được sắp xếp có hệ thống.

---

## 📅 Hành Trình Phát Triển Ứng Dụng AI (Lộ trình 15 Ngày)

Dưới đây là chi tiết lộ trình học tập, nghiên cứu và thực hành được hệ thống hóa qua từng chặng:

### 🧠 Chặng 1: Nền Tảng Lý Thuyết & Định Hình Bài Toán (Ngày 01 - Ngày 03)
*Tập trung vào nền tảng của mô hình ngôn ngữ lớn, đánh giá tính khả thi và định hình kiến trúc Agent cơ bản.*
* **Nền tảng về AI & Mô hình Ngôn ngữ Lớn (LLM)**
  * *Nội dung:* Sự tiến hóa từ Generative AI sang Agentic AI; cách thức hoạt động của Decoder-Only Transformer (cơ chế Attention); kinh tế học Token (Token Economy) tại thị trường Việt Nam; cú pháp gọi API tiêu chuẩn và kỹ thuật truyền dữ liệu streaming; tư duy "Vibe Coding" trong thời đại mới.
* **Xác định bài toán kinh doanh cho AI**
  * *Nội dung:* Ma trận lựa chọn giải pháp phù hợp (AI-Fit Matrix); quy trình phát triển sản phẩm AI qua 6 cổng kiểm định nghiêm ngặt (AI Product Gates); khung tài liệu hóa vấn đề (Problem Statement) và định lượng chỉ số hoàn vốn (ROI).
* **Thiết kế Agentic AI**
  * *Nội dung:* Bản đồ phân cấp Agent; cơ chế hoạt động của ReAct (Reasoning and Acting); thiết kế cấu trúc gọi công cụ (Tool calling) và cách kiểm soát vòng lặp vô tận (Infinite loops).

### 🛠️ Chặng 2: Kỹ Nghệ Prompt & Xây Dựng RAG Pipeline (Ngày 04 - Ngày 08)
*Nâng cao khả năng điều khiển LLM thông qua Prompt nâng cao và cung cấp tri thức doanh nghiệp bằng kỹ thuật RAG.*
* **Kỹ nghệ Prompt & Quản lý Ngữ cảnh**
  * *Nội dung:* Cấu trúc một gói ngữ cảnh (Context Packet) an toàn; bộ chiến lược WSCI (Write-Select-Compress-Isolate) quản lý cửa sổ ngữ cảnh; thang nâng cấp Prompt (Zero-shot $\rightarrow$ Few-shot $\rightarrow$ CoT $\rightarrow$ ToT); thiết kế schema cho công cụ.
* **Khởi động dự án AI (AI Product Kickoff)**
  * *Nội dung:* Tích hợp quy trình Scrum vào phát triển AI; đối phó với sự bất định của mô hình; thiết kế giao diện minh bạch tạo lòng tin (Trust UX); phương pháp nghiệm thu UAT và Dogfooding.
* **Thực hành phát triển MVP**
  * *Nội dung:* Quy trình xây dựng sản phẩm tối thiểu (MVP); phương pháp tích hợp nhanh các API và kết nối các thành phần chức năng.
* **Nền tảng dữ liệu - Embedding & Vector Database**
  * *Nội dung:* Bản chất của không gian vector ngữ nghĩa; cơ chế nhúng (Embedding); thuật toán tìm kiếm độ tương đồng (Cosine, Dot Product); cơ chế nạp dữ liệu và cách tổ chức các cơ sở dữ liệu vector.
* **Xây dựng RAG Pipeline nâng cao**
  * *Nội dung:* Quy trình nạp dữ liệu (Ingestion) và truy xuất (Retrieval) thực tế; kỹ thuật Chunking ngữ nghĩa & làm giàu Metadata; tìm kiếm lai (Hybrid Search: Vector + BM25); mô hình chấm điểm lại (Reranking); Grounding check và tạo trích dẫn (Citations) để chống ảo giác.

### 🤖 Chặng 3: Kiến Trúc Nâng Cao, Multi-Agent & Bảo Mật (Ngày 09 - Ngày 11)
*Mở rộng quy mô giải pháp với nhiều tác nhân cùng phối hợp và thiết lập các chốt chặn an toàn.*
* **Mô hình Đa tác nhân (Multi-Agent) & Tích hợp Hệ thống**
  * *Nội dung:* Phân biệt các mô hình Đa tác nhân (Supervisor-Worker, Pipeline, Debate); lập trình đồ thị hướng có trạng thái (Stateful Graph); giao thức MCP (Model Context Protocol) kết nối công cụ chuẩn hóa; cơ chế giao tiếp Agent-to-Agent (A2A).
* **Quy trình và Giám sát dữ liệu (Data Pipeline & Observability)**
  * *Nội dung:* Kiến trúc ETL/ELT chuyên biệt cho AI; kỹ thuật Change Data Capture (CDC); tính khả trùng (Idempotency) trên pipeline; giám sát sức khỏe dữ liệu qua 5 chiều (Freshness, Distribution, Volume, Schema, Lineage).
* **Bảo mật & Phòng thủ an toàn AI**
  * *Nội dung:* Nhận diện các mối nguy hại (Prompt Injection, Jailbreak, Prompt Leakage); bộ ba nguy hiểm (Lethal Trifecta); xây dựng cổng phòng thủ đầu vào/đầu ra (Input/Output Guardrails); XML Spotlighting và kiến trúc cô lập CaMeL sandbox.

### 🚀 Chặng 4: Triển Khai Cloud, Vận Hành APM & Tối Ưu Quy Mô (Ngày 12 - Ngày 15)
*Đưa hệ thống lên production, giám sát thời gian thực, đánh giá chất lượng và tối ưu hóa chi phí vận hành.*
* **Hạ tầng Cloud & Triển khai mô hình**
  * *Nội dung:* Các giải pháp tự host mô hình local/open-source; lượng tử hóa mô hình (Quantization AWQ/GGUF); quản lý KV Cache; tự động co giãn GPU (Ray Serve/KEDA); phục vụ mô hình nhúng (TEI).
* **Giám sát APM & SRE cho AI**
  * *Nội dung:* 4 trụ cột của AI Observability (Metrics, Logs, Traces, Continuous Eval); quản lý vòng đời vết (Span trace); đo lường chi phí và phiên làm việc của người dùng.
* **Đánh giá chất lượng mô hình (AI Evaluation)**
  * *Nội dung:* Xây dựng Golden Dataset; phương pháp LLM-as-a-judge; đánh giá chất lượng RAG (Faithfulness, Answer Relevance); tích hợp CI/CD kiểm thử tự động.
* **Tổng kết & Tối ưu chi phí quy mô lớn**
  * *Nội dung:* Cơ chế Bộ nhớ đệm ngữ nghĩa (Semantic Cache); bộ phân loại định tuyến mô hình động (Model Routing Classifier); nén ngữ cảnh (Prompt Compression).

---

## 📋 Bản Đồ Giải Pháp Theo Vai Trò (Playbooks)

Các Playbook nghiệp vụ được cấu trúc hóa để giải quyết các bài toán thực tế theo mô hình đối chiếu trực tiếp **Bài toán thực tế $\rightarrow$ Cách tiếp cận $\rightarrow$ Tech Stack $\rightarrow$ Công cụ mã nguồn mở**.

### 🤖 1. Kỹ sư AI (AI Engineer - Multimodal Life Cycle)
Tập trung vào chuỗi nghiệp vụ xử lý dữ liệu đa phương thức qua 8 giai đoạn cốt lõi:
* **Xác định dữ liệu:** Phân loại dữ liệu tĩnh (RAG), dữ liệu động (API/SQL) và dữ liệu ngữ cảnh (profile/session); lập ngân sách và dự báo chi phí token đầu vào/đầu ra.
* **Thu thập (Ingestion):** Cào web động/tĩnh; bypass anti-bot; trích xuất phụ đề; thu nhận luồng stream âm thanh/video trực tiếp (STT).
* **Chuyển đổi (Transformation):** Trích xuất bố cục PDF phức tạp; ẩn danh dữ liệu nhạy cảm (PII Masking); xử lý tệp ảnh/âm thanh (Upscaling, Background Removal, Audio Separation, Noise Reduction).
* **Lưu trữ & Truy xuất:** Thiết lập bộ lọc Metadata; tìm kiếm lai (Hybrid Search: Vector + BM25) và mô hình chấm điểm lại (Reranking); các mô hình truy xuất đồ thị (GraphRAG, ColBERT).
* **Tích hợp & Sáng tạo:** Lập sơ đồ Đa tác nhân (Supervisor-Worker, Pipeline); quản lý lưu trữ phiên dài hạn (Persistence); sinh hình ảnh/giọng nói/video và lipsync.
* **Giám sát & Đánh giá:** Ghi trace APM; phát hiện vòng lặp vô hạn; đánh giá chất lượng RAG (Faithfulness, Answer Relevance); đo lường độ chân thực của ảnh/giọng nói.
* **Bảo mật & Tối ưu:** Chặn tiêm lệnh và rò rỉ system prompt; thiết lập CaMeL sandbox; bộ nhớ đệm ngữ nghĩa (Semantic Caching).
* **Hạ tầng:** Lượng tử hóa mô hình; quản lý KV Cache; phục vụ mô hình nhúng (TEI); cân bằng tải API suy luận.

### 💻 2. Kỹ sư Fullstack (Fullstack Software Engineer)
Quản trị toàn bộ vòng đời phát triển phần mềm (SDLC) tích hợp AI:
* **Thiết kế hệ thống:** Kiến trúc dữ liệu; chuẩn hóa RESTful/GraphQL; lựa chọn Monolith vs Microservices.
* **Frontend:** Giao diện co giãn đa thiết bị; quản lý trạng thái (State Management); tối ưu tốc độ tải trang; kết nối WebSockets thời gian thực.
* **Backend:** Xác thực (JWT/OAuth2); phân quyền RBAC/ABAC; hàng đợi tác vụ nặng ngầm (Celery/BullMQ); giới hạn API Rate Limit.
* **Cơ sở dữ liệu:** Đánh chỉ mục SQL; công cụ chuyển đổi Database Migrations; bộ nhớ đệm Redis; phân chia Read/Write.
* **DevOps & Bảo mật:** Container hóa (Docker); GitOps CI/CD; quản lý khóa bí mật (Secrets); Zero-downtime deployment.
* **Vận hành:** Ghi log lỗi tập trung; theo dõi hiệu năng hệ thống (APM); sao lưu & phục hồi thảm họa.

### 📋 3. Quản trị Sản phẩm & Dự án (PM / PO)
Quản trị vòng đời sản phẩm từ ý tưởng đến tối ưu hóa sau phát hành:
* **Khám phá sản phẩm:** Lập User Persona; xây dựng Lean Canvas; định vị tính năng tối thiểu (MVP).
* **Quản lý yêu cầu:** Soạn thảo tài liệu PRD; kiểm soát phình to phạm vi; ưu tiên công việc (RICE Framework).
* **Lập kế hoạch:** Lộ trình Theme-based; ước lượng độ khó Agile (Fibonacci Sequence); quản lý điểm nghẽn phụ thuộc.
* **Quản lý đội ngũ:** Dung hòa xung đột designer/developer; tổ chức cải tiến Agile Retrospectives; giới hạn khối lượng công việc WIP.
* **Giao tiếp Stakeholders:** Quản lý kỳ vọng ban giám đốc; đồng bộ thông tin giữa các phòng ban.
* **Triển khai Sprint:** Xử lý việc phát sinh đột xuất; kiểm thử nghiệm thu UAT & Dogfooding.
* **Đo lường:** Đánh giá tính năng (HEART Framework); phản hồi khách hàng; đo lường mục tiêu (OKR).

### 💼 4. Sales B2B Chuyên Nghiệp (Sales B2B Full Stack)
Chiến lược bán hàng giải pháp công nghệ giá trị cao cho khách hàng doanh nghiệp:
* **Nghiên cứu:** Lập hồ sơ khách hàng lý tưởng (ICP); định vị người quyết định (BANT); thu thập thông tin tình báo (MEDDPICC).
* **Tiếp cận:** Cá nhân hóa email lạnh; khai thác nhu cầu thực tế (SPIN Selling); vượt qua người gác cổng.
* **Trình bày:** Thiết kế demo cá nhân hóa (Tailored Demo); chứng minh lợi ích kinh tế (Value-Based Selling).
* **Đàm phán:** Xử lý từ chối giá cả (LAER Framework); tháo gỡ rào cản kỹ thuật của bộ phận IT; cam kết giảm thiểu rủi ro.
* **Chốt hợp đồng:** Đẩy nhanh thủ tục pháp lý; giữ kết nối khi khách hàng im lặng.
* **Bàn giao:** Quy trình chuyển giao sang Customer Success; kích hoạt sử dụng trong 30 ngày đầu (Time-to-Value).
* **Nuôi dưỡng:** Đánh giá kinh doanh định kỳ (QBR); chiến lược bán thêm/bán chéo (Land and Expand).

---

## 📐 Kiến Trúc Hệ Thống AI Application Tiêu Chuẩn

Hệ thống kiến trúc AI ứng dụng hiện đại được xây dựng xoay quanh các khối chức năng tương tác chặt chẽ với nhau:

```text
 ┌────────────────────────────────────────────────────────┐
 │                      TRUST UX (Giao Diện Người Dùng)   │
 └──────────────────────────┬─────────────────────────────┘
                            ▼
 ┌────────────────────────────────────────────────────────┐
 │            INPUT GUARDRAILS (Cổng Phòng Thủ Đầu Vào)   │
 │         - Lọc chủ đề, phát hiện Prompt Injection       │
 └──────────────────────────┬─────────────────────────────┘
                            ▼
 ┌────────────────────────────────────────────────────────┐
 │            ORCHESTRATOR (Bộ Điều Phối Trung Tâm)       │
 │   - Shared State: task, plan, worker_results, trace    │
 │   - Supervisor Agent / Routing Decision (Rẽ nhánh)     │
 └──────┬───────────────────┬──────────────────────┬──────┘
        ▼                   ▼                      ▼
 ┌──────────────┐    ┌──────────────┐      ┌──────────────┐
 │   WORKER R   │    │   WORKER T   │      │   WORKER S   │
 │ (Retrieval)  │    │ (Tool Call)  │      │ (Synthesis)  │
 └──────┬───────┘    └──────┬───────┘      └──────┬───────┘
        │                   │                     │
        └─────────────────┐ │ ┌───────────────────┘
                          ▼ ▼ ▼
 ┌────────────────────────────────────────────────────────┐
 │           OUTPUT GUARDRAILS (Cổng Phòng Thủ Đầu Ra)    │
 │       - Chống ảo giác (Grounding check), ẩn danh PII   │
 └──────────────────────────┬─────────────────────────────┘
                            ▼
 ┌────────────────────────────────────────────────────────┐
 │            HUMAN-IN-THE-LOOP (Kiểm Duyệt Con Người)    │
 └──────────────────────────┬─────────────────────────────┘
                            ▼
 ┌────────────────────────────────────────────────────────┐
 │         ACTION EXECUTION (Thực Thi Giao Dịch/API)      │
 └────────────────────────────────────────────────────────┘
```

Mô hình này đảm bảo tính toàn vẹn, bảo mật, khả năng phục hồi lỗi bán phần (Graceful Degradation) và tăng mức độ tin cậy của ứng dụng AI trong môi trường sản xuất thực tế.

---

## 💡 Hướng Dẫn Đọc và Khai Thác Wiki Hiệu Quả

### 1. Môi trường đọc khuyên dùng
Tài liệu được lưu trữ dưới dạng Markdown tiêu chuẩn. Bạn nên sử dụng các trình soạn thảo hỗ trợ Markdown như **Obsidian** hoặc **VS Code** để có trải nghiệm xem nội dung tốt nhất, dễ dàng điều hướng và hiển thị trực quan các sơ đồ Canvas tương tác (.canvas) đi kèm trong thư mục.

### 2. Gợi ý lộ trình đọc theo vai trò

* **Đối với Kỹ Sư AI / Kỹ Sư Phần Mềm:**
  1. Đọc và nắm vững các phần lý thuyết từ **Ngày 01 đến Ngày 15** để xây dựng nền tảng vững chắc.
  2. Nghiên cứu sâu **AI Engineering Playbook** và **Fullstack Playbook** khi bắt tay vào giải quyết bài toán thực tế.
  3. Sử dụng sơ đồ hệ thống hóa và từ điển thuật ngữ để tra cứu nhanh khi phát triển dự án.

* **Đối với Product Manager / Business Analyst:**
  1. Tập trung vào phần bài học liên quan đến **Xác định bài toán**, **Khởi động dự án**, và **Đánh giá chất lượng mô hình**.
  2. Áp dụng trực tiếp các hướng dẫn trong **PM/PO Playbook** để lập kế hoạch, quản lý yêu cầu và điều phối Sprint.

* **Đối với Đội ngũ Sales / Business Development:**
  1. Tham chiếu từ điển thuật ngữ để hiểu rõ ngôn ngữ kỹ thuật khi trao đổi với khách hàng.
  2. Nghiên cứu và ứng dụng cẩm nang bán hàng **Sales B2B Playbook** để cải thiện tỷ lệ tiếp cận và chuyển đổi dự án.

---
*Chúc bạn có hành trình khám phá và làm chủ công nghệ AI thú vị! Mọi thắc mắc hoặc cập nhật tài liệu vui lòng liên hệ điều phối viên dự án.*
