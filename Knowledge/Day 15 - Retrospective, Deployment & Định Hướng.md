# Day 15: Retrospective, Enterprise Deployment & Định Hướng Chuyên Sâu

> Tổng hợp kiến thức từ Day 15, đánh dấu sự kết thúc của Phase 1: 15 ngày từ con số 0 đến việc xây dựng, triển khai (deploy), giám sát (monitor) và đánh giá (evaluate) một AI product thực tế. 

## 1. Retrospective: Nhìn Lại Hành Trình 15 Ngày

Mục tiêu của Retrospective không phải là để kể lể mọi chuyện, mà là **đúc kết những điều có thể dùng tiếp** trong các giai đoạn sau.

### 4 Câu Hỏi Retrospective Cốt Lõi
1. **Điều gì đã chạy tốt? (What went well)**: RAG pipeline chạy ổn định, Pair-debug tiết kiệm thời gian...
2. **Bài học quan trọng? (Lessons learned)**: Eval trước rồi tối ưu sau, prompt rõ ràng giúp giảm thiểu rework...
3. **Lần sau làm khác đi? (Do differently)**: Chốt schema sớm hơn, time-box cho phần research...
4. **Còn mơ hồ / Vướng ở đâu? (Confusing or unclear)**: Khi nào thực sự cần multi-agent? Guardrail bao nhiêu là đủ?...

### Hành Trình 15 Ngày – 4 Cột Mốc
- **Cột mốc 1 (Day 1 - 4): Nền Móng**. Hiểu cách điều khiển LLM (Prompt, Context, Tool, ReAct Agent) và đóng khung bài toán.
- **Cột mốc 2 (Day 5 - 6): Product Hóa**. Biến agent thành sản phẩm có UX tin cậy (Product Canvas, Prototype).
- **Cột mốc 3 (Day 7 - 10): Tri Thức & Kiến Trúc**. Chuyển từ một agent sang hệ nhiều phần (Embedding, Vector Store, RAG, Data pipeline, Multi-agent).
- **Cột mốc 4 (Day 11 - 14): Vận Hành**. An toàn, deploy, quan sát và đánh giá (Guardrails, Docker, Logging, Tracing, RAGAS, LLM-judge).

---

## 2. Triển Khai Thực Tế (Enterprise Deployment)

Lab deploy (ví dụ lên Railway) chỉ là bước đầu. Triển khai trong môi trường Enterprise đòi hỏi phải vượt qua nhiều rào cản kỹ thuật và pháp lý.

### Enterprise Challenges
- **Security & Compliance**: Dữ liệu không được rời khỏi Việt Nam, PII (thông tin cá nhân) phải được mã hóa, cần Audit trail cho mọi quyết định của AI, tuân thủ các tiêu chuẩn (PDPA, ngành tài chính).
- **Technical Constraints**: Phải tích hợp với hệ thống cũ (legacy, mainframe), giới hạn mạng (air-gapped), hạ tầng on-premise, và nguồn lực GPU hạn chế.

### Deployment Models
Do đặc thù doanh nghiệp, **không phải mọi thứ đều "push lên cloud" được**.
- **Cloud API**: Setup nhanh, tính phí per-token. Phù hợp cho MVP, Startup.
- **On-Premise**: Setup tính bằng tuần/tháng, chi phí Capex + GPU. Data control cao nhất. Phù hợp cho Ngân hàng, Chính phủ.
- **Hybrid**: Mix giữa Cloud và On-premise. Đây đang trở thành **mặc định cho doanh nghiệp ở Việt Nam**: *Dữ liệu nhạy cảm chạy on-prem, không nhạy cảm gọi Cloud API*.

### Self-Hosted LLM
Dùng khi có khối lượng truy vấn lớn hoặc cần bảo mật:
- **vLLM**: Production-grade inference, tối ưu throughput với PagedAttention.
- **Ollama**: Dễ cài đặt, chạy cục bộ 1 lệnh, phù hợp dev/demo.
> **Lưu ý**: Self-hosted chỉ thực sự tiết kiệm khi volume cao (VD: > 1M tokens/ngày). Dưới mức đó, dùng Cloud API thường rẻ hơn nếu tính cả chi phí GPU, vận hành và bảo trì.

---

## 3. Chi Phí Vận Hành (Cost Anatomy & Optimization)

### Cost Breakdown của một hệ thống AI
- **API Tokens (40 - 60%)**: Đây là phần chiếm tỷ trọng lớn nhất, tối ưu token mang lại ROI cao nhất.
- **Compute (CPU/GPU) (15 - 25%)**
- **Human Review (10 - 15%)**
- **Storage (Vector DB) (5 - 10%)**
- **Ops Monitor (5 - 10%)**
> **Hidden Costs**: Chi phí ẩn thường gấp 1.5 - 2 lần API cost (overhead khi retry, guardrail calls, monitoring, eval pipeline). Ngân sách dự kiến phải tính *tổng*, không chỉ riêng API.

### 4 Chiến Lược Tối Ưu Chi Phí (Giảm 30 - 70%)
1. **Model Routing**: Dùng mô hình rẻ/nhanh (như Haiku, GPT-4o-mini) cho các task đơn giản (~70% traffic), chỉ gọi mô hình đắt (Opus, GPT-4o) cho các task phức tạp.
2. **Semantic Caching**: Dùng embedding similarity để so sánh và trả về kết quả đã cache nếu query tương tự nhau.
3. **Prompt Compression**: Tóm tắt context trước khi gửi cho LLM để giảm input tokens.
4. **Self-Hosting**: Chuyển qua vLLM, Ollama khi đạt scale lớn (break-even ~1M+ tokens/ngày).

### Budget Planning: 3-Tier Model
- **MVP (< 100 req/ngày)**: $50 - 200/tháng (Cloud API + PaaS đơn giản).
- **Growth (100 - 5K req/ngày)**: $200 - 2K/tháng (Cloud API + Cloud Run/ECS).
- **Scale (> 5K req/ngày)**: $2K+/tháng (Hybrid / Self-hosted).
*Quy tắc: Bắt đầu ở MVP tier, chỉ nâng cấp khi traffic thực sự đòi hỏi để tránh premature optimization.*

---

## 4. Scaling & Reliability (Sẵn Sàng Cho Production)

Doanh nghiệp cần sự ổn định (SLA, uptime commitment - ví dụ 99.9% uptime).
- **Queueing (High Load)**: Dùng request queue (Redis Queue, Celery) để xử lý lượng tải tăng vọt. Thay vì timeout, user sẽ nhận thông báo "đang xử lý".
- **Stateless Agent**: Thiết kế agent không lưu trạng thái nội bộ để dễ dàng scale N instances.
- **Graceful Degradation**: Khi LLM API gặp sự cố, hệ thống không "sập" mà trả về cached response hoặc fallback message. Cần thiết lập circuit breaker (closed → open → half-open).
- **Redundancy & Failover**: Theo dõi liên tục và chuyển đổi hệ thống dự phòng khi cần thiết.

---

## 5. Job Landscape & Tương Lai Nghề Nghiệp

### Xu Hướng Tương Lai (2030)
- Kịch bản lý tưởng nhất: **Co-Pilot Economy** - AI phát triển dần và con người sẵn sàng thích nghi. AI đi vào công việc nhưng chủ yếu hỗ trợ, chưa thay thế hàng loạt.
- Nhóm 16% người dùng AI tạo ra giá trị vượt trội (**Frontier Professionals**):
  - Coi output của AI là điểm bắt đầu (không phải đáp án cuối).
  - Không giao phó tư duy cho AI.
  - Kỹ năng đắt giá nhất = Kiểm soát chất lượng output (50%) + Tư duy phản biện (46%).
  - Chủ động không dùng AI để giữ "tay nghề" trong một số công đoạn cốt lõi.

### Vai Trò Mới: Forward Deployed Engineer (FDE) / AI Success Engineer
- Nằm ở giao điểm giữa Software Engineering, Tư vấn khách hàng, và Thiết kế giải pháp đặc thù.
- Nhiệm vụ: Tới trực tiếp môi trường của khách hàng, hiểu bài toán, build custom workflows, và integrate AI solutions vào hệ thống doanh nghiệp.

### Skills Map Sau 15 Ngày (3 Pillars)
1. **CP1: Business**: Problem Statement, AI Readiness, PRD, Risk Assessment, ROI, UX Design.
2. **CP2: Infrastructure**: Vector Store, Data Pipeline, Docker, Cloud Deploy, Monitoring, Tracing.
3. **CP3: AI Engineering**: LLM API, ReAct Agent, Tool Calling, RAG, Multi-Agent, Guardrails, Evaluation.

---

## 6. Lựa Chọn Track Cho Giai Đoạn 2 (Chuyên Sâu)

Giai đoạn 1 cung cấp nền tảng chung. Giai đoạn 2 mở ra 3 ngã rẽ chuyên sâu (3 tuần):

1. **Track 1: AI Business & Product**
   - **Đối tượng**: AI PM, AI Business Analyst, AI Strategist.
   - **Nội dung**: Product Strategy, Financial Modeling & ROI, AI Governance & Compliance, Go-to-market.
   - **Output**: Business plan, financial model, compliance checklist.

2. **Track 2: AI Infrastructure & Data**
   - **Đối tượng**: AI Data Engineer, Platform Engineer, MLOps Engineer.
   - **Nội dung**: Lakehouse & Feature Store, vLLM deployment & optimization, CI/CD (LLMOps), GPU FinOps.
   - **Output**: Production-grade data pipeline, self-hosted LLM, CI/CD pipeline, monitoring dashboard.

3. **Track 3: AI Application**
   - **Đối tượng**: AI/LLM Engineer, AI Agent Developer.
   - **Nội dung**: Advanced Agent patterns, Memory & long-term context, GraphRAG, Fine-tuning & Model customization.
   - **Output**: Advanced agent system, custom fine-tuned model, production eval pipeline.

> **Kết luận**: Sau 15 ngày, với 15 labs và 1 deployed product, bạn đã từ một "beginner" trở thành một "builder". Portfolio vững vàng chính là cánh cửa mở ra các dự án thực tế lớn hơn trong doanh nghiệp.
