# M2.03 - CARDINALITY & BUSINESS INTEGRITY CONSTRAINTS (BẢN SỐ & RÀNG BUỘC TOÀN VẸN NGHIỆP VỤ)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.0 – Official Standardized Specification
* **Hệ quản trị CSDL mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Kế thừa trực tiếp:**
  * Toàn bộ 55 Yêu cầu Chức năng (FR) và ~200 Business Rules (`BR-ID`, `BR-PR`, `BR-IN`, `BR-PO`, `BR-CU`, `BR-OR`, `BR-PRM`, `BR-PY`, `BR-FS`, `BR-RT`, `BR-SH`, `BR-SA`, `BR-AR`) từ M1.
  * Quyết định nền tảng [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md) (BD-01 đến BD-07).
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (70 Thực thể E01 – E70).
  * [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md) & [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md).

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

## 2. MA TRẬN BẢN SỐ TOÀN DIỆN 70 THỰC THỂ (MASTER CARDINALITY MATRIX)

Dưới đây là ma trận chuẩn hóa toàn bộ các mối quan hệ nội bộ và liên miền của 70 thực thể qua 11 phân hệ:

### 2.1. Phân hệ 01: Product Catalog
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Product` | `|| ─── }o` | `ProductCategory` | $1:0..N$ | Partial | **Total** | SP nháp có thể chưa gán danh mục; SP Published bắt buộc có $\ge 1$ (BR-PR05). |
| `Category` | `|| ─── }o` | `ProductCategory` | $1:0..N$ | Partial | **Total** | Danh mục có thể rỗng hoặc chứa nhiều sản phẩm. |
| `Category` (Parent) | `o| ─── }o` | `Category` (Child) | $0..1:0..N$ | Partial | Partial | Cây danh mục đệ quy cha-con; danh mục gốc có `ParentCategoryID IS NULL` (PC-Q02). |
| `Product` | `|| ─── }|` | `ProductVariant` | $1:1..N$ | **Total** | **Total** | Sản phẩm luôn có ít nhất 1 SKU (sản phẩm đơn có SKU mặc định, PC-Q03). Cấm SKU mồ côi. |
| `Product` | `|| ─── }o` | `ProductAttribute` | $1:0..N$ | Partial | **Total** | Sản phẩm khai báo danh sách thuộc tính áp dụng kèm vai trò `AttributeRole` (PC-Q04). |
| `Attribute` | `|| ─── }o` | `ProductAttribute` | $1:0..N$ | Partial | **Total** | Thuộc tính từ điển toàn cục được áp dụng cho nhiều sản phẩm (PC-Q05). |
| `Attribute` | `|| ─── }|` | `AttributeValue` | $1:1..N$ | Partial | **Total** | Thuộc tính có từ điển các giá trị hợp lệ. Giá trị bắt buộc thuộc 1 thuộc tính. |
| `ProductAttribute` | `|| ─── }o` | `ProductAttributeValue` | $1:0..N$ | Partial | **Total** | Whitelist các giá trị được phép chọn cho thuộc tính trên sản phẩm cụ thể. |
| `AttributeValue` | `|| ─── }o` | `ProductAttributeValue` | $1:0..N$ | Partial | **Total** | Giá trị từ điển được đưa vào whitelist của sản phẩm. |
| `ProductVariant` | `|| ─── }o` | `VariantAttributeValue` | $1:0..N$ | Partial | **Total** | Tổ hợp giá trị thuộc tính cấu thành SKU; SKU đơn giản có tổ hợp rỗng (PC-Q03). |
| `ProductAttributeValue` | `|| ─── }o` | `VariantAttributeValue` | $1:0..N$ | Partial | **Total** | Giá trị cấu thành biến thể bắt buộc nằm trong whitelist của sản phẩm (BR-PR12). |

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
| `OrderWarehouseAllocation` | `|| ─── o|` | `StockReservationLine` | $1:0..1$ | Partial | **Total** | Mỗi phân bổ kho tương ứng tối đa 1 dòng giữ hàng logic đang hiệu lực. |
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
| `ShoppingCart` | `|| ─── o|` | `SalesOrder` | $1:0..1$ | Partial | **Total** | Mỗi giỏ hàng chuyển đổi thành tối đa 1 đơn hàng (`CartID` Unique, BR-SA02). |
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

### 2.8. Phân hệ 08: Promotion & Voucher
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `Promotion` | `|| ─── }o` | `PromotionCondition` | $1:0..N$ | Partial | **Total** | Chương trình khai báo các điều kiện chuẩn hóa cấu hình sẵn (PROMO-Q02). |
| `Promotion` | `|| ─── }o` | `PromotionProduct` | $1:0..N$ | Partial | **Total** | Danh sách SKU áp dụng ưu đãi cấp sản phẩm (`Scope = 'PRODUCT'`, PROMO-Q01). |
| `ProductVariant` | `|| ─── }o` | `PromotionProduct` | $1:0..N$ | Partial | **Total** | SKU được thụ hưởng chiết khấu của chương trình khuyến mãi. |
| `Promotion` | `|| ─── }o` | `Voucher` | $1:0..N$ | Partial | **Total** | Chương trình cấp đơn hàng (`Scope = 'ORDER'`) phát hành mã voucher (PROMO-Q03). |
| `Customer` | `o| ─── }o` | `Voucher` | $0..1:0..N$ | Partial | Partial | Voucher cá nhân hóa trỏ đến khách hàng cụ thể; voucher dùng chung có `CustomerID IS NULL`. |
| `Promotion` | `|| ─── }o` | `PromotionTargetCustomer` | $1:0..N$ | Partial | **Total** | Danh sách khách hàng VIP được hưởng chương trình riêng tư (PROMO-Q03). |
| `Customer` | `|| ─── }o` | `PromotionTargetCustomer` | $1:0..N$ | Partial | **Total** | Khách hàng thuộc nhóm đối tượng của chương trình ưu đãi đặc quyền. |
| `Promotion` | `|| ─── }o` | `PromotionUsage` | $1:0..N$ | Partial | **Total** | Nhật ký tiêu thụ ưu đãi thực tế của chương trình (PROMO-Q05). |
| `Voucher` | `o| ─── }o` | `PromotionUsage` | $0..1:0..N$ | Partial | Partial | Ghi nhận mã voucher được sử dụng khi thanh toán đơn hàng (BD-05). |
| `SalesOrder` | `|| ─── }o` | `PromotionUsage` | $1:0..N$ | Partial | **Total** | Đơn hàng ghi nhận ưu đãi voucher cấp đơn (tối đa 1 voucher, BD-05). |
| `OrderLine` | `o| ─── }o` | `PromotionUsage` | $0..1:0..N$ | Partial | Partial | Dòng đơn hàng ghi nhận ưu đãi chiết khấu cấp SKU (tối đa 1 ưu đãi, BD-05). |

### 2.9. Phân hệ 09: Payment & Refund
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `SalesOrder` | `|| ─── ||` | `Payment` | $1:1$ | **Total** | **Total** | Mỗi đơn hàng bắt buộc có đúng 1 hồ sơ nghĩa vụ thanh toán (`Payment.OrderID` UNIQUE, PAY-Q01). |
| `PaymentMethod` | `|| ─── }o` | `Payment` | $1:0..N$ | Partial | **Total** | Phương thức thanh toán được chọn khi checkout đơn hàng (PAY-Q01). |
| `Payment` | `|| ─── }o` | `PaymentTransaction` | $1:0..N$ | Partial | **Total** | Nhật ký các lần gọi cổng thanh toán/thu tiền COD (kể cả lần thử lỗi, PAY-Q03). |
| `Payment` | `|| ─── }o` | `Refund` | $1:0..N$ | Partial | **Total** | Hồ sơ yêu cầu hoàn tiền gắn với khoản thanh toán đã thực thu (PAY-Q04). |
| `ReturnRequest` | `|| ─── }o` | `Refund` | $1:0..N$ | Partial | Partial | Yêu cầu RMA đã kiểm định kích hoạt phát sinh chứng từ hoàn tiền (RET-Q07). |
| `Refund` | `|| ─── }o` | `RefundTransaction` | $1:0..N$ | Partial | **Total** | Giao dịch chuyển khoản thực tế chi trả tiền hoàn lại cho khách (PAY-Q06). |

### 2.10. Phân hệ 10: Return & Exchange (RMA)
| Thực thể Cha (Nguồn) | Ký hiệu | Thực thể Con (Đích) | Bản số | Tham gia Cha | Tham gia Con | Ý nghĩa & Cơ chế kiểm soát |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| `SalesOrder` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Đơn hàng đã giao thành công có thể phát sinh nhiều yêu cầu đổi trả từng phần (RET-Q01). |
| `Customer` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Khách hàng là chủ sở hữu đơn gửi yêu cầu đổi trả (RET-Q01). |
| `Account` (CreatedBy) | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Tài khoản khách tự tạo hoặc CSKH tạo thay trên hệ thống (RET-Q01). |
| `Warehouse` | `|| ─── }o` | `ReturnRequest` | $1:0..N$ | Partial | **Total** | Yêu cầu chỉ định duy nhất 1 kho vật lý tiếp nhận hàng trả (`WarehouseID NOT NULL`, RET-Q06). |
| `ReturnRequest` | `|| ─── }|` | `ReturnRequestLine` | $1:1..N$ | **Total** | **Total** | Yêu cầu RMA bắt buộc có $\ge 1$ dòng hàng yêu cầu trả (BR-RT05). |
| `OrderLine` | `|| ─── }o` | `ReturnRequestLine` | $1:0..N$ | Partial | **Total** | Dòng sản phẩm trong đơn được yêu cầu trả lại (RET-Q03). |
| `ReturnRequest` | `|| ─── }|` | `ReturnStatusHistory` | $1:1..N$ | **Total** | **Total** | RMA bắt buộc có $\ge 1$ bản ghi lịch sử trạng thái (bản ghi đầu `PENDING_APPROVAL`, RET-Q08). |
| `Account` (ChangedBy) | `|| ─── }o` | `ReturnStatusHistory` | $1:0..N$ | Partial | **Total** | Tài khoản người dùng chuyển đổi trạng thái của yêu cầu RMA. |
| `ReturnRequest` | `|| ─── }o` | `ReturnReceipt` | $1:0..N$ | Partial | **Total** | Yêu cầu RMA có thể được tiếp nhận qua nhiều đợt/phiếu nhận tại kho (BR-RT13). |
| `Warehouse` | `|| ─── }o` | `ReturnReceipt` | $1:0..N$ | Partial | **Total** | Phiếu tiếp nhận tại kho; bắt buộc trùng kho chỉ định trên RMA (RET-Q06). |
| `ReturnReceipt` | `|| ─── }|` | `ReturnReceiptLine` | $1:1..N$ | **Total** | **Total** | Phiếu nhập trả bắt buộc có $\ge 1$ dòng kiểm định chất lượng (BR-RT14). |
| `ReturnRequestLine` | `|| ─── }o` | `ReturnReceiptLine` | $1:0..N$ | Partial | **Total** | Dòng kiểm định đối soát về đúng dòng yêu cầu RMA gốc (RET-Q05). |

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
1. **Phân lập 3 máy trạng thái đơn hàng (SALES-Q02, PAY-Q08, SHIP-Q06):**
   * Tuyệt đối độc lập giữa `SalesOrder.OrderStatus`, `Payment.PaymentStatus` và `Shipment.ShipmentStatus`.
   * Đơn hàng COD hợp lệ: Chuyển `OrderStatus = 'CONFIRMED'` ngay cả khi `PaymentStatus = 'PENDING'`.
   * Đơn hàng thanh toán trực tuyến: Chỉ chuyển `OrderStatus = 'CONFIRMED'` khi có giao dịch `PaymentTransaction` thành công và `PaymentStatus = 'PAID'`.
2. **Máy trạng thái đơn hàng tuần tiến (Order Status Machine, BR-SA08, BR-SA09):**
   * Luồng chuyển trạng thái hợp lệ:
     $$\text{PENDING\_PAYMENT} \longrightarrow \text{CONFIRMED} \longrightarrow \text{PROCESSING} \longrightarrow \text{PARTIALLY\_SHIPPED} \longrightarrow \text{SHIPPED} \longrightarrow \text{COMPLETED}$$
   * Nhánh hủy: Chỉ được chuyển `CANCELLED` khi đơn chưa chuyển sang trạng thái `SHIPPED`.
3. **Máy trạng thái kiện hàng & Xử lý giao thất bại (Shipment State Machine, SHIP-Q04, SHIP-Q06):**
   * Luồng: $\text{PREPARING} \longrightarrow \text{READY\_TO\_DISPATCH} \longrightarrow \text{IN\_TRANSIT} \longrightarrow \text{DELIVERED}$.
   * Nhánh thất bại: $\text{DELIVERY\_FAILED} \longrightarrow$ Giao lại (tối đa 3 lần `AttemptNumber`) $\longrightarrow$ Vượt quá chuyển sang $\text{RETURNING} \longrightarrow \text{RETURNED}$.

### 3.3. Ràng buộc Mô hình Tồn kho 4 Tầng & Kế toán Kho (4-Tier Inventory Constraints)
1. **Công thức tồn khả dụng và nguyên tắc All-or-Nothing (WI-Q02, BD-04, SALES-Q01):**
   $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} \ge 0$$
   * Giao dịch checkout đơn hàng là nguyên tử: Nếu bất kỳ SKU nào trong đơn không đáp ứng đủ $\text{OrderedQty} \le \text{AvailableQty}$ tại các kho phân bổ, toàn bộ transaction bị hủy bỏ (Rollback), giỏ hàng giữ nguyên `ACTIVE`.
2. **Số dư tồn kho vật lý không âm (Non-negative OnHand, WI-Q01):**
   * Ràng buộc: `CHECK (OnHandQty >= 0 AND ReservedQty >= 0)`.
3. **Sổ cái biến động kho hai chiều Dual-Delta (WI-Q02):**
   * Mọi sự kiện làm thay đổi tồn kho đều ghi nhận vào `InventoryTransactionLine` với hai tham số biến động trực tiếp: `OnHandDelta` và `ReservedDelta`.
   * Chứng từ sau khi `PostedAt` là bất biến (Immutable), không được phép sửa đổi hoặc xóa vật lý.
4. **Cô lập hàng In-Transit trong chuyển kho hai bước (WI-Q04):**
   * Bước 1 xuất kho nguồn (`DISPATCHED`): Giảm `OnHandQty` tại kho nguồn. Hàng trong quá trình vận chuyển không thuộc `OnHandQty` của bất kỳ kho nào.
   * Bước 2 nhập kho đích (`RECEIVED`): Tăng `OnHandQty` tại kho đích đúng bằng số lượng thực nhận `ReceivedQty`. Chênh lệch `VarianceQty` ghi nhận xử lý riêng.
5. **Cô lập hàng hoàn khi giao thất bại (BD-07, Principle C):**
   * Shipper báo giao thất bại (`ShipmentStatus = 'DELIVERY_FAILED'`): **Tuyệt đối không tăng tồn kho.** Hàng vẫn đang nằm trên xe hoặc bưu cục.
   * Chỉ khi thủ kho nhận kiện hàng tại bến nhập và ký biên bản tiếp nhận `PostedAt` với loại `DELIVERY_RETURN_IN`, hệ thống mới ghi sổ kế toán và tăng lại `OnHandQty`.
6. **Phân tách trách nhiệm kiểm kê kho (Separation of Duties, WI-Q05, WI-Q06):**
   * Phiếu kiểm kê hoàn tất đếm (`CountStatus = 'COMPLETED'`) không tự động điều chỉnh số dư.
   * Bắt buộc có quản lý phê duyệt (`ApprovedBy NOT NULL`, `ApprovedAt NOT NULL`).
   * Ràng buộc kiểm soát nội bộ: Người kiểm đếm không được tự duyệt phiếu của chính mình (`ApprovedBy <> CreatedBy`).

### 3.4. Ràng buộc Số lượng, Tài chính & Hoàn tiền (Quantitative & Financial Integrity)
1. **Bảo toàn số lượng phân bổ Split Shipment (BD-03, WI-Q03):**
   $$\forall \text{OrderLine}: \sum_{\text{Allocation}} \text{AllocatedQty} = \text{OrderLine.OrderedQty}$$
2. **Kiểm soát đóng gói kiện hàng không vượt quá đặt mua (SHIP-Q01, BR-FS08):**
   $$\forall \text{OrderLine}: \sum_{\text{ShipmentLine}} \text{ShipmentQty} \le \text{OrderLine.OrderedQty}$$
   * Kiện hàng `Shipment` chỉ được đóng gói từ các dòng phân bổ `OrderWarehouseAllocation` có cùng `WarehouseID`.
3. **Bảo toàn số lượng kiểm nhận hàng mua (PR-Q03, PR-Q04):**
   * Trên từng dòng tiếp nhận: $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$.
   * Lũy kế nhận qua các đợt không vượt quá số lượng đặt mua:
     $$\forall \text{POLine}: \sum_{\text{GoodsReceiptLine}} \text{ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$$
4. **Hạn mức hoàn tiền tối đa (Financial Refund Ceiling, PAY-Q04, BR-PY15):**
   $$\sum \text{RefundAmount} \le \text{RemainingRefundable} = \text{AmountPaid} - \text{AmountRefunded}$$
   * Chỉ các yêu cầu `Refund` đã được duyệt (`ApprovedBy NOT NULL`) và có giao dịch `RefundTransaction` thành công (`SUCCESS`) mới được cập nhật tăng `Payment.AmountRefunded`.
5. **Hạn mức đổi trả hàng RMA không vượt quá số lượng mua (RET-Q03, BR-RT08):**
   $$\forall \text{OrderLine}: \sum_{\text{RMA}} \text{ApprovedQty} \le \text{OrderLine.OrderedQty} - \text{AlreadyReturnedQty}$$
   * Chỉ khi kho ký xác nhận nhận hàng trả (`ReturnReceipt.PostedAt` NOT NULL) với phân loại `Disposition = 'SELLABLE'`, hệ thống mới cộng lại tồn kho bán và kích hoạt hoàn tiền sang phân hệ Payment.

### 3.5. Ràng buộc Khuyến mãi Hai Tầng & Kết hợp Mã (Two-Tier Promotion Constraints)
1. **Cấu trúc phân tầng khuyến mãi nghiêm ngặt (BD-05, PROMO-Q01..08):**
   * **Tầng 1 (Product-Level):** Tối đa 1 khuyến mãi áp dụng trên mỗi dòng `OrderLine`.
   * **Tầng 2 (Order-Level):** Tối đa 1 voucher áp dụng trên mỗi đơn hàng `SalesOrder`.
   * **Quy tắc Stacking:** Hai tầng chỉ được áp dụng đồng thời khi chương trình có cờ `IsStackable = 1`. Nếu không stackable, chương trình có độ ưu tiên cao hơn (`Priority` nhỏ hơn) sẽ được chọn.
2. **Kiểm soát hạn mức sử dụng hai cấp (PROMO-Q04, BR-PRM12, BR-PRM13):**
   * Toàn sàn: `UsedCount <= MaxUsage`.
   * Theo khách hàng: $\text{Count}(\text{PromotionUsage where CustomerID}) \le \text{MaxUsagePerCustomer}$.
3. **Vòng đời tiêu thụ ưu đãi (PROMO-Q05, PROMO-Q06):**
   * Tạo đơn: Tạo bản ghi `PromotionUsage` ở trạng thái `PENDING` (khóa quota tạm).
   * Thanh toán thành công: Chuyển `CONSUMED` và tăng chính thức `UsedCount`.
   * Hủy đơn/Hết hạn: Chuyển `REVERSED` và hoàn trả quota cho khách hàng.

### 3.6. Ràng buộc Bất biến Lịch sử & Cấm Xóa Cứng (Snapshot & Immutability)
1. **Snapshot bất biến dòng đơn hàng (SALES-Q04, SALES-Q05):**
   * `OrderLine` bắt buộc lưu bản sao bất biến của: `SKU`, `ProductName`, `VariantName`, `UnitPrice`, và chuỗi JSON `VariantAttributesSnapshot`.
   * Mọi biến động giá bán, tên gọi hay thay đổi thuộc tính trong Catalog sau thời điểm đặt hàng đều không ảnh hưởng đến đơn hàng cũ.
2. **Snapshot bất biến địa chỉ giao hàng (SALES-Q06):**
   * `OrderAddress` là một bản ghi độc lập với `CustomerAddress`, sao chép nguyên văn tên người nhận, số điện thoại và địa chỉ tại thời điểm đặt.
   * Khách hàng sửa hoặc xóa địa chỉ trong sổ địa chỉ cá nhân không làm thay đổi địa chỉ của đơn hàng đã phát sinh.
3. **Nhật ký chỉ ghi (Append-Only / WORM) & Cấm xóa vật lý (No Hard Delete):**
   * Cấm tuyệt đối lệnh `DELETE` vật lý trên toàn bộ các bảng: `SalesOrder`, `OrderLine`, `Payment`, `PaymentTransaction`, `Shipment`, `ReturnRequest`, `InventoryTransaction`, `AuditLog`.
   * Mọi thao tác hủy đều thể hiện qua cập nhật trạng thái `CANCELLED` hoặc ghi nhận giao dịch đảo ngược (Reverse Transaction).

---

## 4. CHIẾN LƯỢC CÀI ĐẶT FILTERED UNIQUE INDEXES TRÊN SQL SERVER

Để hiện thực hóa các ràng buộc nghiệp vụ có điều kiện một cách tối ưu hiệu năng và an toàn dữ liệu, các chỉ mục duy nhất có lọc (Filtered Unique Indexes) sau đây được quy định chuẩn hóa:

| Bảng nghiệp vụ | Tên Index dự kiến | Cột lập chỉ mục (Key Columns) | Điều kiện lọc (WHERE Filter) | Ý nghĩa nghiệp vụ bảo vệ |
| :--- | :--- | :--- | :--- | :--- |
| `ProductCategory` | `UQ_ProductCategory_Primary` | `(ProductID)` | `WHERE IsPrimary = 1` | Mỗi sản phẩm có duy nhất đúng một danh mục chính khi hiển thị (PC-Q01, BD-01). |
| `Account` | `UQ_Account_Email` | `(Email)` | `WHERE Email IS NOT NULL` | Email duy nhất toàn hệ thống nhưng cho phép tài khoản đăng ký bằng Phone (CID-Q03). |
| `Account` | `UQ_Account_Phone` | `(Phone)` | `WHERE Phone IS NOT NULL` | Phone duy nhất toàn hệ thống nhưng cho phép tài khoản đăng ký bằng Email (CID-Q03). |
| `CustomerAddress` | `UQ_CustomerAddress_Default` | `(CustomerID)` | `WHERE IsDefault = 1` | Mỗi khách hàng có tối đa đúng một địa chỉ mặc định trong sổ địa chỉ (CID-Q06). |
| `ShoppingCart` | `UQ_ShoppingCart_Active` | `(CustomerID)` | `WHERE CartStatus = 'ACTIVE'` | Mỗi khách hàng có tối đa đúng một giỏ hàng đang hoạt động tại một thời điểm (SHOP-Q01). |
| `Shipment` | `UQ_Shipment_TrackingNumber` | `(CarrierID, TrackingNumber)` | `WHERE TrackingNumber IS NOT NULL` | Mã vận đơn của đối tác logistics là duy nhất trong cùng một đơn vị vận chuyển. |
| `Payment` | `UQ_Payment_Order` | `(OrderID)` | *(Toàn bảng)* | Mỗi đơn hàng có đúng một hồ sơ thanh toán duy nhất ($1:1$, PAY-Q01). |
| `OrderAddress` | `UQ_OrderAddress_Order` | `(OrderID)` | *(Toàn bảng)* | Mỗi đơn hàng có đúng một địa chỉ giao hàng snapshot duy nhất ($1:1$, SALES-Q06). |
| `Voucher` | `UQ_Voucher_Code` | `(VoucherCode)` | *(Toàn bảng)* | Mã code voucher khuyến mãi là duy nhất toàn hệ thống. |
| `ProductVariant` | `UQ_ProductVariant_SKU` | `(SKU)` | *(Toàn bảng)* | Mã định danh SKU vật lý là duy nhất toàn hệ thống (BR-PR04). |

---

## 5. TỔNG KẾT & CHUYỂN GIAO SANG LOGICAL DESIGN (M3)

Tài liệu này cùng với [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md), [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md), [04-Conceptual-ERD.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/04-Conceptual-ERD.md) và [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md) tạo thành bộ hồ sơ Thiết kế Khái niệm hoàn chỉnh 100%, bảo đảm không có bất kỳ mâu thuẫn hay điểm nghẽn nào cản trở quá trình ánh xạ sang Relational Schema chuẩn 3NF/BCNF của Milestone 3.

