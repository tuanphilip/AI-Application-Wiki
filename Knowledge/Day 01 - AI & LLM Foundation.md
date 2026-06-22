# Day 01 - AI & LLM Foundation

Tài liệu này hệ thống hóa toàn bộ kiến thức của **Day 01: Nền tảng về AI và Mô hình Ngôn ngữ Lớn (LLM)**. Đây là nền móng để phát triển các ứng dụng Agentic AI và Multi-Agent phức tạp hơn.

---

## 1. Bức Tranh AI Hiện Đại & Phân Loại Công Nghệ

### 1.1. Hệ Phân Loại Trí Tuệ Nhân Tạo (AI Taxonomy)
Hệ sinh thái AI được tổ chức theo cấu trúc hình tháp từ rộng đến sâu:
1.  **AI (Artificial Intelligence):** Máy móc thực hiện các tác vụ thông minh bắt chước con người.
2.  **ML (Machine Learning):** Máy học từ dữ liệu để tự tối ưu thuật toán mà không cần lập trình các quy tắc cứng.
3.  **DL (Deep Learning):** Học sâu sử dụng mạng thần kinh nhân tạo nhiều tầng (Neural Networks).
4.  **FM (Foundation Models):** Các mô hình nền tảng khổng lồ được huấn luyện trên lượng dữ liệu cực lớn, có thể áp dụng đa tác vụ.
5.  **LLM (Large Language Model):** Mô hình nền tảng chuyên về xử lý ngôn ngữ — trái tim của Generative AI và Agentic AI.

### 1.2. Ba Nhóm AI Chính
Sự khác biệt về cơ chế và đầu vào/đầu ra giữa các thế hệ AI:

| Nhóm AI | Chức Năng | Đầu Vào $\rightarrow$ Đầu Ra | Ví Dụ Điển Hình |
| :--- | :--- | :--- | :--- |
| **Discriminative AI** | Phân loại dữ liệu, dự đoán nhãn | Input $\rightarrow$ Label | Spam filter, Image classifier, Fraud detection |
| **Generative AI** | Sinh nội dung mới (văn bản, ảnh, code) | Prompt $\rightarrow$ Content | ChatGPT, Claude, Midjourney, GitHub Copilot |
| **Agentic AI** | Tự động lập kế hoạch và hành động để đạt mục tiêu | Goal $\rightarrow$ Plan $\rightarrow$ Action | AI Coding Agents, Auto Customer Support, Research Agents |

*   **Xu hướng phát triển (2024 - 2026):** Sự chuyển dịch mạnh mẽ từ AI "trả lời hay" (Generative AI) sang AI "biết hành động và tạo ROI" (Agentic AI).

---

## 2. Large Language Model (LLM) — Cơ Chế Hoạt Động

### 2.1. Kiến Trúc Transformer (Decoder-Only)
Hầu hết các LLM hiện đại (GPT, Claude, Gemini, Llama) đều sử dụng kiến trúc **Decoder-Only Transformer** (đọc từ trái qua phải để dự đoán token tiếp theo).
*   **Self-Attention (Cơ chế cốt lõi):** Cho phép mô hình tự động gán trọng số (chú ý) vào các phần quan trọng của văn bản đầu vào khi xử lý mỗi token.
    *   *Ví dụ:* *"Con mèo ngồi trên bàn. Nó rất đáng yêu."* $\rightarrow$ Mô hình hiểu từ *"Nó"* có liên kết mạnh nhất với *"mèo"* chứ không phải *"bàn"*.
    *   *Công thức:* $\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$ (Trong đó $Q$: Query, $K$: Key, $V$: Value, $d_k$: Số chiều để chuẩn hóa).

### 2.2. Khái Niệm Token & Token Economy
*   **Token** là đơn vị xử lý nhỏ nhất của LLM.
    *   $1 \text{ Token} \approx 0.75$ từ tiếng Anh.
    *   $1 \text{ Token} \approx 0.5$ từ tiếng Việt (do tiếng Việt có dấu và mã hóa Unicode phức tạp $\rightarrow$ **tốn token và chi phí hơn** khi gọi API cùng một nội dung).
*   **Next-Token Prediction:** LLM thực chất là một công cụ dự đoán xác suất token tiếp theo dựa trên chuỗi token hiện tại (autoregressive).
*   **Temperature (Nhiệt độ):** Điều chỉnh độ ngẫu nhiên của đầu ra.
    *   `Temperature = 0`: Deterministic (luôn chọn token có xác suất cao nhất) $\rightarrow$ Phù hợp cho code, toán, trích xuất dữ liệu.
    *   `Temperature > 0.7`: Sáng tạo, đa dạng hóa câu trả lời.

### 2.3. Quy Trình Huấn Luyện Mô Hình
Để tạo ra một LLM hữu ích và an toàn cho người dùng, mô hình phải trải qua 3 giai đoạn:
1.  **Pre-training (Đọc Internet):** Mô hình đọc hàng nghìn tỷ token văn bản để học cấu trúc ngôn ngữ và kiến thức chung.
2.  **SFT (Supervised Fine-Tuning - Học theo ví dụ):** Mô hình được dạy cách phản hồi theo dạng hội thoại (Prompt - Response) thông qua tập dữ liệu nhãn chuẩn.
3.  **RLHF / DPO (Alignment - Căn chỉnh hành vi):** Uốn nắn mô hình theo sở thích của con người, đảm bảo tính an toàn, lịch sự và từ chối các yêu cầu nguy hại.

### 2.4. Các Giới Hạn Bẩm Sinh Của LLM
*   **Knowledge Cutoff:** Mô hình không biết thông tin mới sau thời điểm huấn luyện nếu không có tool/internet.
*   **Hallucination (Ảo giác):** LLM sinh câu trả lời sai sự thật một cách vô cùng tự tin do nó chỉ tối ưu xác suất xuất hiện của từ, không tối ưu tính đúng đắn.
*   **Context Window (Cửa sổ ngữ cảnh):** Giới hạn dung lượng token xử lý trong một lượt gọi (Input + Output). Lượng thông tin ở giữa context dễ bị ngó lơ (**Lost in the Middle**).

---

## 3. Token Economy & Lựa Chọn Mô Hình

### 3.1. Mô Hình Giá API (Pricing Model)
$$\text{Chi Phí Gọi API} = (\text{Số lượng Input Tokens} \times \text{Đơn giá Input}) + (\text{Số lượng Output Tokens} \times \text{Đơn giá Output})$$
*   *Lưu ý:* Giá trị Output Tokens luôn đắt hơn từ **3x - 5x** so với Input Tokens.

### 3.2. Bảng So Sánh Các Mô Hình Phổ Biến (Tham khảo 2026)

| Model | Input ($/1M) | Output ($/1M) | Context Window | Phù Hợp Cho Tác Vụ |
| :--- | :--- | :--- | :--- | :--- |
| **Claude 4.6 Opus** | \$5.0 | \$25.0 | 1M Tokens | Suy luận chuyên sâu, logic toán/code cực khó |
| **Claude 4 Sonnet** | \$3.0 | \$15.0 | 1M Tokens | Lựa chọn cân bằng, đa mục đích, thông minh |
| **Claude 4.5 Haiku** | \$0.8 | \$4.0 | 200K Tokens | Tác vụ nhanh, giá rẻ, trích xuất, phân loại |
| **GPT-4o** | \$5.0 | \$20.0 | 128K Tokens | Đa phương thức (Multimodal), tích hợp hệ sinh thái |
| **Gemini 2.5 Pro** | \$1.25 | \$10.0 | 1M - 2M Tokens | Tài liệu siêu dài, phân tích đa file |
| **Llama 4 Scout** | Free | Free | 1M Tokens | Mô hình mã nguồn mở (Self-host), bảo mật dữ liệu |

### 3.3. Framework Chọn Mô Hình Nhanh
*   **Ưu tiên Chi Phí / Tốc Độ (Latency):** Chọn *Claude Haiku, Gemini Flash hoặc các mô hình nhỏ/open-source*. Phù hợp cho: FAQ, phân loại, trích xuất thực thể, xử lý hàng loạt (batch jobs).
*   **Ưu tiên Chất Lượng / Suy Luận (Quality/Reasoning):** Chọn *Claude Sonnet/Opus, GPT-4o hoặc Gemini Pro*. Phù hợp cho: Phân tích nhiều bước, lập kế hoạch, sinh code phức tạp, tài liệu dài có độ phức tạp cao.

---

## 4. Lập Trình Gọi API & Streaming

### 4.1. So Sánh Cú Pháp 3 Providers Lớn

#### Cú Pháp Anthropic (Claude)
```python
import anthropic
client = anthropic.Anthropic()

resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024, # Tham số bắt buộc
    system="Bạn là trợ lý tài chính.",
    messages=[
        {"role": "user", "content": "Tóm tắt báo cáo tài chính này..."}
    ]
)
print(resp.content[0].text)
```

#### Cú Pháp OpenAI (GPT)
```python
from openai import OpenAI
client = OpenAI()

resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Hello"}
    ]
)
print(resp.choices[0].message.content)
```

#### Cú Pháp Google Gemini
```python
import google.generativeai as genai
model = genai.GenerativeModel("gemini-2.5-flash")

resp = model.generate_content("Hello")
print(resp.text)
```

### 4.2. Kỹ Thuật Streaming (Phản hồi thời gian thực)
Hữu ích để giảm **Time-to-First-Token (TTFT)** giúp cải thiện trải nghiệm người dùng bằng cách in ra câu trả lời ngay khi model vừa sinh ra token.
```python
import anthropic
client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Viết một bài thơ lục bát về lập trình."}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

# Lấy thông số token thực tế sau khi luồng kết thúc
final_message = stream.get_final_message()
print(f"\nUsage: {final_message.usage.input_tokens} in / {final_message.usage.output_tokens} out")
```

---

## 5. Tư Duy Vibe Coding
Lập trình trong kỷ nguyên AI đòi hỏi chuyển dịch tư duy từ **"mình tự gõ code"** sang **"mình làm đạo diễn AI viết code"**:
1.  **Intent-driven:** Mô tả mục tiêu, đầu ra mong đợi và tiêu chí kiểm thử thật rõ ràng cho AI.
2.  **Context-first:** Cung cấp đầy đủ file code liên quan, tài liệu thư viện, và logs lỗi để AI có ngữ cảnh tốt nhất.
3.  **Human review:** AI đảm nhận tốc độ viết code, con người chịu trách nhiệm kiểm tra logic và chốt chất lượng đầu ra.
