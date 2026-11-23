# M3.01.01 - RELATIONAL SCHEMA MAPPING METHODOLOGY
## QUY CHUẨN & PHƯƠNG PHÁP LUẬN ÁNH XẠ LƯỢC ĐỒ QUAN HỆ

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server 2022+ Enterprise Edition
* **Cột mốc:** Milestone 3 (Logical Database Design) – Công đoạn M3.01
* **Phiên bản:** v1.0 – Official Logical Design Standard
* **Quản lý:** Lead Enterprise Database Engineer & Solution Architect
* **Kế thừa trực tiếp:**
  * Bộ quy tắc kiến trúc luôn có hiệu lực [GEMINI.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/GEMINI.md)
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (72 Thực thể E01 – E72)
  * [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md) (CORR-01 đến CORR-10)
  * [Project-Progress-Report.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Project-Progress-Report.md)

---

## 1. MỤC TIÊU & NGUYÊN TẮC CỐT LÕI CỦA CÔNG ĐOẠN M3.01

Công đoạn **M3.01 (Relational Schema Mapping)** chịu trách nhiệm chuyển dịch toàn diện, chuẩn xác về mặt toán học và kỹ thuật từ **Mô hình Thực thể Quan niệm (Conceptual ERD - M2)** sang **Lược đồ Bảng Quan hệ Mức Logic (Relational Tables - M3)** tương thích 100% với nền tảng Microsoft SQL Server Enterprise Edition.

### Bảy Nguyên tắc Ánh xạ Bắt buộc (The 7 Commandments):
1. **Database-First / Single Source of Truth:** CSDL là trung tâm bảo vệ toàn vẹn dữ liệu. Toàn bộ quy tắc nghiệp vụ cốt lõi phải được biểu diễn bằng ràng buộc khai báo (Declarative Constraints), Filtered Indexes và Stored Procedure Boundaries.
2. **Quy tắc Khóa Chính & Surrogate Keys:**
   * Các bảng giao dịch, chứng từ và sổ cái có lưu lượng phát sinh lớn (`SalesOrder`, `OrderLine`, `InventoryTransactionLine`, `PaymentTransaction`, `ShipmentLine`, `AuditLog`): Sử dụng `BIGINT IDENTITY(1,1)` làm Khóa chính Clustered (`PK_`).
   * Các bảng danh mục Master, cấu hình và từ điển (`Category`, `Brand`, `Warehouse`, `Carrier`, `Attribute`, `Role`): Sử dụng `INT IDENTITY(1,1)` làm Khóa chính Clustered (`PK_`).
   * Các bảng liên kết thuần túy ($N:N$ Pure Junction): Sử dụng Khóa chính phức hợp (Composite Primary Key) kết hợp từ các khóa ngoại của các bảng cha.
3. **Quy tắc Khóa Tự nhiên & Khóa Ứng viên (Candidate Keys):**
   * Các mã nghiệp vụ định danh duy nhất trong thế giới thực (`SKU`, `ProductCode`, `OrderNumber`, `WarehouseCode`, `PONumber`, `CarrierCode`, `VoucherCode`): Bắt buộc gán ràng buộc `UNIQUE NONCLUSTERED` hoặc `UNIQUE FILTERED INDEX`.
4. **Quy tắc Toàn vẹn Tham chiếu (Foreign Keys):**
   * Mọi khóa ngoại đều phải có tên định danh tường minh theo quy ước `FK_<BảngCon>_<BảngCha>[_<MụcĐích>]`.
   * Hành vi toàn vẹn tham chiếu mặc định là `ON DELETE NO ACTION` (hoặc `RESTRICT`) để tuyệt đối ngăn chặn hiện tượng Cascade Delete ngoài ý muốn làm mất dữ liệu lịch sử tài chính, kho vận.
5. **Quy chuẩn Kiểu Dữ liệu Tiền tệ & Số thực:**
   * Số tiền thanh toán VNĐ nguyên số: Bắt buộc dùng `DECIMAL(19,0) NOT NULL`.
   * Đơn giá, tỷ lệ chiết khấu, giá vốn bình quân WAC: Bắt buộc dùng `DECIMAL(19,4) NOT NULL`.
   * Tuyệt đối cấm sử dụng `MONEY`, `SMALLMONEY` (nguy cơ sai lệch làm tròn trung gian) và `FLOAT`, `REAL` (số thực dấu phẩy động thiếu chính xác).
6. **Quy tắc Bảo toàn Lịch sử (Snapshot Pattern):**
   * Các dữ liệu tại thời điểm phát sinh giao dịch là bất biến: Đơn giá `OrderLine.UnitPrice`, Địa chỉ giao hàng `OrderAddress`, Tên sản phẩm, Tỷ lệ thuế, và mảng JSON định danh thuộc tính biến thể `VariantAttributesSnapshot NVARCHAR(MAX)` có ràng buộc `ISJSON(...) = 1`.
7. **Quy tắc Phân định Trạng thái Tồn kho (Stock Quality vs Movement):**
   * Chỉ có 3 trạng thái chất lượng tồn kho vật lý: `SELLABLE`, `QUARANTINED`, `DAMAGED`. Tiêu hủy là hành động xuất kho (`SCRAP_OUT`), không tồn tại trạng thái tồn kho `DISPOSED`.

---

## 2. MA TRẬN ÁNH XẠ KIỂU DỮ LIỆU SANG SQL SERVER 2022+

| Kiểu Dữ liệu Quan niệm (Conceptual Type) | Kiểu Dữ liệu SQL Server 2022 (Physical Target) | Ràng buộc / Kiểm soát Kỹ thuật | Ghi chú Ứng dụng |
| :--- | :--- | :--- | :--- |
| **Identifier (Surrogate Large)** | `BIGINT IDENTITY(1,1)` | `PRIMARY KEY CLUSTERED` | Bảng giao dịch, sổ cái biến động kho, audit trail |
| **Identifier (Surrogate Standard)** | `INT IDENTITY(1,1)` | `PRIMARY KEY CLUSTERED` | Bảng danh mục master data, cấu hình |
| **Business Code / SKU / Slug** | `VARCHAR(50)` hoặc `VARCHAR(120)` | `NOT NULL`, `UNIQUE` | Mã không dấu, chữ hoa/thường, tối ưu hóa kích thước index |
| **Currency Amount (VND)** | `DECIMAL(19,0)` | `NOT NULL`, `CHECK (Amount >= 0)` | Tiền giao dịch, tổng tiền đơn hàng, số dư thanh toán |
| **Unit Price / Cost / Rate** | `DECIMAL(19,4)` | `NOT NULL`, `CHECK (Price >= 0)` | Đơn giá, chi phí WAC, tỷ lệ chiết khấu |
| **Quantity (Physical Inventory)** | `INT` | `NOT NULL`, `CHECK (Qty >= 0)` | Số lượng tồn kho, số lượng đóng gói, số lượng nhận |
| **Quantity Delta (Ledger)** | `INT` | `NOT NULL` (cho phép âm/dương) | Mức thay đổi tồn kho trên dòng chứng từ sổ cái |
| **Status / State Machine** | `VARCHAR(30)` | `NOT NULL`, `CHECK (Status IN (...))` | Trạng thái máy hữu hạn (State Machine Enum) |
| **Text (Short Name / Title)** | `NVARCHAR(100)` – `NVARCHAR(255)` | `NOT NULL` | Hỗ trợ tiếng Việt Unicode có dấu chuẩn mực |
| **Text (Long Description)** | `NVARCHAR(MAX)` hoặc `NVARCHAR(1000)` | `NULL` | Mô tả hàng hóa, ghi chú nghiệp vụ |
| **Boolean Flag** | `BIT` | `NOT NULL DEFAULT (0)` | Cờ nhị phân 0/1 (`IsPrimary`, `IsActive`, `IsStackable`) |
| **Timestamp / Date & Time** | `DATETIMEOFFSET(7)` | `NOT NULL DEFAULT SYSDATETIMEOFFSET()` | Lưu vết thời gian có múi giờ (+07:00 VN), chuẩn ISO 8601 |
| **Date Only** | `DATE` | `NULL` hoặc `NOT NULL` | Ngày sinh, ngày bắt đầu/kết thúc hiệu lực không cần giờ |
| **JSON Snapshot / Payload** | `NVARCHAR(MAX)` | `CHECK (ISJSON(...) = 1)` | Snapshot thuộc tính biến thể, payload webhook cổng TT |

---

## 3. QUY ƯỚC ĐẶT TÊN RÀNG BUỘC (NAMED CONSTRAINTS CONVENTION)

Để bảo đảm tính chuyên nghiệp enterprise và hỗ trợ quản trị tự động hóa bằng Flyway/Liquibase, 100% các ràng buộc phải được đặt tên tường minh (Named Constraints):

| Loại Ràng buộc (Constraint Type) | Cú pháp Quy ước Chuẩn | Ví dụ Thực tế |
| :--- | :--- | :--- |
| **Primary Key** | `PK_<TableName>` | `PK_Product`, `PK_ProductCategory` |
| **Foreign Key** | `FK_<TableName>_<TargetTable>[_<Purpose>]` | `FK_ProductVariant_Product`, `FK_Category_Category_Parent` |
| **Unique Constraint** | `UQ_<TableName>_<Columns>` | `UQ_Product_ProductCode`, `UQ_ProductVariant_SKU` |
| **Unique Filtered Index** | `UX_<TableName>_<Purpose>` | `UX_ProductCategory_Primary`, `UX_Allocation_ActiveReservation` |
| **Check Constraint** | `CK_<TableName>_<ConditionDescription>` | `CK_ProductVariant_SellingPrice`, `CK_ProductAttribute_Role` |
| **Default Constraint** | `DF_<TableName>_<ColumnName>` | `DF_Product_ProductStatus`, `DF_Category_IsActive` |

---

## 4. CHIẾN LƯỢC XỬ LÝ QUAN HỆ ĐẶC THÙ

### 4.1. Quan hệ Đệ quy Cây phân cấp (Hierarchical Recursive Tree)
* Bảng `Category` chứa khóa ngoại đệ quy `ParentCategoryID REFERENCES Category(CategoryID)`.
* Cột `CategoryLevel TINYINT NOT NULL` (1 = Gốc, 2 = Cấp 2, ...) kết hợp `DisplayOrder INT` để sắp xếp thứ tự hiển thị.
* Ràng buộc ngăn chặn nút trỏ chính mình: `CK_Category_NoSelfParent CHECK (ParentCategoryID IS NULL OR ParentCategoryID <> CategoryID)`.

### 4.2. Quan hệ $N:N$ có Thuộc tính Nghiệp vụ (Associative Junction Entity)
* Tách thành bảng quan hệ trung gian có Composite Primary Key.
* Ví dụ `ProductCategory`: mang cờ `IsPrimary BIT NOT NULL`. Cưỡng chế quy tắc nghiệp vụ BD-01 bằng Filtered Unique Index:
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_ProductCategory_Primary 
  ON ProductCategory(ProductID) 
  WHERE IsPrimary = 1;
  ```

### 4.3. Mô hình Thuộc tính Động 2 Lớp (Dynamic Attributes Model - BD-06)
* Ngăn chặn hoàn toàn bẫy trôi nổi của mô hình EAV (Entity-Attribute-Value) phi cấu trúc bằng cấu trúc 5 bảng có kiểm soát chặt chẽ:
  1. `Attribute`: Từ điển danh mục thuộc tính toàn hệ thống.
  2. `AttributeValue`: Từ điển giá trị chuẩn hóa của từng thuộc tính.
  3. `ProductAttribute`: Khai báo thuộc tính được áp dụng cho sản phẩm và chỉ định vai trò (`AttributeRole` $\in$ `{'VARIANT_DEFINING', 'DESCRIPTIVE'}`).
  4. `ProductAttributeValue`: Bộ lọc Whitelist giá trị được phép cho sản phẩm.
  5. `VariantAttributeValue`: Tổ hợp giá trị thực tế cấu thành nên một SKU độc nhất (`ProductVariant`).

---

## 5. MA TRẬN ÁNH XẠ TOÀN DIỆN 72 THỰC THỂ (MASTER ROADMAP)

Lộ trình ánh xạ 72 thực thể được tổ chức theo 11 Subject Areas với thứ tự ưu tiên giảm dần mức độ phụ thuộc:
1. **SA01 – Product Catalog (E01 – E09):** Ưu tiên 1 (Nền tảng danh mục sản phẩm, tính độc lập cao).
2. **SA02 – Warehouse & Inventory (E10 – E20):** Quản lý kho, tồn kho cách ly, reservation, ledger biến động.
3. **SA03 – Procurement (E21 – E28):** Nhà cung ứng, PO, kiểm nhận hàng, xuất trả NCC.
4. **SA04 – Customer & Identity (E29 – E37):** Tài khoản, RBAC, phân cấp TPT Customer/Employee.
5. **SA05 – Shopping (E38 – E39):** Giỏ hàng tập trung và dòng giỏ hàng.
6. **SA06 – Sales & Order Management (E40 – E43):** Đơn hàng, dòng đơn, snapshot địa chỉ, lịch sử đơn.
7. **SA07 – Fulfillment & Shipping (E44 – E48, ReturnShipmentAllocation):** Đơn vị vận chuyển, kiện hàng, attempt giao hàng.
8. **SA08 – Promotion & Voucher (E49 – E54):** Khuyến mãi phân tầng, voucher, ledger sử dụng.
9. **SA09 – Payment & Settlement (E55 – E59, E71, COD 3 lớp):** Thanh toán, hoàn tiền, đối soát COD 3 lớp, webhook log.
10. **SA10 – Return & Exchange (E60 – E64, E72):** Yêu cầu RMA, phiếu thu hồi, kiểm định KCS ReturnLineAssessment.
11. **SA11 – Audit & Reporting (E65 – E70):** Nhật ký kiểm toán an ninh WORM, báo cáo và snapshot kỳ kế toán.

