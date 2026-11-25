# RETAILCORE ENTERPRISE SYSTEM ARCHITECTURE RULES (GEMINI.md)

Hệ thống Quản trị Bán lẻ & Thương mại Điện tử Đa kênh RetailCore (Enterprise Information System).
Tài liệu quy chuẩn luôn có hiệu lực (Always-On Workspace Rules) bắt buộc áp dụng cho mọi thao tác thiết kế CSDL (M1–M4), Backend (M5), Frontend (M6), Testing (M7) và Deployment (M8).
Danh mục chuẩn toàn hệ thống gồm **79 Thực thể (`E01` – `E79`)** trên **11 Subject Areas (`SA01` – `SA11`)**, ánh xạ chuẩn xác `1:1` sang **79 Bảng Quan hệ** trên Microsoft SQL Server 2022+.

---

## 1. NGUYÊN TẮC KIẾN TRÚC CỐT LÕI (CORE ARCHITECTURAL PRINCIPLES)

1. **Database-Centric / Single Source of Truth:**
   * CSDL Microsoft SQL Server 2022+ (Developer Edition cho môi trường phát triển/kiểm thử; Enterprise Edition cho môi trường vận hành HA) là trung tâm bảo vệ tính toàn vẹn toán học và nghiệp vụ.
   * Logic toàn vẹn nghiệp vụ nhạy cảm (tồn kho, công nợ, giao dịch, đối soát) phải được cưỡng chế bởi Declarative Constraints, Filtered Indexes và ACID Boundary Transactions / Stored Procedures.
2. **Cấm Xóa Cứng (No Hard Delete / Append-Only):**
   * Tuyệt đối không dùng lệnh `DELETE` trên các bảng giao dịch và nhật ký (`SalesOrder`, `OrderLine`, `Payment`, `PaymentTransaction`, `Refund`, `RefundTransaction`, `Shipment`, `ReturnRequest`, `ReturnLineAssessment`, `InventoryTransaction`, `PaymentWebhookLog`, `AuditLog`).
   * Sử dụng máy trạng thái kết hợp cờ `IsActive` đồng bộ có ràng buộc `CHECK` cho danh mục master data.
3. **Bảo Toàn Lịch Sử (Immutable Snapshot Pattern):**
   * Thông tin tại thời điểm giao dịch là bất biến: Đơn giá `OrderLine.UnitPrice`, Tên hàng hóa, Địa chỉ giao hàng `OrderAddress`, Tỷ lệ thuế GTGT `OrderLine.VatRate` & `TaxAmount`, Giá vốn xuất kho `ShipmentLine.OutboundUnitCost`, và mảng JSON định danh thuộc tính biến thể `OrderLine.VariantAttributesSnapshot`.
   * Thay đổi thông tin trong Catalog (`Product`, `Brand`, `ProductVariant`) hoặc sổ địa chỉ (`CustomerAddress`) không bao giờ được phép làm sai lệch chứng từ đơn hàng cũ.
4. **Nguyên Tắc Trạng Thái Chất Lượng Tồn Kho (Stock Quality vs Movement Type):**
   * Chỉ có 3 trạng thái chất lượng tồn kho vật lý: `SELLABLE`, `QUARANTINED`, `DAMAGED`.
   * `RESERVED` là trạng thái giữ hàng logic theo đơn, không phải trạng thái chất lượng.
   * `DISPOSED` là kết luận định đoạt dẫn tới hành động xuất tiêu hủy khỏi kho (`SCRAP_OUT`), tuyệt đối không tồn tại trạng thái tồn kho `DISPOSED` trong kho.

---

## 2. BẢY QUYẾT ĐỊNH NGHIỆP VỤ NỀN TẢNG (BD-01 ĐẾN BD-07)

* **BD-01 (Product – Category $N:N$):** Một sản phẩm thuộc nhiều danh mục qua `ProductCategory` (E03). Cưỡng chế tối đa 1 danh mục chính bằng `UNIQUE FILTERED INDEX (ProductID) WHERE IsPrimary = 1`, kết hợp Boundary Stored Procedure / Trigger bảo đảm sản phẩm ở trạng thái `'PUBLISHED'` bắt buộc có **đúng 1** danh mục chính (`IsPrimary = 1`) và $\ge 1$ SKU hoạt động.
* **BD-02 (Cấm Guest Checkout):** Bắt buộc đăng ký/đăng nhập tài khoản trước khi đặt hàng (`SalesOrder.CustomerID NOT NULL` trỏ về `Customer.AccountID`). Cấm vãng lai để bảo toàn quyền đổi trả và kiểm soát hạn mức voucher.
* **BD-03 (Split Shipment & Tách Bạch Phí Vận Chuyển):** Một dòng đơn hàng có thể phân bổ xuất từ nhiều kho vật lý qua `OrderWarehouseAllocation` (E12). Tách bạch phí vận chuyển thu của khách (`SalesOrder.ShippingFee`) với chi phí cước thực tế trả cho hãng trên từng kiện (`Shipment.ShippingFee`).
* **BD-04 (Stock Reservation Có Hạn):** Khóa giữ tồn kho logic ngay khi tạo đơn qua `StockReservation` (E13) / `StockReservationLine` (E14) với thời hạn `ExpiresAt` (chuẩn 15 phút; rút ngắn cho chiến dịch Flash Sale). Áp dụng nguyên tắc **All-or-Nothing**: Chỉ tạo đơn hàng khi giữ đủ 100% số lượng của tất cả SKU trong đơn.
* **BD-05 (Khuyến Mãi Phân Tầng):** Tách bạch 2 tầng: Tầng 1 (`PRODUCT` Promotion trên từng `OrderLine`) và Tầng 2 (`ORDER` Voucher trên toàn `SalesOrder`). Chỉ kết hợp khi chương trình có cờ `IsStackable = 1`. Cấm vượt quá 1 chương trình cùng tầng.
* **BD-06 (Thuộc Tính Động 2 Lớp):** Tách bạch qua `ProductAttribute.AttributeRole`: `'VARIANT_DEFINING'` (cấu thành SKU như Size, Color; `ProductAttributeValue` đóng vai trò Whitelist cho `VariantAttributeValue`) và `'DESCRIPTIVE'` (thông số mô tả chung cấp sản phẩm lưu tại `ProductAttributeValue`).
* **BD-07 (Tồn Kho Khi Giao Vận & Giao Thất Bại):**
  * Bàn giao shipper (`SHIPMENT_OUT`): Giảm đồng thời cả `OnHandQty` và `ReservedQty`. Hàng trên đường giao theo dõi qua `Shipment` (E45), không nằm trong `OnHandQty` kho xuất.
  * Giao thất bại: Hàng chưa quay về kho thì tuyệt đối chưa cộng tồn kho. Chỉ khi thủ kho kiểm đếm thực nhận và ký biên bản `DeliveryReturnReceipt` (E75), hệ thống mới ghi `DELIVERY_RETURN_IN` và tăng `OnHandQty`.

---

## 3. MƯỜI HIỆU CHỈNH KIẾN TRÚC BẮT BUỘC (CORR-01 ĐẾN CORR-10)

1. **CORR-01 (P0 - Tách quyền hoàn tiền khỏi tồn kho):** Quyền hoàn tiền (`RefundEligibilityStatus`) độc lập với định đoạt vật lý (`InventoryDisposition`). Thực thể `ReturnLineAssessment` (E72) thẩm định theo từng phần số lượng thực nhận ($1:N$ với `ReturnReceiptLine`) kèm phiên bản kiểm toán (`AssessmentGroupNo`, `AssessmentVersion`, `IsCurrent`, chỉ mục lọc `UNIQUE (ReturnReceiptLineID, AssessmentGroupNo) WHERE IsCurrent = 1`). Hàng tiêu hủy `DISPOSED` do `VENDOR_FAULT` hoặc `CARRIER_FAULT` vẫn được duyệt hoàn tiền 100%. Tách rời `ResolutionType` (`REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION`).
2. **CORR-02 (P0 - Quản lý tồn kho cách ly & 5 Check Constraints):**
   * $\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$
   * $\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty}$
   * Bắt buộc có 5 Declarative CHECK Constraints trên `InventoryBalance` (E11): không âm cho 4 cột số lượng và $\text{QuarantinedQty} + \text{DamagedQty} + \text{ReservedQty} \le \text{OnHandQty}$.
3. **CORR-03 (P0 - Thuật toán LRM & Hoàn tiền lũy tiến $\Delta F$):**
   * Phân bổ chiết khấu voucher cấp đơn bằng thuật toán **Largest Remainder Method (LRM)** trên `EligibleBase`, phá hòa bằng `OrderLineID ASC`. Bảo đảm $\sum \text{AllocatedOrderDiscount} \equiv \text{ActualVoucherDiscount}$.
   * Công thức tích lũy cho $q$ đơn vị đầu tiên trên dòng mua số lượng $Q$: $F(q) = B \cdot q + \min(q, R)$ với $B = \lfloor \text{LineNetAmount} / Q \rfloor$ và $R = \text{LineNetAmount} \pmod{Q}$.
   * Khi khách trả thêm $b$ đơn vị sau khi đã được duyệt hoàn $a$ đơn vị (`OrderLine.RefundedQty` được khóa bằng `UPDLOCK`): $\Delta\text{Refund}(a \to a+b) = F(a + b) - F(a)$. Bảo đảm hoàn tiền nguyên số VNĐ qua nhiều đợt không lệch 1 đồng.
4. **CORR-04 (P1 - Ràng buộc Mandatory 1:1, Revoke DML & TPT Account):**
   * SQL Server không có Deferrable FK; thu hồi quyền DML trực tiếp (`REVOKE INSERT, UPDATE, DELETE`) đối với tài khoản runtime `retailcore_app` trên `SalesOrder`, `OrderAddress`, `Payment`, `Account`, `Customer`, `Employee` (và cấm `UPDATE, DELETE` trên `AuditLog`). Mọi thao tác tạo lập thực thi qua Boundary Stored Procedure giao dịch nguyên tử.
   * Phân cấp tài khoản theo Table-Per-Type (TPT) thuần khiết: Bảng cha `Account(AccountID, AccountType)` XOR giữa hai bảng con `Customer(AccountID PK, FK)` và `Employee(AccountID PK, FK)`. Các bảng giao dịch dùng cột FK vai trò `CustomerID` tham chiếu trực tiếp `Customer(AccountID)`.
5. **CORR-05 (P1 - Idempotency Record & Payment Webhook Log):**
   * Phân biệt rõ 3 định danh: `IdempotencyKey` (kèm `RequestHash` SHA-256 và `ResponseSnapshot` lưu tại `IdempotencyRecord` E74 với `UNIQUE (AccountID, OperationType, IdempotencyKey)`), `TransactionReference`, và `ProviderEventID`.
   * Bảng `PaymentWebhookLog` (E71) lưu bền vững trước khi trả HTTP 200, có `UNIQUE INDEX (PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL` chống replay.
6. **CORR-06 (P1 - Độc lập 4 Máy trạng thái):**
   * $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$, đồng thời tách trạng thái hồ sơ `Payment.PaymentStatus` khỏi từng lần thử `PaymentTransaction.TransactionStatus`.
   * Đơn COD chuyển `OrderStatus = 'CONFIRMED'` khi đơn hợp lệ dù `PaymentStatus = 'PENDING'`.
7. **CORR-07 (P1 - Đối soát COD 2 đầu & Mô hình COD 3 lớp cho Split Shipment):**
   * Nghĩa vụ khách hàng: Shipper thu tiền mặt theo kiện $\to$ ghi nhận `CODCollection` (E77), cập nhật tiến độ thanh toán của khách (`PARTIALLY_PAID` / `PAID`).
   * Công nợ hãng vận chuyển: Quản lý tổng hợp qua `Payment.CarrierSettlementStatus` (`NOT_COLLECTED` $\to$ `COLLECTED_BY_CARRIER` $\to$ `REMITTED` $\to$ `RECONCILED`) và chi tiết theo từng kiện qua bộ 3 thực thể `CODCollection` (E77) $\to$ `CODRemittance` (E78) $\to$ `CODReconciliation` (E79).
8. **CORR-08 (P2 - Nguồn tiền hoàn & Định giá WAC chuẩn kế toán):**
   * `OriginalPaymentTransactionID` đặt tại `RefundTransaction` (E59) trỏ về `PaymentTransaction` (E57) gốc.
   * Giá vốn tồn kho theo Bình quân Gia quyền Liên tục (**Moving Weighted Average Cost - WAC**) tính trên tồn kho đạt chuẩn (`SELLABLE`), cập nhật theo `AcceptedQty` khi nhập PO (`GoodsReceiptLine`). Lưu snapshot `OutboundUnitCost` trên `ShipmentLine` và `UnitCost` trên `WarehouseTransferLine` để bảo toàn giá vốn khi hoàn hàng/chuyển kho; chênh lệch xuất trả NCC (`SupplierReturn`) hạch toán vào `CostVarianceAccountCode`.
9. **CORR-09 (P2 - Chuẩn hóa JSON Snapshot biến thể):**
   * Cột `VariantAttributesSnapshot NVARCHAR(MAX)` trên `OrderLine` kèm `CHECK (ISJSON(VariantAttributesSnapshot) = 1)`.
   * Thống nhất 1 cấu trúc **JSON Array** duy nhất toàn hệ thống:
     `[{"attributeId": 1, "attributeCode": "SIZE", "attributeName": "Kích cỡ", "valueId": 105, "valueCode": "XL", "displayValue": "Size XL"}]`.
10. **CORR-10 (P2 - Lịch sử giữ hàng $1:0..N$ & Hạn RMA theo kiện):**
    * Quan hệ `OrderWarehouseAllocation` (E12) $\leftrightarrow$ `StockReservationLine` (E14) là $1:0..N$, cưỡng chế tối đa 1 dòng `ACTIVE` bằng `UNIQUE FILTERED INDEX (AllocationID) WHERE ReservationLineStatus = 'ACTIVE'`.
    * Bảng cầu nối `ReturnShipmentAllocation` (E76) liên kết `ReturnRequestLine` (E61) với `ShipmentLine` (E46), bảo đảm thời hạn đổi trả tính độc lập theo từng kiện: $\text{RequestedAt} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$.

---

## 4. QUY CHUẨN KỸ THUẬT T-SQL & CSDL SQL SERVER (M3 & M4)

1. **Kiểu dữ liệu tiền tệ:**
   * Tiền giao dịch & thanh toán VNĐ: `DECIMAL(19,0) NOT NULL` (số nguyên đồng, không dùng số lẻ xu).
   * Đơn giá, tỷ lệ chiết khấu, giá vốn WAC: `DECIMAL(19,4) NOT NULL` (riêng giá niêm yết bán lẻ B2C `ProductVariant.SellingPrice` / `CompareAtPrice` kèm `CHECK (Price = FLOOR(Price))` để nhân số lượng luôn ra số nguyên VNĐ).
   * **Tuyệt đối cấm:** `MONEY`, `SMALLMONEY`, `FLOAT`, `REAL`.
2. **Khóa chính & Khóa thay thế (Surrogate Key):**
   * Bảng giao dịch, tài khoản và hạt nhân SKU (`ProductVariant`, `Account`, `SalesOrder`, `OrderLine`, `InventoryTransactionLine`, `PaymentTransaction`, `AuditLog`): `BIGINT IDENTITY(1,1)` làm `PRIMARY KEY CLUSTERED` (trừ bảng phân vùng theo thời gian thì khóa chính bắt buộc bao gồm cột phân vùng để thỏa Partition-Aligned Index).
   * Bảng danh mục master vừa và nhỏ (`Category`, `Brand`, `Attribute`, `AttributeValue`, `Warehouse`, `Supplier`, `Carrier`, `Role`, `Permission`, `PaymentMethod`, `ReportDefinition`): `INT IDENTITY(1,1)`.
   * Khóa tự nhiên nghiệp vụ bắt buộc có `UNIQUE` hoặc `UNIQUE FILTERED INDEX` (`OrderNumber`, `SKU`, `ProductCode`, `BrandCode`, `WarehouseCode`, `PONumber`, `VoucherCode`).
3. **Quy ước đặt tên Named Constraints & Indexes bắt buộc:**
   * Primary Key: `PK_<TableName>`
   * Foreign Key: `FK_<ChildTable>_<ParentTable>[_<Purpose>]` (kèm `ON DELETE NO ACTION` và bắt buộc đánh Nonclustered Index hỗ trợ chiều truy vấn FK).
   * Unique Constraint / Filtered Index: `UQ_<TableName>_<Columns>` / `UX_<TableName>_<Purpose>`
   * Nonclustered Search Index: `IX_<TableName>_<Columns>`
   * Check / Default Constraint: `CK_<TableName>_<RuleDescription>` / `DF_<TableName>_<ColumnName>`
4. **Giao dịch nguyên tử & Kiểm soát đồng thời:**
   * Mọi Stored Procedure thay đổi trạng thái nhiều bảng bắt buộc bật `SET NOCOUNT ON; SET XACT_ABORT ON;`, quản lý `@@TRANCOUNT` / Savepoint an toàn và khóa tài nguyên theo thứ tự cố định (`ORDER BY VariantID ASC, WarehouseID ASC`) để triệt tiêu Deadlock.

---

## 5. CHÍNH SÁCH CHUẨN HÓA DỮ LIỆU (CONTROLLED NORMALIZATION POLICY)

* **100% bảng đạt Dạng chuẩn 3 (3NF):** Mọi thuộc tính không khóa đều phụ thuộc hàm đầy đủ và trực tiếp vào mọi khóa ứng viên (hoặc thuộc tính phụ thuộc là Prime Attribute nhằm bảo toàn phụ thuộc hàm không cần JOIN liên bảng).
* **Đánh giá BCNF & Controlled Cache Attributes:** Các cột Snapshot lịch sử (`OrderLine.UnitPrice`, `OrderAddress`, `OutboundUnitCost`) bảo toàn ngữ nghĩa thời gian hợp đồng; các cột cache phân cấp có kiểm soát (`Category.CategoryLevel`, `OrderLine.RefundedQty`) phải đi kèm văn bản Giải trình Kỹ thuật (Technical Trade-off) và cơ chế đồng bộ nguyên tử tại M3.02/M3.04.

---

## 6. QUY CHUẨN BACKEND SPRING BOOT (M5)

1. **Kiến trúc phân lớp & Hybrid Data Access:** Controller $\to$ Service $\to$ Spring Data JPA (CRUD Master & `@Immutable` Read Entities cho các bảng bị `REVOKE DML`) + `JdbcClient` (gọi Boundary Stored Procedures).
2. **Dual Database Login:** Tách biệt tài khoản `flyway_migrator` (chạy DDL migration khi khởi động) và `retailcore_app` (chạy ứng dụng runtime với quyền hạn tối thiểu `SELECT` + `EXECUTE`, tuyệt đối không chạy ứng dụng bằng `sa`).
3. **Zero-Trust Pricing & Shipping:** Tuyệt đối không nhận `UnitPrice`, `VatRate`, `ShippingFee` hay `TotalAmount` từ Client; Backend và Stored Procedure tự tính từ CSDL dựa trên giá SKU, khuyến mãi và khối lượng/kích thước vật lý.
4. **Connection Pool Safety:** Tuyệt đối không giữ SQL Lock / Transaction trong thời gian gọi HTTP ra cổng thanh toán hoặc đối tác vận chuyển bên ngoài.
5. **Idempotency & Anti-IDOR:** Cưỡng chế `Idempotency-Key` nguyên tử qua `IdempotencyRecord` (E74) cho `POST /api/orders` và `POST /api/payments`; kiểm tra quyền sở hữu `CustomerID` trên mọi API khách hàng; cấm lộ `PasswordHash`/OTP.

---

## 7. QUY CHUẨN FRONTEND REACT + TYPESCRIPT (M6)

1. **State Management & Auth Security:** TanStack Query cho Server State, Zustand cho UI State; lưu Access Token trong bộ nhớ và Refresh Token trong `HttpOnly; Secure; SameSite` Cookie (tránh lộ JWT qua `localStorage` XSS).
2. **Server-side Processing:** Phân trang, sắp xếp và Faceted Filter (Danh mục, Thương hiệu `Brand`, Thuộc tính động) thực hiện tại Server/Database.
3. **Đầy đủ 3 trạng thái UI & Checkout UX:** Đủ `Skeleton`, `Empty`, `Error`; hỗ trợ đếm ngược giữ hàng `ExpiresAt`, sinh và tái sử dụng `Idempotency-Key` theo phiên đặt hàng, hiển thị đầy đủ trạng thái Split Shipment.
4. **Xác thực Form:** Zod + React Hook Form; ánh xạ lỗi nghiệp vụ chi tiết từ Server Response.

---

## 8. QUY CHUẨN KIỂM THỬ (M7) & VẬN HÀNH DOCKER CI/CD (M8)

1. **Kiểm thử Chịu tải Đồng thời (Concurrency Stress Test):** k6/JMeter sử dụng tập hàng trăm tài khoản khách hàng độc lập đồng thời mua cùng 1 SKU Flash Sale hoặc tranh chấp voucher lượt cuối; kiểm chứng 0% Overselling và 0 Deadlock.
2. **Integration Test với Database Thật:** Sử dụng Testcontainers SQL Server 2022 với cặp tài khoản `flyway_migrator` / `retailcore_app` để kiểm chứng 100% Declarative Constraints, Stored Procedures và lệnh `REVOKE DML`.
3. **Docker Orchestration & Observability:** Docker Compose đồng bộ SQL Server 2022 (`Developer` cho Dev/Test), container khởi tạo `RetailCoreDB`, Spring Boot, React, Prometheus (kèm SQL Server metrics exporter) và Grafana Dashboard.
