# Day 06 - Mini-Hackathon (Thực hành)

Tài liệu này ghi nhận nội dung và cơ chế vận hành của **Day 06: Mini-Hackathon (Thực hành)**. Đây là ngày thực hành tập trung cao độ để chuyển hóa các đặc tả Specs của dự án MVP thành một prototype chạy được và thực hiện demo thực tế trước hội đồng đánh giá.

---

## 1. Triết Lý Vận Hành: Học Qua Hành Động (Learning by Doing)

Kể từ **16:00 ngày Day 05**, lớp học kết thúc toàn bộ các bài giảng lý thuyết dài. Mọi kỹ năng và tư duy thiết kế bắt buộc phải đổ về các sản phẩm đầu ra (artifacts):
$$\text{Evidence Pack} \longrightarrow \text{Thin SPEC} \longrightarrow \text{Working Prototype} \longrightarrow \text{Live Demo}$$

*   **Không code mò mẫm:** Mọi dòng code viết ra trong ngày hackathon đều phải hướng đến việc chứng minh một giả thuyết về nỗi đau của người dùng (pain point) và kiểm thử khả năng xử lý lỗi của AI.
*   **Vibe Coding:** Sử dụng các công cụ hỗ trợ như Cursor, Copilot hoặc các thư viện mẫu để đẩy nhanh tốc độ xây dựng sản phẩm, tập trung vai trò con người vào thiết kế logic luồng và kiểm soát đầu ra.

---

## 2. Lộ Trình Triển Khai & 6 Cột Mốc Quan Trọng (Milestones)

Lộ trình chi tiết ngày Hackathon được chia thành 6 mốc kiểm soát chất lượng nghiêm ngặt (từ M1 đến M6):

```
[M1: Evidence Gate] ──> [M2: Prototype Running] ──> [M3: Freeze Scope] ──> [M4: Dry Run] ──> [M5/M6: Demo Zone]
```

*   **M1 - Cổng Bằng Chứng (Evidence Gate):**
    *   *Tiêu chí duyệt:* Đảm bảo dự án nhắm đúng đối tượng sử dụng thật (user), nỗi đau thật (pain) và lát cắt giải pháp (build slice) đủ mỏng để hoàn thành trong 24 giờ.
*   **M2 - Prototype Chạy Được (Show a Running Thing):**
    *   *Tiêu chí duyệt:* Hệ thống thực hiện được luồng dữ liệu cơ bản: **Input $\rightarrow$ AI $\rightarrow$ Output** (dù giao diện còn thô sơ). Chứng minh AI giải quyết được một ví dụ sạch đầu tiên.
*   **M3 - Khóa Phạm Vi & Hoàn Thiện Specs (Freeze Scope & Final Spec):**
    *   *Tiêu chí duyệt:* Đóng băng toàn bộ các tính năng mới (không thêm bớt tính năng để tránh làm vỡ hệ thống). Hoàn thiện bản tài liệu đặc tả kỹ thuật và nghiệp vụ cuối cùng (**Final SPEC**).
*   **M4 - Chạy Thử & Chuẩn Bị Dự Phòng (Dry Run & Backup Demo):**
    *   *Tiêu chí duyệt:* Chạy thử toàn bộ slide thuyết trình và chuẩn bị sẵn một video quay màn hình hoạt động của ứng dụng (backup video) đề phòng trường hợp lỗi mạng hoặc sập API lúc demo trực tiếp.
*   **M5 & M6 - Khu Vực Thuyết Trình (Demo Zone & Peer Feedback):**
    *   *Tiêu chí duyệt:* Thuyết trình sản phẩm thực tế trước lớp học và ban giám khảo, thực hiện chấm điểm chéo (peer review) và vinh danh các đội xuất sắc nhất.

---

## 3. Sản Phẩm Bàn Giao Cuối Giai Đoạn (Deliverables)

Vào lúc **23:59 ngày Day 06**, mỗi học viên/nhóm bắt buộc phải nộp một kho mã nguồn (GitHub Repository) được tổ chức theo cấu trúc:
1.  **Thư mục Group (`group/`):** Chứa specs dự án, mã nguồn prototype chung của cả nhóm và slide demo.
2.  **Thư mục Cá nhân (`individual/`):** Ghi nhận phần đóng góp code riêng biệt và báo cáo tự đánh giá của cá nhân trong dự án.
