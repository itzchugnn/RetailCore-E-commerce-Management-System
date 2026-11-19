# M2.07 - PRE-M3 ARCHITECTURAL GATE REVIEW REPORT
## BÁO CÁO RÀ SOÁT KIẾN TRÚC MÔ HÌNH DỮ LIỆU KHÁI NIỆM TRƯỚC CHUYỂN GIAO M3

* **Dự án:** RetailCore – E-commerce Management System
* **Cột mốc:** Milestone 2 (M2 – Conceptual Design) $\to$ Milestone 3 (M3 – Logical Design)
* **Vai trò phê duyệt:** Lead Enterprise Database Engineer & Solution Architect
* **Mục tiêu:** Rà soát toàn diện 70 thực thể, 40 quan hệ liên miền và 5 khu vực rủi ro kiến trúc cao theo 6 nhóm tiêu chuẩn kỹ thuật trước khi đóng M2 và ánh xạ Relational Schema.
* **Quy chuẩn tài liệu:** Định dạng văn bản phân tích chuyên sâu, bảng biểu đối soát kỹ thuật, công thức ràng buộc toán học, không sử dụng hình ảnh.

---

## 1. TỔNG QUAN PHẠM VI RÀ SOÁT & TIÊU CHUẨN ĐÁNH GIÁ

Quá trình rà soát được thực hiện độc lập dựa trên 6 chiều kích thước kỹ thuật cốt lõi:
1. **Entity Ownership (Quyền sở hữu thực thể):** Đảm bảo nguyên tắc đơn trách nhiệm (Single Responsibility Principle) — mỗi thực thể thuộc đúng một Subject Area duy nhất, không chồng chéo thẩm quyền.
2. **Cross-Area Relationships (Quan hệ liên miền):** Rà soát tính đúng đắn của 40 liên kết liên miền (CR-01 đến CR-40), chiều phụ thuộc khóa ngoại và triệt tiêu vòng lặp phụ thuộc (Circular Dependency).
3. **Cardinality & Participation (Bản số & Tính tham gia):** Kiểm tra tính hợp lý của các cặp bản số $1:1, 1:N, N:N$ và tính tham gia Toàn phần / Một phần (Total / Partial Participation).
4. **Business Rule Consistency (Nhất quán quy tắc nghiệp vụ):** Phát hiện và hóa giải các xung đột logic giữa các phân hệ Sales, Inventory, Shipping, Payment, Promotion và Return.
5. **Data Lifecycle & History Preservation (Vòng đời dữ liệu & Bảo toàn lịch sử):** Kiểm tra cơ chế tạo, cập nhật, hủy giao dịch, snapshot lịch sử bất biến và nguyên tắc cấm xóa cứng (Append-Only / No Hard Delete).
6. **Completeness & M3 Readiness (Độ đầy đủ trước khi sang M3):** Rà soát các thuộc tính kỹ thuật cần thiết (Idempotency Key, Versioning, Trạng thái phân loại) để đảm bảo chuyển đổi mượt mà sang mô hình quan hệ 3NF/BCNF.

---

## 2. RÀ SOÁT CHI TIẾT THEO 6 CHIỀU KÍCH THƯỚC KIẾN TRÚC

### 2.1. Nhóm 1: Entity Ownership (Quyền sở hữu 70 Thực thể qua 11 Phân hệ)

| Subject Area | Thực thể trực thuộc | Rà soát phân định ranh giới (Boundary Review) | Đánh giá |
| :--- | :--- | :--- | :---: |
| **01. Product Catalog** | `Product` (E01), `Category` (E02), `ProductCategory` (E03), `ProductVariant` (E04), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09) | Sở hữu toàn bộ từ điển thuộc tính và cấu trúc SKU. Giá bán `SellingPrice` gắn tại `ProductVariant`, không đặt tại `Product`. | **Chuẩn xác (100%)** |
| **02. Warehouse & Inventory** | `Warehouse` (E10), `InventoryBalance` (E11), `OrderWarehouseAllocation` (E12), `StockReservation` (E13), `StockReservationLine` (E14), `InventoryTransaction` (E15), `InventoryTransactionLine` (E16), `StockCount` (E17), `StockCountLine` (E18), `WarehouseTransfer` (E19), `WarehouseTransferLine` (E20) | Sở hữu trọn vẹn số dư vật lý, kế hoạch phân bổ kho và chứng từ giữ tồn kho. Không gộp chứng từ kho với đơn hàng thương mại. | **Chuẩn xác (100%)** |
| **03. Procurement** | `Supplier` (E21), `SupplierProduct` (E22), `PurchaseOrder` (E23), `PurchaseOrderLine` (E24), `GoodsReceipt` (E25), `GoodsReceiptLine` (E26), `SupplierReturn` (E27), `SupplierReturnLine` (E28) | Khép kín chu trình mua hàng từ NCC. Tách bạch `PurchaseOrder` (cam kết mua) với `GoodsReceipt` (nhập kho thực tế). | **Chuẩn xác (100%)** |
| **04. Customer & Identity** | `Account` (E29), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `AccountVerification` (E37) | Phân tách tài khoản xác thực `Account` với hồ sơ nghiệp vụ `Customer`/`Employee` (Disjoint Subtype). `CustomerAddress` là sổ địa chỉ người dùng. | **Chuẩn xác (100%)** |
| **05. Shopping** | `ShoppingCart` (E38), `CartItem` (E39) | Quản lý trạng thái tiền giao dịch của khách hàng. Hoàn toàn tách biệt khỏi hợp đồng bán hàng chính thức. | **Chuẩn xác (100%)** |
| **06. Sales** | `SalesOrder` (E40), `OrderLine` (E41), `OrderAddress` (E42), `OrderStatusHistory` (E43) | Sở hữu hợp đồng bán hàng chính thức. `OrderAddress` snapshot địa chỉ độc lập với `CustomerAddress`. | **Chuẩn xác (100%)** |
| **07. Fulfillment & Shipping** | `Carrier` (E44), `Shipment` (E45), `ShipmentLine` (E46), `DeliveryAttempt` (E47), `ShipmentStatusHistory` (E48) | Sở hữu quy trình giao vận bưu kiện bên ngoài kho. Theo dõi chặt chẽ bưu tá và các lần giao thất bại. | **Chuẩn xác (100%)** |
| **08. Promotion & Voucher** | `Promotion` (E49), `PromotionCondition` (E50), `PromotionProduct` (E51), `Voucher` (E52), `PromotionUsage` (E53), `PromotionTargetCustomer` (E54) | Quản lý quy tắc giảm giá và quota tiêu thụ. `PromotionUsage` là sổ cái hạch toán chiết khấu. | **Chuẩn xác (100%)** |
| **09. Payment & Refund** | `PaymentMethod` (E55), `Payment` (E56), `PaymentTransaction` (E57), `Refund` (E58), `RefundTransaction` (E59) | Sở hữu nghĩa vụ tài chính và dòng tiền (thu tiền & chuyển hoàn tiền). Tách biệt khỏi khiếu nại đổi trả hàng. | **Chuẩn xác (100%)** |
| **10. Return & Exchange** | `ReturnRequest` (E60), `ReturnRequestLine` (E61), `ReturnReceipt` (E62), `ReturnReceiptLine` (E63), `ReturnStatusHistory` (E64) | Sở hữu nghiệp vụ RMA và kiểm định hiện vật tại kho bến nhập. Kích hoạt chứng từ hoàn tiền chứ không tự quản lý giao dịch tiền tệ. | **Chuẩn xác (100%)** |
| **11. Audit & Reporting** | `AuditLog` (E65), `AuditLogDetail` (E66), `ReportDefinition` (E67), `ReportExecution` (E68), `ReportSnapshot` (E69), `ReportPermission` (E70) | Quản lý an ninh vết kiểm toán và kết xuất báo cáo quản trị. Phân quyền báo cáo gắn kết với RBAC của Identity. | **Chuẩn xác (100%)** |

> **Kết luận Nhóm 1:** Cả 70 thực thể đều có ranh giới chủ quyền rõ ràng, không có thực thể nào bị trùng lặp trách nhiệm hoặc tranh chấp thẩm quyền giữa các phân hệ.

---

### 2.2. Nhóm 2: Cross-Area Relationships (Rà soát 40 Liên kết Liên miền CR-01 đến CR-40)

* **Phát hiện 1: Khử hoàn toàn vòng lặp phụ thuộc (Zero Circular Dependencies):**
  * Chuỗi liên kết bán hàng và kho vận:  
    `SalesOrder` $\to$ `OrderLine` $\to$ `OrderWarehouseAllocation` $\to$ `StockReservationLine` $\to$ `StockReservation` $\to$ `SalesOrder`.
  * *Kiểm chứng:* `StockReservation` trỏ khóa ngoại `OrderID` về `SalesOrder` để quản lý theo cấp chứng từ tổng, trong khi dòng chi tiết `StockReservationLine` trỏ `AllocationID` về dòng phân bổ kho. Đây là mô hình phân rã 2 cấp (Header-Line) song song, không tạo vòng lặp khóa ngoại cứng giữa `OrderLine` và `SalesOrder`.
* **Phát hiện 2: Ràng buộc tính nhất quán kho xuất hàng (Warehouse Consistency Constraint):**
  * Trong phân hệ Fulfillment: `Shipment` có `WarehouseID` (kho xuất bưu kiện) và `ShipmentLine` có `AllocationID` (tham chiếu đến `OrderWarehouseAllocation`).
  * *Ràng buộc bắt buộc ở M3:* $\text{Shipment.WarehouseID} \equiv \text{OrderWarehouseAllocation.WarehouseID}$. Một kiện hàng của một kho không thể đóng gói dòng phân bổ thuộc kho khác.

---

### 2.3. Nhóm 3: Cardinality & Participation Constraints (Bản số & Tính tham gia)

1. **Quan hệ $1:1$ Bắt buộc (Mandatory $1:1$):**
   * `SalesOrder` $\leftrightarrow$ `OrderAddress`: Đúng $1:1$. Mỗi đơn hàng bắt buộc có đúng 1 bản chụp địa chỉ giao hàng (`OrderAddress.OrderID` UNIQUE).
   * `SalesOrder` $\leftrightarrow$ `Payment`: Đúng $1:1$. Mỗi hợp đồng bán hàng chính thức gắn liền với duy nhất 1 hồ sơ thanh toán (`Payment.OrderID` UNIQUE). Dù thanh toán nhiều lần hoặc nhiều cổng, các lần thử đều nằm trong `PaymentTransaction` ($1:N$ với `Payment`).
2. **Quan hệ $1:0..1$ Tùy chọn (Optional $1:0..1$):**
   * `Account` $\leftrightarrow$ `Customer`: Tối đa 1 hồ sơ Customer.
   * `Account` $\leftrightarrow$ `Employee`: Tối đa 1 hồ sơ Employee.
   * *Ràng buộc loại trừ (Disjoint XOR):* Một `Account` chỉ được thuộc tối đa một subtype:
     $$\text{IsCustomer} \oplus \text{IsEmployee} = 1$$
   * `ShoppingCart` $\leftrightarrow$ `SalesOrder`: Một giỏ hàng chuyển đổi thành tối đa 1 đơn hàng (`SalesOrder.CartID` UNIQUE). Khi đã chuyển đổi, giỏ hàng khóa trạng thái `CONVERTED`.
3. **Tính tham gia Toàn phần của Thực thể Yếu (Total Participation):**
   * Toàn bộ các thực thể chi tiết dòng (`OrderLine`, `ShipmentLine`, `StockReservationLine`, `GoodsReceiptLine`, `ReturnRequestLine`, `ReturnReceiptLine`) đều có tính tham gia **Toàn phần (Total)** đối với thực thể Header cha. Cấm tuyệt đối dòng hàng mồ côi (Orphan lines).

---

### 2.4. Nhóm 4: Business Rule Consistency (Tính Nhất quán Quy tắc Nghiệp vụ)

* **Quy tắc Kiểm soát Tồn kho vs Trạng thái Đơn hàng:**
  * Khi tạo đơn hàng thành công (`SalesOrder = PENDING_PAYMENT`), hàng được khóa giữ bằng `StockReservation`. Tồn kho vật lý `OnHandQty` **chưa giảm**, chỉ có `ReservedQty` tăng:
    $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$$
  * Khi đơn hàng hết hạn thanh toán (sau 15 phút), Trigger/Job tự động hủy đơn (`OrderStatus = CANCELLED`), giải phóng giữ kho (`ReservationStatus = RELEASED`). `ReservedQty` giảm, `OnHandQty` giữ nguyên. Tuyệt đối không sinh giao dịch xuất nhập kho vật lý.
* **Quy tắc Xuất bãi vs Đang giao (In-Transit):**
  * Hàng chỉ thực sự trừ `OnHandQty` và tiêu thụ `ReservedQty` khi nhân viên kho đóng gói và bàn giao bưu kiện cho shipper (`ShipmentStatus = DISPATCHED`). Lúc này ghi nhận `InventoryTransaction` loại `SHIPMENT_OUT`.

---

### 2.5. Nhóm 5: Data Lifecycle & History Preservation (Bảo toàn Lịch sử Dữ liệu)

1. **Cơ chế Snapshot 3 Điểm Trọng Yếu:**
   * **Giá và Thuộc tính Sản phẩm (`OrderLine`):** Snapshot `UnitPrice`, `ProductName`, `VariantName`, và đặc biệt là chuỗi JSON `VariantAttributesSnapshot` (lưu vết chính xác: Size L, Màu Xanh Navy tại thời điểm mua). Dù sau này Catalog có tăng giá hay đổi tên thuộc tính, chứng từ đơn cũ không bao giờ bị biến động.
   * **Địa chỉ Giao hàng (`OrderAddress`):** Snapshot nguyên trạng người nhận, số điện thoại, tỉnh/huyện/xã, địa chỉ chi tiết. Khách hàng sửa sổ địa chỉ cá nhân (`CustomerAddress`) không làm sai lệch đơn hàng đang giao.
   * **Giá vốn Mua hàng (`GoodsReceiptLine`):** Snapshot `UnitCost` tại thời điểm ký biên bản nhập kho, phục vụ tính giá vốn bình quân gia quyền cho kế toán kho.
2. **Nguyên tắc Bất biến & Cấm Xóa Cứng (No Hard Delete):**
   * Các bảng chứng từ giao dịch (`SalesOrder`, `Payment`, `Shipment`, `ReturnRequest`, `InventoryTransaction`, `AuditLog`) **tuyệt đối cấm lệnh `DELETE`**.
   * Hủy đơn chuyển `OrderStatus = 'CANCELLED'`. Thất bại chuyển `PaymentStatus = 'FAILED'`.
   * Danh mục dùng cờ xóa mềm: `IsActive = 0`.

---

### 2.6. Nhóm 6: Completeness & M3 Readiness (Độ Đầy đủ Kỹ thuật cho M3)

* **Định danh Khóa chính và Mã tự nhiên:**
  * Toàn bộ 70 thực thể đều đã xác định rõ Surrogate Key (`bigint IDENTITY`) hoặc Composite Key hợp lý.
  * Các thực thể nghiệp vụ cốt lõi đều có Natural Business Key duy nhất: `ProductCode`, `SKU`, `WarehouseCode`, `SupplierCode`, `PONumber`, `ReceiptNumber`, `OrderNumber`, `ShipmentNumber`, `VoucherCode`, `TransactionReference`, `ReturnNumber`, `ReportCode`.
* **Idempotency & Concurrency:**
  * Đã chuẩn bị sẵn trường `TransactionReference` trên `PaymentTransaction` và `RefundTransaction` để cài đặt Unique Index, chống trùng lặp Webhook từ cổng thanh toán.

---

## 3. RÀ SOÁT CHUYÊN SÂU 5 KHU VỰC RỦI RO THIẾT KẾ CAO (HIGH-RISK AREAS)

### 3.1. Rủi ro 1: Quan hệ Phân bổ Kho vs Giữ Tồn Kho (Allocation – Reservation)

* **Vấn đề rủi ro:** Một đơn hàng có thể gồm nhiều sản phẩm nằm ở các kho khác nhau (Split Shipment). Nếu thiết kế giữ hàng gắn cứng vào `SalesOrder` mà không định danh kho, hệ thống sẽ không biết trừ tồn tại kho nào khi xuất bãi.
* **Giải pháp kiến trúc đã chốt:**
  * `OrderWarehouseAllocation` (E12) đóng vai trò là **kế hoạch phân bổ**.
  * `StockReservation` (E13) và `StockReservationLine` (E14) đóng vai trò là **khóa giữ vật lý tại kho**.
  * Cơ chế liên kết: Mỗi dòng `StockReservationLine` liên kết trực tiếp với một `AllocationID` và `WarehouseID`.
  * **Xử lý tình huống Hủy / Hết hạn đơn hàng:**
    * Nếu khách không thanh toán trong 15 phút: Hệ thống hủy `StockReservation`, hoàn lại `ReservedQty` tại đúng kho đó. Kế hoạch `OrderWarehouseAllocation` chuyển sang `CANCELLED`. Tồn vật lý `OnHandQty` không bị tác động sai lệch.

---

### 3.2. Rủi ro 2: Thanh toán COD vs Đối soát Thu hộ Tiền mặt (Payment – COD Collection)

* **Vấn đề rủi ro:** Nếu gộp chung trạng thái đơn hàng và thanh toán, đơn COD sẽ bị bế tắc: Đơn đã giao cho khách (`Shipment = DELIVERED`) nhưng tiền mặt shipper cầm chưa nộp về cho công ty.
* **Giải pháp kiến trúc đã chốt:**
  * Tách độc lập 3 máy trạng thái:
    $$\text{SalesOrder.OrderStatus} \neq \text{Payment.PaymentStatus} \neq \text{Shipment.ShipmentStatus}$$
  * **Vòng đời đối soát đơn COD:**
    1. Khi đặt hàng: `OrderStatus = CONFIRMED`, `PaymentStatus = PENDING`, `ShipmentStatus = PREPARING`.
    2. Xuất kho giao hàng: `OrderStatus = IN_TRANSIT`, `PaymentStatus = PENDING`, `ShipmentStatus = IN_TRANSIT`.
    3. Khách nhận hàng và trả tiền mặt cho shipper: `ShipmentStatus = DELIVERED`. Lúc này đơn hàng `OrderStatus = DELIVERED`, nhưng `PaymentStatus` vẫn là `PENDING` (tiền đang nằm ở tài khoản trung gian của hãng vận chuyển).
    4. Đối soát bưu tá / Hãng vận chuyển chuyển khoản tiền COD: Hệ thống ghi nhận một `PaymentTransaction` loại `COD_SETTLEMENT` kèm mã bảng kê. Lúc này `Payment.AmountPaid` được cập nhật đủ và `PaymentStatus` chính thức chuyển sang `PAID`.

---

### 3.3. Rủi ro 3: Chu trình Logistics Ngược (Return – Refund – Inventory)

* **Vấn đề rủi ro:** Khách trả hàng thì tiền hoàn trước hay hàng về trước? Nếu hoàn tiền trước mà hàng chuyển về bị mất hoặc vỡ nát thì doanh nghiệp chịu thất thoát. Nếu hàng nhập kho tự động cộng tồn kho thì hàng hỏng sẽ bị bán tiếp cho khách khác.
* **Giải pháp kiến trúc đã chốt (Quy trình 4 chặng nghiêm ngặt):**
  1. **Chặng 1 (Tiếp nhận & Duyệt RMA):** Khách gửi `ReturnRequest`, CSKH kiểm tra tính hợp lệ và duyệt `APPROVED`, cấp mã trả hàng và chỉ định duy nhất một `WarehouseID` tiếp nhận.
  2. **Chặng 2 (Tiếp nhận tại kho):** Bưu kiện về đến bến nhập, thủ kho lập `ReturnReceipt` ghi nhận `ReceivedQty`. **Chưa tăng tồn kho bán.**
  3. **Chặng 3 (Kiểm định chất lượng - Disposition):** Nhân viên kiểm định ghi nhận `ReturnReceiptLine.Disposition`:
     * `SELLABLE` (Hàng mới, nguyên tem): Sinh `InventoryTransaction` loại `CUSTOMER_RETURN_IN`, cộng lại vào `OnHandQty` để bán tiếp.
     * `QUARANTINE` (Chờ giám định hãng): Chuyển vào kho cách ly, không cộng tồn bán được.
     * `DISPOSED` (Hỏng hóc, tiêu hủy): Ghi nhận chi phí tổn thất, cấm đưa vào tồn bán.
  4. **Chặng 4 (Kích hoạt hoàn tiền):** Chỉ sau khi `ReturnReceipt` có dòng `Disposition = SELLABLE` được `POSTED`, hệ thống mới tự động tạo chứng từ `Refund` bên Subject Area 09 theo tỷ lệ hàng thực nhận.

---

### 3.4. Rủi ro 4: Phân bổ Chiết khấu Khuyến mãi khi Hoàn tiền (PromotionUsage – Payment – Refund)

* **Vấn đề rủi ro:** Đơn hàng áp Voucher giảm 100.000đ cho đơn từ 500.000đ. Khách mua 2 áo (mỗi áo 250.000đ), tổng thanh toán 400.000đ. Sau đó khách trả lại 1 áo. Nếu hoàn đủ 250.000đ theo giá niêm yết thì doanh nghiệp bị lỗ do khách gian lận voucher.
* **Giải pháp kiến trúc đã chốt:**
  * Bảng `PromotionUsage` lưu vết chi tiết số tiền chiết khấu phân bổ cho từng dòng `OrderLineID`:
    $$\text{LineRefundableAmount} = \text{OrderLine.UnitPrice} - \text{AllocatedVoucherDiscount} - \text{LineProductDiscount}$$
  * Khi duyệt `Refund`, hệ thống căn cứ chính xác vào số tiền thực thu sau chiết khấu được truy vết từ `PromotionUsage` để sinh `Refund.RefundAmount`.
  * Ràng buộc kiểm soát ngân sách hoàn tiền:
    $$\sum \text{Refund.RefundAmount} \le \text{Payment.AmountPaid} - \text{PreviousRefundedAmount}$$

---

### 3.5. Rủi ro 5: Tính Bất biến của Dữ liệu Lịch sử (Historical Snapshots)

* **Vấn đề rủi ro:** Khách hàng thay đổi địa chỉ trong sổ địa chỉ, hoặc bộ phận Marketing sửa tên sản phẩm/tăng giá biến thể làm biến dạng dữ liệu kế toán và hóa đơn của các đơn hàng cũ.
* **Giải pháp kiến trúc đã chốt:**
  * **Tách rời hoàn toàn Entity Sổ địa chỉ và Entity Đơn hàng:**
    * `CustomerAddress`: Là sổ địa chỉ cá nhân, khách hàng có quyền thêm, sửa, xóa, đổi mặc định tùy ý.
    * `OrderAddress`: Là một thực thể riêng biệt mang bản chất **Weak Snapshot**, sao chép nguyên văn thông tin địa chỉ tại giây phút đặt hàng. Bảng này không có khóa ngoại trỏ về `CustomerAddress` mà chỉ trỏ về `SalesOrder.OrderID`.
  * **Snapshot Dòng hàng đa tầng:**
    * `OrderLine.UnitPrice`: Snapshot giá bán tại thời điểm bấm đặt hàng.
    * `OrderLine.ProductName` & `VariantName`: Snapshot tên hiển thị.
    * `OrderLine.VariantAttributesSnapshot`: Lưu toàn bộ cặp `AttributeName: AttributeValue` dưới dạng chuỗi chuẩn hóa JSON (ví dụ: `{"Size": "XL", "Color": "Navy Blue"}`).

---

## 4. MA TRẬN ĐỐI SOÁT & DANH MỤC KHUYẾN NGHỊ KỸ THUẬT CHO M3

| Khu vực kỹ thuật | Rủi ro tiềm ẩn | Cơ chế kiểm soát bắt buộc chuyển giao sang M3 (Logical Design) |
| :--- | :--- | :--- |
| **Identity Subtypes** | Một tài khoản vừa là Customer vừa là Employee gây xung đột bảo mật | Thiết lập Trigger hoặc Check Constraint loại trừ: `(AccountType = 'CUSTOMER' AND CustomerID IS NOT NULL) OR (AccountType = 'EMPLOYEE' AND EmployeeID IS NOT NULL)`. |
| **Active Cart Limit** | Khách hàng mở nhiều giỏ hàng `ACTIVE` gây lỗi đồng bộ | Tạo Filtered Unique Index trên SQL Server: `CREATE UNIQUE INDEX UX_Customer_ActiveCart ON ShoppingCart(CustomerID) WHERE CartStatus = 'ACTIVE'`. |
| **Allocation Integrity** | Tổng số lượng phân bổ kho lệch với số lượng khách đặt | Ràng buộc kiểm toán: $\sum \text{AllocatedQty} \equiv \text{OrderLine.OrderedQty}$. Kiểm soát bằng Stored Procedure tạo đơn All-or-Nothing. |
| **Ledger Immutability** | Cố tình sửa đổi hoặc xóa nhật ký biến động kho và sổ cái thanh toán | Cấm quyền `UPDATE` và `DELETE` trên bảng `InventoryTransactionLine` và `PaymentTransaction`. Chỉ cho phép `INSERT` (Append-Only). |
| **Voucher Quota Abuse** | Tranh chấp đồng thời làm số lượt dùng voucher vượt quá hạn mức | Áp dụng Transaction Isolation Level `SERIALIZABLE` hoặc cơ chế Pessimistic Locking (`UPDLOCK, HOLDLOCK`) trong Stored Procedure áp mã voucher. |

---

## 5. KẾT LUẬN & KIẾN NGHỊ BÀN GIAO (ARCHITECTURAL SIGN-OFF)

Sau khi rà soát toàn diện theo 6 nhóm tiêu chuẩn kỹ thuật và đánh giá sâu 5 khu vực rủi ro cao:
1. **Mô hình Khái niệm M2 đạt tính toàn vẹn 100%:** 70 thực thể và 40 quan hệ liên miền đã được chuẩn hóa chặt chẽ, khép kín về mặt nghiệp vụ, không có quan hệ mồ côi và bảo đảm nguyên tắc bảo toàn dữ liệu tài chính.
2. **Hóa giải triệt để các rủi ro vận hành:** Các cơ chế phân bổ kho (Split Shipment), đối soát COD hai bước, logistics ngược (RMA 4 chặng) và snapshot lịch sử đã có phương án giải quyết cụ thể, khả thi trên SQL Server.
3. **Phê duyệt chuyển giao (Approved for M3):** Hồ sơ kiến trúc M2 chính thức được ký duyệt nghiệm thu chất lượng, sẵn sàng $100\%$ làm đầu vào chuẩn xác để tiến hành **Milestone 3 – Logical Database Design** (Ánh xạ Relational Schema, chuẩn hóa 3NF/BCNF, quy định kiểu dữ liệu T-SQL và thiết lập Constraints).

