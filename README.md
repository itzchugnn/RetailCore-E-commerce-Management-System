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

Toàn bộ tài liệu kiến trúc và đặc tả triển khai được tổ chức chuyên biệt trong thư mục `docs/`:

### 2.1. Blueprints Thiết kế Tổng thể (`docs/Blueprints/`)
| STT | File Tài liệu | Nội dung chính | Phần tương ứng |
| :--- | :--- | :--- | :--- |
| **01** | `docs/Blueprints/01_scope_and_business_analysis.md` | Phạm vi hệ thống, Bối cảnh bài toán, Actors (RBAC), 5 Quy trình cốt lõi | Phần I & II |
| **02** | `docs/Blueprints/02_business_rules.md` | Bộ quy tắc nghiệp vụ (Business Rules v1.0) cho Sản phẩm, Kho, Đơn, Khuyến mãi | Phần III |
| **03** | `docs/Blueprints/03_conceptual_and_logical_design.md` | Thiết kế CSDL Quan niệm (ERD/EER) & Thiết kế CSDL Logic (Relational Schema, 3NF/BCNF) | Phần IV & V |
| **04** | `docs/Blueprints/04_physical_database_sqlserver.md` | Thiết kế CSDL Vật lý SQL Server: DDL mẫu, Stored Procedures, Concurrency & Indexing | Phần VI |
| **05** | `docs/Blueprints/05_backend_frontend_and_testing.md` | Kiến trúc Backend Spring Boot, Frontend React + TypeScript & Chiến lược Kiểm thử | Phần VII, VIII & IX |

### 2.2. Tài liệu Thực thi Chi tiết theo Milestone (`docs/Milestones/`)
* **Milestone M1 – Business Analysis (`docs/Milestones/M1-Business-Analysis/`) [COMPLETED]:**
  * `01-Project-Charter.md`: Tuyên ngôn dự án, bối cảnh, phạm vi và mục tiêu.
  * `02-Stakeholder-Register.md`: Danh mục bên liên quan, tác nhân hệ thống (Actors) và ma trận RBAC.
  * `03-Use-Case-Diagram.md`: Sơ đồ Use Case tổng thể và phân rã các quan hệ include/extend.
  * `04-Use-Case-Specifications.md`: Đặc tả chi tiết từng Use Case trọng tâm và kịch bản luồng xử lý.
  * `05-Business-Processes.md`: Sơ đồ quy trình nghiệp vụ cốt lõi (Đặt hàng, Giữ kho, Xử lý đơn).
  * `06-Business-Rules.md`: Bộ quy tắc nghiệp vụ v1.1 (BR-PR01..12, BR-IN01..21, BR-PM01..14...).
  * `07-Functional-Requirements.md`: Danh sách yêu cầu chức năng theo từng phân hệ.
  * `08-Non-Functional-Requirements.md`: Yêu cầu phi chức năng (Hiệu năng, ACID, Bảo mật, Sao lưu).
  * `09-Data-Requirements.md`: Đặc tả yêu cầu dữ liệu v1.1 (DR-11..24, HDR-13..18) và ràng buộc CSDL.
  * `10-Traceability-Matrix.md`: Ma trận truy vết toàn diện & Điều kiện nghiệm thu M1 (11 Exit Criteria).
  * `11-Business-Decisions.md`: Chính thức hóa 7 quyết định nghiệp vụ (BD-01..07) với phân tích sâu BD-05 & BD-07.

* **Milestone M2 – Conceptual Database Design (`docs/Milestones/M2-Conceptual-Design/`) [IN PROGRESS]:**
  * `01-Entity-Catalog.md`: Danh mục thực thể quan niệm (Strong, Weak, Associative), đào sâu Product Catalog & Inventory.
  * `02-Relationship-Analysis.md`: Phân tích quan hệ giữa các thực thể và bản số (Cardinality).
  * `03-Cardinality-and-Constraints.md`: Ràng buộc nghiệp vụ và tính tham gia toàn phần/một phần.
  * `04-Conceptual-ERD.md`: Sơ đồ ERD/EER quan niệm hoàn chỉnh (Mermaid/PlantUML).
  * `05-Design-Decisions.md`: Quyết định thiết kế cốt lõi (OrderAllocation, vòng đời tồn kho 5 bước, M2-Q01..Q10).
  * `06-M1-M2-Traceability.md`: Ma trận truy vết từ yêu cầu M1 sang thực thể M2.

---

## 3. Cấu trúc Thư mục Dự án (Repository Structure)

```text
RetailCore/
├── database/
│   ├── 01_database/            # DDL tạo database, collation, encoding
│   ├── 02_tables/              # DDL khởi tạo bảng và kiểu dữ liệu
│   ├── 03_constraints/         # Khóa ngoại, CHECK, UNIQUE constraints
│   ├── 04_indexes/             # Clustered, Non-clustered & Filtered Indexes
│   ├── 05_views/               # Views phục vụ báo cáo và giao diện
│   ├── 06_procedures/          # Stored Procedures xử lý transaction cốt lõi
│   ├── 07_triggers/            # Triggers phục vụ audit trail và data integrity
│   ├── 08_seed_data/           # Dữ liệu mẫu (Master data & Mock data)
│   └── 09_test_scripts/        # Kịch bản kiểm thử SQL & Concurrency test
│
├── backend/                    # Mã nguồn Backend (Java Spring Boot REST API)
│
├── frontend/                   # Mã nguồn Frontend (React + TypeScript)
│
├── docs/                       # Toàn bộ tài liệu kỹ thuật & nghiệp vụ
│   ├── Blueprints/             # Bộ blueprint kiến trúc 10 phần tổng thể
│   │   ├── 01_scope_and_business_analysis.md
│   │   ├── 02_business_rules.md
│   │   ├── 03_conceptual_and_logical_design.md
│   │   ├── 04_physical_database_sqlserver.md
│   │   └── 05_backend_frontend_and_testing.md
│   │
│   ├── Milestones/             # Chi tiết sản phẩm bàn giao theo 8 Milestones
│   │   ├── M1-Business-Analysis/       # [COMPLETED] 11 tài liệu phân tích nghiệp vụ
│   │   │   ├── 01-Project-Charter.md
│   │   │   ├── 02-Stakeholder-Register.md
│   │   │   ├── 03-Use-Case-Diagram.md
│   │   │   ├── 04-Use-Case-Specifications.md
│   │   │   ├── 05-Business-Processes.md
│   │   │   ├── 06-Business-Rules.md
│   │   │   ├── 07-Functional-Requirements.md
│   │   │   ├── 08-Non-Functional-Requirements.md
│   │   │   ├── 09-Data-Requirements.md
│   │   │   ├── 10-Traceability-Matrix.md
│   │   │   └── 11-Business-Decisions.md
│   │   ├── M2-Conceptual-Design/       # [IN PROGRESS] Thiết kế CSDL Quan niệm
│   │   │   ├── 01-Entity-Catalog.md
│   │   │   ├── 02-Relationship-Analysis.md
│   │   │   ├── 03-Cardinality-and-Constraints.md
│   │   │   ├── 04-Conceptual-ERD.md
│   │   │   ├── 05-Design-Decisions.md
│   │   │   └── 06-M1-M2-Traceability.md
│   │   ├── M3-Logical-Design/          # Relational Schema & Chuẩn hóa 3NF/BCNF
│   │   ├── M4-Physical-Design/         # Thiết kế vật lý SQL Server
│   │   ├── M5-Backend/                 # Thiết kế kiến trúc Backend Spring Boot
│   │   ├── M6-Frontend/                # Thiết kế UI/UX & Luồng ứng dụng
│   │   ├── M7-Testing-Debugging/       # Kế hoạch & Báo cáo kiểm thử
│   │   └── M8-Deployment/              # Tài liệu đóng gói & Triển khai Docker
│   │
│   ├── api/                    # Đặc tả OpenAPI / Postman collection
│   ├── business-rules/         # Tham chiếu quy tắc nghiệp vụ mở rộng
│   ├── data-dictionary/        # Từ điển dữ liệu chi tiết từng cột/bảng
│   ├── erd/                    # File thiết kế sơ đồ ERD (Draw.io / DBML / PlantUML)
│   ├── logical-design/         # Tài liệu chứng minh phụ thuộc hàm & chuẩn hóa
│   ├── requirements/           # SRS & Đặc tả yêu cầu người dùng
│   └── test-cases/             # Kịch bản kiểm thử nghiệp vụ
│
├── docker-compose.yml          # Cấu hình khởi chạy môi trường CSDL & Dịch vụ
└── README.md                   # Tổng quan dự án & Hướng dẫn tra cứu
```

> **Lưu ý quan trọng về Quản lý Schema:** Các thay đổi schema cần được quản lý bằng công cụ migration (ví dụ: **Flyway**), thay vì mỗi lần chỉnh sửa lại chạy thủ công một file SQL tùy ý trên Database.

---

## 4. Lộ trình Thực hiện (PHẦN X – Project Roadmap)

Dự án được chia thành **8 Milestones** thực hiện tuần tự (dự kiến khoảng $10 - 12$ tuần làm việc đều đặn). Nguyên tắc cốt lõi: **Không vội xây dựng Web khi Database còn chưa ổn định.**

| Milestone | Công việc | Sản phẩm bàn giao (Deliverables) | Trạng thái |
| :---: | :--- | :--- | :---: |
| **M1** | Khảo sát nghiệp vụ (Business Analysis) | SRS, Use Case, Business Rules, Decision Register | **Completed** |
| **M2** | Thiết kế quan niệm (Conceptual Design) | ERD / EER Diagram | **In Progress** |
| **M3** | Thiết kế logic (Logical Design) | Relational Schema, Functional Dependencies, Normalization | Pending |
| **M4** | Thiết kế vật lý (Physical Design) | SQL Server Database, DDL, Stored Procedures, Indexes | Pending |
| **M5** | Phát triển Backend | REST API (Java Spring Boot, Data JPA / JDBC) | Pending |
| **M6** | Phát triển Frontend | Website Storefront & Admin Portal (React + TypeScript) | Pending |
| **M7** | Kiểm thử (Testing) | Test Report, Test Scripts, Concurrency & Query Optimization | Pending |
| **M8** | Triển khai (Deployment) | Sản phẩm chạy thực tế (Docker Compose / Cloud Hosting) | Pending |

---

## 5. Tiêu chí Đánh giá Chất lượng Project

Dự án được đánh giá dưới góc nhìn của một **Database Engineer**, với phân bổ trọng số như sau:

* **Database Design:** $35\%$
* **SQL Server & Performance:** $25\%$
* **Backend Engineering:** $20\%$
* **Frontend & UX:** $10\%$
* **Testing & Deployment:** $10\%$

### Tiêu chuẩn hoàn thành tốt (Definition of Done):
- [x] Làm rõ mô hình kinh doanh, phạm vi hệ thống, tác nhân và quy trình nghiệp vụ cốt lõi (M1).
- [x] Thống nhất bộ Business Rules và chốt 7 quyết định kiến trúc CSDL BD-01 đến BD-07 (M1).
- [ ] Giải thích được toàn bộ ERD và các quyết định mô hình hóa dữ liệu (M2).
- [ ] Chứng minh được tính đúng đắn của thiết kế logic (phụ thuộc hàm, khóa, dạng chuẩn, bảo toàn thông tin) (M3).
- [ ] Triển khai được Database hoàn chỉnh trên SQL Server với Stored Procedures và Triggers (M4).
- [ ] Xử lý được các giao dịch (Transactions) có nhiều thao tác và kiểm soát truy cập đồng thời (Concurrency Control).
- [ ] Đọc hiểu được Execution Plan và giải thích được lý do đằng sau từng quyết định tạo Index.
- [ ] Xây dựng REST API sử dụng Database đúng chuẩn và an toàn (M5).
- [ ] Xây dựng ứng dụng Web thực sự sử dụng được cho cả Khách hàng và Admin (M6).
- [ ] Có bộ tài liệu kỹ thuật đầy đủ và kịch bản kiểm thử (Test Cases) có thể tái lập (M7).

---

## 6. Trạng thái Hiện tại & Bước Tiếp theo

* **Trạng thái hiện tại:** `Milestone M1 Completed` (Đã hoàn tất phân tích nghiệp vụ, đặc tả Use Case, quy tắc nghiệp vụ và chốt 7 quyết định kỹ thuật BD-01 đến BD-07).
* **Bước tiếp theo:** **Milestone M2 – Thiết kế CSDL Quan niệm (Conceptual Design - ERD/EER)**.
  1. Xác định danh sách thực thể mạnh (Strong Entities), thực thể yếu (Weak Entities) và thực thể liên kết (Associative Entities).
  2. Xác định các thuộc tính đơn, thuộc tính phức hợp, thuộc tính đa trị và thuộc tính dẫn xuất.
  3. Mô hình hóa mối quan hệ và bản số (Cardinality & Participation Constraints: $1-1, 1-N, N-N$, Total/Partial).
  4. Áp dụng kỹ thuật kế thừa/chuyên biệt hóa EER Diagram (Generalization / Specialization) nếu cần thiết.
  5. Xây dựng sơ đồ ERD hoàn chỉnh phản ánh đúng 7 quyết định nghiệp vụ đã chốt.
