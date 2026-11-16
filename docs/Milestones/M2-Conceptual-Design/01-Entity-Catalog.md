# M2.01 - ENTITY CATALOG (DANH MỤC THỰC THỂ KHÁI NIỆM)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.0
* **Trạng thái:** Working Draft (Tập trung chuyên sâu Product Catalog & Inventory)
* **Kế thừa:** Business Rules v1.1, Data Requirements v1.1, Quyết định BD-01 đến BD-07

---

## 1. Phân loại Loại hình Thực thể trong Mô hình Quan niệm

1. **Thực thể Mạnh (Strong Entity):** Tồn tại độc lập, có khóa định danh riêng trong thế giới thực (ví dụ: `Product`, `Warehouse`, `Category`, `Attribute`, `Supplier`, `User`).
2. **Thực thể Phụ thuộc / Yếu (Dependent / Weak Entity):** Sự tồn tại phụ thuộc vào thực thể cha, định danh cần dựa vào ngữ cảnh của cha (ví dụ: `AttributeValue`, `CustomerAddress`, `CartLine`).
3. **Thực thể Biến thể Trung tâm (Core SKU Entity):** `ProductVariant` – Có surrogate key riêng nhưng phụ thuộc logic vào `Product`, mang mã tự nhiên `SKU` duy nhất toàn hệ thống.
4. **Thực thể Liên kết (Associative Entity):** Giải quyết quan hệ $N-N$ và mang thuộc tính nghiệp vụ riêng (ví dụ: `ProductCategory` mang cờ `IsPrimary`, `ProductAttribute`, `VariantAttributeValue`, `UserRole`).
5. **Thực thể Giao dịch & Phân bổ (Transactional & Allocation Entities):** Đại diện cho các cam kết hoặc chứng từ luân chuyển (ví dụ: `Order`, `OrderLine`, `OrderAllocation`, `StockReservation`, `Shipment`).
6. **Thực thể Sổ cái Bất biến (Immutable Ledger / Audit Entity):** `StockMovement`, `OrderStatusHistory`, `AuditLog`.

---

## 2. Chi tiết Miền 1: Product Catalog (Sản phẩm & Biến thể động)

### 2.1. `Product` (Thực thể Mạnh)
* **Ý nghĩa nghiệp vụ:** Đại diện cho dòng sản phẩm chung (ví dụ: "Áo thun Polo Nam", "Bàn làm việc gỗ sồi"). Không chứa thông tin kích cỡ hay màu sắc cụ thể, không trực tiếp quản lý tồn kho hay bán lẻ.
* **Định danh nghiệp vụ:** `ProductID` (PK nhân tạo) hoặc `ProductCode` (Mã sản phẩm).
* **Thuộc tính ứng viên:** `ProductID`, `ProductCode`, `Name`, `Description`, `BrandID`, `BasePrice` (Giá tham chiếu), `IsActive`, `CreatedAt`, `UpdatedAt`.
* **Ràng buộc nghiệp vụ:** Không được xóa vật lý khi đã có SKU phát sinh đơn hàng (BR-PR07).

### 2.2. `Category` (Thực thể Mạnh – Cấu trúc Cây Đa cấp)
* **Ý nghĩa nghiệp vụ:** Danh mục phân loại sản phẩm theo cấu trúc phân cấp (ví dụ: Thời trang nam $\rightarrow$ Áo $\rightarrow$ Áo thun).
* **Định danh nghiệp vụ:** `CategoryID`.
* **Thuộc tính ứng viên:** `CategoryID`, `Name`, `Slug`, `ParentCategoryID` (Khóa ngoại đệ quy trỏ về chính nó), `Level`, `IsActive`.

### 2.3. `ProductCategory` (Thực thể Liên kết – BD-01 & BR-PR05, BR-PR10)
* **Ý nghĩa nghiệp vụ:** Bảng liên kết quan hệ $N-N$ giữa Sản phẩm và Danh mục. Cho phép 1 chiếc áo vừa thuộc danh mục "Áo nam" (danh mục gốc), vừa thuộc "Bộ sưu tập Thu Đông 2026" (danh mục chiến dịch).
* **Định danh nghiệp vụ:** Khóa phức hợp `(ProductID, CategoryID)`.
* **Thuộc tính ứng viên:** `ProductID`, `CategoryID`, `IsPrimary` (BIT).
* **Ràng buộc nghiệp vụ:** Mỗi `ProductID` chỉ được có **tối đa một dòng** có `IsPrimary = 1` để xác định Breadcrumb điều hướng và tránh tính trùng doanh thu khi lập báo cáo theo danh mục.

### 2.4. `ProductVariant` (Thực thể Trung tâm – SKU)
* **Ý nghĩa nghiệp vụ:** Đại diện cho **mặt hàng bán thực tế** mà khách hàng cầm nắm được (ví dụ: Áo thun Polo Nam - Đen - Size L). Đây là thực thể trung tâm gắn liền với mã vạch, giá niêm yết bán ra, quản lý tồn kho tại các kho và xuất hiện trong các đơn hàng.
* **Định danh nghiệp vụ:** `VariantID` (Surrogate Key) và `SKU` (Natural Unique Key toàn hệ thống).
* **Thuộc tính ứng viên:** `VariantID`, `ProductID`, `SKU`, `Barcode`, `Price`, `CostPrice`, `WeightGram`, `IsActive`, `CreatedAt`, `UpdatedAt`.
* **Ràng buộc nghiệp vụ:** `SKU` phải duy nhất toàn hệ thống (BR-PR02). Một sản phẩm có thể có nhiều SKU (BR-PR03). Không được tồn tại 2 SKU có cùng tập giá trị thuộc tính trong 1 sản phẩm (BR-PR12).

### 2.5. `Attribute` (Thực thể Mạnh – Từ điển Thuộc tính Toàn cục)
* **Ý nghĩa nghiệp vụ:** Danh mục các loại thuộc tính biến thể trên toàn hệ thống (ví dụ: `COLOR`, `SIZE`, `MATERIAL`, `CAPACITY`).
* **Định danh nghiệp vụ:** `AttributeID` và `Code` (Unique).
* **Thuộc tính ứng viên:** `AttributeID`, `Code`, `Name`, `DataType`.

### 2.6. `AttributeValue` (Thực thể Phụ thuộc vào `Attribute`)
* **Ý nghĩa nghiệp vụ:** Các giá trị hợp lệ cụ thể của một thuộc tính (ví dụ: Thuộc tính `COLOR` có các giá trị: `Đen`, `Trắng`, `Xanh navy`; Thuộc tính `SIZE` có: `S`, `M`, `L`, `XL`; Thuộc tính `MATERIAL` có: `Cotton 100%`, `Gỗ sồi tự nhiên`).
* **Định danh nghiệp vụ:** `ValueID`.
* **Thuộc tính ứng viên:** `ValueID`, `AttributeID`, `Value`, `DisplayOrder`.

### 2.7. `ProductAttribute` (Thực thể Liên kết – Quy tắc Sản phẩm hỗ trợ)
* **Ý nghĩa nghiệp vụ:** Khai báo một `Product` cụ thể được phép sử dụng những `Attribute` nào. Ví dụ: Áo thun Polo chỉ được gắn `COLOR` và `SIZE`, tuyệt đối không được gắn thuộc tính `CAPACITY` (Công suất).
* **Định danh nghiệp vụ:** Khóa phức hợp `(ProductID, AttributeID)`.
* **Thuộc tính ứng viên:** `ProductID`, `AttributeID`.

### 2.8. `VariantAttributeValue` (Thực thể Liên kết – Cấu hình SKU)
* **Ý nghĩa nghiệp vụ:** Gán tập giá trị thuộc tính cụ thể cấu thành nên một `ProductVariant`.
* **Định danh nghiệp vụ:** Khóa phức hợp `(VariantID, ValueID)`.
* **Thuộc tính ứng viên:** `VariantID`, `ValueID`.
* **Ràng buộc:** Một SKU không được gán 2 giá trị của cùng một `AttributeID` (ví dụ: 1 SKU không thể vừa là Size M vừa là Size L).

---

## 3. Chi tiết Miền 2: Warehouse & Inventory (Kho hàng & Quản lý Tồn kho 2 Bước)

### 3.1. `Warehouse` (Thực thể Mạnh)
* **Ý nghĩa nghiệp vụ:** Kho hàng vật lý của doanh nghiệp (ví dụ: Kho Tổng Tân Bình, Kho Phân phối Thủ Đức, Kho Quận 1).
* **Định danh nghiệp vụ:** `WarehouseID` (PK) và `WarehouseCode` (Unique).
* **Thuộc tính ứng viên:** `WarehouseID`, `WarehouseCode`, `Name`, `AddressLine`, `Ward`, `District`, `City`, `ManagerName`, `ContactPhone`, `IsActive`.

### 3.2. `InventoryBalance` (Thực thể Liên kết Cốt lõi – Số dư Tồn kho)
* **Ý nghĩa nghiệp vụ:** Đại diện cho trạng thái số dư tồn kho tức thời của một SKU tại một kho hàng vật lý nhất định.
* **Định danh nghiệp vụ:** Khóa phức hợp `(WarehouseID, VariantID)`.
* **Thuộc tính ứng viên:**
  * `WarehouseID` (FK)
  * `VariantID` (FK)
  * `OnHandQty` (Số lượng hàng thực tế đang có trên giá kệ kho)
  * `ReservedQty` (Số lượng hàng đang được giữ tạm thời cho các đơn hàng chưa xuất)
  * `UpdatedAt` (Thời điểm biến động gần nhất)
* **Ràng buộc bất biến:**
  * `OnHandQty >= 0`
  * `ReservedQty >= 0`
  * `ReservedQty <= OnHandQty`
  * $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$ (Thuộc tính dẫn xuất, luôn $\ge 0$).

### 3.3. `StockReservation` (Thực thể Cam kết Giữ kho – BD-04 & BR-IN13..16)
* **Ý nghĩa nghiệp vụ:** Bản ghi giữ hàng tạm thời cho đơn hàng ngay khi khách bấm Đặt hàng (Checkout). Có thời hạn cụ thể (`ExpiresAt` = 15 phút).
* **Định danh nghiệp vụ:** `ReservationID`.
* **Thuộc tính ứng viên:**
  * `ReservationID` (PK)
  * `AllocationID` (FK trỏ tới `OrderAllocation` của kho)
  * `WarehouseID` (FK)
  * `VariantID` (FK)
  * `ReservedQty` (Số lượng giữ)
  * `CreatedAt` (Thời điểm tạo giữ hàng)
  * `ExpiresAt` (Thời điểm hết hạn = CreatedAt + 15 phút)
  * `Status` (`ACTIVE`, `FULFILLED`, `RELEASED`, `EXPIRED`)
* **Vòng đời:**
  * `ACTIVE`: Đang giữ hàng tại kho, tính vào `InventoryBalance.ReservedQty`.
  * `FULFILLED`: Kho đã xuất hàng thành công (`SHIPMENT_OUT`), reservation tiêu thụ và kết thúc.
  * `RELEASED` / `EXPIRED`: Khách hủy đơn hoặc quá 15 phút không thanh toán, hệ thống giải phóng lượng giữ, hoàn trả về `AvailableQty`.

### 3.4. `StockMovement` (Thực thể Sổ cái Biến động Tồn kho Bất biến – Audit Trail)
* **Ý nghĩa nghiệp vụ:** Sổ nhật ký ghi lại **mọi sự kiện** làm thay đổi số lượng tồn kho vật lý tại kho. Dữ liệu mang tính bất biến (Append-only, tuyệt đối không UPDATE/DELETE).
* **Định danh nghiệp vụ:** `MovementID`.
* **Thuộc tính ứng viên:**
  * `MovementID` (PK)
  * `WarehouseID` (FK)
  * `VariantID` (FK)
  * `MovementType` (Loại biến động: `GOODS_RECEIPT`, `SHIPMENT_OUT`, `TRANSFER_OUT`, `TRANSFER_IN`, `STOCK_ADJUSTMENT`, `DELIVERY_RETURN_IN`)
  * `QuantityChanged` (Số lượng thay đổi: số dương nếu tăng tồn, số âm nếu giảm tồn)
  * `BalanceAfter` (Số dư tồn kho vật lý `OnHandQty` sau khi biến động xảy ra)
  * `ReferenceType` (Loại chứng từ nguồn: `PURCHASE_RECEIPT`, `SHIPMENT`, `TRANSFER_ORDER`, `DELIVERY_RETURN`)
  * `ReferenceID` (Mã chứng từ nguồn cụ thể)
  * `CreatedBy` (Người hoặc tiến trình thực hiện)
  * `CreatedAt` (Thời điểm phát sinh)

---

## 4. Chi tiết Miền 3: Cầu nối Phân bổ & Vận hành (Order Allocation & Split Shipment)

### 4.1. `OrderAllocation` (Thực thể Cốt lõi giải quyết Split Shipment)
* **Ý nghĩa nghiệp vụ:** Đại diện cho quyết định phân bổ số lượng từ một dòng đơn hàng (`OrderLine`) cho một kho hàng vật lý cụ thể.
* **Định danh nghiệp vụ:** `AllocationID`.
* **Thuộc tính ứng viên:**
  * `AllocationID` (PK)
  * `OrderLineID` (FK trỏ về `OrderLine`)
  * `WarehouseID` (FK trỏ về `Warehouse` được chỉ định lấy hàng)
  * `VariantID` (FK trỏ về `ProductVariant`)
  * `AllocatedQty` (Số lượng phân bổ)
  * `Status` (`ALLOCATED`, `RESERVED`, `PACKED`, `SHIPPED`, `CANCELLED`)
* **Ví dụ thực tế:**
  * Đơn hàng `ORD001` có `OrderLine 1`: 5 Áo Polo Đen Size L.
  * Phân bổ 1 (`Allocation 101`): `Warehouse = Kho Q1`, `AllocatedQty = 3`.
  * Phân bổ 2 (`Allocation 102`): `Warehouse = Kho Thủ Đức`, `AllocatedQty = 2`.
  * `Allocation 101` sẽ liên kết với `StockReservation` tại Kho Q1 và đóng vào `Shipment 01` (Kho Q1).
  * `Allocation 102` sẽ liên kết với `StockReservation` tại Kho Thủ Đức và đóng vào `Shipment 02` (Kho Thủ Đức).

### 4.2. `Shipment` & `ShipmentLine` (Kiện hàng Vận chuyển Vật lý)
* **`Shipment`:**
  * Đại diện cho một kiện hàng thực tế được đóng gói từ một kho xuất để giao cho khách hàng.
  * Thuộc tính: `ShipmentID`, `OrderID`, `WarehouseID` (Kho xuất), `ShippingProviderID`, `TrackingCode`, `Status` (`PREPARING`, `SHIPPED`, `DELIVERED`, `DELIVERY_FAILED`, `RETURNED_TO_WAREHOUSE`), `ShippedAt`, `DeliveredAt`.
* **`ShipmentLine`:**
  * Thuộc tính: `ShipmentLineID`, `ShipmentID`, `AllocationID`, `Quantity` (Số lượng thực tế xuất trong kiện này).

### 4.3. `DeliveryReturnReceipt` (Biên bản Tiếp nhận Hàng Giao Thất bại Quay về)
* **Ý nghĩa nghiệp vụ:** Chứng từ ghi nhận sự kiện hàng giao thất bại và **thực tế được chở quay trở lại kho tiếp nhận**.
* **Định danh nghiệp vụ:** `ReturnReceiptID`.
* **Thuộc tính ứng viên:** `ReturnReceiptID`, `ShipmentID`, `WarehouseID` (Kho tiếp nhận lại), `ReceivedQty`, `ConditionStatus` (Hàng nguyên vẹn / Hỏng hóc), `ReceivedBy`, `ReceivedAt`, `Notes`.
* **Tác động tồn kho:** Khi chứng từ này được ký xác nhận, hệ thống mới phát sinh `StockMovement` loại `DELIVERY_RETURN_IN` để tăng lại `OnHandQty` của kho.

---

## 5. Tổng kết Quan hệ Bền vững giữa Product Catalog, Warehouse và Fulfillment

```text
[Product] 1 ──── N [ProductVariant / SKU] 1 ──── N [InventoryBalance] N ──── 1 [Warehouse]
   │                      │                               ▲                      │
   │                      │                               │ (giữ kho)            │
   ▼                      ▼                               │                      ▼
[OrderLine] 1 ──── N [OrderAllocation] 1 ──────── 1 [StockReservation]    [Shipment]
                          │                                                      │
                          └────────────────────── 1 ──── N [ShipmentLine] ───────┘
```

