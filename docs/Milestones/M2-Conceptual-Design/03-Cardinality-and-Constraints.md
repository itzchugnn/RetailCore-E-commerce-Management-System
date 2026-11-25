# M2.03 - CARDINALITY & BUSINESS INTEGRITY CONSTRAINTS (BẢN SỐ & RÀNG BUỘC TOÀN VẸN NGHIỆP VỤ)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 2.0 – Canonical SSOT Standardized Specification
* **Hệ quản trị CSDL mục tiêu:** Microsoft SQL Server 2022+ (Developer / Enterprise Edition)
* **Kế thừa trực tiếp:**
  * Toàn bộ 55 Yêu cầu Chức năng (FR) và 215 Business Rules (`M1.06`) từ M1.
  * Quyết định nền tảng [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md) (BD-01 đến BD-07).
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (**79 Thực thể `E01` – `E79`**).
  * [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md) & [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md).
  * [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md) (10 Hiệu chỉnh kiến trúc chuẩn hóa Pre-M3).

---

## 1. PHƯƠNG PHÁP LUẬN BẢN SỐ & TÍNH THAM GIA (CROW'S FOOT NOTATION)

Trong thiết kế Cơ sở dữ liệu Quan niệm chuẩn quốc tế, mỗi mối quan hệ giữa hai thực thể được xác định bởi hai chiều:
1. **Bản số Tối đa (Maximum Cardinality / Cardinality Ratio):** Xác định một thể hiện của thực thể A có thể liên kết với tối đa bao nhiêu thể hiện của thực thể B ($1$ hoặc $N$).
2. **Bản số Tối thiểu (Minimum Cardinality / Participation Constraint):**
   * **Toàn phần (Total / Mandatory Participation - Bản số tối thiểu = $1$):** Mọi thể hiện của thực thể bắt buộc phải tham gia vào mối quan hệ. Không được phép tồn tại bản ghi mồ côi. Ký hiệu vạch đứng (`|`).
   * **Một phần (Partial / Optional Participation - Bản số tối thiểu = $0$):** Một thể hiện của thực thể có thể tồn tại độc lập mà không cần tham gia vào mối quan hệ tại thời điểm tạo lập. Ký hiệu vòng tròn (`o`).

### Bảng Ký hiệu Crow's Foot chuẩn hóa:
| Ký hiệu | Tên gọi | Bản số | Diễn giải |
| :---: | :--- | :---: | :--- |
| `|| ─── ||` | Exactly One to Exactly One | $1:1$ (Bắt buộc hai phía) | Quan hệ tương hỗ $1:1$ toàn phần. Ví dụ: `SalesOrder` $\leftrightarrow$ `OrderAddress`. |
| `|| ─── o|` | Exactly One to Zero or One | $1:0..1$ (Tùy chọn một phía) | Phụ thuộc tùy chọn. Ví dụ: `Account` $\leftrightarrow$ `Customer` (do ràng buộc XOR). |
| `|| ─── }|` | Exactly One to One or Many | $1:1..N$ (Bắt buộc cha - con) | Quan hệ thực thể mạnh – yếu bắt buộc có chi tiết. Ví dụ: `SalesOrder` $\leftrightarrow$ `OrderLine`. |
| `|| ─── }o` | Exactly One to Zero or Many | $1:0..N$ (Quan hệ chuẩn $1:N$) | Quan hệ cha con tiêu chuẩn. Ví dụ: `Warehouse` $\leftrightarrow$ `InventoryBalance`. |
| `o| ─── }o` | Zero or One to Zero or Many | $0..1:0..N$ (Tùy chọn cả hai đầu) | Khóa ngoại cho phép `NULL`. Ví dụ: `Account` $\leftrightarrow$ `AuditLog` (hành động hệ thống). |

---

## 2. MA TRẬN BẢN SỐ TOÀN DIỆN 79 THỰC THỂ (MASTER CARDINALITY MATRIX)

Dưới đây là ma trận chuẩn hóa toàn bộ các mối quan hệ nội bộ và liên miền của **79 thực thể (`E01` – `E79`)** qua 11 phân hệ:

### 2.1. Phân hệ 01: Product Catalog (`E01` – `E09`, `E73`)
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Brand` | `o| ─── }o` | `Product` | $0..1:0..N$ | Partial | Partial | Sản phẩm gắn thương hiệu chuẩn hóa `Brand` (E73); sản phẩm chưa gán thương hiệu để `BrandID IS NULL`. |
| `Product` | `|| ─── }o` | `ProductCategory` | $1:0..N$ | Partial | **Total** | SP nháp có thể chưa gán danh mục; SP Published bắt buộc có đúng 1 `IsPrimary = 1` (BD-01). |
| `Category` | `|| ─── }o` | `ProductCategory` | $1:0..N$ | Partial | **Total** | Danh mục có thể rỗng hoặc chứa nhiều sản phẩm. |
| `Category` (Parent) | `o| ─── }o` | `Category` (Child) | $0..1:0..N$ | Partial | Partial | Cây danh mục đệ quy cha-con; danh mục gốc có `ParentCategoryID IS NULL` (PC-Q02). |
| `Product` | `|| ─── }|` | `ProductVariant` | $1:1..N$ | **Total** | **Total** | Sản phẩm luôn có ít nhất 1 SKU (sản phẩm đơn có SKU mặc định, PC-Q03). Cấm SKU mồ côi. |
| `Product` | `|| ─── }o` | `ProductAttribute` | $1:0..N$ | Partial | **Total** | Sản phẩm khai báo danh sách thuộc tính áp dụng kèm vai trò `AttributeRole` (PC-Q04). |
| `Attribute` | `|| ─── }o` | `ProductAttribute` | $1:0..N$ | Partial | **Total** | Thuộc tính từ điển toàn cục được áp dụng cho nhiều sản phẩm (PC-Q05). |
| `Attribute` | `|| ─── }|` | `AttributeValue` | $1:1..N$ | Partial | **Total** | Thuộc tính có từ điển các giá trị hợp lệ. Giá trị bắt buộc thuộc 1 thuộc tính. |
| `ProductAttribute` | `|| ─── }o` | `ProductAttributeValue` | $1:0..N$ | Partial | **Total** | Whitelist các giá trị được phép chọn cho thuộc tính trên sản phẩm cụ thể. |
| `AttributeValue` | `|| ─── }o` | `ProductAttributeValue` | $1:0..N$ | Partial | **Total** | Giá trị từ điển được đưa vào whitelist của sản phẩm. |
| `ProductVariant` | `|| ─── }o` | `VariantAttributeValue` | $1:0..N$ | Partial | **Total** | Tổ hợp giá trị thuộc tính cấu thành SKU; SKU đơn giản có tổ hợp rỗng (PC-Q03). |
| `Attribute` & `AttributeValue` | `|| ─── }o` | `VariantAttributeValue` | $1:0..N$ | Partial | **Total** | Khóa ngoại phức hợp `(AttributeID, AttributeValueID)` bắt buộc thuộc tập Whitelist `ProductAttributeValue` (`BR-PR12`, `BR-PR14`). |

### 2.2. Phân hệ 02: Warehouse & Inventory
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Warehouse` | `|| ─── }o` | `InventoryBalance` | $1:0..N$ | Partial | **Total** | Kho lưu trữ số dư của nhiều SKU. Mỗi bản ghi số dư thuộc đúng 1 kho. |
| `ProductVariant` | `|| ─── }o` | `InventoryBalance` | $1:0..N$ | Partial | **Total** | Mỗi SKU có bản ghi số dư tại các kho có lưu hàng (WI-Q01). |
| `OrderLine` | `|| ─── }|` | `OrderWarehouseAllocation` | $1:1..N$ | **Total** | **Total** | Mỗi dòng đơn hàng bắt buộc phân bổ cho $\ge 1$ kho xuất (Split Shipment, BD-03). |
| `Warehouse` | `|| ─── }o` | `OrderWarehouseAllocation` | $1:0..N$ | Partial | **Total** | Kho xuất nhận cam kết kế hoạch xuất hàng. |
| `ProductVariant` | `|| ─── }o` | `OrderWarehouseAllocation` | $1:0..N$ | Partial | **Total** | Xác định chính xác SKU xuất kho. |
| `SalesOrder` | `|| ─── }o` | `StockReservation` | $1:0..N$ | Partial | **Total** | Đơn hàng tạo các chứng từ khóa giữ tồn kho logic (BD-04, WI-Q02). |
| `StockReservation` | `|| ─── }|` | `StockReservationLine` | $1:1..N$ | **Total** | **Total** | Phiếu giữ hàng bắt buộc có ít nhất 1 dòng chi tiết giữ hàng theo kho. |
| `OrderWarehouseAllocation` | `|| ─── }o` | `StockReservationLine` | $1:0..N$ | Partial | **Total** | Phân bổ kho có thể trải qua nhiều lượt giữ hàng lịch sử; tối đa 1 dòng giữ hàng `ACTIVE` qua Filtered Unique Index (CORR-10). |
| `Warehouse` | `|| ─── }o` | `StockReservationLine` | $1:0..N$ | Partial | **Total** | Xác định kho vật lý thực hiện giữ hàng. |
| `ProductVariant` | `|| ─── }o` | `StockReservationLine` | $1:0..N$ | Partial | **Total** | Xác định SKU bị khóa giữ. |
| `InventoryTransaction` | `|| ─── }|` | `InventoryTransactionLine` | $1:1..N$ | **Total** | **Total** | Chứng từ biến động kho bắt buộc có $\ge 1$ dòng chi tiết ghi sổ kế toán. |
| `Warehouse` | `|| ─── }o` | `InventoryTransactionLine` | $1:0..N$ | Partial | **Total** | Kho chịu ảnh hưởng thay đổi tồn vật lý hoặc tồn giữ. |
| `ProductVariant` | `|| ─── }o` | `InventoryTransactionLine` | $1:0..N$ | Partial | **Total** | SKU chịu ảnh hưởng thay đổi tồn kho (`OnHandDelta`, `ReservedDelta`). |
| `Warehouse` | `|| ─── }o` | `StockCount` | $1:0..N$ | Partial | **Total** | Phiếu kiểm kê được mở tại 1 kho vật lý (WI-Q05). |
| `StockCount` | `|| ─── }|` | `StockCountLine` | $1:1..N$ | **Total** | **Total** | Phiếu kiểm kê bắt buộc có $\ge 1$ dòng kiểm đếm thực tế SKU. |
| `ProductVariant` | `|| ─── }o` | `StockCountLine` | $1:0..N$ | Partial | **Total** | SKU được kiểm đếm thực tế tại kho. |
| `Warehouse` (Source) | `|| ─── }o` | `WarehouseTransfer` | $1:0..N$ | Partial | **Total** | Kho xuất chuyển đi trong giao dịch chuyển kho 2 bước (WI-Q04). |
| `Warehouse` (Destination) | `|| ─── }o` | `WarehouseTransfer` | $1:0..N$ | Partial | **Total** | Kho nhận trong giao dịch chuyển kho 2 bước. |
| `WarehouseTransfer` | `|| ─── }|` | `WarehouseTransferLine` | $1:1..N$ | **Total** | **Total** | Phiếu chuyển kho bắt buộc có $\ge 1$ dòng SKU luân chuyển. |
| `ProductVariant` | `|| ─── }o` | `WarehouseTransferLine` | $1:0..N$ | Partial | **Total** | SKU luân chuyển giữa hai kho. |

### 2.3. Phân hệ 03: Procurement
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Supplier` | `|| ─── }o` | `SupplierProduct` | $1:0..N$ | Partial | **Total** | NCC cung ứng nhiều SKU; mỗi liên kết thuộc đúng 1 NCC. |
| `ProductVariant` | `|| ─── }o` | `SupplierProduct` | $1:0..N$ | Partial | **Total** | Mỗi SKU có thể có nhiều nguồn cung ứng khác nhau. |
| `Supplier` | `|| ─── }o` | `PurchaseOrder` | $1:0..N$ | Partial | **Total** | Đơn đặt hàng mua gửi đến duy nhất 1 NCC. |
| `Warehouse` | `|| ─── }o` | `PurchaseOrder` | $1:0..N$ | Partial | **Total** | PO chỉ định duy nhất 1 kho nhận cố định cho toàn bộ đơn (PR-Q01). |
| `PurchaseOrder` | `|| ─── }|` | `PurchaseOrderLine` | $1:1..N$ | **Total** | **Total** | PO bắt buộc có $\ge 1$ dòng đặt mua; không cho phép PO rỗng. |
| `ProductVariant` | `|| ─── }o` | `PurchaseOrderLine` | $1:0..N$ | Partial | **Total** | Dòng đặt mua ghi nhận số lượng và đơn giá của 1 SKU. |
| `PurchaseOrder` | `|| ─── }o` | `GoodsReceipt` | $1:0..N$ | Partial | **Total** | Cho phép nhận hàng nhiều đợt (Partial Receipt) qua nhiều phiếu nhập (PR-Q02). |
| `Warehouse` | `|| ─── }o` | `GoodsReceipt` | $1:0..N$ | Partial | **Total** | Phiếu tiếp nhận tại kho; bắt buộc trùng kho nhận của PO (PR-Q01). |
| `GoodsReceipt` | `|| ─── }|` | `GoodsReceiptLine` | $1:1..N$ | **Total** | **Total** | Phiếu nhập kho bắt buộc có $\ge 1$ dòng hàng kiểm nhận thực tế. |
| `PurchaseOrderLine` | `|| ─── }o` | `GoodsReceiptLine` | $1:0..N$ | Partial | **Total** | Dòng nhập đối soát về đúng dòng PO đặt ban đầu (PR-Q03). |
| `ProductVariant` | `|| ─── }o` | `GoodsReceiptLine` | $1:0..N$ | Partial | **Total** | SKU thực tế được kiểm nhận tại cầu cảng kho. |
| `Supplier` | `|| ─── }o` | `SupplierReturn` | $1:0..N$ | Partial | **Total** | Phiếu trả hàng xuất gửi về nhà cung cấp (PR-Q05). |
| `Warehouse` | `|| ─── }o` | `SupplierReturn` | $1:0..N$ | Partial | **Total** | Kho vật lý thực hiện xuất trả hàng cho NCC. |
| `SupplierReturn` | `|| ─── }|` | `SupplierReturnLine` | $1:1..N$ | **Total** | **Total** | Phiếu trả hàng bắt buộc có $\ge 1$ dòng hàng trả. |
| `GoodsReceiptLine` | `|| ─── }o` | `SupplierReturnLine` | $1:0..N$ | Partial | **Total** | Trả hàng tham chiếu chứng từ tiếp nhận gốc để kiểm soát số lượng (PR-Q05). |
| `ProductVariant` | `|| ─── }o` | `SupplierReturnLine` | $1:0..N$ | Partial | **Total** | SKU thực tế xuất trả NCC. |

### 2.4. Phân hệ 04: Customer & Identity
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Account` | `|| ─── o|` | `Customer` | $1:0..1$ | Partial | **Total** | Tài khoản có tối đa 1 hồ sơ Customer (XOR với Employee, CID-Q01). |
| `Account` | `|| ─── o|` | `Employee` | $1:0..1$ | Partial | **Total** | Tài khoản có tối đa 1 hồ sơ Employee (XOR với Customer, CID-Q01). |
| `Customer` | `|| ─── }o` | `CustomerAddress` | $1:0..N$ | Partial | **Total** | Khách hàng có sổ địa chỉ nhận hàng; tối đa 1 địa chỉ mặc định (CID-Q06). |
| `Account` | `|| ─── }o` | `AccountRole` | $1:0..N$ | Partial | **Total** | Tài khoản được gán các vai trò RBAC (CID-Q02). |
| `Role` | `|| ─── }o` | `AccountRole` | $1:0..N$ | Partial | **Total** | Vai trò được cấp phát cho các tài khoản người dùng. |
| `Role` | `|| ─── }o` | `RolePermission` | $1:0..N$ | Partial | **Total** | Vai trò bao gồm tập các quyền hạn nghiệp vụ. |
| `Permission` | `|| ─── }o` | `RolePermission` | $1:0..N$ | Partial | **Total** | Quyền hạn tài nguyên được gán vào vai trò. |
| `Account` | `|| ─── }o` | `AccountVerification` | $1:0..N$ | Partial | **Total** | Lịch sử các lần gửi mã OTP xác minh email/sđt của tài khoản (CID-Q05). |

### 2.5. Phân hệ 05: Shopping
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Customer` | `|| ─── }o` | `ShoppingCart` | $1:0..N$ | Partial | **Total** | Khách hàng sở hữu giỏ hàng; tối đa đúng 1 giỏ `ACTIVE` tại một thời điểm (SHOP-Q01). |
| `ShoppingCart` | `|| ─── }o` | `CartItem` | $1:0..N$ | Partial | **Total** | Giỏ hàng chứa danh sách mặt hàng đã chọn; tự cộng dồn nếu trùng SKU (SHOP-Q02). |
| `ProductVariant` | `|| ─── }o` | `CartItem` | $1:0..N$ | Partial | **Total** | Dòng giỏ hàng liên kết trực tiếp với SKU (không giữ tồn, SHOP-Q04). |

### 2.6. Phân hệ 06: Sales
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Customer` | `|| ─── }o` | `SalesOrder` | $1:0..N$ | Partial | **Total** | Khách hàng đặt mua hàng; bắt buộc đăng nhập (BD-02, cấm Guest Checkout). |
| `ShoppingCart` | `o| ─── o|` | `SalesOrder` | $0..1:0..1$ | Partial | Partial | Mỗi giỏ hàng chuyển đổi tối đa 1 đơn hàng (`UX_SalesOrder_Cart WHERE CartID IS NOT NULL`); đơn đổi bù có `CartID IS NULL` (BR-SA03). |
| `SalesOrder` (Parent) | `o| ─── }o` | `SalesOrder` (Exchange) | $0..1:0..N$ | Partial | Partial | Đơn đổi bù RMA (`OrderType = 'EXCHANGE_REPLACEMENT'`) liên kết về đơn hàng gốc qua `ParentOrderID` (`CORR-01`). |
| `SalesOrder` | `|| ─── }|` | `OrderLine` | $1:1..N$ | **Total** | **Total** | Đơn hàng bắt buộc có $\ge 1$ dòng hàng; cấm đơn hàng rỗng (BR-SA04). |
| `ProductVariant` | `|| ─── }o` | `OrderLine` | $1:0..N$ | Partial | **Total** | SKU bán trong đơn; `OrderLine` snapshot đầy đủ để bảo toàn lịch sử (SALES-Q05). |
| `SalesOrder` | `|| ─── ||` | `OrderAddress` | $1:1$ | **Total** | **Total** | Mỗi đơn hàng bắt buộc có đúng 1 địa chỉ giao hàng snapshot (`OrderID` Unique, SALES-Q06). |
| `SalesOrder` | `|| ─── }|` | `OrderStatusHistory` | $1:1..N$ | **Total** | **Total** | Đơn hàng bắt buộc có $\ge 1$ bản ghi lịch sử trạng thái (bản ghi đầu `PENDING_PAYMENT`, SALES-Q02). |

### 2.7. Phân hệ 07: Fulfillment & Shipping
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `SalesOrder` | `|| ─── }o` | `Shipment` | $1:0..N$ | Partial | **Total** | Đơn hàng phân tách thành nhiều kiện giao xuất phát từ các kho (Split Shipment, BD-03). |
| `Warehouse` | `|| ─── }o` | `Shipment` | $1:0..N$ | Partial | **Total** | Mỗi kiện hàng chỉ xuất phát từ **duy nhất 1 kho vật lý** (`WarehouseID NOT NULL`, SHIP-Q01). |
| `Carrier` | `|| ─── }o` | `Shipment` | $1:0..N$ | Partial | **Total** | Kiện hàng được vận chuyển bởi 1 đơn vị giao nhận đối tác (SHIP-Q03). |
| `Shipment` | `|| ─── }|` | `ShipmentLine` | $1:1..N$ | **Total** | **Total** | Kiện hàng bắt buộc chứa $\ge 1$ dòng sản phẩm đóng gói (BR-FS05). |
| `OrderLine` | `|| ─── }o` | `ShipmentLine` | $1:0..N$ | Partial | **Total** | Dòng sản phẩm trong đơn được đưa vào kiện hàng (BR-FS08). |
| `OrderWarehouseAllocation` | `|| ─── }o` | `ShipmentLine` | $1:0..N$ | Partial | **Total** | Dòng kiện hàng tiêu thụ phân bổ của kho xuất tương ứng (BR-FS06, BR-FS07). |
| `Shipment` | `|| ─── }o` | `DeliveryAttempt` | $1:0..N$ | Partial | **Total** | Lịch sử các lần shipper thực hiện giao hàng đến khách (tối đa 3 lần, SHIP-Q04). |
| `Shipment` | `|| ─── }|` | `ShipmentStatusHistory` | $1:1..N$ | **Total** | **Total** | Kiện hàng bắt buộc có $\ge 1$ bản ghi nhật ký trạng thái (bản ghi đầu `PREPARING`, SHIP-Q06). |
| `Shipment` | `|| ─── o|` | `DeliveryReturnReceipt` | $1:0..1$ | Partial | **Total** | Kiện hàng giao thất bại hoàn kho được lập đúng 1 biên bản thực nhận `DeliveryReturnReceipt` (E75, BD-07). |
| `Warehouse` | `|| ─── }o` | `DeliveryReturnReceipt` | $1:0..N$ | Partial | **Total** | Kho vật lý tiếp nhận kiện hàng giao thất bại quay về. |

### 2.8. Phân hệ 08: Promotion & Voucher
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Promotion` | `|| ─── }o` | `PromotionCondition` | $1:0..N$ | Partial | **Total** | Chương trình khai báo các điều kiện chuẩn hóa cấu hình sẵn (PROMO-Q02). |
| `Promotion` | `|| ─── }o` | `PromotionProduct` | $1:0..N$ | Partial | **Total** | Danh sách SKU áp dụng ưu đãi cấp sản phẩm (`Scope = 'PRODUCT'`, PROMO-Q01). |
| `ProductVariant` | `|| ─── }o` | `PromotionProduct` | $1:0..N$ | Partial | **Total** | SKU được thụ hưởng chiết khấu của chương trình khuyến mãi. |
| `Promotion` | `|| ─── }o` | `Voucher` | $1:0..N$ | Partial | **Total** | Chương trình cấp đơn hàng (`'ORDER'`) hoặc Freeship (`'SHIPPING'`) phát hành mã voucher (PROMO-Q03). |
| `Customer` | `o| ─── }o` | `Voucher` | $0..1:0..N$ | Partial | Partial | Voucher cá nhân hóa trỏ đến khách hàng cụ thể; voucher dùng chung có `AssignedCustomerID IS NULL`. |
| `Promotion` | `|| ─── }o` | `PromotionTargetCustomer` | $1:0..N$ | Partial | **Total** | Danh sách khách hàng VIP được hưởng chương trình riêng tư (PROMO-Q07). |
| `Customer` | `|| ─── }o` | `PromotionTargetCustomer` | $1:0..N$ | Partial | **Total** | Khách hàng thuộc nhóm đối tượng của chương trình ưu đãi đặc quyền. |
| `Promotion` | `|| ─── }o` | `PromotionUsage` | $1:0..N$ | Partial | **Total** | Nhật ký tiêu thụ ưu đãi thực tế của chương trình (PROMO-Q05). |
| `Voucher` | `o| ─── }o` | `PromotionUsage` | $0..1:0..N$ | Partial | Partial | Ghi nhận mã voucher được sử dụng khi thanh toán đơn hàng (BD-05). |
| `SalesOrder` | `|| ─── }o` | `PromotionUsage` | $1:0..N$ | Partial | **Total** | Đơn hàng ghi nhận ưu đãi (tối đa 1 voucher `'ORDER'` + 1 voucher `'SHIPPING'` + các ưu đãi `'PRODUCT'`, BD-05). |
| `OrderLine` | `o| ─── }o` | `PromotionUsage` | $0..1:0..N$ | Partial | Partial | Dòng đơn hàng ghi nhận ưu đãi chiết khấu cấp SKU (tối đa 1 ưu đãi, BD-05). |

### 2.9. Phân hệ 09: Payment & Refund
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `SalesOrder` | `|| ─── ||` | `Payment` | $1:1$ | **Total** | **Total** | Mỗi đơn hàng bắt buộc có đúng 1 hồ sơ nghĩa vụ thanh toán (`Payment.OrderID` UNIQUE, PAY-Q01). |
| `PaymentMethod` | `|| ─── }o` | `Payment` | $1:0..N$ | Partial | **Total** | Phương thức thanh toán được chọn khi checkout đơn hàng (PAY-Q01). |
| `Payment` | `|| ─── }o` | `PaymentTransaction` | $1:0..N$ | Partial | **Total** | Nhật ký các lần gọi cổng thanh toán/thu tiền COD (kể cả lần thử lỗi, PAY-Q03). |
| `Payment` | `|| ─── }o` | `Refund` | $1:0..N$ | Partial | **Total** | Hồ sơ yêu cầu hoàn tiền gắn với khoản thanh toán đã thực thu (PAY-Q04). |
| `Refund` | `|| ─── }o` | `RefundTransaction` | $1:0..N$ | Partial | **Total** | Giao dịch chuyển khoản thực tế chi trả tiền hoàn lại cho khách (PAY-Q06). |
| `PaymentMethod` | `|| ─── }o` | `PaymentWebhookLog` | $1:0..N$ | Partial | **Total** | Bảng đệm webhook lưu payload thô và chống replay qua `(PaymentMethodID, ProviderEventID, Environment)` (E71, CORR-05). |
| `Account` | `|| ─── }o` | `IdempotencyRecord` | $1:0..N$ | Partial | **Total** | Khóa chống gửi lặp API đặt đơn & thanh toán theo `(AccountID, OperationType, IdempotencyKey)` (E74, CORR-05). |
| `PaymentTransaction` | `|| ─── }o` | `RefundTransaction` | $1:0..N$ | Partial | **Total** | Giao dịch hoàn tiền tham chiếu trực tiếp giao dịch thu tiền gốc qua `OriginalPaymentTransactionID` (CORR-08). |
| `Shipment` | `|| ─── o|` | `CODCollection` | $1:0..1$ | Partial | **Total** | Ghi nhận tiền mặt bưu tá thực thu trên từng kiện giao COD (E77, CORR-07). |
| `Payment` | `|| ─── }o` | `CODCollection` | $1:0..N$ | Partial | **Total** | Hồ sơ thanh toán COD tổng hợp các khoản thu theo từng kiện Split Shipment. |
| `Carrier` | `|| ─── }o` | `CODCollection` | $1:0..N$ | Partial | **Total** | Hãng vận chuyển chịu trách nhiệm quản lý khoản tiền thu hộ trên kiện hàng (`CR-46`, CORR-07). |
| `Carrier` | `|| ─── }o` | `CODRemittance` | $1:0..N$ | Partial | **Total** | Bảng kê chuyển trả tiền thu hộ COD của hãng vận chuyển (E78, CORR-07). |
| `CODRemittance` | `|| ─── }|` | `CODReconciliation` | $1:1..N$ | **Total** | **Total** | Dòng đối soát khớp tiền thu hộ kiện hàng vào bảng kê chuyển khoản (E79, CORR-07). |
| `CODCollection` | `|| ─── o|` | `CODReconciliation` | $1:0..1$ | Partial | **Total** | Mỗi khoản thu COD của kiện hàng được đối soát quyết toán tối đa 1 lần. |
| `PaymentTransaction` | `o| ─── }o` | `CODReconciliation` | $0..1:0..N$ | Partial | Partial | Khi đối soát thành công, dòng quyết toán liên kết với giao dịch `PaymentTransaction` loại `'COD_SETTLEMENT'`. |

### 2.10. Phân hệ 10: Return & Exchange (RMA)
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `SalesOrder` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Đơn hàng đã giao thành công có thể phát sinh nhiều yêu cầu đổi trả từng phần (RET-Q01). |
| `Customer` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Khách hàng là chủ sở hữu đơn gửi yêu cầu đổi trả (RET-Q01). |
| `Account` (CreatedBy) | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Tài khoản khách tự tạo hoặc CSKH tạo thay trên hệ thống (RET-Q01). |
| `Warehouse` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Yêu cầu chỉ định duy nhất 1 kho vật lý tiếp nhận hàng trả (`WarehouseID NOT NULL`, RET-Q06). |
| `ReturnRequest` | `|| ─── }|` | `ReturnRequestLine` | $1:1..N$ | **Total** | **Total** | Yêu cầu RMA bắt buộc có $\ge 1$ dòng hàng yêu cầu trả (BR-RT05). |
| `OrderLine` | `|| ─── }o` | `ReturnRequestLine` | $1:0..N$ | Partial | **Total** | Dòng sản phẩm trong đơn được yêu cầu trả lại (RET-Q03). |
| `ReturnRequestLine` | `|| ─── }|` | `ReturnShipmentAllocation` | $1:1..N$ | **Total** | **Total** | Phân bổ số lượng trả về đúng kiện giao `ShipmentLine` để kiểm tra hạn đổi trả theo kiện (E76, CORR-10). |
| `ShipmentLine` | `|| ─── }o` | `ReturnShipmentAllocation` | $1:0..N$ | Partial | **Total** | Dòng kiện giao hàng được đối chiếu hạn mức số lượng trả lại. |
| `ReturnRequest` | `|| ─── }|` | `ReturnStatusHistory` | $1:1..N$ | **Total** | **Total** | RMA bắt buộc có $\ge 1$ bản ghi lịch sử trạng thái (bản ghi đầu `PENDING_APPROVAL`, RET-Q08). |
| `Account` (ChangedBy) | `|| ─── }o` | `ReturnStatusHistory` | $1:0..N$ | Partial | **Total** | Tài khoản người dùng chuyển đổi trạng thái của yêu cầu RMA. |
| `ReturnRequest` | `|| ─── }o` | `ReturnReceipt` | $1:0..N$ | Partial | **Total** | Yêu cầu RMA có thể được tiếp nhận qua nhiều đợt/phiếu nhận tại kho (BR-RT13). |
| `Warehouse` | `|| ─── }o` | `ReturnReceipt` | $1:0..N$ | Partial | **Total** | Phiếu tiếp nhận tại kho; bắt buộc trùng kho chỉ định trên RMA (RET-Q06). |
| `ReturnReceipt` | `|| ─── }|` | `ReturnReceiptLine` | $1:1..N$ | **Total** | **Total** | Phiếu nhập trả bắt buộc có $\ge 1$ dòng kiểm định chất lượng (BR-RT14). |
| `ReturnRequestLine` | `|| ─── }o` | `ReturnReceiptLine` | $1:0..N$ | Partial | **Total** | Dòng kiểm định đối soát về đúng dòng yêu cầu RMA gốc (RET-Q05). |
| `ReturnReceiptLine` | `|| ─── }|` | `ReturnLineAssessment` | $1:1..N$ | **Total** | **Total** | Dòng tiếp nhận hàng trả phát sinh các đợt đánh giá lỗi và xét duyệt hoàn tiền (E72, CORR-01). |
| `ReturnLineAssessment` | `o| ─── o|` | `Refund` | $0..1:0..1$ | Partial | Partial | Đánh giá KCS được duyệt liên kết chứng từ hoàn tiền; hoàn tiền do hủy đơn trả trước trước giao có `ReturnLineAssessmentID IS NULL` (CORR-01). |

### 2.11. Phân hệ 11: Audit & Reporting
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Account` | `o| ─── }o` | `AuditLog` | $0..1:0..N$ | Partial | Partial | Nhật ký kiểm toán; thao tác người dùng có `AccountID`, sự kiện nền để NULL (AR-Q01). |
| `AuditLog` | `|| ─── }o` | `AuditLogDetail` | $1:0..N$ | Partial | **Total** | Chi tiết các thuộc tính bị thay đổi giá trị trước/sau (Old/New Value, AR-Q02). |
| `ReportDefinition` | `|| ─── }o` | `ReportExecution` | $1:0..N$ | Partial | **Total** | Mẫu báo cáo được người dùng thực thi chạy nhiều lần (AR-Q06). |
| `Account` | `|| ─── }o` | `ReportExecution` | $1:0..N$ | Partial | **Total** | Tài khoản nhân viên yêu cầu chạy hoặc xuất file báo cáo (AR-Q06). |
| `ReportDefinition` | `|| ─── }o` | `ReportSnapshot` | $1:0..N$ | Partial | **Total** | Mẫu báo cáo có các bản chụp số liệu chốt kỳ kế toán cố định (AR-Q05). |
| `Account` | `o| ─── }o` | `ReportSnapshot` | $0..1:0..N$ | Partial | Partial | Tài khoản hoặc job tự động tạo bản chụp chốt kỳ số liệu. |
| `ReportDefinition` | `|| ─── }o` | `ReportPermission` | $1:0..N$ | Partial | **Total** | Báo cáo được phân quyền cho các vai trò trong hệ thống (AR-Q07). |
| `Role` | `|| ─── }o` | `ReportPermission` | $1:0..N$ | Partial | **Total** | Vai trò được cấp quyền xem/xuất/chốt snapshot trên mẫu báo cáo. |

---

## 3. CÁC RÀNG BUỘC NGHIỆP VỤ BẬC CAO (HIGH-LEVEL BUSINESS INTEGRITY CONSTRAINTS)

Để bảo đảm tính toàn vẹn tuyệt đối của hệ thống Cơ sở dữ liệu Bán lẻ Doanh nghiệp, các ràng buộc nghiệp vụ dưới đây được định nghĩa chặt chẽ và sẽ được triển khai thông qua Declarative Constraints (Check/Unique), Triggers hoặc Stored Procedures trong SQL Server:

### 3.1. Ràng buộc Cấu trúc & Chuyên biệt hóa (Structural & Specialization Constraints)
1. **Ràng buộc loại trừ định danh (XOR Disjoint Subtype, CID-Q01, BR-ID04):**
   * Một `Account` chỉ được liên kết với một loại hồ sơ nghiệp vụ duy nhất: hoặc `Customer`, hoặc `Employee`.
   * $$\forall a \in \text{Account}: \neg (\exists c \in \text{Customer}: c.\text{AccountID} = a.\text{AccountID} \land \exists e \in \text{Employee}: e.\text{AccountID} = a.\text{AccountID})$$
   * *Cài đặt Logical/Physical:* Sử dụng Trigger toàn vẹn hoặc ràng buộc Check kiểm tra loại tài khoản kết hợp cờ `AccountType`.
2. **Cây danh mục đệ quy không chu trình (Acyclic Recursive Tree, PC-Q02):**
   * Quan hệ đệ quy `Category.ParentCategoryID REFERENCES Category(CategoryID)` tuyệt đối không được tạo chu trình lặp (Cyclic Reference) như $A \rightarrow B \rightarrow C \rightarrow A$.
   * Một danh mục không thể tự làm cha của chính nó: `ParentCategoryID <> CategoryID`.
3. **Whitelist thuộc tính biến thể (Attribute Whitelist Enforcement, PC-Q04, BR-PR12):**
   * Giá trị thuộc tính cấu thành biến thể (`VariantAttributeValue`) bắt buộc phải tồn tại trong tập whitelist `ProductAttributeValue` của sản phẩm sở hữu biến thể đó.
   * Variant không được phép chứa 2 giá trị khác nhau cho cùng một `AttributeID`.

### 3.2. Ràng buộc Bất biến Trạng thái & Máy Trạng thái Độc lập (State Invariants)
1. **Phân lập 4 máy trạng thái giao dịch (SALES-Q02, PAY-Q08, SHIP-Q06, CORR-06):**
   * Tuyệt đối độc lập giữa `SalesOrder.OrderStatus`, `Payment.PaymentStatus`, `Shipment.ShipmentStatus` và `ReturnRequest.ReturnStatus` (đồng thời tách `Payment.PaymentStatus` khỏi từng lần thử `PaymentTransaction.TransactionStatus`).
   * Đơn hàng COD hợp lệ: Chuyển `OrderStatus = 'CONFIRMED'` ngay cả khi `PaymentStatus = 'PENDING'`.
   * Đơn hàng thanh toán trực tuyến: Chỉ chuyển `OrderStatus = 'CONFIRMED'` khi có giao dịch `PaymentTransaction` thành công (`SUCCEEDED`) và `PaymentStatus = 'PAID'`.
2. **Máy trạng thái đơn hàng tuần tiến (`SalesOrder.OrderStatus`, CORR-06):**
   * Luồng chuyển trạng thái hợp lệ:
     $$\text{PENDING\_PAYMENT} \longrightarrow \text{CONFIRMED} \longrightarrow \text{PROCESSING} \longrightarrow \text{PARTIALLY\_SHIPPED} \longrightarrow \text{SHIPPED} \longrightarrow \text{PARTIALLY\_DELIVERED} \longrightarrow \text{DELIVERED}$$
   * Nhánh hủy: `PARTIALLY_CANCELLED` (hủy phần chưa giao), `CANCELLED` (hủy toàn bộ trước khi xuất giao); đóng hồ sơ hành chính qua `ClosedAt`.
3. **Máy trạng thái kiện hàng & Xử lý giao thất bại (`Shipment.ShipmentStatus`, CORR-06):**
   * Luồng: $\text{PREPARING} \longrightarrow \text{PACKED} \longrightarrow \text{DISPATCHED} \longrightarrow \text{IN\_TRANSIT} \longrightarrow \text{OUT\_FOR\_DELIVERY} \longrightarrow \text{DELIVERED}$.
   * Nhánh thất bại: Ghi nhận `DeliveryAttempt` (tối đa 3 lần `AttemptNumber`) $\longrightarrow$ Vượt quá chuyển sang $\text{RETURNED\_TO\_SENDER} \longrightarrow \text{RETURN\_RECEIVED\_AT\_WAREHOUSE}$ (ký `DeliveryReturnReceipt` E75).

### 3.3. Ràng buộc Mô hình Tồn kho Đa Trạng thái & Kế toán Kho (Inventory Quality & Accounting Constraints)
1. **Công thức tồn khả dụng và nguyên tắc All-or-Nothing (WI-Q02, BD-04, SALES-Q01, CORR-02):**
   $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$$
   * Giao dịch checkout đơn hàng là nguyên tử: Nếu bất kỳ SKU nào trong đơn không đáp ứng đủ $\text{OrderedQty} \le \text{AvailableQty}$ tại các kho phân bổ, toàn bộ transaction bị hủy bỏ (Rollback), giỏ hàng giữ nguyên `ACTIVE`.
2. **Bộ 5 Ràng buộc toàn vẹn số dư tồn kho (5 CHECK Constraints trên `InventoryBalance`, CORR-02):**
   * Ràng buộc không âm: `CHECK (OnHandQty >= 0)`, `CHECK (ReservedQty >= 0)`, `CHECK (QuarantinedQty >= 0)`, `CHECK (DamagedQty >= 0)`.
   * Ràng buộc tổng dung lượng tồn vật lý: `CHECK (OnHandQty >= ReservedQty + QuarantinedQty + DamagedQty)`.
   * Chuyển đổi trạng thái phẩm chất kho được ghi sổ qua `InventoryTransactionLine` với các trường `FromStockStatus`, `ToStockStatus` và bộ 4 delta (`OnHandDelta`, `ReservedDelta`, `QuarantinedDelta`, `DamagedDelta`).
3. **Sổ cái biến động kho đa chiều (Multi-Delta Ledger, WI-Q02, CORR-02):**
   * Mọi sự kiện làm thay đổi tồn kho đều ghi nhận vào `InventoryTransactionLine`.
   * Chứng từ sau khi `PostedAt` là bất biến (Immutable), không được phép sửa đổi hoặc xóa vật lý.
4. **Cô lập hàng In-Transit trong chuyển kho hai bước (WI-Q04):**
   * Bước 1 xuất kho nguồn (`DISPATCHED`): Giảm `OnHandQty` tại kho nguồn. Hàng trong quá trình vận chuyển không thuộc `OnHandQty` của bất kỳ kho nào.
   * Bước 2 nhập kho đích (`RECEIVED`): Tăng `OnHandQty` tại kho đích đúng bằng số lượng thực nhận `ReceivedQty`. Chênh lệch `VarianceQty` ghi nhận xử lý riêng.
5. **Cô lập hàng hoàn khi giao thất bại (BD-07, Principle C):**
   * Shipper báo giao thất bại qua `DeliveryAttempt` (tối đa 3 lần, sau đó chuyển `ShipmentStatus = 'RETURNED_TO_SENDER'`): **Tuyệt đối không tăng tồn kho.** Hàng vẫn đang nằm trên xe hoặc bưu cục.
   * Chỉ khi thủ kho nhận kiện hàng tại bến nhập và ký biên bản `DeliveryReturnReceipt` (E75, `POSTED`) sinh giao dịch loại `'DELIVERY_RETURN_IN'`, hệ thống mới ghi sổ kế toán và tăng lại `OnHandQty`.
6. **Phân tách trách nhiệm kiểm kê kho (Separation of Duties, WI-Q05, WI-Q06):**
   * Phiếu kiểm kê hoàn tất đếm (`CountStatus = 'COMPLETED'`) không tự động điều chỉnh số dư.
   * Bắt buộc có quản lý phê duyệt (`ApprovedBy NOT NULL`, `ApprovedAt NOT NULL`).
   * Ràng buộc kiểm soát nội bộ: Người kiểm đếm không được tự duyệt phiếu của chính mình (`ApprovedBy <> CreatedBy`).

### 3.4. Ràng buộc Số lượng, Tài chính & Hoàn tiền (Quantitative & Financial Integrity)
1. **Bảo toàn số lượng phân bổ Split Shipment (BD-03, WI-Q03):**
   $$\forall \text{OrderLine}: \sum_{\text{Allocation}} \text{AllocatedQty} = \text{OrderLine.OrderedQty} - \text{OrderLine.CancelledQty}$$
2. **Kiểm soát đóng gói kiện hàng không vượt quá đặt mua còn hiệu lực (SHIP-Q01, BR-FS08):**
   $$\forall \text{OrderLine}: \sum_{\text{ShipmentLine}} \text{ShipmentQty} \le \text{OrderLine.OrderedQty} - \text{OrderLine.CancelledQty}$$
   * Kiện hàng `Shipment` chỉ được đóng gói từ các dòng phân bổ `OrderWarehouseAllocation` có cùng `WarehouseID`.
3. **Bảo toàn số lượng kiểm nhận hàng mua (PR-Q03, PR-Q04):**
   * Trên từng dòng tiếp nhận: $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$.
   * Lũy kế nhận qua các đợt không vượt quá số lượng đặt mua:
     $$\forall \text{POLine}: \sum_{\text{GoodsReceiptLine}} \text{ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$$
4. **Hạn mức hoàn tiền tối đa (Financial Refund Ceiling, PAY-Q04, BR-PY22):**
   $$\sum \text{RefundAmount} \le \text{RemainingRefundable} = \text{AmountPaid} - \text{AmountRefunded}$$
   * Chỉ các yêu cầu `Refund` đã được duyệt (`ApprovedBy NOT NULL`) và có giao dịch `RefundTransaction` thành công (`SUCCEEDED`) mới được cập nhật tăng `Payment.AmountRefunded`.
5. **Hạn mức đổi trả hàng RMA và Tách rời Hoàn tiền khỏi Tồn kho (RET-Q03, BR-RT08, CORR-01, CORR-02):**
   $$\forall \text{OrderLine}: \sum_{\text{RMA}} \text{ApprovedQty} \le \text{OrderLine.OrderedQty} - \text{OrderLine.CancelledQty} - \text{AlreadyReturnedQty}$$
   * Phân định độc lập giữa xử lý tồn kho và xét duyệt hoàn tiền: Khi bến nhập chốt `ReturnReceipt` (E62), hàng được ghi `RETURN_RECEIPT_IN` vào bể `QUARANTINED`; sau đó `ReturnLineAssessment` (E72) định đoạt `InventoryDisposition` (`'SELLABLE'` giải phóng về tồn bán, `'QUARANTINED'` giữ cách ly, hoặc `'DISPOSED'` xuất hủy qua `'SCRAP_OUT'`). Quyền hoàn tiền (`RefundEligibilityStatus`) và phương án (`ResolutionType`) xét duyệt độc lập theo `FaultParty`, tính tiền hoàn theo công thức hiệu lũy tiến $\Delta F = F(a+b) - F(a)$ (kèm Clawback Voucher khi lỗi khách vi phạm `MinOrderAmount`).

### 3.5. Ràng buộc Khuyến mãi Ba Tầng & Kết hợp Mã (Three-Tier Promotion Constraints)
1. **Cấu trúc phân tầng khuyến mãi nghiêm ngặt (BD-05, PROMO-Q01..08):**
   * **Tầng 1 (`'PRODUCT'`):** Tối đa 1 khuyến mãi áp dụng trên mỗi dòng `OrderLine` (kèm trần `FlashSaleStockLimit` / `MaxQtyPerCustomer`).
   * **Tầng 2 (`'ORDER'`):** Tối đa 1 voucher tiền hàng áp dụng trên mỗi đơn `SalesOrder` (phân bổ LRM).
   * **Tầng 3 (`'SHIPPING'`):** Tối đa 1 voucher Freeship áp dụng giảm phí vận chuyển `ShippingDiscountAmount <= ShippingFee` trên mỗi đơn `SalesOrder`.
   * **Quy tắc Stacking:** Các tầng khác nhau chỉ được áp dụng đồng thời khi chương trình có cờ `IsStackable = 1`; cấm áp dụng quá 1 chương trình trong cùng tầng.
2. **Kiểm soát hạn mức sử dụng hai cấp (PROMO-Q04, BR-PRM12, BR-PRM13):**
   * Toàn sàn: `PendingCount + UsedCount <= MaxUsage`.
   * Theo khách hàng: $\text{Count}(\text{PromotionUsage where CustomerID}) \le \text{MaxUsagePerCustomer}$.
3. **Vòng đời tiêu thụ ưu đãi (PROMO-Q05, PROMO-Q06):**
   * Tạo đơn trả trước: Tạo bản ghi `PromotionUsage` ở trạng thái `PENDING` (khóa quota tạm `PendingCount`).
   * Thanh toán thành công (hoặc đơn COD xác nhận): Chuyển `CONSUMED` và tăng chính thức `UsedCount`.
   * Hủy đơn/Hết hạn: Chuyển `REVERSED` và hoàn trả quota nếu còn hạn hiệu lực.

### 3.6. Ràng buộc Bất biến Lịch sử & Cấm Xóa Cứng (Snapshot & Immutability)
1. **Snapshot bất biến dòng đơn hàng & giá vốn xuất kho (SALES-Q04, SALES-Q05, CORR-08, CORR-09):**
   * `OrderLine` bắt buộc lưu bản sao bất biến của: `SKU`, `ProductName`, `VariantName`, `UnitPrice` (đã gồm GTGT), `VatRate`, `TaxAmount`, và mảng JSON `VariantAttributesSnapshot`; `ShipmentLine` lưu cứng `OutboundUnitCost`.
   * Mọi biến động giá bán, tên gọi hay thay đổi thuộc tính trong Catalog sau thời điểm đặt hàng đều không ảnh hưởng đến đơn hàng cũ.
2. **Snapshot bất biến địa chỉ giao hàng & hóa đơn B2B (SALES-Q06):**
   * `OrderAddress` là một bản ghi độc lập với `CustomerAddress`, sao chép nguyên văn người nhận, số điện thoại, địa chỉ và thông tin pháp nhân xuất hóa đơn B2B tùy chọn tại thời điểm đặt.
   * Khách hàng sửa hoặc xóa địa chỉ trong sổ địa chỉ cá nhân không làm thay đổi địa chỉ của đơn hàng đã phát sinh.
3. **Nhật ký chỉ ghi (Append-Only / WORM) & Cấm xóa vật lý (No Hard Delete):**
   * Cấm tuyệt đối lệnh `DELETE` vật lý trên toàn bộ các bảng: `SalesOrder`, `OrderLine`, `Payment`, `PaymentTransaction`, `Shipment`, `ReturnRequest`, `InventoryTransaction`, `AuditLog`.
   * Mọi thao tác hủy đều thể hiện qua cập nhật trạng thái `CANCELLED` hoặc ghi nhận giao dịch đảo ngược (Reverse Transaction).

---

## 4. CHIẾN LƯỢC CÀI ĐẶT FILTERED UNIQUE INDEXES TRÊN SQL SERVER

Để hiện thực hóa các ràng buộc nghiệp vụ có điều kiện một cách tối ưu hiệu năng và an toàn dữ liệu, các chỉ mục duy nhất có lọc (Filtered Unique Indexes) sau đây được quy định chuẩn hóa:

| Bảng nghiệp vụ | Tên Index dự kiến | Cột lập chỉ mục (Key Columns) | Điều kiện lọc (WHERE Filter) | Ý nghĩa nghiệp vụ bảo vệ |
| :--- | :--- | :--- | :--- | :--- |
| `ProductCategory` | `UX_ProductCategory_Primary` | `(ProductID)` | `WHERE IsPrimary = 1` | Mỗi sản phẩm có tối đa đúng một danh mục chính khi hiển thị (PC-Q01, BD-01). |
| `Account` | `UX_Account_Email` | `(Email)` | `WHERE Email IS NOT NULL` | Email duy nhất toàn hệ thống nhưng cho phép tài khoản đăng ký bằng Phone (CID-Q03). |
| `Account` | `UX_Account_Phone` | `(Phone)` | `WHERE Phone IS NOT NULL` | Phone duy nhất toàn hệ thống nhưng cho phép tài khoản đăng ký bằng Email (CID-Q03). |
| `CustomerAddress` | `UX_CustomerAddress_Default` | `(CustomerID)` | `WHERE IsDefault = 1` | Mỗi khách hàng có tối đa đúng một địa chỉ mặc định trong sổ địa chỉ (CID-Q06). |
| `ShoppingCart` | `UX_ShoppingCart_Active` | `(CustomerID)` | `WHERE CartStatus = 'ACTIVE'` | Mỗi khách hàng có tối đa đúng một giỏ hàng đang hoạt động tại một thời điểm (SHOP-Q01). |
| `Shipment` | `UX_Shipment_TrackingNumber` | `(CarrierID, TrackingNumber)` | `WHERE TrackingNumber IS NOT NULL` | Mã vận đơn của đối tác logistics là duy nhất trong cùng một đơn vị vận chuyển. |
| `Payment` | `UQ_Payment_Order` | `(OrderID)` | *(Toàn bảng)* | Mỗi đơn hàng có đúng một hồ sơ thanh toán duy nhất ($1:1$, PAY-Q01). |
| `OrderAddress` | `UQ_OrderAddress_Order` | `(OrderID)` | *(Toàn bảng)* | Mỗi đơn hàng có đúng một địa chỉ giao hàng snapshot duy nhất ($1:1$, SALES-Q06). |
| `Voucher` | `UQ_Voucher_Code` | `(VoucherCode)` | *(Toàn bảng)* | Mã code voucher khuyến mãi là duy nhất toàn hệ thống. |
| `ProductVariant` | `UQ_ProductVariant_SKU` | `(SKU)` | *(Toàn bảng)* | Mã định danh SKU vật lý là duy nhất toàn hệ thống (BR-PR02). |
| `SalesOrder` | `UX_SalesOrder_Cart` | `(CartID)` | `WHERE CartID IS NOT NULL` | Mỗi giỏ hàng chuyển đổi thành tối đa 1 đơn hàng nhưng vẫn cho phép đơn đổi bù `EXCHANGE_REPLACEMENT`. |
| `StockReservationLine` | `UX_StockReservationLine_ActiveAlloc` | `(AllocationID)` | `WHERE ReservationLineStatus = 'ACTIVE'` | Mỗi phân bổ xuất kho có tối đa đúng một dòng giữ hàng `ACTIVE` tại một thời điểm (CORR-10). |
| `PaymentWebhookLog` | `UX_PaymentWebhookLog_ProviderEvent` | `(PaymentMethodID, ProviderEventID, Environment)` | `WHERE ProviderEventID IS NOT NULL` | Chống trùng lặp Webhook xử lý idempotent từ cổng thanh toán (E71, CORR-05). |
| `IdempotencyRecord` | `UQ_IdempotencyRecord_ScopeKey` | `(AccountID, OperationType, IdempotencyKey)` | *(Toàn bảng)* | Chống gửi lặp yêu cầu đặt đơn và khởi tạo thanh toán (E74, CORR-05). |
| `Refund` | `UX_Refund_Assessment` | `(ReturnLineAssessmentID)` | `WHERE ReturnLineAssessmentID IS NOT NULL` | Mỗi lô kết luận KCS được duyệt sinh tối đa 1 chứng từ `Refund`, cho phép `NULL` với hủy đơn trả trước. |
| `ReturnLineAssessment` | `UX_ReturnLineAssessment_Current` | `(ReturnReceiptLineID, AssessmentGroupNo)` | `WHERE IsCurrent = 1` | Bảo toàn tách lô giám định $1:N$ và giữ duy nhất 1 phiên bản hiện hành cho mỗi lô (E72, CORR-01). |

---

## 5. TỔNG KẾT & CHUYỂN GIAO SANG LOGICAL DESIGN (M3)

Tài liệu này cùng với bộ hồ sơ `M2.01` – `M2.08` tạo thành bộ đặc tả Thiết kế Khái niệm chuẩn hóa 100% với **79 thực thể (`E01` – `E79`)**, bảo đảm ánh xạ `1:1` không mâu thuẫn sang Relational Schema chuẩn 3NF/BCNF của Milestone 3.

