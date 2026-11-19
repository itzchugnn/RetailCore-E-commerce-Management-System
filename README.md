# RetailCore – E-commerce Management System

**RetailCore** là hệ thống quản lý thương mại điện tử và bán lẻ đa kênh (E-commerce Management System) dành cho doanh nghiệp quy mô vừa, được xây dựng theo định hướng tiếp cận **Database-First** và chuẩn kiến trúc Hệ thống Thông tin Doanh nghiệp (Enterprise Information System).

Hệ thống đặt **Cơ sở dữ liệu làm trung tâm (Database-Centric)**, tập trung bảo toàn tính toàn vẹn dữ liệu (Data Integrity), chuẩn hóa mô hình dữ liệu (3NF/BCNF), kiểm soát truy cập đồng thời (Concurrency Control) và bảo vệ nghiêm ngặt các giao dịch tài chính - kho vận (ACID).

---

## 1. Tính Năng & Phân Hệ Cốt Lõi

Hệ thống được thiết kế bao phủ toàn diện 11 phân hệ nghiệp vụ bán lẻ:

1. **Product Catalog:** Quản lý danh mục đa tầng, sản phẩm, phiên bản (SKU) và bộ thuộc tính động (Dynamic Attributes).
2. **Warehouse & Inventory:** Quản lý đa kho vật lý, cơ chế giữ hàng (Stock Reservation), số dư tức thời và sổ cái kho (Dual-Delta Ledger).
3. **Procurement:** Chu trình mua hàng từ nhà cung cấp: Đơn đặt hàng mua (PO), Nhập kho kiểm định (Goods Receipt) và Trả hàng NCC (Supplier Return).
4. **Customer & Identity:** Quản lý tài khoản, phân tách hồ sơ Khách hàng - Nhân viên, sổ địa chỉ và phân quyền vai trò (RBAC).
5. **Shopping:** Quản lý giỏ hàng trực tuyến và danh sách hàng chờ đặt.
6. **Sales & Orders:** Vòng đời hợp đồng bán hàng, cơ chế snapshot giá/địa chỉ lịch sử và hỗ trợ tách kho giao hàng (Split Shipment).
7. **Fulfillment & Shipping:** Quản lý đơn vị vận chuyển, đóng gói lô hàng theo từng kho và lịch sử các lần thử giao hàng.
8. **Promotion & Voucher:** Cấu hình khuyến mãi 2 tầng (cấp dòng sản phẩm & voucher cấp đơn) kèm quy tắc cộng dồn ưu đãi (Stacking Rules).
9. **Payment & Refund:** Quản lý các phương thức thanh toán, nhật ký thử giao dịch và quy trình hoàn tiền liên kết.
10. **Return & Exchange (RMA):** Quy trình tiếp nhận yêu cầu đổi trả, kiểm định chất lượng tại kho và kích hoạt hoàn tiền.
11. **Audit & Reporting:** Nhật ký kiểm toán an ninh vết thay đổi dữ liệu và định nghĩa/kết xuất báo cáo quản trị.

---

## 2. Tech Stack

* **DBMS:** Microsoft SQL Server
* **Quản lý Schema Migration:** Flyway
* **Backend:** Java Spring Boot (RESTful API, Spring Security, Spring Data JPA / JDBC Client)
* **Frontend:** React, TypeScript, Tailwind CSS
* **Hạ tầng & Đóng gói:** Docker, Docker Compose

---

## 3. Directory Structure

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
