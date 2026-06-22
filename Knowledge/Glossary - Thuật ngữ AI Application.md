# Glossary - Thuật Ngữ AI Application

Tài liệu này tổng hợp toàn bộ các thuật ngữ cốt lõi trong ngành phát triển ứng dụng trí tuệ nhân tạo (**AI Application / Agentic Systems**) dựa trên nội dung khóa học và cập nhật kiến thức công nghệ hiện đại. Các thuật ngữ được phân nhóm theo cấu trúc hệ thống để người đọc dễ dàng tra cứu.

---

## 1. Kiến Trúc Mô Hình & Nền Tảng (LLM Foundations)

*   **Transformer:** Kiến trúc mạng thần kinh nhân tạo (Neural Network) mang tính cách mạng được giới thiệu vào năm 2017 (Vaswani et al.), là nền tảng của mọi mô hình ngôn ngữ lớn (LLM) hiện đại.
*   **Decoder-Only:** Kiến trúc Transformer chỉ sử dụng bộ giải mã, hoạt động theo nguyên lý đọc từ trái sang phải để dự đoán từ tiếp theo. Đây là kiến trúc của các mô hình sinh văn bản phổ biến hiện nay như GPT, Claude, Gemini, Llama.
*   **Self-Attention (Tự chú ý):** Cơ chế cốt lõi của Transformer cho phép mô hình gán các trọng số chú ý khác nhau vào các từ trong câu đầu vào để hiểu mối quan hệ ngữ cảnh giữa chúng.
*   **Token:** Đơn vị xử lý cơ bản nhất của LLM. Văn bản đầu vào được cắt nhỏ thành các token (subwords) trước khi nạp vào mô hình. $1 \text{ Token} \approx 0.75$ từ tiếng Anh hoặc $0.5$ từ tiếng Việt.
*   **Tokenization:** Quá trình chuyển đổi văn bản thô thành một chuỗi các Token.
*   **Next-Token Prediction:** Nguyên lý hoạt động của LLM, dự đoán token tiếp theo có xác suất xuất hiện cao nhất dựa trên chuỗi token ngữ cảnh hiện tại.
*   **Temperature (Nhiệt độ):** Tham số điều chỉnh độ ngẫu nhiên của kết quả đầu ra của LLM. Giá trị bằng $0$ cho ra kết quả ổn định và cố định (deterministic); giá trị $> 0.7$ cho ra kết quả sáng tạo và ngẫu nhiên hơn.
*   **Pre-training (Tiền huấn luyện):** Giai đoạn mô hình tự học cấu trúc ngôn ngữ và kiến thức chung bằng cách đọc lượng dữ liệu khổng lồ từ Internet. Đầu ra là mô hình nền tảng thô (Base Model).
*   **SFT (Supervised Fine-Tuning):** Giai đoạn huấn luyện có giám sát, dạy mô hình cách trả lời theo dạng hội thoại dựa trên các cặp dữ liệu mẫu chuẩn (Prompt - Response).
*   **RLHF / DPO (Alignment):** Phương pháp căn chỉnh mô hình theo sở thích của con người để đảm bảo tính an toàn, trung thực và hữu ích.
*   **Knowledge Cutoff:** Giới hạn thời gian của dữ liệu huấn luyện. LLM sẽ không biết bất kỳ sự kiện nào xảy ra sau thời điểm này nếu không có công cụ tra cứu ngoài.
*   **Hallucination (Ảo giác):** Hiện tượng LLM tạo ra các thông tin sai lệch, không có thật nhưng trình bày một cách vô cùng tự tin và mạch lạc.
*   **Context Window (Cửa sổ ngữ cảnh):** Giới hạn số lượng token tối đa mô hình có thể xử lý trong một lượt gọi API (bao gồm cả Input và Output).
*   **Time-to-First-Token (TTFT):** Thời gian tính từ lúc gửi request API cho đến khi nhận được token đầu tiên trả về. Đây là chỉ số quan trọng đo lường độ trễ trải nghiệm người dùng.
*   **Streaming:** Cơ chế truyền dữ liệu theo từng mảnh (chunk) nhỏ liên tục từ server về client, giúp hiển thị câu trả lời ngay khi mô hình sinh ra mà không cần đợi hoàn tất cả câu.

---

## 2. Quản Lý Sản Phẩm AI & Lập Hồ Sơ Bài Toán (AI Product Scoping)

*   **Problem Scoping:** Giai đoạn đầu tiên của dự án AI nhằm làm rõ đối tượng sử dụng, nỗi đau, quy trình hiện tại và thiết lập ranh giới vận hành.
*   **Actor / Operator:** Người hoặc vai trò trực tiếp vận hành quy trình công việc hàng ngày cần AI hỗ trợ.
*   **Current Workflow:** Quy trình xử lý các bước công việc hiện tại của con người khi chưa tích hợp AI.
*   **Bottleneck (Điểm nghẽn):** Bước gây chậm trễ, tốn thời gian hoặc dễ phát sinh sai sót nhất trong quy trình hiện tại.
*   **Failure Cost (Chi phí tổn thất):** Định lượng thiệt hại về tài chính, thời gian hoặc SLA khi xảy ra lỗi trong quy trình.
*   **Success Metric:** Chỉ số và ngưỡng cụ thể để nghiệm thu dự án AI (ví dụ: rút ngắn thời gian xử lý xuống dưới 2 phút).
*   **Operational Boundary (Ranh giới vận hành):** Giới hạn quyền tự trị của AI, phân định những việc AI được làm trực tiếp và điểm bắt buộc con người phải kiểm duyệt (HITL).
*   **AI-Fit Matrix (Ma trận giải pháp):** Bản đồ phân loại bài toán dựa trên Độ mơ hồ của tác vụ và Độ phức tạp vận hành để chọn kiến trúc phù hợp (Rule, Workflow, RAG, Agent).
*   **Gate Criteria (Tiêu chuẩn cổng duyệt):** Các mốc đánh giá chất lượng bắt buộc phải đạt được trước khi dự án AI được chuyển sang giai đoạn phát triển tiếp theo.
*   **Anti-pattern:** Các mẫu thiết kế hoặc cách làm việc sai lầm, phản tác dụng gây lãng phí nguồn lực (ví dụ: chạy theo trend công nghệ trước khi làm rõ bài toán).
*   **Uncertainty (Sự bất định):** Đặc tính cốt lõi của sản phẩm AI, biểu hiện qua việc đầu vào bẩn, đầu ra biến thiên và quá trình xử lý suy luận không cố định.
*   **Production Drift (Sự trôi lệch):** Hiện tượng hành vi của Agent thay đổi ngoài ý muốn khi chạy trên môi trường thực tế (bao gồm trôi lệch do cập nhật mô hình, dữ liệu thay đổi, cách hỏi của người dùng thay đổi).
*   **False Positive (Báo nhầm / Vu oan):** Lỗi hệ thống nhận diện sai một trạng thái bình thường thành bất thường (ví dụ: lọc email quan trọng vào thư mục spam).
*   **False Negative (Bỏ sót / Lọt lưới):** Lỗi hệ thống bỏ qua một trạng thái bất thường nguy hại (ví dụ: email spam lọt vào inbox chính).
*   **Augment (Hỗ trợ):** Mô hình thiết kế AI đóng vai trò đồng hành, gợi ý và soạn nháp để con người ra quyết định.
*   **Automate (Tự động hóa):** Mô hình thiết kế AI tự động thực thi hành động mà không cần sự can thiệp của con người.

---

## 3. Kỹ Nghệ Prompt & Quản Lý Ngữ Cảnh (Prompt & Context Engineering)

*   **Context Packet (Gói ngữ cảnh):** Gói thông tin tổng hợp được ứng dụng lắp ráp trước khi gọi mô hình, gồm Prompt, thông tin người dùng, lịch sử chat, tài liệu RAG và định nghĩa công cụ.
*   **System Prompt:** Chỉ thị hệ thống có mức độ ưu tiên cao nhất, định hình vai trò, nhiệm vụ và các ràng buộc hành vi của Agent.
*   **Prompt Template:** Bản mẫu Prompt được viết trong code chứa các biến số để điền động dữ liệu trước khi chạy (ví dụ: `Xin chào {user_name}`).
*   **Chat Template:** Cấu trúc chuẩn hóa của các nhà cung cấp mô hình để chuyển đổi mảng hội thoại (`system`, `user`, `assistant`, `tool`) thành chuỗi token đầu vào của mô hình.
*   **Delimiters (Ký tự phân tách):** Các thẻ XML hoặc ký hiệu cấu trúc (như `---`, `"""`, `<user_query>`) dùng để ngăn cách rõ ràng giữa chỉ thị của hệ thống và dữ liệu ngoài được cung cấp.
*   **Ask-if-missing:** Quy tắc thiết kế bắt buộc Agent phải hỏi lại người dùng khi thiếu các thông tin cốt lõi (blocking info) thay vì tự ý giả định.
*   **Lost in the Middle:** Hiện tượng mô hình chú ý nhiều vào thông tin ở đầu và cuối prompt mà ngó lơ dữ liệu nằm ở giữa context window.
*   **WSCI (Write - Select - Compress - Isolate):** Bộ chiến lược quản lý ngữ cảnh: Ghi bộ nhớ ra ngoài (Write) $\rightarrow$ Chỉ chọn dữ liệu liên quan (Select) $\rightarrow$ Rút gọn thông tin (Compress) $\rightarrow$ Phân tách ranh giới tin cậy (Isolate).
*   **Untrusted Context (Ngữ cảnh không tin cậy):** Mọi dữ liệu lấy từ bên ngoài (website, email, file tải lên) vốn có khả năng chứa mã độc tiêm lệnh, không được phép coi là chỉ thị hệ thống.
*   **Scaffolding Ladder (Thang nâng cấp):** Lộ trình tăng độ phức tạp của kỹ thuật Prompt: Zero-shot $\rightarrow$ One-shot $\rightarrow$ Few-shot $\rightarrow$ Chain-of-Thought (CoT) $\rightarrow$ Tree-of-Thought (ToT).
*   **Few-shot:** Cung cấp một vài ví dụ mẫu trong prompt để mô hình học theo định dạng và logic xử lý của các trường hợp biên.
*   **Chain-of-Thought (CoT):** Kỹ thuật prompt bắt buộc mô hình viết ra các bước suy luận trung gian trước khi đưa ra kết quả cuối cùng để tăng độ chính xác của logic.
*   **Tree-of-Thought (ToT):** Kỹ thuật prompt cho phép mô hình tự tạo nhiều hướng suy luận khác nhau như các nhánh cây, tự đánh giá và chọn nhánh tốt nhất.
*   **Tool Discovery:** Cơ chế cho phép Agent tự đọc mô tả (schema) của các công cụ có sẵn để chủ động chọn công cụ phù hợp với nhiệm vụ.
*   **Tool Access Control:** Giới hạn danh sách các công cụ Agent được phép nhìn thấy và gọi dựa trên vai trò hoặc quyền của người dùng hiện tại.
*   **Risk Ladder (Thang rủi ro):** Phân loại rủi ro của công cụ từ Đọc thông tin công khai (rủi ro thấp) $\rightarrow$ Tạo bản nháp $\rightarrow$ Giữ chỗ tạm thời $\rightarrow$ Giao dịch/Hủy dịch vụ (rủi ro cao).
*   **Enforce Approval:** Cơ chế bắt buộc tạm dừng hệ thống ở tầng code để chờ con người bấm xác nhận trước khi Agent thực thi các công cụ có tính thay đổi trạng thái (write action) hoặc rủi ro cao.
*   **Tiny Eval:** Quy trình chạy kiểm thử nhanh từ 3-8 test cases chuẩn mỗi khi sửa đổi prompt để phát hiện sớm các lỗi cơ bản.
*   **Prompt Eval vs. Agent Eval:** Đánh giá chất lượng của một câu trả lời đơn lẻ (Prompt Eval) so với đánh giá toàn bộ hành trình gọi tool và ra quyết định của Agent (Agent Eval).

---

## 4. Kiến Trúc RAG & Tầng Dữ Liệu (RAG & Data Foundations)

*   **RAG (Retrieval-Augmented Generation):** Kỹ thuật kết hợp tìm kiếm thông tin và tạo sinh văn bản để trả lời dựa trên tài liệu ngoài.
*   **Ingestion Pipeline:** Chuỗi tự động hóa thu thập tài liệu thô, làm sạch, cắt nhỏ, tạo vector và lưu trữ vào Vector DB.
*   **Chunking:** Kỹ thuật chia một tài liệu lớn thành các đoạn văn ngắn (chunks) có kích thước phù hợp để tìm kiếm chính xác và tiết kiệm token.
*   **Chunk Overlap:** Phần nội dung lặp lại giữa hai chunk liên tiếp để giữ nguyên vẹn ngữ cảnh ở ranh giới cắt.
*   **Cosine Similarity:** Metric tính toán độ gần nhau của nghĩa giữa hai vector trong không gian toán học.
*   **Metadata:** Dữ liệu đặc tả đi kèm với chunk văn bản (như tên file, ngày tạo, chuyên mục, quyền truy cập).
*   **Metadata Filtering:** Quá trình áp các điều kiện lọc thuộc tính (ví dụ: `date >= 2025-01-01`) trước khi thực hiện tìm kiếm vector để loại bỏ nhiễu.
*   **Hybrid Search:** Kết hợp song song tìm kiếm theo nghĩa (Vector Search) và tìm kiếm theo từ khóa chính xác (Lexical Search/BM25).
*   **Reranking:** Sử dụng mô hình chấm điểm (Cross-Encoder) để sắp xếp lại danh sách kết quả tìm kiếm, đưa những chunk có giá trị trả lời cao nhất lên đầu.
*   **Hit Rate:** Chỉ số đo lường tỷ lệ các lượt truy vấn tìm thấy ít nhất một chunk chứa thông tin đúng trong Top-K kết quả.
*   **Mean Reciprocal Rank (MRR):** Chỉ số đánh giá vị trí xuất hiện của chunk chính xác nhất trên danh sách kết quả. Vị trí càng cao điểm MRR càng lớn.
*   **Faithfulness (Độ trung thực):** Chỉ số đánh giá câu trả lời của LLM có hoàn toàn dựa trên context được cung cấp hay tự bịa thông tin ngoài.

---

## 5. Hệ Thống Multi-Agent & Kết Nối (Multi-Agent & System Integration)

*   **Multi-Agent:** Hệ thống gồm nhiều Agent LLM chuyên biệt phối hợp làm việc với nhau để giải quyết các bài toán lớn vượt quá khả năng của một Single-Agent.
*   **Supervisor-Worker Pattern:** Mẫu thiết kế gồm một Agent điều phối trung tâm (Supervisor) làm nhiệm vụ phân việc, giám sát và tổng hợp kết quả của các Agent thực thi (Workers).
*   **Pipeline Pattern:** Mẫu thiết kế các Agent chạy tuần tự theo chuỗi tuyến tính, đầu ra của Agent này là đầu vào của Agent tiếp theo.
*   **Debate Pattern:** Mẫu thiết kế các Agent cùng đưa ra câu trả lời độc lập, sau đó thảo luận, phản biện chéo để thống nhất câu trả lời tối ưu.
*   **Hierarchical Pattern:** Mẫu thiết kế phân cấp nhiều tầng supervisor lồng nhau để quản trị các nhóm task phức tạp ở quy mô lớn.
*   **Shared State:** Trạng thái bộ nhớ dùng chung lưu giữ nhiệm vụ, kết quả trung gian của các worker và nhật ký hành trình (trace).
*   **Model Context Protocol (MCP):** Giao thức mở chuẩn hóa kết nối giữa mô hình AI và các công cụ/tài nguyên bên ngoài theo kiến trúc Client-Server.
*   **A2A (Agent to Agent):** Cơ chế giao tiếp trực tiếp giữa các Agent thông qua các bản hợp đồng thông điệp (Message Contract) xác định rõ Nhiệm vụ, Ngữ cảnh và Định dạng đầu ra.
*   **LangGraph:** Thư viện điều phối đồ thị hướng có trạng thái (Stateful Graph) của LangChain, chuyên dùng để lập trình hệ thống Multi-Agent có vòng lặp suy luận phức tạp.
*   **Nodes & Edges:** Nút đại diện cho tác nhân thực thi (Agent/Hàm) và Cạnh đại diện cho luồng di chuyển dữ liệu hoặc rẽ nhánh điều kiện trong đồ thị.
*   **Graceful Degradation (Hạ cấp có kiểm soát):** Cơ chế xử lý lỗi của hệ thống Multi-Agent, đảm bảo khi một worker phụ thất bại thì toàn hệ thống vẫn hoạt động và trả ra câu trả lời bán phần thay vì crash hoàn toàn.

---

## 6. Kỹ Nghệ Dữ Liệu & Giám Sát Dữ Liệu (Data Engineering & Observability)

*   **ETL (Extract - Transform - Load):** Quy trình trích xuất, chuyển đổi làm sạch và nạp dữ liệu vào kho lưu trữ. ETL ưu tiên bảo mật bằng cách lọc thông tin nhạy cảm trước khi lưu.
*   **ELT (Extract - Load - Transform):** Quy trình nạp dữ liệu thô vào kho trước, thực hiện chuyển đổi và chunking sau để phục vụ cho các thử nghiệm.
*   **CDC (Change Data Capture):** Kỹ thuật tự động phát hiện và thu thập mọi hành động INSERT/UPDATE/DELETE trong database nguồn để đồng bộ dữ liệu thời gian thực.
*   **Idempotency (Tính khả trùng):** Đặc tính đảm bảo chạy một pipeline nhiều lần vẫn cho ra một kết quả duy nhất như chạy một lần, ngăn chặn việc nhân bản dữ liệu rác.
*   **Data Quality Gates:** Các chốt chặn tự động trên pipeline để kiểm tra cấu trúc dữ liệu, độ tươi mới và lọc PII trước khi cho phép nạp vào Vector DB.
*   **Data Observability:** Trụ cột giám sát sức khỏe dữ liệu thông qua 5 yếu tố: Freshness (Độ tươi), Distribution (Phân phối), Volume (Thể tích), Schema (Sơ đồ), Lineage (Nguồn gốc).
*   **Data Lineage:** Khả năng truy vết hành trình của dữ liệu từ file thô ở nguồn gốc $\rightarrow$ bước transform $\rightarrow$ phiên bản chunk $\rightarrow$ ngữ cảnh truy xuất $\rightarrow$ đầu ra của mô hình.

---

## 7. Bảo Mật AI & Cổng Phòng Thủ (AI Safety & Guardrails)

*   **Prompt Injection:** Tấn công chèn câu lệnh độc hại thay đổi mục tiêu của Agent.
*   **Jailbreaking:** Bypass các bộ lọc an toàn của mô hình để ép mô hình sinh ra nội dung cấm hoặc nhạy cảm.
*   **Data Leakage:** Lỗ lọt dữ liệu nội bộ hoặc thông tin cá nhân của người dùng qua câu trả lời của LLM.
*   **Lethal Trifecta:** Bộ ba nguy hiểm khi cấp đồng thời cho một Agent: Quyền đọc Private Data + Quyền đọc Untrusted Content + Quyền gọi External Comms $\rightarrow$ Cho phép rò rỉ dữ liệu tự động.
*   **Input Guardrails:** Lớp bảo vệ chặn đứng các đầu vào độc hại trước khi chuyển tới mô hình.
*   **Output Guardrails:** Lớp bảo vệ kiểm tra tính an toàn, định dạng cấu trúc và độ xác thực của câu trả lời trước khi hiển thị cho người dùng.
*   **Spotlighting:** Chiến lược an toàn sử dụng các ranh giới ký tự hoặc mã hóa để LLM nhận diện rõ đâu là dữ liệu đầu vào không được phép thực thi như lệnh.
*   **CaMeL (Privileged & Quarantined LLM):** Thiết kế an toàn cô lập LLM xử lý dữ liệu ngoài (Quarantined) khỏi LLM có quyền gọi công cụ (Privileged).
*   **Human-in-the-loop (HITL):** Các mô hình con người kiểm duyệt:
    *   *Human-on-the-loop:* Agent tự chạy, con người xem lại nhật ký sau (tác vụ rủi ro thấp).
    *   *Human-in-the-loop:* Agent soạn thảo, con người bấm nút duyệt trước khi chạy (tác vụ rủi ro trung bình).
    *   *Human-as-tiebreaker:* Con người ra quyết định chính, Agent chỉ hỗ trợ tìm kiếm (tác vụ rủi ro cao).
*   **Trust UX:** Thiết kế giao diện minh bạch tạo lòng tin bằng cách hiển thị suy nghĩ của Agent, định lượng độ tự tin, cung cấp nút Undo (hoàn tác) và cho phép cấu hình phân quyền Agent.
