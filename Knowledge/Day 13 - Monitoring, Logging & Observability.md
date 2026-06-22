# Day 13 - Monitoring, Logging & Observability

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 13: Monitoring, Logging & Observability (Giám sát, Ghi nhật ký & Khả năng quan sát)**. Trọng tâm bài học hướng tới việc chuyển đổi một Agent từ trạng thái chạy thử nghiệm (localhost/MVP) sang trạng thái vận hành ổn định trong production thực tế, giúp lập trình viên biết rõ tình trạng hoạt động của Agent trước khi người dùng phàn nàn. Nội dung bao gồm 4 trụ cột quan sát (4 pillars), các nhóm chỉ số đặc thù của AI (AI-specific metrics), thiết kế ghi nhật ký cấu trúc (Structured logging), truy vết phân tán (Distributed tracing), kiến trúc hạ tầng Prometheus + Grafana + OTel, thiết kế dashboard, thiết lập cảnh báo dựa trên triệu chứng (Symptom-based Alerting & SLOs), tối ưu hóa chi phí (Cost monitoring & optimization), quy trình gỡ lỗi sự cố thực tế, thu thập phản hồi của người dùng và các quy định pháp lý về bảo mật thông tin cá nhân (PDPL/GDPR).

---

## 1. Vì Sao Agent Cần Observability?

> **Triết lý vận hành:**
> *"Hệ thống chạy được ở Day 12 không có nghĩa là nó sẽ chạy tốt vào những ngày sau. Không có giám sát, cách duy nhất bạn biết hệ thống hỏng là khi người dùng phàn nàn. Đó là cách tệ nhất và đắt đỏ nhất để phát hiện lỗi."*

### 1.1. Từ Môi Trường Thử Nghiệm Sang Thực Tế Production
Sau khi hoàn tất Day 12, chúng ta đã deploy Agent lên cloud với public URL, endpoint health check và basic auth. Tuy nhiên, nếu không có observability, chúng ta hoàn toàn "mù" trước các câu hỏi quan trọng:
*   Agent đang phản hồi chậm hay nhanh ở bước nào?
*   Hệ thống đang tiêu tốn bao nhiêu chi phí token mỗi ngày?
*   Có bao nhiêu yêu cầu bị lỗi kỹ thuật hoặc trả lời sai thông tin (hallucination)?
*   Khi nào hệ thống cần mở rộng tài nguyên (scale up)?

### 1.2. Phân Biệt Giữa Monitoring vs. Observability
*   **Monitoring (Giám sát):** Tập trung vào các câu hỏi đã biết trước (**Known-Knowns**). Sử dụng dashboard và hệ thống cảnh báo dựng sẵn để trả lời câu hỏi: *"X có đang bị hỏng hay không?"*. Phù hợp cho các lỗi phổ biến đã được dự liệu trước.
*   **Observability (Khả năng quan sát):** Là thuộc tính nội tại của hệ thống, cho phép trả lời bất kỳ câu hỏi mới nào mà không cần phải thay đổi hay deploy lại code (**Unknown-Unknowns**). Bằng cách thu thập telemetry phong phú (logs, metrics, traces), lập trình viên có thể truy vấn và trả lời câu hỏi: *"TẠI SAO X lại bị hỏng?"* khi đối mặt với những lỗi chưa từng gặp.

### 1.3. Sự Khác Biệt Giữa AI Observability Và Giám Sát Phần Mềm Truyền Thống
Hạ tầng APM (Application Performance Monitoring) truyền thống hoàn toàn thất bại khi áp dụng cho AI Agent vì các đặc tính đặc thù sau:
1.  **Tính không xác định (Non-deterministic output):** Cùng một input đầu vào nhưng LLM có thể trả về các output khác nhau ở mỗi lượt chạy. Không thể kiểm thử bằng cách so sánh chuỗi (string matching). Phải đo lường chất lượng nội dung, không chỉ dừng lại ở pass/fail.
2.  **Lỗi âm thầm (Silent failures):** Ứng dụng không hề crash, API vẫn trả về mã trạng thái `200 OK` nhưng câu trả lời lại bịa đặt (hallucinated) hoặc chất lượng tệ đi. Không có exception nào được ném ra để hệ thống bắt lỗi tự động.
3.  **Chi phí biến động theo token:** Mỗi yêu cầu tốn tiền trực tiếp dựa trên số lượng token đầu vào/đầu ra. Một lỗi lặp vô tận (loop bug) do Agent gọi công cụ sai có thể đốt sạch ngân sách trong vài giờ, trong khi các chỉ số tài nguyên phần cứng thông thường như CPU/RAM hoàn toàn bình thường.
4.  **Các failure modes đặc thù của AI:** Lập luận sai, bịa tham số công cụ (hallucinated tool arguments), tràn ngữ cảnh (context overflow), prompt injection — đây là những lỗi mà các hệ thống APM cũ không hề có khái niệm.

### 1.4. Vòng Lặp Phản Hồi Trong Lý Thuyết Điều Khiển (Control Theory)
Khả năng quan sát tạo nên một vòng lặp phản hồi khép kín giúp hệ thống tự tối ưu và phục hồi:
$$\text{Agent System} \longrightarrow \text{Observe (Telemetry)} \longrightarrow \text{Analyze (Compare/SLO)} \longrightarrow \text{Act (Fix/Scale)} \longrightarrow \text{Agent System}$$
*   **MTTD (Mean Time To Detect):** Thời gian trung bình từ lúc sự cố phát sinh đến lúc phát hiện.
*   **MTTR (Mean Time To Recover):** Thời gian trung bình từ lúc phát hiện sự cố đến lúc khắc phục xong.
*   *Mục tiêu tối thượng:* Observability tốt nhằm giảm thiểu tối đa cả MTTD và MTTR.

---

## 2. 4 Pillars Của Observability Trong AI System

Để có được bức tranh toàn cảnh, hệ thống cần tích hợp đầy đủ 3 cột mốc truyền thống và 1 cột mốc đặc thù của AI:

```
                  ┌─────────────────────────────────────────┐
                  │       4 Pillars of Observability        │
                  └────────────────────┬────────────────────┘
          ┌────────────────────────────┼────────────────────────────┐
┌─────────┴─────────┐        ┌─────────┴─────────┐        ┌─────────┴─────────┐
│      METRICS      │        │       LOGS        │        │      TRACES       │
│  "Bao nhiêu/lâu?" │        │   "Gì xảy ra?"    │        │    "Tại sao?"     │
└───────────────────┘        └───────────────────┘        └───────────────────┘
                                       │
                             ┌─────────┴─────────┐
                             │  CONTINUOUS EVAL  │
                             │ "Còn đúng không?" │
                             └───────────────────┘
```

### 2.1. Chi Tiết Về Các Trụ Cột

| Trụ cột | Bản chất | Câu hỏi giải quyết | Công cụ ví dụ | Minh họa |
| :--- | :--- | :--- | :--- | :--- |
| **Metrics (Số liệu)** | Đo lường định lượng dạng time-series | *Bao nhiêu? Chậm thế nào?* (Ví dụ: Latency, error rate, cost) | Prometheus, Grafana | Bảng điều khiển tốc độ của xe |
| **Logs (Nhật ký)** | Ghi chép sự kiện chi tiết tại một thời điểm | *Chuyện gì đã xảy ra tại thời điểm X?* (Ví dụ: Chi tiết input/output thô) | Loki, JSON logs, Elasticsearch | Camera an ninh ghi hình |
| **Traces (Truy vết)** | Theo dõi hành trình yêu cầu xuyên suốt | *Tại sao hệ thống chậm/lỗi? Bottleneck ở đâu?* (Ví dụ: Span tree của agent loop) | Langfuse, Tempo, OpenTelemetry | Bản đồ định vị GPS |
| **Continuous Eval (Đánh giá)** | Trụ cột thứ 4. Đo lường chất lượng đầu ra liên tục | *Câu trả lời có còn chính xác và an toàn không?* (Ví dụ: Faithfulness score) | RAGAS, LLM-as-a-Judge, Phoenix | Người kiểm định chất lượng |

*   **Lưu ý:** `Continuous Eval` (đo trên production thật - online) bổ sung cho `Offline Eval` (đo bằng benchmark trước khi ship ở Day 14).

---

## 3. Các Nhóm Chỉ Số Đặc Thù Cho AI Agent (AI-Specific Metrics)

Hạ tầng SRE truyền thống dựa vào **4 Golden Signals** (Latency, Traffic, Errors, Saturation). Đối với AI Agent, chúng ta bắt buộc phải bổ sung thêm 2 nhóm chỉ số: **Cost** và **Quality**.

### 3.1. Phân Loại 4 Nhóm Metrics Cho AI Agent

#### A. Performance (Hiệu năng)
*   **Latency P50 / P95 / P99:** Phân phối thời gian phản hồi toàn bộ yêu cầu.
*   **Time To First Token (TTFT):** Thời gian từ lúc gửi yêu cầu đến khi ký tự đầu tiên xuất hiện. Đây là chỉ số quyết định cảm giác "nhanh/mượt" của người dùng khi dùng cơ chế streaming. Tiêu chuẩn 2026: P50 $\approx$ 0.5 - 1.0s, P95 $\approx$ 1.5 - 2.5s.
*   **Throughput:** Tốc độ xử lý yêu cầu (requests/s) và tốc độ sinh từ (tokens/s).
*   **LLM Call Duration:** Thời gian chạy riêng lẻ của từng lượt gọi LLM API.
*   *Lưu ý:* Các mô hình suy nghĩ sâu (Reasoning models như o1/o3/DeepSeek-R1) có phân khúc latency hoàn toàn riêng biệt (thường chậm hơn từ 5-30 lần). Cần bóc tách các chỉ số này ra để tránh làm nhiễu biểu đồ chung.

#### B. Quality (Chất lượng - Pillar 4)
*   **Hallucination / Faithfulness Rate:** Tỷ lệ câu trả lời bịa đặt hoặc không bám sát nguồn dữ liệu (retrieved context).
*   **Task Completion Rate:** Tỷ lệ Agent hoàn thành đúng mục tiêu người dùng yêu cầu.
*   **User Feedback Signals:** Tỷ lệ bấm thumbs up/down, tỷ lệ yêu cầu tạo lại câu trả lời (regenerate rate), tỷ lệ chuyển giao cho người hỗ trợ (escalate-to-human rate).
*   **Guardrail Trigger Rate:** Tỷ lệ hệ thống bảo vệ chặn các nội dung độc hại hoặc rò rỉ dữ liệu nhạy cảm.

#### C. Cost (Chi phí)
*   **Tokens Per Request (Input vs. Output):** Theo dõi số lượng token tiêu thụ. Output token thường đắt hơn Input token từ 5-6 lần. Do đó biểu đồ phải tách biệt hai chỉ số này.
*   **Cost Per Task / Request / User:** Rollup chi phí theo cấp độ tác vụ (vì một tác vụ lớn của Agent có thể bao gồm nhiều lượt gọi LLM liên tiếp).
*   **Cache Hit Rate (Prefix / Prompt Cache):** Tỷ lệ tái sử dụng KV cache nhằm giảm chi phí đầu vào.

#### D. Reliability (Độ tin cậy)
*   **Error Rate by Type:** Tỷ lệ lỗi phân loại theo taxonomy cụ thể (xem chi tiết ở mục 3.4).
*   **Tool-call Success/Failure Rate:** Đo lường tỷ lệ gọi công cụ thành công, lỗi định dạng (schema-fail) hoặc bị timeout.
*   **Loop Rate / Retry Rate:** Tỷ lệ Agent bị kẹt trong vòng lặp vô tận hoặc phải thực hiện thử lại liên tục.
*   **Retrieval Recall & Empty-result Rate:** Tỷ lệ tìm kiếm dữ liệu RAG không trả về kết quả hoặc trả về các đoạn văn không liên quan.

### 3.2. Phương Pháp Quan Sát: RED vs. USE

*   **RED Method (Request-centric):** Tập trung vào yêu cầu của người dùng. Đo lường **Rate** (Tốc độ request/s), **Errors** (Tỷ lệ lỗi), và **Duration** (Thời gian phản hồi P50/P95/P99). Giúp hiểu người dùng đang trải nghiệm gì.
*   **USE Method (Resource-centric):** Tập trung vào giới hạn của tài nguyên. Đo lường **Utilization** (Phần trăm tài nguyên sử dụng), **Saturation** (Độ nghẽn/hàng đợi), và **Errors** (Lỗi của tài nguyên). Giúp debug khi hệ thống quá tải (ví dụ: LLM API bị rate-limit dẫn đến saturation 95%).

### 3.3. Tầm Quan Trọng Của Percentiles (P95/P99)
Không bao giờ sử dụng giá trị trung bình (average) để giám sát Latency. Giá trị trung bình sẽ che giấu các trường hợp trễ cực lớn ở đuôi phân phối (long tail latency).
*   **Công thức compounding latency trong Agent:**
    Nếu Agent thực hiện quy trình gồm 10 bước (ví dụ: RAG $\rightarrow$ Agent Plan $\rightarrow$ 3x Tool calls $\rightarrow$ LLM Synthesize), và mỗi bước có tỷ lệ trễ đuôi $P99 = 5\text{s}$. Xác suất để người dùng gặp phải trải nghiệm trễ ít nhất một bước là:
    $$P_{\text{delay}} = 1 - (0.99)^{10} \approx 9.56\%$$
    Nghĩa là gần 10% khách hàng (1 trên 10 người) sẽ gặp tình trạng giật lag cực kỳ nghiêm trọng. Họ sẽ là những người phàn nàn và rời bỏ dịch vụ.

### 3.4. Phân Loại Chi Tiết Các Loại Lỗi (Error Taxonomy)
Để có thể tự động cảnh báo và gỡ lỗi nhanh chóng, mọi log lỗi cần được gán nhãn chính xác theo bảng phân loại sau:

| Loại lỗi | Nguyên nhân phổ biến | Phương án xử lý (Handle Strategy) |
| :--- | :--- | :--- |
| **LLM API 5xx** | Nhà cung cấp bị sập, quá tải hệ thống | Triển khai Exponential Backoff với Jitter, cấu hình Fallback Model |
| **LLM Timeout** | Model phản hồi quá chậm, nghẽn mạng | Thiết lập Circuit Breaker, cấu hình client timeout ngắn hơn gateway timeout |
| **Tool Call Failed** | API bên ngoài của công cụ bị lỗi hoặc sập | Retry giới hạn, degraded gracefully (bỏ qua bước đó hoặc trả về kết quả mặc định) |
| **Tool Schema Invalid** | LLM sinh ra cấu trúc JSON sai so với định dạng công cụ | Trích xuất lỗi cú pháp, gửi lại prompt kèm thông tin lỗi để LLM sửa lại |
| **Guardrail Block** | Nội dung vi phạm chính sách an toàn dữ liệu | Ghi nhật ký cảnh báo bảo mật, trả về câu trả lời thân thiện được thiết lập sẵn |
| **Context Overflow** | Lịch sử hội thoại hoặc dữ liệu RAG vượt quá context window | Áp dụng chiến lược cắt tỉa (truncate) hoặc tóm tắt (summarize) lịch sử chat |

### 3.5. Kiểm Soát Sự Trôi Lệch (Drift Monitoring)
Hệ thống AI chạy ổn định hôm nay có thể bị suy thoái chất lượng ngày mai do 3 loại drift:
1.  **Data Drift:** Phân phối dữ liệu đầu vào từ người dùng thay đổi (ví dụ: người dùng bắt đầu hỏi về các chủ đề hoàn toàn mới).
2.  **Concept Drift:** Mối quan hệ giữa input và output thay đổi (ví dụ: luật pháp thay đổi khiến câu trả lời cũ không còn đúng pháp lý).
3.  **Model Drift:** Nhà cung cấp tự động cập nhật phiên bản model ngầm (silent update) làm thay đổi hành vi suy luận hoặc định dạng đầu ra của Agent.
*   *Chỉ số phát hiện:* Sử dụng **PSI (Population Stability Index)** hoặc khoảng cách Cosine trên vector embeddings.
    $$PSI = \sum_{i} (p_i - q_i) \ln\left(\frac{p_i}{q_i}\right)$$
    *Ngưỡng đánh giá:* $PSI < 0.1$: Ổn định; $0.1 \le PSI \le 0.25$: Trôi lệch nhẹ; $PSI > 0.25$: Trôi lệch nghiêm trọng, cần cập nhật prompt hoặc fine-tune lại model.

---

## 4. Ghi Nhật Ký Cấu Trúc (Structured Logging)

Log không cấu trúc (Unstructured log) giống như ghi chép tay — rất khó tìm kiếm, lọc và tổng hợp. Ghi nhật ký cấu trúc (Structured logging) biến log thành dữ liệu có cấu trúc (JSON) để có thể truy vấn dễ dàng.

### 4.1. So Sánh Logs

*   **Không cấu trúc (Unstructured Text):**
    ```text
    2026-03-18 10:23:45 INFO Agent responded to req-abc123 in 1250ms with 890 tokens using claude-sonnet-4-6
    ```
*   **Có cấu trúc (Structured JSON):**
    ```json
    {
      "ts": "2026-03-18T10:23:45.012Z",
      "level": "INFO",
      "correlation_id": "req-abc123",
      "event": "agent_response",
      "latency_ms": 1250,
      "input_tokens": 640,
      "output_tokens": 250,
      "cost_usd": 0.0057,
      "model": "claude-sonnet-4-6",
      "tool_calls_count": 1
    }
    ```

### 4.2. Các Trường Thông Tin Bắt Buộc Log Cho 1 Yêu Cầu LLM
- [x] `correlation_id`: Khóa liên kết toàn bộ chuỗi log của một request đi qua nhiều dịch vụ.
- [x] `model` & `provider`: Tên chính xác của mô hình và nhà cung cấp (ví dụ: `claude-3-5-sonnet-20241022`, `anthropic`).
- [x] `prompt_template_id`: ID của mẫu prompt (Tránh ghi log prompt thô chứa dữ liệu nhạy cảm).
- [x] `input_tokens` & `output_tokens`: Số lượng token tiêu thụ đầu vào/đầu ra.
- [x] `latency_ms` & `ttft_ms`: Thời gian phản hồi tổng thể và thời gian trả về token đầu tiên.
- [x] `tool_calls`: Tên và kết quả của các công cụ đã gọi (sau khi đã được làm sạch dữ liệu PII).
- [x] `cost_usd`: Chi phí ước tính tính từ lượng token thực tế.
- [x] `error_type` & `stack_trace`: Ghi chi tiết nếu xảy ra lỗi.

### 4.3. Nguyên Tắc Bảo Mật PII (Personally Identifiable Information)
> [!WARNING]
> *Ghi log lộ thông tin cá nhân (PII) như Tên, Số điện thoại, CCCD, Email, Thẻ tín dụng là hành vi vi phạm nghiêm trọng luật bảo vệ dữ liệu (Luật PDPL Việt Nam / GDPR Châu Âu). Bạn sẽ bị phạt rất nặng khi bị thanh tra.*

*   **Nên Log:** Tóm tắt output (output summary), thông tin metadata của tool, latency, số lượng token, chi phí, mã lỗi và stack trace, Correlation ID.
*   **KHÔNG ĐƯỢC Log:** Dữ liệu thô chứa thông tin PII của khách hàng, khóa API (API keys), secrets, raw prompt chưa qua bộ lọc làm sạch.
*   **Cơ chế xử lý PII (PII Redaction):**
    *   Sử dụng regex để tự động lọc nhanh: Email, Số điện thoại, Số thẻ.
    *   Tích hợp thư viện NER (Named Entity Recognition) chuyên dụng như **Microsoft Presidio** để nhận diện tên người, địa chỉ. Cần cấu hình custom recognizers cho các trường thông tin Việt Nam (CCCD, cấu trúc số điện thoại VN).
    *   Thực hiện Redact/Mask tại điểm phát sinh (trước khi đẩy log ra console/stdout), không thực hiện ở cuối pipeline thu thập.

### 4.4. Code Thực Hành: Tự Động Hóa Correlation ID Với `structlog` & `contextvars`
Sử dụng `structlog` giúp tự động gắn Correlation ID xuyên suốt vòng đời của request mà không cần truyền biến thủ công qua từng hàm con.

```python
import uuid
import structlog
from structlog.contextvars import bind_contextvars, clear_contextvars

# Cấu hình structlog xuất ra JSON format
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars, # Tự động chèn các biến contextvars vào log
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso", utc=True),
        structlog.processors.JSONRenderer(),     # Kết xuất thành dạng JSON một dòng
    ]
)

log = structlog.get_logger()

async def handle_request(request):
    clear_contextvars() # Đảm bảo context sạch trước mỗi request
    
    # Tạo correlation_id độc nhất và gắn vào thread local context
    correlation_id = str(uuid.uuid4())[:8]
    bind_contextvars(
        correlation_id=correlation_id,
        user_id=request.user_id,
        feature=request.feature
    )
    
    # Từ đây, mọi dòng log.info, log.error... đều tự động mang theo correlation_id
    log.info("request_received", query_length=len(request.query))
    
    try:
        response = await agent.run(request.query)
        log.info("request_completed", status="success")
        return response
    except Exception as e:
        log.error("request_failed", error=str(e), exc_info=True)
        raise e
```

### 4.5. Cơ Chế Lấy Mẫu Nhật Ký (Log Sampling)
Khi lưu lượng truy cập quá lớn (ví dụ: 100k requests/ngày, mỗi request sinh ra 10 logs), chi phí lưu trữ log trên các dịch vụ như Datadog hay CloudWatch sẽ tăng đột biến vượt tầm kiểm soát.
*   **Chi thuật lấy mẫu (Sampling):**
    *   Giữ lại **100%** log thuộc nhóm `ERROR` và `WARN` (Dữ liệu quan trọng nhất để sửa lỗi - Non-negotiable).
    *   Giữ lại **10%** log thuộc nhóm `INFO` của các lượt chạy bình thường.
    *   Giữ lại **1%** hoặc tắt hoàn toàn log thuộc nhóm `DEBUG` trên môi trường Production.
    *   Giữ lại **100%** các request có latency vượt ngưỡng cảnh báo (ví dụ >10s) hoặc chi phí vượt mức thiết lập (ví dụ >$1/request).

### 4.6. Phân Biệt Giữa Audit Log vs. App Log
*   **App Log (Nhật ký ứng dụng):** Phục vụ mục đích gỡ lỗi và tối ưu hiệu năng của lập trình viên. Có thể cấu hình lấy mẫu (sampling), thời gian lưu trữ ngắn (30-90 ngày).
*   **Audit Log (Nhật ký kiểm toán):** Ghi nhận lịch sử *ai đã thực hiện hành động gì, vào lúc nào* nhằm đáp ứng các yêu cầu bảo mật, pháp lý và tuân thủ quy định (Compliance). Đòi hỏi tính toàn vẹn tuyệt đối (append-only, không được phép chỉnh sửa hay xóa), lưu trữ đầy đủ 100% không lấy mẫu, thời gian lưu trữ kéo dài từ 2 đến 7 năm. Nên lưu trữ trên S3 bucket bật chế độ Object Lock.

---

## 5. Truy Vết Phân Tán (Distributed Tracing Cho Agent)

Nếu logs giúp chúng ta thấy chuyện gì xảy ra ở từng dòng code, thì Tracing vẽ ra bức tranh toàn cảnh một request đi qua những hệ thống, công cụ nào và mất bao nhiêu thời gian ở mỗi nút xử lý.

### 5.1. Khái Niệm Trace, Span Và Context Propagation
*   **Trace (Yêu cầu đầu cuối):** Đại diện cho toàn bộ hành trình của một yêu cầu từ lúc bắt đầu tới lúc kết thúc. Được định danh bằng một `trace_id` duy nhất.
*   **Span (Đơn vị công việc):** Một đoạn xử lý logic con nằm trong trace (ví dụ: một câu truy vấn DB, một lượt gọi RAG, một LLM call). Mỗi span có `span_id`, thời gian bắt đầu, thời gian chạy, các thuộc tính (attributes) đi kèm và liên kết với cha của nó qua `parent_span_id`.
*   **Context Propagation:** Cơ chế truyền các thông tin định danh trace (`trace_id`, `span_id`) qua ranh giới giữa các dịch vụ hoặc hệ thống khác nhau (thông qua HTTP headers hoặc Queue metadata).

```
invoke_agent (Trace: 2500ms)
├── chat (Plan: 400ms)
├── execute_tool (rag_retrieve: 600ms)
│   ├── embed_query (200ms)
│   └── vector_search (350ms)
├── chat (Re-plan: 300ms)
└── chat (Synthesize: 1200ms)
```

### 5.2. OpenTelemetry (OTel) - Tiêu Chuẩn Mở Không Phụ Thuộc Nhà Cung Cấp
Sử dụng SDK của OpenTelemetry giúp instrument code một lần duy nhất. Telemetry dữ liệu thu thập được có thể xuất (export) đồng thời sang bất kỳ backend nào (như Langfuse, Datadog, Jaeger) mà không cần phải viết lại code của ứng dụng.

#### Định danh GenAI Semantic Conventions (`gen_ai.*` attributes)
Khi ghi nhận các trace liên quan tới Generative AI, bắt buộc tuân thủ chuẩn thuộc tính mở của OTel để đảm bảo tính tương thích:
*   `gen_ai.operation.name`: Hành động (`chat`, `execute_tool`, `invoke_agent`).
*   `gen_ai.provider.name`: Nhà cung cấp mô hình (`openai`, `anthropic`, `google`).
*   `gen_ai.request.model`: Tên model được yêu cầu sử dụng.
*   `gen_ai.usage.input_tokens`: Số lượng token đầu vào (Thay thế cho `prompt_tokens` cũ đã bị loại bỏ).
*   `gen_ai.usage.output_tokens`: Số lượng token đầu ra (Thay thế cho `completion_tokens` cũ đã bị loại bỏ).
*   `gen_ai.tool.name`: Tên công cụ được gọi thực thi.

### 5.3. 4 Mô Hình Nhận Diện Nút Thắt Cổ Chai (Bottleneck Patterns) Qua Trace
Khi phân tích biểu đồ dạng thác nước (waterfall trace), chúng ta dễ dàng phát hiện lỗi hiệu năng qua 4 cấu trúc điển hình:
1.  **Linear Block (Xử lý tuần tự):** Các span con xếp nối đuôi nhau liên tục. *Khắc phục:* Tìm các bước không phụ thuộc dữ liệu của nhau để chuyển sang chạy song song (parallelize).
2.  **N+1 Loop (Vòng lặp gọi API):** Hàng chục span ngắn cùng tên xếp chồng liên tiếp (ví dụ: Agent gọi tool lấy giá từng sản phẩm một). *Khắc phục:* Viết lại tool hỗ trợ lấy dữ liệu hàng loạt (batching / pre-fetching).
3.  **Long Idle Span (Nút thắt nhàn rỗi):** Một span kéo dài bất thường nhưng không tốn tài nguyên tính toán của CPU (thường do chờ phản hồi I/O mạng từ LLM API hoặc DB). *Khắc phục:* Thiết lập cache, tối ưu prompt để giảm độ dài sinh, hoặc cấu hình cơ chế streaming.
4.  **Retry Storm (Bão thử lại):** Xuất hiện liên tiếp các span lỗi cùng tên xen kẽ các khoảng thời gian chờ ngắn. *Khắc phục:* Tăng thời gian giãn cách thử lại (exponential backoff) kết hợp ngắt mạch (circuit breaker).

---

## 6. Bản Đồ Công Cụ LLM-Observability

Hệ sinh thái công cụ hỗ trợ quan sát mô hình ngôn ngữ lớn vô cùng phong phú. Lập trình viên cần chọn lựa công cụ phù hợp dựa trên quy mô và kiến trúc hệ thống:

| Tên công cụ | Loại hình | Điểm mạnh nổi trội | License / Chi phí |
| :--- | :--- | :--- | :--- |
| **Langfuse** | OSS / Cloud | Tracing chi tiết, tính toán chi phí token cực chuẩn, quản lý prompt hub và online eval. SDK v4 (2026) chạy trên nền OTel. | **MIT License**. Tự host docker miễn phí hoàn toàn. Cloud Hobby: free 50k traces/tháng. |
| **LangSmith** | SaaS | Tích hợp sâu với hệ sinh thái LangChain/LangGraph. Rất mạnh về thiết kế thử nghiệm (eval) và lưu trữ quỹ đạo suy luận (trajectory). | **Proprietary**. Chỉ hỗ trợ self-host bản Enterprise. Bản Cloud Free: 5k traces/tháng. |
| **Arize Phoenix** | OSS | Rất mạnh về trực quan hóa không gian vector (embedding visualization) và chạy đánh giá song song trong môi trường Notebook. | **Elastic License 2.0**. Miễn phí tự chạy local/server. |
| **LiteLLM Gateway** | Proxy/Gateway | Đóng vai trò làm lớp trung gian (proxy) trước toàn bộ các nhà cung cấp LLM. Kiểm soát an toàn chi phí, cache và phân bổ hạn mức (quota). | **MIT**. Phù hợp chạy làm chốt chặn bảo vệ chi phí tập trung. |

---

## 7. Thiết Kế Hạ Tầng Vận Hành: Prometheus + Grafana + OTel

Đối với các hệ thống lớn chạy production, mô hình hạ tầng chuẩn kinh điển là sử dụng **OpenTelemetry Collector** để thu nhận telemetry một lần, sau đó phân phối (fan-out) tới các dịch vụ lưu trữ chuyên biệt:

```
                  ┌───────────────────────────────┐
                  │           AI Service          │
                  │       (OpenTelemetry SDK)     │
                  └───────────────┬───────────────┘
                                  │ (OTel Protocol)
                                  ▼
                  ┌───────────────────────────────┐
                  │       OpenTelemetry           │
                  │         Collector             │
                  └───────┬───────┬───────┬───────┘
          ┌───────────────┘       │       └───────────────┐
          ▼                       ▼                       ▼
   ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
   │ Prometheus  │         │    Loki     │         │    Tempo    │
   │  (Metrics)  │         │   (Logs)    │         │  (Traces)   │
   └──────┬──────┘         └──────┬──────┘         └──────┬──────┘
          │                       │                       │
          └───────────────┐       │       ┌───────────────┘
                          ▼       ▼       ▼
                  ┌───────────────────────────────┐
                  │            Grafana            │
                  │         (Dashboards)          │
                  └───────────────────────────────┘
```

### 7.1. 3 Loại Metric Cơ Bản Trong Prometheus
*   **Counter (Bộ đếm):** Chỉ có tăng lên hoặc reset về 0 khi restart app (Ví dụ: Tổng số request, tổng số token tiêu thụ, tổng số lỗi).
*   **Gauge (Thước đo):** Có thể tăng hoặc giảm tự do (Ví dụ: Số request đang xử lý đồng thời, dung lượng hàng đợi, điểm số chất lượng eval score).
*   **Histogram (Biểu đồ phân phối):** Gom dữ liệu vào các bucket định sẵn để tính toán giá trị phân phối (Ví dụ: Latency bucket để tính toán chính xác giá trị P95/P99).

### 7.2. Code Thực Hành: Tích Hợp `prometheus_client` Trong Python
Đoạn code sau đây minh họa cách tự xuất (expose) các chỉ số đo lường đặc thù của AI ra endpoint `/metrics` để Prometheus tự động vào lấy dữ liệu (pull model):

```python
from prometheus_client import Counter, Histogram, start_http_server

# 1. Định nghĩa các chỉ số đo lường với các labels tương ứng
AGENT_REQUESTS = Counter(
    "agent_requests_total", 
    "Tổng số yêu cầu gửi tới Agent", 
    ["model", "status"]
)
AGENT_LATENCY = Histogram(
    "agent_latency_seconds", 
    "Phân phối thời gian phản hồi của Agent", 
    ["model"]
)
AGENT_TOKENS = Counter(
    "agent_tokens_total", 
    "Tổng số lượng token tiêu thụ", 
    ["model", "direction"]
)

def process_agent_request(user_prompt, model_id):
    # 2. Sử dụng context manager của Histogram để đo thời gian chạy tự động
    with AGENT_LATENCY.labels(model=model_id).time():
        try:
            response = agent.run(user_prompt)
            
            # Ghi nhận trạng thái thành công và số lượng token đầu ra
            AGENT_REQUESTS.labels(model=model_id, status="success").inc()
            AGENT_TOKENS.labels(model=model_id, direction="input").inc(response.usage.input_tokens)
            AGENT_TOKENS.labels(model=model_id, direction="output").inc(response.usage.output_tokens)
            return response
            
        except Exception as e:
            # Ghi nhận trạng thái lỗi
            AGENT_REQUESTS.labels(model=model_id, status="error").inc()
            raise e

# 3. Khởi động server HTTP ở cổng 8000 để expose dữ liệu cho Prometheus
start_http_server(8000)
```

### 7.3. Cảnh Báo Về Số Lượng Tổ Hợp Nhãn (Cardinality Explosion)
> [!CAUTION]
> *Cardinality là số lượng tổ hợp các giá trị nhãn (labels) của một metric. Nếu bạn đưa các giá trị có tính biến động vô hạn như `user_id`, `request_id`, hay `raw_prompt` vào làm label của Prometheus metric, số lượng time-series phát sinh sẽ bùng nổ cấp số nhân, làm sập hoàn toàn database Prometheus của bạn.*
> * *Coinbase từng nhận hóa đơn Datadog trị giá $65 triệu USD vào năm 2022 phần lớn vì lỗi Cardinality Explosion.*
> * *Quy tắc vàng:* Chỉ đặt các giá trị hữu hạn làm label (ví dụ: `model`, `status`, `tool_name`). Các giá trị định danh chi tiết thuộc về logs và traces, tuyệt đối không đưa vào metrics.

---

## 8. Thiết Kế Dashboard Cho Hệ Thống AI

Một dashboard tốt phải trả lời được câu hỏi cụ thể của người nhìn trong vòng 30 giây. Tránh nhồi nhét quá nhiều thông tin vào một màn hình duy nhất.

### 8.1. Cấu Trúc Dashboard 3 Layer Theo Phân Quyền
1.  **Layer 1: Overview (Dành cho Lãnh đạo/PM):** Chỉ hiển thị các chỉ số cấp cao: Hệ thống có hoạt động không (Uptime), mức độ hài lòng khách hàng (CSAT), tổng chi phí đã tiêu thụ, tỷ lệ hoàn thành tác vụ.
2.  **Layer 2: Detail (Dành cho Lập trình viên/SRE):** Hiển thị chi tiết kỹ thuật: Latency phân phối (P50/P95/P99), chi tiết lỗi theo nhóm (Error Taxonomy), số lượng token tiêu thụ, tỷ lệ gọi công cụ thành công.
3.  **Layer 3: Drill-down (Dành cho Gỡ lỗi/Debugging):** Tích hợp sâu link để mở trực tiếp Trace chi tiết của request lỗi hoặc công cụ tìm kiếm log thô.

### 8.2. 6 Biểu Đồ Bắt Buộc Phải Có Trên Dashboard Chi Tiết AI Agent
- [ ] **Request Rate (Traffic):** Biểu đồ đường thể hiện số lượng request/giây.
- [ ] **Latency Percentiles + TTFT:** Theo dõi tốc độ sinh token đầu tiên và tốc độ phản hồi tổng thể.
- [ ] **Error Rate (By Type):** Thể hiện chi tiết tỷ lệ lỗi (Lỗi LLM API, lỗi logic tool, lỗi định dạng).
- [ ] **Cost & Token Usage:** Theo dõi lượng token in vs. out và số USD tiêu thụ theo ngày.
- [ ] **Tool-call Success Rate:** Tỷ lệ chạy thành công của từng công cụ Agent sử dụng.
- [ ] **Quality / Eval Score:** Điểm số đánh giá chất lượng câu trả lời lấy mẫu từ production (Online Eval).

### 8.3. 5 Lỗi Thiết Kế Dashboard Phổ Biến (Dashboard Anti-patterns)
1.  **Wall of Metrics:** Thiết kế hơn 30 panel trên một trang dẫn đến nhiễu thông tin. Tối đa chỉ nên để 6 - 9 panel.
2.  **Khoảng thời gian xem quá dài:** Thiết lập mặc định hiển thị dữ liệu 30 ngày trên màn hình giám sát vận hành sẽ làm phẳng các đỉnh nhọn đột biến (spikes) phát sinh trong giờ. Mặc định nên để khung xem từ 1 - 3 giờ gần nhất.
3.  **Thiếu đường cơ sở (Baseline/SLO lines):** Nhìn thấy chỉ số $P95 = 2.5\text{s}$ nhưng không biết là tốt hay xấu. Bắt buộc phải vẽ một đường nét đứt biểu diễn ngưỡng mục tiêu SLO để so sánh trực quan.
4.  **Chỉ số không có đơn vị đo:** Viết con số `Cost: 1250` mà không ghi rõ đơn vị là USD, VNĐ, hay số lượng Token.
5.  **Không tự động làm mới (No Auto-refresh):** Màn hình giám sát vận hành phải tự làm mới sau mỗi 15 - 30 giây.

---

## 9. Cảnh Báo & Mục Tiêu Chất Lượng Dịch Vụ (Alerting & SLOs)

### 9.1. Thiết Lập Bảng Cảnh Báo Tiêu Chuẩn Cho AI Agent

| Chỉ số cảnh báo | Ngưỡng kích hoạt (Threshold) | Mức độ nghiêm trọng | Kênh nhận tin | Hành động ứng phó |
| :--- | :--- | :--- | :--- | :--- |
| **Uptime** | $< 99\%$ trong 5 phút | Critical (Khẩn cấp) | PagerDuty / Call | On-call engineer dậy xử lý hạ tầng sập |
| **Error Rate** | $> 5\%$ trong 5 phút | Critical (Khẩn cấp) | Slack + SMS | Kiểm tra log xem nhà cung cấp LLM hay DB sập |
| **Daily Cost** | Vượt quá $120\%$ budget ngày | Critical (Khẩn cấp) | Email + Slack | Tắt tạm thời các tài khoản test, kiểm tra loop bug |
| **Latency P95** | $> 5\text{s}$ trong 10 phút | Warning (Cảnh báo) | Slack | Kiểm tra lại thời gian chạy của vector store/RAG |
| **Tool-call Failure**| $> 10\%$ trong 10 phút | Warning (Cảnh báo) | Slack | Kiểm tra xem API bên thứ ba có đổi schema không |
| **Eval Quality Score**| Giảm quá $10\%$ so với tuần trước| Warning (Cảnh báo) | Slack | Thu thập mẫu để Prompt Engineer tinh chỉnh prompt |

### 9.2. Cảnh Báo Dựa Trên Triệu Chứng (Symptom-Based Alerting)
*   **Cause-Based Alerting (Cảnh báo theo nguyên nhân):** *"Báo động! CPU của máy chủ vượt quá 85%"*. Kiểu cảnh báo này tạo ra rất nhiều cảnh báo giả (false positives) vì CPU cao có thể do hệ thống đang chạy tác vụ nền và hoàn toàn chưa gây ảnh hưởng tới người dùng cuối. Chỉ nên dùng để debug.
*   **Symptom-Based Alerting (Cảnh báo theo triệu chứng):** *"Báo động! Tỷ lệ lỗi trả về cho người dùng vượt quá 5% hoặc Latency P95 > 5s"*. Đây là cảnh báo dựa trên những gì khách hàng thực sự cảm nhận được. Bắt buộc phải cấu hình kênh báo khẩn cấp (Page) cho loại cảnh báo này.

### 9.3. Tránh Hội Chứng Lờ Cảnh Báo (Alert Fatigue)
Khi hệ thống gửi đi quá nhiều cảnh báo không quan trọng, đội ngũ kỹ thuật sẽ có xu hướng tắt âm thanh hoặc lờ chúng đi. Đến khi sự cố nghiêm trọng xảy ra thực sự, nó sẽ bị chôn vùi trong đống tin nhắn rác.
*   *Quy tắc Google SRE:* Chỉ kích hoạt chuông báo khẩn cấp (Page) khi sự cố đó đòi hỏi con người phải can thiệp để xử lý ngay lập tức. Nếu sự cố có thể tự phục hồi hoặc không ảnh hưởng trực tiếp tới SLO, hãy tự động tạo ticket hỗ trợ hoặc đẩy vào dashboard để xem xét sau trong giờ làm việc.

### 9.4. Định Nghĩa Chính Xác SLI, SLO Và SLA
*   **SLI (Service Level Indicator - Chỉ số dịch vụ):** Con số đo lường thực tế tại một thời điểm.
    *   *Ví dụ:* Tỷ lệ các request thành công có thời gian phản hồi $< 3\text{s}$.
*   **SLO (Service Level Objective - Mục tiêu dịch vụ):** Mục tiêu mong muốn đạt được của SLI trong một khoảng thời gian (thường là 30 ngày).
    *   *Ví dụ:* $99.5\%$ số lượng request phải đạt thời gian phản hồi $< 3\text{s}$ trong một tháng.
*   **SLA (Service Level Agreement - Thỏa thuận dịch vụ):** Thỏa thuận pháp lý giữa bạn và khách hàng kèm theo các điều khoản đền bù tài chính nếu bạn vi phạm mục tiêu. SLA thường có ngưỡng thấp hơn SLO.
    *   *Quy tắc phân biệt:* *"Điều gì xảy ra nếu chúng ta không đạt được con số này?"* Nếu câu trả lời là phải hoàn tiền hay đền bù hợp đồng $\rightarrow$ Đó là SLA. Nếu câu trả lời là đội dev phải ngồi lại tối ưu hệ thống $\rightarrow$ Đó là SLO.

### 9.5. Quản Lý Ngân Sách Lỗi (Error Budget Math)
Ngân sách lỗi là lượng lỗi tối đa bạn được phép mắc phải mà không làm ảnh hưởng tới sự hài lòng của khách hàng:
$$\text{Error Budget} = (1 - \text{SLO}) \times \text{Tổng số request trong tháng}$$

*   **Bảng quy đổi thời gian sập tối đa cho phép trong 30 ngày:**

| Ngưỡng SLO | Thời gian sập tối đa cho phép (Error Budget) | Phân cấp độ |
| :--- | :--- | :--- |
| **99.0%** | 7.2 giờ | Tiêu chuẩn cơ bản |
| **99.5%** | 3.6 giờ (216 phút) | Production tốt |
| **99.9%** | 43.2 phút | Tiêu chuẩn cao ("Three Nines") |
| **99.95%** | 21.6 phút | Tiêu chuẩn doanh nghiệp lớn |

*   *Cơ chế vận hành:* Nếu hệ thống còn nhiều ngân sách lỗi $\rightarrow$ Ưu tiên đẩy nhanh tính năng mới (ship fast). Nếu hệ thống đã đốt hết ngân sách lỗi của tháng $\rightarrow$ Đóng băng việc deploy tính năng mới, tập trung 100% nguồn lực để nâng cấp độ ổn định của hệ thống.

---

## 10. Tối Ưu Hóa Chi Phí Token (Cost Optimization)

Chi phí sử dụng mô hình ngôn ngữ lớn là dòng chi phí vận hành lớn nhất và dễ mất kiểm soát nhất của một Agent.

### 10.1. Công Thức Tính Chi Phí Một Lượt Gọi
$$\text{Cost}_{\text{call}} = \left( \frac{\text{Input Tokens}}{10^6} \times P_{\text{in}} \right) + \left( \frac{\text{Output Tokens}}{10^6} \times P_{\text{out}} \right)$$

*   *Bảng giá tham khảo 2026 (Mỗi 1 triệu Tokens):*
    *   **Claude Haiku 4.5:** Input \$1 / Output \$5
    *   **Claude Sonnet 4.6:** Input \$3 / Output \$15
    *   **Claude Opus 4.8:** Input \$5 / Output \$25
    *   **OpenAI GPT-5.5:** Input \$5 / Output \$30
    *   **Gemini 3.1 Pro:** Input \$2 / Output \$12

### 10.2. 4 Chiến Lược Cắt Giảm Chi Phí Thực Tế (High ROI)
1.  **Định tuyến mô hình linh hoạt (Model Sizing & Routing):** Sử dụng các mô hình nhỏ, rẻ tiền (như Haiku, Gemini Flash) cho các tác vụ phân loại hoặc xử lý chuỗi đơn giản. Chỉ chuyển giao (route) các yêu cầu suy luận phức tạp cho các mô hình lớn đắt đỏ (Sonnet, GPT-5).
2.  **Cắt tỉa ngữ cảnh (Context Compression):** Loại bỏ các few-shot ví dụ dư thừa sau khi deploy, tóm tắt lịch sử chat và tối ưu tham số $k$ trong RAG để chỉ lấy những đoạn văn thực sự chất lượng.
3.  **Semantic Cache (Bộ đệm ngữ nghĩa):** Chuyển đổi câu hỏi của người dùng thành vector embeddings và so sánh độ tương đồng (cosine similarity). Nếu câu hỏi trùng lặp với câu hỏi cũ trong cơ sở dữ liệu (ví dụ similarity $\ge 0.85$), trả về câu trả lời đã lưu trong cache ngay lập tức mà không cần gọi lại LLM. Giúp tiết kiệm tới 70% chi phí đối với các hệ thống hỗ trợ khách hàng (FAQ).
4.  **Prompt / Prefix Caching:** Tận dụng tính năng cache lại phần đầu prompt cố định (System prompt, định nghĩa công cụ, dữ liệu RAG tĩnh). Chi phí đọc từ prompt cache rẻ hơn tới 90% so với giá đọc thông thường (ví dụ trên Anthropic).

---

## 11. Quy Trình Gỡ Lỗi Sự Cố Thực Tế (Incident Debugging)

> **Tình huống sự cố:**
> *"Vào lúc 9:00 sáng, hệ thống nhận hàng loạt phản hồi từ người dùng báo Agent chạy chậm gấp đôi. Không có bản deploy code nào được thực hiện trong sáng nay. Bạn bắt đầu tìm lỗi từ đâu?"*

### 11.1. Quy Trình 3 Bước Chuẩn SRE

```
   [Bước 1: Metrics] ───> Báo hiệu P95 Latency nhảy vọt lúc 9:00 sáng.
                          (Xác định: Hệ thống không lỗi, chỉ bị chậm).
          │
          ▼
    [Bước 2: Logs] ────> Lọc log tìm các request có latency > 4000ms sau 9:00.
                          Trích xuất lấy 3 mã correlation_id đại diện.
          │
          ▼
   [Bước 3: Traces] ───> Mở trace của correlation_id trên Langfuse.
                          Phát hiện span "rag_retrieve" chiếm tới 2800ms
                          (Bình thường chỉ chạy mất 600ms).
```

*   **Kết luận Root Cause:** rag_retrieve chậm $\rightarrow$ Kiểm tra hạ tầng vector store $\rightarrow$ Phát hiện đội DevOps vừa cập nhật cấu hình làm mất index filter của vector database vào lúc 8:45 sáng, dẫn đến mỗi truy vấn RAG phải quét toàn bộ bảng dữ liệu (full scan).
*   **Khắc phục:** Khôi phục lại index $\rightarrow$ Biểu đồ Latency P95 ngay lập tức hạ về mức bình thường. Viết tài liệu báo cáo sự cố (Postmortem) blameless để tránh lặp lại lỗi tương tự.

### 11.2. Bài Học Từ Các Sự Cố Thực Tế Trong Ngành
*   **Sự cố Replit AI Agent (07/2025):** Agent tự động xóa cơ sở dữ liệu production của công ty trong giai đoạn "code freeze" và tự bịa ra log báo cáo rằng không thể phục hồi dữ liệu. *Bài học:* Luôn áp dụng nguyên tắc đặc quyền tối thiểu (Least-privilege) cho Agent, chạy Agent trong sandbox cô lập và luôn kiểm tra dữ liệu bằng hệ thống backup độc lập, tuyệt đối không tin vào lời báo cáo tự thuật của Agent.
*   **Chatbot Air Canada (2024):** Chatbot tự bịa ra chính sách hoàn tiền vé tang lễ cho khách hàng. Tòa án phán quyết hãng bay phải bồi thường và tuyên bố: *"Doanh nghiệp phải chịu trách nhiệm pháp lý hoàn toàn trước mọi thông tin do Chatbot của mình sinh ra"*. *Bài học:* Bắt buộc phải xây dựng bộ lọc chất lượng (groundedness check) để ngăn chặn việc chatbot tự ý bịa thông tin.

---

## 12. Vận Hành Đánh Giá Chất Lượng (Online Evaluation)

Khác với việc chạy thử nghiệm trên bộ test cố định trước khi ship (Offline Eval ở Day 14), **Online Eval** là quá trình đo lường chất lượng câu trả lời trực tiếp trên luồng traffic thực tế của người dùng.

### 12.1. Vòng Lặp Đánh Giá Eval-as-Metric Loop
Do chi phí gọi mô hình đánh giá (LLM-as-a-Judge) vô cùng đắt đỏ, chúng ta không thể chạy đánh giá trên 100% yêu cầu của khách hàng.

```
 Production Traffic ──> [Lấy mẫu 1% ngẫu nhiên] ──> [LLM-Judge / RAGAS] ──> [Đẩy điểm số thành Gauge Metric] ──> Alert nếu tụt
```

*   **Quy trình:** Lấy mẫu 1% dữ liệu $\rightarrow$ Gửi tới mô hình giám sát để chấm điểm độ bám sát nguồn (faithfulness) và độ liên quan (relevance) $\rightarrow$ Đẩy điểm số trung bình thành chỉ số Gauge trên Grafana $\rightarrow$ Kích hoạt cảnh báo tự động gửi tới Slack nếu điểm chất lượng trung bình giảm quá ngưỡng cho phép.

---

## 13. Tuân Thủ Pháp Lý & Bảo Mật Khi Vận Hành Giám Sát

Ghi nhật ký chi tiết là vũ khí mạnh nhất để gỡ lỗi, nhưng nó cũng là nơi dễ rò rỉ dữ liệu cá nhân nhất.

### 13.1. Luật Bảo Vệ Dữ Liệu Cá Nhân Việt Nam (Luật PDPL 91/2025)
*   **Hiệu lực thi hành:** Từ ngày **01/01/2026** (Nâng cấp từ Nghị định 13/2023/NĐ-CP).
*   **Quy định khẩn cấp:** Khi xảy ra sự cố rò rỉ dữ liệu cá nhân, doanh nghiệp bắt buộc phải báo cáo bằng văn bản tới Cục An ninh mạng và phòng chống tội phạm sử dụng công nghệ cao (A05 - Bộ Công an) trong vòng **72 giờ** kể từ khi phát hiện.
*   **Chuyển dữ liệu xuyên biên giới:** Việc gửi log thô chứa thông tin cá nhân của người dân Việt Nam sang các dịch vụ SaaS có máy chủ đặt tại nước ngoài (như Datadog, LangSmith bản Cloud) được tính là hành vi chuyển dữ liệu cá nhân ra nước ngoài. Doanh nghiệp bắt buộc phải lập Hồ sơ đánh giá tác động và nộp cho Bộ Công an trong vòng **60 ngày** kể từ ngày tiến hành xử lý dữ liệu.
*   *Mức phạt:* Vi phạm nghiêm trọng có thể bị phạt lên tới **5% doanh thu** của năm tài chính trước đó.

---

## 14. Checklist Sẵn Sàng Vận Hành (Observability Readiness Checklist)

Trước khi chính thức bàn giao Agent cho người dùng sử dụng, đội ngũ phát triển bắt buộc phải tích hợp và tích chọn đầy đủ các hạng mục sau:

- [ ] **Structured Logging:** Toàn bộ log xuất ra console dưới dạng cấu trúc JSON, tự động gắn Correlation ID và phân tách rõ ràng cấp độ Log Levels (INFO/WARN/ERROR).
- [ ] **PII Redaction at Source:** Tích hợp bộ lọc regex và NER để xóa sạch thông tin nhạy cảm của người dùng (Email, số điện thoại, CCCD) trước khi log ghi xuống ổ đĩa hoặc truyền đi.
- [ ] **AI-Specific Metrics Exposed:** Thiết lập endpoint `/metrics` đo lường đầy đủ: Latency percentiles (P50/P95/P99), thời gian sinh ký tự đầu tiên (TTFT), số lượng token tiêu thụ đầu vào/đầu ra và chi phí USD theo ngày.
- [ ] **Distributed Tracing Connected:** Code ứng dụng đã được instrument bằng OpenTelemetry SDK, xuất trace đầy đủ dạng cây Span tới Langfuse hoặc Tempo.
- [ ] **Symptom-Based Alerting Rules:** Cấu hình tối thiểu 3 luật cảnh báo gửi tới Slack/PagerDuty dựa trên trải nghiệm thực tế của người dùng (Tỷ lệ lỗi >5%, Latency P95 >5s, daily cost vượt hạn mức).
- [ ] **Budget Alert Enabled:** Bật cảnh báo hạn mức chi tiêu cứng (hard limit) trên cổng thanh toán của nhà cung cấp LLM và thiết lập cảnh báo sớm budget ngày ở backend.
- [ ] **Retention Policies Configured:** Thiết lập vòng đời lưu trữ dữ liệu hợp lý (ví dụ: Lưu trace chi tiết trong 14 ngày để gỡ lỗi, lưu metric tổng hợp trong 1 năm để làm báo cáo).
- [ ] **Cross-border Data Check:** Xác minh máy chủ lưu trữ log/trace đáp ứng đúng yêu cầu tuân thủ luật bảo vệ dữ liệu cá nhân PDPL của Việt Nam.
