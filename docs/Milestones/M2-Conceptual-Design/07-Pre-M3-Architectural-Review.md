# M2.07 - PRE-M3 ARCHITECTURAL GATE REVIEW REPORT
## BÁO CÁO RÀ SOÁT KIẾN TRÚC MÔ HÌNH DỮ LIỆU KHÁI NIỆM TRƯỚC CHUYỂN GIAO M3

* **Dự án:** RetailCore – E-commerce Management System
* **Cột mốc:** Milestone 2 (M2 – Conceptual Design) $\to$ Milestone 3 (M3 – Logical Design)
* **Vai trò phê duyệt:** Lead Enterprise Database Engineer & Solution Architect
* **Mục tiêu:** Rà soát toàn diện **79 thực thể (`E01` – `E79`)**, các quan hệ liên miền và 5 khu vực rủi ro kiến trúc cao theo 6 nhóm tiêu chuẩn kỹ thuật trước khi đóng M2 và ánh xạ Relational Schema `1:1`.
* **Quy chuẩn tài liệu:** Định dạng văn bản phân tích chuyên sâu, bảng biểu đối soát kỹ thuật, công thức ràng buộc toán học, không sử dụng hình ảnh.

---

## 1. TỔNG QUAN PHẠM VI RÀ SOÁT & TIÊU CHUẨN ĐÁNH GIÁ

Quá trình rà soát được thực hiện độc lập dựa trên 6 chiều kích thước kỹ thuật cốt lõi:
1. **Entity Ownership (Quyền sở hữu thực thể):** Đảm bảo nguyên tắc đơn trách nhiệm (Single Responsibility Principle) — mỗi thực thể thuộc đúng một Subject Area duy nhất, không chồng chéo thẩm quyền.
2. **Cross-Area Relationships (Quan hệ liên miền):** Rà soát tính đúng đắn của 46 liên kết liên miền (CR-01 đến CR-46), chiều phụ thuộc khóa ngoại và triệt tiêu vòng lặp phụ thuộc (Circular Dependency).
3. **Cardinality & Participation (Bản số & Tính tham gia):** Kiểm tra tính hợp lý của các cặp bản số $1:1, 1:N, N:N$ và tính tham gia Toàn phần / Một phần (Total / Partial Participation).
4. **Business Rule Consistency (Nhất quán quy tắc nghiệp vụ):** Phát hiện và hóa giải các xung đột logic giữa các phân hệ Sales, Inventory, Shipping, Payment, Promotion và Return.
5. **Data Lifecycle & History Preservation (Vòng đời dữ liệu & Bảo toàn lịch sử):** Kiểm tra cơ chế tạo, cập nhật, hủy giao dịch, snapshot lịch sử bất biến và nguyên tắc cấm xóa cứng (Append-Only / No Hard Delete).
6. **Completeness & M3 Readiness (Độ đầy đủ trước khi sang M3):** Rà soát các thuộc tính kỹ thuật cần thiết (Idempotency Key, Versioning, Trạng thái phân loại) để đảm bảo chuyển đổi mượt mà sang mô hình quan hệ 3NF/BCNF.

---

## 2. RÀ SOÁT CHI TIẾT THEO 6 CHIỀU KÍCH THƯỚC KIẾN TRÚC

### 2.1. Nhóm 1: Entity Ownership (Quyền sở hữu 79 Thực thể qua 11 Phân hệ)

| Subject Area | Thực thể trực thuộc | Rà soát phân định ranh giới (Boundary Review) | Đánh giá |
| :--- | :--- | :--- | :---: |
| **01. Product Catalog** | `Product` (E01), `Category` (E02), `ProductCategory` (E03), `ProductVariant` (E04), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09), `Brand` (E73) | Sở hữu toàn bộ danh mục thương hiệu, từ điển thuộc tính và cấu trúc SKU. Giá bán `SellingPrice` và quy cách vận chuyển gắn tại `ProductVariant`. | **Chuẩn xác (100%)** |
| **02. Warehouse & Inventory** | `Warehouse` (E10), `InventoryBalance` (E11), `OrderWarehouseAllocation` (E12), `StockReservation` (E13), `StockReservationLine` (E14), `InventoryTransaction` (E15), `InventoryTransactionLine` (E16), `StockCount` (E17), `StockCountLine` (E18), `WarehouseTransfer` (E19), `WarehouseTransferLine` (E20) | Sở hữu trọn vẹn số dư vật lý theo 3 phẩm chất, giá vốn WAC, kế hoạch phân bổ kho và chứng từ giữ tồn kho. | **Chuẩn xác (100%)** |
| **03. Procurement** | `Supplier` (E21), `SupplierProduct` (E22), `PurchaseOrder` (E23), `PurchaseOrderLine` (E24), `GoodsReceipt` (E25), `GoodsReceiptLine` (E26), `SupplierReturn` (E27), `SupplierReturnLine` (E28) | Khép kín chu trình mua hàng từ NCC. Tách bạch `PurchaseOrder` với `GoodsReceipt` và hạch toán chênh lệch WAC khi xuất trả NCC. | **Chuẩn xác (100%)** |
| **04. Customer & Identity** | `Account` (E29), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `AccountVerification` (E37) | Phân tách tài khoản xác thực `Account` với hồ sơ nghiệp vụ `Customer`/`Employee` theo mô hình TPT Disjoint XOR (`AccountID PK, FK`). | **Chuẩn xác (100%)** |
| **05. Shopping** | `ShoppingCart` (E38), `CartItem` (E39) | Quản lý trạng thái tiền giao dịch của khách hàng. Hoàn toàn tách biệt khỏi hợp đồng bán hàng chính thức. | **Chuẩn xác (100%)** |
| **06. Sales** | `SalesOrder` (E40), `OrderLine` (E41), `OrderAddress` (E42), `OrderStatusHistory` (E43) | Sở hữu hợp đồng bán hàng chính thức. Lưu snapshot địa chỉ độc lập, thuế GTGT, phân bổ voucher LRM và vết hoàn tiền lũy tiến. | **Chuẩn xác (100%)** |
| **07. Fulfillment & Shipping** | `Carrier` (E44), `Shipment` (E45), `ShipmentLine` (E46), `DeliveryAttempt` (E47), `ShipmentStatusHistory` (E48), `DeliveryReturnReceipt` (E75) | Sở hữu quy trình giao vận bên ngoài kho, theo dõi lần giao thất bại và biên bản thủ kho thực nhận kiện hoàn về (`E75`). | **Chuẩn xác (100%)** |
| **08. Promotion & Voucher** | `Promotion` (E49), `PromotionCondition` (E50), `PromotionProduct` (E51), `Voucher` (E52), `PromotionUsage` (E53), `PromotionTargetCustomer` (E54) | Quản lý quy tắc giảm giá phiên bản hóa và quota tiêu thụ. `PromotionUsage` là sổ cái hạch toán chiết khấu. | **Chuẩn xác (100%)** |
| **09. Payment & Refund** | `PaymentMethod` (E55), `Payment` (E56), `PaymentTransaction` (E57), `Refund` (E58), `RefundTransaction` (E59), `PaymentWebhookLog` (E71), `IdempotencyRecord` (E74), `CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79) | Sở hữu nghĩa vụ tài chính, chống gửi lặp `IdempotencyRecord`, lưu vết Webhook, đối soát COD 3 lớp theo kiện và truy vết nguồn tiền hoàn trả. | **Chuẩn xác (100%)** |
| **10. Return & Exchange** | `ReturnRequest` (E60), `ReturnRequestLine` (E61), `ReturnReceipt` (E62), `ReturnReceiptLine` (E63), `ReturnStatusHistory` (E64), `ReturnLineAssessment` (E72), `ReturnShipmentAllocation` (E76) | Sở hữu nghiệp vụ RMA, cầu nối kiện giao (`E76`) và thẩm định chất lượng phiên bản hóa (`E72`) tách biệt quyền hoàn tiền khỏi định đoạt kho. | **Chuẩn xác (100%)** |
| **11. Audit & Reporting** | `AuditLog` (E65), `AuditLogDetail` (E66), `ReportDefinition` (E67), `ReportExecution` (E68), `ReportSnapshot` (E69), `ReportPermission` (E70) | Quản lý an ninh vết kiểm toán WORM và kết xuất báo cáo quản trị / chốt kỳ kế toán. | **Chuẩn xác (100%)** |

> **Kết luận Nhóm 1:** Cả **79 thực thể (`E01` – `E79`)** đều có ranh giới chủ quyền rõ ràng, không có thực thể nào bị trùng lặp trách nhiệm hoặc tranh chấp thẩm quyền giữa các phân hệ.

---

### 2.2. Nhóm 2: Cross-Area Relationships (Rà soát 46 Liên kết Liên miền CR-01 đến CR-46)

* **Phát hiện 1: Khử hoàn toàn vòng lặp phụ thuộc (Zero Circular Dependencies):**
  * Chuỗi liên kết bán hàng và kho vận:  
    `SalesOrder` $\to$ `OrderLine` $\to$ `OrderWarehouseAllocation` $\to$ `StockReservationLine` $\to$ `StockReservation` $\to$ `SalesOrder`.
  * *Kiểm chứng:* `StockReservation` trỏ khóa ngoại `OrderID` về `SalesOrder` để quản lý theo cấp chứng từ tổng, trong khi dòng chi tiết `StockReservationLine` trỏ `AllocationID` về dòng phân bổ kho. Đây là mô hình phân rã 2 cấp (Header-Line) song song, không tạo vòng lặp khóa ngoại cứng giữa `OrderLine` và `SalesOrder`.
* **Phát hiện 2: Ràng buộc tính nhất quán kho xuất hàng (Warehouse Consistency Constraint):**
  * Trong phân hệ Fulfillment: `Shipment` có `WarehouseID` (kho xuất bưu kiện) và `ShipmentLine` có `AllocationID` (tham chiếu đến `OrderWarehouseAllocation`).
  * *Ràng buộc bắt buộc ở M3:* $\text{Shipment.WarehouseID} \equiv \text{OrderWarehouseAllocation.WarehouseID}$. Một kiện hàng của một kho không thể đóng gói dòng phân bổ thuộc kho khác.

---

### 2.3. Nhóm 3: Cardinality & Participation Constraints (Bản số & Tính tham gia)

1. **Quan hệ $1:1$ Bắt buộc (Mandatory $1:1$) & Giới hạn của DDL:**
   * `SalesOrder` $\leftrightarrow$ `OrderAddress`: Đúng $1:1$. Mỗi đơn hàng bắt buộc có đúng 1 bản chụp địa chỉ giao hàng (`OrderAddress.OrderID` UNIQUE).
   * `SalesOrder` $\leftrightarrow$ `Payment`: Đúng $1:1$. Mỗi hợp đồng bán hàng chính thức gắn liền với duy nhất 1 hồ sơ thanh toán (`Payment.OrderID` UNIQUE). Dù thanh toán nhiều lần hoặc nhiều cổng, các lần thử đều nằm trong `PaymentTransaction` ($1:N$ với `Payment`).
   * *Lưu ý kỹ thuật then chốt (CORR-04):* Khóa ngoại kèm UNIQUE ở bảng con chỉ bảo đảm cận trên $\le 1$ ($0..1$), không tự bảo đảm cận dưới $\ge 1$ (Mandatory Participation) ở tầng Declarative DDL. Bắt buộc phải thực thi qua Stored Procedure giao dịch nguyên tử (`BEGIN TRANSACTION ... COMMIT`) tạo đồng thời Header và Detail để ngăn chặn bản ghi mồ côi.
2. **Quan hệ $1:0..1$ Tùy chọn (Optional $1:0..1$) & Mô hình Kế thừa Table-Per-Type (TPT):**
   * `Account` $\leftrightarrow$ `Customer`: Tối đa 1 hồ sơ Customer.
   * `Account` $\leftrightarrow$ `Employee`: Tối đa 1 hồ sơ Employee.
   * *Ràng buộc loại trừ (Disjoint XOR & TPT, CORR-04):* Bảng `Account` bổ sung trường định danh subtype `AccountType` (`CUSTOMER` / `EMPLOYEE`). Bảng con `Customer` và `Employee` sử dụng `AccountID PK_FK`. Stored Procedure đăng ký tài khoản bắt buộc tạo đồng thời `Account` và bảng con tương ứng trong một transaction.
   * `ShoppingCart` $\leftrightarrow$ `SalesOrder` ($0..1:0..1$): Một giỏ hàng chuyển đổi thành tối đa 1 đơn hàng, và đơn hàng mua ngay hoặc đơn đổi trả (`EXCHANGE_REPLACEMENT`) có thể để `CartID IS NULL` (`UX_SalesOrder_Cart ON dbo.SalesOrder(CartID) WHERE CartID IS NOT NULL`). Khi đã chuyển đổi, giỏ hàng khóa trạng thái `CONVERTED`.
3. **Tính tham gia Toàn phần của Thực thể Yếu (Total Participation):**
   * Toàn bộ các thực thể chi tiết dòng (`OrderLine`, `ShipmentLine`, `StockReservationLine`, `GoodsReceiptLine`, `ReturnRequestLine`, `ReturnReceiptLine`) đều có tính tham gia **Toàn phần (Total)** đối với thực thể Header cha. Cấm tuyệt đối dòng hàng mồ côi (Orphan lines).

---

### 2.4. Nhóm 4: Business Rule Consistency (Tính Nhất quán Quy tắc Nghiệp vụ)

* **Quy tắc Kiểm soát Tồn kho vs Trạng thái Đơn hàng:**
  * Khi tạo đơn hàng thành công (`SalesOrder = PENDING_PAYMENT` hoặc `CONFIRMED`), hàng được khóa giữ bằng `StockReservation`. Tồn kho vật lý `OnHandQty` **chưa giảm**, chỉ có `ReservedQty` tăng:
    $$\text{AvailableQty} = (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}) - \text{ReservedQty}$$
  * Khi đơn hàng hết hạn thanh toán (sau 15 phút), Trigger/Job tự động hủy đơn (`OrderStatus = CANCELLED`), giải phóng giữ kho (`ReservationStatus = RELEASED`). `ReservedQty` giảm, `OnHandQty` giữ nguyên. Tuyệt đối không sinh giao dịch xuất nhập kho vật lý.
* **Quy tắc Xuất bãi vs Đang giao (In-Transit):**
  * Hàng chỉ thực sự trừ `OnHandQty` và tiêu thụ `ReservedQty` khi nhân viên kho đóng gói và bàn giao bưu kiện cho shipper (`ShipmentStatus = DISPATCHED`). Lúc này ghi nhận `InventoryTransaction` loại `SHIPMENT_OUT`.

---

### 2.5. Nhóm 5: Data Lifecycle & History Preservation (Bảo toàn Lịch sử Dữ liệu)

1. **Cơ chế Snapshot Đa Tầng Trọng Yếu:**
   * **Giá đã gồm VAT, Thuế GTGT và Thuộc tính Sản phẩm (`OrderLine`):** Snapshot `UnitPrice` (giá bán lẻ B2C đã bao gồm VAT), `VatRate`, `TaxAmount` (bóc tách nội bộ), `ProductName`, `VariantName`, và mảng JSON chuẩn hóa `VariantAttributesSnapshot`. Dù sau này Catalog tăng giá hay đổi tên thuộc tính, chứng từ đơn cũ không bao giờ biến động.
   * **Địa chỉ Giao hàng & Hóa đơn GTGT B2B (`OrderAddress`):** Snapshot nguyên trạng người nhận, số điện thoại, tỉnh/huyện/xã, địa chỉ chi tiết cùng thông tin xuất hóa đơn GTGT tùy chọn (`BuyerCompanyName`, `BuyerTaxCode`, `BuyerCompanyAddress`, `BuyerInvoiceEmail`). Khách hàng sửa sổ địa chỉ cá nhân (`CustomerAddress`) không làm sai lệch đơn hàng đang giao.
   * **Giá vốn Nhập, Xuất & Chuyển kho (`GoodsReceiptLine`, `ShipmentLine`, `WarehouseTransferLine`):** Snapshot `GoodsReceiptLine.UnitCost` khi nhập kho, `ShipmentLine.OutboundUnitCost` khi xuất giao và `WarehouseTransferLine.UnitCost` khi điều chuyển nội bộ để bảo toàn giá vốn bình quân gia quyền (**Moving WAC**, `CORR-08`).
2. **Nguyên tắc Bất biến & Cấm Xóa Cứng (No Hard Delete):**
   * Các bảng chứng từ giao dịch (`SalesOrder`, `Payment`, `Shipment`, `ReturnRequest`, `InventoryTransaction`, `AuditLog`) **tuyệt đối cấm lệnh `DELETE`**.
   * Hủy đơn chuyển `OrderStatus = 'CANCELLED'`. Hết hạn hoặc hủy thanh toán chuyển `PaymentStatus = 'EXPIRED' / 'CANCELLED'` (từng lần giao dịch cổng bị lỗi ghi nhận tại `PaymentTransaction.TransactionStatus = 'FAILED'`).
   * Danh mục dùng cờ xóa mềm: `IsActive = 0`.

---

### 2.6. Nhóm 6: Completeness & M3 Readiness (Độ Đầy đủ Kỹ thuật cho M3)

* **Định danh Khóa chính và Mã tự nhiên:**
  * Toàn bộ 79 thực thể đều đã xác định rõ Surrogate Key (`INT`/`BIGINT IDENTITY`) hoặc Composite Key hợp lý.
  * Các thực thể nghiệp vụ cốt lõi đều có Natural Business Key duy nhất: `BrandCode`, `ProductCode`, `SKU`, `WarehouseCode`, `SupplierCode`, `PONumber`, `ReceiptNumber`, `OrderNumber`, `ShipmentNumber`, `VoucherCode`, `TransactionReference`, `ReturnNumber`, `ReportCode`.
* **Idempotency & Concurrency:**
  * Bổ sung `IdempotencyRecord` (E74) với `UNIQUE (AccountID, OperationType, IdempotencyKey)` và bảng đệm `PaymentWebhookLog` (E71) với Filtered Unique Index `(PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL` để chống trùng lặp request và Webhook xử lý bất đồng bộ (CORR-05).
  * Chuẩn bị sẵn trường `TransactionReference` trên `PaymentTransaction` và `RefundTransaction` để cài đặt Unique Index, cùng `OriginalPaymentTransactionID` để truy vết đối soát giao dịch gốc.

---

## 3. RÀ SOÁT CHUYÊN SÂU 5 KHU VỰC RỦI RO THIẾT KẾ CAO (HIGH-RISK AREAS)

### 3.1. Rủi ro 1: Quan hệ Phân bổ Kho vs Giữ Tồn Kho (Allocation – Reservation)

* **Vấn đề rủi ro:** Một đơn hàng có thể gồm nhiều sản phẩm nằm ở các kho khác nhau (Split Shipment). Nếu thiết kế giữ hàng gắn cứng vào `SalesOrder` mà không định danh kho, hệ thống sẽ không biết trừ tồn tại kho nào khi xuất bãi. Hơn nữa, việc cố định $1:0..1$ sẽ làm mất lịch sử giữ hàng nếu khách thanh toán lại sau khi timeout.
* **Giải pháp kiến trúc đã chốt (CORR-10):**
  * `OrderWarehouseAllocation` (E12) đóng vai trò là **kế hoạch phân bổ**.
  * `StockReservation` (E13) và `StockReservationLine` (E14) đóng vai trò là **khóa giữ vật lý tại kho**.
  * **Quan hệ $1 : 0..N$ linh hoạt:** Một `Allocation` có thể trải qua nhiều lượt giữ hàng trong vòng đời (lần 1 hết hạn EXPIRED, lần 2 tạo dòng giữ hàng mới).
  * **Kiểm soát tính duy nhất qua Filtered Unique Index:** Trên SQL Server cài đặt `CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAlloc ON dbo.StockReservationLine(AllocationID) WHERE ReservationLineStatus = 'ACTIVE'`, bảo đảm tại một thời điểm chỉ có tối đa 1 dòng giữ hàng đang hiệu lực cho mỗi phân bổ kho.
  * **Xử lý tình huống Hủy / Hết hạn đơn hàng:**
    * Nếu khách không thanh toán trong 15 phút: Hệ thống hủy `StockReservation`, hoàn lại `ReservedQty` tại đúng kho đó. Kế hoạch `OrderWarehouseAllocation` chuyển sang `CANCELLED`. Tồn vật lý `OnHandQty` không bị tác động sai lệch.

---

### 3.2. Rủi ro 2: Thanh toán COD vs Đối soát Thu hộ Tiền mặt (Payment – COD Collection)

* **Vấn đề rủi ro:** Nếu gộp chung trạng thái đơn hàng và thanh toán, hoặc không phân định nghĩa vụ thanh toán của khách với khoản phải thu từ hãng vận chuyển, số liệu tài chính sẽ bị phản ánh sai lệch nghiêm trọng.
* **Giải pháp kiến trúc đã chốt (CORR-06 & CORR-07):**
  * Tách độc lập 4 máy trạng thái giao dịch (kèm trạng thái từng lần thử `PaymentTransaction.TransactionStatus`):
    $$\text{SalesOrder.OrderStatus} \neq \text{Payment.PaymentStatus} \neq \text{Shipment.ShipmentStatus} \neq \text{ReturnRequest.ReturnStatus}$$
  * **Mô hình Đối soát COD Hai Đầu & Ba Lớp theo Kiện (Two-Sided & 3-Layer COD Reconciliation):**
    * **Đầu 1 (Nghĩa vụ khách hàng):** Khi bưu tá thu tiền mặt trên kiện hàng (`ShipmentStatus = 'DELIVERED'`), hệ thống tạo bản ghi `CODCollection` (E77) và cập nhật tiến độ thanh toán của khách trên `Payment.PaymentStatus` (`'PARTIALLY_PAID'` với đơn đa kiện giao từng phần, hoặc `'PAID'` khi đã thu đủ phần đơn hàng có hiệu lực).
    * **Đầu 2 (Công nợ thu hộ từ đơn vị vận chuyển):** Số tiền COD nằm tại hãng vận chuyển là khoản phải thu của doanh nghiệp, theo dõi tổng hợp qua `Payment.CarrierSettlementStatus` (`NOT_COLLECTED` $\to$ `COLLECTED_BY_CARRIER` $\to$ `REMITTED` $\to$ `RECONCILED`) và đối soát chi tiết từng kiện qua chuỗi 3 thực thể `CODCollection` (E77) $\to$ `CODRemittance` (E78) $\to$ `CODReconciliation` (E79).

---

### 3.3. Rủi ro 3: Chu trình Logistics Ngược (Return – Refund – Inventory)

* **Vấn đề rủi ro:** Nhầm lẫn giữa quyền được hoàn tiền của khách hàng và tình trạng vật lý của hàng hóa sau kiểm định. Nếu bắt buộc hàng phải `SELLABLE` mới được hoàn tiền thì khách nhận hàng lỗi từ nhà sản xuất hoặc bị vỡ do vận chuyển (hàng phải `DISPOSED`) sẽ bị từ chối hoàn tiền sai chính sách. Ngoài ra, vì `SHIPMENT_OUT` đã trừ `OnHandQty`, nếu xuất hủy `SCRAP_OUT` mà chưa ghi nhận nhập kho cách ly trước sẽ làm âm hoặc lẹm vào tồn kho tốt đang có tại kho.
* **Giải pháp kiến trúc đã chốt (Tách rời Quyền Hoàn tiền khỏi Tồn kho, CORR-01 & CORR-02):**
  1. **Chặng 1 (Tiếp nhận & Duyệt RMA):** Khách gửi `ReturnRequest` trong thời hạn chính sách tính theo từng kiện hàng qua `ReturnShipmentAllocation` (E76, `Shipment.DeliveredAt`, CORR-10). CSKH duyệt `APPROVED` và chỉ định duy nhất một `WarehouseID` tiếp nhận.
  2. **Chặng 2 (Tiếp nhận cách ly tại bến nhập - Dock Intake):** Bưu kiện về đến bến nhập, thủ kho ký `ReturnReceipt` (E62) và `ReturnReceiptLine` (E63); hệ thống ghi ngay `InventoryTransaction` loại `RETURN_RECEIPT_IN` (`+OnHandQty`, `+QuarantinedQty`), giữ nguyên `SellableOnHandQty` bằng 0 biến động cho đến khi KCS thẩm định xong.
  3. **Chặng 3 (Định đoạt vật lý tại kho sau giám định - Inventory Disposition):** Căn cứ kết luận KCS để chuyển tiếp từ bể `QUARANTINED`:
     * `SELLABLE` (Hàng mới, nguyên vẹn): Ghi `CUSTOMER_RETURN_IN` giải phóng `-QuarantinedQty` sang tồn bán `SellableOnHandQty`, hòa trộn lại giá vốn bình quân WAC theo `ShipmentLine.OutboundUnitCost` (CORR-08).
     * `QUARANTINED` (Chờ bảo hành/giám định sâu): Giữ nguyên trong `QuarantinedQty`, cách ly khỏi tồn bán được.
     * `DISPOSED` (Hỏng hóc, tiêu hủy): Ghi `SCRAP_OUT` giảm đồng thời `-OnHandQty` và `-QuarantinedQty`, hạch toán tổn thất, bảo toàn 5 CHECK Constraints trên `InventoryBalance` (CORR-02).
  4. **Chặng 4 (Đánh giá nguyên nhân lỗi, Hoàn tiền & Đổi hàng - ReturnLineAssessment E72):**
     * Nhân viên thẩm định lập bản ghi `ReturnLineAssessment` (E72) phiên bản hóa cho từng phần số lượng thực nhận, xác định rõ: `FaultParty` (`VENDOR_FAULT`, `CARRIER_FAULT`, `CUSTOMER_FAULT`, `NO_FAULT`), `RefundEligibilityStatus` (`PENDING_INSPECTION`, `APPROVED`, `PARTIALLY_APPROVED`, `REJECTED`), `InventoryDisposition` (`SELLABLE`, `QUARANTINED`, `DISPOSED`) và `ResolutionType` (`REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION`).
     * Hàng lỗi do `VENDOR_FAULT` hoặc `CARRIER_FAULT` vẫn được duyệt hoàn tiền 100% hoặc sinh đơn hàng đổi mới `SalesOrder` (`OrderType = 'EXCHANGE_REPLACEMENT'`, `ParentOrderID`, tổng thu `0đ`) kể cả khi vật lý hàng hóa bị `DISPOSED`.
  5. **Chặng 5 (Kích hoạt Hoàn tiền Từng phần):** Kích hoạt tạo chứng từ `Refund` bên Subject Area 09 ngay khi có lượt đánh giá E72 được duyệt với `ResolutionType = 'REFUND'`, cho phép hoàn tiền theo tiến độ nhận hàng thực tế mà không cần chờ toàn bộ yêu cầu trả hàng hoàn tất.

---

### 3.4. Rủi ro 4: Phân bổ Chiết khấu Khuyến mãi khi Hoàn tiền (PromotionUsage – Payment – Refund)

* **Vấn đề rủi ro:** Đơn hàng áp Voucher cấp đơn giảm giá trên giá bán lẻ đã bao gồm VAT. Nếu dùng phép chia hoặc làm tròn `Round()` thông thường thì tổng chiết khấu phân bổ bị lệch tiền lẻ (Rounding Error). Khi hủy thiếu hàng trước giao (`CancelledQty`) hoặc khách hoàn trả từng đơn vị lẻ (`RefundedQty`), nếu không dùng chung một hàm lũy tiến xác định hoặc không kiểm soát hành vi lạm dụng ngưỡng `MinOrderAmount` thì doanh nghiệp sẽ lệch sổ hoặc thất thoát ngân sách khuyến mãi.
* **Giải pháp kiến trúc đã chốt (Giá VAT-Inclusive, LRM, $\Delta F$ & Voucher Clawback, CORR-03):**
  1. **Phân bổ chiết khấu cấp đơn chuẩn mực bằng thuật toán Largest Remainder Method (LRM):**
     * Tính phần nguyên và phần dư số tiền giảm giá cấp đơn cho từng `OrderLine` (phá hòa theo `OrderLineID ASC`), bảo đảm tổng chiết khấu phân bổ đúng 100% không lệch 1 đồng:
       $$\sum \text{AllocatedOrderDiscount} \equiv \text{ActualVoucherDiscount}$$
     * Lưu trữ trực tiếp trên `OrderLine`: `LineGrossAmount`, `LineProductDiscount`, `AllocatedOrderDiscount`, `LineNetAmount` và `TaxAmount` (bóc tách thuế GTGT nội bộ từ `LineNetAmount`).
  2. **Công thức hoàn tiền từng đơn vị lũy tiến (Unit-Level Progressive Refund Formula):**
     * Với dòng hàng mua số lượng $Q$ có `LineNetAmount`:
       $$B = \left\lfloor \frac{\text{LineNetAmount}}{Q} \right\rfloor, \quad R = \text{LineNetAmount} \pmod{Q}$$
     * Hàm số tích lũy tiền hoàn cho $q$ đơn vị:
       $$F(q) = B \cdot q + \min(q, R)$$
     * Mọi sự kiện hủy thiếu hàng trước xuất kho (`CancelledQty`) và trả hàng sau giao (`RefundedQty`) cùng tiến trên một biến tích lũy $a = \text{CancelledQty} + \text{RefundedQty}$ (khóa `UPDLOCK`): khi tăng thêm $b$ đơn vị, $\Delta \text{Refund} = F(a + b) - F(a)$, bảo đảm khi $a+b=Q$ tổng tiền hoàn luôn khớp đúng `LineNetAmount`.
     * **Thu hồi Voucher khi trả hàng lỗi khách (`Voucher Clawback`):** Nếu `FaultParty = 'CUSTOMER_FAULT'` làm giá trị hàng giữ lại thấp hơn ngưỡng `MinOrderAmount` của voucher cấp đơn, hệ thống khấu trừ phần ưu đãi voucher không còn đủ điều kiện hưởng khỏi khoản hoàn (cận dưới `0đ`).

---

### 3.5. Rủi ro 5: Tính Bất biến của Dữ liệu Lịch sử (Historical Snapshots)

* **Vấn đề rủi ro:** Khách hàng thay đổi địa chỉ trong sổ địa chỉ, hoặc bộ phận Marketing sửa tên sản phẩm/tăng giá biến thể làm biến dạng dữ liệu kế toán và hóa đơn của các đơn hàng cũ. Việc lưu JSON snapshot vô cấu trúc hoặc text tự do khiến không thể truy vấn hoặc phân tích dữ liệu lịch sử.
* **Giải pháp kiến trúc đã chốt (CORR-09):**
  * **Tách rời hoàn toàn Entity Sổ địa chỉ và Entity Đơn hàng:**
    * `CustomerAddress`: Là sổ địa chỉ cá nhân, khách hàng có quyền thêm, sửa, xóa, đổi mặc định tùy ý.
    * `OrderAddress`: Là một thực thể riêng biệt mang bản chất **Weak Snapshot**, sao chép nguyên văn thông tin địa chỉ tại giây phút đặt hàng. Bảng này không có khóa ngoại trỏ về `CustomerAddress` mà chỉ trỏ về `SalesOrder.OrderID`.
  * **Snapshot Dòng hàng đa tầng & Chuẩn hóa JSON Định danh (CORR-09):**
    * `OrderLine.UnitPrice`: Snapshot giá bán tại thời điểm bấm đặt hàng.
    * `OrderLine.ProductName` & `VariantName`: Snapshot tên hiển thị.
    * `OrderLine.VariantAttributesSnapshot`: Lưu toàn bộ thuộc tính cấu thành biến thể dưới dạng **mảng JSON có định danh ID và mã Code chuẩn mực**:
      ```json
      [
        {"attributeId": 1, "attributeCode": "SIZE", "attributeName": "Kích cỡ", "valueId": 105, "valueCode": "XL", "displayValue": "Size XL"},
        {"attributeId": 2, "attributeCode": "COLOR", "attributeName": "Màu sắc", "valueId": 204, "valueCode": "NAVY", "displayValue": "Xanh Navy"}
      ]
      ```
    * Cấm lưu text tự do hoặc chuỗi vô cấu trúc `{"Size": "XL"}` để bảo đảm khả năng lập chỉ mục full-text hoặc bóc tách dữ liệu bằng hàm `JSON_VALUE` / `OPENJSON` trên SQL Server.

---

## 4. MA TRẬN ĐỐI SOÁT & DANH MỤC KHUYẾN NGHỊ KỸ THUẬT CHO M3

| Khu vực kỹ thuật | Rủi ro tiềm ẩn | Cơ chế kiểm soát bắt buộc chuyển giao sang M3 (Logical Design) |
| :--- | :--- | :--- |
| **Identity Subtypes (TPT)** | Một tài khoản vừa là Customer vừa là Employee gây xung đột bảo mật | Mô hình Table-Per-Type (TPT): Bảng cha `Account(AccountID, AccountType)`, bảng con `Customer(AccountID PK_FK)`, `Employee(AccountID PK_FK)`. Stored Procedure đăng ký tài khoản thực thi giao dịch nguyên tử. |
| **Active Cart Limit** | Khách hàng mở nhiều giỏ hàng `ACTIVE` gây lỗi đồng bộ | Tạo Filtered Unique Index trên SQL Server: `CREATE UNIQUE NONCLUSTERED INDEX UX_ShoppingCart_Active ON dbo.ShoppingCart(CustomerID) WHERE CartStatus = 'ACTIVE'`. |
| **Active Allocation Reservation** | Nhiều dòng giữ hàng xung đột đồng thời trên cùng một phân bổ kho | Tạo Filtered Unique Index: `CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAlloc ON dbo.StockReservationLine(AllocationID) WHERE ReservationLineStatus = 'ACTIVE'` (CORR-10). |
| **Idempotent API & Webhook** | Gửi lặp request đặt đơn/thanh toán hoặc Webhook từ cổng gửi trùng lặp | Bổ sung `IdempotencyRecord` (E74) và `PaymentWebhookLog` (E71) với Filtered Unique Index: `(PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL` (CORR-05). |
| **Two-Sided & 3-Layer COD** | Nhầm lẫn nghĩa vụ thanh toán của khách với công nợ của hãng vận chuyển trong đơn đa kiện | Phân định `PaymentStatus` (nghĩa vụ khách) với `CarrierSettlementStatus` và cụm 3 bảng `CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79) (CORR-07). |
| **Inventory Quarantine Quality** | Hàng cách ly hoặc hư hỏng bị bán nhầm cho khách hàng khác | Bộ 5 Declarative CHECK Constraints trên `InventoryBalance`: kiểm tra không âm và bảo đảm `OnHandQty >= ReservedQty + QuarantinedQty + DamagedQty` (CORR-02). |
| **LRM & Unit Refund Integrity** | Lệch tiền lẻ khi phân bổ voucher cấp đơn hoặc tranh chấp khi hoàn hàng lẻ cái | Áp dụng thuật toán LRM phân bổ chiết khấu nguyên số và công thức hiệu tích lũy $\Delta\text{Refund} = F(a+b) - F(a)$ với `UPDLOCK` trên `OrderLine` (CORR-03). |
| **Ledger Immutability** | Cố tình sửa đổi hoặc xóa nhật ký biến động kho và sổ cái thanh toán | Cấm quyền `UPDATE` và `DELETE` trên bảng `InventoryTransactionLine`, `PaymentTransaction` và `AuditLog`. Chỉ cho phép `INSERT` (Append-Only). |

---

## 5. KẾT LUẬN & KIẾN NGHỊ BÀN GIAO (AMENDED ARCHITECTURAL SIGN-OFF)

Sau khi rà soát toàn diện theo 6 nhóm tiêu chuẩn kỹ thuật, tiếp thu toàn bộ phản biện kiến trúc độc lập và ban hành bản ghi hiệu chỉnh [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md):
1. **Mô hình Khái niệm M2 đã hoàn thiện đạt chuẩn 100% (Baseline Amendment v2.0):** Chuẩn hóa toàn bộ **79 Thực thể (`E01` đến `E79`)** ánh xạ `1:1` sang tầng Logic M3. Toàn bộ 10 khiếm khuyết kỹ thuật (`CORR-01` đến `CORR-10`) đã có giải pháp thiết kế dứt điểm và khóa chặt bằng các ràng buộc toàn vẹn.
2. **Hóa giải triệt để các rủi ro vận hành & tài chính:** Khắc phục triệt để logic hoàn tiền, quản lý tồn kho cách ly theo chất lượng, phân bổ chiết khấu LRM và công thức lũy tiến $F(q)$ cho số lẻ tiền VND, đối soát COD hai đầu, mô hình TPT cho Account, và quản lý lịch sử giữ hàng $1:0..N$.
3. **Phê duyệt chuyển giao chính thức (Officially Cleared & Signed-off for M3):** Hồ sơ kiến trúc M2 cùng Bản ghi Hiệu chỉnh M2.08 chính thức được ký duyệt nghiệm thu chất lượng, sẵn sàng $100\%$ làm đầu vào chuẩn xác để tiến hành **Milestone 3 – Logical Database Design** (Ánh xạ Relational Schema, chuẩn hóa 3NF/BCNF, quy định kiểu dữ liệu T-SQL và thiết lập Constraints/Triggers).

