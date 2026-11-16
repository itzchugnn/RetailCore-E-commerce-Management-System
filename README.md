# RetailCore – E-commerce Management System (Project Blueprint)

* **Định hướng kiến trúc:** Database-first
* **Quy mô:** Medium scale (Enterprise Information System approach)
* **Loại dự án:** Portfolio Project chuyên sâu về Kỹ thuật Cơ sở dữ liệu & Hệ thống Thông tin Doanh nghiệp

---

## 1. Triết lý & Định hướng Thiết kế

**RetailCore** là hệ thống thương mại điện tử dành cho doanh nghiệp bán lẻ quy mô vừa, quản lý tập trung sản phẩm, kho hàng, đơn hàng, thanh toán, khuyến mãi và khách hàng thông qua một nền tảng Web.

Dự án không được tiếp cận như một bài tập quản lý bán hàng CRUD thông thường mà được xây dựng như một **hệ thống thông tin doanh nghiệp thực sự**, trong đó **Database là trung tâm của project**, không phải một thành phần phụ của Backend.

Điều này đồng nghĩa với 5 nguyên tắc cốt lõi:
1. **Nghiệp vụ đi trước:** Nghiệp vụ phải được phân tích kỹ lưỡng trước khi xác định bảng.
2. **Lý do tồn tại:** Mỗi thực thể trong cơ sở dữ liệu phải có lý do tồn tại rõ ràng về mặt nghiệp vụ.
3. **Phản ánh quy tắc:** Mỗi quan hệ dữ liệu phải phản ánh đúng quy tắc nghiệp vụ (Business Rules).
4. **Phân định ràng buộc:** Mỗi ràng buộc phải được xác định rõ nên đặt ở tầng Database hay tầng Application.
5. **Tối ưu vật lý:** Mọi quyết định thiết kế vật lý phải có lý do cụ thể về tính toàn vẹn dữ liệu (Data Integrity) hoặc hiệu năng (Performance).

---

## 2. Cấu trúc Danh mục Tài liệu Thiết kế (Documentation Index)

Toàn bộ Blueprint 10 phần của dự án được tổ chức thành các tài liệu chuyên biệt trong thư mục `docs/`:

| STT | File Tài liệu | Nội dung chính | Phần tương ứng trong Blueprint |
| :--- | :--- | :--- | :--- |
| **01** | `docs/01_scope_and_business_analysis.md` | Phạm vi hệ thống, Bối cảnh bài toán, Actors (RBAC), 5 Quy trình nghiệp vụ cốt lõi & Danh mục tài liệu phân tích | Phần I & Phần II |
| **02** | `docs/02_business_rules.md` | Bộ quy tắc nghiệp vụ (Business Rules v1.0) cho Sản phẩm, Kho hàng, Đơn hàng, Khuyến mãi và Thanh toán | Phần III |
| **03** | `docs/03_conceptual_and_logical_design.md` | Thiết kế CSDL Quan niệm (ERD/EER - Deliverable 01) và Thiết kế CSDL Logic (Relational Schema, Chuẩn hóa - Deliverable 02) | Phần IV & Phần V |
| **04** | `docs/04_physical_database_sqlserver.md` | Thiết kế CSDL Vật lý trên SQL Server (Deliverable 03): Quy ước kiểu dữ liệu, DDL mẫu, Stored Procedures, Concurrency & Indexing | Phần VI |
| **05** | `docs/05_backend_frontend_and_testing.md` | Kiến trúc Backend Spring Boot (Deliverable 04), Frontend React + TypeScript (Deliverable 05) và Chiến lược Kiểm thử & Triển khai | Phần VII, VIII & IX |

---

## 3. Cấu trúc Thư mục Dự án (Repository Structure)

~~~text
RetailCore/
├── database/
│   ├── 01_database/
│   ├── 02_tables/
│   ├── 03_constraints/
│   ├── 04_indexes/
│   ├── 05_views/
│   ├── 06_procedures/
│   ├── 07_triggers/
│   ├── 08_seed_data/
│   └── 09_test_scripts/
│
├── backend/
│
├── frontend/
│
├── docs/
│   ├── 01_scope_and_business_analysis.md
│   ├── 02_business_rules.md
│   ├── 03_conceptual_and_logical_design.md
│   ├── 04_physical_database_sqlserver.md
│   ├── 05_backend_frontend_and_testing.md
│   ├── requirements/
│   ├── business-rules/
│   ├── erd/
│   ├── logical-design/
│   ├── data-dictionary/
│   ├── api/
│   └── test-cases/
│
├── docker-compose.yml
└── README.md
~~~

> **Lưu ý quan trọng về Quản lý Schema:** Các thay đổi schema cần được quản lý bằng công cụ migration (ví dụ: **Flyway**), thay vì mỗi lần chỉnh sửa lại chạy thủ công một file SQL tùy ý trên Database.

---

## 4. Lộ trình Thực hiện (PHẦN X – Project Roadmap)

Dự án được chia thành **8 Milestones** thực hiện tuần tự (dự kiến khoảng $10 - 12$ tuần làm việc đều đặn). Nguyên tắc cốt lõi: **Không vội xây dựng Web khi Database còn chưa ổn định.**

| Milestone | Công việc | Sản phẩm bàn giao (Deliverables) |
| :--- | :--- | :--- |
| **M1** | Khảo sát nghiệp vụ (Business Analysis) | SRS, Use Case, Business Rules (BR) |
| **M2** | Thiết kế quan niệm (Conceptual Design) | ERD / EER Diagram |
| **M3** | Thiết kế logic (Logical Design) | Relational Schema, Functional Dependencies (FD), Normalization |
| **M4** | Thiết kế vật lý (Physical Design) | SQL Server Database |
| **M5** | Phát triển Backend | REST API (Java Spring Boot) |
| **M6** | Phát triển Frontend | Website (React + TypeScript) |
| **M7** | Kiểm thử (Testing) | Test Report & Test Scripts |
| **M8** | Triển khai (Deployment) | Sản phẩm chạy thực tế (Docker / Environment) |

---

## 5. Tiêu chí Đánh giá Chất lượng Project

Dự án được đánh giá dưới góc nhìn của một **Database Engineer**, với phân bổ trọng số như sau:

* **Database Design:** $35\%$
* **SQL Server & Performance:** $25\%$
* **Backend Engineering:** $20\%$
* **Frontend & UX:** $10\%$
* **Testing & Deployment:** $10\%$

### Tiêu chuẩn hoàn thành tốt (Definition of Done):
- [ ] Giải thích được toàn bộ ERD và các quyết định mô hình hóa dữ liệu.
- [ ] Chứng minh được tính đúng đắn của thiết kế logic (phụ thuộc hàm, khóa, dạng chuẩn, bảo toàn thông tin).
- [ ] Triển khai được Database hoàn chỉnh trên SQL Server.
- [ ] Xử lý được các giao dịch (Transactions) có nhiều thao tác và kiểm soát truy cập đồng thời (Concurrency Control).
- [ ] Đọc hiểu được Execution Plan và giải thích được lý do đằng sau từng quyết định tạo Index.
- [ ] Xây dựng REST API sử dụng Database đúng chuẩn và an toàn.
- [ ] Xây dựng ứng dụng Web thực sự sử dụng được cho cả Khách hàng và Admin.
- [ ] Có bộ tài liệu kỹ thuật đầy đủ và kịch bản kiểm thử (Test Cases) có thể tái lập.

---