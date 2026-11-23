# BÁO CÁO TIẾN ĐỘ DỰ ÁN & TIẾN TRÌNH XÂY DỰNG KIẾN TRÚC RETAILCORE
## SYSTEM CONSTRUCTION CHRONICLE & PROGRESS REPORT MILESTONE-BY-MILESTONE

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Mô hình Kiến trúc:** Monolithic Layered Architecture, Database-Centric (EIS)
* **Quy mô Dự phóng:** 1.000.000 đơn hàng/năm, 10.000.000 biến động kho/năm, tính sẵn sàng 99.99%
* **Phiên bản Tài liệu:** v3.1 – Tiến trình Xây dựng theo Milestone (Milestone-by-Milestone Engineering Chronicle)
* **Trạng thái:** **Hoàn thành M1 & M2, Giải quyết 10 Pre-M3 Issues, Chuẩn bị M3.01**
* **Quản lý:** Lead Enterprise Database Engineer & Solution Architect

---

## 1. TỔNG QUAN TIẾN TRÌNH XÂY DỰNG HỆ THỐNG

Dự án RetailCore được thiết lập và phát triển dựa trên phương pháp luận **Database-First / Database-Centric** dành cho Hệ thống Thông tin Doanh nghiệp quy mô vừa (Enterprise Information System). Khác với các hệ thống CRUD thông thường, RetailCore đặt mô hình CSDL làm "Single Source of Truth" để bảo vệ tính toàn vẹn toán học của dữ liệu, kiểm soát truy cập đồng thời và bảo toàn các giao dịch tài chính - kho vận (ACID).

Tiến trình xây dựng toàn diện của dự án được phân định qua **8 Cột mốc (Milestones)** kế thừa liên tục:

```
[M1: Business Analysis] ──(112 BRs & BD-01..07)──> [M2: Conceptual Design] ──(72 Entities & CORR-01..10)──> 
──[Pre-M3 Gate Resolution] ──(10 Issues Resolved)──> [M3: Logical Design] ──> [M4: Physical] ──> [M5: Backend] ──> ...
```

### Bảng theo dõi tiến độ lũy tiến qua các Milestones:

| Cột mốc (Milestone) | Trọng số | Tiến độ | Trạng thái Nghiệm thu | Nội dung Đã Xây dựng & Điểm Chuyển giao |
| :--- | :---: | :---: | :---: | :--- |
| **M1: Phân tích Nghiệp vụ & Yêu cầu** *(Business Analysis)* | 20% | **100%** | **NGHIỆM THU BASELINE (Signed Off)** | • Hoàn thành bộ 11 tài liệu đặc tả nghiệp vụ (`M1.01` – `M1.11`).<br>• Mã hóa 112 Quy tắc nghiệp vụ (`BR-01` – `BR-112`).<br>• Thiết lập 7 Quyết định nghiệp vụ nền tảng (`BD-01` – `BD-07`). |
| **M2: Thiết kế CSDL Mức Quan niệm** *(Conceptual Design)* | 30% | **100%** | **CONDITIONAL APPROVAL (Nghiệm thu Có Điều kiện)** | • Định danh và chuẩn hóa 72 thực thể quan niệm (**E01 – E72**) trên 11 Subject Areas (**SA01 – SA11**).<br>• Đóng gói 12 sơ đồ Mermaid ERD và 12 ảnh PNG render tương ứng.<br>• Ban hành Bản ghi hiệu chỉnh `M2.08` xử lý 10 lỗi gốc (CORR-01..10). |
| **Giai đoạn Tiền M3 (Pre-M3 Gate Resolution)** | — | **100%** | **APPROVED & FULLY ALIGNED (Đã Giải quyết 100%)** | • Tháo gỡ triệt để 10 Open Issues kỹ thuật (B1–B7, C1–C4, D1–D4).<br>• Đặc tả DDL schema cho `ReturnShipmentAllocation`, COD 3 lớp, Audit Trail E72.<br>• Ban hành chính sách chuẩn hóa 3NF/BCNF thực tế và chuẩn kiểu tiền tệ VNĐ. |
| **M3: Thiết kế CSDL Mức Logic** *(Logical Design)* | 25% | **0%** | **KHỞI ĐỘNG (Starting M3.01)** | • Khởi động công đoạn **M3.01 (Relational Schema Mapping)** cho `SA01 – Product Catalog`.<br>• Lộ trình gồm 4 bước: Ánh xạ quan hệ $\to$ Chuẩn hóa 3NF/BCNF $\to$ Định cỡ dữ liệu Sizing $\to$ DDL Constraints & Stored Procedures. |
| **M4: Thiết kế Vật lý & Tối ưu Hiệu năng** *(Physical Design)* | 15% | **0%** | **CHỜ THỰC HIỆN** | • Partitioning Scheme theo quý/năm, Chỉ mục chuyên sâu (Filtered/Columnstore), Chiến lược HA/DR. |
| **M5: DDL Triển khai & Backend API** | 10% | **0%** | **CHỜ THỰC HIỆN** | • Script T-SQL tạo Schema hoàn chỉnh, Flyway Migration, Seed Data 1.000.000 dòng tải thử nghiệm, Spring Boot REST API. |
| **M6: Frontend (React / TypeScript)** | — | **0%** | **CHỜ THỰC HIỆN** | • Giao diện Web App điều hành & bán hàng đa kênh (Admin, Staff, Customer). |
| **M7: Testing & Quality Assurance** | — | **0%** | **CHỜ THỰC HIỆN** | • Concurrency Stress Test, ACID Transaction Verification, Automated Integration Test. |
| **M8: Đóng gói & Triển khai (CI/CD)** | — | **0%** | **CHỜ THỰC HIỆN** | • Docker Compose, CI/CD pipeline, Health check & Prometheus/Grafana Monitoring. |

---

## 2. QUÁ TRÌNH XÂY DỰNG MILESTONE 1: PHÂN TÍCH NGHIỆP VỤ & ĐẶC TẢ YÊU CẦU (M1)

### 2.1. Bối cảnh & Bài toán Doanh nghiệp
Trong các doanh nghiệp bán lẻ đa kênh quy mô vừa, các hoạt động cốt lõi thường bị phân mảnh:
* Danh mục và biến thể sản phẩm (SKU) thiếu tính nhất quán giữa kho và sàn thương mại điện tử.
* Tồn kho đa kho thiếu cơ chế Reservation dẫn đến hiện tượng Overselling trong các chiến dịch Flash Sale.
* Không hỗ trợ chia tách kiện hàng giao từ nhiều kho (Split Shipment) hoặc thất thoát công nợ thu hộ COD khi một đơn giao làm nhiều đợt.
* Khuyến mãi nhiều tầng gây sai lệch hạch toán kế toán và làm tròn tiền lẻ VNĐ.

Cột mốc M1 đã tập trung phân tích sâu và mô hình hóa toàn bộ yêu cầu nghiệp vụ độc lập với công nghệ lưu trữ dữ liệu.

### 2.2. Đóng gói bộ 11 tài liệu M1 (Deliverables M1.01 – M1.11)
Quá trình xây dựng M1 được cấu trúc hóa thành 11 văn bản đặc tả hoàn chỉnh tại thư mục `docs/Milestones/M1-Business-Analysis/`:
1. [01-Project-Charter.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/01-Project-Charter.md): Xác lập điều lệ dự án, phạm vi quy mô vừa, đối tượng sử dụng (Customer, Sales Staff, Warehouse Staff, Marketing Staff, Admin).
2. [02-Stakeholder-Register.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/02-Stakeholder-Register.md): Định danh vai trò, trách nhiệm và ma trận kỳ vọng của các bên liên quan.
3. [03-Use-Case-Diagram.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/03-Use-Case-Diagram.md) & [04-Use-Case-Specifications.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/04-Use-Case-Specifications.md): Đặc tả chi tiết kịch bản sử dụng cho 10 phân hệ nghiệp vụ.
4. [05-Business-Processes.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/05-Business-Processes.md): Chuẩn hóa luồng quy trình nghiệp vụ (BPMN) cho các chuỗi hoạt động mua sắm, nhập kho, giao vận, đổi trả.
5. [06-Business-Rules.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/06-Business-Rules.md): **Bộ 112 Quy tắc nghiệp vụ (BR01 – BR112)** được định danh duy nhất theo 10 nhóm (`BR-ID`, `BR-PR`, `BR-IN`, `BR-PO`, `BR-CU`, `BR-OR`, `BR-PRM`, `BR-PY`, `BR-FS`, `BR-RT`).
6. [07-Functional-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/07-Functional-Requirements.md) & [08-Non-Functional-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/08-Non-Functional-Requirements.md): Hệ thống hóa 55 yêu cầu chức năng và các chỉ số phi chức năng (RPO < 5 phút, RTO < 30 phút, tải 1.000.000 đơn/năm).
7. [09-Data-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/09-Data-Requirements.md): Từ điển dữ liệu nghiệp vụ sơ bộ.
8. [10-Traceability-Matrix.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/10-Traceability-Matrix.md): Ma trận truy vết 2 chiều bảo đảm 100% yêu cầu nghiệp vụ đều có căn cứ xuất phát.
9. [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md): **7 Quyết định Nghiệp vụ Nền tảng (BD-01 đến BD-07)** định hình toàn bộ mô hình CSDL sau này.

### 2.3. Bảy quyết định nghiệp vụ trọng tâm (BD-01 đến BD-07)
* **BD-01 (Sản phẩm – Danh mục $N:N$):** Một sản phẩm thuộc nhiều danh mục, bắt buộc có đúng 1 danh mục chính (`IsPrimary = 1`) khi Published để phục vụ breadcrumb và hạch toán doanh thu không trùng lặp.
* **BD-02 (Bắt buộc đăng nhập):** Cấm Guest Checkout; `Orders.CustomerID` bắt buộc `NOT NULL` để kiểm soát voucher cá nhân và thời hạn đổi trả 7 ngày.
* **BD-03 (Split Shipment):** Đơn hàng không bị gắn cứng vào 1 kho; 1 dòng đơn hàng có thể phân bổ xuất từ nhiều kho vật lý.
* **BD-04 (Stock Reservation có hạn):** Khóa giữ tồn kho logic ngay khi tạo đơn với thời hạn `ExpiresAt` (15 phút), chống âm kho.
* **BD-05 (Promotion Stacking phân tầng):** Tách bạch 2 tầng khuyến mãi: Tầng 1 (Product Promotion trên từng dòng) và Tầng 2 (Order Voucher trên toàn đơn). Chỉ được kết hợp khi voucher có cờ `IsStackable = 1`.
* **BD-06 (Mô hình Thuộc tính Động 2 Lớp):** Tách bạch vai trò thuộc tính qua `AttributeRole`: `'VARIANT_DEFINING'` (cấu thành SKU) và `'DESCRIPTIVE'` (mô tả chung).
* **BD-07 (Tồn kho khi giao hàng thất bại):** Xuất kho giảm đồng thời `OnHandQty` và `ReservedQty`. Khi shipper báo giao thất bại, hàng chưa về kho thìnchưa được cộng tồn kho cho đến khi thủ kho kiểm đếm ký biên bản tiếp nhận `DeliveryReturnReceipt`.

---

## 3. QUÁ TRÌNH XÂY DỰNG MILESTONE 2: THIẾT KẾ CƠ SỞ DỮ LIỆU MỨC QUAN NIỆM (M2)

Từ các yêu cầu và quy tắc nghiệp vụ của M1, Milestone 2 đã tiến hành mô hình hóa thành **Mô hình Thực thể - Quan hệ Quan niệm (Conceptual ERD)** độc lập nền tảng phần cứng.

### 3.1. Danh mục 72 Thực thể Quan niệm (E01 – E72) trên 11 Subject Areas
Quá trình Bounded Contexts đã định danh 72 thực thể cốt lõi tại [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md):

* **SA01 – Product Catalog (E01 – E09):** `Category` (E01), `Product` (E02), `ProductCategory` (E03), `ProductVariant` (E04 - Hạt nhân SKU), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09).
* **SA02 – Warehouse & Inventory (E10 – E20):** `Warehouse` (E10), `InventoryBalance` (E11), `OrderWarehouseAllocation` (E13), `StockReservation` (E14), `StockReservationLine` (E15), `InventoryTransaction` (E16), `InventoryTransactionLine` (E17), `StockCount` (E18), `StockCountLine` (E19), `WarehouseTransfer` (E20), `WarehouseTransferLine` (E12).
* **SA03 – Procurement (E21 – E28):** `Supplier` (E21), `SupplierContact` (E22), `SupplierProduct` (E23), `PurchaseOrder` (E24), `PurchaseOrderLine` (E25), `GoodsReceipt` (E26), `GoodsReceiptLine` (E27), `SupplierReturn` (E28), `SupplierReturnLine` (E28b).
* **SA04 – Customer & Identity (E29 – E37):** `Account` (E29), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `CustomerGroup` (E37).
* **SA05 – Shopping (E38 – E39):** `Cart` (E38), `CartItem` (E39).
* **SA06 – Sales & Order Management (E40 – E43):** `SalesOrder` (E40), `OrderLine` (E41), `OrderAddress` (E42), `OrderPromotion` (E43).
* **SA07 – Fulfillment & Shipping (E44 – E48):** `Shipment` (E44), `Carrier` (E45), `ShipmentLine` (E46), `ShipmentStatusHistory` (E47), `DeliveryReturnReceipt` (E48).
* **SA08 – Promotion & Voucher (E49 – E54):** `Promotion` (E49), `Voucher` (E50), `PromotionRule` (E51), `PromotionReward` (E52), `PromotionUsage` (E53), `VoucherCustomer` (E54).
* **SA09 – Payment & Settlement (E55 – E59, E71):** `Payment` (E55), `PaymentTransaction` (E56), `Refund` (E57), `RefundTransaction` (E58), `RefundAllocation` (E59), `PaymentWebhookLog` (E71).
* **SA10 – Return & Exchange (E60 – E64, E72):** `ReturnRequest` (E60), `ReturnRequestLine` (E61), `ReturnReceipt` (E62), `ReturnReceiptLine` (E63), `ReturnShipmentAllocation` (E64b), `ReturnLineAssessment` (E72).
* **SA11 – Audit & Reporting (E65 – E70):** `AuditLog` (E65), `SystemConfig` (E66), `Notification` (E67), `NotificationTemplate` (E68), `DailySalesSummary` (E69), `InventoryDailySnapshot` (E70).

### 3.2. Danh mục Chỉ dẫn Sơ đồ Mermaid ERD & Ảnh Render Đã Có Sẵn
Toàn bộ sơ đồ ERD được tổ chức độc lập tại thư mục `docs/Milestones/M2-Conceptual-Design/ERD/` với cú pháp chuẩn `PK`, `FK`, `UK` và có ảnh PNG render sẵn, bảo đảm extension markdown mở file nhanh chóng:

* **Sơ đồ Khung xương Liên miền:** [Cross-Domain-Backbone.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Cross-Domain-Backbone.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Cross-Domain-Backbone.png)
* **SA01 Product Catalog:** [Product-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Product-Catalog.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Product-Catalog.png)
* **SA02 Warehouse & Inventory:** [Warehouse-Inventory.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Warehouse-Inventory.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Warehouse-Inventory.png)
* **SA03 Procurement:** [Procurement.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Procurement.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Procurement.png)
* **SA04 Customer & Identity:** [Customer-Identity.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Customer-Identity.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Customer-Identity.png)
* **SA05 Shopping:** [Shopping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Shopping.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Shopping.png)
* **SA06 Sales & Order:** [Sales.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Sales.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Sales.png)
* **SA07 Fulfillment & Shipping:** [Fulfillment-Shipping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Fulfillment-Shipping.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Fulfillment-Shipping.png)
* **SA08 Promotion & Voucher:** [Promotion-Voucher.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Promotion-Voucher.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Promotion-Voucher.png)
* **SA09 Payment & Settlement:** [Payment-Refund.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Payment-Refund.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Payment-Refund.png)
* **SA10 Return & Exchange:** [Return-Exchange.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Return-Exchange.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Return-Exchange.png)
* **SA11 Audit & Reporting:** [Audit-Reporting.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Audit-Reporting.md) | [Ảnh PNG](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Audit-Reporting.png)

### 3.3. Phản biện kiến trúc (M2.07) & hiệu chỉnh M2.08
Trong giai đoạn cuối M2, đội ngũ kỹ thuật đã tiến hành đợt rà soát phản biện kiến trúc độc lập [07-Pre-M3-Architectural-Review.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/07-Pre-M3-Architectural-Review.md) theo 6 tiêu chuẩn kỹ thuật (Entity Ownership, Cross-Area Relations, Cardinality, Business Rule Consistency, Lifecycle Preservation, Completeness).

Đợt rà soát đã phát hiện ra các khiếm khuyết tiềm ẩn nếu chuyển mô hình hiện tại sang M3:
* **Lỗi logic hoàn tiền:** Quy định cũ gắn chặt việc hoàn tiền vào trạng thái `SELLABLE` của kho, làm tước đoạt quyền lợi của khách hàng khi hàng bị lỗi do nhà sản xuất (áo rách phải tiêu hủy `DISPOSED`).
* **Lỗi ngữ nghĩa tồn kho:** Sử dụng trạng thái tiêu hủy `DISPOSED` như một trạng thái tồn kho là sai bản chất, vì hàng tiêu hủy đã rời kho.
* **Lỗi phân bổ chiết khấu:** Chiết khấu voucher chia tỷ lệ phần trăm thông thường gây ra sai số làm tròn phân số đồng lẻ khi chia cho tiền tệ VNĐ.
* **Lỗi ràng buộc 1:1 trong SQL Server:** SQL Server không có Deferrable FK, nếu khai báo `FOREIGN KEY` thông thường sẽ gây ra bế tắc "con gà - quả trứng" khi khởi tạo đồng thời đơn hàng và địa chỉ.

Để khắc phục, đội ngũ đã ban hành tài liệu chính thức [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md) với **10 Hiệu chỉnh Kiến trúc (CORR-01 đến CORR-10)**:
1. **CORR-01 (P0):** Tách biệt quyền hoàn tiền (`RefundEligibilityStatus`) khỏi định đoạt vị trí kho (`InventoryDisposition`). Bổ sung thực thể mới `ReturnLineAssessment` (E72) giám định theo số lượng thực tế.
2. **CORR-02 (P0):** Phân định 3 trạng thái chất lượng tồn kho (`SELLABLE`, `QUARANTINED`, `DAMAGED`) và nghiệp vụ xuất tiêu hủy (`SCRAP_OUT`). Thiết lập hệ thống 5 ràng buộc `CHECK` toán học trên `InventoryBalance` (E11).
3. **CORR-03 (P0):** Ứng dụng thuật toán **Hare-Niemeyer (LRM)** trên tập dòng đủ điều kiện (`EligibleBase`) và hàm nguyên tích lũy $F(q)$ triệt tiêu hoàn toàn sai số phân số đồng lẻ VNĐ.
4. **CORR-04 (P1):** Thực thi ràng buộc bắt buộc 1:1 qua Stored Procedure giao dịch nguyên tử (`SET XACT_ABORT ON; BEGIN TRANSACTION ... COMMIT`) và thu hồi quyền DML trực tiếp.
5. **CORR-05 (P1):** Chuẩn hóa cấu trúc Table-Per-Type (TPT) giữa `Account` và `Customer`/`Employee` với ràng buộc loại trừ XOR.
6. **CORR-06 (P1):** Bổ sung thực thể `PaymentWebhookLog` (E71) lưu trữ bền vững trước khi trả HTTP 200, kiểm soát Idempotency chống tấn công Replay.
7. **CORR-07 (P1):** Độc lập 4 máy trạng thái $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$; loại bỏ `RMA_PROCESSING` khỏi `OrderStatus`.
8. **CORR-08 (P1):** Phân định nghĩa vụ thanh toán của khách với công nợ đối soát của hãng vận chuyển.
9. **CORR-09 (P2):** Định giá xuất trả NCC theo giá vốn bình quân di động (WAC) hiện tại, hạch toán chênh lệch vào tài khoản điều chỉnh giá vốn.
10. **CORR-10 (P2):** Quản lý giữ hàng đa kho $1 : 0..N$ qua Filtered Unique Index trên `AllocationID WHERE ReservationStatus = 'ACTIVE'`.

---

## 4. GIAI ĐOẠN CHUYỂN TIẾP: PRE-M3 GATE RESOLUTION

Để việc chuyển dịch sang Milestone 3 không bị vướng mắc hoặc tranh cãi trong quá trình ánh xạ quan hệ, toàn bộ 10 vấn đề kiến trúc mở (Open Issues B1–B7, Giả định C1–C4, Rủi ro D1–D4) đã được tháo gỡ triệt để và chốt DDL mẫu:

### Bảng Giải quyết Triệt để 10 Vấn đề Kiến trúc Tiền M3:

| Mã Issue | Tên Vấn đề Kiến trúc | Quyết định Kỹ thuật & Đặc tả Schema Đã Chốt |
| :---: | :--- | :--- |
| **Issue 1** | Truy vết Đổi trả Bưu kiện Đa kiện Lệch ngày (B1) | Thiết kế thực thể kết hợp `ReturnShipmentAllocation` nối `ReturnRequestLine` (E61) và `ShipmentLine` (E46). Hạn đổi trả tính theo `Shipment.DeliveredAt`. Ràng buộc: $\sum \text{AllocatedReturnQty} \le \text{ShipmentQty} - \text{PreviouslyReturnedQty}$. |
| **Issue 2** | Đối soát COD trong Đơn hàng Split Shipment (B2) | Chuẩn hóa mô hình 3 lớp: `CODCollection` (Khách trả shipper theo kiện $\to$ Đơn cập nhật `PARTIALLY_PAID`) $\to$ `CODRemittance` (Hãng chuyển trả bưu cục) $\to$ `CODReconciliation` (Khớp đối soát chênh lệch). |
| **Issue 3** | Chính sách Chuẩn hóa 3NF / BCNF Thực tiễn (B3) | Ban hành Controlled Normalization Policy: 100% bảng đạt 3NF. Giữ lại 3NF không phân rã lên BCNF cho các bảng giao dịch bất biến (`OrderLine.UnitPriceSnapshot`) kèm văn bản giải trình kỹ thuật bảo toàn phụ thuộc hàm. |
| **Issue 4** | Thống nhất Kiểu Dữ liệu Tiền tệ VNĐ (B4) | Dùng `DECIMAL(19,0)` cho tiền giao dịch nguyên số đồng; `DECIMAL(19,4)` cho đơn giá, tỷ lệ chiết khấu, giá vốn bình quân WAC. Tuyệt đối cấm dùng `MONEY` hay `FLOAT`. |
| **Issue 5** | Audit Trail Giám định `ReturnLineAssessment` (B5) | Phiên bản hóa `ReturnLineAssessment` (E72) với `IsCurrent`, `AssessmentVersion`, `AssessmentStatus` kèm Filtered Unique Index bảo đảm duy nhất 1 bản ghi hiện hành, cấm sửa đè lịch sử. |
| **Issue 6** | Xác nhận Định giá Tồn kho WAC (B6 / C1) | `CurrentAverageCost` là giá vốn bình quân di động của toàn bộ `OnHandQty` của SKU tại từng kho; tính toán tự động sau mỗi lần nhập kho `GoodsReceipt`. |
| **Issue 7** | Triệt tiêu Sai số Làm tròn LRM & $F(q)$ (B7 / D1) | Thuật toán LRM Hare-Niemeyer trên `EligibleBase` + Hàm tích lũy nguyên $F(q)$ bảo đảm tổng số tiền hoàn đúng bằng tổng giá trị thực trả, triệt tiêu sai số làm tròn phân số. |
| **Issue 8** | Tích hợp Policy Engine Phí Vận chuyển (C2) | Phí vận chuyển được tách thành trường `ShippingRefundAmount` độc lập với tiền hàng, ủy quyền cho Policy Engine bên ngoài tính toán theo chính sách linh hoạt. |
| **Issue 9** | Tài khoản Hạch toán Chênh lệch PPV/WAC (C3) | Bổ sung trường `CostVarianceAccountCode` (TK 632 / TK 156 chênh lệch) để hạch toán chênh lệch giá xuất trả NCC so với giảm giá trị tồn kho WAC. |
| **Issue 10** | Lưu trữ Webhook Bền vững & Idempotency (C4 / D3) | Bảng `PaymentWebhookLog` có khóa `UNIQUE(ProviderID, MerchantAccountID, ProviderEventID, Environment)`; lưu bền vững trước khi phản hồi HTTP 200. |

---

## 5. KẾ HOẠCH & LỘ TRÌNH TRIỂN KHAI MILESTONE 3: THIẾT KẾ CƠ SỞ DỮ LIỆU MỨC LOGIC (M3)

Không bắt đầu bằng việc viết DDL vội vã, Milestone 3 sẽ tiến hành chuyển đổi toán học nghiêm ngặt theo **4 công đoạn chuẩn mực**:

```
docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/
├── 01-Mapping-Methodology.md          # Quy tắc ánh xạ thực thể sang quan hệ
├── 02-Entity-to-Relation-Mapping.md   # Ánh xạ chi tiết 72 thực thể thành các bảng
├── 03-Primary-and-Candidate-Keys.md  # Surrogate Keys, Natural Keys & Candidate Keys
├── 04-Foreign-Key-Mapping.md          # Khóa ngoại và hành vi toàn vẹn ON DELETE NO ACTION
├── 05-Relationship-Transformation.md  # Xử lý 1:1, 1:N, N:N, TPT Subtypes, Cây đệ quy
├── 06-Cross-Area-Relations.md         # Kiểm soát 40 liên kết liên vùng nghiệp vụ
├── 07-Open-Design-Decisions.md        # Tích hợp COD 3 lớp và ReturnShipmentAllocation
└── 08-Mapping-Validation.md           # Kiểm chứng chéo và nghiệm thu chuyển giao M3.02
```

### Chi tiết 4 Công đoạn của Milestone 3:
1. **Bước 1 (M3.01) – Ánh xạ Lược đồ Quan hệ (Relational Schema Mapping):**
   * Chuyển đổi 72 thực thể quan niệm thành các bảng quan hệ tương thích SQL Server, bổ sung các bảng giao vận đa kiện (`ReturnShipmentAllocation`) và bảng đối soát COD 3 lớp.
   * *Trình tự ưu tiên:* Bắt đầu ngay từ **Ưu tiên 1: `SA01 – Product Catalog` (E01 – E09)** do tính độc lập cao, tiếp sau đó là `SA02 – Warehouse & Inventory`.
2. **Bước 2 (M3.02) – Chuẩn hóa Dữ liệu Toán học (3NF & BCNF Evaluation):**
   * Lập danh sách Khóa ứng viên (Candidate Keys) và tập Phụ thuộc hàm đầy đủ (FDs) cho từng bảng. Đánh giá tính chuẩn 1NF, 2NF, 3NF và BCNF.
   * Áp dụng Controlled Normalization Policy: 100% bảng đạt 3NF; lập văn bản giải trình kỹ thuật (Technical Trade-off) cho các bảng giữ 3NF không phân rã lên BCNF (như snapshot giá bán và VAT).
3. **Bước 3 (M3.03) – Định kiểu Dữ liệu Vật lý SQL Server & Dự phóng Dung lượng (Sizing):**
   * Quy định chi tiết kiểu dữ liệu từng cột (Tiền tệ `DECIMAL(19,0)`, đơn giá `DECIMAL(19,4)`, Surrogate Key `BIGINT IDENTITY`, chuỗi `NVARCHAR(len)`, thời gian `DATETIMEOFFSET(7)`).
   * Lập bảng tính toán dung lượng 3 năm (Capacity Planning): Tính kích thước dòng (Row Size), dung lượng bảng (Data Space) và dung lượng chỉ mục (Index Space) dựa trên tải 1.000.000 đơn hàng/năm và 10.000.000 biến động kho/năm.
4. **Bước 4 (M3.04) – Đặc tả Chi tiết Ràng buộc Kỹ thuật & Stored Procedures:**
   * Viết mã nguồn DDL cho toàn bộ các ràng buộc khai báo (`PK`, `FK ON DELETE NO ACTION`, `CHECK`, `DEFAULT`).
   * Thiết kế chữ ký và kịch bản thực thi cho 5 Stored Procedures giao dịch cốt lõi:
     1. `usp_CreateSalesOrderWithAddress` (Tạo đơn hàng và snapshot địa chỉ 1:1).
     2. `usp_AllocateAndReserveStock` (Phân bổ đa kho và khóa giữ tồn kho).
     3. `usp_PostInventoryTransaction` (Ghi sổ cái biến động kho và cập nhật số dư nguyên tử).
     4. `usp_ReceivePaymentWebhook` (Lưu vết webhook bền vững và xử lý bất đồng bộ).
     5. `usp_AssessReturnReceiptLine` (Phiên bản hóa biên bản giám định KCS E72).

---

## 6. ĐỊNH HƯỚNG CÁC MILESTONE TIẾP THEO (M4 ĐẾN M8)

* **M4: Thiết kế CSDL Mức Vật lý & Tối ưu Hiệu năng (Physical Design):**
  * Phân vùng bảng (Table Partitioning) theo thời gian cho các bảng giao dịch khổng lồ (`SalesOrder`, `InventoryTransactionLine`, `AuditLog`).
  * Chỉ mục chuyên sâu: Filtered Indexes, Clustered Columnstore Indexes cho bảng snapshot và bảng hạch toán BI, Covering Indexes cho các truy vấn then chốt.
  * Quy hoạch Filegroups vật lý tách biệt Data, Index và Blob/Log; thiết lập giải pháp sẵn sàng cao (Always On Availability Groups).
* **M5: DDL Triển khai & Xây dựng Backend:**
  * Script T-SQL triển khai Schema, tích hợp công cụ quản lý phiên bản Flyway Migration.
  * Bộ dữ liệu Seed/Mock 1.000.000 dòng để thử nghiệm tải thực tế.
  * Xây dựng tầng dịch vụ Java Spring Boot (Spring Data JPA / JDBC Client, RESTful APIs, Spring Security).
* **M6: Xây dựng Giao diện Frontend:**
  * Ứng dụng Web Single Page Application (SPA) bằng React, TypeScript và Tailwind CSS.
  * Giao diện điều hành phân hệ cho Admin, nhân viên kho, nhân viên bán hàng và cổng mua sắm khách hàng.
* **M7: Kiểm thử & Đảm bảo Chất lượng (Testing & QA):**
  * Kiểm thử chịu tải đồng thời (Concurrency Stress Test) chống bán âm kho Flash Sale.
  * Kiểm thử toàn vẹn giao dịch ACID, kiểm tra ranh giới deadlock và race condition.
* **M8: Đóng gói & Triển khai Vận hành (DevOps & Deployment):**
  * Đóng gói Docker, Docker Compose cho toàn bộ hệ sinh thái (SQL Server, Spring Boot, React).
  * Xây dựng CI/CD pipeline tự động hóa kiểm thử và triển khai; tích hợp hệ thống giám sát Prometheus và Grafana.

---

## 7. BIÊN BẢN NGHIỆM THU CỔNG CHUYỂN GIAO (GATE SIGN-OFF DECISION)

```
========================================================================================
            GATE REVIEW SIGN-OFF: READINESS FOR MILESTONE 3 (LOGICAL DESIGN)
========================================================================================
Dự án:                 RetailCore Enterprise E-Commerce Management System
Cơ sở dữ liệu:         Microsoft SQL Server Enterprise Edition
Giai đoạn:             Chuyển giao từ Milestone 2 (M2) sang Milestone 3 (M3)
Đánh giá Kỹ thuật:     HOÀN TẤT VÀ CHUẨN HÓA 100% TIỀN ĐỀ M1 & M2
Quyết định Cổng:       CONDITIONAL APPROVAL FOR M3.01 (Phê duyệt Khởi động M3.01)
----------------------------------------------------------------------------------------
ĐIỀU KIỆN THỰC THI:
1. ĐƯỢC PHÉP KHỞI ĐỘNG NGAY:
   - Tiến hành công đoạn M3.01 (Relational Schema Mapping) bắt đầu từ SA01 - Product Catalog.
2. NGUYÊN TẮC QUẢN TRỊ KIẾN TRÚC:
   - Cấu trúc bảng vật lý được quyết định dựa trên yêu cầu chuẩn hóa, lưu vết kiểm toán và 
     tối ưu truy vấn (không ép buộc cứng nhắc 72 bảng).
   - Áp dụng Controlled Normalization Policy (100% đạt 3NF, giải trình kỹ thuật khi giữ 3NF).
3. ĐỒNG BỘ NGƯỢC (REVERSE-SYNC):
   - Mọi điều chỉnh phát sinh trong M3 sẽ được cập nhật ngược lại tài liệu M2 để bảo đảm
     tính nhất quán xuyên suốt vòng đời dự án.
========================================================================================
```

*(Tài liệu này là Báo cáo Tiến trình Xây dựng và Tiến độ Duy nhất của dự án RetailCore, lưu trữ chính thức tại `docs/Project-Progress-Report.md`).*
