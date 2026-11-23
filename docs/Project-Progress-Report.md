# BÁO CÁO TIẾN ĐỘ DỰ ÁN RETAILCORE & THUYẾT MINH KIẾN TRÚC HỆ THỐNG
## PROJECT PROGRESS REPORT & ENTERPRISE ARCHITECTURAL BASELINE

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Quy mô Dự phóng:** 1.000.000 đơn hàng/năm, 10.000.000 biến động kho/năm, tính sẵn sàng 99.99%
* **Phiên bản Báo cáo:** v3.0 (Tài liệu Báo cáo Tiến độ Hợp nhất Duy nhất Toàn Dự án)
* **Trạng thái Cổng M2:** **CONDITIONAL APPROVAL FOR M3 (Nghiệm thu Chuyển giao Có Điều kiện)**
* **Trạng thái Tiền M3 (Pre-M3 Issues):** **RESOLVED & APPROVED 100% (Đã Giải quyết Toàn bộ 10 Vấn đề Kiến trúc)**
* **Phê duyệt:** Lead Enterprise Database Engineer & Solution Architect

---

## 1. TỔNG QUAN TIẾN ĐỘ DỰ ÁN (PROJECT PROGRESS OVERVIEW)

### 1.1. Triết lý Quản trị Kiến trúc và Nguyên tắc Tiếp cận
RetailCore được xây dựng theo định hướng tiếp cận **Database-First / Database-Centric** dành cho Hệ thống Thông tin Doanh nghiệp (Enterprise Information System). Dự án đặt CSDL làm trung tâm bảo toàn tính toàn vẹn nghiệp vụ, kiểm soát truy cập đồng thời và bảo vệ các giao dịch tài chính - kho vận (ACID).

1. **M2 Baseline là nền tảng vững chắc:** Xác lập ranh giới nghiệp vụ (Bounded Contexts), định danh thực thể dữ liệu cốt lõi và quan hệ bản số liên miền.
2. **Pre-M3 Issues đã được tháo gỡ triệt để:** Toàn bộ 10 vướng mắc kỹ thuật phức tạp (từ truy vết RMA đa kiện, đối soát COD 3 lớp, audit trail giám định đến triệt tiêu sai số làm tròn VNĐ) đã được chốt giải pháp và DDL mẫu.
3. **M3 (Logical Design) là giai đoạn chuẩn hóa toán học:** Chuyển đổi mô hình quan niệm sang mô hình quan hệ toán học (Candidate Keys, Functional Dependencies, 3NF/BCNF, Physical Data Types, Declarative Constraints & Core Stored Procedures).
4. **Tối ưu hóa hiệu năng tài liệu:** Toàn bộ các sơ đồ Mermaid ERD được tách thành file độc lập kèm ảnh PNG render sẵn, giúp trình xem file của IDE/Extension mở ngay lập tức mà không gặp hiện tượng giật lag.

---

### 1.2. Bảng theo dõi tiến độ tổng thể các Cột mốc (Milestone Scorecard)

| Cột mốc (Milestone) | Trọng số | Tiến độ Thực tế | Trạng thái Kỹ thuật | Kết quả Nghiệm thu & Bàn giao |
| :--- | :---: | :---: | :---: | :--- |
| **M1: Phân tích Nghiệp vụ & Yêu cầu** *(Business Analysis)* | 20% | **100%** | **HOÀN THÀNH (Baseline Signed Off)** | • 10 Phân hệ nghiệp vụ cốt lõi.<br>• 112 Quy tắc nghiệp vụ (`BR-01` đến `BR-112`) được mã hóa.<br>• Đóng gói 11 tài liệu đặc tả chuẩn mực (Project Charter, Stakeholder Register, Use Cases, Process Flows, Traceability Matrix). |
| **M2: Thiết kế Cơ sở Dữ liệu Mức Quan niệm** *(Conceptual Design)* | 30% | **100%** | **NGHIỆM THU CÓ ĐIỀU KIỆN (Conditional Approval)** | • 72 Thực thể quan niệm (**E01 – E72**) chuẩn hóa trên 11 Vùng nghiệp vụ (**SA01 – SA11**).<br>• 12 Lược đồ Mermaid ERD cú pháp chuẩn `PK`, `FK`, `UK` và 12 ảnh PNG render tương ứng.<br>• Ban hành Bản ghi hiệu chỉnh `M2.08` xử lý 10 lỗi gốc (CORR-01..10). |
| **Giai đoạn Tiền M3 (Pre-M3 Gate Resolution)** | — | **100%** | **ĐÃ GIẢI QUYẾT 100% (Fully Resolved)** | • Giải quyết dứt điểm 10 Open Issues kỹ thuật (B1–B7, C1–C4, D1–D4).<br>• Đặc tả DDL schema cho `ReturnShipmentAllocation`, COD 3 lớp, Audit Trail E72.<br>• Ban hành chính sách chuẩn hóa 3NF/BCNF thực tế và chuẩn kiểu tiền tệ VNĐ. |
| **M3: Thiết kế Cơ sở Dữ liệu Mức Logic** *(Logical Design)* | 25% | **0%** | **KHỞI ĐỘNG (Starting M3.01)** | • Khởi động công đoạn M3.01 (Relational Schema Mapping) cho `SA01 – Product Catalog`.<br>• Kế hoạch 4 bước: Ánh xạ quan hệ $\to$ Chuẩn hóa 3NF/BCNF $\to$ Định cỡ dữ liệu Sizing $\to$ DDL Constraints & Stored Procedures. |
| **M4: Thiết kế Vật lý & Tối ưu Hiệu năng** *(Physical Design)* | 15% | **0%** | **CHỜ THỰC HIỆN (Pending)** | • Partitioning Scheme theo quý/năm.<br>• Chỉ mục chuyên sâu (Filtered, Columnstore, Covering).<br>• Kế hoạch sao lưu phục hồi (HA/DR, Availability Groups). |
| **M5: DDL Triển khai & Kiểm thử Toàn diện** | 10% | **0%** | **CHỜ THỰC HIỆN (Pending)** | • Script T-SQL tạo Schema hoàn chỉnh.<br>• Bộ dữ liệu Seed/Mock 1.000.000 dòng kiểm thử tải.<br>• Kiểm thử tự động tính toàn vẹn (Stress test / Concurrency). |
| **M6: Frontend (React / TypeScript)** | — | **0%** | **CHỜ THỰC HIỆN (Pending)** | • Giao diện Web App điều hành & bán hàng đa kênh. |
| **M7: Testing & Debugging** | — | **0%** | **CHỜ THỰC HIỆN (Pending)** | • Stress test, Integration test, Concurrency test. |
| **M8: Đóng gói & Triển khai (CI/CD / Docker)** | — | **0%** | **CHỜ THỰC HIỆN (Pending)** | • Docker hóa, CI/CD pipeline, Monitoring. |

---

## 2. CHI TIẾT CỘT MỐC M2: 72 THỰC THỂ & 11 VÙNG NGHIỆP VỤ (SUBJECT AREAS)

Mô hình dữ liệu khái niệm bao phủ toàn diện 72 thực thể (E01 – E72) trên 11 Subject Areas, đảm bảo phân ranh giới ngữ cảnh (Bounded Contexts) độc lập và chính xác:

### 2.1. Danh mục 72 Thực thể Quan niệm (E01 – E72)
1. **SA01 – Product Catalog (Quản lý Danh mục & Biến thể Sản phẩm):**
   * **Thực thể (E01 – E09):** `Category` (E01), `Product` (E02), `ProductCategory` (E03), `ProductVariant` (E04 - Hạt nhân SKU thương mại), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09).
   * **Đặc điểm:** Tách biệt sản phẩm tổng thể (`Product`) và đơn vị lưu kho biến thể (`ProductVariant`). Quản lý cây danh mục đa cấp đệ quy và ma trận thuộc tính 2 lớp (`DESCRIPTIVE` vs `VARIANT_DEFINING`).
2. **SA02 – Warehouse & Inventory (Kho bãi & Quản lý Tồn kho Đa kho):**
   * **Thực thể (E10 – E20):** `Warehouse` (E10), `InventoryBalance` (E11 - Sổ số dư kho), `OrderWarehouseAllocation` (E13 - Cam kết kế hoạch phân bổ kho), `StockReservation` (E14 - Chứng từ giữ hàng), `StockReservationLine` (E15), `InventoryTransaction` (E16 - Sổ cái biến động kho), `InventoryTransactionLine` (E17), `StockCount` (E18 - Phiếu kiểm kê), `StockCountLine` (E19), `WarehouseTransfer` (E20 - Chuyển kho 2 bước), `WarehouseTransferLine` (E12).
   * **Đặc điểm:** Tách bạch 4 khái niệm (Phân bổ, Giữ hàng, Giao hàng, Ghi sổ); kiểm soát số dư tồn kho đa kho cho Split Shipment.
3. **SA03 – Procurement (Chuỗi Cung ứng & Mua hàng):**
   * **Thực thể (E21 – E28):** `Supplier` (E21), `SupplierContact` (E22), `SupplierProduct` (E23), `PurchaseOrder` (E24 - Đơn PO), `PurchaseOrderLine` (E25), `GoodsReceipt` (E26 - Biên bản tiếp nhận nhập kho), `GoodsReceiptLine` (E27), `SupplierReturn` (E28 - Phiếu xuất trả NCC), `SupplierReturnLine` (Kế thừa E28b).
   * **Đặc điểm:** PO chỉ định đúng 1 kho nhận; cho phép nhận hàng nhiều đợt (Partial Receipt); phân tách hàng đạt và hàng lỗi (`AcceptedQty` vs `RejectedQty`).
4. **SA04 – Customer & Identity (Khách hàng & Quản lý Phân quyền):**
   * **Thực thể (E29 – E37):** `Account` (E29 - Hạt nhân định danh), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `CustomerGroup` (E37).
   * **Đặc điểm:** Kiến trúc Table-Per-Type (TPT) giữa `Account` và `Customer`/`Employee` với ràng buộc loại trừ XOR.
5. **SA05 – Shopping (Trải nghiệm Mua sắm & Giỏ hàng):**
   * **Thực thể (E38 – E39):** `Cart` (E38), `CartItem` (E39).
   * **Đặc điểm:** Quản lý giỏ hàng trực tuyến theo session/customer; chuyển đổi nguyên tử sang dòng đơn hàng khi checkout.
6. **SA06 – Sales & Order Management (Bán hàng & Đơn hàng):**
   * **Thực thể (E40 – E43):** `SalesOrder` (E40 - Đơn hàng hạt nhân), `OrderLine` (E41), `OrderAddress` (E42 - Snapshot địa chỉ bất biến), `OrderPromotion` (E43).
   * **Đặc điểm:** Snapshot 100% giá bán, thuế VAT, địa chỉ giao hàng tại thời điểm đặt; độc lập với việc thay đổi dữ liệu sau đó.
7. **SA07 – Fulfillment & Shipping (Đóng gói & Giao vận):**
   * **Thực thể (E44 – E48):** `Shipment` (E44 - Bưu kiện thực tế xuất từ 1 kho), `Carrier` (E45), `ShipmentLine` (E46), `ShipmentStatusHistory` (E47), `DeliveryReturnReceipt` (E48).
   * **Đặc điểm:** Hỗ trợ tách đơn thành nhiều kiện xuất từ nhiều kho (Split Shipment); xử lý giao hàng thất bại quay đầu độc lập với kho.
8. **SA08 – Promotion & Voucher (Khuyến mãi & Mã giảm giá):**
   * **Thực thể (E49 – E54):** `Promotion` (E49), `Voucher` (E50), `PromotionRule` (E51), `PromotionReward` (E52), `PromotionUsage` (E53), `VoucherCustomer` (E54).
   * **Đặc điểm:** Kế thừa Promotion $\to$ Voucher; kiểm soát áp dụng khuyến mãi đa tầng và mã giảm giá cá nhân hóa.
9. **SA09 – Payment & Settlement (Thanh toán & Đối soát Cổng):**
   * **Thực thể (E55 – E59, E71):** `Payment` (E55 - Hồ sơ nghĩa vụ thanh toán đơn), `PaymentTransaction` (E56 - Giao dịch cổng), `Refund` (E57 - Chứng từ hoàn tiền), `RefundTransaction` (E58), `RefundAllocation` (E59), `PaymentWebhookLog` (E71 - Nhật ký Webhook).
   * **Đặc điểm:** Quan hệ 1:1 với Đơn hàng; 1:N với các lần thanh toán thử; lưu trữ bền vững Webhook chống tấn công Replay.
10. **SA10 – Return & Exchange (Hậu cần Ngược, Đổi trả & Giám định):**
    * **Thực thể (E60 – E64, E72):** `ReturnRequest` (E60 - Yêu cầu trả hàng), `ReturnRequestLine` (E61), `ReturnReceipt` (E62 - Phiếu tiếp nhận hàng trả), `ReturnReceiptLine` (E63), `ReturnShipmentAllocation` (Kế thừa E64b), `ReturnLineAssessment` (E72 - Giám định KCS phân định lỗi).
    * **Đặc điểm:** Tách biệt quyền hoàn tiền khỏi định đoạt vị trí kho; giám định chi tiết từng sản phẩm theo phiên bản kiểm toán.
11. **SA11 – Audit & Reporting (Kiểm toán & Báo cáo Quản trị BI):**
    * **Thực thể (E65 – E70):** `AuditLog` (E65), `SystemConfig` (E66), `Notification` (E67), `NotificationTemplate` (E68), `DailySalesSummary` (E69), `InventoryDailySnapshot` (E70).
    * **Đặc điểm:** Lưu vết kiểm toán bất biến; tổng hợp số liệu theo ngày phục vụ dashboard điều hành tốc độ cao.

---

### 2.2. Danh mục Chỉ dẫn Tra cứu Sơ đồ Mermaid ERD & Ảnh Render Đã Có Sẵn
Để bảo đảm tốc độ mở và đọc file tức thì trên mọi extension markdown, toàn bộ sơ đồ cấu trúc đã được tách thành các file chuyên biệt có thể tra cứu trực tiếp:

| STT | Phân hệ nghiệp vụ | File Mã nguồn Mermaid ERD | File Ảnh Render Trực quan (PNG) |
| :---: | :--- | :--- | :--- |
| **00** | **Khung xương Liên miền (Backbone)** | [Cross-Domain-Backbone.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Cross-Domain-Backbone.md) | [Cross-Domain-Backbone.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Cross-Domain-Backbone.png) |
| **01** | **Product Catalog** | [Product-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Product-Catalog.md) | [Product-Catalog.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Product-Catalog.png) |
| **02** | **Warehouse & Inventory** | [Warehouse-Inventory.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Warehouse-Inventory.md) | [Warehouse-Inventory.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Warehouse-Inventory.png) |
| **03** | **Procurement** | [Procurement.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Procurement.md) | [Procurement.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Procurement.png) |
| **04** | **Customer & Identity** | [Customer-Identity.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Customer-Identity.md) | [Customer-Identity.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Customer-Identity.png) |
| **05** | **Shopping** | [Shopping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Shopping.md) | [Shopping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Shopping.png) |
| **06** | **Sales & Order Management** | [Sales.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Sales.md) | [Sales.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Sales.png) |
| **07** | **Fulfillment & Shipping** | [Fulfillment-Shipping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Fulfillment-Shipping.md) | [Fulfillment-Shipping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Fulfillment-Shipping.png) |
| **08** | **Promotion & Voucher** | [Promotion-Voucher.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Promotion-Voucher.md) | [Promotion-Voucher.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Promotion-Voucher.png) |
| **09** | **Payment & Settlement** | [Payment-Refund.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Payment-Refund.md) | [Payment-Refund.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Payment-Refund.png) |
| **10** | **Return & Exchange** | [Return-Exchange.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Return-Exchange.md) | [Return-Exchange.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Return-Exchange.png) |
| **11** | **Audit & Reporting** | [Audit-Reporting.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Audit-Reporting.md) | [Audit-Reporting.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Audit-Reporting.png) |

---

## 3. HỆ THỐNG RÀNG BUỘC KỸ THUẬT & QUY TẮC TOÀN VẸN (TECHNICAL CONSTRAINTS BASELINE)

Hệ thống RetailCore thực thi tính toàn vẹn dữ liệu thông qua các bất đẳng thức toán học và ràng buộc CSDL nghiêm ngặt:

### 3.1. Ràng buộc Số dư Tồn kho & 5 CHECK Constraints Toán học (CORR-02)
Trên thực thể số dư kho `InventoryBalance` (E11), dữ liệu được kiểm soát bởi hệ phương trình cân bằng:
$$\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$$
$$\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty} \ge 0$$

5 Ràng buộc toàn vẹn khai báo (Declarative CHECK Constraints):
1. `CK_InventoryBalance_OnHand_Positive`: $\text{OnHandQty} \ge 0$
2. `CK_InventoryBalance_Quarantined`: $0 \le \text{QuarantinedQty} \le \text{OnHandQty}$
3. `CK_InventoryBalance_Damaged`: $0 \le \text{DamagedQty} \le \text{OnHandQty}$
4. `CK_InventoryBalance_TotalDefect`: $\text{QuarantinedQty} + \text{DamagedQty} \le \text{OnHandQty}$
5. `CK_InventoryBalance_Reserved`: $0 \le \text{ReservedQty} \le (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty})$

*Quy tắc biến động chất lượng:* Thống nhất 3 trạng thái chất lượng tồn kho (`SELLABLE`, `QUARANTINED`, `DAMAGED`) và loại nghiệp vụ xuất tiêu hủy (`SCRAP_OUT`). Cấm tuyệt đối đưa `RESERVED` vào trạng thái chất lượng hàng hóa.

### 3.2. Quản lý Giữ hàng Đa kho $1 : 0..N$ qua Filtered Unique Index (CORR-10)
Cho phép một dòng đơn hàng phân bổ và giữ hàng tại nhiều kho khác nhau (Split Allocation). Mỗi phân bổ có thể trải qua nhiều lượt giữ hàng lịch sử, nhưng tại một thời điểm chỉ có tối đa duy nhất 1 lượt giữ hàng có trạng thái `ACTIVE`:
```sql
CREATE UNIQUE NONCLUSTERED INDEX UQ_StockReservationLine_ActiveAllocation 
ON dbo.StockReservationLine (AllocationID) 
WHERE ReservationStatus = 'ACTIVE';
```

### 3.3. Thuật toán Phân bổ Giảm giá LRM & Triệt tiêu Sai số Làm tròn VNĐ (CORR-03 / Issue 7)
* Áp dụng phương pháp **Hare-Niemeyer (Largest Remainder Method - LRM)** phân bổ chiết khấu voucher trên tập các dòng sản phẩm đủ điều kiện (`EligibleBase`) với quy tắc phá hòa xác định (`OrderLineID ASC`).
* **Chuẩn hóa Kiểu Dữ liệu Tiền tệ:** Tiền tệ giao dịch VNĐ là số nguyên đồng sử dụng `DECIMAL(19,0)` (hoặc `BIGINT`). Đơn giá, tỷ lệ chiết khấu, giá vốn bình quân WAC sử dụng `DECIMAL(19,4)`. Tuyệt đối không dùng `MONEY` hoặc `FLOAT`.
* Khi hoàn tiền một phần (Partial Return), số tiền hoàn lũy tiến theo số lượng được tính bằng hàm nguyên xác định $F(q)$ bảo đảm khi hoàn tất cả số lượng thì tổng số tiền hoàn đúng bằng tổng giá trị ròng thực trả, triệt tiêu hoàn toàn sai số phân số đồng lẻ.

### 3.4. Thực thi Quan hệ Bắt buộc 1:1 trong SQL Server (CORR-04)
Microsoft SQL Server không hỗ trợ Deferrable FK. Do đó, quan hệ bắt buộc 1:1 (`SalesOrder` $\leftrightarrow$ `OrderAddress`, `SalesOrder` $\leftrightarrow$ `Payment`) được bảo vệ bằng cơ chế phòng thủ 2 lớp:
1. `UNIQUE FK` tại bảng con ở tầng DDL.
2. Thu hồi toàn bộ quyền DML trực tiếp (`INSERT`, `UPDATE`, `DELETE`) trên các bảng này khỏi tài khoản ứng dụng; cưỡng chế khởi tạo thông qua Stored Procedure giao dịch nguyên tử (`SET XACT_ABORT ON; BEGIN TRANSACTION ... COMMIT`).

### 3.5. Kế thừa Tài khoản TPT & Ràng buộc Loại trừ XOR (CORR-05)
Áp dụng mô hình Table-Per-Type (TPT): `Account` (cha) $\to$ `Customer` (con) / `Employee` (con). Ràng buộc loại trừ (Disjoint) được kiểm soát qua cột `AccountType` (`'CUSTOMER'`, `'EMPLOYEE'`) kết hợp Stored Procedure đăng ký và Trigger kiểm chứng toàn vẹn.

### 3.6. Lưu trữ Webhook Bền vững & Chống Tấn công Replay (CORR-06 / Issue 10)
Bảng `PaymentWebhookLog` (E71) được bảo vệ bằng ràng buộc khóa duy nhất:
`UNIQUE NONCLUSTERED (ProviderID, MerchantAccountID, ProviderEventID, Environment)`
Phản hồi HTTP 200 chỉ được gửi cho cổng thanh toán sau khi payload webhook đã được ghi nhận bền vững xuống đĩa CSDL qua Stored Procedure `usp_ReceivePaymentWebhook`.

### 3.7. Độc lập 4 Máy Trạng thái (CORR-07)
Phân định độc lập tuyệt đối giữa 4 máy trạng thái:
$$\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$$
Loại bỏ hoàn toàn trạng thái xử lý đổi trả (`RMA_PROCESSING`) khỏi `OrderStatus`. Tách biệt rõ 3 mốc thời gian: `DeliveredAt` (giao kiện cuối), `ReturnEligibleUntil` (hạn khiếu nại cấp đơn), và `ClosedAt` (đóng hồ sơ tài chính).

### 3.8. Cơ chế Truy vết Bưu kiện Đổi trả qua `ReturnShipmentAllocation` (Issue 1)
Khi đơn hàng bị tách nhiều kiện giao lệch ngày (Split Shipment), hạn đổi trả được tính độc lập theo từng kiện `Shipment.DeliveredAt`. Số lượng trả được khống chế bởi bất đẳng thức:
$$\sum \text{AllocatedReturnQty} \le \text{ShipmentLine.ShipmentQty} - \text{PreviouslyReturnedQty}$$

### 3.9. Mô hình Đối soát COD 3 Lớp (Issue 2)
Phân định dòng tiền thu hộ COD qua 3 bảng độc lập:
1. `CODCollection`: Thu tiền khách theo từng kiện khi phát hàng (cập nhật đơn hàng thành `PARTIALLY_PAID`).
2. `CODRemittance`: Khoản tiền bưu cục chuyển trả cho RetailCore theo bảng kê định kỳ.
3. `CODReconciliation`: Đối chiếu chênh lệch giữa tiền Shipper đã thu và tiền hãng thực chuyển sau cước.

### 3.10. Audit Trail Giám định Đổi trả KCS E72 (Issue 5)
Bảng `ReturnLineAssessment` áp dụng mô hình phiên bản hóa (`IsCurrent`, `AssessmentVersion`, `AssessmentStatus`: DRAFT, SUBMITTED, FINAL, SUPERSEDED) kèm Filtered Unique Index bảo đảm chỉ có duy nhất 1 bản ghi `IsCurrent = 1` cho mỗi đợt nhận hàng, cấm xóa hoặc sửa đè lịch sử.

---

## 4. CHI TIẾT GIẢI QUYẾT 10 VẤN ĐỀ KIẾN TRÚC TIỀN M3 (PRE-M3 RESOLUTION BASELINE)

Toàn bộ 10 vấn đề kiến trúc mở đã được giải quyết dứt điểm và đặc tả giải pháp kỹ thuật, DDL chuẩn hóa:

| Mã Issue | Tên Vấn đề Kiến trúc | Quyết định Kỹ thuật Đã Chốt |
| :---: | :--- | :--- |
| **Issue 1** | Truy vết Đổi trả Đa kiện Lệch ngày (B1) | Bổ sung thực thể kết hợp `ReturnShipmentAllocation` nối `ReturnRequestLine` (E61) và `ShipmentLine` (E46), kiểm tra hạn đổi trả theo từng kiện `Shipment.DeliveredAt`. |
| **Issue 2** | Đối soát COD Đơn hàng Split Shipment (B2) | Chuẩn hóa mô hình 3 lớp: `CODCollection` (Khách trả) $\to$ `CODRemittance` (Hãng chuyển) $\to$ `CODReconciliation` (Khớp đối soát chênh lệch). Hỗ trợ `PARTIALLY_PAID`. |
| **Issue 3** | Chính sách Chuẩn hóa 3NF / BCNF Thực tiễn (B3) | Ban hành Controlled Normalization Policy: 100% bảng đạt 3NF. Giữ lại 3NF không phân rã lên BCNF cho các bảng giao dịch bất biến (như `OrderLine.UnitPriceSnapshot`) kèm biên bản đánh đổi kỹ thuật. |
| **Issue 4** | Thống nhất Kiểu Dữ liệu Tiền tệ VNĐ (B4) | Dùng `DECIMAL(19,0)` cho tiền giao dịch nguyên số đồng; `DECIMAL(19,4)` cho đơn giá, chiết khấu, giá vốn WAC. Tuyệt đối cấm dùng `MONEY` hay `FLOAT`. |
| **Issue 5** | Audit Trail Giám định `ReturnLineAssessment` (B5) | Phiên bản hóa `ReturnLineAssessment` (E72) với `IsCurrent`, `AssessmentVersion`, `AssessmentStatus` kèm Filtered Unique Index kiểm soát 1 phiên bản hiện hành. |
| **Issue 6** | Xác nhận Định giá Tồn kho WAC (B6 / C1) | `CurrentAverageCost` là giá vốn bình quân di động của toàn bộ `OnHandQty` của SKU tại từng kho; tính toán tự động sau mỗi lần nhập kho `GoodsReceipt`. |
| **Issue 7** | Triệt tiêu Sai số Làm tròn LRM & $F(q)$ (B7 / D1) | Phân bổ LRM Hare-Niemeyer trên `EligibleBase` + Hàm tích lũy nguyên $F(q)$ + `DECIMAL(19,0)` đảm bảo tổng các phần bằng đúng tổng ban đầu. |
| **Issue 8** | Tích hợp Policy Engine Phí Vận chuyển (C2) | Phí vận chuyển được tách thành trường `ShippingRefundAmount` độc lập với tiền hàng, ủy quyền cho Policy Engine bên ngoài tính toán theo chính sách linh hoạt. |
| **Issue 9** | Tài khoản Hạch toán Chênh lệch PPV/WAC (C3) | Thêm trường `CostVarianceAccountCode` (TK 632 / TK 156 chênh lệch) để hạch toán chênh lệch giá xuất trả NCC so với giảm giá trị tồn kho WAC. |
| **Issue 10** | Lưu trữ Webhook Bền vững & Idempotency (C4 / D3) | Bảng `PaymentWebhookLog` có khóa `UNIQUE(ProviderID, MerchantAccountID, ProviderEventID, Environment)`; lưu bền vững trước khi gửi phản hồi HTTP 200. |

---

## 5. CÁC VẤN ĐỀ TỒN ĐỌNG & LỘ TRÌNH TRIỂN KHAI TẠI MILESTONE 3

Để chuyển giao thành công sang CSDL Quan hệ vật lý, đội ngũ kỹ thuật sẽ thực hiện tuần tự 4 công đoạn của **Milestone 3 (Logical Database Design)**:

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

### Chi tiết 4 Bước Triển khai Milestone 3:
1. **Bước 1 (M3.01) – Ánh xạ Lược đồ Quan hệ (Relational Schema Mapping):**
   * *Nội dung:* Chuyển đổi 72 thực thể quan niệm thành các bảng quan hệ SQL Server, bổ sung các bảng giao vận đa kiện (`ReturnShipmentAllocation`) và bảng đối soát COD.
   * *Trình tự triển khai:* Bắt đầu ngay từ **Ưu tiên 1: `SA01 – Product Catalog` (E01 – E09)** do ít phụ thuộc liên vùng, tiếp sau đó là `SA02 – Warehouse & Inventory`.
2. **Bước 2 (M3.02) – Chuẩn hóa Dữ liệu Toán học (3NF & BCNF Evaluation):**
   * *Nội dung:* Lập danh sách Khóa ứng viên và tập Phụ thuộc hàm đầy đủ (FDs) cho từng bảng. Đánh giá 1NF, 2NF, 3NF và BCNF.
   * *Chính sách:* Áp dụng Controlled Normalization Policy; các bảng giữ 3NF không phân rã lên BCNF (để lưu snapshot giá bán, thuế, giá vốn) phải lập văn bản giải trình kỹ thuật bảo toàn phụ thuộc hàm và nối không mất mát thông tin.
3. **Bước 3 (M3.03) – Định kiểu Dữ liệu SQL Server & Dự phóng Dung lượng (Sizing):**
   * *Nội dung:* Quy định chi tiết kiểu dữ liệu từng cột (Tiền tệ `DECIMAL(19,0)`, đơn giá `DECIMAL(19,4)`, Surrogate Key `BIGINT IDENTITY`, chuỗi `NVARCHAR(len)`, thời gian `DATETIMEOFFSET(7)`).
   * *Capacity Planning:* Tính toán kích thước dòng (Row Size), dung lượng bảng (Data Space) và dung lượng chỉ mục (Index Space) trong 3 năm dựa trên tải 1.000.000 đơn hàng/năm và 10.000.000 biến động kho/năm.
4. **Bước 4 (M3.04) – Đặc tả Chi tiết Ràng buộc Kỹ thuật & Stored Procedures:**
   * *Nội dung:* Viết mã nguồn DDL cho toàn bộ các ràng buộc khai báo (`PK`, `FK ON DELETE NO ACTION`, `CHECK`, `DEFAULT`).
   * *Stored Procedures:* Thiết kế chi tiết chữ ký và kịch bản thực thi cho 5 Stored Procedures giao dịch huyết mạch: Đặt hàng kèm địa chỉ (1:1), Phân bổ & giữ kho đa kho, Ghi sổ cái biến động kho, Tiếp nhận Webhook thanh toán, và Giám định KCS đổi trả.

---

## 6. QUYẾT ĐỊNH NGHIỆM THU CỔNG CHUYỂN GIAO (GATE SIGN-OFF)

```
========================================================================================
                      M3 GATE REVIEW SIGN-OFF: CONDITIONAL APPROVAL
========================================================================================
Dự án:                 RetailCore Enterprise E-Commerce Management System
Cơ sở dữ liệu:         Microsoft SQL Server Enterprise Edition
Giai đoạn:             Chuyển giao từ Milestone 2 (M2) sang Milestone 3 (M3)
Quyết định:            CONDITIONAL APPROVAL FOR M3.01 (Nghiệm thu Chuyển giao Có Điều kiện)
----------------------------------------------------------------------------------------
ĐIỀU KIỆN THỰC THI:
1. ĐƯỢC PHÉP KHỞI ĐỘNG NGAY:
   - Tiến hành công đoạn M3.01 (Relational Schema Mapping) bắt đầu từ SA01 - Product Catalog.
2. NGUYÊN TẮC QUẢN TRỊ KIẾN TRÚC:
   - Không áp đặt cứng nhắc 72 bảng vật lý; cấu trúc bảng được quyết định dựa trên yêu cầu
     chuẩn hóa, lưu vết kiểm toán và tối ưu hóa truy vấn.
   - Áp dụng Controlled Normalization Policy (100% đạt 3NF, giải trình kỹ thuật khi giữ 3NF).
3. BẢO TOÀN VÀ ĐỒNG BỘ:
   - Mọi điều chỉnh cấu trúc phát sinh trong M3 sẽ được đồng bộ ngược (Reverse-Sync) vào
     tài liệu M2 để đảm bảo tính nhất quán xuyên suốt vòng đời dự án.
========================================================================================
```

*(Tài liệu này là Báo cáo Tiến độ Duy nhất và Chính thức của dự án RetailCore, thay thế và tích hợp toàn bộ các báo cáo tiền đề trước đó).*

