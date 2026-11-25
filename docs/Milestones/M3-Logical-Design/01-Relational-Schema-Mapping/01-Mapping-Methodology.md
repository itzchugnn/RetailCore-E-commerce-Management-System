# M3.01.01 - RELATIONAL SCHEMA MAPPING METHODOLOGY
## QUY CHUẨN & PHƯƠNG PHÁP LUẬN ÁNH XẠ LƯỢC ĐỒ QUAN HỆ (79 THỰC THỂ $\to$ 79 BẢNG)

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server 2022+ (Developer / Enterprise Edition)
* **Cột mốc:** Milestone 3 (Logical Database Design) – Công đoạn M3.01
* **Phiên bản:** v2.0 – Canonical SSOT Relational Mapping Standard
* **Kế thừa trực tiếp:**
  * Bộ quy tắc kiến trúc luôn có hiệu lực [GEMINI.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/GEMINI.md)
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (**79 Thực thể `E01` – `E79`**)
  * [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md) (`CORR-01` đến `CORR-10`)
  * [Project-Progress-Report.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Project-Progress-Report.md)

---

## 1. MỤC TIÊU & BẢY NGUYÊN TẮC ÁNH XẠ CỐT LÕI

Công đoạn **M3.01 (Relational Schema Mapping)** chuyển dịch chính xác `1:1` toàn bộ **79 Thực thể Quan niệm (`E01` – `E79`)** từ Milestone 2 sang **79 Bảng Quan hệ Mức Logic** trên Microsoft SQL Server 2022+, bảo đảm tính toàn vẹn tham chiếu tuyệt đối và đạt chuẩn 3NF/BCNF có kiểm soát.

1. **Database-First / Single Source of Truth:** Toàn bộ bất biến nghiệp vụ cốt lõi phải được bảo vệ ngay tại tầng CSDL bằng Declarative Constraints, Composite Foreign Keys, Filtered Unique Indexes và Boundary Stored Procedures.
2. **Quy tắc Khóa Chính & Surrogate Keys:**
   * Các bảng giao dịch, tài khoản và hạt nhân SKU (`ProductVariant`, `Account`, `SalesOrder`, `OrderLine`, `Payment`, `PaymentTransaction`, `Shipment`, `ReturnRequest`, `InventoryTransactionLine`, `AuditLog`): Dùng `BIGINT IDENTITY(1,1)` làm `PRIMARY KEY CLUSTERED` (riêng các bảng phân vùng theo thời gian kết hợp cột phân vùng vào Composite Clustered PK để tuân thủ luật Partition-Aligned Index).
   * Các bảng danh mục Master, cấu hình và từ điển (`Brand`, `Category`, `Product`, `Attribute`, `AttributeValue`, `Warehouse`, `Supplier`, `Carrier`, `Role`, `Permission`, `PaymentMethod`, `ReportDefinition`): Dùng `INT IDENTITY(1,1)` làm `PRIMARY KEY CLUSTERED`.
   * Các bảng liên kết $N:N$ thuần túy hoặc có thuộc tính phụ (`ProductCategory`, `ProductAttribute`, `ProductAttributeValue`, `VariantAttributeValue`, `InventoryBalance`, `SupplierProduct`, `AccountRole`, `RolePermission`, `PromotionProduct`, `PromotionTargetCustomer`, `ReportPermission`): Dùng Composite Clustered Primary Key kết hợp từ khóa của các bảng cha.
   * Các bảng kế thừa Table-Per-Type (`Customer`, `Employee`): Dùng chính `AccountID BIGINT` vừa làm `PRIMARY KEY CLUSTERED` vừa làm `FOREIGN KEY` trỏ về `Account(AccountID)` (`CORR-04`).
3. **Quy tắc Khóa Tự nhiên & Khóa Ứng viên (Candidate Keys):**
   * Mọi mã nghiệp vụ tự nhiên (`BrandCode`, `ProductCode`, `SKU`, `AttributeCode`, `WarehouseCode`, `SupplierCode`, `EmployeeCode`, `OrderNumber`, `PONumber`, `ReceiptNumber`, `ShipmentNumber`, `VoucherCode`, `TransactionReference`, `ReturnNumber`, `ReportCode`) bắt buộc gắn `UNIQUE NONCLUSTERED` hoặc `UNIQUE FILTERED INDEX`.
4. **Quy tắc Toàn vẹn Tham chiếu, Composite FK & Chỉ mục FK:**
   * Mọi khóa ngoại đặt tên tường minh `FK_<ChildTable>_<ParentTable>[_<Purpose>]` kèm `ON DELETE NO ACTION`.
   * Đối với quan hệ hình thoi có phụ thuộc loại thuộc tính (như `AttributeValue` $\to$ `ProductAttributeValue` / `VariantAttributeValue`), bắt buộc tạo khóa siêu ứng viên `UNIQUE (AttributeID, AttributeValueID)` trên bảng cha và dùng **Composite Foreign Key `(AttributeID, AttributeValueID)`** ở bảng con để ngăn chặn 100% lỗi nhiễm chéo giá trị khác loại thuộc tính.
   * Mọi cột FK không nằm ở vị trí dẫn đầu của PK/UK đều bắt buộc khai báo Nonclustered Index (`IX_`) để chống quét toàn bảng khi JOIN hoặc kiểm tra toàn vẹn.
5. **Quy chuẩn Kiểu Dữ liệu Tiền tệ, Thuế & Quy cách:**
   * Tiền giao dịch VNĐ nguyên số: `DECIMAL(19,0) NOT NULL`.
   * Đơn giá, tỷ lệ chiết khấu, giá vốn bình quân WAC: `DECIMAL(19,4) NOT NULL` (riêng giá niêm yết bán lẻ B2C `SellingPrice` / `CompareAtPrice` là **giá đã bao gồm thuế GTGT — VAT-Inclusive**, kèm `CHECK (Price = FLOOR(Price))` để nhân với số lượng luôn khớp tiền nguyên VNĐ theo `CORR-03`).
   * Tỷ lệ thuế GTGT: `DECIMAL(5,2) NOT NULL` (`0.00`, `5.00`, `8.00`, `10.00`).
   * Tuyệt đối cấm `MONEY`, `SMALLMONEY`, `FLOAT`, `REAL`.
6. **Quy tắc Bảo toàn Lịch sử (Snapshot Pattern):**
   * Bất biến thông tin hợp đồng tại thời điểm giao dịch: `OrderAddress`, `OrderLine` (`UnitPrice`, `VatRate`, `TaxAmount`, JSON Array `VariantAttributesSnapshot` có `ISJSON(...) = 1`), `ShipmentLine.OutboundUnitCost` và `WarehouseTransferLine.UnitCost`.
7. **Quy tắc Phân định Phẩm chất Tồn kho:**
   * Chỉ có 3 phẩm chất tồn vật lý (`SELLABLE`, `QUARANTINED`, `DAMAGED`); tiêu hủy là giao dịch xuất kho (`SCRAP_OUT`).

---

## 2. MA TRẬN ÁNH XẠ KIỂU DỮ LIỆU SANG SQL SERVER 2022+

| Kiểu Dữ liệu Quan niệm | Kiểu Dữ liệu SQL Server 2022 | Ràng buộc / Kiểm soát Kỹ thuật | Ghi chú Ứng dụng |
| :--- | :--- | :--- | :--- |
| **Identifier (Large / Transaction)** | `BIGINT IDENTITY(1,1)` | `PRIMARY KEY CLUSTERED` | SKU, Account, đơn hàng, sổ cái kho, thanh toán, audit |
| **Identifier (Master / Dictionary)** | `INT IDENTITY(1,1)` | `PRIMARY KEY CLUSTERED` | Danh mục master (`Brand`, `Category`, `Product`, `Warehouse`...) |
| **Business Code / SKU / Slug** | `VARCHAR(50)` hoặc `VARCHAR(120)` | `NOT NULL`, `UNIQUE` | Mã ASCII không dấu tối ưu độ rộng khóa chỉ mục B-Tree |
| **Currency Amount (VND)** | `DECIMAL(19,0)` | `NOT NULL`, `CHECK (Amount >= 0)` | Tiền giao dịch VNĐ nguyên đồng, tổng tiền, số dư thanh toán |
| **Unit Price / WAC / Rate** | `DECIMAL(19,4)` | `NOT NULL`, `CHECK (Price >= 0)` | Đơn giá, giá vốn bình quân WAC, tỷ lệ quy đổi |
| **VAT Tax Rate (%)** | `DECIMAL(5,2)` | `NOT NULL`, `CHECK (VatRate IN (0,5,8,10))` | Thuế suất GTGT chuẩn pháp luật Việt Nam |
| **Physical Weight / Dimensions** | `INT` (gram) / `DECIMAL(8,2)` (cm) | `CHECK (> 0)` | Khối lượng và kích thước đóng gói phục vụ tính cước vận chuyển |
| **Quantity (Physical Balance)** | `INT` | `NOT NULL`, `CHECK (Qty >= 0)` | Số lượng tồn kho, số lượng đặt/giao/nhận |
| **Quantity Delta (Ledger)** | `INT` | `NOT NULL` (âm/dương) | Độ lệch số dư trên dòng chứng từ sổ cái |
| **Status / State Machine** | `VARCHAR(30)` | `NOT NULL`, `CHECK (Status IN (...))` | Trạng thái hữu hạn rõ nghĩa (cấm dùng magic `TINYINT`) |
| **Text (Unicode Name / Title)** | `NVARCHAR(100)` – `NVARCHAR(255)` | `NOT NULL` | Hỗ trợ đầy đủ tiếng Việt Unicode |
| **Boolean Flag** | `BIT` | `NOT NULL DEFAULT (0/1)` | Cờ nhị phân (`IsPrimary`, `IsActive`, `IsStackable`, `IsCurrent`) |
| **Timestamp with Timezone** | `DATETIMEOFFSET(7)` | `NOT NULL DEFAULT SYSDATETIMEOFFSET()` | Lưu vết thời gian chuẩn múi giờ (`+07:00`) |
| **JSON Snapshot / Payload** | `NVARCHAR(MAX)` | `CHECK (ISJSON(...) = 1)` | Snapshot thuộc tính biến thể JSON Array, payload Webhook/Idempotency |

---

## 3. QUY ƯỚC ĐẶT TÊN RÀNG BUỘC & CHỈ MỤC (NAMED CONSTRAINTS)

| Loại Đối tượng | Cú pháp Quy ước Chuẩn | Ví dụ Thực tế |
| :--- | :--- | :--- |
| **Primary Key** | `PK_<TableName>` | `PK_Brand`, `PK_Product`, `PK_ProductCategory` |
| **Foreign Key** | `FK_<ChildTable>_<ParentTable>[_<Purpose>]` | `FK_Product_Brand`, `FK_VariantAttributeValue_AttributeValue` |
| **Unique Constraint** | `UQ_<TableName>_<Columns>` | `UQ_Brand_BrandCode`, `UQ_AttributeValue_Attribute_ValueID` |
| **Unique Filtered Index** | `UX_<TableName>_<Purpose>` | `UX_ProductCategory_Primary`, `UX_StockReservationLine_ActiveAlloc` |
| **Nonclustered Search Index** | `IX_<TableName>_<Columns>` | `IX_ProductCategory_CategoryID_ProductID`, `IX_ProductVariant_ProductID_Status` |
| **Check Constraint** | `CK_<TableName>_<ConditionDescription>` | `CK_Category_RootLevelConsistency`, `CK_ProductVariant_SellingPrice_IntegerVND` |
| **Default Constraint** | `DF_<TableName>_<ColumnName>` | `DF_Product_VatRate`, `DF_ProductVariant_WeightGrams` |

---

## 4. LỘ TRÌNH ÁNH XẠ TOÀN DIỆN 79 THỰC THỂ SANG 79 BẢNG (MASTER ROADMAP)

1. **SA01 – Product Catalog (10 bảng: `E01` – `E09`, `E73`):** `Brand`, `Category`, `Product`, `ProductCategory`, `ProductVariant`, `Attribute`, `AttributeValue`, `ProductAttribute`, `ProductAttributeValue`, `VariantAttributeValue`.
2. **SA02 – Warehouse & Inventory (11 bảng: `E10` – `E20`):** `Warehouse`, `InventoryBalance`, `OrderWarehouseAllocation`, `StockReservation`, `StockReservationLine`, `InventoryTransaction`, `InventoryTransactionLine`, `StockCount`, `StockCountLine`, `WarehouseTransfer`, `WarehouseTransferLine`.
3. **SA03 – Procurement (8 bảng: `E21` – `E28`):** `Supplier`, `SupplierProduct`, `PurchaseOrder`, `PurchaseOrderLine`, `GoodsReceipt`, `GoodsReceiptLine`, `SupplierReturn`, `SupplierReturnLine`.
4. **SA04 – Customer & Identity (9 bảng: `E29` – `E37`):** `Account`, `Customer`, `Employee`, `Role`, `Permission`, `AccountRole`, `RolePermission`, `CustomerAddress`, `AccountVerification`.
5. **SA05 – Shopping (2 bảng: `E38` – `E39`):** `ShoppingCart`, `CartItem`.
6. **SA06 – Sales & Order Management (4 bảng: `E40` – `E43`):** `SalesOrder`, `OrderLine`, `OrderAddress`, `OrderStatusHistory`.
7. **SA07 – Fulfillment & Shipping (6 bảng: `E44` – `E48`, `E75`):** `Carrier`, `Shipment`, `ShipmentLine`, `DeliveryAttempt`, `ShipmentStatusHistory`, `DeliveryReturnReceipt`.
8. **SA08 – Promotion & Voucher (6 bảng: `E49` – `E54`):** `Promotion`, `PromotionCondition`, `PromotionProduct`, `Voucher`, `PromotionUsage`, `PromotionTargetCustomer`.
9. **SA09 – Payment & Settlement (10 bảng: `E55` – `E59`, `E71`, `E74`, `E77` – `E79`):** `PaymentMethod`, `Payment`, `PaymentTransaction`, `Refund`, `RefundTransaction`, `PaymentWebhookLog`, `IdempotencyRecord`, `CODCollection`, `CODRemittance`, `CODReconciliation`.
10. **SA10 – Return & Exchange (7 bảng: `E60` – `E64`, `E72`, `E76`):** `ReturnRequest`, `ReturnRequestLine`, `ReturnReceipt`, `ReturnReceiptLine`, `ReturnStatusHistory`, `ReturnLineAssessment`, `ReturnShipmentAllocation`.
11. **SA11 – Audit & Reporting (6 bảng: `E65` – `E70`):** `AuditLog`, `AuditLogDetail`, `ReportDefinition`, `ReportExecution`, `ReportSnapshot`, `ReportPermission`.
