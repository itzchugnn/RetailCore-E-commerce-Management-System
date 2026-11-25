# RetailCore – Enterprise E-Commerce Management System

**RetailCore** là hệ thống quản trị bán lẻ & thương mại điện tử đa kênh (Enterprise Information System) được thiết kế theo triết lý **Database-First / Database-Centric**, đặt **Microsoft SQL Server 2022+** làm nguồn sự thật duy nhất (Single Source of Truth) bảo vệ toàn vẹn toán học, kiểm soát đồng thời (Concurrency Control), hạch toán giá vốn bình quân gia quyền (**Moving WAC**) và bảo toàn lịch sử giao dịch ACID.

Toàn bộ mô hình dữ liệu chuẩn hóa gồm **79 Thực thể (`E01` – `E79`)** phân bổ trên **11 Phân hệ Nghiệp vụ (`SA01` – `SA11`)**, ánh xạ chuẩn xác `1:1` sang **79 Bảng Quan hệ** đạt chuẩn **3NF / Controlled BCNF**.

---

## Tech Stack Chuẩn Hóa

* **Cơ sở dữ liệu:** Microsoft SQL Server 2022+ (`Developer Edition` cho môi trường Dev/CI; `Enterprise Edition` cho HA Production)
* **Quản lý Schema Migration & Bảo mật DB:** Flyway T-SQL Migrations kết hợp mô hình **Dual Database Logins** (`flyway_migrator` cho DDL & `retailcore_app` tối thiểu quyền với `REVOKE DML` trên các bảng giao dịch trọng yếu)
* **Backend (M5):** Java 21+, Spring Boot 3.x, Spring Security (JWT + RBAC), Hybrid Data Access (`Spring Data JPA` với `@Immutable` Read Entities + `Spring JdbcClient` gọi các Boundary Stored Procedures nguyên tử)
* **Frontend (M6):** React, TypeScript (`strict`), Tailwind CSS, TanStack Query (Server State), Zustand (UI State), React Hook Form + Zod
* **Kiểm thử & Chịu tải (M7):** JUnit 5, Testcontainers (SQL Server 2022), k6 Concurrency Stress Suite (kiểm chứng 0% Overselling & 0 Deadlock dưới tải Flash Sale)
* **Hạ tầng & Giám sát (M8):** Docker, Docker Compose (kèm container khởi tạo `sqlserver-init`), GitHub Actions CI/CD, Prometheus (Micrometer + MSSQL Exporter) & Grafana

---

## Cấu Trúc Thư Mục & Nguồn Sự Thật (SSOT)

```text
RetailCore/
├── GEMINI.md                          # Bộ quy tắc kiến trúc luôn có hiệu lực (Always-On Rules)
├── docker-compose.yml                 # Orchestration chuẩn hóa (SQL Server 2022 + Init + App + Monitoring)
├── backend/                           # [M5] Ứng dụng Java Spring Boot & Flyway DDL Migrations
│   └── src/main/resources/db/migration/
├── database/                          # Script bootstrap DB/Logins cho Docker & tài liệu kiểm thử T-SQL
│   ├── 01_database/                   # Script khởi tạo RetailCoreDB, Filegroups, flyway_migrator & retailcore_app
│   └── 09_test_scripts/               # Kịch bản kiểm thử ACID, Deadlock & Concurrency
├── docs/
│   ├── Project-Progress-Report.md     # Báo cáo tiến độ tổng thể qua 8 Milestones
│   ├── erd/                           # [SSOT] 12 mã nguồn Mermaid ERD liên miền/phân hệ & ảnh render
│   ├── Milestones/                    # [SSOT] Hồ sơ kỹ thuật chính thức từ M1 đến M8
│   │   ├── M1-Business-Analysis/      # 11 tài liệu phân tích nghiệp vụ (215 BRs, 55 FRs, 30 DRs, BD-01..07)
│   │   ├── M2-Conceptual-Design/      # 8 tài liệu thiết kế quan niệm (79 thực thể E01..E79, CORR-01..10)
│   │   ├── M3-Logical-Design/         # Thiết kế luận lý 79 bảng, chuẩn hóa 3NF/BCNF, 8KB Sizing & SPs
│   │   ├── M4-Physical-Design/        # Thiết kế vật lý, Partitioning, Indexing & HA/DR
│   │   ├── M5-Backend/                # Đặc tả kiến trúc REST API, Security & Idempotency
│   │   ├── M6-Frontend/               # Đặc tả Storefront & Admin Back-office
│   │   ├── M7-Testing-Debugging/      # Kịch bản kiểm thử tích hợp & chịu tải đồng thời
│   │   └── M8-Deployment/             # CI/CD, Docker multi-stage & Observability
│   └── Blueprints/                    # Tài liệu đề cương ban đầu (chỉ lưu trữ tham khảo lịch sử)
└── frontend/                          # [M6] Ứng dụng Web React + TypeScript (Storefront & Admin)
```
