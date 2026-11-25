# M2.01 - ENTITY CATALOG (DANH MỤC 79 THỰC THỂ KHÁI NIỆM CHUẨN HÓA)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 2.1 – Canonical SSOT Baseline (**79 Thực thể Chuẩn hóa `E01` đến `E79`**, ánh xạ `1:1` sang 79 Bảng Quan hệ M3)
* **Kế thừa & Đồng bộ:** [GEMINI.md](../../../GEMINI.md), Business Rules v2.0 (`M1.06`), Data Requirements v2.0 (`M1.09`), Quyết định `BD-01` đến `BD-07` & Bản ghi Hiệu chỉnh Kiến trúc [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md) (`CORR-01` đến `CORR-10`).

---

## 1. Phân loại Loại hình Thực thể trong Mô hình Quan niệm (79 Thực thể)

1. **Thực thể Mạnh (Strong / Master / Document Entity):** Tồn tại độc lập, có khóa định danh tự nhiên trong nghiệp vụ (`Brand`, `Category`, `Product`, `Attribute`, `Warehouse`, `Supplier`, `Account`, `Role`, `Permission`, `Carrier`, `PaymentMethod`, `ReportDefinition`, cùng các chứng từ Header `PurchaseOrder`, `GoodsReceipt`, `SupplierReturn`, `ShoppingCart`, `SalesOrder`, `Shipment`, `DeliveryReturnReceipt`, `StockReservation`, `InventoryTransaction`, `StockCount`, `WarehouseTransfer`, `Promotion`, `Voucher`, `Payment`, `Refund`, `CODRemittance`, `ReturnRequest`, `ReturnReceipt`, `AuditLog`, `ReportSnapshot`).
2. **Thực thể Biến thể Trung tâm (Core SKU Entity):** `ProductVariant` (E04) – Có surrogate key `BIGINT` riêng, phụ thuộc toàn phần vào `Product` và mang mã tự nhiên `SKU` duy nhất toàn hệ thống.
3. **Thực thể Kế thừa Chuyên biệt hóa (TPT Subtype Entity):** `Customer` (E30) và `Employee` (E31) – Kế thừa loại trừ XOR từ `Account` (E29), sử dụng chính `AccountID` vừa làm Khóa chính vừa làm Khóa ngoại (`CORR-04`).
4. **Thực thể Phụ thuộc / Yếu / Sự kiện (Dependent / Weak / Event Entity):** Phụ thuộc vào thực thể cha (`AttributeValue`, `StockReservationLine`, `InventoryTransactionLine`, `StockCountLine`, `WarehouseTransferLine`, `PurchaseOrderLine`, `GoodsReceiptLine`, `SupplierReturnLine`, `CustomerAddress`, `AccountVerification`, `CartItem`, `OrderLine`, `OrderAddress`, `OrderStatusHistory`, `ShipmentLine`, `DeliveryAttempt`, `ShipmentStatusHistory`, `PromotionCondition`, `PaymentTransaction`, `RefundTransaction`, `PaymentWebhookLog`, `IdempotencyRecord`, `CODCollection`, `CODReconciliation`, `ReturnRequestLine`, `ReturnReceiptLine`, `ReturnStatusHistory`, `ReturnLineAssessment`, `AuditLogDetail`, `ReportExecution`).
5. **Thực thể Liên kết (Associative Entity - $N:N$):** Giải quyết quan hệ nhiều-nhiều kèm thuộc tính nghiệp vụ (`ProductCategory`, `ProductAttribute`, `ProductAttributeValue`, `VariantAttributeValue`, `InventoryBalance`, `OrderWarehouseAllocation`, `SupplierProduct`, `AccountRole`, `RolePermission`, `PromotionProduct`, `PromotionUsage`, `PromotionTargetCustomer`, `ReturnShipmentAllocation`, `ReportPermission`).

---

## 2. Subject Area 01: Product Catalog (10 Thực thể: `E01` – `E09`, `E73`)

### 2.1. Danh mục 10 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E01** | `Product` | Strong Entity | Sản phẩm ở cấp độ thương mại chung, gắn thương hiệu và mức thuế suất GTGT. | `ProductID`, `ProductCode`, `BrandID`, `ProductName`, `Description`, `VatRate`, `ProductStatus`, `IsActive`, `CreatedAt`, `UpdatedAt` | **PK:** `ProductID`<br>**UK:** `ProductCode` |
| **E02** | `Category` | Strong Entity (Đệ quy) | Danh mục phân loại sản phẩm theo cấu trúc cây phân cấp đa tầng. | `CategoryID`, `CategoryName`, `Slug`, `Description`, `ParentCategoryID`, `CategoryLevel`, `DisplayOrder`, `IsActive`, `CreatedAt`, `UpdatedAt` | **PK:** `CategoryID`<br>**UK:** `Slug` |
| **E03** | `ProductCategory` | Associative Entity | Liên kết sản phẩm thuộc danh mục ($N:N$) kèm cờ danh mục chính (`BD-01`). | `ProductID`, `CategoryID`, `IsPrimary`, `AssignedAt` | **PK:** `(ProductID, CategoryID)`<br>**UK:** `(ProductID) WHERE IsPrimary = 1` |
| **E04** | `ProductVariant` | Core SKU Entity | Phiên bản bán hàng, định giá nguyên đồng VNĐ đã gồm GTGT, tồn kho và quy cách vận chuyển. | `VariantID`, `ProductID`, `SKU`, `VariantName`, `SellingPrice`, `CompareAtPrice`, `WeightGrams`, `LengthCm`, `WidthCm`, `HeightCm`, `VariantStatus`, `IsActive`, `CreatedAt`, `UpdatedAt` | **PK:** `VariantID`<br>**UK:** `SKU`, `(ProductID, VariantID)` |
| **E05** | `Attribute` | Strong Entity | Từ điển loại thuộc tính dùng chung toàn hệ thống catalog. | `AttributeID`, `AttributeCode`, `AttributeName`, `DataType`, `AttributeDescription`, `IsActive`, `CreatedAt` | **PK:** `AttributeID`<br>**UK:** `AttributeCode` |
| **E06** | `AttributeValue` | Weak Entity | Giá trị chuẩn hóa thuộc một `Attribute` cụ thể. | `AttributeValueID`, `AttributeID`, `ValueCode`, `Value`, `DisplayValue`, `DisplayOrder`, `IsActive`, `CreatedAt` | **PK:** `AttributeValueID`<br>**UK:** `(AttributeID, ValueCode)`, `(AttributeID, Value)`, `(AttributeID, AttributeValueID)` |
| **E07** | `ProductAttribute` | Associative Entity | Khai báo thuộc tính áp dụng cho sản phẩm và phân vai trò (`BD-06`). | `ProductID`, `AttributeID`, `AttributeRole`, `IsRequired`, `DisplayOrder`, `CreatedAt` | **PK:** `(ProductID, AttributeID)` |
| **E08** | `ProductAttributeValue` | Associative Entity | Whitelist giá trị cho `VARIANT_DEFINING` hoặc giá trị mô tả cho `DESCRIPTIVE`. | `ProductID`, `AttributeID`, `AttributeValueID`, `AssignedAt` | **PK:** `(ProductID, AttributeID, AttributeValueID)` |
| **E09** | `VariantAttributeValue` | Associative Entity | Tổ hợp giá trị thuộc tính `VARIANT_DEFINING` thực tế cấu thành một SKU. | `VariantID`, `AttributeID`, `AttributeValueID`, `CreatedAt` | **PK:** `(VariantID, AttributeID)`<br>**UK:** `(VariantID, AttributeValueID)` |
| **E73** | `Brand` | Strong / Master Entity | Danh mục thương hiệu/nhãn hàng chuẩn hóa phục vụ quản trị và Faceted Filter. | `BrandID`, `BrandCode`, `BrandName`, `Slug`, `CountryOfOrigin`, `LogoURL`, `Description`, `IsActive`, `CreatedAt`, `UpdatedAt` | **PK:** `BrandID`<br>**UK:** `BrandCode`, `Slug` |

### 2.2. Các quy ước và Bất biến Nghiệp vụ Product Catalog
1. **Chuẩn hóa Thương hiệu (`Brand` E73) & Thuế/Vận chuyển (`BR-PR11`, `BR-PR15`, `CORR-09`):** Thương hiệu tách thành danh mục `Brand` độc lập (`BrandCode`, `Slug` duy nhất). `Product.VatRate` lưu thuế suất GTGT hợp lệ (`0%`, `5%`, `8%`, `10%`); `ProductVariant.SellingPrice` là giá bán lẻ đã bao gồm GTGT nguyên đồng VNĐ (`SellingPrice = FLOOR(SellingPrice)`), đi kèm `WeightGrams > 0` và kích thước `LengthCm`/`WidthCm`/`HeightCm` để tính cước vận chuyển Zero-Trust.
2. **Quy ước hai lớp thuộc tính & Chống nhiễm chéo (`BD-06`, `BR-PR13`, `BR-PR14`):** `ProductAttribute.AttributeRole` nhận `'VARIANT_DEFINING'` (sinh SKU qua `VariantAttributeValue`, có `ProductAttributeValue` làm Whitelist) hoặc `'DESCRIPTIVE'` (lưu giá trị đặc tả cấp sản phẩm tại `ProductAttributeValue`). Siêu khóa `(AttributeID, AttributeValueID)` trên `AttributeValue` và Composite FK ngăn chặn tuyệt đối việc gán chéo giá trị khác loại thuộc tính.
3. **SKU mặc định & Danh mục chính (`PC-Q01`, `PC-Q03`, `BD-01`):** Sản phẩm đơn chiếc không phân biến thể vẫn có đúng 1 `ProductVariant` mặc định rỗng thuộc tính. Filtered Unique Index `UX_ProductCategory_Primary` cưỡng chế tối đa 1 danh mục chính; khi `ProductStatus = 'PUBLISHED'` bắt buộc có **đúng 1** dòng `IsPrimary = 1` và $\ge 1$ SKU `ACTIVE`. Cấm xóa cứng danh mục/sản phẩm/SKU đã phát sinh giao dịch (`IsActive` đồng bộ trạng thái).

---

## 3. Subject Area 02: Warehouse & Inventory (11 Thực thể: `E10` – `E20`)

### 3.1. Danh mục 11 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E10** | `Warehouse` | Strong Entity | Kho hàng vật lý thuộc mạng lưới phân phối của doanh nghiệp. | `WarehouseID`, `WarehouseCode`, `WarehouseName`, `Address`, `WarehouseStatus`, `CreatedAt` | **PK:** `WarehouseID`<br>**UK:** `WarehouseCode` |
| **E11** | `InventoryBalance` | Associative Entity | Số dư tồn kho tức thời của SKU tại kho theo 3 phẩm chất và giá vốn WAC. | `WarehouseID`, `VariantID`, `OnHandQty`, `ReservedQty`, `QuarantinedQty`, `DamagedQty`, `CurrentAverageCost`, `LastUpdatedAt` | **PK:** `(WarehouseID, VariantID)` |
| **E12** | `OrderWarehouseAllocation` | Associative Entity | Kế hoạch phân bổ số lượng của dòng đơn hàng cho kho vật lý (`BD-03`). | `AllocationID`, `OrderLineID`, `WarehouseID`, `VariantID`, `AllocatedQty`, `AllocationStatus`, `AllocatedAt` | **PK:** `AllocationID`<br>**UK:** `(OrderLineID, WarehouseID)` |
| **E13** | `StockReservation` | Strong / Document Entity | Chứng từ giữ hàng tạm thời gắn với đơn hàng (Header). | `ReservationID`, `OrderID`, `ReservationStatus`, `CreatedAt`, `ExpiresAt`, `ReleasedAt` | **PK:** `ReservationID` |
| **E14** | `StockReservationLine` | Weak Entity | Dòng chi tiết giữ hàng tại kho liên kết với phân bổ ($1:0..N$, `CORR-10`). | `ReservationLineID`, `ReservationID`, `AllocationID`, `VariantID`, `WarehouseID`, `ReservedQty`, `ReservationLineStatus` | **PK:** `ReservationLineID`<br>**UK:** `(AllocationID) WHERE ReservationLineStatus = 'ACTIVE'` |
| **E15** | `InventoryTransaction` | Strong / Document Entity | Chứng từ sổ cái ghi nhận nghiệp vụ thay đổi tồn kho (Header). | `TransactionID`, `TransactionType`, `TransactionStatus`, `ReferenceType`, `ReferenceID`, `CreatedAt`, `PostedAt` | **PK:** `TransactionID` |
| **E16** | `InventoryTransactionLine` | Weak Entity | Dòng sổ cái biến động tồn kho 4-Delta, chuyển phẩm chất và giá vốn. | `TransactionLineID`, `TransactionID`, `VariantID`, `WarehouseID`, `OnHandDelta`, `ReservedDelta`, `QuarantinedDelta`, `DamagedDelta`, `FromStockStatus`, `ToStockStatus`, `UnitCost`, `CreatedAt` | **PK:** `(TransactionLineID, CreatedAt)` *(Partition-Aligned)* |
| **E17** | `StockCount` | Strong / Document Entity | Phiếu kiểm kê tồn kho định kỳ hoặc đột xuất tại một kho (Header). | `StockCountID`, `WarehouseID`, `CountStatus`, `StartedAt`, `CompletedAt`, `CreatedBy`, `ApprovedAt`, `ApprovedBy` | **PK:** `StockCountID` |
| **E18** | `StockCountLine` | Weak Entity | Dòng ghi nhận số lượng hệ thống và số lượng kiểm đếm thực tế. | `StockCountLineID`, `StockCountID`, `VariantID`, `SystemQty`, `CountedQty`, `VarianceQty` | **PK:** `StockCountLineID`<br>**UK:** `(StockCountID, VariantID)` |
| **E19** | `WarehouseTransfer` | Strong / Document Entity | Phiếu điều chuyển hàng hóa 2 bước giữa hai kho vật lý (Header). | `TransferID`, `TransferNumber`, `SourceWarehouseID`, `DestinationWarehouseID`, `TransferStatus`, `CreatedAt`, `DispatchedAt`, `ReceivedAt`, `CreatedBy`, `ApprovedBy` | **PK:** `TransferID`<br>**UK:** `TransferNumber` |
| **E20** | `WarehouseTransferLine` | Weak Entity | Chi tiết SKU, số lượng xuất/nhận và snapshot giá vốn điều chuyển. | `TransferLineID`, `TransferID`, `VariantID`, `TransferQty`, `ReceivedQty`, `VarianceQty`, `UnitCost` | **PK:** `TransferLineID`<br>**UK:** `(TransferID, VariantID)` |

### 3.2. Các quy ước và Bất biến Nghiệp vụ Warehouse & Inventory
1. **Ba phẩm chất vật lý, Sổ cái 4-Delta & 5 CHECK Constraints (`CORR-02`):** Tồn kho vật lý chỉ có 3 phẩm chất: `SELLABLE`, `QUARANTINED`, `DAMAGED` (`SCRAP_OUT` là biến động xuất hủy với `ToStockStatus = NULL`). Bất biến: $\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$ và $\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty} \ge 0$. Mọi biến động vật lý/phẩm chất ghi sổ bất biến vào `InventoryTransactionLine` (4 delta); riêng khóa giữ/nhả giữ hàng chỉ cập nhật `ReservedQty` qua `StockReservationLine`.
2. **Giữ hàng All-or-Nothing, Lịch sử $1:0..N$ & Chống Deadlock (`BD-04`, `CORR-10`, `BR-IN11`):** Khóa `InventoryBalance` theo thứ tự cố định `VariantID ASC, WarehouseID ASC`; thời hạn `ExpiresAt` 15 phút chuẩn / 5 phút cho đơn Flash Sale. `StockReservationLine` liên kết $1:0..N$ với `AllocationID` kèm `UX_StockReservationLine_ActiveAlloc WHERE ReservationLineStatus = 'ACTIVE'`.
3. **Định giá Moving WAC, Điều chuyển 2 bước & Kiểm kê SoD (`WI-Q04..06`, `CORR-08`):** `CurrentAverageCost` tính trên bể `SELLABLE` theo `AcceptedQty` nhập PO, `OutboundUnitCost` hoàn kho và `WarehouseTransferLine.UnitCost` nhận điều chuyển. Phiếu kiểm kê `StockCount` bắt buộc cấp quản lý phê duyệt độc lập (`ApprovedBy <> CreatedBy`) mới sinh giao dịch `STOCK_ADJUSTMENT`.

---

## 4. Subject Area 03: Procurement (8 Thực thể: `E21` – `E28`)

### 4.1. Danh mục 8 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E21** | `Supplier` | Strong / Master Entity | Nhà cung cấp hàng hóa cho doanh nghiệp. | `SupplierID`, `SupplierCode`, `SupplierName`, `TaxCode`, `Email`, `Phone`, `Address`, `SupplierStatus`, `CreatedAt` | **PK:** `SupplierID`<br>**UK:** `SupplierCode` |
| **E22** | `SupplierProduct` | Associative Entity | Danh mục cung ứng giữa nhà cung cấp và SKU (giá mua tham khảo, lead time). | `SupplierID`, `VariantID`, `SupplierSKU`, `PurchasePrice`, `LeadTimeDays`, `IsPreferred` | **PK:** `(SupplierID, VariantID)` |
| **E23** | `PurchaseOrder` | Strong / Document Entity | Đơn đặt hàng mua gửi nhà cung cấp chỉ định 1 kho nhận cố định (Header). | `PurchaseOrderID`, `PONumber`, `SupplierID`, `DestinationWarehouseID`, `OrderDate`, `ExpectedDeliveryDate`, `PurchaseOrderStatus`, `CreatedBy`, `ApprovedBy`, `ApprovedAt` | **PK:** `PurchaseOrderID`<br>**UK:** `PONumber` |
| **E24** | `PurchaseOrderLine` | Weak / Detail Entity | Chi tiết mặt hàng SKU và số lượng đặt mua trên PO. | `PurchaseOrderLineID`, `PurchaseOrderID`, `VariantID`, `OrderedQty`, `UnitPrice`, `LineAmount` | **PK:** `PurchaseOrderLineID`<br>**UK:** `(PurchaseOrderID, VariantID)` |
| **E25** | `GoodsReceipt` | Strong / Document Entity | Phiếu tiếp nhận một đợt giao hàng thực tế từ nhà cung cấp (Header). | `GoodsReceiptID`, `ReceiptNumber`, `PurchaseOrderID`, `WarehouseID`, `ReceivedAt`, `ReceiptStatus`, `ReceivedBy`, `PostedAt` | **PK:** `GoodsReceiptID`<br>**UK:** `ReceiptNumber` |
| **E26** | `GoodsReceiptLine` | Weak / Detail Entity | Chi tiết kiểm nhận QC (`AcceptedQty` + `RejectedQty`) của SKU trong đợt giao. | `GoodsReceiptLineID`, `GoodsReceiptID`, `PurchaseOrderLineID`, `VariantID`, `ReceivedQty`, `AcceptedQty`, `RejectedQty`, `UnitCost`, `RejectionReason` | **PK:** `GoodsReceiptLineID`<br>**UK:** `(GoodsReceiptID, PurchaseOrderLineID)` |
| **E27** | `SupplierReturn` | Strong / Document Entity | Phiếu xuất trả hàng cho nhà cung cấp kèm mã tài khoản chênh lệch giá vốn. | `SupplierReturnID`, `ReturnNumber`, `SupplierID`, `WarehouseID`, `CostVarianceAccountCode`, `ReturnStatus`, `CreatedAt`, `Reason`, `ApprovedBy`, `ApprovedAt` | **PK:** `SupplierReturnID`<br>**UK:** `ReturnNumber` |
| **E28** | `SupplierReturnLine` | Weak / Detail Entity | Chi tiết SKU, số lượng và giá vốn WAC tại thời điểm xuất trả NCC. | `SupplierReturnLineID`, `SupplierReturnID`, `GoodsReceiptLineID`, `VariantID`, `ReturnQty`, `UnitCost`, `ReturnReason` | **PK:** `SupplierReturnLineID` |

### 4.2. Các quy ước và Bất biến Nghiệp vụ Procurement
1. **Một kho nhận cố định & Phê duyệt PO độc lập (`PR-Q01`, `PR-Q06`, `BR-PO02`, `BR-PO03`):** Mỗi `PurchaseOrder` ấn định duy nhất 1 `DestinationWarehouseID` và phải được phê duyệt (`ApprovedBy <> CreatedBy`) trước khi gửi NCC; mọi phiếu `GoodsReceipt` của PO bắt buộc trùng `WarehouseID`.
2. **Tiếp nhận nhiều đợt & Kiểm định QC đầu vào (`PR-Q02..04`, `BR-PO05..08`):** Hỗ trợ nhận hàng nhiều đợt với $\sum \text{ReceivedQty} \le \text{OrderedQty}$ và $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$. Chỉ `AcceptedQty` mới sinh giao dịch `'PURCHASE_RECEIPT'`, tăng `OnHandQty` và cập nhật giá vốn `CurrentAverageCost`.
3. **Xuất trả NCC & Hạch toán chênh lệch giá vốn WAC (`PR-Q05`, `CORR-08`, `BR-PO10`):** `SupplierReturnLine` tham chiếu `GoodsReceiptLineID` gốc; khi xuất trả (`SUPPLIER_RETURN_OUT`), giá trị tồn kho giảm theo WAC hiện hành của kho xuất, phần chênh lệch so với đơn giá mua PO gốc hạch toán vào `SupplierReturn.CostVarianceAccountCode`.

---

## 5. Subject Area 04: Customer & Identity (9 Thực thể: `E29` – `E37`)

### 5.1. Danh mục 9 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E29** | `Account` | Strong Entity | Tài khoản xác thực danh tính trung tâm (`AccountType`: `'CUSTOMER'` / `'EMPLOYEE'`). | `AccountID`, `AccountType`, `Email`, `Phone`, `PasswordHash`, `AccountStatus`, `CreatedAt`, `LastLoginAt` | **PK:** `AccountID`<br>**UK:** `Email` *(Filtered)*, `Phone` *(Filtered)* |
| **E30** | `Customer` | TPT Subtype Entity | Hồ sơ nghiệp vụ của khách hàng mua sắm B2C (`CORR-04`). | `AccountID`, `FullName`, `DateOfBirth`, `Gender`, `CustomerStatus`, `CreatedAt` | **PK, FK:** `AccountID` *(trỏ `Account.AccountID`)* |
| **E31** | `Employee` | TPT Subtype Entity | Hồ sơ nhân sự nội bộ vận hành hệ thống (`CORR-04`). | `AccountID`, `EmployeeCode`, `FullName`, `Department`, `EmployeeStatus`, `CreatedAt` | **PK, FK:** `AccountID` *(trỏ `Account.AccountID`)*<br>**UK:** `EmployeeCode` |
| **E32** | `Role` | Strong / Master Entity | Danh mục vai trò người dùng trong hệ thống phân quyền RBAC. | `RoleID`, `RoleName`, `RoleDescription` | **PK:** `RoleID`<br>**UK:** `RoleName` |
| **E33** | `Permission` | Strong / Master Entity | Danh mục quyền hạn thao tác trên tài nguyên nghiệp vụ. | `PermissionID`, `PermissionCode`, `PermissionName`, `Description` | **PK:** `PermissionID`<br>**UK:** `PermissionCode` |
| **E34** | `AccountRole` | Associative Entity | Gán một hoặc nhiều vai trò cho một tài khoản `Account`. | `AccountID`, `RoleID`, `AssignedAt`, `AssignedBy` | **PK:** `(AccountID, RoleID)` |
| **E35** | `RolePermission` | Associative Entity | Ma trận cấp quyền chi tiết cho từng vai trò `Role`. | `RoleID`, `PermissionID` | **PK:** `(RoleID, PermissionID)` |
| **E36** | `CustomerAddress` | Weak Entity | Sổ địa chỉ nhận hàng cá nhân của khách (`CustomerID` trỏ `Customer.AccountID`). | `AddressID`, `CustomerID`, `RecipientName`, `RecipientPhone`, `Province`, `District`, `Ward`, `StreetAddress`, `IsDefault`, `AddressStatus`, `CreatedAt` | **PK:** `AddressID`<br>**UK:** `(CustomerID) WHERE IsDefault = 1` |
| **E37** | `AccountVerification` | Weak / Event Entity | Lịch sử yêu cầu xác minh OTP / Email có thời hạn `ExpiresAt`. | `VerificationID`, `AccountID`, `VerificationType`, `VerificationTarget`, `TokenHash`, `VerificationStatus`, `CreatedAt`, `ExpiresAt`, `VerifiedAt` | **PK:** `VerificationID` |

### 5.2. Các quy ước và Bất biến Nghiệp vụ Customer & Identity
1. **Kế thừa Table-Per-Type (TPT) Thuần khiết & Ràng buộc XOR (`CID-Q01`, `CORR-04`):** `Account(AccountID, AccountType)` phân tách loại trừ tuyệt đối giữa `Customer(AccountID PK, FK)` và `Employee(AccountID PK, FK)`. Mọi thao tác đăng ký thực thi nguyên tử qua `usp_Identity_RegisterAccount`; thu hồi quyền DML trực tiếp trên `Account`, `Customer`, `Employee`.
2. **Định danh Linh hoạt & Bảo mật Xác thực (`CID-Q03..05`, `BR-ID01..03`, `BR-ID13`):** Bắt buộc có ít nhất `Email` hoặc `Phone` (`CHECK (Email IS NOT NULL OR Phone IS NOT NULL)`), cưỡng chế duy nhất bằng Filtered Unique Indexes `UX_Account_Email` và `UX_Account_Phone`. Mật khẩu và mã OTP chỉ lưu dạng băm (`PasswordHash`, `TokenHash`).
3. **RBAC Động & Sổ địa chỉ Mặc định (`CID-Q02`, `CID-Q06`):** Phân quyền đa vai trò qua `AccountRole` – `RolePermission`; mỗi khách hàng có tối đa 1 địa chỉ mặc định qua `UX_CustomerAddress_Default WHERE IsDefault = 1`.

---

## 6. Subject Area 05: Shopping (2 Thực thể: `E38` – `E39`)

### 6.1. Danh mục 2 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E38** | `ShoppingCart` | Strong / Document Entity | Giỏ hàng tập trung của khách hàng (`CustomerID` trỏ `Customer.AccountID`). | `CartID`, `CustomerID`, `CartStatus`, `CreatedAt`, `UpdatedAt`, `LastActivityAt` | **PK:** `CartID`<br>**UK:** `(CustomerID) WHERE CartStatus = 'ACTIVE'` |
| **E39** | `CartItem` | Weak / Detail Entity | Dòng sản phẩm SKU được khách thêm vào giỏ (không khóa giá, không giữ kho). | `CartItemID`, `CartID`, `VariantID`, `Quantity`, `AddedAt`, `UpdatedAt` | **PK:** `CartItemID`<br>**UK:** `(CartID, VariantID)` |

### 6.2. Các quy ước và Bất biến Nghiệp vụ Shopping
1. **Duy nhất 1 Giỏ hàng `ACTIVE` & Cộng dồn SKU (`SHOP-Q01`, `SHOP-Q02`):** Cưỡng chế bằng `UX_ShoppingCart_Active ON ShoppingCart(CustomerID) WHERE CartStatus = 'ACTIVE'` và `UQ_CartItem_Cart_Variant ON (CartID, VariantID)`.
2. **Không giữ tồn kho, Không khóa giá & Đóng băng khi Checkout (`SHOP-Q03..06`):** `CartItem` luôn đối chiếu giá và tồn khả dụng thời gian thực từ `ProductVariant` / `InventoryBalance`; khi đặt đơn thành công, giỏ hàng chuyển nguyên tử sang `'CONVERTED'` và khóa mọi chỉnh sửa.

---

## 7. Subject Area 06: Sales & Order Management (4 Thực thể: `E40` – `E43`)

### 7.1. Danh mục 4 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E40** | `SalesOrder` | Strong / Document Entity | Hợp đồng mua bán chính thức hoặc đơn đổi hàng liên kết (`OrderType`, `ParentOrderID`) (Header). | `OrderID`, `OrderNumber`, `CustomerID`, `CartID`, `OrderType`, `ParentOrderID`, `OrderStatus`, `OrderDate`, `SubtotalAmount`, `DiscountAmount`, `TaxAmount`, `ShippingFee`, `ShippingDiscountAmount`, `TotalAmount`, `CustomerNote`, `CreatedAt`, `UpdatedAt`, `DeliveredAt`, `ClosedAt` | **PK:** `OrderID`<br>**UK:** `OrderNumber`, `(CartID) WHERE CartID IS NOT NULL` |
| **E41** | `OrderLine` | Weak / Detail Entity | Dòng hợp đồng bán lưu snapshot SKU, giá đã gồm GTGT, thuế bóc tách, LRM, hủy thiếu hàng và hoàn tiền. | `OrderLineID`, `OrderID`, `VariantID`, `SKU`, `ProductName`, `VariantName`, `VariantAttributesSnapshot`, `UnitPrice`, `VatRate`, `TaxAmount`, `OrderedQty`, `CancelledQty`, `LineGrossAmount`, `LineProductDiscount`, `AllocatedOrderDiscount`, `LineNetAmount`, `RefundedQty`, `AccumulatedRefundedAmount` | **PK:** `OrderLineID` |
| **E42** | `OrderAddress` | Weak / Snapshot Entity | Bản chụp bất biến địa chỉ nhận hàng và thông tin xuất hóa đơn B2B tại giây phút đặt đơn ($1:1$). | `OrderAddressID`, `OrderID`, `RecipientName`, `RecipientPhone`, `Province`, `District`, `Ward`, `StreetAddress`, `BuyerCompanyName`, `BuyerTaxCode`, `BuyerCompanyAddress`, `BuyerInvoiceEmail` | **PK:** `OrderAddressID`<br>**UK:** `OrderID` |
| **E43** | `OrderStatusHistory` | Weak / Event Entity | Nhật ký bất biến mọi lần chuyển đổi `OrderStatus` của đơn hàng. | `StatusHistoryID`, `OrderID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedBy`, `ChangeReason` | **PK:** `StatusHistoryID` |

### 7.2. Các quy ước và Bất biến Nghiệp vụ Sales & Order Management
1. **Cấm Guest Checkout, Giao dịch Nguyên tử & `REVOKE DML` (`BD-02`, `SALES-Q01`, `CORR-04`, `CORR-05`):** `CustomerID NOT NULL`; việc tạo `SalesOrder`, `OrderAddress`, `OrderLine`, phân bổ kho, giữ hàng và `Payment` bắt buộc thực thi nguyên tử trong `usp_Checkout_CreateOrder` dưới sự bảo vệ của `IdempotencyRecord` (E74).
2. **Định giá Đã Gồm GTGT, LRM & Hủy thiếu hàng (`CORR-03`):** Giá bán lẻ `UnitPrice` đã gồm thuế GTGT: $\text{LineNetAmount} = \text{LineGrossAmount} - \text{LineProductDiscount} - \text{AllocatedOrderDiscount}$, thuế nội hàm $\text{TaxAmount} = \text{LineNetAmount} - \text{ROUND}(\text{LineNetAmount}/(1+\text{VatRate}/100), 0)$, và $\text{TotalAmount} = \text{SubtotalAmount} - \text{DiscountAmount} + (\text{ShippingFee} - \text{ShippingDiscountAmount})$. `OrderLine.CancelledQty` theo dõi lượng hủy thiếu hàng trước xuất kho (`PARTIALLY_CANCELLED`).
3. **Snapshot Bất biến & Đơn đổi hàng Liên kết (`SALES-Q05..07`, `CORR-01`, `CORR-09`):** `OrderAddress` snapshot độc lập địa chỉ nhận và thông tin hóa đơn B2B; `OrderLine.VariantAttributesSnapshot` lưu mảng JSON 6 khóa chuẩn hóa. Đơn đổi hàng RMA khởi tạo `SalesOrder` mới với `OrderType = 'EXCHANGE_REPLACEMENT'` và `ParentOrderID` trỏ về đơn gốc.

---

## 8. Subject Area 07: Fulfillment & Shipping (6 Thực thể: `E44` – `E48`, `E75`)

### 8.1. Danh mục 6 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E44** | `Carrier` | Strong / Master Entity | Đơn vị vận chuyển đối tác (GHN, GHTK, ViettelPost, J&T...). | `CarrierID`, `CarrierCode`, `CarrierName`, `ContactPhone`, `Email`, `Website`, `CarrierStatus`, `CreatedAt` | **PK:** `CarrierID`<br>**UK:** `CarrierCode` |
| **E45** | `Shipment` | Strong / Document Entity | Kiện hàng xuất từ duy nhất 1 kho vật lý giao cho khách theo đơn. | `ShipmentID`, `ShipmentNumber`, `OrderID`, `WarehouseID`, `CarrierID`, `TrackingNumber`, `ShipmentStatus`, `ShippingFee`, `CODAmountToCollect`, `CreatedAt`, `HandedOverAt`, `DeliveredAt` | **PK:** `ShipmentID`<br>**UK:** `ShipmentNumber`, `(CarrierID, TrackingNumber) WHERE TrackingNumber IS NOT NULL` |
| **E46** | `ShipmentLine` | Associative / Weak Entity | Chi tiết SKU đóng gói trong kiện theo `AllocationID` kèm giá vốn xuất kho. | `ShipmentLineID`, `ShipmentID`, `OrderLineID`, `AllocationID`, `ShipmentQty`, `OutboundUnitCost` | **PK:** `ShipmentLineID`<br>**UK:** `(ShipmentID, AllocationID)` |
| **E47** | `DeliveryAttempt` | Weak Entity | Lịch sử từng lần bưu tá thực hiện giao kiện hàng (tối đa 3 lần). | `AttemptID`, `ShipmentID`, `AttemptNumber`, `AttemptedAt`, `AttemptStatus`, `FailureReason`, `RecipientName`, `DeliveryNote` | **PK:** `AttemptID`<br>**UK:** `(ShipmentID, AttemptNumber)` |
| **E48** | `ShipmentStatusHistory` | Weak / Audit Entity | Lịch sử chuyển đổi trạng thái kiện hàng từ Webhook hãng hoặc kho. | `ShipmentStatusHistoryID`, `ShipmentID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedBy`, `ChangeReason` | **PK:** `ShipmentStatusHistoryID` |
| **E75** | `DeliveryReturnReceipt` | Strong / Document Entity | Biên bản thủ kho xác nhận thực nhận kiện hàng giao thất bại quay về kho (`BD-07`). | `DeliveryReturnReceiptID`, `ReceiptNumber`, `ShipmentID`, `WarehouseID`, `ReceivedAt`, `ReceivedBy`, `ReceiptStatus`, `ConditionNote`, `PostedAt` | **PK:** `DeliveryReturnReceiptID`<br>**UK:** `ReceiptNumber`, `ShipmentID` |

### 8.2. Các quy ước và Bất biến Nghiệp vụ Fulfillment & Shipping
1. **Một Kho Xuất Mỗi Kiện & Tách bạch Phí Vận chuyển (`BD-03`, `SHIP-Q01`, `SHIP-Q07`):** Mỗi `Shipment` xuất phát từ đúng 1 `WarehouseID` khớp `OrderWarehouseAllocation.WarehouseID`, thỏa $\sum \text{ShipmentQty} \le \text{OrderedQty} - \text{CancelledQty}$. Tách bạch cước phí thực trả cho hãng `Shipment.ShippingFee` với phí thu khách `SalesOrder.ShippingFee`.
2. **Xuất bãi `SHIPMENT_OUT` & Hoàn kho Giao thất bại qua `E75` (`BD-07`, `CORR-06`, `CORR-08`):** Khi bàn giao `DISPATCHED`, hệ thống snapshot `ShipmentLine.OutboundUnitCost` và ghi `SHIPMENT_OUT` giảm cả `OnHandQty` lẫn `ReservedQty`. Quá 3 lần `DeliveryAttempt` thất bại, kiện chuyển `'RETURNED_TO_SENDER'` (chưa cộng tồn kho); chỉ khi thủ kho ký chốt `DeliveryReturnReceipt` (E75, `POSTED`) mới ghi `'DELIVERY_RETURN_IN'` và tăng lại `OnHandQty`.

---

## 9. Subject Area 08: Promotion & Voucher (6 Thực thể: `E49` – `E54`)

### 9.1. Danh mục 6 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E49** | `Promotion` | Strong / Document Entity | Chương trình khuyến mãi phiên bản hóa 3 tầng (`PromotionScope`: `'PRODUCT'`, `'ORDER'`, `'SHIPPING'`). | `PromotionID`, `PromotionCode`, `VersionNumber`, `SupersedesPromotionID`, `PromotionName`, `PromotionType`, `PromotionScope`, `DiscountType`, `DiscountValue`, `MaxDiscountAmount`, `MinOrderAmount`, `StartAt`, `EndAt`, `PromotionStatus`, `Priority`, `IsStackable`, `MaxUsage`, `MaxUsagePerCustomer`, `PendingCount`, `UsedCount`, `CreatedAt` | **PK:** `PromotionID`<br>**UK:** `(PromotionCode, VersionNumber)` |
| **E50** | `PromotionCondition` | Weak Entity | Điều kiện cấu hình chuẩn hóa mà đơn hàng/dòng hàng phải thỏa mãn. | `ConditionID`, `PromotionID`, `ConditionType`, `ComparisonOperator`, `ConditionValue` | **PK:** `ConditionID` |
| **E51** | `PromotionProduct` | Associative Entity | Danh sách biến thể SKU hưởng khuyến mãi cấp sản phẩm kèm hạn mức Flash Sale. | `PromotionID`, `VariantID`, `FlashSaleStockLimit`, `FlashSaleSoldCount`, `MaxQtyPerCustomer`, `AssignedAt` | **PK:** `(PromotionID, VariantID)` |
| **E52** | `Voucher` | Strong / Document Entity | Mã giảm giá cấp đơn hàng (`'ORDER'`) hoặc miễn giảm vận chuyển (`'SHIPPING'`). | `VoucherID`, `PromotionID`, `VoucherCode`, `VoucherType`, `AssignedCustomerID`, `VoucherStatus`, `MaxUsage`, `PendingCount`, `UsedCount`, `StartAt`, `EndAt` | **PK:** `VoucherID`<br>**UK:** `VoucherCode` |
| **E53** | `PromotionUsage` | Associative / Ledger Entity | Sổ cái tiêu thụ ưu đãi và snapshot mức chiết khấu thực tế. | `PromotionUsageID`, `PromotionID`, `OrderID`, `OrderLineID`, `VoucherID`, `DiscountAmount`, `UsageStatus`, `UsedAt`, `ReversedAt`, `ReversalReason` | **PK:** `PromotionUsageID` |
| **E54** | `PromotionTargetCustomer` | Associative Entity | Danh sách khách hàng VIP được chỉ định hưởng chương trình riêng tư. | `PromotionID`, `CustomerID`, `AssignedAt` | **PK:** `(PromotionID, CustomerID)` |

### 9.2. Các quy ước và Bất biến Nghiệp vụ Promotion & Voucher
1. **Kiến trúc Khuyến mãi 3 Tầng & Chống Gom Hàng Flash Sale (`BD-04`, `BD-05`):** Tách 3 phạm vi độc lập `'PRODUCT'` (tối đa 1 trên mỗi `OrderLine`), `'ORDER'` (tối đa 1 Voucher tiền hàng trên `SalesOrder`, phân bổ LRM) và `'SHIPPING'` (tối đa 1 Voucher Freeship trên `SalesOrder`, ghi vào `ShippingDiscountAmount`), chỉ kết hợp khác tầng khi `IsStackable = 1`. Chiến dịch Flash Sale kiểm soát thêm trần `FlashSaleStockLimit` và `MaxQtyPerCustomer` trên `PromotionProduct`.
2. **Hạn mức Nguyên tử & Phiên bản hóa Bất biến (`PROMO-Q04..08`):** Khóa kiểm tra nguyên tử `MaxUsage` / `MaxUsagePerCustomer`; vòng đời `PromotionUsage` đi từ `'PENDING'` $\to$ `'CONSUMED'` (khi đơn trả trước `'PAID'` hoặc đơn COD `'CONFIRMED'`) hoặc `'REVERSED'`. Cấm sửa chương trình đã phát sinh `PromotionUsage` (phải tạo `VersionNumber + 1` trỏ `SupersedesPromotionID`).

---

## 10. Subject Area 09: Payment & Settlement (10 Thực thể: `E55` – `E59`, `E71`, `E74`, `E77` – `E79`)

### 10.1. Danh mục 10 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E55** | `PaymentMethod` | Strong / Master Entity | Danh mục phương thức thanh toán (COD, VNPAY, MOMO, BANK_TRANSFER, CARD...). | `PaymentMethodID`, `MethodCode`, `MethodName`, `MethodType`, `MethodStatus`, `CreatedAt` | **PK:** `PaymentMethodID`<br>**UK:** `MethodCode` |
| **E56** | `Payment` | Strong / Document Entity | Hồ sơ nghĩa vụ và trạng thái thanh toán tổng thể của đơn hàng ($1:1$). | `PaymentID`, `OrderID`, `PaymentMethodID`, `PaymentStatus`, `CarrierSettlementStatus`, `AmountDue`, `AmountPaid`, `AmountRefunded`, `CreatedAt`, `PaidAt` | **PK:** `PaymentID`<br>**UK:** `OrderID` |
| **E57** | `PaymentTransaction` | Weak / Event Entity | Nhật ký từng lần thực hiện thu tiền/gọi cổng thanh toán hoặc quyết toán COD. | `PaymentTransactionID`, `PaymentID`, `TransactionReference`, `GatewayTransactionID`, `TransactionType`, `TransactionStatus`, `Amount`, `TransactionFee`, `InitiatedAt`, `CompletedAt`, `GatewayResponseCode`, `GatewayResponseMessage`, `GatewayResponseData` | **PK:** `PaymentTransactionID`<br>**UK:** `TransactionReference` |
| **E58** | `Refund` | Strong / Document Entity | Hồ sơ yêu cầu hoàn tiền (từ hủy đơn trả trước hoặc từ kết luận KCS `E72`). | `RefundID`, `PaymentID`, `ReturnLineAssessmentID`, `RefundReference`, `RefundAmount`, `RefundReason`, `RefundStatus`, `RequestedAt`, `RequestedBy`, `ApprovedBy`, `ApprovedAt` | **PK:** `RefundID`<br>**UK:** `RefundReference`, `(ReturnLineAssessmentID) WHERE ReturnLineAssessmentID IS NOT NULL` |
| **E59** | `RefundTransaction` | Weak / Event Entity | Giao dịch thực thi chi trả tiền hoàn, truy vết `OriginalPaymentTransactionID`. | `RefundTransactionID`, `RefundID`, `OriginalPaymentTransactionID`, `TransactionReference`, `GatewayTransactionID`, `RefundMethod`, `TransactionStatus`, `Amount`, `TransactionFee`, `InitiatedAt`, `CompletedAt`, `GatewayResponseCode`, `GatewayResponseMessage` | **PK:** `RefundTransactionID`<br>**UK:** `TransactionReference` |
| **E71** | `PaymentWebhookLog` | Event / Support Entity | Nhật ký lưu bền vững sự kiện Webhook cổng thanh toán chống xử lý trùng (`CORR-05`). | `WebhookLogID`, `PaymentMethodID`, `MerchantAccountID`, `ProviderEventID`, `Environment`, `PayloadJSON`, `SignatureVerified`, `ProcessingStatus`, `ReceivedAt`, `ProcessedAt`, `ErrorMessage` | **PK:** `(WebhookLogID, ReceivedAt)`<br>**UK:** `(PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL` |
| **E74** | `IdempotencyRecord` | Event / Support Entity | Bảng khóa chống gửi lặp API đặt đơn & thanh toán kèm `RequestHash` (`CORR-05`). | `IdempotencyRecordID`, `AccountID`, `OperationType`, `IdempotencyKey`, `RequestHash`, `ProcessingStatus`, `HttpStatusCode`, `ResponseSnapshot`, `ResourceType`, `ResourceID`, `CreatedAt`, `ExpiresAt` | **PK:** `IdempotencyRecordID`<br>**UK:** `(AccountID, OperationType, IdempotencyKey)` |
| **E77** | `CODCollection` | Weak / Event Entity | Lớp 1 đối soát COD: Ghi nhận tiền mặt bưu tá thực thu trên từng kiện `Shipment` (`CORR-07`). | `CODCollectionID`, `ShipmentID`, `PaymentID`, `CarrierID`, `ExpectedCODAmount`, `CollectedAmount`, `CollectionStatus`, `CollectedAt`, `CarrierReceiptRef` | **PK:** `CODCollectionID`<br>**UK:** `ShipmentID` |
| **E78** | `CODRemittance` | Strong / Document Entity | Lớp 2 đối soát COD: Bảng kê và đợt chuyển khoản tiền thu hộ từ `Carrier` về công ty. | `CODRemittanceID`, `RemittanceNumber`, `CarrierID`, `StatementReference`, `TotalCollectedAmount`, `CarrierFeeDeducted`, `NetRemittedAmount`, `RemittanceStatus`, `RemittedAt`, `ReceivedBankTxRef`, `CreatedAt` | **PK:** `CODRemittanceID`<br>**UK:** `RemittanceNumber`, `(CarrierID, StatementReference)` |
| **E79** | `CODReconciliation` | Weak / Settlement Entity | Lớp 3 đối soát COD: Khớp từng `CODCollection` vào `CODRemittance` và xử lý chênh lệch. | `CODReconciliationID`, `CODRemittanceID`, `CODCollectionID`, `PaymentTransactionID`, `ReconciledAmount`, `VarianceAmount`, `ReconciliationStatus`, `ReconciledBy`, `ReconciledAt`, `Notes` | **PK:** `CODReconciliationID`<br>**UK:** `CODCollectionID` |

### 10.2. Các quy ước và Bất biến Nghiệp vụ Payment & Settlement
1. **Quan hệ Mandatory $1:1$, Idempotency & Webhook Bền vững (`PAY-Q01`, `CORR-04`, `CORR-05`, `CORR-06`):** Mỗi `SalesOrder` có đúng 1 `Payment`. Tách biệt `Payment.PaymentStatus` (`PENDING`, `AUTHORIZED`, `PARTIALLY_PAID`, `PAID`, `PARTIALLY_REFUNDED`, `REFUNDED`, `EXPIRED`, `CANCELLED`) khỏi `PaymentTransaction.TransactionStatus` (`INITIATED`, `PENDING`, `SUCCEEDED`, `FAILED`, `CANCELLED`). Khóa `IdempotencyRecord` (E74) chống gửi lặp API và lưu bền vững `PaymentWebhookLog` (E71) trước khi phản hồi HTTP 200.
2. **Đối soát COD 2 Đầu & 3 Lớp (`CORR-07`):** Khi bưu tá thu tiền từng kiện, ghi nhận `CODCollection` (E77) cập nhật tiến độ khách trả (`PARTIALLY_PAID` / `PAID`), đồng thời theo dõi công nợ hãng qua `CarrierSettlementStatus` và khớp quyết toán qua `CODRemittance` (E78) $\to$ `CODReconciliation` (E79) $\to$ `PaymentTransaction` (`COD_SETTLEMENT`). Nếu kiện COD giao thất bại bị hủy phần chưa thu (`PARTIALLY_CANCELLED`), điều chỉnh giảm `Payment.AmountDue` tương ứng để khép kín công nợ các kiện đã giao.
3. **Hoàn tiền Phê duyệt Nội bộ & Truy vết Giao dịch Gốc (`PAY-Q04..06`, `CORR-08`):** Mọi yêu cầu `Refund` phải có `ApprovedBy`, thỏa $\sum \text{RefundAmount} \le \text{AmountPaid} - \text{AmountRefunded}$; `RefundTransaction` bắt buộc trỏ `OriginalPaymentTransactionID` về giao dịch `'SUCCEEDED'` gốc và chỉ cộng `AmountRefunded` khi `RefundTransaction` đạt `'SUCCEEDED'`.

---

## 11. Subject Area 10: Return & Exchange (7 Thực thể: `E60` – `E64`, `E72`, `E76`)

### 11.1. Danh mục 7 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E60** | `ReturnRequest` | Strong / Document Entity | Yêu cầu đổi trả hàng chính thức từ khách hàng hoặc CSKH tạo hộ (RMA Header). | `ReturnRequestID`, `ReturnNumber`, `OrderID`, `CustomerID`, `WarehouseID`, `CreatedByAccountID`, `ReturnStatus`, `ReturnReason`, `CustomerNote`, `RequestedAt`, `ReviewedBy`, `ReviewedAt`, `ReviewNote` | **PK:** `ReturnRequestID`<br>**UK:** `ReturnNumber` |
| **E61** | `ReturnRequestLine` | Weak / Detail Entity | Chi tiết từng dòng đơn hàng gốc và số lượng khách yêu cầu đổi trả. | `ReturnRequestLineID`, `ReturnRequestID`, `OrderLineID`, `RequestedQty`, `ApprovedQty`, `ReturnReason`, `LineStatus` | **PK:** `ReturnRequestLineID`<br>**UK:** `(ReturnRequestID, OrderLineID)` |
| **E62** | `ReturnReceipt` | Strong / Document Entity | Phiếu tiếp nhận bưu kiện hoàn trả thực tế tại kho vật lý chỉ định. | `ReturnReceiptID`, `ReceiptNumber`, `ReturnRequestID`, `WarehouseID`, `ReceivedAt`, `ReceivedBy`, `ReceiptStatus`, `InspectionNote`, `PostedAt` | **PK:** `ReturnReceiptID`<br>**UK:** `ReceiptNumber` |
| **E63** | `ReturnReceiptLine` | Weak / Detail Entity | Chi tiết số lượng thực nhận tại bến nhập và mô tả ngoại quan ban đầu. | `ReturnReceiptLineID`, `ReturnReceiptID`, `ReturnRequestLineID`, `ReceivedQty`, `ConditionDescription` | **PK:** `ReturnReceiptLineID` |
| **E64** | `ReturnStatusHistory` | Weak / Audit Entity | Nhật ký bất biến mọi lần chuyển đổi trạng thái của yêu cầu RMA. | `ReturnStatusHistoryID`, `ReturnRequestID`, `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedByAccountID`, `ChangeReason` | **PK:** `ReturnStatusHistoryID` |
| **E72** | `ReturnLineAssessment` | Weak / Assessment Entity | Biên bản thẩm định KCS phân tách lô & phiên bản hóa, tách quyền hoàn tiền khỏi kho (`CORR-01`). | `AssessmentID`, `ReturnReceiptLineID`, `AssessmentGroupNo`, `AssessmentVersion`, `IsCurrent`, `AssessmentStatus`, `AssessedQty`, `InventoryDisposition`, `FaultParty`, `RefundEligibilityStatus`, `ResolutionType`, `ApprovedRefundAmount`, `ShippingRefundAmount`, `DisputeReason`, `InspectionNotes`, `AssessedAt`, `AssessedBy` | **PK:** `AssessmentID`<br>**UK:** `(ReturnReceiptLineID, AssessmentGroupNo) WHERE IsCurrent = 1` |
| **E76** | `ReturnShipmentAllocation` | Associative Entity | Cầu nối truy vết số lượng trả thuộc kiện giao `ShipmentLine` nào (`CORR-10`). | `ReturnShipmentAllocationID`, `ReturnRequestLineID`, `ShipmentLineID`, `AllocatedReturnQty`, `CreatedAt` | **PK:** `ReturnShipmentAllocationID`<br>**UK:** `(ReturnRequestLineID, ShipmentLineID)` |

### 11.2. Các quy ước và Bất biến Nghiệp vụ Return & Exchange
1. **Truy vết Kiện giao & Hạn Đổi trả Độc lập (`RET-Q02`, `CORR-10`):** Mỗi dòng RMA phân bổ vào dòng kiện giao qua `ReturnShipmentAllocation` (E76), cưỡng chế `RequestedAt <= Shipment.DeliveredAt + PolicyReturnDays`. Mỗi `ReturnRequest` chỉ định đúng 1 kho tiếp nhận `WarehouseID`.
2. **Tiếp nhận Bến nhập vào Cách ly & Giám định KCS Phiên bản hóa (`CORR-01`, `CORR-02`):** Khi chốt phiếu bến nhập `ReturnReceipt` (`POSTED`), hàng thực nhận được ghi sổ `RETURN_RECEIPT_IN` vào bể cách ly (`OnHandDelta = +ReceivedQty, QuarantinedDelta = +ReceivedQty`, tồn bán `SELLABLE` chưa đổi). Sau đó `ReturnLineAssessment` (E72) tách lô (`AssessmentGroupNo`) và phiên bản hóa (`IsCurrent = 1`) để chuyển từ `QUARANTINED` sang `'SELLABLE'` (`QuarantinedDelta = -AssessedQty`, cập nhật WAC theo `OutboundUnitCost`), giữ `'QUARANTINED'`, hoặc xuất hủy `'DISPOSED'` (`SCRAP_OUT`: `OnHandDelta = -AssessedQty, QuarantinedDelta = -AssessedQty`).
3. **Tách Quyền Hoàn tiền, Công thức $\Delta F$, Clawback & Đơn đổi bù `EXCHANGE` (`CORR-01`, `CORR-03`):** Hàng `'DISPOSED'` do `'VENDOR_FAULT'` / `'CARRIER_FAULT'` vẫn được duyệt hoàn tiền 100%; hoàn tiền từng mốc tính theo $\Delta\text{Refund} = F(a+b) - F(a)$ (khấu trừ Clawback nếu lỗi khách làm phần hàng giữ lại tụt dưới `MinOrderAmount` của voucher cấp đơn); phương án `'EXCHANGE'` sinh đơn đổi bù `EXCHANGE_REPLACEMENT`.

---

## 12. Subject Area 11: Audit & Reporting (6 Thực thể: `E65` – `E70`)

### 12.1. Danh mục 6 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E65** | `AuditLog` | Audit / Ledger Entity | Nhật ký an ninh WORM ghi nhận thao tác nhạy cảm trong hệ thống (Header). | `AuditLogID`, `AccountID`, `ActionType`, `EntityName`, `RecordID`, `ActionAt`, `IPAddress`, `UserAgent`, `Result`, `Reason`, `CorrelationID`, `RetentionUntil` | **PK:** `(AuditLogID, ActionAt)` *(Partition-Aligned)* |
| **E66** | `AuditLogDetail` | Weak / Audit Entity | Chi tiết cặp giá trị trước/sau (`OldValue`/`NewValue`) của sự kiện audit. | `AuditLogDetailID`, `AuditLogID`, `ActionAt`, `FieldName`, `OldValue`, `NewValue` | **PK:** `AuditLogDetailID` |
| **E67** | `ReportDefinition` | Catalog / Master Entity | Danh mục mẫu báo cáo vận hành và tài chính được hệ thống hỗ trợ. | `ReportDefinitionID`, `ReportCode`, `ReportName`, `Description`, `DataSource`, `ReportType`, `IsActive`, `CreatedAt` | **PK:** `ReportDefinitionID`<br>**UK:** `ReportCode` |
| **E68** | `ReportExecution` | Event / Audit Entity | Nhật ký lịch sử các lần chạy truy vấn hoặc xuất file báo cáo. | `ReportExecutionID`, `ReportDefinitionID`, `AccountID`, `Parameters`, `ExecutionStatus`, `RequestedAt`, `CompletedAt`, `ExportFormat`, `ResultLocation`, `ErrorMessage` | **PK:** `ReportExecutionID` |
| **E69** | `ReportSnapshot` | Strong / Document Entity | Ảnh chụp số liệu chốt kỳ kế toán cố định không bị biến động hồi tố. | `ReportSnapshotID`, `ReportDefinitionID`, `SnapshotCode`, `PeriodStart`, `PeriodEnd`, `SnapshotAt`, `SnapshotStatus`, `DataLocation`, `CreatedBy`, `CreatedAt` | **PK:** `ReportSnapshotID`<br>**UK:** `SnapshotCode` |
| **E70** | `ReportPermission` | Associative Entity | Ma trận phân quyền Xem / Xuất / Chốt snapshot báo cáo theo vai trò `Role`. | `ReportDefinitionID`, `RoleID`, `CanView`, `CanExport`, `CanCreateSnapshot` | **PK:** `(ReportDefinitionID, RoleID)` |

### 12.2. Các quy ước và Bất biến Nghiệp vụ Audit & Reporting
1. **Nhật ký An ninh Chỉ Ghi WORM & Partition-Aligned (`AR-Q01..04`):** Chỉ ghi log các thao tác nhạy cảm, cấm lưu mật khẩu/hash/OTP/token; thu hồi hoàn toàn quyền `UPDATE` và `DELETE` trên `AuditLog` / `AuditLogDetail`; khóa chính `(AuditLogID, ActionAt)` căn chỉnh phân vùng thời gian phục vụ lưu trữ lạnh theo `RetentionUntil`.
2. **Báo cáo Vận hành Song song Snapshot Chốt kỳ & Phân quyền Ma trận (`AR-Q05..07`):** Báo cáo vận hành đọc không khóa dưới `RCSI`; báo cáo tài chính/kế toán đóng băng bất biến qua `ReportSnapshot`; phân quyền chi tiết `CanView`, `CanExport`, `CanCreateSnapshot` qua `ReportPermission`.
