# Day 12 - Cloud Infrastructure & Deployment

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 12: Cloud Infrastructure & Deployment (Đưa Agent Lên Cloud)**. Trọng tâm bài học phân tích khoảng cách giữa môi trường phát triển (localhost) và vận hành thực tế (production), cách đóng gói Agent tối ưu bằng Docker, giải quyết các thách thức đặc thù của Agent (Timeout, Statefulness, Cost explosion), thiết kế hệ thống bảo vệ (API Gateway & Cost-Guard), các cấp độ deploy (từ PaaS Railway đến Tier 0 Managed Runtimes), cơ chế tối ưu GPU ở tầng phục vụ (Frontier-scale serving) và quy trình CI/CD tích hợp cổng đánh giá tự động (Eval Gates).

---

## 1. Từ Localhost Đến Production: Khoảng Cách Thực Tế

> **Triết lý vận hành:**
> *"It works on my machine (Chạy tốt trên máy tôi)* là câu nói nổi tiếng nhất trong lịch sử phần mềm. Đưa một Agent chạy thử nghiệm trên laptop lên phục vụ hàng trăm người dùng thực tế không đơn giản là copy code lên máy chủ. Sự khác biệt nằm ở tính nhất quán môi trường (Environment Parity) và khả năng tự phục hồi.*"

### 1.1. Bảng So Sánh Môi Trường Phát Triển vs. Vận Hành

| Khía cạnh | Dev (Localhost) | Production |
| :--- | :--- | :--- |
| **Dependencies** | Cài đặt thủ công (`pip install`) | Đóng gói immutable trong Container |
| **Config & Secrets** | File `.env` chứa khóa API thô | Biến môi trường hệ thống, Secrets Manager |
| **Networking** | `localhost:8000` | HTTPS, Domain riêng, Load Balancer |
| **User Access** | 1 người dùng duy nhất (chính mình) | $N$ người dùng đồng thời |
| **Failures & Logs** | Khởi động lại thủ công, log terminal | Tự động hồi sinh (auto-restart), Health check, Centralized Log |

### 1.2. 3 Điểm Khác Biệt Bản Chất Của AI Agent Với Web App Truyền Thống
Hạ tầng web thông thường dễ bị "gãy" khi vận hành Agent vì 3 lý do:
1.  **Vòng lặp suy nghĩ kéo dài (Long reasoning loop):** Agent chạy tool, lập luận có thể mất từ 10s - 60s+ (hoặc vài phút). Điều này dễ phá vỡ mức giới hạn thời gian chờ (timeout) mặc định của API Gateways/Proxies (29s - 60s).
2.  **Bộ nhớ hội thoại & trạng thái công cụ (Statefulness):** Agent lưu lịch sử chat, bộ nhớ làm việc. Điều này vi phạm nguyên tắc "Stateless Process" (Viết ứng dụng không lưu trạng thái) của chuẩn thiết kế phần mềm hiện đại 12-Factor App.
3.  **Chi phí tăng siêu tuyến tính (Non-linear token cost):** Mỗi lượt tương tác, Agent phải gửi lại toàn bộ lịch sử hội thoại + RAG context cũ $\rightarrow$ Tiêu tốn từ 50 - 1000× token so với một API chat thông thường.

---

## 2. Docker & Containerization Cho AI Agent (Chuẩn 2026)

Container đóng gói code ứng dụng, môi trường thực thi và thư viện phụ thuộc thành một đơn vị duy nhất chạy được ở mọi nơi.

### 2.1. Viết Dockerfile Hiện Đại (Multi-Stage + uv + Slim Base)
Để tối ưu kích thước ảnh (Target < 500MB) và tốc độ build, quy trình chuẩn 2026 khuyến nghị sử dụng **Multi-Stage Build** kết hợp với trình quản lý thư viện siêu tốc **uv** (viết bằng Rust, nhanh hơn pip ~10 lần).

```dockerfile
# Stage 1: Build dependencies bằng uv (Rust-fast)
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
WORKDIR /app
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project --no-dev

# Stage 2: Runtime image siêu nhẹ
FROM python:3.12-slim
WORKDIR /app
# Sao chép virtual environment đã build sạch ở stage trước
COPY --from=builder /app/.venv /app/.venv
COPY . .
ENV PATH="/app/.venv/bin:$PATH"

# Bảo mật: Không chạy dưới quyền root
RUN useradd -m app && chown -R app /app
USER app

CMD ["fastapi", "run", "main.py", "--host", "0.0.0.0"]
```

### 2.2. So Sánh Các Base Image Cho Python
*   `python:3.12` (~1.0 GB): Chứa quá nhiều công cụ biên dịch thừa $\rightarrow$ Dung lượng lớn, nguy cơ bảo mật cao.
*   `python:3.12-slim` (~150 MB): **Khuyến nghị.** Đầy đủ thư viện C chuẩn (glibc) tương thích tốt với các thư viện ML của Python (pandas, numpy...).
*   `distroless` (~66 MB): Bảo mật nhất vì không chứa shell, nhưng rất khó debug khi lỗi.
*   `python:3.12-alpine` (~55 MB): **TRÁNH DÙNG cho AI/ML.** Alpine sử dụng thư viện `musl libc` thay vì `glibc`. Các package Python biên dịch sẵn (manylinux wheels) không chạy được trên musl, buộc Docker phải biên dịch lại từ mã nguồn gốc $\rightarrow$ Thời gian build lâu hơn 50 lần (ví dụ build pandas+matplotlib: slim mất 30 giây/363MB, alpine mất 26 phút/851MB).

### 2.3. Quy Chuẩn Bảo Mật Image Docker (Container Security)
1.  **Quét CVE:** Thực hiện quét lỗi bảo mật tự động bằng `Trivy` hoặc `Docker Scout` trước khi đẩy lên Registry.
2.  **Non-root User:** Khai báo `USER app` để hạn chế quyền truy cập hệ thống của container nếu bị hacker chiếm quyền điều khiển.
3.  **Pin Digest:** Tránh ghi `FROM python:3.12`, hãy dùng SHA digest cố định (`FROM python:3.12-slim@sha256:...`) để đảm bảo ảnh build không bị thay đổi ngầm.
4.  **SBOM (Software Bill of Materials):** Liệt kê mọi thư viện trong ảnh Docker để đáp ứng quy định kiểm duyệt phần mềm (bằng lệnh `docker buildx --sbom`).

---

## 3. Khắc Phục Các Thách Thức Đặc Thù Của Agent

### 3.1. Giải Quyết Vấn Đề Timeout
Hạ tầng mạng luôn có ngưỡng giới hạn ngắt kết nối (AWS API Gateway: 29s, Heroku: 30s, Cloud Run: 60 phút).

> [!TIP]
> **Giải pháp khắc phục:**
> 1. **Token Streaming (SSE):** Gửi dữ liệu về client ngay khi sinh ra token đầu tiên. Chỉ cần byte đầu tiên được truyền đi trong vòng 30s, kết nối sẽ được giữ mở.
> 2. **Chuyển sang Async Job (Submit-and-poll):**
>    * Client gọi API $\rightarrow$ Backend nhận việc, ghi vào Queue (Redis/Celery/Cloud Tasks) và trả ngay một `job_id` (mất <100ms).
>    * Client định kỳ thăm dò (polling) trạng thái `job_id` hoặc đợi server gọi Webhook báo hoàn thành.
>    * Đối với các tác vụ xử lý hàng loạt không cần gấp: Dùng Batch API của các nhà cung cấp (OpenAI/Anthropic) để được giảm 50% chi phí và nhận kết quả sau 24h.

### 3.2. Triển Khai Streaming Bằng SSE (Server-Sent Events)
SSE là chuẩn truyền dữ liệu một chiều tối ưu cho LLM, chạy trên giao thức HTTP/1.1 thông thường, không cần nâng cấp phức tạp lên WebSockets.

```
Client ─────── HTTP POST /chat ───────> Agent Service
Client <─── SSE Stream (text/event-stream) ── Agent Service (yield chunks)
```

**Lưu ý khi cấu hình Proxy (Nginx):**
*   Mặc định Nginx bật tính năng đệm (`proxy_buffering on`), nó sẽ gom toàn bộ kết quả của Agent lại rồi mới trả về một lần $\rightarrow$ Làm mất tính năng streaming.
*   *Khắc phục:* Tắt đệm bằng cách cấu hình `proxy_buffering off;` hoặc trả về header `X-Accel-Buffering: no` từ backend ứng dụng.
*   Gửi heartbeat ping trống (ví dụ gửi chuỗi `: ping` mỗi 15s) để ngăn các gateway tự động ngắt kết nối vì nghĩ đường truyền nhàn rỗi (idle connection timeout).

### 3.3. Quản Lý Trạng Thái (Externalize State)
Hạ tầng Cloud có thể tắt hoặc khởi động lại (restart) các instance bất kỳ lúc nào. Do đó, Agent không được lưu bộ nhớ hội thoại trong biến global của RAM instance.

*   **Giải pháp:** Đẩy toàn bộ State ra ngoài (externalize state) sang cơ sở dữ liệu chuyên dụng (Postgres, Redis).
*   **LangGraph checkpointer:** Sử dụng `PostgresSaver` liên kết trực tiếp với mã `thread_id` của cuộc hội thoại. Khi đó, bất kỳ instance nào trong cụm Load Balancer nhận request cũng đều có thể phục hồi đầy đủ context để trả lời tiếp.
*   **Durable Execution (Thiết kế bền vững):** Với các Agent chạy nhiều ngày (long-running) hoặc cần khôi phục lại chính xác vị trí lỗi sau khi crash mà không muốn gọi lại LLM (tiết kiệm chi phí), hãy nâng cấp lên các framework như **Temporal**, **Restate** hoặc **Inngest**. Chúng ghi nhật ký (journal) từng bước đi của Agent; khi replay, hệ thống sẽ đọc từ nhật ký thay vì gọi lại API LLM ở các bước cũ.

---

## 4. Phổ Vị Trí Chạy Agent (Server vs. Client vs. On-Device)

Kiến trúc triển khai phụ thuộc rất lớn vào việc: Vòng lặp Agent, các API Keys và Công cụ (Tools) thực sự chạy ở đâu.

| Kiến trúc | Điểm chạy | API Key Security | Chi phí hạ tầng | Khả năng xử lý |
| :--- | :--- | :--- | :--- | :--- |
| **Server-side** | Máy chủ Backend | Rất an toàn (Server giữ) | Bạn trả 100% | Tốt nhất (Frontier models) |
| **Client-side BFF** | Trình duyệt + BFF Proxy | An toàn (BFF giữ key) | Bạn trả 100% | Tốt nhất (Frontier models) |
| **In-browser** | Trình duyệt (WebGPU) | Không cần API key | Bằng 0 (Client gánh) | Giới hạn (Model 1-3B) |
| **On-device** | NPU/GPU thiết bị | Không cần API key | Bằng 0 (Client gánh) | Giới hạn (Model 3B) |
| **Edge Node** | Cloudflare Workers AI | Ở phía Provider | Trả theo lượt gọi | Khá tốt (Model 7B+) |

> [!WARNING]
> **Cảnh báo bảo mật BFF (Backend-for-Frontend):**
> Nhiều lập trình viên thường dùng các biến môi trường dạng `VITE_` hoặc `NEXT_PUBLIC_` trực tiếp trong code frontend để gọi LLM. Các build tools sẽ đóng gói cứng các khóa này vào file Javascript bundle tải xuống trình duyệt, giúp kẻ xấu dễ dàng trích xuất để đánh cắp. Mọi Agent Client-side bắt buộc phải gọi LLM thông qua một lớp API Gateway/BFF Proxy ở backend để giấu khóa API.

---

## 5. Cấu Trúc API Gateway & Bảo Vệ Chi Phí (Cost Protection)

API Gateway là chốt chặn đầu tiên kiểm soát an ninh mạng, xác thực người dùng và bảo vệ ngân sách chạy Agent.

```
Client Request ──> [1. Authentication] ──> [2. Rate Limiter] ──> [3. Cost Admission Control] ──> Agent Logic
```

### 5.1. Mô Hình Phân Quyền Xác Thực (Authentication)
*   **API Key Header (X-API-Key):** Thích hợp cho MVP, giao tiếp nội bộ hệ thống (M2M), dịch vụ B2B đơn giản.
*   **JWT / OAuth 2.0:** Thích hợp cho ứng dụng web phục vụ người dùng cuối (User-facing App).
*   **OAuth 2.1 với PKCE:** Tiêu chuẩn bắt buộc đối với các **Remote MCP Servers** (đặc tả tháng 06/2025). MCP server hoạt động như một Resource Server xác minh token để tránh lỗi giả mạo quyền lực ("Confused Deputy").

### 5.2. Chốt Chặn Chi Phí (Cost-Guard & Admission Control)
Lỗi lặp vô hạn (Infinite loop) của Agent do logic retry sai lầm có thể đốt hàng chục nghìn USD chỉ trong vài ngày.

> [!IMPORTANT]
> **Quy tắc vàng chống cháy tài khoản:**
> 1. Không phụ thuộc hoàn toàn vào cảnh báo "Monthly Budget" của OpenAI/Anthropic vì nó là phản ứng trễ (alert gửi đi sau khi tiền đã bị tiêu).
> 2. Phải tự xây dựng **Pre-call Admission Control** ở phía backend để tính toán ước lượng chi phí của request dựa trên độ dài input và giới hạn token output tối đa trước khi thực hiện lượt gọi LLM thực tế:

```python
import os

MAX_USD_LIMIT = float(os.getenv("MAX_USD_PER_REQ", "0.05"))

def check_budget_guard(messages, model_id, max_output_tokens):
    # Đếm token đầu vào thực tế
    input_tokens = count_tokens(messages, model_id)
    
    # Ước lượng chi phí tối đa (Input + Max Output)
    est_cost = (input_tokens / 1e6 * INPUT_PRICE[model_id]) + \
               (max_output_tokens / 1e6 * OUTPUT_PRICE[model_id])
               
    if est_cost > MAX_USD_LIMIT:
        raise BudgetExceededException(f"Chi phí ước lượng ${est_cost:.4f} vượt giới hạn ${MAX_USD_LIMIT:.4f}")
    return est_cost
```

---

## 6. Các Cấp Độ Triển Khai Cloud (4 Tiers Deployment)

### 6.1. Bảng So Sánh Các Tầng Hạ Tầng

| Tier | Đại Diện | Thời gian Deploy | Cách tính giá | Đặc điểm phù hợp |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 0** *(Managed)* | Bedrock Agent Core, Vertex Agent Engine | Không cần lo infra | Theo token/session | Ship siêu tốc, có sẵn session memory, bảo mật microVM cô lập |
| **Tier 1** *(PaaS)* | Railway, Render, Fly.io | < 5 phút | Trả theo tài nguyên | Tuyệt vời cho MVP, dự án demo, học tập |
| **Tier 2** *(Managed Container)* | Google Cloud Run, AWS ECS Fargate | 10 - 15 phút | Pay-per-request / CPU | Sẵn sàng cho Production thực tế, hỗ trợ Scale-to-Zero |
| **Tier 3** *(Orchestrator)* | Kubernetes (EKS, GKE) | Phức tạp | Trả theo cụm server | Doanh nghiệp lớn, yêu cầu bảo mật nội bộ/compliance nghiêm ngặt |

*   *Lưu ý thay đổi công nghệ:* Dịch vụ **AWS App Runner** đã bị khai tử (Mar 2026), thay thế bằng **ECS Express Mode**. Google Cloud Run đã hỗ trợ triển khai GPU (GA 6/2025) cho phép chạy mô hình offline và scale-to-zero để giảm chi phí khi không có request.

---

## 7. Serving Model Ở Quy Mô Lớn (Frontier-Scale Serving)

Khi tự host mô hình hoặc xây dựng hệ thống quy mô lớn, việc tối ưu hóa hiệu năng GPU là yếu tố then chốt quyết định chi phí vận hành.

### 7.1. Các Kỹ Thuật Tối Ưu Tầng GPU
*   **Continuous Batching (Orca OSDI'22):** Thay vì đợi toàn bộ các câu trong lô xử lý xong (Static Batching), Continuous Batching chèn request mới vào ngay lập tức khi một request cũ hoàn thành decoding ở mỗi bước. Giúp tăng hiệu suất sử dụng GPU lên tới 36.9×.
*   **PagedAttention (vLLM SOSP'23):** Giải quyết phân mảnh bộ nhớ của KV Cache bằng cách chia nhỏ cache thành các block địa chỉ không liền mạch, tương tự như bộ nhớ ảo (Virtual Memory) trên OS. Giúp tăng throughput xử lý lên gấp 2-4 lần.
*   **Prefix Caching:** Tự động phát hiện và tái sử dụng KV Cache đối với các đoạn text cố định đầu Prompt (System prompt, Few-shot examples, RAG Context). Các nhà cung cấp giảm giá rất mạnh cho hit prefix cache (Anthropic giảm ~90%, OpenAI giảm ~50%, DeepSeek giảm ~10×).
*   **Speculative Decoding:** Dùng một model nhỏ (Draft model) dự đoán nhanh 5-8 token, sau đó đưa cho model lớn xác minh song song. Giúp giảm độ trễ (latency) 2 - 3 lần mà không ảnh hưởng tới chất lượng câu trả lời.
*   **Disaggregated Prefill/Decode (DistServe OSDI'24):** Tách biệt GPU xử lý lượt đọc prompt đầu vào (Prefill - hướng tính toán Compute-bound) và GPU xử lý sinh token đầu ra (Decode - hướng băng thông Memory-bandwidth-bound) để scale độc lập, tối ưu hóa tối đa thời gian trễ.

---

## 8. CI/CD Pipeline & Cổng Đánh Giá Eval Gates

Do tính chất ngẫu nhiên (Non-deterministic) của Generative AI, các bài test kiểm thử truyền thống (Assert chính xác chuỗi kết quả) không còn hiệu quả. Pipeline hiện đại tích hợp các cổng đánh giá chất lượng (Eval Gates) trước khi deploy.

### 8.1. Sơ đồ Pipeline CI/CD:

```
[Mã nguồn mới] ──> [Build & Push Container] ──> [Trivy CVE Scan] ──> [Eval Gate (Promptfoo)] ──> [Deploy Railway]
```

### 8.2. Ví dụ Cấu Hình GitHub Actions (`deploy.yml`)

```yaml
name: Deploy Agent to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Push Docker Image
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/my-org/my-agent:latest
          
      - name: Run Trivy Security Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'ghcr.io/my-org/my-agent:latest'
          exit-code: '1' # Dừng build nếu có lỗi bảo mật nghiêm trọng (HIGH/CRITICAL)
          severity: 'CRITICAL,HIGH'
          
      - name: Run AI Eval Gate
        run: |
          npm install -g promptfoo
          promptfoo eval --fail-on-error # Đánh giá độ an toàn và chất lượng câu trả lời, fail pipeline nếu điểm dưới ngưỡng
          
      - name: Deploy to Railway
        run: |
          npm install -g @railway/cli
          railway up --service my-agent-service
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

---

## 9. Các Mô Hình Sản Xuất Nâng Cao (Production-Grade Patterns)

### 9.1. Idempotency & Tránh Cái Bẫy "Just Retry"
*   **Nguy cơ:** Đối với Web thông thường, khi gọi API lỗi ta sẽ thử lại (retry). Tuy nhiên, Agent có khả năng thực hiện hành động ra đời thực (gọi tool gửi email, thanh toán tiền). Nếu retry mù quáng do mạng chập chờn (mạng lỗi sau khi thanh toán thành công), Agent sẽ thực hiện hành động đó lần thứ hai $\rightarrow$ Trừ tiền 2 lần.
*   **Giải pháp:** Mọi API gọi công cụ đột biến (mutating tools) phải kèm theo một khóa định danh duy nhất (`Idempotency-Key` dạng UUID). Server lưu kết quả chạy của key này trong Redis. Nếu nhận request trùng key, server trả lại ngay kết quả cũ mà không chạy lại logic công cụ.
*   **Thách thức của LLM:** LLM không sinh ra tham số giống hệt nhau ở hai lần suy nghĩ. Do đó, không thể hash nội dung bytes của parameters để so sánh. Cần định danh và dedup theo **Ý định (Intent-based)** hoặc liên kết chặt chẽ với mã giao dịch cụ thể từ trước (ví dụ: `order_id` do client truyền vào).

### 9.2. Quy Tắc Saga Pattern & Hành Động Bất Khả Hoàn
*   **Saga Pattern:** Thiết kế mỗi hành động của Agent luôn đi kèm một hành động bù trừ ngược lại (Compensating action) để undo nếu bước sau bị lỗi (ví dụ: `Book Hotel` $\leftrightarrow$ `Cancel Hotel`).
*   **Pivot Action:** Đối với các hành động không thể hoàn tác (gửi email đi, chuyển khoản thực sự), hãy đặt chúng ở cuối cùng của quy trình xử lý và bắt buộc phải có bước xác nhận từ con người (Human-in-the-Loop Approval Gate) trước khi thực thi.

### 9.3. An Toàn Tầng Mạng (Egress Control & Sandbox)
*   **Egress Allowlist:** Giới hạn chặt chẽ các địa chỉ IP/Domain bên ngoài mà container của Agent được phép gọi ra (chỉ cho gọi API OpenAI, Qdrant...). Đây là biện pháp hữu hiệu nhất chống rò rỉ dữ liệu qua tấn công gián tiếp (Indirect injection).
*   **Sandboxing:** Không chạy mã code do Agent tự sinh ra trực tiếp trên OS máy chủ. Sử dụng các môi trường microVM cô lập cực mạnh có thời gian khởi động ms như **Firecracker** hoặc **gVisor** (như cách E2B / Modal triển khai).

---

## 10. Checklist Sẵn Sàng Triển Khai Production (Checklist)

Trước khi bấm nút đưa ứng dụng lên phục vụ người dùng thực tế, hãy đảm bảo đã kiểm tra đầy đủ các hạng mục sau:

- [ ] **Dockerized:** Dockerfile thiết kế multi-stage, sử dụng ảnh python:slim, non-root user, dung lượng dưới 500MB.
- [ ] **Parity & Secrets:** Không hardcode API key. Tất cả config được nạp từ biến môi trường của Cloud provider.
- [ ] **SSE Streaming:** Endpoint giao tiếp chính hỗ trợ stream kết quả kèm theo cơ chế gửi heartbeat ping trống để lách timeout.
- [ ] **Stateless Design:** Trạng thái hội thoại được lưu ở database ngoài (Redis/Postgres), không lưu trong RAM ứng dụng.
- [ ] **Health Checks:** Cấu hình đầy đủ hai probe `/health` (Liveness) và `/ready` (Readiness).
- [ ] **API Security:** Bật xác thực API Key hoặc JWT ở cổng API Gateway.
- [ ] **Cost Guard:** Tích hợp kiểm tra budget ước tính trước mỗi lượt gọi LLM. Cấu hình spending limit cứng trên tài khoản provider.
- [ ] **Model Pinning:** Đóng băng ID phiên bản model (ví dụ dùng `claude-3-5-sonnet-20241022` thay vì `claude-3-5-sonnet-latest`) để tránh việc nhà cung cấp cập nhật ngầm làm gãy prompt.
- [ ] **Eval Gates:** Pipeline CI/CD tự động quét CVE và chạy bộ đánh giá chất lượng prompt trước khi deploy bản build mới.
