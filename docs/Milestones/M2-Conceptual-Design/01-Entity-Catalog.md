# M2.01 - ENTITY CATALOG (DANH MỤC THỰC THỂ KHÁI NIỆM)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.1 – Standardized Attribute Names & Multi-Area Catalog
* **Kế thừa:** Business Rules v1.1, Data Requirements v1.1, Quyết định BD-01 đến BD-07 & PC-Q01 đến PC-Q06

---

## 1. Phân loại Loại hình Thực thể trong Mô hình Quan niệm

1. **Thực thể Mạnh (Strong Entity):** Tồn tại độc lập, có khóa định danh riêng trong thế giới thực (`Product`, `Warehouse`, `Category`, `Attribute`, `Supplier`, `User`, `InventoryTransaction`, `StockCount`, `StockReservation`).
2. **Thực thể Phụ thuộc / Yếu (Dependent / Weak Entity):** Sự tồn tại phụ thuộc vào thực thể cha, định danh gắn liền với cha (`AttributeValue`, `StockReservationLine`, `InventoryTransactionLine`, `StockCountLine`, `CustomerAddress`, `CartLine`, `OrderLine`, `ShipmentLine`).
3. **Thực thể Biến thể Trung tâm (Core SKU Entity):** `ProductVariant` – Có surrogate key riêng nhưng phụ thuộc logic vào `Product`, mang mã tự nhiên `SKU` duy nhất toàn hệ thống.
4. **Thực thể Liên kết (Associative Entity):** Giải quyết quan hệ $N-N$ và mang thuộc tính nghiệp vụ riêng (`ProductCategory` mang cờ `IsPrimary`, `ProductAttribute` mang `AttributeRole`, `ProductAttributeValue`, `VariantAttributeValue`, `InventoryBalance`, `OrderWarehouseAllocation`).

---

## 2. Subject Area 01: Product Catalog (Đã khóa Baseline v1.0)

### 2.1. Danh mục 9 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E01** | `Product` | Strong Entity | Đại diện cho sản phẩm ở cấp độ thương mại, không phụ thuộc phiên bản. | `ProductID`, `ProductName`, `Description`, `Brand`, `ProductStatus`, `CreatedAt` | **PK:** `ProductID`<br>*(UK: ProductCode nếu có)* |
| **E02** | `Category` | Strong Entity (Đệ quy) | Danh mục phân loại sản phẩm theo cấu trúc cây phân cấp đa tầng. | `CategoryID`, `CategoryName`, `Description`, `ParentCategoryID`, `CategoryStatus` | **PK:** `CategoryID` |
| **E03** | `ProductCategory` | Associative Entity | Ghi nhận sản phẩm thuộc danh mục nào ($N-N$) kèm cờ danh mục chính. | `ProductID`, `CategoryID`, `IsPrimary`, `AssignedAt` | **PK:** `(ProductID, CategoryID)` |
| **E04** | `ProductVariant` | Core SKU Entity | Phiên bản có thể bán và quản lý tồn kho, tương ứng một mã SKU vật lý. | `VariantID`, `ProductID`, `SKU`, `VariantName`, `SellingPrice`, `VariantStatus` | **PK:** `VariantID`<br>**UK:** `SKU` |
| **E05** | `Attribute` | Strong Entity | Định nghĩa loại thuộc tính dùng chung trong toàn bộ hệ thống catalog. | `AttributeID`, `AttributeName`, `DataType`, `AttributeDescription` | **PK:** `AttributeID` |
| **E06** | `AttributeValue` | Weak Entity | Giá trị chuẩn hóa của một thuộc tính trong từ điển dùng chung. | `AttributeValueID`, `AttributeID`, `Value`, `DisplayValue` | **PK:** `AttributeValueID`<br>*(UK: AttributeID, Value)* |
| **E07** | `ProductAttribute` | Associative Entity | Khai báo thuộc tính được áp dụng cho sản phẩm và vai trò của nó. | `ProductID`, `AttributeID`, `AttributeRole`, `IsRequired`, `DisplayOrder` | **PK:** `(ProductID, AttributeID)` |
| **E08** | `ProductAttributeValue` | Associative Entity | Tập các giá trị được phép (Whitelist) của thuộc tính cho sản phẩm cụ thể. | `ProductID`, `AttributeID`, `AttributeValueID` | **PK:** `(ProductID, AttributeID, AttributeValueID)` |
| **E09** | `VariantAttributeValue` | Associative Entity | Giá trị thuộc tính thực tế được chọn cấu thành nên một SKU cụ thể. | `VariantID`, `AttributeID`, `AttributeValueID` | **PK:** `(VariantID, AttributeID)` |

### 2.2. Các quy ước nghiệp vụ của Product Catalog
1. **Quy ước vai trò thuộc tính (`ProductAttribute.AttributeRole`):**
   * `DESCRIPTIVE`: Thuộc tính mô tả chung, không tham gia phân biệt SKU (ví dụ: Chất liệu, Xuất xứ, Hướng dẫn bảo quản).
   * `VARIANT_DEFINING`: Thuộc tính định nghĩa biến thể, bắt buộc tham gia tổ hợp SKU (ví dụ: Kích thước, Màu sắc).
   * *Quy tắc loại trừ:* Một thuộc tính trong một sản phẩm chỉ nhận duy nhất một vai trò.
2. **Quy ước SKU mặc định:** Nếu sản phẩm không có thuộc tính định nghĩa biến thể (sản phẩm đơn chiếc), hệ thống tự sinh một `ProductVariant` mặc định với tổ hợp thuộc tính rỗng.
3. **Danh mục chính:** Sản phẩm Published bắt buộc có đúng 1 danh mục chính (`IsPrimary = 1`).

---

## 3. Subject Area 02: Warehouse & Inventory (Baseline đã khóa)

### 3.1. Danh mục 11 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E10** | `Warehouse` | Strong Entity | Kho hàng vật lý thuộc mạng lưới phân phối của doanh nghiệp. | `WarehouseID`, `WarehouseCode`, `WarehouseName`, `Address`, `WarehouseStatus`, `CreatedAt` | **PK:** `WarehouseID`<br>**UK:** `WarehouseCode` |
| **E11** | `InventoryBalance` | Associative Entity | Số dư tồn kho tức thời của một SKU tại một kho hàng vật lý. | `WarehouseID`, `VariantID`, `OnHandQty`, `ReservedQty`, `LastUpdatedAt` | **PK:** `(WarehouseID, VariantID)` |
| **E12** | `OrderWarehouseAllocation` | Associative Entity | Quyết định phân bổ một phần số lượng của dòng đơn hàng cho một kho. | `AllocationID`, `OrderLineID`, `WarehouseID`, `VariantID`, `AllocatedQty`, `AllocationStatus`, `AllocatedAt` | **PK:** `AllocationID` |
| **E13** | `StockReservation` | Strong / Document Entity | Chứng từ/Phiếu giữ hàng tạm thời gắn với đơn hàng (Header). | `ReservationID`, `OrderID`, `ReservationStatus`, `CreatedAt`, `ExpiresAt`, `ReleasedAt` | **PK:** `ReservationID` |
| **E14** | `StockReservationLine` | Weak Entity | Chi tiết số lượng SKU được giữ tại một kho cụ thể (Line). | `ReservationLineID`, `ReservationID`, `AllocationID`, `VariantID`, `WarehouseID`, `ReservedQty`, `ReservationLineStatus` | **PK:** `ReservationLineID` |
| **E15** | `InventoryTransaction` | Strong / Document Entity | Chứng từ sổ cái ghi nhận nghiệp vụ làm thay đổi tồn kho (Header). | `TransactionID`, `TransactionType`, `TransactionStatus`, `ReferenceType`, `ReferenceID`, `CreatedAt`, `PostedAt` | **PK:** `TransactionID` |
| **E16** | `InventoryTransactionLine` | Weak Entity | Chi tiết dòng biến động tồn kho theo SKU và kho (`OnHandDelta`, `ReservedDelta`). | `TransactionLineID`, `TransactionID`, `VariantID`, `WarehouseID`, `OnHandDelta`, `ReservedDelta` | **PK:** `TransactionLineID` |
| **E17** | `StockCount` | Strong / Document Entity | Phiếu kiểm kê tồn kho định kỳ hoặc đột xuất tại một kho (Header). | `StockCountID`, `WarehouseID`, `CountStatus`, `StartedAt`, `CompletedAt`, `ApprovedAt`, `ApprovedBy` | **PK:** `StockCountID` |
| **E18** | `StockCountLine` | Weak Entity | Chi tiết ghi nhận số lượng hệ thống và số lượng thực tế kiểm đếm (Line). | `StockCountLineID`, `StockCountID`, `VariantID`, `SystemQty`, `CountedQty`, `VarianceQty` | **PK:** `StockCountLineID` |
| **E19** | `WarehouseTransfer` | Strong / Document Entity | Phiếu điều chuyển hàng hóa giữa hai kho vật lý trong mạng lưới (Header). | `TransferID`, `TransferNumber`, `SourceWarehouseID`, `DestinationWarehouseID`, `TransferStatus`, `CreatedAt`, `DispatchedAt`, `ReceivedAt`, `CreatedBy`, `ApprovedBy` | **PK:** `TransferID`<br>**UK:** `TransferNumber` |
| **E20** | `WarehouseTransferLine` | Weak Entity | Chi tiết SKU và số lượng xuất chuyển / thực nhận giữa hai kho (Line). | `TransferLineID`, `TransferID`, `VariantID`, `TransferQty`, `ReceivedQty`, `VarianceQty` | **PK:** `TransferLineID` |

### 3.2. Cơ chế và Quy tắc cốt lõi đã chốt của Warehouse & Inventory
1. **Công thức tồn khả dụng & Nguyên tắc All-or-nothing (WI-Q02):**
   $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$$
   * Giao dịch checkout đơn hàng chỉ được xác nhận thành công khi hệ thống khóa giữ đủ 100% số lượng cần thiết cho toàn bộ các dòng hàng trong đơn (All-or-nothing).
2. **Cơ chế ghi sổ hai chiều (Dual-Delta Ledger):**
   * Mỗi dòng `InventoryTransactionLine` lưu trực tiếp `OnHandDelta` (mức tăng/giảm tồn vật lý) và `ReservedDelta` (mức tăng/giảm tồn giữ).
   * Mọi chứng từ sau khi `POSTED` là bất biến (Immutable), không được sửa đổi trực tiếp.
3. **Quy trình chuyển kho hai bước & Trạng thái In-Transit (WI-Q04):**
   * **Bước 1 (Xuất kho nguồn):** Ghi nhận `TRANSFER_OUT`, giảm `OnHandQty` tại kho nguồn. Hàng chuyển sang trạng thái In-Transit (`DISPATCHED`). Hàng đang trên đường không thuộc `OnHandQty` của bất kỳ kho nào.
   * **Bước 2 (Nhập kho đích):** Kho đích kiểm đếm thực nhận và ghi nhận `TRANSFER_IN`, chỉ tăng `OnHandQty` tại kho đích theo `ReceivedQty` thực tế. Nếu có chênh lệch (`VarianceQty`), ghi nhận xử lý riêng, không tự động làm tròn.
4. **Quy trình kiểm kê và phê duyệt chênh lệch (WI-Q05, WI-Q06):**
   * Phiếu kiểm kê hoàn tất kiểm đếm (`COMPLETED`) tuyệt đối không tự động cập nhật số dư kho.
   * Chênh lệch chỉ được hạch toán khi có cấp quản lý phê duyệt (`ApprovedAt`, `ApprovedBy`). Sau khi duyệt, hệ thống tự động sinh `InventoryTransaction` loại `STOCK_ADJUSTMENT` để cập nhật `InventoryBalance`. Người kiểm đếm không được tự duyệt phiếu của chính mình (Separation of Duties).

---

## 4. Subject Area 03: Procurement (Baseline đã khóa)

### 4.1. Danh mục 8 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E21** | `Supplier` | Strong / Master | Nhà cung cấp hàng hóa cho doanh nghiệp. | `SupplierID`, `SupplierCode`, `SupplierName`, `TaxCode`, `Email`, `Phone`, `Address`, `SupplierStatus`, `CreatedAt` | **PK:** `SupplierID`<br>**UK:** `SupplierCode` |
| **E22** | `SupplierProduct` | Associative Entity | Quan hệ cung ứng giữa nhà cung cấp và SKU (giá tham khảo, lead time). | `SupplierID`, `VariantID`, `SupplierSKU`, `PurchasePrice`, `LeadTimeDays`, `IsPreferred` | **PK:** `(SupplierID, VariantID)` |
| **E23** | `PurchaseOrder` | Strong / Document Entity | Đơn đặt hàng mua gửi nhà cung cấp (Header). | `PurchaseOrderID`, `PONumber`, `SupplierID`, `DestinationWarehouseID`, `OrderDate`, `ExpectedDeliveryDate`, `PurchaseOrderStatus`, `CreatedBy`, `ApprovedBy`, `ApprovedAt` | **PK:** `PurchaseOrderID`<br>**UK:** `PONumber` |
| **E24** | `PurchaseOrderLine` | Weak / Detail Entity | Chi tiết mặt hàng SKU và số lượng đặt mua (Line). | `PurchaseOrderLineID`, `PurchaseOrderID`, `VariantID`, `OrderedQty`, `UnitPrice`, `LineAmount` | **PK:** `PurchaseOrderLineID` |
| **E25** | `GoodsReceipt` | Strong / Document Entity | Phiếu tiếp nhận một đợt hàng thực tế từ nhà cung cấp (Header). | `GoodsReceiptID`, `ReceiptNumber`, `PurchaseOrderID`, `WarehouseID`, `ReceivedAt`, `ReceiptStatus`, `ReceivedBy`, `PostedAt` | **PK:** `GoodsReceiptID`<br>**UK:** `ReceiptNumber` |
| **E26** | `GoodsReceiptLine` | Weak / Detail Entity | Chi tiết kiểm nhận SKU trong một đợt giao (Line). | `GoodsReceiptLineID`, `GoodsReceiptID`, `PurchaseOrderLineID`, `ReceivedQty`, `AcceptedQty`, `RejectedQty`, `UnitCost`, `RejectionReason` | **PK:** `GoodsReceiptLineID` |
| **E27** | `SupplierReturn` | Strong / Document Entity | Phiếu trả hàng cho nhà cung cấp (Header). | `SupplierReturnID`, `ReturnNumber`, `SupplierID`, `WarehouseID`, `ReturnStatus`, `CreatedAt`, `Reason`, `ApprovedBy`, `ApprovedAt` | **PK:** `SupplierReturnID`<br>**UK:** `ReturnNumber` |
| **E28** | `SupplierReturnLine` | Weak / Detail Entity | Chi tiết SKU và số lượng trả lại cho nhà cung cấp (Line). | `SupplierReturnLineID`, `SupplierReturnID`, `GoodsReceiptLineID`, `ReturnQty`, `ReturnReason` | **PK:** `SupplierReturnLineID` |

### 4.2. Các quy ước và Bất biến Nghiệp vụ Procurement đã chốt
1. **Một PO – Một Kho nhận cố định (PR-Q01):**
   * `PurchaseOrder.DestinationWarehouseID` xác định duy nhất kho đích tiếp nhận hàng của toàn bộ PO.
   * `GoodsReceipt.WarehouseID` bắt buộc phải trùng khớp với `PurchaseOrder.DestinationWarehouseID`.
2. **Nhận hàng nhiều đợt & Giới hạn số lượng (PR-Q02, PR-Q03):**
   * Cho phép nhận hàng thành nhiều đợt qua nhiều phiếu `GoodsReceipt` ($1:N$).
   * Bất biến trên từng dòng tiếp nhận:
     $$\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$$
   * Bất biến lũy kế kiểm soát không nhận vượt:
     $$\sum \text{GoodsReceiptLine.ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$$
3. **Phân loại chất lượng kiểm nhận & Kiểm soát tồn kho (PR-Q04):**
   * Chỉ `AcceptedQty` trên phiếu nhập đã xác nhận (`ReceiptStatus = 'COMPLETED'`) mới sinh giao dịch `PURCHASE_RECEIPT` và tăng `OnHandQty` tại kho.
   * `RejectedQty` (hàng lỗi, móp méo, sai quy cách) được ghi nhận riêng kèm `RejectionReason`, tuyệt đối không đưa vào `InventoryBalance.OnHandQty` như hàng đạt chuẩn.
4. **Vòng đời xuất trả nhà cung cấp (PR-Q05):**
   * Hàng trả NCC được quản lý qua `SupplierReturn` và `SupplierReturnLine` (tham chiếu nguồn gốc `GoodsReceiptLineID`).
   * Trả hàng sau khi đã nhập kho (`AcceptedQty` lỗi sau) sẽ sinh chứng từ `SUPPLIER_RETURN_OUT` làm giảm `OnHandQty` tại kho xuất trả. Tổng số lượng trả cộng dồn không được vượt quá số lượng thực tế còn có thể trả của dòng `GoodsReceiptLine`.
5. **Phân tách trách nhiệm phê duyệt PO (PR-Q06):**
   * PO bắt buộc phải được phê duyệt (`ApprovedBy`, `ApprovedAt`) trước khi gửi nhà cung cấp (`SENT`). Chỉ PO ở trạng thái hợp lệ mới được phép tạo `GoodsReceipt`.
