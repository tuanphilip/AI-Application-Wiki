# Day 14 - AI Evaluation & Benchmarking

> Tài liệu được tổng hợp từ slide bài giảng "AI Evaluation & Benchmarking" (AICB-P1). Đo lường chất lượng AI một cách khoa học.

---

## 1. Evaluation Fundamentals & Eval-Driven Development

### 1.1. Evaluation là phương pháp khoa học cho AI
*   **Scientific Method**: Hypothesis ("Agent tốt hơn") $\rightarrow$ Experiment (Chạy benchmark) $\rightarrow$ Measure (Score có số) $\rightarrow$ Conclude (Evidence-based).
*   **Không đo = không cải thiện**. Mỗi giả thuyết về chất lượng phải đi qua một experiment đo được, rồi kết luận bằng số — không phải bằng cảm giác.

### 1.2. Eval là Unit Test thế hệ mới
*   Vì sao không dùng unit test cũ?
    *   Agent là **probabilistic** + **multi-step**: cùng input, output khác nhau mỗi lần.
    *   Không có một chuỗi đúng duy nhất ($\rightarrow$ `assert output == expected` sẽ vỡ ngay).
*   Thay vào đó, ta chấm **outcome/behavior**, không chấm exact-match.
*   **Eval-Driven Development (EDD)**: Viết eval *trước* để định nghĩa "đúng nghĩa là gì", rồi mới optimize prompt/model.

### 1.3. Eval Maturity Ladder (Thang đo trưởng thành)
*   **L0 — Vibe-checks**: Đọc tay vài output, "trông ổn" (chỉ đáng tin khi builder là domain power user).
*   **L1 — Error analysis + offline evals**: Code-based + LLM-as-judge. Bắt đầu bằng việc review tay 20-50 output thực tế.
*   **L2 — EDD gating releases**: Eval pass thì mới được deploy.

### 1.4. Bốn trục vận hành của Eval
1.  **Dataset cố định**: Chạy trong dev + CI/CD. Đóng vai trò là *release gate* và *regression tracking*.
2.  **Live traffic**: Monitoring liên tục (dùng Observability/Traces).
3.  **Inline**: Chặn trước khi trả response cho user (Guardrails).
4.  **Expert review**: Sampled outputs để human đánh giá, dùng để calibrate 3 trục trên.

### 1.5. Outcome vs. Trajectory
*   **Outcome**: Trạng thái cuối của môi trường. Ví dụ: Có một row đặt chỗ trong SQL DB hay không.
*   **Trajectory**: Toàn bộ vết step-by-step, bao gồm mọi tool call, reasoning, kết quả trung gian.
$\rightarrow$ **Ưu tiên grade the outcome; read the trajectory để hiểu vì sao**.

---

## 2. Bài Học Thực Tế Khi Eval Thất Bại

Mỗi sự cố chatbot nổi tiếng đều là một eval mà ai đó đã không viết:
*   **Air Canada (2024)**: Bịa chính sách hoàn tiền tang lễ (Hallucination) $\rightarrow$ Đáng lẽ bắt được bằng **RAGAS Faithfulness** + golden set expert.
*   **DPD (2024)**: Update hệ thống làm bot chửi thề $\rightarrow$ Đáng lẽ bắt được bằng **Release Gate** / regression eval.
*   **Chevrolet (2023)**: Prompt injection dụ bán xe $1 $\rightarrow$ Đáng lẽ bắt được bằng **Adversarial test set**.
*   **NYC MyCity (2023-24)**: Khuyên người dân phạm luật $\rightarrow$ Đáng lẽ bắt được bằng **Domain-expert eval set**.
*   **GPT-4o (2025)**: Sycophancy (quá nịnh người dùng) $\rightarrow$ Đáng lẽ bắt được bằng **Behavioral / multi-judge eval** cho launch-blocking.
*   **Luật sư hallucination (Mata v. Avianca)**: Trích dẫn án lệ giả $\rightarrow$ Đáng lẽ bắt được bằng **Factuality / citation-grounding eval**.
*   **Sakana "AI Scientist" (2024)**: Agent tự sửa code để vượt harness (Eval-gaming) $\rightarrow$ Cần eval **độc lập**, khó game.

---

## 3. Metrics Cho AI Agent

Chọn metric phải gắn với use case và business outcome.

### 3.1. Phân loại Metric
1.  **North Star**: Một metric duy nhất phản ánh giá trị cốt lõi cho user/business (VD: Task success rate).
2.  **Guardrail**: Những chỉ số không được tệ đi (VD: Faithfulness, p95 latency, cost/req). Vi phạm $\rightarrow$ block.
3.  **Diagnostic**: Dùng để debug khi North Star tụt (VD: Context recall, tool-call accuracy).
> Tối ưu North Star mà không phá Guardrail. Đừng tối ưu Diagnostic metric vì sẽ rơi vào định luật Goodhart.

### 3.2. Vì sao String-Match thất bại
Các metric từ vựng (Exact Match, BLEU, ROUGE-L, WER) rẻ và deterministic nhưng **không hiểu nghĩa**. Dễ mắc bẫy phủ định ("Nên" vs "Không nên" trùng 9/10 token nhưng nghĩa ngược nhau).

### 3.3. Embedding Similarity & Contradiction Trap
*   So khớp ngữ nghĩa bằng cosine của embedding bắt được paraphrase tốt hơn.
*   **Contradiction Trap**: Hai câu ngược nghĩa (vd: "hoàn tiền" vs "không hoàn tiền") vẫn có cosine > 0.90 vì cùng chủ đề/từ vựng.
$\rightarrow$ Cần tiến tới **LLM-as-Judge** (và NLI) để chấm đúng/sai thực sự.

### 3.4. 4 Nhóm Metrics Chính
1.  **Task Completion**: Binary (đúng/sai ở state cuối), Partial credit (đúng bao nhiêu phần), Steps completed.
2.  **Answer Quality**: Accuracy, Completeness, Coherence, Citation accuracy.
3.  **RAG-Specific**: Faithfulness, Answer relevancy, Context recall, Context precision.
4.  **Business**: User satisfaction, Time saved, Cost per interaction, Adoption rate.

---

## 4. Retrieval Evaluation: Hit Rate, MRR, NDCG

Trước khi chấm generation, hãy chấm retrieval. **Retrieval là trần (ceiling) của chất lượng RAG** (Nếu retriever bỏ lỡ tài liệu đúng, không generator nào cứu được).

*   **Order-UNAWARE (Chỉ quan tâm tập top-k, đảo thứ tự không đổi điểm)**:
    *   **Hit Rate@k**: Có ít nhất 1 doc đúng trong top-k không? (Metric chính cho lab).
    *   **Recall@k**: Lấy được bao nhiêu % doc đúng trong corpus?
    *   **Precision@k**: Top-k sạch đến đâu?
*   **Order-AWARE (Vị trí có trọng số, doc đúng nằm thấp bị phạt)**:
    *   **MRR (Mean Reciprocal Rank)**: Doc đúng *đầu tiên* nằm ở hạng nào? ($1/rank$).
    *   **NDCG@k**: Có graded relevance, phạt sai thứ hạng bằng hàm log.

$\rightarrow$ **Ground Truth Miễn Phí**: Khi dùng Synthetic Data Generation (SDG) để sinh câu hỏi từ mỗi chunk, ta biết sẵn chunk id đúng. Có `gold_doc_ids` thì tính Hit Rate & MRR rẻ và deterministic, **không cần LLM**.

---

## 5. Benchmark Design & Golden Datasets

**Chất lượng eval không bao giờ cao hơn chất lượng golden set.**

### 5.1. Golden Dataset gồm những gì?
*   Question-answer pairs được expert (SME) review.
*   Cover tất cả use cases chính.
*   Difficulty levels: easy / medium / hard.
*   Edge cases và adversarial inputs.
*   Mỗi item gắn metadata (để tính điểm theo slice).

### 5.2. Edge Cases và Stratified Sampling
*   Aggregate score (điểm trung bình) có thể đẹp trong khi một nhóm nhỏ (slice) hỏng nặng.
*   Cần stratify (phân tầng) để báo cáo điểm: Proportional cho mỗi use case, đủ sample cho các độ khó, cân bằng happy path vs edge case (ambiguous, out-of-scope, adversarial, multilingual, long context).

### 5.3. Sizing (Quy mô)
*   **50 cases** là mức sàn cho lab, chỉ đủ để bắt lỗi thô. Margin of error (với p=0.5) lên tới $\pm 14\%$.
*   Để phát hiện chênh lệch 3% với 80% power, cần tới $\sim 969$ paired cases. Đừng vội kết luận agent A tốt hơn agent B 3 điểm nếu chỉ test trên 50 cases (đó là noise).

### 5.4. Benchmark phải tiến hóa (Evolve) & Versioning
*   Mỗi sprint: Thêm mọi failure case mới gặp vào set (thành regression test vĩnh viễn).
*   Giữ một **held-out / private split** không bao giờ public để chống data contamination.
*   Version benchmark bằng Git để kiểm soát sự thay đổi.

---

## 6. Synthetic Data Generation (SDG)

LLM sinh câu hỏi (generation) rất nhanh và dễ, nhưng **filtering** (loại bỏ câu quá dễ, câu không trả lời được, câu lệch distribution) mới là phần khó và làm nên sản phẩm thật sự.

*   SDG **không thay thế** golden dataset. Nó tạo ra **silver candidates** để con người review (thành gold) nhanh hơn.
*   **Cạm bẫy (Pitfalls)**:
    *   Câu hỏi sinh ra máy móc, đột ngột, dễ drift.
    *   Mang bias của model sinh ra (lối hỏi, chủ đề, độ khó nghiêng theo thói quen của LLM).
    *   Model vừa sinh test set vừa làm judge sẽ thổi phồng điểm.
*   **Quy tắc vàng**: `Generator != Validator != Judge`. Nên dùng Cross-model validation (VD: GPT sinh, Mistral kiểm tra).

---

## 7. Bức Tranh Benchmark Công Khai 2026

*   Các benchmark kinh điển (MMLU, HumanEval, GSM8K) đã **bão hòa (maxed out)**. Các frontier model chen chúc sát trần (điểm >90%), chênh lệch chỉ nằm trong vùng nhiễu (noise) hoặc do data contamination.
*   **Thế hệ benchmark mới**: Câu hỏi reasoning-heavy khó hơn, nhiều options (10 thay vì 4), year-stamped (chống memorization).
*   **Leaderboard Illusion**: Điểm Elo/LMArena có thể bị "mua" bằng format/length. Một model tối ưu cho arena chưa chắc đã tối ưu cho task của bạn.
$\rightarrow$ **Kết luận**: Bạn phải tự xây **eval task-specific, contamination-resistant**.

---

## 8. Agentic & Trajectory Evaluation

Với agent nhiều bước, chấm điểm câu trả lời cuối (Outcome) là chưa đủ, phải chấm cả **Trajectory** (chuỗi tool call và bước suy luận).
*   **Final-outcome eval**: Trạng thái cuối của môi trường có khớp target không? (Đo cái user quan tâm).
*   **Trajectory eval**: Đúng chuỗi tool call / reasoning step không? (Bắt đường đi lãng phí, sai, lặp, gọi thừa).

**Tool-Call Correctness (BFCL Benchmark)**:
*   Chấm bằng AST evaluation: Đúng function chưa? Parallel/serial đúng chưa? Kiểu argument đúng chưa?

**Dual-Control Environment (tau2-bench)**:
*   Cả agent và một simulated user cùng tương tác với môi trường. Chấm bằng state comparison (so sánh DB cuối).

**Reliability**:
*   Một lần chạy đúng là không đủ (Agent là probabilistic). Cần quan tâm pass@k và consensus qua nhiều lần chạy để đánh giá độ ổn định.

---

## 9. LLM-as-Judge & Multi-Judge Consensus

**LLM-as-Judge**: Truyền `Question + Agent Answer + Reference + Rubric` vào LLM Judge để lấy `Score (1-5) + Rationale` (Giải thích).

### 9.1. Rubric Design Tốt
*   Tiêu chí cụ thể, rõ ràng cho từng mức điểm.
*   Thêm quy định: `"length is NOT a factor; concise is fine"`.
*   Bắt judge suy luận (`reasoning`) **trước**, cho điểm (`score`) **sau**.
*   Tránh chấm Pointwise (1 điểm cô lập), nên ưu tiên Pairwise (hỏi judge A hay B tốt hơn) nếu cần so sánh.

### 9.2. Định Lượng Bias Trong LLM-as-Judge
1.  **Position Bias**: Ưu tiên answer đứng trước (khắc phục: Swap order).
2.  **Verbosity Bias**: Chọn answer dài hơn (khắc phục: Rubric chặn length).
3.  **Self-Enhancement Bias**: Thiên vị cùng họ model (khắc phục: Dùng judge khác họ, multi-judge).

### 9.3. 5 Kỹ Thuật De-bias Bắt Buộc
1.  **Swap order**: Chạy cả 2 thứ tự rồi average.
2.  **Force rationale + CoT**: Bắt giải thích trước khi cho điểm.
3.  **Rubric + multi-dimension**: Chấm từng tiêu chí riêng rẽ.
4.  **Calibrate vs human**: So sánh judge với expert trên tập có nhãn (dùng chỉ số Cohen's $\kappa$).
5.  **Multi-judge jury**: Dùng nhiều model khác họ kết hợp (Consensus).

### 9.4. Consensus Engine (PoLL)
*   Sử dụng một nhóm (panel) các model nhỏ khác họ. Chi phí rẻ hơn và bias triệt tiêu lẫn nhau.
*   Tính **Agreement rate** (độ đồng thuận) bằng majority vote hoặc average.
*   Khi có bất đồng (Conflict), dùng 1 Frontier Model đắt tiền để phân xử (tie-break).

---

## 10. RAGAS Framework & Metrics

RAGAS cung cấp metrics chuẩn để đánh giá RAG pipeline và Agent.
*   **Lưu ý API v0.4+**: Đã chuyển sang async (`ascore`), data struct mới (`EvaluationDataset`, `SingleTurnSample`).

**Core 4 Metrics trong RAGAS**:
1.  **Faithfulness**: Supported claims / total claims. Đo lường hallucination (bịa thông tin). Tính bằng NLI (Natural Language Inference).
2.  **Answer Relevancy**: Answer có trả lời đúng/trúng câu hỏi không, hay lạc đề.
3.  **Context Precision**: Order-aware metric đánh giá việc retriever xếp hạng chunk relevant lên cao hay không (thấp = nhiễu).
4.  **Context Recall**: Claim-based metric đánh giá xem có lấy đủ evidence so với reference answer không (thấp = thiếu).

*Nguyên tắc debug*:
*   Generation kém $\rightarrow$ Check Faithfulness + Relevancy.
*   Retrieval kém $\rightarrow$ Check Context Recall + Context Precision.

---

## 11. Statistical Rigor in Eval

*   **Paired Comparison**: So sánh 2 agent, phải chạy CẢ HAI trên **cùng test cases** và phân tích hiệu số per-question. Giúp loại bỏ độ khó của câu hỏi, giảm variance, tăng statistical power.
*   **Significance cho Pass/Fail**: Dùng **McNemar's Test** (hoặc exact binomial test) chỉ trên các cặp kết quả bất đồng (discordant pairs).
*   **Confidence Interval (CI)**: Không dùng Wald/CLT cho n nhỏ (<100). Dùng **Wilson score** hoặc Bayesian Beta-Bernoulli cho pass-rate.
*   **Multiple Seeds**: Chạy mỗi cấu hình nhiều seed (vì Temperature=0 vẫn có độ ngẫu nhiên), report Mean $\pm$ Std.

---

## 12. Eval Economics & Async

*   Sử dụng LLM-as-judge chuyển chi phí từ lao động (Dollars) sang token bill (Cents).
*   Tối ưu chi phí bằng cách **cắt verbosity** (giảm output token), dùng short rationale.
*   Tận dụng **Prompt Caching** (cho rubric/few-shots) và **Batch API** (giảm 50% chi phí).
*   Sử dụng **Async runner** (vd: `RunConfig(max_workers=16)` trong RAGAS) để chạy song song, rút ngắn thời gian eval từ vài giờ xuống vài phút.
*   **Tiered Sampling**: Dùng judge rẻ tiền (mini/flash) trên tập nhỏ để "smoke test" trước, bắt trend lỗi thô. Chỉ dùng frontier judge trên full suite cho Release Gate.

---

## 13. Regression Release Gates & Eval CI/CD

*   **Eval Gate**: Đóng vai trò như Unit Test cho agent phi định. Đặt trong CI/CD (vd: Promptfoo, LangSmith).
*   **Cơ chế**: Bất kỳ prompt/agent change nào cũng phải qua Eval Gate. Pass thì deploy, Fail (chất lượng tụt, metrics giảm) thì block merge.
*   **Composite Score**: Quyết định dựa trên sự kết hợp giữa **Quality + Cost + Latency** (vd: TTFT < 1s, TPS > 10 tok/s), không chỉ tối ưu accuracy đơn lẻ.
*   Đánh giá sự thay đổi (Delta vs Baseline) trên từng phân phối (Distribution/Category) thay vì chỉ nhìn điểm trung bình tổng.
*   **Judge Drift**: Khi judge model thay đổi, điểm sẽ đổi (false regression). Khắc phục bằng cách Pin judge model + temperature, version rubric file trong Git.
*   **Rollout an toàn**: Shadow (mirror traffic, zero blast radius) $\rightarrow$ Canary (1% $\rightarrow$ 10% $\rightarrow$ 100%) $\rightarrow$ Rollback tự động khi thủng guardrail.

---

## 14. Failure Analysis & Continuous Improvement

Evaluation cho biết *điểm số*, **Failure Analysis** cho biết *tại sao điểm thấp và fix ở đâu*. Bắt đầu từ error analysis thay vì infrastructure.

### 14.1. Failure Taxonomy (Phân Loại Lỗi)
*   **Wrong Answer**: Retrieval miss, chunking cắt mất ngữ cảnh.
*   **Hallucination**: Faithfulness guardrail yếu, prompt không ép cite.
*   **Tool Failure**: API down, sai params, sai schema.
*   **Refusal**: Guardrails quá chặt.
*   **Slow**: Model quá lớn, context quá dài.
*   **Stale Data**: Ingestion không re-index định kỳ.

### 14.2. Kỹ thuật Phân Tích
*   **5 Whys**: Truy vết liên tục để tìm ra Root Cause (Ingestion $\rightarrow$ Chunking $\rightarrow$ Retrieval $\rightarrow$ Prompting). Nếu lỗi ở Ingestion mà đi sửa Prompt thì vô ích.
*   **Failure Clustering**:
    *   Gom failures theo type $\rightarrow$ stage root-cause $\rightarrow$ Đếm và fix cụm lớn nhất trước (ROI cao nhất).
    *   Có thể tự động hóa bằng cách Embed mọi query fail $\rightarrow$ **K-Means** $\rightarrow$ Rút ra nguyên nhân chung.
*   **Continuous Improvement Loop**:
    Mỗi failure đã xác nhận $\rightarrow$ Thêm vào Golden Set $\rightarrow$ Trở thành regression test vĩnh viễn (đảm bảo bug không tái xuất).
