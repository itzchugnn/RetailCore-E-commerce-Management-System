---
name: retailcore-schema-mapping
description: >-
  Use when mapping conceptual entities to logical relational tables in the RetailCore database project (Milestone 3 / M3.01).
  Guides transforming 72 conceptual entities, relationships, subtype hierarchies, and business constraints into SQL Server relational schemas.
---

# RetailCore Relational Schema Mapping Skill

Bộ quy chuẩn hướng dẫn chuyển đổi từ Mô hình Thực thể Quan niệm (Conceptual ERD - M2) sang Lược đồ Quan hệ Logic (Relational Schema - M3.01) cho hệ thống **RetailCore Enterprise E-Commerce Management System**.

---

## 1. MỤC TIÊU & ĐẦU VÀO BẮT BUỘC

Khi thực hiện công đoạn M3.01, Agent phải kiểm tra và kế thừa đầy đủ các tài liệu:
1. `docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md` (72 Thực thể E01 – E72).
2. `docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md` & `03-Cardinality-and-Constraints.md`.
3. `docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md` (CORR-01 đến CORR-10).
4. `docs/Project-Progress-Report.md` (Chỉ đạo kiến trúc và biên bản Gate Sign-off).

---

## 2. NGUYÊN TẮC ÁNH XẠ THỰC THỂ SANG BẢNG QUAN HỆ

### 2.1. Phân loại và Quy tắc Ánh xạ
1. **Thực thể Mạnh (Strong Entity):**
   * Ánh xạ thành 1 bảng độc lập.
   * Sử dụng Surrogate Key `BIGINT IDENTITY(1,1)` cho bảng giao dịch hoặc `INT IDENTITY(1,1)` cho bảng danh mục làm `PRIMARY KEY CLUSTERED`.
   * Khóa tự nhiên mang ràng buộc `UNIQUE` hoặc `UNIQUE FILTERED INDEX`.
2. **Thực thể Phụ thuộc / Yếu (Weak Entity):**
   * Ánh xạ thành bảng con có Khóa ngoại (FK) trỏ về bảng cha kèm `ON DELETE NO ACTION`.
   * Khóa chính có thể là Surrogate Key riêng kèm Unique Index `(ParentID, LineNumber)` hoặc Composite Key `(ParentID, LineID)`.
3. **Thực thể Liên kết (Associative Entity - $N:N$):**
   * Luôn tách thành bảng liên kết trung gian chứa các thuộc tính bổ sung của quan hệ.
   * Ví dụ:
     * `ProductCategory`: `(ProductID, CategoryID)` + `IsPrimary`.
     * `ProductAttribute`: `(ProductID, AttributeID)` + `AttributeRole` + `IsRequired`.
     * `OrderWarehouseAllocation`: Cầu nối trung gian cho Split Shipment nối `OrderLine` và `Warehouse`.
     * `ReturnShipmentAllocation`: Cầu nối giải quyết đổi trả đa kiện lệch ngày nối `ReturnRequestLine` và `ShipmentLine`.
4. **Mô hình Phân cấp / Kế thừa Subtypes (Table-Per-Type - TPT):**
   * Áp dụng TPT cho `Account`, `Customer`, `Employee`:
     * Bảng cha `Account`: chứa thuộc tính xác thực chung và cột phân loại `AccountType ('CUSTOMER', 'EMPLOYEE')`.
     * Bảng con `Customer` và `Employee`: sử dụng `AccountID` vừa làm `PRIMARY KEY` vừa làm `FOREIGN KEY` trỏ về `Account.AccountID`.
     * Ràng buộc loại trừ Disjoint XOR và Total Participation được bảo đảm qua Boundary Stored Procedure tạo tài khoản.
5. **Cấu trúc Cây Phân cấp Đệ quy (Hierarchical Data):**
   * `Category`: Khóa ngoại đệ quy `ParentCategoryID REFERENCES Category(CategoryID) NULL`. Gốc cây có `ParentCategoryID IS NULL`.
6. **Bảng Chụp Lịch sử (Snapshot Pattern):**
   * `OrderAddress`: Bản chụp địa chỉ nhận hàng tại thời điểm đặt đơn ($1:1$ với `SalesOrder`). Không trỏ FK về sổ địa chỉ `CustomerAddress`.
   * `OrderLine`: Snapshot `UnitPrice`, `ProductName`, `VariantName`, và mảng JSON chuẩn hóa `VariantAttributesSnapshot NVARCHAR(MAX)` có `CHECK (ISJSON(...) = 1)`.

---

## 3. CÁC ĐẶC TẢ KIẾN TRÚC MỞ CẦN TÍCH HỢP TRONG M3.01

Khi thực hiện ánh xạ, agent bắt buộc phải bổ sung và chuẩn hóa các cấu trúc mở sau:
1. **Đối soát COD 3 lớp:**
   * Lớp 1: `CODCollection` (Ghi nhận tiền bưu tá thu từ khách $\to$ Đơn chuyển `PARTIALLY_PAID` / `CUSTOMER_PAID`).
   * Lớp 2: `CODRemittance` (Hãng vận chuyển chuyển tiền thu hộ về tài khoản công ty kèm mã bảng kê).
   * Lớp 3: `CODReconciliation` (Khớp đối soát chênh lệch công nợ hãng và phí vận chuyển).
2. **Audit Trail Giám định `ReturnLineAssessment` (E72):**
   * Phiên bản hóa biên bản KCS: `AssessmentVersion`, `IsCurrent BIT`, `AssessmentStatus`.
   * Cài đặt Filtered Unique Index: `CREATE UNIQUE INDEX UX_Assessment_Current ON ReturnLineAssessment(ReturnReceiptLineID) WHERE IsCurrent = 1;`.
3. **Phân bổ Kho & Giữ hàng $1:0..N$ (CORR-10):**
   * `StockReservationLine.OrderWarehouseAllocationID` kèm Filtered Unique Index: `WHERE ReservationLineStatus = 'ACTIVE'`.

---

## 4. QUY TRÌNH 8 TÀI LIỆU CỦA CÔNG ĐOẠN M3.01

Toàn bộ kết quả ánh xạ của M3.01 được cấu trúc thành 8 tài liệu trong `docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/`:
1. `01-Mapping-Methodology.md`: Nguyên tắc và quy ước chuẩn hóa.
2. `02-Entity-to-Relation-Mapping.md`: Chi tiết ánh xạ toàn bộ thực thể theo 11 Subject Areas.
3. `03-Primary-and-Candidate-Keys.md`: Danh mục Khóa chính, Khóa tự nhiên và Khóa ứng viên.
4. `04-Foreign-Key-Mapping.md`: Danh mục Khóa ngoại và quy tắc toàn vẹn tham chiếu.
5. `05-Relationship-Transformation.md`: Chi tiết xử lý $1:1, 1:N, N:N$, TPT và cây phân cấp.
6. `06-Cross-Area-Relations.md`: Kiểm soát 40 liên kết liên miền (CR-01 đến CR-40).
7. `07-Open-Design-Decisions.md`: Tích hợp COD 3 lớp và ReturnShipmentAllocation.
8. `08-Mapping-Validation.md`: Kiểm chứng chéo và ma trận nghiệm thu chuyển giao sang M3.02.

