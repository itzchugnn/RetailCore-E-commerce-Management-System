# RetailCore – E-commerce Management System

**RetailCore** là hệ thống quản lý thương mại điện tử và bán lẻ đa kênh (E-commerce Management System) dành cho doanh nghiệp quy mô vừa, được xây dựng theo định hướng tiếp cận **Database-First** và chuẩn kiến trúc Hệ thống Thông tin Doanh nghiệp (Enterprise Information System).

Hệ thống đặt **Cơ sở dữ liệu làm trung tâm (Database-Centric)**, tập trung bảo toàn tính toàn vẹn dữ liệu (Data Integrity), chuẩn hóa mô hình dữ liệu (3NF/BCNF), kiểm soát truy cập đồng thời (Concurrency Control) và bảo vệ nghiêm ngặt các giao dịch tài chính - kho vận (ACID).

---

## Tech Stack

* **DBMS:** Microsoft SQL Server
* **Quản lý Schema Migration:** Flyway
* **Backend:** Java Spring Boot (RESTful API, Spring Security, Spring Data JPA / JDBC Client)
* **Frontend:** React, TypeScript, Tailwind CSS
* **Hạ tầng & Đóng gói:** Docker, Docker Compose

---

## Directory Structure

```text
RetailCore/
├── backend/
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
├── docs/
│   ├── api/
│   ├── Blueprints/
│   ├── business-rules/
│   ├── data-dictionary/
│   ├── erd/
│   ├── logical-design/
│   ├── Milestones/
│   │   ├── M1-Business-Analysis/
│   │   ├── M2-Conceptual-Design/
│   │   ├── M3-Logical-Design/
│   │   ├── M4-Physical-Design/
│   │   ├── M5-Backend/
│   │   ├── M6-Frontend/
│   │   ├── M7-Testing-Debugging/
│   │   └── M8-Deployment/
│   ├── requirements/
│   └── test-cases/
└── frontend/
```
