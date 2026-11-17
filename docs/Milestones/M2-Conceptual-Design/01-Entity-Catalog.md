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

---

## 5. Subject Area 04: Customer & Identity (Baseline đã khóa)

### 5.1. Danh mục 9 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E29** | `Account` | Strong Entity | Tài khoản xác thực danh tính dùng chung để truy cập hệ thống. | `AccountID`, `Email`, `Phone`, `PasswordHash`, `AccountStatus`, `CreatedAt`, `LastLoginAt` | **PK:** `AccountID`<br>**UK:** `Email` *(Filtered)*, `Phone` *(Filtered)* |
| **E30** | `Customer` | Subtype / Profile | Hồ sơ nghiệp vụ của khách hàng mua sắm B2C. | `CustomerID`, `AccountID`, `FullName`, `DateOfBirth`, `Gender`, `CustomerStatus`, `CreatedAt` | **PK:** `CustomerID`<br>**UK:** `AccountID` |
| **E31** | `Employee` | Subtype / Profile | Hồ sơ nhân sự nội bộ vận hành hệ thống RetailCore. | `EmployeeID`, `AccountID`, `EmployeeCode`, `FullName`, `Department`, `EmployeeStatus`, `CreatedAt` | **PK:** `EmployeeID`<br>**UK:** `AccountID`, `EmployeeCode` |
| **E32** | `Role` | Strong / Master | Danh mục vai trò trong hệ thống (RBAC). | `RoleID`, `RoleName`, `RoleDescription` | **PK:** `RoleID`<br>**UK:** `RoleName` |
| **E33** | `Permission` | Strong / Master | Danh mục quyền hạn cụ thể trên từng tài nguyên/hành động. | `PermissionID`, `PermissionCode`, `PermissionName`, `Description` | **PK:** `PermissionID`<br>**UK:** `PermissionCode` |
| **E34** | `AccountRole` | Associative Entity | Gán một hoặc nhiều vai trò cho tài khoản người dùng. | `AccountID`, `RoleID`, `AssignedAt`, `AssignedBy` | **PK:** `(AccountID, RoleID)` |
| **E35** | `RolePermission` | Associative Entity | Ma trận phân quyền chi tiết cho từng vai trò. | `RoleID`, `PermissionID` | **PK:** `(RoleID, PermissionID)` |
| **E36** | `CustomerAddress` | Weak Entity | Sổ địa chỉ giao hàng / liên hệ của khách hàng. | `AddressID`, `CustomerID`, `RecipientName`, `RecipientPhone`, `Province`, `District`, `Ward`, `StreetAddress`, `IsDefault`, `AddressStatus`, `CreatedAt` | **PK:** `AddressID` |
| **E37** | `AccountVerification` | Weak / Event | Ghi nhận lịch sử và trạng thái yêu cầu xác minh Email / Phone. | `VerificationID`, `AccountID`, `VerificationType`, `VerificationTarget`, `VerificationStatus`, `CreatedAt`, `ExpiresAt`, `VerifiedAt` | **PK:** `VerificationID` |

### 5.2. Các quy ước và Bất biến Nghiệp vụ Customer & Identity đã chốt
1. **Nguyên tắc loại trừ hồ sơ (CID-Q01, BR-ID04):**
   * Một `Account` chỉ được liên kết với một loại hồ sơ duy nhất: hoặc `Customer`, hoặc `Employee` (Ràng buộc XOR). Không bao giờ tồn tại một tài khoản vừa là khách hàng vừa là nhân viên trên cùng định danh.
2. **Phân quyền RBAC linh hoạt (CID-Q02, BR-ID05..07):**
   * Một tài khoản có thể được cấp nhiều `Role` qua `AccountRole`. Không cấp trùng lặp vai trò cho cùng một tài khoản.
   * Quyền hiệu lực của tài khoản là hợp (Union) các `Permission` của tất cả các `Role` mà tài khoản đó sở hữu.
3. **Quy ước định danh đăng nhập (CID-Q03, CID-Q04, BR-ID01..03):**
   * Bắt buộc có ít nhất một trong hai thông tin `Email` hoặc `Phone` (`CHECK (Email IS NOT NULL OR Phone IS NOT NULL)`).
   * Cả `Email` và `Phone` nếu được cung cấp phải là duy nhất toàn hệ thống (`Filtered Unique Index`).
   * Mật khẩu lưu dưới dạng giá trị băm bảo mật (`PasswordHash`), tuyệt đối không lưu văn bản gốc.
4. **Quản lý địa chỉ mặc định (CID-Q06, BR-ID08..11):**
   * Mỗi khách hàng có tối đa đúng một địa chỉ mặc định (`IsDefault = 1`). Cài đặt bằng `UNIQUE FILTERED INDEX (CustomerID) WHERE IsDefault = 1`.
   * Địa chỉ đầu tiên của khách hàng tự động được đặt làm mặc định. Khi địa chỉ mặc định bị vô hiệu hóa, phải chỉ định địa chỉ hợp lệ khác làm mặc định.
5. **Lịch sử xác minh bảo mật (CID-Q05, BR-ID12..14):**
   * `AccountVerification` lưu vết mọi yêu cầu xác thực (`EMAIL`, `PHONE`) kèm thời hạn `ExpiresAt`. Không sử dụng lại yêu cầu đã hết hạn hoặc đã xác minh thành công. Mã xác minh không lưu dạng thô (lưu hash).

---

## 6. Subject Area 05: Shopping (Baseline đã khóa)

### 6.1. Danh mục 2 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E38** | `ShoppingCart` | Strong / Document Entity | Giỏ hàng thuộc về khách hàng, lưu trạng thái mua sắm hiện tại. | `CartID`, `CustomerID`, `CartStatus`, `CreatedAt`, `UpdatedAt`, `LastActivityAt` | **PK:** `CartID`<br>**UK:** `CustomerID` *(Filtered: CartStatus = 'ACTIVE')* |
| **E39** | `CartItem` | Weak / Detail Entity | Dòng sản phẩm/SKU cụ thể được khách hàng thêm vào giỏ. | `CartItemID`, `CartID`, `VariantID`, `Quantity`, `AddedAt`, `UpdatedAt` | **PK:** `CartItemID`<br>**UK:** `(CartID, VariantID)` |

### 6.2. Các quy ước và Bất biến Nghiệp vụ Shopping đã chốt
1. **Một giỏ hàng Active duy nhất (SHOP-Q01, BR-SH01, BR-SH02):**
   * Mỗi khách hàng có thể có nhiều giỏ hàng trong lịch sử, nhưng tại một thời điểm chỉ có tối đa một giỏ hàng ở trạng thái `ACTIVE`.
   * CSDL cài đặt: `UNIQUE FILTERED INDEX (CustomerID) WHERE CartStatus = 'ACTIVE'`.
2. **Cộng dồn số lượng cho cùng SKU (SHOP-Q02, BR-SH05, BR-SH06):**
   * Trong cùng một giỏ hàng, một `VariantID` chỉ xuất hiện tối đa một dòng (`UQ_CartItem_Cart_Variant`).
   * Khi khách hàng thêm cùng một SKU nhiều lần, hệ thống tự động cộng dồn `Quantity = Quantity + @AddedQty`.
   * Số lượng trong dòng giỏ hàng bắt buộc lớn hơn không (`Quantity > 0`, `BR-SH07`).
3. **Cảnh báo và xử lý SKU không khả dụng (SHOP-Q03, BR-SH08):**
   * Khi SKU bị hết hàng hoặc ngừng kinh doanh, dòng `CartItem` vẫn được giữ lại để lưu vết và hệ thống hiển thị cảnh báo cho khách.
   * Chặn không cho phép tiến hành Checkout nếu giỏ hàng còn chứa SKU không khả dụng.
4. **Không khóa giá và không giữ hàng (SHOP-Q04, BR-SH09, BR-SH10, BR-SH14):**
   * Không lưu giá bán chính thức trong `CartItem`. Khi hiển thị giỏ hàng, hệ thống lấy giá hiện hành từ `ProductVariant.SellingPrice`.
   * Giá bán chỉ được kiểm tra lại và snapshot bất biến khi tạo `OrderLine`.
   * Thao tác thêm hàng vào giỏ **tuyệt đối không làm thay đổi `OnHandQty` và không giữ tồn kho `ReservedQty`**.
5. **Đồng bộ đa thiết bị và bảo toàn lịch sử chuyển đổi (SHOP-Q05, SHOP-Q06, BR-SH11..13, BR-SH15):**
   * Giỏ hàng được lưu tập trung trên Database Server theo `CustomerID`, tự động đồng bộ giữa các thiết bị (Web, Mobile App).
   * Khi chuyển đổi thành đơn hàng thành công, giỏ hàng chuyển trạng thái sang `CONVERTED` và được đóng băng bất biến (`BR-SH15`).
   * Khi khách hàng tiếp tục mua sắm sau khi giỏ hàng đã chuyển đổi, hệ thống tự động sinh một `ShoppingCart` mới (`BR-SH13`).

---

## 7. Subject Area 06: Sales (Baseline đã khóa)

### 7.1. Danh mục 4 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E40** | `SalesOrder` | Strong / Document Entity | Hợp đồng mua bán chính thức giữa khách hàng và RetailCore (Header). | `OrderID`, `OrderNumber`, `CustomerID`, `CartID`, `OrderStatus`, `OrderDate`, `SubtotalAmount`, `DiscountAmount`, `ShippingFee`, `TotalAmount`, `CustomerNote`, `CreatedAt`, `UpdatedAt` | **PK:** `OrderID`<br>**UK:** `OrderNumber`, `CartID` |
| **E41** | `OrderLine` | Weak / Detail Entity | Chi tiết mặt hàng SKU và số lượng cam kết bán trong đơn (Line). | `OrderLineID`, `OrderID`, `VariantID`, `SKU`, `ProductName`, `VariantName`, `VariantAttributesSnapshot`, `UnitPrice`, `OrderedQty`, `LineSubtotal` | **PK:** `OrderLineID` |
| **E42** | `OrderAddress` | Weak / Snapshot Entity | Bản sao bất biến địa chỉ giao hàng tại thời điểm tạo đơn. | `OrderAddressID`, `OrderID`, `RecipientName`, `RecipientPhone`, `Province`, `District`, `Ward`, `StreetAddress` | **PK:** `OrderAddressID`<br>**UK:** `OrderID` |
| **E43** | `OrderStatusHistory` | Weak / Event Entity | Nhật ký lưu vết mọi lần chuyển đổi trạng thái của đơn hàng. | `StatusHistoryID`, `OrderID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedBy`, `ChangeReason` | **PK:** `StatusHistoryID` |

### 7.2. Các quy ước và Bất biến Nghiệp vụ Sales đã chốt
1. **Tạo đơn và giữ hàng nguyên tử (SALES-Q01, BR-SA07, BR-SA17):**
   * Quá trình tạo đơn và giữ hàng là một giao dịch nguyên tử (All-or-nothing): Chỉ khi hệ thống phân bổ và khóa giữ đủ 100% số lượng hàng cần thiết (`StockReservation`) thì đơn hàng mới được tạo chính thức và giỏ hàng mới chuyển sang `CONVERTED`.
   * Nếu thiếu hàng, toàn bộ giao dịch bị rollback và giỏ hàng giữ nguyên `ACTIVE`.
2. **Máy trạng thái đơn hàng (SALES-Q02, BR-SA08, BR-SA09):**
   * Vòng đời chuẩn: `PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `COMPLETED` (nhánh kết thúc bất thường: `CANCELLED`).
   * Trạng thái ban đầu luôn là `PENDING_PAYMENT`. Đơn chỉ chuyển sang `CONFIRMED` khi điều kiện thanh toán được thỏa mãn (COD hợp lệ hoặc cổng thanh toán xác nhận `PAID`).
3. **Bảo toàn bất biến thông tin dòng hàng (SALES-Q04, SALES-Q05, BR-SA10, BR-SA11):**
   * Sau khi đơn hàng được xác nhận, tuyệt đối cấm chỉnh sửa dòng hàng `OrderLine` (không sửa SKU, không đổi số lượng). Muốn thay đổi, khách hàng phải hủy đơn và đặt lại.
   * `OrderLine` lưu snapshot toàn bộ: `SKU`, `ProductName`, `VariantName`, `UnitPrice`, và `VariantAttributesSnapshot` (JSON) để độc lập hoàn toàn với các biến động của Catalog trong tương lai.
4. **Snapshot địa chỉ giao hàng duy nhất (SALES-Q06, BR-SA12, BR-SA13):**
   * Mỗi đơn hàng có đúng một địa chỉ giao hàng snapshot (`OrderAddress`, quan hệ $1:1$).
   * Tách lô giao hàng từ nhiều kho (Split Shipment) không làm phát sinh nhiều địa chỉ nhận; mọi kiện hàng đều hướng về địa chỉ của đơn.
   * Thay đổi địa chỉ trong sổ địa chỉ cá nhân (`CustomerAddress`) không làm ảnh hưởng đến đơn hàng cũ.
5. **Lưu trữ kiểm toán bất biến (SALES-Q07, BR-SA14, BR-SA15):**
   * Tuyệt đối không xóa vật lý (`No Hard Delete`) đơn hàng và dòng hàng đã phát sinh. Đơn hủy vẫn giữ nguyên trạng thái `CANCELLED`.
   * Mọi lần chuyển trạng thái đều sinh một bản ghi mới trong `OrderStatusHistory` kèm thời điểm `ChangedAt`, tác nhân `ChangedBy`, và lý do `ChangeReason`.
