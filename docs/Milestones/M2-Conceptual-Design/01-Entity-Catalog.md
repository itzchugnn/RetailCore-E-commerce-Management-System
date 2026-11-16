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
| **E09** | `VariantAttributeValue` | Associative Entity | Giá trị thuộc tính thực tế được chọn cấu thành nên một SKU cụ thể. | `VariantID`, `ProductID`, `AttributeID`, `AttributeValueID` | **PK:** `(VariantID, AttributeID)` |

### 2.2. Các quy ước nghiệp vụ của Product Catalog
1. **Quy ước vai trò thuộc tính (`ProductAttribute.AttributeRole`):**
   * `DESCRIPTIVE`: Thuộc tính mô tả chung, không tham gia phân biệt SKU (ví dụ: Chất liệu, Xuất xứ, Hướng dẫn bảo quản).
   * `VARIANT_DEFINING`: Thuộc tính định nghĩa biến thể, bắt buộc tham gia tổ hợp SKU (ví dụ: Kích thước, Màu sắc).
   * *Quy tắc loại trừ:* Một thuộc tính trong một sản phẩm chỉ nhận duy nhất một vai trò.
2. **Quy ước SKU mặc định:** Nếu sản phẩm không có thuộc tính định nghĩa biến thể (sản phẩm đơn chiếc), hệ thống tự sinh một `ProductVariant` mặc định với tổ hợp thuộc tính rỗng.
3. **Danh mục chính:** Sản phẩm Published bắt buộc có đúng 1 danh mục chính (`IsPrimary = 1`).

---

## 3. Subject Area 02: Warehouse & Inventory (Mô hình Đề xuất v0.1)

### 3.1. Danh mục 9 Thực thể Chuẩn hóa

| Mã | Thực thể (Entity) | Phân loại | Ý nghĩa nghiệp vụ | Danh mục Thuộc tính Chuẩn hóa | Định danh chính (PK / UK) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E10** | `Warehouse` | Strong Entity | Kho hàng vật lý thuộc mạng lưới phân phối của doanh nghiệp. | `WarehouseID`, `WarehouseCode`, `WarehouseName`, `Address`, `WarehouseStatus`, `CreatedAt` | **PK:** `WarehouseID`<br>**UK:** `WarehouseCode` |
| **E11** | `InventoryBalance` | Associative Entity | Số dư tồn kho tức thời của một SKU tại một kho hàng vật lý. | `WarehouseID`, `VariantID`, `OnHandQty`, `ReservedQty`, `LastUpdatedAt` | **PK:** `(WarehouseID, VariantID)` |
| **E12** | `OrderWarehouseAllocation` | Associative Entity | Quyết định phân bổ một phần số lượng của dòng đơn hàng cho một kho. | `AllocationID`, `OrderLineID`, `WarehouseID`, `VariantID`, `AllocatedQty`, `AllocationStatus`, `AllocatedAt` | **PK:** `AllocationID` |
| **E13** | `StockReservation` | Strong / Document Entity | Chứng từ/Phiếu giữ hàng tạm thời gắn với đơn hàng (Header). | `ReservationID`, `OrderID`, `ReservationStatus`, `CreatedAt`, `ExpiresAt`, `ReleasedAt` | **PK:** `ReservationID` |
| **E14** | `StockReservationLine` | Weak Entity | Chi tiết số lượng SKU được giữ tại một kho cụ thể (Line). | `ReservationLineID`, `ReservationID`, `AllocationID`, `VariantID`, `WarehouseID`, `ReservedQty`, `ReservationLineStatus` | **PK:** `ReservationLineID` |
| **E15** | `InventoryTransaction` | Strong / Document Entity | Chứng từ sổ cái ghi nhận nghiệp vụ làm thay đổi tồn kho (Header). | `TransactionID`, `TransactionType`, `TransactionStatus`, `ReferenceType`, `ReferenceID`, `CreatedAt`, `PostedAt` | **PK:** `TransactionID` |
| **E16** | `InventoryTransactionLine` | Weak Entity | Chi tiết dòng biến động tồn kho theo SKU và kho (`OnHandDelta`, `ReservedDelta`). | `TransactionLineID`, `TransactionID`, `VariantID`, `WarehouseID`, `OnHandDelta`, `ReservedDelta` | **PK:** `TransactionLineID` |
| **E17** | `StockCount` | Strong / Document Entity | Phiếu kiểm kê tồn kho định kỳ hoặc đột xuất tại một kho (Header). | `StockCountID`, `WarehouseID`, `CountStatus`, `StartedAt`, `CompletedAt`, `ApprovedAt` | **PK:** `StockCountID` |
| **E18** | `StockCountLine` | Weak Entity | Chi tiết ghi nhận số lượng hệ thống và số lượng thực tế kiểm đếm (Line). | `StockCountLineID`, `StockCountID`, `VariantID`, `SystemQty`, `CountedQty`, `VarianceQty` | **PK:** `StockCountLineID` |

### 3.2. Cơ chế cốt lõi của Warehouse & Inventory
1. **Công thức tồn khả dụng:**
   $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$$
2. **Cơ chế ghi sổ hai chiều (Dual-Delta Ledger):**
   * Mỗi dòng `InventoryTransactionLine` lưu trực tiếp `OnHandDelta` (mức tăng/giảm tồn vật lý) và `ReservedDelta` (mức tăng/giảm tồn giữ).
   * Giúp hệ thống cập nhật `InventoryBalance` đồng nhất và kiểm soát toàn vẹn không bị âm kho.
3. **Mô hình Header – Detail:**
   * Tách bạch rõ giữa phiếu tổng (`StockReservation`, `InventoryTransaction`, `StockCount`) và các dòng chi tiết phụ thuộc (`Line`) giúp hỗ trợ giao dịch nhiều kho, nhiều SKU trên cùng một đơn hàng hoặc chứng từ chuyển kho.
