---
name: retailcore-schema-mapping
description: >-
  Use when mapping conceptual entities to logical relational tables in the RetailCore database project (Milestone 3 / M3.01).
  Guides transforming 79 conceptual entities (E01–E79), relationships, subtype hierarchies, and business constraints 1:1 into SQL Server relational schemas.
---

# RetailCore Relational Schema Mapping Skill

Bộ quy chuẩn hướng dẫn chuyển đổi `1:1` từ Mô hình Thực thể Quan niệm (Conceptual ERD - M2: **79 Thực thể `E01` – `E79`**) sang Lược đồ Quan hệ Logic (Relational Schema - M3.01: **79 Bảng Quan hệ**) cho hệ thống **RetailCore Enterprise E-Commerce Management System**.

---

## 1. MỤC TIÊU & ĐẦU VÀO BẮT BUỘC

Khi thực hiện công đoạn M3.01, Agent phải kiểm tra và kế thừa đầy đủ các tài liệu chuẩn hóa SSOT:
1. `GEMINI.md` (Quy tắc kiến trúc tối cao luôn có hiệu lực).
2. `docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md` (**79 Thực thể `E01` – `E79`** trên 11 Subject Areas).
3. `docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md` & `03-Cardinality-and-Constraints.md`.
4. `docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md` (`CORR-01` đến `CORR-10`).
5. `docs/erd/mermaid/*.md` (12 sơ đồ Mermaid ERD chuẩn hóa).

---

## 2. NGUYÊN TẮC ÁNH XẠ THỰC THỂ SANG BẢNG QUAN HỆ

### 2.1. Phân loại và Quy tắc Ánh xạ (`1:1` Entity-to-Table)
1. **Thực thể Mạnh (Strong Entity):**
   * Ánh xạ thành 1 bảng độc lập.
   * Sử dụng Surrogate Key `BIGINT IDENTITY(1,1)` cho bảng giao dịch/tài khoản/SKU hoặc `INT IDENTITY(1,1)` cho 12 bảng danh mục master (`Brand`, `Category`, `Product`, `Attribute`, `AttributeValue`, `Warehouse`, `Supplier`, `Carrier`, `Role`, `Permission`, `PaymentMethod`, `ReportDefinition`) làm `PRIMARY KEY CLUSTERED`.
   * Khóa tự nhiên mang ràng buộc `UNIQUE` hoặc `UNIQUE FILTERED INDEX`.
2. **Thực thể Phụ thuộc / Yếu (Weak Entity):**
   * Ánh xạ thành bảng con có Khóa ngoại (FK) trỏ về bảng cha kèm `ON DELETE NO ACTION` và Nonclustered Index hỗ trợ truy vấn FK.
   * Với quan hệ hình thoi (như `AttributeValue` $\to$ `ProductAttributeValue` / `VariantAttributeValue`), bắt buộc khai báo `UNIQUE (AttributeID, AttributeValueID)` ở bảng cha và dùng **Composite Foreign Key `(AttributeID, AttributeValueID)`** ở bảng con để chống nhiễm chéo giá trị giữa các thuộc tính.
3. **Thực thể Liên kết (Associative Entity - $N:N$):**
   * Tách thành bảng liên kết trung gian chứa các thuộc tính bổ sung của quan hệ:
     * `ProductCategory` (E03): `(ProductID, CategoryID)` + `IsPrimary` + `UX_ProductCategory_Primary`.
     * `ProductAttribute` (E07): `(ProductID, AttributeID)` + `AttributeRole` (`'VARIANT_DEFINING'` / `'DESCRIPTIVE'`).
     * `ProductAttributeValue` (E08): Whitelist cho `VARIANT_DEFINING` và lưu giá trị đặc tả cho `DESCRIPTIVE`.
     * `OrderWarehouseAllocation` (E12): Cầu nối trung gian cho Split Shipment nối `OrderLine` và `Warehouse`.
     * `ReturnShipmentAllocation` (E76): Cầu nối giải quyết đổi trả đa kiện lệch ngày nối `ReturnRequestLine` (E61) và `ShipmentLine` (E46).
4. **Mô hình Phân cấp / Kế thừa Subtypes (Table-Per-Type - TPT):**
   * Áp dụng TPT thuần khiết cho `Account` (E29), `Customer` (E30), `Employee` (E31):
     * Bảng cha `Account`: chứa thuộc tính xác thực chung và cột phân loại `AccountType ('CUSTOMER', 'EMPLOYEE')`.
     * Bảng con `Customer` và `Employee`: sử dụng `AccountID` vừa làm `PRIMARY KEY` vừa làm `FOREIGN KEY` trỏ về `Account.AccountID` (không sinh thêm cột khóa thay thế thừa). Các bảng giao dịch dùng cột FK vai trò `CustomerID` tham chiếu `Customer(AccountID)`.
5. **Cấu trúc Cây Phân cấp Đệ quy (Hierarchical Data):**
   * `Category` (E02): Khóa ngoại đệ quy `ParentCategoryID REFERENCES Category(CategoryID) NULL` kèm `CHECK ((ParentCategoryID IS NULL AND CategoryLevel = 1) OR (ParentCategoryID IS NOT NULL AND CategoryLevel BETWEEN 2 AND 10))`.
6. **Bảng Chụp Lịch sử (Snapshot Pattern):**
   * `OrderAddress` (E42): Bản chụp địa chỉ nhận hàng và thông tin hóa đơn GTGT B2B tùy chọn tại thời điểm đặt đơn ($1:1$ với `SalesOrder`). Không trỏ FK về `CustomerAddress`.
   * `OrderLine` (E41): Snapshot `UnitPrice` (đã gồm VAT), `VatRate`, `TaxAmount`, `ProductName`, `VariantName`, và JSON Array `VariantAttributesSnapshot NVARCHAR(MAX)` có `CHECK (ISJSON(...) = 1)`.
   * `ShipmentLine` (E46) & `WarehouseTransferLine` (E20): Snapshot `OutboundUnitCost` và `UnitCost` phục vụ hoàn nhập/chuyển kho chuẩn giá vốn WAC.

---

## 3. CÁC CẤU TRÚC KIẾN TRÚC ĐẶC THÙ TRONG 79 THỰC THỂ

1. **Đối soát COD 3 lớp cho Split Shipment (`CORR-07`):**
   * `CODCollection` (E77): Ghi nhận tiền bưu tá thu từ khách theo từng kiện `Shipment`.
   * `CODRemittance` (E78): Hãng vận chuyển chuyển tiền thu hộ theo bảng kê về tài khoản công ty.
   * `CODReconciliation` (E79): Khớp đối soát từng khoản thu `CODCollection` vào bảng kê `CODRemittance` và xử lý chênh lệch.
2. **Kiểm soát Idempotency & Webhook (`CORR-05`):**
   * `IdempotencyRecord` (E74): `UNIQUE (AccountID, OperationType, IdempotencyKey)`, `RequestHash`, `ResponseSnapshot`.
   * `PaymentWebhookLog` (E71): `UNIQUE (PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL`.
3. **Audit Trail Giám định `ReturnLineAssessment` (E72 — `CORR-01`):**
   * Tách lô giám định (`AssessmentGroupNo`) kết hợp phiên bản hóa (`AssessmentVersion`, `IsCurrent BIT`, `AssessmentStatus`, `InventoryDisposition`).
   * Filtered Unique Index bảo toàn tách lô $1:N$:
     `CREATE UNIQUE NONCLUSTERED INDEX UX_ReturnLineAssessment_Current ON dbo.ReturnLineAssessment(ReturnReceiptLineID, AssessmentGroupNo) WHERE IsCurrent = 1;`
4. **Phân bổ Kho & Giữ hàng $1:0..N$ (`CORR-10`):**
   * `StockReservationLine` (E14) tham chiếu `AllocationID` kèm Filtered Unique Index:
     `CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAlloc ON dbo.StockReservationLine(AllocationID) WHERE ReservationLineStatus = 'ACTIVE';`

---

## 4. QUY TRÌNH 8 TÀI LIỆU CỦA CÔNG ĐOẠN M3.01

Toàn bộ kết quả ánh xạ của M3.01 được cấu trúc thành 8 tài liệu trong `docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/` (kèm mã nguồn Mermaid tại `docs/Milestones/M3-Logical-Design/mermaid/`):
1. `01-Mapping-Methodology.md`: Nguyên tắc ánh xạ và lộ trình 79 thực thể.
2. `02-Entity-to-Relation-Mapping.md`: Chi tiết ánh xạ toàn bộ 79 thực thể sang 79 bảng theo 11 Subject Areas.
3. `03-Primary-and-Candidate-Keys.md`: Danh mục Khóa chính, Khóa tự nhiên và Khóa ứng viên.
4. `04-Foreign-Key-Mapping.md`: Danh mục Khóa ngoại (đơn & phức hợp) và chỉ mục FK.
5. `05-Relationship-Transformation.md`: Chi tiết xử lý $1:1, 1:N, N:N$, TPT và cây phân cấp.
6. `06-Cross-Area-Relations.md`: Kiểm soát toàn bộ liên kết liên miền trên trục Backbone.
7. `07-Open-Design-Decisions.md`: Đặc tả chi tiết Idempotency, COD 3 lớp, E72 Versioning và ReturnShipmentAllocation.
8. `08-Mapping-Validation.md`: Kiểm chứng chéo 100% và ma trận nghiệm thu chuyển giao sang M3.02.
