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

---

## 8. Subject Area 07 – Fulfillment & Shipping (FINAL v1.0)

Subject Area này quản lý toàn bộ vòng đời thực hiện đơn hàng (Fulfillment) và giao nhận hàng hóa (Shipping), từ khi đơn hàng được xác nhận (`CONFIRMED`), qua khâu chuẩn bị xuất kho, bàn giao đơn vị vận chuyển, theo dõi hành trình giao nhận, đến khi giao hàng thành công hoặc xử lý hàng giao thất bại quay về kho.

### 8.1. Danh mục 5 Thực thể Chuẩn hóa (E44 đến E48)

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E44** | `Carrier` | Strong Entity | Đơn vị vận chuyển / Đối tác logistics (GHTK, GHN, ViettelPost, J&T...). | `CarrierID`, `CarrierCode`, `CarrierName`, `ContactPhone`, `Email`, `Website`, `CarrierStatus`, `CreatedAt` | **PK:** `CarrierID`<br>**UK:** `CarrierCode` |
| **E45** | `Shipment` | Document / Strong Entity | Lô giao hàng / Kiện hàng xuất kho giao cho khách theo đơn hàng. | `ShipmentID`, `ShipmentNumber`, `OrderID`, `WarehouseID`, `CarrierID`, `TrackingNumber`, `ShipmentStatus`, `ShippingFee`, `CreatedAt`, `HandedOverAt`, `DeliveredAt` | **PK:** `ShipmentID`<br>**UK:** `ShipmentNumber`<br>**UK:** `TrackingNumber` *(Filtered khi có)* |
| **E46** | `ShipmentLine` | Associative / Weak Entity | Chi tiết số lượng SKU thuộc một dòng đơn hàng được đưa vào một lô giao cụ thể. | `ShipmentLineID`, `ShipmentID`, `OrderLineID`, `AllocationID`, `ShipmentQty` | **PK:** `ShipmentLineID`<br>**UK:** `(ShipmentID, AllocationID)` |
| **E47** | `DeliveryAttempt` | Weak Entity | Ghi nhận từng lần nhân viên giao hàng hoặc đơn vị vận chuyển thực hiện giao kiện hàng. | `AttemptID`, `ShipmentID`, `AttemptNumber`, `AttemptedAt`, `AttemptStatus`, `FailureReason`, `RecipientName`, `DeliveryNote` | **PK:** `AttemptID`<br>**UK:** `(ShipmentID, AttemptNumber)` |
| **E48** | `ShipmentStatusHistory` | Weak / Audit Entity | Lịch sử chuyển trạng thái của một lô giao phục vụ truy vết và kiểm toán vận hành. | `ShipmentStatusHistoryID`, `ShipmentID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedBy`, `ChangeReason` | **PK:** `ShipmentStatusHistoryID` |

### 8.2. Các quy ước và Bất biến Nghiệp vụ Shipping đã chốt (SHIP-Q01 đến SHIP-Q07)
1. **Xuất hàng từ đúng một kho vật lý (SHIP-Q01, BR-FS02, BR-FS09):**
   * Mỗi `Shipment` chỉ xuất phát từ duy nhất 1 kho (`WarehouseID NOT NULL`). Tuyệt đối không gom hàng từ nhiều kho vào chung 1 mã vận đơn. Đơn chia kho sẽ tự động tách thành nhiều Shipment (Split Shipment theo BD-03).
2. **Thời điểm tạo lô giao hàng (SHIP-Q02, BR-FS04):**
   * Chỉ tạo `Shipment` sau khi đơn hàng được xác nhận (`CONFIRMED`) và kho bắt đầu chuẩn bị xuất hàng (`PROCESSING`). Không tạo Shipment khi đơn đang `PENDING_PAYMENT` hoặc đã bị hủy.
3. **Lựa chọn đơn vị vận chuyển (SHIP-Q03, BR-FS10, BR-FS11):**
   * Hệ thống tự động đề xuất `Carrier` tối ưu dựa trên cấu hình cước, vùng miền và SLA; nhân viên/điều phối viên có quyền xem xét và xác nhận lựa chọn cuối cùng.
4. **Giới hạn số lần giao và quy trình chuyển hoàn (SHIP-Q04, BR-FS13, BR-FS14):**
   * Một `Shipment` được giao lại tối đa $N$ lần (chuẩn là 3 lần). Mỗi lần ghi nhận 1 bản ghi trong `DeliveryAttempt`. Khi vượt quá số lần giao cho phép, Shipment tự động chuyển sang quy trình chuyển hoàn (`RETURNING`).
5. **Bằng chứng giao hàng (SHIP-Q05, BR-FS15, BR-FS16):**
   * Ghi nhận đầy đủ thông tin người thực tế nhận hàng, thời gian và ghi chú giao hàng. Ảnh chụp bưu kiện/chữ ký điện tử có thể mở rộng lưu trữ sau này.
6. **Cập nhật trạng thái đa kênh (SHIP-Q06, BR-FS17, BR-FS18):**
   * Hỗ trợ cập nhật trạng thái tự động qua API/Webhook đối tác vận chuyển và cập nhật thủ công bởi nhân viên. Mọi biến động đều được lưu vết trong `ShipmentStatusHistory`.
7. **Phân bổ và quản lý phí vận chuyển (SHIP-Q07, BR-FS19, BR-FS20):**
   * `SalesOrder.ShippingFee` lưu tổng cước phí tính cho khách hàng; `Shipment.ShippingFee` lưu phần chi phí thực tế phân bổ cho từng lô giao phục vụ đối soát công nợ đối tác.
8. **Phối hợp kế toán tồn kho với Inventory (BR-FS21, BR-FS22):**
   * Khi hàng bàn giao cho Carrier (`ShipmentStatus = 'IN_TRANSIT'` / `DISPATCHED`): Ghi nhận `InventoryTransaction` loại `SHIPMENT_OUT` (`OnHandDelta = -Qty`, `ReservedDelta = -Qty`).
   * Khi hàng chuyển hoàn về tới kho (`ShipmentStatus = 'RETURNED'`): Thủ kho kiểm đếm thực tế và ghi nhận `InventoryTransaction` loại `DELIVERY_RETURN_IN` (`OnHandDelta = +ReturnedQty`). Chỉ khi kho thực tế nhận lại hàng mới được tăng tồn kho.

---

## 9. Subject Area 08 – Promotion & Voucher (FINAL v1.0)

Subject Area này quản lý toàn bộ các chương trình khuyến mãi, điều kiện áp dụng, chiết khấu sản phẩm theo SKU, mã voucher cấp đơn hàng, giới hạn sử dụng, và lịch sử tiêu thụ ưu đãi thực tế của khách hàng. Tuân thủ triệt để quyết định **BD-05** từ M1: mỗi dòng hàng tối đa 1 khuyến mãi cấp sản phẩm, mỗi đơn hàng tối đa 1 voucher cấp đơn hàng, chỉ kết hợp khi `IsStackable = 1` và đúng quy tắc ưu tiên.

### 9.1. Danh mục 6 Thực thể Chuẩn hóa (E49 đến E54)

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E49** | `Promotion` | Strong / Document Entity | Chương trình khuyến mãi tổng thể có quản lý phiên bản (Header). | `PromotionID`, `PromotionCode`, `VersionNumber`, `SupersedesPromotionID`, `PromotionName`, `PromotionType`, `PromotionScope`, `DiscountType`, `DiscountValue`, `MaxDiscountAmount`, `MinOrderAmount`, `StartAt`, `EndAt`, `PromotionStatus`, `Priority`, `IsStackable`, `MaxUsage`, `MaxUsagePerCustomer`, `UsedCount`, `CreatedAt` | **PK:** `PromotionID`<br>**UK:** `(PromotionCode, VersionNumber)` |
| **E50** | `PromotionCondition` | Weak Entity | Các điều kiện cấu hình sẵn mà đơn hàng hoặc dòng hàng phải thỏa mãn. | `ConditionID`, `PromotionID`, `ConditionType`, `ComparisonOperator`, `ConditionValue` | **PK:** `ConditionID` |
| **E51** | `PromotionProduct` | Associative Entity | Danh sách các SKU (`ProductVariant`) được áp dụng ưu đãi cấp sản phẩm. | `PromotionID`, `VariantID`, `AssignedAt` | **PK:** `(PromotionID, VariantID)` |
| **E52** | `Voucher` | Strong / Document Entity | Mã ưu đãi dùng khi thanh toán (mã công khai hoặc mã cấp riêng cho khách). | `VoucherID`, `PromotionID`, `VoucherCode`, `VoucherType`, `AssignedCustomerID`, `VoucherStatus`, `MaxUsage`, `UsedCount`, `StartAt`, `EndAt` | **PK:** `VoucherID`<br>**UK:** `VoucherCode` |
| **E53** | `PromotionUsage` | Associative / Ledger Entity | Lịch sử tiêu thụ và ghi nhận mức giảm giá thực tế trên đơn/dòng hàng. | `PromotionUsageID`, `PromotionID`, `OrderID`, `OrderLineID`, `VoucherID`, `DiscountAmount`, `UsageStatus`, `UsedAt`, `ReversedAt`, `ReversalReason` | **PK:** `PromotionUsageID` |
| **E54** | `PromotionTargetCustomer` | Associative Entity | Danh sách khách hàng được chỉ định hưởng chương trình ưu đãi riêng tư/VIP. | `PromotionID`, `CustomerID`, `AssignedAt` | **PK:** `(PromotionID, CustomerID)` |

### 9.2. Các quy ước và Bất biến Nghiệp vụ Promotion đã chốt (PROMO-Q01 đến PROMO-Q08)
1. **Phạm vi ưu đãi cấp sản phẩm theo SKU (PROMO-Q01, BR-PRM03, BR-PRM04):**
   * Khuyến mãi cấp sản phẩm chỉ áp dụng ở mức SKU (`ProductVariant`) qua bảng liên kết `PromotionProduct`. Điều này giúp kiểm soát chính xác chiết khấu từng biến thể, không nhập nhằng giá bán.
2. **Chuẩn hóa điều kiện cấu hình sẵn (PROMO-Q02, BR-PRM05, BR-PRM06):**
   * Tuyệt đối không cho phép quản trị viên nhập biểu thức tùy ý hoặc mã thực thi. Điều kiện được chuẩn hóa theo `ConditionType` định sẵn (`MIN_ORDER_AMOUNT`, `MIN_ITEM_QTY`, `CUSTOMER_TIER`, v.v.) để bảo đảm tốc độ truy vấn index và an toàn hệ thống.
3. **Phân loại Voucher dùng chung và Voucher cá nhân (PROMO-Q03, BR-PRM09, BR-PRM10):**
   * Voucher dùng chung: `AssignedCustomerID IS NULL` (mã công khai như `HELLOSUMMER`, `FREESHIP`).
   * Voucher riêng: `AssignedCustomerID` trỏ đến khách hàng cụ thể (mã tri ân sinh nhật, đền bù dịch vụ).
4. **Kiểm soát hạn mức sử dụng hai tầng (PROMO-Q04, BR-PRM12, BR-PRM13):**
   * Hệ thống giới hạn tổng lượt dùng toàn sàn (`MaxUsage`) và số lượt dùng tối đa cho mỗi khách hàng (`MaxUsagePerCustomer`). Ngăn chặn gian lận và tài khoản ảo vét mã.
5. **Vòng đời tiêu thụ ưu đãi (PROMO-Q05, BR-PRM14):**
   * Khi tạo đơn (`PENDING_PAYMENT`): Tạo bản ghi `PromotionUsage` ở trạng thái `PENDING` (giữ chỗ tạm thời để tránh bán vượt mã trong thời hạn thanh toán).
   * Khi thanh toán thành công (`CONFIRMED`): Chuyển trạng thái sang `CONSUMED` và chính thức tăng `UsedCount`.
   * Khi đơn bị hủy hoặc hết hạn thanh toán: Chuyển sang `REVERSED` để hoàn trả quota voucher cho khách hàng (PROMO-Q06, BR-PRM19).
6. **Bất biến và Quản lý phiên bản chương trình (PROMO-Q08, BR-PRM21, BR-PRM22):**
   * Tuyệt đối không chỉnh sửa các tham số ảnh hưởng đến kết quả giảm giá (giá trị giảm, SKU, điều kiện) của Promotion đã phát sinh `PromotionUsage`.
   * Khi cần thay đổi chính sách, quản trị viên tạo phiên bản mới (`VersionNumber += 1`, `SupersedesPromotionID = OldPromotionID`). Đơn hàng cũ luôn giữ nguyên liên kết đến đúng phiên bản Promotion tại thời điểm mua.
7. **Bảo toàn snapshot giá trị giảm (BR-PRM18):**
   * `PromotionUsage.DiscountAmount` lưu chính xác số tiền giảm thực tế tại thời điểm checkout, độc lập hoàn toàn với việc chương trình có thay đổi hoặc hết hạn sau này.
   * `SalesOrder.DiscountAmount` phải bằng tổng `DiscountAmount` của các dòng `PromotionUsage` thuộc đơn.

---

## 10. Subject Area 09 – Payment & Refund (FINAL v1.0)

Subject Area này quản lý toàn bộ nghĩa vụ tài chính, dòng tiền thu vào (Payment) và hoàn trả (Refund) của đơn hàng trong RetailCore, chịu trách nhiệm bảo toàn tính toán kế toán, quản lý đối soát và kiểm toán dòng tiền với cổng thanh toán và đối tác vận chuyển.

### 10.1. Danh mục 5 Thực thể Chuẩn hóa (E55 đến E59)

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E55** | `PaymentMethod` | Strong Entity | Danh mục các phương thức thanh toán được hệ thống hỗ trợ (COD, BANK_TRANSFER, E_WALLET, CARD...). | `PaymentMethodID`, `MethodCode`, `MethodName`, `MethodType`, `MethodStatus`, `CreatedAt` | **PK:** `PaymentMethodID`<br>**UK:** `MethodCode` |
| **E56** | `Payment` | Document / Strong Entity | Hồ sơ nghĩa vụ và trạng thái thanh toán tổng thể của đơn hàng (Header). | `PaymentID`, `OrderID`, `PaymentMethodID`, `PaymentStatus`, `AmountDue`, `AmountPaid`, `AmountRefunded`, `CreatedAt`, `PaidAt` | **PK:** `PaymentID`<br>**UK:** `OrderID` |
| **E57** | `PaymentTransaction` | Weak / Event Entity | Nhật ký giao dịch ghi nhận từng lần thực hiện thanh toán hoặc gọi cổng thanh toán bên ngoài (kể cả lần thử thất bại). | `PaymentTransactionID`, `PaymentID`, `TransactionReference`, `GatewayTransactionID`, `TransactionType`, `TransactionStatus`, `Amount`, `TransactionFee`, `InitiatedAt`, `CompletedAt`, `GatewayResponseCode`, `GatewayResponseMessage`, `GatewayResponseData` | **PK:** `PaymentTransactionID`<br>**UK:** `TransactionReference` |
| **E58** | `Refund` | Document / Strong Entity | Yêu cầu hoàn tiền gắn với hồ sơ thanh toán đã phát sinh thực thu (Header kiểm soát nội bộ). | `RefundID`, `PaymentID`, `RefundReference`, `RefundAmount`, `RefundReason`, `RefundStatus`, `RequestedAt`, `RequestedBy`, `ApprovedBy`, `ApprovedAt` | **PK:** `RefundID`<br>**UK:** `RefundReference` |
| **E59** | `RefundTransaction` | Weak / Event Entity | Giao dịch thực thi chuyển tiền hoàn qua cổng thanh toán hoặc ủy nhiệm chi ngân hàng. | `RefundTransactionID`, `RefundID`, `TransactionReference`, `GatewayTransactionID`, `RefundMethod`, `TransactionStatus`, `Amount`, `TransactionFee`, `InitiatedAt`, `CompletedAt`, `GatewayResponseCode`, `GatewayResponseMessage` | **PK:** `RefundTransactionID`<br>**UK:** `TransactionReference` |

### 10.2. Các quy ước và Bất biến Nghiệp vụ Payment đã chốt (PAY-Q01 đến PAY-Q09)
1. **Một phương thức thanh toán cho mỗi đơn hàng (PAY-Q01, BR-PY01, BR-PY02):**
   * Mỗi `SalesOrder` có quan hệ $1:1$ bắt buộc với đúng một hồ sơ `Payment` (`Payment.OrderID` UNIQUE). Khách hàng chọn 1 phương thức thanh toán khi checkout và không đổi phương thức sau khi giao dịch đã phát sinh.
2. **Hỗ trợ song song COD và Thanh toán trực tuyến (PAY-Q02, BR-PY03):**
   * Đơn online: Chuyển `CONFIRMED` khi có giao dịch `PaymentTransaction` thành công và `AmountPaid == AmountDue` (PAY-Q08, BR-PY09).
   * Đơn COD: Chuyển `CONFIRMED` ngay sau khi tạo đơn và thẩm định hợp lệ; `PaymentTransaction` loại `COD_COLLECTION` chỉ được ghi nhận thành công khi Shipper giao hàng thành công và bàn giao tiền mặt (PAY-Q08, BR-PY10).
3. **Thanh toán lại không làm vỡ đơn hàng (PAY-Q03, BR-PY04, BR-PY05):**
   * Khi khách hàng gặp lỗi thẻ/quên OTP, đơn giữ nguyên trạng thái `PENDING_PAYMENT` trong thời gian timeout. Mỗi lần bấm thanh toán lại tạo một bản ghi `PaymentTransaction` mới độc lập. Giao dịch thất bại được giữ nguyên phục vụ đối soát.
4. **Hoàn tiền một phần và nhiều lần (PAY-Q04, BR-PY15, BR-PY16, BR-PY18):**
   * Hỗ trợ hoàn tiền toàn phần hoặc một phần qua nhiều đợt. Bất biến tài chính: $\sum RefundAmount \le RemainingRefundable = AmountPaid - AmountRefunded$.
   * `Payment.AmountRefunded` chỉ được tăng khi `RefundTransaction` thực tế thành công (`SUCCESS`).
5. **Kiểm soát nội bộ và phê duyệt hoàn tiền (PAY-Q05, BR-PY14):**
   * Mọi yêu cầu `Refund` bắt buộc phải được nhân viên có thẩm quyền phê duyệt (`ApprovedBy NOT NULL`) trước khi thực hiện giao dịch chuyển tiền.
6. **Phương thức hoàn tiền COD (PAY-Q06, BR-PY20):**
   * Đơn COD được hoàn tiền qua chuyển khoản ngân hàng (Bank Transfer) và lưu vết thông tin đối soát tại `RefundTransaction`.
7. **Lưu vết đối soát và phí giao dịch cổng (PAY-Q07, PAY-Q09, BR-PY06, BR-PY07):**
   * Lưu đầy đủ `GatewayTransactionID`, `GatewayResponseCode`, `GatewayResponseMessage`, `GatewayResponseData` và `TransactionFee` để tính toán chính xác số tiền thực nhận (Net Settlement) và biên lợi nhuận ròng.
8. **Bất biến nhật ký giao dịch và Idempotency (BR-PY05, BR-PY22, BR-PY09):**
   * `PaymentTransaction` và `RefundTransaction` là nhật ký bất biến (Append-only). Tuyệt đối cấm xóa vật lý hoặc ghi đè giao dịch cũ. Xử lý triệt để webhook trùng lặp (Idempotency) để không tăng tiền trùng.

---

## 11. Subject Area 10 – Return & Exchange (FINAL v1.0)

Subject Area này quản lý toàn bộ quy trình hậu mãi sau bán hàng khi khách hàng có nhu cầu đổi trả hàng hóa sau khi đã nhận hàng thành công (`DELIVERED`/`COMPLETED`), bao gồm: tiếp nhận yêu cầu RMA, phê duyệt trước khi gửi hàng, tiếp nhận bưu kiện tại kho, kiểm định phân loại chất lượng hàng hóa và kích hoạt hoàn tiền sang phân hệ Payment & Refund.

### 11.1. Danh mục 5 Thực thể Chuẩn hóa (E60 đến E64)

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E60** | `ReturnRequest` | Document / Strong Entity | Yêu cầu đổi trả hàng chính thức từ khách hàng hoặc CSKH tạo thay (RMA Header). | `ReturnRequestID`, `ReturnNumber`, `OrderID`, `CustomerID`, `WarehouseID`, `CreatedByAccountID`, `ReturnStatus`, `ReturnReason`, `CustomerNote`, `RequestedAt`, `ReviewedBy`, `ReviewedAt`, `ReviewNote` | **PK:** `ReturnRequestID`<br>**UK:** `ReturnNumber` |
| **E61** | `ReturnRequestLine` | Weak / Detail Entity | Chi tiết từng SKU và số lượng yêu cầu trả thuộc một dòng đơn hàng gốc (RMA Line). | `ReturnRequestLineID`, `ReturnRequestID`, `OrderLineID`, `RequestedQty`, `ApprovedQty`, `ReturnReason`, `LineStatus` | **PK:** `ReturnRequestLineID`<br>**UK:** `(ReturnRequestID, OrderLineID)` |
| **E62** | `ReturnReceipt` | Document / Strong Entity | Phiếu tiếp nhận hàng hoàn trả thực tế tại kho vật lý được chỉ định (Receipt Header). | `ReturnReceiptID`, `ReceiptNumber`, `ReturnRequestID`, `WarehouseID`, `ReceivedAt`, `ReceivedBy`, `ReceiptStatus`, `InspectionNote`, `PostedAt` | **PK:** `ReturnReceiptID`<br>**UK:** `ReceiptNumber` |
| **E63** | `ReturnReceiptLine` | Weak / Detail Entity | Chi tiết số lượng thực nhận, kết quả kiểm tra và phân loại xử lý tồn kho của từng dòng hàng. | `ReturnReceiptLineID`, `ReturnReceiptID`, `ReturnRequestLineID`, `ReceivedQty`, `Disposition`, `InspectionResult`, `InspectionNote` | **PK:** `ReturnReceiptLineID` |
| **E64** | `ReturnStatusHistory` | Weak / Audit Entity | Nhật ký lưu vết mọi lần chuyển đổi trạng thái của yêu cầu trả hàng phục vụ kiểm toán và theo dõi SLA. | `ReturnStatusHistoryID`, `ReturnRequestID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedByAccountID`, `ChangeReason` | **PK:** `ReturnStatusHistoryID` |

### 11.2. Các quy ước và Bất biến Nghiệp vụ Return đã chốt (RET-Q01 đến RET-Q08)
1. **Đối tượng tạo yêu cầu trả hàng (RET-Q01, BR-RT01, BR-RT02, BR-RT03):**
   * Cho phép khách hàng tự tạo yêu cầu trên Web/App hoặc nhân viên CSKH tạo yêu cầu thay khách hàng trên tổng đài (`CreatedByAccountID`). Yêu cầu luôn gắn chặt với đúng `CustomerID` và `OrderID` sở hữu.
2. **Thời hạn yêu cầu trả hàng hợp lệ (RET-Q02, BR-RT04):**
   * Chỉ cho phép tạo yêu cầu đổi trả trong thời hạn chính sách được cấu hình (ví dụ 7 hoặc 14 ngày) tính từ thời điểm giao hàng thành công (`SalesOrder.DeliveredAt`).
3. **Trả hàng một phần và nhiều lần (RET-Q03, BR-RT07, BR-RT08, BR-RT09):**
   * Cho phép khách hàng yêu cầu trả một phần số lượng của dòng hàng. Tổng số lượng được duyệt qua tất cả các lần RMA không được vượt số lượng còn đủ điều kiện: $\sum ApprovedQty \le OrderedQty - AlreadyReturnedQty$.
4. **Quy trình phê duyệt hai bước bắt buộc (RET-Q04, BR-RT10):**
   * Nhân viên CSKH/Kiểm duyệt phải xem xét lý do và bấm duyệt (`APPROVED`) trước khi khách gửi hàng về kho. Tuyệt đối không cho phép gửi hàng khi yêu cầu chưa được phê duyệt.
5. **Chỉ định kho tiếp nhận duy nhất (RET-Q06, BR-RT11, BR-RT12):**
   * Mỗi `ReturnRequest` chỉ định đúng một kho tiếp nhận (`WarehouseID NOT NULL`). Mọi phiếu `ReturnReceipt` thuộc yêu cầu này bắt buộc phải xử lý tại đúng kho đã chỉ định.
6. **Phân loại xử lý hàng trả và biến động tồn kho (RET-Q05, BR-RT15, BR-RT16, BR-RT17, BR-RT18, BR-RT19):**
   * CSDL phân loại rõ trường `Disposition`:
     - `SELLABLE`: Hàng nguyên vẹn tem mác $\to$ tạo `InventoryTransaction` loại `CUSTOMER_RETURN_IN`, cộng lại vào `OnHandQty` kho hàng bán.
     - `QUARANTINE`: Hàng cần kiểm tra sâu/chờ hãng giám định $\to$ chuyển khu cách ly, không cộng vào tồn bán được.
     - `DISPOSED`: Hàng hỏng/vỡ nát $\to$ ghi nhận tổn thất, tiêu hủy, không cộng vào tồn kho bán.
7. **Cơ chế kích hoạt hoàn tiền an toàn (RET-Q07, BR-RT20, BR-RT21):**
   * Tuyệt đối không hoàn tiền khi chưa nhận được hàng. Chỉ sau khi thủ kho ký xác nhận phiếu nhận hàng (`PostedAt` NOT NULL) với `Disposition = 'SELLABLE'`, hệ thống mới tính toán số tiền hoàn thực tế (đã trừ voucher phân bổ) và sinh yêu cầu `Refund` sang Subject Area 09.
8. **Nhật ký chuyển trạng thái bất biến (RET-Q08, BR-RT22, BR-RT23):**
   * Mọi lần thay đổi trạng thái RMA từ `PENDING_APPROVAL` $\to$ `APPROVED` $\to$ `AWAITING_RETURN` $\to$ `RECEIVED` $\to$ `INSPECTING` $\to$ `COMPLETED` đều được lưu vết trong `ReturnStatusHistory`. Cấm xóa vật lý chứng từ RMA.

---

## 12. Subject Area 11 – Audit & Reporting (FINAL v1.0)

Subject Area này quản lý việc ghi nhận nhật ký kiểm toán hệ thống (Audit Trail) để phục vụ an ninh thông tin, tuân thủ pháp lý, và quản lý các tác vụ thực thi báo cáo (Reporting Execution) cùng các bản chụp số liệu theo kỳ (Snapshot) nhằm phục vụ quản trị ra quyết định mà không làm ảnh hưởng đến hiệu năng của hệ thống giao dịch OLTP.

### 12.1. Danh mục 6 Thực thể Chuẩn hóa (E65 đến E70)

#### Nhóm A – Audit Trail (Kiểm toán an ninh)
| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E65** | `AuditLog` | Audit / Ledger Entity | Nhật ký lưu vết các thao tác và sự kiện quan trọng, nhạy cảm trong hệ thống (Header). | `AuditLogID`, `AccountID`, `ActionType`, `EntityName`, `RecordID`, `ActionAt`, `IPAddress`, `UserAgent`, `Result`, `Reason`, `CorrelationID`, `RetentionUntil` | **PK:** `AuditLogID` |
| **E66** | `AuditLogDetail` | Weak / Audit Entity | Chi tiết các thuộc tính bị thay đổi (giá trị trước và sau) của một sự kiện audit (Detail). | `AuditLogDetailID`, `AuditLogID`, `FieldName`, `OldValue`, `NewValue` | **PK:** `AuditLogDetailID` |

#### Nhóm B – Reporting (Báo cáo quản trị & Snapshot)
| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E67** | `ReportDefinition` | Catalog Entity | Danh mục các mẫu báo cáo quản trị và nghiệp vụ được hệ thống hỗ trợ. | `ReportDefinitionID`, `ReportCode`, `ReportName`, `Description`, `DataSource`, `ReportType`, `IsActive`, `CreatedAt` | **PK:** `ReportDefinitionID`<br>**UK:** `ReportCode` |
| **E68** | `ReportExecution` | Event / Audit Entity | Nhật ký lịch sử các lần người dùng yêu cầu chạy hoặc xuất báo cáo ra file. | `ReportExecutionID`, `ReportDefinitionID`, `AccountID`, `Parameters`, `ExecutionStatus`, `RequestedAt`, `CompletedAt`, `ExportFormat`, `ResultLocation`, `ErrorMessage` | **PK:** `ReportExecutionID` |
| **E69** | `ReportSnapshot` | Document / Snapshot Entity | Ảnh chụp dữ liệu báo cáo được chốt số liệu cố định tại một thời điểm hoặc kỳ kế toán xác định. | `ReportSnapshotID`, `ReportDefinitionID`, `SnapshotCode`, `PeriodStart`, `PeriodEnd`, `SnapshotAt`, `SnapshotStatus`, `DataLocation`, `CreatedBy`, `CreatedAt` | **PK:** `ReportSnapshotID`<br>**UK:** `SnapshotCode` |
| **E70** | `ReportPermission` | Associative Entity | Bảng phân quyền chi tiết cho phép một vai trò (`Role`) được xem, xuất file hoặc tạo snapshot báo cáo. | `ReportDefinitionID`, `RoleID`, `CanView`, `CanExport`, `CanCreateSnapshot` | **PK:** `(ReportDefinitionID, RoleID)` |

### 12.2. Các quy ước và Bất biến Nghiệp vụ Audit & Reporting đã chốt (AR-Q01 đến AR-Q07)
1. **Phạm vi ghi log chọn lọc (AR-Q01, BR-AR01, BR-AR02):**
   * Chỉ ghi nhận AuditLog cho các thao tác quan trọng và nhạy cảm: duyệt đơn, hủy đơn, kiểm kê kho, điều chỉnh tồn, duyệt hoàn tiền, đổi giá bán SKU, cấp quyền Role/Permission. Không ghi log toàn bộ CRUD đọc/ghi thông thường để bảo vệ hiệu năng I/O của OLTP.
2. **Chi tiết thay đổi và Bảo vệ dữ liệu nhạy cảm (AR-Q02, BR-AR05, BR-AR06):**
   * Lưu `OldValue` và `NewValue` qua `AuditLogDetail` để so sánh trực quan. Tuyệt đối cấm ghi mật khẩu, chuỗi băm mật khẩu, mã OTP, token hoặc thông tin thẻ ngân hàng vào nội dung log.
3. **Tính bất biến của AuditLog (AR-Q03, BR-AR07):**
   * Bảng `AuditLog` và `AuditLogDetail` là nhật ký chỉ ghi (`Append-Only / WORM`), tuyệt đối cấm chỉnh sửa (`UPDATE`) và cấm xóa (`DELETE`) qua bất kỳ chức năng nghiệp vụ nào.
4. **Chính sách lưu trữ và Archive (AR-Q04, BR-AR08, BR-AR09):**
   * Nhật ký có thời hạn lưu trữ hoạt động được cấu hình qua `RetentionUntil` (thường 12–24 tháng). Sau thời hạn, dữ liệu được chuyển vào kho lưu trữ lạnh (Cold Storage) theo chính sách lưu trữ của doanh nghiệp.
5. **Cơ chế báo cáo kết hợp (AR-Q05, BR-AR14, BR-AR15, BR-AR16, BR-AR17):**
   * Báo cáo vận hành hàng ngày truy vấn trực tiếp từ CSDL nghiệp vụ qua Views/Index tối ưu.
   * Báo cáo tài chính/kế toán cần chốt số liệu kỳ sử dụng `ReportSnapshot` để bảo đảm số liệu không bị biến động hồi tố khi có phát sinh sau này.
6. **Kiểm toán hành vi xuất dữ liệu (AR-Q06, BR-AR11, BR-AR12):**
   * Mọi lần thực thi hoặc trích xuất báo cáo ra file đều sinh một bản ghi `ReportExecution` kèm tài khoản yêu cầu, tham số lọc và vị trí lưu file kết quả `ResultLocation`.
7. **Phân quyền báo cáo ma trận (AR-Q07, BR-AR13, BR-AR18, BR-AR19):**
   * Phân quyền báo cáo được quản lý độc lập qua thực thể kết hợp `ReportPermission`, phân tách rõ 3 quyền hạn: `CanView` (xem trên web), `CanExport` (xuất file Excel/CSV), và `CanCreateSnapshot` (chốt kỳ kế toán). Khóa chính là `(ReportDefinitionID, RoleID)`.
