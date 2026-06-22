# PM/PO Playbook - Bản đồ Giải pháp Quản trị Sản phẩm & Dự án

Tài liệu này hệ thống hóa các bài toán thực tế từ lớn đến nhỏ và phương pháp giải quyết dành cho **Product Manager (PM) / Product Owner (PO)**, được sắp xếp theo **Vòng đời quản trị sản phẩm (Product Management Lifecycle)**. 

Tài liệu tập trung vào các bài toán phi kỹ thuật (quy trình, con người, giao tiếp, phạm vi, tiến độ) mà một PM/PO phải đối mặt hàng ngày để đưa sản phẩm đến thành công.

---

## 1. Giai Đoạn 1: Khám Phá & Định Nghĩa Sản Phẩm (Product Discovery & Definition)
*Khâu nghiên cứu thị trường, thấu hiểu khách hàng và định hình giá trị cốt lõi của sản phẩm.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Xác định chân dung khách hàng** | Tránh việc xây dựng sản phẩm "cho tất cả mọi người" dẫn đến loãng tính năng và sai lệch định hướng. | Phỏng vấn người dùng, Khảo sát khảo sát | **User Persona Template** |
| **Xác định giá trị cốt lõi** | Tránh xây dựng sản phẩm không ai cần hoặc không thể cạnh tranh được với các đối thủ đang có trên thị trường. | Vẽ bản đồ giá trị mang lại so với nỗi đau của người dùng (Pain vs Gain). | **Lean Canvas** / Value Proposition Canvas |
| **Xác định phạm vi tính năng tối thiểu** | Lọc bỏ các ý tưởng thừa để đóng gói phiên bản MVP đưa ra thị trường thử nghiệm nhanh nhất có thể. | Gom cụm và phân loại tính năng theo độ quan trọng thực tế. | **Scrum MVP Scoping** / MoSCoW Method (Must, Should, Could, Won't) |
| **Xây dựng hành trình khách hàng** | Hình dung luồng đi của người dùng qua từng bước trong app để phát hiện điểm rơi (drop-off) và tối ưu trải nghiệm. | Sơ đồ hóa các điểm chạm (touchpoints) của người dùng từ lúc biết đến lúc mua hàng. | **Customer Journey Map (CJM)** / User Story Mapping |

---

## 2. Giai Đoạn 2: Quản Lý Yêu Cầu & Phạm Vi (Requirements & Scope Management)
*Lập tài liệu, kiểm soát tính năng và ưu tiên danh mục công việc để tối ưu hóa giá trị sản phẩm.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Viết mô tả tính năng rõ ràng** | Đảm bảo đội phát triển (Dev) và kiểm thử (QA) hiểu đúng 100% nghiệp vụ để code đúng tiến độ, tránh sửa đi sửa lại. | Viết tài liệu yêu cầu chi tiết kèm các kịch bản kiểm thử mẫu và điều kiện nghiệm thu. | **PRD (Product Requirement Document)** / User Story & Acceptance Criteria |
| **Ngăn chặn phình to phạm vi dự án** | Khách hàng hoặc sếp liên tục yêu cầu thêm tính năng mới trong lúc dự án đang chạy khiến dự án bị vỡ hạn. | Đánh giá tác động của thay đổi đối với tiến độ và yêu cầu ký duyệt điều chỉnh thời gian (Change Request). | **Change Management Process** (Quy trình quản lý thay đổi) |
| **Ưu tiên hóa danh sách tính năng** | Quyết định tính năng nào cần làm trước để mang lại doanh thu hoặc giá trị lớn nhất cho công ty với nguồn lực hạn chế. | Chấm điểm các tính năng dựa trên các chỉ số: Tầm ảnh hưởng, Độ tự tin, và Công sức bỏ ra. | **RICE Framework** (Reach, Impact, Confidence, Effort) / Kano Model |

---

## 3. Giai Đoạn 3: Lập Kế Hoạch & Quản Lý Tiến Độ (Planning & Timeline Management)
*Xây dựng lộ trình phát triển sản phẩm, ước lượng thời gian và quản lý các điểm nghẽn phụ thuộc.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Xây dựng lộ trình sản phẩm** | Trực quan hóa kế hoạch phát triển dài hạn để toàn bộ công ty và nhà đầu tư có chung tầm nhìn. | Sử dụng lộ trình dựa trên chủ đề (Theme-based) thay vì cam kết ngày tháng cứng nhắc để giữ tính linh hoạt. | **Product Roadmap** (Now-Next-Later) / Gantt Chart |
| **Ước lượng thời gian thiếu chính xác** | Đội dev thường ước lượng quá ngắn (dẫn đến trễ hạn) hoặc quá dài (dẫn đến lãng phí thời gian). | Ước lượng dựa trên độ khó tương đối thay vì quy đổi ra số giờ làm việc cứng nhắc. | **Agile Planning Poker** (Story Points / Fibonacci Sequence) |
| **Quản lý điểm nghẽn phụ thuộc** | Tính năng bị trì hoãn do phải đợi thiết kế từ UI/UX hoặc đợi API từ đối tác bên ngoài. | Phát hiện sớm các điểm phụ thuộc chéo (cross-dependencies) trong buổi lập kế hoạch và gỡ nút thắt trước khi Sprint bắt đầu. | **Dependency Mapping** / Critical Path Method (CPM) |

---

## 4. Giai Đoạn 4: Quản Lý Con Người & Đội Ngũ (Team & People Management)
*Giải quyết xung đột nội bộ, giữ vững động lực làm việc và tối ưu hóa phân bổ công việc.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Giải quyết xung đột Designer vs Dev** | Designer muốn giao diện cực kỳ đẹp mắt/phức tạp nhưng Dev lại muốn code đơn giản, nhanh để kịp deadline. | Tổ chức họp ba bên (PM, Dev, Designer) để thảo luận giải pháp dung hòa (Trade-off) dựa trên thời gian và trải nghiệm người dùng tối thiểu. | **Design-to-Dev Handoff Process** |
| **Giữ vững động lực đội ngũ** | Thành viên trong đội mệt mỏi do OT liên tục hoặc cảm thấy công việc lặp đi lặp lại vô nghĩa. | Chia sẻ rõ ràng tầm nhìn sản phẩm (tại sao chúng ta làm việc này) và tổ chức các buổi cải tiến quy trình để dev được nói lên tiếng nói của mình. | **Agile Retrospectives** (Mad, Sad, Glad / Start, Stop, Continue) |
| **Phân bổ công việc không đồng đều** | Tình trạng người thì quá tải nhiệm vụ, người thì ngồi chơi đợi việc vì nghẽn luồng. | Giới hạn số lượng công việc đang xử lý đồng thời để tập trung hoàn thành dứt điểm từng task. | **Kanban WIP Limits** (Work In Progress) |

---

## 5. Giai Đoạn 5: Giao Tiếp & Quản Lý Stakeholders (Stakeholder Alignment)
*Quản lý kỳ vọng của ban giám đốc, khách hàng lớn và đồng bộ thông tin giữa các phòng ban.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Quản lý kỳ vọng của sếp / Khách hàng** | Ban giám đốc muốn có tính năng A ngay lập tức nhưng đội dev không đủ nguồn lực. | Từ chối khéo léo bằng cách đưa ra dữ liệu thực tế: "Nếu làm tính năng A thì sẽ phải hoãn tính năng B, C đang mang lại doanh thu". | **Stakeholder Matrix** (Power-Interest Grid) / Trade-off Matrix |
| **Lệch pha thông tin giữa các phòng ban** | Bộ phận Sales/Marketing đi quảng cáo bán một tính năng chưa hề được phát triển hoặc chưa hoàn thiện. | Tổ chức họp đồng bộ định kỳ hàng tuần giữa Product và Business (Sales, Marketing, CS) và phát hành bản tin cập nhật sản phẩm. | **Product-to-Business Sync** / Release Notes Portal |

---

## 6. Giai Đoạn 6: Triển Khai Sprint & Chất Lượng (Sprint Execution & Quality)
*Vận hành quy trình làm việc Agile hàng ngày, xử lý việc đột xuất và nghiệm thu sản phẩm.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Xử lý yêu cầu đột xuất giữa Sprint** | Bug nghiêm trọng phát sinh hoặc yêu cầu khẩn cấp từ sếp phá vỡ kế hoạch Sprint đang chạy. | Bảo vệ mục tiêu Sprint cốt lõi. Nếu tính năng mới bắt buộc phải vào, phải rút bớt task có công sức tương đương ra khỏi Sprint. | **Scrum Guide Sprint Cancellation / Task Swap** |
| **Nghiệm thu tính năng trước khi phát hành** | Đảm bảo tính năng chạy đúng logic và thân thiện với người dùng trước khi cho deploy rộng rãi. | PM trực tiếp đóng vai trò người dùng cuối để test thử nghiệm và ký duyệt nghiệm thu. | **UAT (User Acceptance Testing)** / Dogfooding |

---

## 7. Giai Đoạn 7: Đo Lường & Tối Ưu Hóa Sau Phát Hành (Post-Release & Metrics)
*Đo lường mức độ thành công của sản phẩm, thu thập phản hồi và cải tiến liên tục.*

| Bài Toán Cụ Thể (Bài Toán Nhỏ) | Hướng Tiếp Cận / Giải Pháp | Công Cụ Khuyên Dùng | Khung Quản Trị Chuẩn (Framework) |
| :--- | :--- | :--- | :--- |
| **Đo lường sự thành công của tính năng** | Làm sao biết tính năng mới có mang lại hiệu quả kinh doanh hay người dùng không hề đụng tới? | Thiết lập trước các chỉ số đo lường hiệu năng cốt lõi (như số lượt click, thời gian ở lại trang) trước khi phát hành. | **HEART Framework** (Happiness, Engagement, Adoption, Retention, Task Success) |
| **Xử lý phản hồi tiêu cực của khách hàng** | Khách hàng phàn nàn, đánh giá 1 sao trên App Store hoặc khiếu nại qua kênh Chăm sóc khách hàng. | Phân loại feedback theo nhóm (Bug, Usability, Feature Request) để đưa vào Backlog sắp xếp ưu tiên cho các Sprint sau. | **Feedback Loop / Bug Triage** |
| **Đánh giá mục tiêu kinh doanh sản phẩm** | Đo lường xem sản phẩm có đi đúng hướng doanh thu và chiến lược lâu dài của toàn công ty hay không. | Đặt ra các mục tiêu lớn hàng quý và các chỉ số đo lường kết quả cụ thể. | **OKR Framework** (Objectives and Key Results) |
