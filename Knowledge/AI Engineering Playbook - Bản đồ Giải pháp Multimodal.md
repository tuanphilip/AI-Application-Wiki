# AI Engineering Playbook - Bản đồ Giải pháp Multimodal

Tài liệu này hệ thống hóa các bài toán thực tế từ lớn đến nhỏ và giải pháp công nghệ dành cho **AI Application Engineer**, được sắp xếp tuần tự theo **Vòng đời nghiệp vụ của Dữ liệu (Data Lifecycle)**, hỗ trợ cả dữ liệu văn bản, hình ảnh, video và âm thanh.

---

## 1. Giai Đoạn 1: Xác Định & Đánh Giá Dữ Liệu (Data Scoping & Identification)
*Bước phân loại nguồn dữ liệu và đánh giá mức độ sẵn sàng của hệ thống trước khi xây dựng giải pháp.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Phân loại nguồn dữ liệu nội bộ** | Tách biệt rõ 3 nhóm dữ liệu để có chiến lược xử lý: <br> 1. **Knowledge Data** (tài liệu tĩnh $\rightarrow$ RAG). <br> 2. **Operational Data** (dữ liệu động $\rightarrow$ API/SQL). <br> 3. **Contextual Data** (session/profile $\rightarrow$ inject prompt). | **AI-Fit Matrix** | Tài liệu template trên Obsidian |
| **Đánh giá mức độ sẵn sàng (Readiness)** | Chạy bộ checklist đánh giá 5 khía cạnh cốt lõi (Value, Baseline, Eval, Tolerance, Operations) để quyết định Go/No-Go. | **AI Readiness Checklist** | [Great Expectations](https://github.com/great-expectations/great_expectations) (kiểm thử chất lượng data) |
| **Xác định ranh giới & rủi ro nghiệp vụ** | Lập tài liệu Problem Statement tiêu chuẩn (xác định rõ Actor, Workflow, Bottleneck, Impact, Metric, Boundary và Human-in-the-loop). | **RACI-lite Matrix** | Obsidian Canvas templates |
| **Ước lượng ngân sách Token (Token Budgeting)** | Dự báo và tính toán trước chi phí API dựa trên khối lượng dữ liệu thô đầu vào và tần suất sử dụng của người dùng. | LiteLLM Token Counter | [LiteLLM](https://github.com/BerriAI/litellm) |
| **Thiết lập Baseline hiệu năng (Performance Baselining)** | Xác định ngưỡng chất lượng tối thiểu bằng tập dữ liệu test nhỏ hoặc heuristic truyền thống trước khi áp dụng AI để đo lường độ cải tiến thực tế. | Đánh giá Heuristic vs LLM | [Promptfoo](https://github.com/promptfoo/promptfoo) |
| **Phát hiện độ lệch phân phối dữ liệu (Data Bias & Imbalance)** | Tìm kiếm sự thiên lệch hoặc thiếu hụt dữ liệu trong tập ví dụ/tài liệu RAG (ví dụ: mất cân bằng danh mục sản phẩm). | Phân tích phân phối Embedding | [Cleanlab](https://github.com/cleanlab/cleanlab) |
| **Ước lượng mật độ thông tin ngữ nghĩa (Semantic Density)** | Đo lường mức độ cô đọng thông tin của văn bản gốc để xác định chiến lược chunking (tách đoạn) tối ưu. | Token-to-Content Ratio | Custom Python (SentenceTransformers) |
| **Đánh giá chất lượng tập ảnh/video đầu vào (Visual Quality Assessment)** | Đo lường độ phân giải, độ nhiễu và độ tương phản của tài nguyên đa phương tiện trước khi đưa vào huấn luyện LoRA/ControlNet. | Không gian màu & Phân tích tần số | PyTorch / [Torchvision](https://github.com/pytorch/vision) |
| **Kiểm soát cấu trúc tỷ lệ khung hình (Aspect Ratio & Resolution Scoping)** | Định hình các kích thước chuẩn (1:1, 16:9, 9:16) để tối ưu hóa năng lực render GPU và độ sắc nét khi sinh ảnh/video. | Trích xuất tỷ lệ tự động | [Pillow](https://github.com/python-pillow/Pillow) / [FFmpeg](https://github.com/FFmpeg/FFmpeg) |
| **Ước tính thời gian & năng lực kết xuất (GPU Render Scoping)** | Tính toán trước tài nguyên phần cứng GPU cần thiết cho các tác vụ render video/audio (dựa trên fps, sampling steps) để dự trù chi phí hạ tầng. | GPU Benchmarking sheets | Custom Python scripts |

---

## 2. Giai Đoạn 2: Thu Thập & Lấy Dữ Liệu (Data Ingestion & Gathering)
*Khâu kết nối và hút dữ liệu thô từ các nguồn đa dạng về hệ thống.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Cào trang web tĩnh & động (Web Scraping)** | Tự động quét website, bypass block, loại bỏ HTML rác, và trả về định dạng Markdown sạch dành riêng cho LLM. | **Crawl4AI** hoặc **Firecrawl API** | [Crawl4AI](https://github.com/unclecode/crawl4ai) <br> [Firecrawl](https://github.com/mendableai/firecrawl) |
| **Cào trang cuộn vô hạn (Infinite Scroll/JS)** | Sử dụng headless browser tự động hóa để cuộn trang và render Javascript trước khi trích xuất dữ liệu. | **Playwright + Crawl4AI** | [Playwright](https://github.com/microsoft/playwright-python) |
| **Cào nhanh qua URL không cần setup** | Sử dụng proxy reader trung gian để chuyển đổi nhanh trang web sang Markdown. | **Jina Reader API** | [Jina Reader](https://jina.ai/reader/) |
| **Bypass tường lửa chống bot (Anti-scraping)** | Sử dụng cơ chế xoay vòng IP (Proxy Rotation) và thay đổi User-Agent linh hoạt để vượt Cloudflare/Akamai. | Proxy Rotation | [Scrapling](https://github.com/bcambero/scrapling) |
| **Lấy dữ liệu từ tài khoản có đăng nhập** | Tích hợp xác thực bảo mật OAuth2 để trích xuất dữ liệu từ các ứng dụng được phân quyền. | **Gmail/Slack API Connectors** | [MCP SDK](https://github.com/modelcontextprotocol) |
| **Đồng bộ cơ sở dữ liệu thời gian thực** | Quét thay đổi của Database nguồn (INSERT/UPDATE/DELETE) và sync sang Vector Store/Warehouse. | **CDC (Change Data Capture)** | [Debezium](https://github.com/debezium/debezium) |
| **Gọi API ngoài có giới hạn (Rate Limits)** | Xây dựng cơ chế hàng đợi xử lý ngầm, tự động thử lại (exponential backoff) và điều tiết hàng đợi (backpressure). | **Celery + Redis** | [Celery](https://github.com/celery/celery) |
| **Trích xuất livestream âm thanh/hình ảnh (Real-time Stream Ingestion)** | Thu nhận luồng âm thanh phát trực tiếp (livestream bán hàng, họp trực tuyến) và tự động nhận diện giọng nói sang văn bản (STT). | **Whisper Live + FFmpeg** | [Whisper](https://github.com/openai/whisper) <br> [faster-whisper](https://github.com/SYSTRAN/faster-whisper) |
| **Tự động nhận diện cấu trúc trang web động (Adaptive Crawling)** | Sử dụng LLM hoặc AI Vision phân tích cấu trúc DOM để tự động xác định nút chuyển trang, nút "xem thêm" mà không cần viết script tĩnh. | LLM Selector Parser | [Crawl4AI](https://github.com/unclecode/crawl4ai) (LLM extraction mode) |
| **Giải mã CAPTCHA tự động bằng AI Vision** | Giải quyết các rào cản kiểm tra bot (Image Grid, Text CAPTCHA) bằng mô hình thị giác hoặc dịch vụ API chuyên dụng. | CapSolver API / Custom YOLO | [CapSolver SDK](https://github.com/capsolver/capsolver-python) |
| **Thu nhận luồng video stream trực tiếp (Livestream Video Ingestion)** | Chia nhỏ luồng video (RTMP/HLS) từ camera hoặc buổi livestream thành các frame ảnh tĩnh tần suất cao để xử lý phân tích thời gian thực. | Stream Decoders | [OpenCV](https://github.com/opencv/opencv) / [FFmpeg](https://github.com/FFmpeg/FFmpeg) |
| **Trích xuất âm thanh từ file video (Audio Extraction / De-muxing)** | Tách biệt hoàn toàn luồng âm thanh ra khỏi tệp video đa phương tiện mà không làm suy giảm chất lượng âm thanh gốc. | Demuxing / Audio extraction | [Pydub](https://github.com/jiaaro/pydub) / [FFmpeg wrapper](https://github.com/FFmpeg/FFmpeg) |
| **Thu thập dữ liệu phụ đề từ video (Subtitle/Metadata Ingestion)** | Tải phụ đề (.srt, .vtt) hoặc thông tin chương (chapters), mô tả chi tiết của video từ các nền tảng chia sẻ trực tuyến. | Metadata scrapers | [yt-dlp](https://github.com/yt-dlp/yt-dlp) |

---

## 3. Giai Đoạn 3: Xử Lý & Chuyển Đổi Dữ Liệu (Data Transformation & Processing)
*Làm sạch dữ liệu thô, che dấu thông tin nhạy cảm và chia cắt dữ liệu phù hợp với LLM.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Chuyển đổi PDF phức tạp sang Markdown** | Nhận diện bố cục trang (layout), trích xuất bảng biểu phức tạp và công thức toán bằng AI Vision/Layout model. | **Marker** hoặc **LlamaParse** | [Marker](https://github.com/VikParuchuri/marker) <br> [PyMuPDF](https://github.com/pymupdf/PyMuPDF) (siêu nhanh) |
| **Phân tích phân vùng tài liệu (Layout Analysis)** | Tách biệt khối văn bản, hình ảnh, bảng biểu trong PDF để đưa vào các luồng xử lý riêng biệt. | **Layoutparser** | [Unstructured.io](https://github.com/Unstructured-IO/unstructured) |
| **Sửa lỗi chính tả sau OCR (OCR Correction)** | Dùng mô hình LLM nhỏ chạy tự động để chỉnh sửa các lỗi chính tả phát sinh từ quá trình quét tài liệu cũ/mờ. | Custom Correction Prompt | [Ollama](https://github.com/ollama/ollama) |
| **Ẩn danh thông tin nhạy cảm (PII Masking)** | Tự động nhận diện và che dấu thông tin cá nhân (SĐT, Email, CMND) trước khi nạp vào Vector DB hoặc gửi sang LLM API. | **Presidio** (của Microsoft) | [Microsoft Presidio](https://github.com/microsoft/presidio) |
| **Chia nhỏ tài liệu & Gắn Metadata** | Chia nhỏ tài liệu theo ngữ nghĩa (Semantic) hoặc cấu trúc (Section/Heading) kèm overlap 10-20% và gán metadata (version, source, ACL). | **LlamaIndex Parsers** hoặc **LangChain Splitters** | [LlamaIndex](https://github.com/run-llama/llama_index) <br> [LangChain](https://github.com/langchain-ai/langchain) |
| **Lọc trùng lặp ngữ nghĩa (Deduplication)** | Sử dụng so khớp độ tương đồng Vector để phát hiện và loại bỏ các đoạn văn bản trùng lặp gần tuyệt đối nhằm tiết kiệm token. | Vector Similarity Check | Custom Python scripts |
| **Chuẩn hóa văn bản tiếng Việt** | Chuyển đổi mã Unicode dựng sẵn/tổ hợp (NFC/NFD) để tránh lỗi lệch nghĩa khi tạo vector nhúng (embedding). | Python `unicodedata` | Thư viện chuẩn Python |
| **Phân tách văn bản đa phương thức (Multimodal Chunking)** | Tách nhỏ và liên kết hình ảnh, sơ đồ kỹ thuật với văn bản mô tả trực tiếp đi kèm để không làm mất ngữ cảnh trực quan. | Multimodal Document Splitter | [LlamaIndex Multimodal](https://github.com/run-llama/llama_index) |
| **Trích xuất thực thể có cấu trúc định dạng sẵn (Structured Extraction)** | Trích xuất thông tin có cấu trúc (JSON) từ mô tả sản phẩm hoặc tài liệu tự do theo schema định sẵn một cách tin cậy. | **Instructor** hoặc **Outlines** | [Instructor](https://github.com/jxnl/instructor) <br> [Outlines](https://github.com/outlines-dev/outlines) |
| **Dịch máy chuẩn bị ngữ cảnh (Context Pre-translation)** | Dịch toàn bộ văn bản từ ngôn ngữ gốc sang ngôn ngữ của hệ thống đích trước khi thực hiện nhúng (embed) nhằm đồng nhất hóa không gian vector. | Open-source Translation models | [Hugging Face Transformers](https://github.com/huggingface/transformers) |
| **Tách nền hình ảnh tự động (Background Removal - Matting)** | Tách vật thể hoặc người ra khỏi nền hình ảnh một cách sắc nét để ghép vào thiết kế mới (hỗ trợ đắc lực cho POD). | U2Net / Matte-Anything | [Rembg](https://github.com/danielgatis/rembg) |
| **Làm nét và nâng độ phân giải (Image Upscaling / Super-Resolution)** | Tăng kích thước ảnh chất lượng thấp lên độ phân giải cao (300 DPI, 4K) để phục vụ in ấn mà không bị vỡ ảnh. | GAN-based Super-Resolution | [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) / ControlNet Tile |
| **Vẽ lại hoặc mở rộng ảnh (Inpainting / Outpainting)** | Xóa bớt vật thể thừa trong ảnh hoặc mở rộng thêm khung cảnh xung quanh bằng cơ chế khuếch tán thông minh. | Diffusion Inpainting models | [Stable Diffusion Inpainting](https://github.com/huggingface/diffusers) / [LAMA](https://github.com/advimman/lama) |
| **Tách âm thanh đa kênh (Audio Source Separation)** | Tách một file âm thanh hỗn hợp thành các kênh riêng biệt như nhạc nền (instrumental) và giọng hát (vocals). | Waveform Separation models | [Demucs](https://github.com/facebookresearch/demucs) / [Spleeter](https://github.com/deezer/spleeter) |
| **Chuẩn hóa âm thanh & Khử nhiễu (Audio Noise Reduction)** | Loại bỏ tiếng ồn trắng, tiếng gió hoặc tạp âm môi trường và đưa biên độ âm lượng về mức chuẩn hóa. | Spectral Gating / RNN | [RNNoise](https://github.com/xiph/rnnoise) / [Librosa](https://github.com/librosa/librosa) |

---

## 4. Giai Đoạn 4: Lưu Trữ & Truy Vấn Dữ Liệu (Data Storage & Retrieval)
*Lưu trữ dữ liệu vào các database chuyên dụng và tổ chức tìm kiếm tối ưu ngữ cảnh.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Lưu trữ và tìm kiếm theo nghĩa (Semantic)** | Vector Database hỗ trợ tìm kiếm độ tương đồng vector Cosine kết hợp bộ lọc Metadata Filtering. | **Chroma** (local/MVP) <br> **Qdrant** (production/scale) | [Qdrant](https://github.com/qdrant/qdrant) <br> [Chroma](https://github.com/chroma-core/chroma) |
| **Tìm kiếm chính xác mã lỗi / số hiệu** | Kết hợp tìm kiếm vector (ngữ nghĩa) và tìm kiếm từ khóa truyền thống (BM25/Lexical) $\rightarrow$ chấm điểm lại bằng Cross-Encoder. | **Hybrid Search + Reranking** | [Fastembed](https://github.com/qdrant/fastembed) (BM25) <br> [BGE-Reranker](https://github.com/FlagOpen/FlagEmbedding) |
| **Tìm kiếm xuyên ngôn ngữ (Cross-lingual)** | Người dùng hỏi bằng tiếng Việt nhưng hệ thống truy xuất tài liệu nguồn tiếng Anh và trả lời bằng tiếng Việt. | **Multilingual Embeddings** | Cohere v3 / [BGE-M3](https://github.com/FlagOpen/FlagEmbedding) |
| **Phân quyền truy cập tài liệu (Multi-tenant ACL)** | Áp điều kiện lọc metadata dựa trên ID người dùng hoặc nhóm quyền hạn để tránh xem lén tài liệu nhạy cảm. | **Metadata Attribute Filters** | Qdrant Filtering API |
| **Nén ngữ cảnh RAG (Context Pruning)** | Rút gọn tài liệu tìm thấy, chỉ giữ lại các từ chứa nhiều thông tin nhất để tiết kiệm chi phí và tăng tốc sinh. | **LLMLingua** (của Microsoft) | [LLMLingua](https://github.com/microsoft/LLMLingua) |
| **Truy vấn cơ sở dữ liệu SQL bằng chat** | Trích xuất Schema DB làm ngữ cảnh RAG cho LLM sinh SQL (Text-to-SQL) chạy trên Database read-only có sandbox. | **Vanna.ai** | [Vanna](https://github.com/vanna-ai/vanna) |
| **Truy vấn SQL an toàn cho Enterprise** | Đặt một lớp Semantic Layer trung gian. LLM chỉ sinh JSON truy vấn gọi API của Semantic Layer, API tự dịch thành SQL an toàn. | **Cube.js** | [Cube](https://github.com/cube-js/cube) |
| **Truy xuất tương tác muộn (Late Interaction / ColBERT)** | Nhúng chi tiết từng token (mức độ từ) thay vì toàn bộ câu để nâng cao độ chính xác khi tìm thông tin chi tiết. | **ColBERT** / **RAGatouille** | [RAGatouille](https://github.com/bclavie/RAGatouille) |
| **RAG trên Đồ thị Tri thức (GraphRAG)** | Kết hợp RAG với Đồ thị tri thức để tìm kiếm các thực thể có quan hệ phức tạp chéo nhau và tóm tắt toàn bộ dữ liệu. | **Microsoft GraphRAG** | [GraphRAG](https://github.com/microsoft/graphrag) |
| **Tự động viết lại và mở rộng câu hỏi (Query Rewriting)** | LLM tự sửa lỗi chính tả, dịch thuật ngữ, và mở rộng câu hỏi gốc thành nhiều câu hỏi đồng nghĩa để tối đa hóa xác suất tìm thấy tài liệu liên quan. | Query Transform Module | LangChain / LlamaIndex query engines |
| **Tìm kiếm hình ảnh tương đồng (Reverse Image Search / Visual Search)** | Tìm các tệp thiết kế, hình ảnh đồ họa có bố cục hoặc nội dung tương đương trong kho lưu trữ. | **CLIP Embeddings** | [CLIP](https://github.com/openai/CLIP) + Qdrant / Milvus |
| **Tìm khoảnh khắc trong video bằng văn bản (Temporal Video Search)** | Xác định chính xác số giây/phút trong một video dài chứa hành động hoặc cảnh tượng mô tả bằng ngôn ngữ tự nhiên. | Video-LLaVA / CLIP temporal | [Video-LLaVA](https://github.com/PKU-YuanGroup/Video-LLaVA) |
| **Lập chỉ mục và tìm kiếm âm thanh (Audio Semantic Search)** | Tìm kiếm các tệp âm thanh, nhạc nền hoặc hiệu ứng tiếng động tương đồng dựa trên giai điệu và sóng âm. | **CLAP Embeddings** | [CLAP](https://github.com/LAION-AI/CLAP) |

---

## 5. Giai Đoạn 5: Tích Hợp & Sáng Tạo / Sinh Kết Quả (Integration & Generation)
*Kết nối bộ não suy luận (LLM) với các công cụ ngoài, quản lý hội thoại và sinh ra nội dung đa phương tiện.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Kết nối Agent với hệ thống doanh nghiệp** | Sử dụng chuẩn kết nối mở của ngành để Agent tự động tương tác với Slack, Jira, Postgres, GitHub qua cổng cắm chuẩn. | **Model Context Protocol (MCP)** | [MCP Servers](https://github.com/modelcontextprotocol) |
| **Xác thực tham số công cụ (Schema Validation)** | Sử dụng mô hình kiểm tra nghiêm ngặt kiểu dữ liệu của tham số LLM sinh ra trước khi kích hoạt gọi API hệ thống. | **Pydantic Validation** | [Pydantic](https://github.com/pydantic/pydantic) |
| **Điều phối Agent chạy vòng lặp phức tạp** | Xây dựng sơ đồ chạy có quản lý trạng thái dùng chung (Shared State) và rẽ nhánh điều kiện linh hoạt. | **LangGraph** hoặc **CrewAI** | [LangGraph](https://github.com/langchain-ai/langgraph) <br> [CrewAI](https://github.com/crewAIInc/crewAI) |
| **Lưu trữ phiên làm việc dài hạn (Persistence)** | Ghi nhận và khôi phục trạng thái đồ thị quyết định của Agent qua nhiều phiên làm việc của người dùng. | **Checkpointers (Postgres/Redis)** | LangGraph Persistence API |
| **Dự phòng lỗi API nhà cung cấp (API Fallback)** | Cấu hình proxy tự động chuyển hướng request sang nhà cung cấp khác khi API chính lỗi hoặc chạm rate limit. | **LiteLLM Router** | [LiteLLM](https://github.com/BerriAI/litellm) <br> [One API](https://github.com/songquanpeng/one-api) |
| **Định tuyến LLM động theo độ khó (Routing)** | Tự động chọn model nhỏ cho tác vụ dễ (trích xuất) và model lớn/đắt cho tác vụ khó (suy luận phức tạp). | **RouteLLM** | [RouteLLM](https://github.com/lm-sys/RouteLLM) |
| **Cổng duyệt hành vi nhạy cảm (Human-in-the-loop Gate)** | Tạm dừng quy trình chạy của Agent tại các node quyết định rủi ro (thực hiện thanh toán, xóa DB) để chờ phê duyệt thủ công từ con người. | **LangGraph Interrupts** | LangGraph State Management |
| **Ánh xạ ngôn ngữ tự nhiên sang JSON Schema** | Phân tích ý định của người dùng và chuyển đổi thành cấu trúc gọi API khớp chính xác với đặc tả OpenAPI của backend. | **Function Calling API** | LangChain Tool Call / Gemini SDK |
| **Tự động sửa lỗi truy vấn (Self-Correction Loop)** | Tự phát hiện lỗi thực thi (như cú pháp SQL lỗi, lỗi compile code Python) và đưa thông báo lỗi ngược lại LLM để tự sửa và chạy lại. | Agent Re-try Loop | LangGraph conditional edges |
| **Sinh hình ảnh chất lượng cao (Text-to-Image Generation)** | Tự động vẽ tranh minh họa, thiết kế vector, thiết kế áo thun từ mô tả chi tiết của Agent. | **Flux.1-Dev** / **Stable Diffusion XL** | [Diffusers](https://github.com/huggingface/diffusers) |
| **Sinh giọng nói tự nhiên (Text-to-Speech - TTS)** | Chuyển đổi văn bản thành giọng đọc truyền cảm, hỗ trợ nhiều vùng miền với tốc độ xử lý nhanh. | **Kokoro-82M** / **F5-TTS** | [Kokoro-82M](https://github.com/hexgrad/kokoro) / [F5-TTS](https://github.com/SWHL/F5-TTS) |
| **Sinh video từ ảnh/văn bản (Video Generation)** | Tạo chuyển động cho ảnh tĩnh hoặc sinh hoạt cảnh từ kịch bản phân cảnh chi tiết. | Luma API / HunyuanVideo | [HunyuanVideo](https://github.com/Tencent/HunyuanVideo) |
| **Nhái giọng nói tự nhiên (Voice Cloning)** | Sao chép chất giọng của một người thật chỉ từ tệp ghi âm mẫu cực ngắn (3-10 giây) để đọc nội dung bất kỳ. | **GPT-SoVITS** / **XTTS-v2** | [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) |
| **Đồng bộ khẩu hình khớp âm thanh (Lipsync / Dubbing)** | Tự động thay đổi cử động môi của nhân vật trong video khớp hoàn hảo với luồng nói mới của ngôn ngữ đích. | Wav2Lip / SadTalker | [Wav2Lip](https://github.com/Rudrabha/Wav2Lip) |
| **Sinh nhạc nền tự động (AI Music Generation)** | Tạo tệp âm nhạc không lời làm nhạc nền (BGM) theo chủ đề hoặc cảm xúc yêu cầu của kịch bản. | AudioCraft / MusicGen | [Audiocraft](https://github.com/facebookresearch/audiocraft) |

---

## 6. Giai Đoạn 6: Giám Sát, Gỡ Lỗi & Đánh Giá Agent (Observability & Evaluation)
*Theo dõi sức khỏe hệ thống AI trong production và đánh giá chất lượng tự động.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Giám sát hành trình quyết định (LLM APM)** | Ghi log chi tiết toàn bộ trace: input $\rightarrow$ bước suy nghĩ (thought) $\rightarrow$ tham số gọi tool $\rightarrow$ output $\rightarrow$ latency & cost. | **Langfuse** hoặc **Arize Phoenix** | [Langfuse](https://github.com/langfuse/langfuse) <br> [Phoenix](https://github.com/Arize-AI/phoenix) |
| **Phát hiện vòng lặp vô hạn (Infinite Loop)** | Giới hạn số bước chạy tối đa (step limit) của Agent và gửi cảnh báo APM khi Agent gọi tool liên tiếp không ra kết quả. | State Step Counter | LangGraph config |
| **Đo lường chi phí người dùng thời gian thực** | Gắn nhãn metadata cho từng session để tính toán chính xác số tiền API tiêu tốn trên mỗi tài khoản khách hàng. | Langfuse Session Billing | Langfuse API |
| **Đánh giá chất lượng RAG tự động** | Chạy bộ công cụ đo lường tự động các chỉ số: Faithfulness (độ trung thực), Answer Relevance (độ liên quan). | **Ragas** | [Ragas](https://github.com/explodinggradients/ragas) |
| **Unit Test cho ứng dụng LLM** | Thiết lập bộ test suite tự động chạy đánh giá mỗi khi thay đổi Prompt/Context trước khi deploy (CI/CD integration). | **DeepEval** | [DeepEval](https://github.com/confident-ai/deepeval) |
| **Phát hiện trôi lệch prompt/mô hình (Drift Detection)** | Theo dõi chất lượng output theo thời gian để phát hiện khi nhà cung cấp nâng cấp mô hình ngầm làm giảm hiệu năng hệ thống. | Prompt/Model Monitoring | Langfuse Metrics / [Braintrust](https://www.braintrust.dev/) |
| **Đo lường mức độ ảo giác tự động (Hallucination Detection)** | Quét câu trả lời được sinh ra so với ngữ cảnh đầu vào để tính điểm độ tương đồng thực tế và loại trừ câu trả lời bịa đặt. | NLI model / HaluEval | [Vectara Hallucination Model](https://github.com/vectara/hallucination-evaluation-model) |
| **Đánh giá hội thoại nhiều lượt (Multi-turn Evaluation)** | Thiết lập bot mô phỏng người dùng để trò chuyện liên tục với Agent nhằm test khả năng duy trì ngữ cảnh dài hạn. | Agent-based Testing | [MT-Bench framework](https://github.com/lm-sys/FastChat/tree/main/fastchat/llm_judge) |
| **Đánh giá độ tương hợp ảnh - prompt (CLIP Score / Alignment)** | Đo lường mức độ trùng khớp giữa hình ảnh vẽ ra và nội dung prompt yêu cầu ban đầu để phát hiện ảnh bị thiếu chi tiết. | **CLIP Score** | TorchMetrics CLIP |
| **Đo độ chân thực của hình ảnh sinh (FID Score)** | Đo lường khoảng cách phân phối đặc trưng giữa tập ảnh thực tế và tập ảnh sinh ra để phát hiện ảnh méo/lỗi cấu trúc. | **Fréchet Inception Distance** | [Torch-Fidelity](https://github.com/toshas/torch-fidelity) |
| **Đánh giá chất lượng âm thanh nhân tạo (Speech Quality)** | Đo lường độ trong và độ tự nhiên của giọng nói TTS được tạo ra so với giọng người thật. | ViSQOL / PESQ algorithms | [ViSQOL](https://github.com/google/visqol) |

---

## 7. Giai Đoạn 7: Tối Ưu Chi Phí, Bảo Mật & An Toàn (Cost, Safety & Guardrails)
*Tối ưu tài nguyên máy chủ và dựng hàng rào bảo mật chống tấn công mô hình.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Chạy mô hình local hiệu năng cao** | Tự host các LLM mã nguồn mở (DeepSeek-R1, Qwen 2.5) trên hạ tầng riêng để bảo mật dữ liệu tuyệt đối và giảm chi phí. | **Ollama** (development) <br> **vLLM** (production/high throughput) | [vLLM](https://github.com/vllm-project/vllm) <br> [Ollama](https://github.com/ollama/ollama) |
| **Chống Prompt Injection & Jailbreak** | Thiết lập hàng rào kiểm soát cứng chặn các câu lệnh độc hại đầu vào và nội dung nhạy cảm đầu ra. | **Input/Output Guardrails** | [NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) <br> [Guardrails AI Hub](https://github.com/guardrails-ai/guardrails) |
| **Chống rò rỉ System Prompt (Prompt Leakage)** | Chèn các lớp lọc đầu vào chặn đứng hành vi hỏi dò system prompt hoặc cấu trúc hệ thống. | Prompt Protection rules | [Llama Prompt Guard](https://github.com/meta-llama/llama-recipes) |
| **Lọc mã độc trong kết quả sinh (Sanitization)** | Loại bỏ mã độc XSS, SQLi vô tình được LLM sinh ra trong câu trả lời trước khi gửi lên Client. | Output Sanitizers | Thư viện HTML Sanitizer |
| **Phân loại nội dung độc hại** | Sử dụng mô hình AI chuyên dụng nhỏ để quét nhanh tính an toàn của nội dung chat. | **Llama Guard** | [Llama Guard](https://github.com/meta-llama/llama-recipes) |
| **Ngăn rò rỉ dữ liệu tự động (Exfiltration)** | Áp dụng cấu trúc cách ly: Privileged LLM (có tool) riêng và Quarantined LLM (đọc untrusted data) riêng. Tránh cấp đủ 3 quyền Lethal Trifecta cho 1 agent. | **CaMeL Architecture** | [AgentDojo](https://github.com/suriya-kothandaraman/agent-dojo) (test safety agent) |
| **Bộ nhớ đệm ngữ nghĩa (Semantic Caching)** | Tránh gọi API LLM tốn kém bằng cách tra cứu câu trả lời đã lưu nếu câu hỏi mới có độ tương đồng ngữ nghĩa cao với câu hỏi cũ. | **GPTCache** | [GPTCache](https://github.com/zilliztech/GPTCache) |
| **Kiểm duyệt hành vi hội thoại (AI Moderation)** | Tự động phân tích và chặn các câu trả lời chứa ngôn từ không phù hợp, phân biệt chủng tộc hoặc thông tin bất lợi cho doanh nghiệp. | Moderation API / Small classifier | [OpenAI Moderation API](https://platform.openai.com/docs/guides/moderation) |
| **Giới hạn tốc độ sử dụng thích ứng (Adaptive Rate Limiting)** | Bảo vệ tài nguyên hệ thống bằng cách theo dõi số lượng token đã tiêu thụ của người dùng theo thời gian và giới hạn khi cần. | Token Buckets / Sliding Window | Redis sliding window rate limiter |
| **Tự động đóng dấu bản quyền vô hình (Invisible Watermarking)** | Chèn watermark ẩn vào tần số ảnh/âm thanh sinh ra để nhận diện tác quyền mà không làm ảnh hưởng mặt thị giác/thính giác. | IMwatermark / WavMark | [IMwatermark](https://github.com/ShieldMnt/invisible-watermark) / [WavMark](https://github.com/wavmark/wavmark) |
| **Chặn sinh hình ảnh nhạy cảm (NSFW Checker)** | Che mờ hoặc hủy kết quả sinh hình ảnh ngay lập tức nếu mô hình khuếch tán tạo ra nội dung khiêu dâm, bạo lực. | CLIP NSFW checker | Stable Diffusion Safety Checker |
| **Phát hiện nội dung Deepfake (Deepfake/AI Detection)** | Phát hiện và phân loại các tệp âm thanh giả giọng nói hoặc video bị sửa đổi gương mặt bằng mô hình AI học sâu chuyên dụng. | Deepfake Classifiers | [Deepfake-Detector](https://github.com/deepfake-detector/deepfake-detector) |

---

## 8. Giai Đoạn 8: Hạ Tầng & Triển Khai AI (AI Infrastructure & Deployment)
*Các bài toán kỹ thuật vận hành mô hình trên phần cứng vật lý, tối ưu tài nguyên GPU/CPU và xây dựng pipeline CI/CD cho ứng dụng AI.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Tech Stack Khuyên Dùng | Mã Nguồn Mở / Public Repos (GitHub) |
| :--- | :--- | :--- | :--- |
| **Host LLM cục bộ tối ưu throughput** | Phục vụ suy luận mô hình ngôn ngữ lớn (Qwen, DeepSeek) trên hạ tầng GPU riêng với độ trễ tối thiểu thông qua các kỹ thuật như PagedAttention, Continuous Batching. | **vLLM** / **TGI** | [vLLM](https://github.com/vllm-project/vllm) <br> [Text Generation Inference](https://github.com/huggingface/text-generation-inference) |
| **Lượng tử hóa mô hình tiết kiệm VRAM** | Nén kích thước mô hình (từ float16 xuống 4-bit, 8-bit) để chạy được trên các GPU có bộ nhớ hạn chế mà không làm giảm đáng kể độ chính xác. | **AWQ** / **GPTQ** / **GGUF** | [AutoAWQ](https://github.com/mit-han-lab/llm-awq) <br> [llama.cpp](https://github.com/ggerganov/llama.cpp) |
| **Load Balancing và Auto-scaling API** | Cân bằng tải lượng request inference và tự động scale số lượng container chạy GPU dựa trên hàng đợi request theo thời gian thực. | **KEDA** + **Ray Serve** | [Ray Serve](https://github.com/ray-project/ray) <br> [KEDA](https://github.com/kedacore/keda) |
| **Tối ưu hóa bộ nhớ đệm KV Cache** | Quản lý phân bổ động bộ nhớ KV Cache trong quá trình sinh token dài để tránh crash hệ thống do tràn VRAM GPU. | PagedAttention / Chunked Prefill | vLLM Engine / [SGLang](https://github.com/sgl-project/sglang) |
| **Giám sát hiệu năng phần cứng GPU** | Đo lường mức độ sử dụng VRAM, nhiệt độ, điện năng tiêu thụ và hiệu suất tính toán của GPU trong cụm Kubernetes. | DCGM Exporter + Prometheus | [NVIDIA DCGM Exporter](https://github.com/NVIDIA/dcgm-exporter) + Grafana |
| **Serving mô hình ảnh/video serverless** | Phục vụ các mô hình Stable Diffusion, Flux hoặc HunyuanVideo dưới dạng serverless có cơ chế hàng đợi xử lý bất đồng bộ. | Triton Inference Server / Ray | [Triton Inference Server](https://github.com/triton-inference-server/server) <br> [RunPod Serverless](https://github.com/runpod/runpod-python) |
| **Serving mô hình Embedding siêu tốc** | Triển khai mô hình nhúng vector (Embedding) và reranker hiệu năng cao với độ trễ cực thấp (dưới 5ms). | **TEI (Text Embeddings Inference)** | [TEI](https://github.com/huggingface/text-embeddings-inference) <br> [ONNX Runtime](https://github.com/microsoft/onnxruntime) |
| **Quản lý phiên bản & phân phối weights** | Lưu trữ tập trung các phiên bản mô hình nặng hàng trăm GB và kéo về các GPU node một cách nhanh chóng. | Private Model Registry + S3 | [MLflow Model Registry](https://github.com/mlflow/mlflow) <br> [MinIO](https://github.com/minio/minio) |
| **Mạng truyền thông cụm GPU tốc độ cao** | Tối ưu hóa băng thông mạng khi chạy phân tán mô hình siêu lớn trên nhiều máy chủ GPU vật lý khác nhau thông qua kết nối RDMA. | NVLink / InfiniBand / RoCE | [NVIDIA Collective Communications Library (NCCL)](https://github.com/NVIDIA/nccl) |
| **Tự động đóng gói & CI/CD hạ tầng AI** | Đóng gói ứng dụng chứa driver CUDA, thư viện PyTorch/TensorRT một cách gọn nhẹ và xây dựng luồng deploy tự động lên cụm GPU. | Docker (NVIDIA Toolkit) + GitOps | [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-container-toolkit) <br> [ArgoCD](https://github.com/argoproj/argo-cd) |

