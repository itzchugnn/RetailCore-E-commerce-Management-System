# RETAILCORE ENTERPRISE SYSTEM ARCHITECTURE RULES (GEMINI.md)

Hệ thống Quản trị Bán lẻ & Thương mại Điện tử Đa kênh RetailCore (Enterprise Information System).
Tài liệu quy chuẩn luôn có hiệu lực (Always-On Workspace Rules) bắt buộc áp dụng cho mọi thao tác thiết kế CSDL (M1-M4), Backend (M5), Frontend (M6), Testing (M7) và Deployment (M8).

---

## 1. NGUYÊN TẮC KIẾN TRÚC CỐT LÕI (CORE ARCHITECTURAL PRINCIPLES)

1. **Database-Centric / Single Source of Truth:**
   * CSDL Microsoft SQL Server Enterprise là trung tâm bảo vệ tính toàn vẹn toán học và nghiệp vụ.
   * Logic toàn vẹn nghiệp vụ nhạy cảm (tồn kho, công nợ, giao dịch, đối soát) phải được cưỡng chế bởi Declarative Constraints, Filtered Indexes và ACID Boundary Transactions / Stored Procedures.
2. **Cấm Xóa Cứng (No Hard Delete / Append-Only):**
   * Tuyệt đối không dùng lệnh `DELETE` trên các bảng giao dịch (`SalesOrder`, `Payment`, `PaymentTransaction`, `Shipment`, `ReturnRequest`, `InventoryTransaction`, `AuditLog`).
   * Sử dụng máy trạng thái hoặc cờ xóa mềm (`IsActive = 0`) cho danh mục master data.
3. **Bảo Toàn Lịch Sử (Immutable Snapshot Pattern):**
   * Thông tin tại thời điểm giao dịch là bất biến: Đơn giá `OrderLine.UnitPrice`, Tên hàng hóa, Địa chỉ giao hàng `OrderAddress`, Tỷ lệ thuế VAT, và mảng JSON định danh thuộc tính biến thể `VariantAttributesSnapshot`.
   * Thay đổi thông tin trong Catalog hoặc sổ địa chỉ của khách hàng không bao giờ được phép làm sai lệch chứng từ đơn hàng cũ.
4. **Nguyên Tắc Trạng Thái Chất Lượng Tồn Kho (Stock Quality vs Movement Type):**
   * Chỉ có 3 trạng thái chất lượng tồn kho vật lý: `SELLABLE`, `QUARANTINED`, `DAMAGED`.
   * `RESERVED` là trạng thái giữ hàng logic theo đơn, không phải trạng thái chất lượng.
   * `DISPOSED` là hành động xuất tiêu hủy khỏi kho (`SCRAP_OUT`), không tồn tại trạng thái tồn kho `DISPOSED` trong kho.

---

## 2. BẢY QUYẾT ĐỊNH NGHIỆP VỤ NỀN TẢNG (BD-01 ĐẾN BD-07)

* **BD-01 (Product – Category $N:N$):** Một sản phẩm thuộc nhiều danh mục. Khi ở trạng thái `Published`, bắt buộc có **đúng một** danh mục chính (`IsPrimary = 1`) để render Breadcrumb, Canonical URL và hạch toán doanh thu không trùng lặp (`UNIQUE FILTERED INDEX (ProductID) WHERE IsPrimary = 1`).
* **BD-02 (Cấm Guest Checkout):** Bắt buộc đăng ký/đăng nhập tài khoản trước khi đặt hàng (`SalesOrder.CustomerID NOT NULL`). Cấm vãng lai để bảo toàn quyền đổi trả 7 ngày và quản lý voucher.
* **BD-03 (Split Shipment):** Một đơn hàng / một dòng đơn hàng có thể phân bổ xuất từ nhiều kho vật lý khác nhau. Thực thể `OrderWarehouseAllocation` là cầu nối bắt buộc giữa `OrderLine` và `Warehouse`.
* **BD-04 (Stock Reservation Có Hạn):** Khóa giữ tồn kho logic ngay khi tạo đơn với thời hạn `ExpiresAt` (15 phút). Áp dụng nguyên tắc **All-or-Nothing**: Chỉ xác nhận đơn hàng khi giữ đủ 100% số lượng của tất cả SKU trong đơn. Hết hạn hoặc hủy đơn thì tự động giải phóng `ReservedQty`.
* **BD-05 (Khuyến Mãi Phân Tầng):** Tách bạch 2 tầng: Tầng 1 (Product Promotion trên từng dòng) và Tầng 2 (Order Voucher trên toàn đơn). Chỉ kết hợp khi voucher có cờ `IsStackable = 1`. Cấm vượt quá 1 chương trình cùng tầng.
* **BD-06 (Thuộc Tính Động 2 Lớp):** Tách bạch qua `ProductAttribute.AttributeRole`: `'VARIANT_DEFINING'` (thuộc tính cấu thành SKU như Size, Color) và `'DESCRIPTIVE'` (mô tả chung). Một thuộc tính trên một sản phẩm chỉ nhận duy nhất một vai trò.
* **BD-07 (Tồn Kho Khi Giao Vận & Giao Thất Bại):** 
  * Bàn giao shipper (`SHIPMENT_OUT`): Giảm đồng thời cả `OnHandQty` và `ReservedQty`. Hàng trên đường giao theo dõi qua `Shipment`, không nằm trong `OnHandQty` kho xuất.
  * Giao thất bại: Hàng chưa quay về kho thì tuyệt đối chưa cộng tồn kho. Chỉ khi thủ kho kiểm đếm thực nhận và ký biên bản `DeliveryReturnReceipt`, hệ thống mới ghi `DELIVERY_RETURN_IN` và tăng `OnHandQty`.

---

## 3. MƯỜI HIỆU CHỈNH KIẾN TRÚC BẮT BUỘC (CORR-01 ĐẾN CORR-10)

1. **CORR-01 (P0 - Tách quyền hoàn tiền khỏi tồn kho):** Quyền hoàn tiền của khách (`RefundEligibilityStatus`) độc lập với định đoạt vị trí kho (`InventoryDisposition`). Bổ sung thực thể `ReturnLineAssessment` (E72) thẩm định theo số lượng thực tế. Hàng bị tiêu hủy `DISPOSED` do lỗi NCC (`VENDOR_FAULT`) hoặc vận chuyển (`CARRIER_FAULT`) vẫn được duyệt hoàn tiền 100%. Tách rời hình thức bồi hoàn `ResolutionType` (`REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION`).
2. **CORR-02 (P0 - Quản lý tồn kho cách ly & 5 Check Constraints):**
   * $\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$
   * $\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty}$
   * Bắt buộc có 5 Declarative CHECK Constraints trên `InventoryBalance`: không âm và $\text{QuarantinedQty} + \text{DamagedQty} + \text{ReservedQty} \le \text{OnHandQty}$.
3. **CORR-03 (P0 - Thuật toán LRM & Hoàn tiền lũy tiến F(q)):**
   * Phân bổ chiết khấu voucher cấp đơn bằng thuật toán **Largest Remainder Method (LRM)** trên `EligibleBase`, phá hòa bằng `OrderLineID ASC`. Bảo đảm $\sum \text{AllocatedOrderDiscount} \equiv \text{ActualVoucherDiscount}$.
   * Công thức hoàn tiền lũy tiến từng đơn vị: $F(q) = B \cdot q + \min(q, R)$ với $B = \lfloor \text{LineNetAmount} / Q \rfloor$ và $R = \text{LineNetAmount} \pmod{Q}$. Bảo đảm hoàn tiền lẻ VNĐ nguyên số không lệch 1 đồng.
4. **CORR-04 (P1 - Ràng buộc Mandatory 1:1 & TPT Account):**
   * SQL Server không có Deferrable FK; thu hồi quyền DML trực tiếp trên `SalesOrder`, `OrderAddress`, `Payment`. Mọi đơn hàng tạo qua Stored Procedure giao dịch nguyên tử (`BEGIN TRANSACTION ... COMMIT`).
   * Phân cấp tài khoản theo Table-Per-Type (TPT): Bảng cha `Account(AccountID, AccountType)` với `AccountType` XOR giữa bảng con `Customer(AccountID PK_FK)` và `Employee(AccountID PK_FK)`.
5. **CORR-05 (P1 - Idempotency & Webhook Log):**
   * Phân biệt rõ `IdempotencyKey` (kèm `RequestHash` SHA-256), `TransactionReference`, và `ProviderEventID`.
   * Bảng `PaymentWebhookLog` (E71) lưu bền vững trước khi trả HTTP 200, có `UNIQUE INDEX (PaymentMethodID, ProviderEventID, Environment)` chống replay.
6. **CORR-06 (P1 - Độc lập 4 Máy trạng thái):**
   * $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$.
   * Đơn COD chuyển `CONFIRMED` khi đơn hợp lệ, không phụ thuộc `PaymentStatus`.
7. **CORR-07 (P1 - Đối soát COD 2 đầu):**
   * Nghĩa vụ khách hàng: Shipper xác nhận thu tiền mặt $\to$ `PaymentStatus = 'PAID'` (xuất hóa đơn VAT, tích điểm).
   * Công nợ hãng vận chuyển: Quản lý qua `CarrierSettlementStatus`: `NOT_COLLECTED` $\to$ `COLLECTED_BY_CARRIER` $\to$ `REMITTED` $\to$ `RECONCILED`.
8. **CORR-08 (P2 - Nguồn tiền hoàn & Định giá WAC):**
   * `OriginalPaymentTransactionID` đặt tại `RefundTransaction` (E59).
   * Giá vốn tồn kho theo Bình quân Gia quyền Liên tục (**Moving Weighted Average Cost - WAC**). Trả hàng NCC giảm tồn theo WAC hiện tại, chênh lệch so với giá PO hạch toán vào tài khoản điều chỉnh giá vốn `CostVarianceAccountCode`.
9. **CORR-09 (P2 - Chuẩn hóa JSON Snapshot biến thể):**
   * Cột `VariantAttributesSnapshot NVARCHAR(MAX)` trên `OrderLine` kèm Check Constraint `ISJSON(...) = 1`.
   * Cấu trúc mảng JSON có định danh: `[{"attributeId": 1, "attributeCode": "SIZE", "valueId": 105, "displayValue": "XL"}]`.
10. **CORR-10 (P2 - Lịch sử giữ hàng & Hạn RMA theo kiện):**
    * Quan hệ `OrderWarehouseAllocation` $\leftrightarrow$ `StockReservationLine` là $1:0..N$. Duy nhất 1 dòng giữ hàng `ACTIVE` bằng Filtered Unique Index `(AllocationID) WHERE ReservationLineStatus = 'ACTIVE'`.
    * Thời hạn đổi trả RMA tính độc lập theo từng kiện hàng: $\text{CurrentTime} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$.

---

## 4. QUY CHUẨN KỸ THUẬT T-SQL & CSDL SQL SERVER (M3 & M4)

1. **Kiểu dữ liệu tiền tệ:**
   * Tiền giao dịch VNĐ: `DECIMAL(19,0) NOT NULL` (không dùng số thập phân lẻ xu).
   * Đơn giá, tỷ lệ chiết khấu, giá vốn WAC: `DECIMAL(19,4) NOT NULL`.
   * **Tuyệt đối cấm:** `MONEY`, `SMALLMONEY`, `FLOAT`, `REAL`.
2. **Khóa chính & Khóa thay thế (Surrogate Key):**
   * Bảng giao dịch lớn (`SalesOrder`, `OrderLine`, `InventoryTransactionLine`, `PaymentTransaction`, `AuditLog`): `BIGINT IDENTITY(1,1)` làm `PRIMARY KEY CLUSTERED`.
   * Bảng danh mục master vừa và nhỏ (`Category`, `Brand`, `Warehouse`, `Carrier`, `Role`): `INT IDENTITY(1,1)`.
   * Khóa tự nhiên nghiệp vụ có ràng buộc `UNIQUE` hoặc `UNIQUE FILTERED INDEX` (`OrderNumber`, `SKU`, `WarehouseCode`, `PONumber`, `VoucherCode`).
3. **Quy ước đặt tên Named Constraints bắt buộc:**
   * Primary Key: `PK_<TableName>`
   * Foreign Key: `FK_<TableName>_<ReferencedTable>[_<Purpose>]` (kèm `ON DELETE NO ACTION`)
   * Unique Constraint / Index: `UQ_<TableName>_<Columns>` hoặc `UX_<TableName>_<Columns>`
   * Check Constraint: `CK_<TableName>_<ConditionDescription>`
   * Default Constraint: `DF_<TableName>_<ColumnName>`
4. **Giao dịch nguyên tử & Kiểm soát đồng thời:**
   * Mọi Stored Procedure thay đổi trạng thái nhiều bảng bắt buộc có:
     ```sql
     SET NOCOUNT ON;
     SET XACT_ABORT ON;
     BEGIN TRANSACTION;
     ...
     COMMIT TRANSACTION;
     ```
   * Chống Deadlock và Race Condition khi giữ hàng bằng cách cập nhật tồn kho theo thứ tự sắp xếp cố định (`ORDER BY VariantID ASC, WarehouseID ASC`).

---

## 5. CHÍNH SÁCH CHUẨN HÓA DỮ LIỆU (CONTROLLED NORMALIZATION POLICY)

* **100% bảng đạt Dạng chuẩn 3 (3NF):** Mọi thuộc tính không khóa đều phụ thuộc đầy đủ và trực tiếp vào khóa chính, không có phụ thuộc bắc cầu.
* **Đánh giá BCNF có kiểm soát:** Với các thuộc tính lưu bản chụp thời điểm (Snapshot như `OrderLine.UnitPrice`, `OrderLine.ProductName`, `OrderAddress`), việc lưu trữ là bảo toàn giá trị hợp đồng lịch sử, không phải vi phạm chuẩn hóa. Khi giữ lại 3NF mà không phân rã lên BCNF, bắt buộc lập tài liệu giải trình kỹ thuật (Technical Trade-off) chứng minh tính toàn vẹn và hiệu năng truy vấn.

---

## 6. QUY CHUẨN BACKEND SPRING BOOT (M5)

1. **Kiến trúc phân lớp:** Controller $\to$ Service $\to$ Repository (Spring Data JPA) / DAO (`JdbcClient` gọi Stored Procedure) $\to$ Database.
2. **Zero-Trust Pricing:** Tuyệt đối không nhận giá `UnitPrice` hay `TotalAmount` từ Client; Backend và Stored Procedure tự tính từ CSDL.
3. **Connection Pool Safety:** Tuyệt đối không giữ SQL Lock / Transaction trong thời gian chờ gọi mạng HTTP ra cổng thanh toán bên ngoài (Payment Gateway).
4. **Idempotency bắt buộc:** Bắt buộc có `IdempotencyKey` và `RequestHash` SHA-256 cho `POST /api/orders` và `POST /api/payments`.
5. **Security & Anti-IDOR:** Phân quyền RBAC, kiểm tra quyền sở hữu bản ghi theo `CustomerID` trên mọi endpoint nhạy cảm; không trả `PasswordHash`.

---

## 7. QUY CHUẨN FRONTEND REACT + TYPESCRIPT (M6)

1. **State Management:** TanStack Query cho Server State (caching, revalidation, optimistic updates) và Zustand cho Client UI State.
2. **Server-side Processing:** Phân trang, lọc Faceted Filter và tìm kiếm thực hiện tại Server/Database.
3. **Đầy đủ 3 trạng thái UI:** Mọi màn hình phải có giao diện Đang tải (`Skeleton`), Dữ liệu trống (`Empty`), và Báo lỗi (`Error`).
4. **Xác thực Form:** Zod + React Hook Form; hiển thị thông báo lỗi chi tiết từ Server Response.

---

## 8. QUY CHUẨN KIỂM THỬ (M7) & VẬN HÀNH DOCKER CI/CD (M8)

1. **Kiểm thử Chịu tải Đồng thời (Concurrency Stress Test):** k6/JMeter giả lập hàng trăm kết nối đồng thời mua cùng 1 SKU Flash Sale hoặc tranh chấp voucher lượt cuối; kiểm chứng 0% Overselling.
2. **Integration Test với Database Thật:** Sử dụng Testcontainers with SQL Server 2022 để kiểm chứng 100% Declarative Constraints và Stored Procedures.
3. **Docker Orchestration:** Docker Compose đồng bộ SQL Server, Spring Boot, React, Prometheus và Grafana.
4. **Observability:** Đo lường kết nối DB Pool, active locks, deadlocks, thời gian phản hồi qua Prometheus `/actuator/prometheus` và Grafana Dashboard.
