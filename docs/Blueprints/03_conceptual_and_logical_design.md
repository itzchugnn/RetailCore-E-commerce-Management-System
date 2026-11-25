# PHẦN IV & V: THIẾT KẾ CƠ SỞ DỮ LIỆU QUAN NIỆM VÀ LOGIC

> [!WARNING]
> **TÀI LIỆU KHỞI THỦY LỊCH SỬ (HISTORICAL SEED v0.1 — SUPERSEDED):** Mô hình 28 thực thể / 42 bảng tên số nhiều trong bản nháp này **đã được thay thế hoàn toàn** bởi chuẩn **79 Thực thể danh từ số ít (`E01`–`E79`)** tại [01-Entity-Catalog.md](../Milestones/M2-Conceptual-Design/01-Entity-Catalog.md), `docs/erd/mermaid/*.md` và `docs/Milestones/M3-Logical-Design/`. Tuyệt đối không sử dụng tên bảng số nhiều hoặc lược đồ cũ trong văn bản này.

---

## PHẦN IV. THIẾT KẾ CƠ SỞ DỮ LIỆU QUAN NIỆM (CONCEPTUAL DESIGN)
**Deliverable 01 – ERD / EER Diagram**

Ở giai đoạn này, trọng tâm là tư duy mô hình hóa dữ liệu dựa trên bản chất nghiệp vụ, độc lập với hệ quản trị cơ sở dữ liệu vật lý.

### 4.1. Phân chia 10 nhóm thực thể nghiệp vụ

| Nhóm nghiệp vụ | Các thực thể dự kiến (Conceptual Entities) |
| :--- | :--- |
| **A. Identity & Access** | `User`, `Role`, `Permission` |
| **B. Product Catalog** | `Category`, `Brand`, `Product`, `SKU` (Variant), `Attribute` |
| **C. Warehouse** | `Warehouse`, `Inventory`, `StockMovement` |
| **D. Procurement** | `Supplier`, `PurchaseOrder`, `PurchaseOrderDetail` |
| **E. Customer** | `Customer`, `CustomerAddress` |
| **F. Shopping** | `Cart`, `CartItem` |
| **G. Sales Order** | `SalesOrder`, `OrderDetail`, `OrderStatusHistory` |
| **H. Promotion** | `Promotion`, `Voucher`, `PromotionProduct` |
| **I. Payment & Shipping** | `Payment`, `Refund`, `Shipment` |
| **J. Return & Reporting** | `ReturnRequest`, `ReturnDetail`, `AuditLog` |

> *Lưu ý: Đây là danh sách thực thể sơ bộ ở mức quan niệm. Khi chuyển sang mức logic, một thực thể quan niệm có thể được ánh xạ thành một hoặc nhiều bảng quan hệ.*

---

### 4.2. Các quan hệ trọng tâm và Bản số (Cardinality)

| Quan hệ | Cardinality định hướng | Ý nghĩa nghiệp vụ |
| :--- | :--- | :--- |
| **Category – Product** | $N - N$ (hoặc $1 - N$ tùy quyết định) | Một sản phẩm có thể thuộc nhiều danh mục kinh doanh/hiển thị. |
| **Product – SKU** | $1 - N$ | Một sản phẩm gốc có nhiều biến thể (SKU) cụ thể. |
| **SKU – Warehouse** | $N - N$ | Một biến thể SKU có thể được lưu trữ tại nhiều kho hàng. |
| **Customer – Order** | $1 - N$ | Một khách hàng có thể đặt nhiều đơn hàng theo thời gian. |
| **Order – OrderDetail** | $1 - N$ | Một đơn hàng bao gồm một hoặc nhiều dòng chi tiết hàng hóa. |
| **Order – Payment** | $1 - N$ | Một đơn hàng có thể phát sinh nhiều giao dịch/lần thử thanh toán. |
| **Promotion – Product** | $N - N$ | Một chương trình khuyến mãi áp dụng cho nhiều sản phẩm và ngược lại. |
| **Order – Shipment** | $1 - N$ | Một đơn hàng có thể được chia nhỏ để giao thành nhiều đợt/kiện hàng từ các kho khác nhau. |
| **Order – ReturnRequest** | $1 - N$ | Một đơn hàng có thể phát sinh một hoặc nhiều yêu cầu đổi trả (trả từng phần). |

---

### 4.3. Bảy bài toán kiến trúc dữ liệu mức quan niệm cần giải quyết
Đây là 7 câu hỏi kiến trúc dữ liệu cốt lõi phải có lập luận bảo vệ rõ ràng (không được chọn phương án chỉ vì muốn giảm số lượng bảng):

1. **Phân loại danh mục:** Một sản phẩm thuộc nhiều danh mục ($N - N$) hay chỉ thuộc duy nhất một danh mục lá ($1 - N$)?
2. **Mô hình hóa biến thể:** Biến thể sản phẩm nên được mô hình hóa thế nào khi mỗi ngành hàng (Thời trang vs. Đồ gia dụng) có tập thuộc tính hoàn toàn khác nhau?
3. **Định vị thuộc tính Giá bán:** Giá bán (`Price`) là thuộc tính của `Product` hay của `SKU` (`ProductVariant`), hay cả hai (giá cơ sở ở Product và giá ghi đè ở SKU)?
4. **Đơn vị quản lý Tồn kho:** Tồn kho phải được quản lý ở mức `Product` hay bắt buộc ở mức `SKU`?
5. **Khách vãng lai (Guest Checkout):** Khách hàng có thể đặt hàng mà không cần đăng ký tài khoản (`User`) hay không? Nếu có, thực thể `Customer` và `Order` liên kết thế nào?
6. **Phân bổ kho khi giao hàng (Split Shipment):** Một đơn hàng có thể được xuất giao từ nhiều kho hàng khác nhau hay không?
7. **Kết hợp khuyến mãi:** Một đơn hàng có thể áp dụng đồng thời nhiều chương trình khuyến mãi và voucher hay không?

---

## PHẦN V. THIẾT KẾ CƠ SỞ DỮ LIỆU LOGIC (LOGICAL DESIGN)
**Deliverable 02 – Relational Schema**

Quy mô mục tiêu cho phiên bản đầu tiên: **35–45 bảng quan hệ**. Đây là quy mô đủ độ sâu để giải quyết các bài toán toàn vẹn dữ liệu thực tế mà không biến dự án thành một hệ thống ERP quá cồng kềnh.

### 5.1. Danh sách Lược đồ Quan hệ Dự kiến

**Ký hiệu quy ước:**
* `PK`: Primary Key (Khóa chính)
* `FK`: Foreign Key (Khóa ngoại)
* `UQ`: Unique (Ràng buộc duy nhất)
* `NN`: Not Null (Bắt buộc có giá trị)

#### A. Identity & Access Management (5 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Users** | `UserID (PK)`, `Email (UQ, NN)`, `PasswordHash (NN)`, `FullName (NN)`, `Phone`, `Status (NN)` |
| **Roles** | `RoleID (PK)`, `RoleName (UQ, NN)` |
| **Permissions** | `PermissionID (PK)`, `PermissionCode (UQ, NN)` |
| **UserRoles** | `UserID (PK, FK)`, `RoleID (PK, FK)` |
| **RolePermissions** | `RoleID (PK, FK)`, `PermissionID (PK, FK)` |

* **Trọng tâm nghiên cứu:** Mô hình RBAC và nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege).

#### B. Product Catalog (8 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Categories** | `CategoryID (PK)`, `ParentCategoryID (FK)`, `CategoryName (NN)` |
| **Brands** | `BrandID (PK)`, `BrandName (UQ, NN)` |
| **Products** | `ProductID (PK)`, `BrandID (FK)`, `ProductName (NN)`, `Description`, `Status (NN)` |
| **ProductCategories** | `ProductID (PK, FK)`, `CategoryID (PK, FK)` |
| **ProductVariants** | `VariantID (PK)`, `ProductID (FK, NN)`, `SKU (UQ, NN)`, `Price (NN)`, `Status (NN)` |
| **Attributes** | `AttributeID (PK)`, `AttributeName (UQ, NN)` |
| **VariantAttributeValues** | `VariantID (PK, FK)`, `AttributeID (PK, FK)`, `AttributeValue (NN)` |
| **ProductImages** | `ImageID (PK)`, `ProductID (FK, NN)`, `ImageURL (NN)`, `DisplayOrder (NN)` |

* **Ví dụ minh họa dữ liệu Biến thể (SKU):**

| Product | SKU | Size | Color |
| :--- | :--- | :--- | :--- |
| Áo thun Basic | `AT001-S-W` | S | White |
| Áo thun Basic | `AT001-M-W` | M | White |
| Áo thun Basic | `AT001-L-B` | L | Black |

#### C. Warehouse & Inventory (4 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Warehouses** | `WarehouseID (PK)`, `WarehouseName (NN)`, `Address (NN)`, `Status (NN)` |
| **Inventory** | `WarehouseID (PK, FK)`, `VariantID (PK, FK)`, `OnHandQty (NN)`, `ReservedQty (NN)` |
| **StockMovements** | `MovementID (PK)`, `WarehouseID (FK, NN)`, `VariantID (FK, NN)`, `QuantityChange (NN)`, `MovementType (NN)`, `ReferenceID`, `CreatedAt (NN)` |
| **StockReservations** | `ReservationID (PK)`, `OrderID (FK, NN)`, `WarehouseID (FK, NN)`, `VariantID (FK, NN)`, `Quantity (NN)`, `Status (NN)`, `ExpiresAt (NN)` |

> **Nguyên tắc thiết kế sống còn:** Bảng `Inventory` lưu trạng thái tồn kho hiện tại để truy vấn nhanh, còn bảng `StockMovements` đóng vai trò là sổ cái bất biến (Immutable Ledger) ghi nhận toàn bộ lịch sử biến động. Tuyệt đối không chỉ đặt một cột `Quantity` trong `ProductVariants` rồi cộng trừ trực tiếp mà không có đối soát lịch sử.

#### D. Procurement (3 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Suppliers** | `SupplierID (PK)`, `SupplierName (NN)`, `Phone`, `Email` |
| **PurchaseOrders** | `PurchaseOrderID (PK)`, `SupplierID (FK, NN)`, `WarehouseID (FK, NN)`, `CreatedAt (NN)`, `Status (NN)` |
| **PurchaseOrderDetails** | `PurchaseOrderDetailID (PK)`, `PurchaseOrderID (FK, NN)`, `VariantID (FK, NN)`, `Quantity (NN)`, `UnitCost (NN)` |

#### E. Customer Management (2 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Customers** | `CustomerID (PK)`, `UserID (FK, UQ)`, `CustomerCode (UQ, NN)`, `CreatedAt (NN)` |
| **CustomerAddresses** | `AddressID (PK)`, `CustomerID (FK, NN)`, `RecipientName (NN)`, `Phone (NN)`, `AddressDetail (NN)`, `IsDefault (NN)` |

#### F. Shopping Cart (2 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Carts** | `CartID (PK)`, `CustomerID (FK, NN)`, `CreatedAt (NN)`, `Status (NN)` |
| **CartItems** | `CartItemID (PK)`, `CartID (FK, NN)`, `VariantID (FK, NN)`, `Quantity (NN)` |

* **Điểm cần chốt:** Xác định rõ mỗi khách hàng chỉ có duy nhất một giỏ hàng đang hoạt động (`Active`) hay có thể lưu nhiều giỏ hàng cùng lúc.

#### G. Sales Order (4 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Orders** | `OrderID (PK)`, `OrderCode (UQ, NN)`, `CustomerID (FK, NN)`, `OrderDate (NN)`, `Status (NN)`, `Subtotal (NN)`, `DiscountAmount (NN)`, `ShippingFee (NN)`, `TotalAmount (NN)` |
| **OrderDetails** | `OrderDetailID (PK)`, `OrderID (FK, NN)`, `VariantID (FK, NN)`, `ProductNameSnapshot (NN)`, `SKUSnapshot (NN)`, `UnitPrice (NN)`, `Quantity (NN)`, `DiscountAmount (NN)` |
| **OrderAddresses** | `OrderAddressID (PK)`, `OrderID (FK, UQ, NN)`, `RecipientName (NN)`, `Phone (NN)`, `AddressDetail (NN)` |
| **OrderStatusHistory** | `HistoryID (PK)`, `OrderID (FK, NN)`, `OldStatus`, `NewStatus (NN)`, `ChangedAt (NN)`, `ChangedBy (FK)` |

> **Bảo toàn dữ liệu lịch sử (Snapshot Pattern):**
> Khi khách mua một sản phẩm với giá $500.000\text{ VNĐ}$, một tuần sau giá danh mục tăng lên $600.000\text{ VNĐ}$, đơn hàng cũ bắt buộc vẫn phải hiển thị đúng giá $500.000\text{ VNĐ}$. Do đó, `OrderDetails` và `OrderAddresses` phải lưu bản chụp (Snapshot) tại thời điểm giao dịch.

#### H. Promotion Management (5 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **Promotions** | `PromotionID (PK)`, `PromotionName (NN)`, `StartDate (NN)`, `EndDate (NN)`, `Priority (NN)`, `Status (NN)` |
| **PromotionProducts** | `PromotionID (PK, FK)`, `ProductID (PK, FK)` |
| **Vouchers** | `VoucherID (PK)`, `PromotionID (FK, NN)`, `VoucherCode (UQ, NN)`, `UsageLimit (NN)`, `UsedCount (NN)` |
| **OrderPromotions** | `OrderID (PK, FK)`, `PromotionID (PK, FK)`, `DiscountAmount (NN)` |
| **VoucherUsages** | `VoucherUsageID (PK)`, `VoucherID (FK, NN)`, `CustomerID (FK, NN)`, `OrderID (FK, NN)`, `UsedAt (NN)` |

* **Phân định rõ trách nhiệm:**
  * `Promotions`: Định nghĩa chương trình và khung thời gian/quy tắc khuyến mãi.
  * `Vouchers`: Định nghĩa mã giảm giá cụ thể mà khách hàng nhập lúc Checkout.
  * `OrderPromotions`: Ghi nhận các chương trình thực tế đã được áp dụng vào một đơn hàng cùng số tiền giảm tương ứng.
  * `VoucherUsages`: Theo dõi lịch sử từng lượt sử dụng voucher của từng khách hàng trên từng đơn hàng.

#### I. Payment & Shipping (6 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **PaymentMethods** | `PaymentMethodID (PK)`, `MethodName (UQ, NN)` |
| **Payments** | `PaymentID (PK)`, `OrderID (FK, NN)`, `PaymentMethodID (FK, NN)`, `Amount (NN)`, `Status (NN)`, `TransactionReference` |
| **Refunds** | `RefundID (PK)`, `PaymentID (FK, NN)`, `Amount (NN)`, `Status (NN)`, `CreatedAt (NN)` |
| **Shipments** | `ShipmentID (PK)`, `OrderID (FK, NN)`, `WarehouseID (FK, NN)`, `TrackingCode (UQ)`, `Status (NN)` |
| **ShipmentDetails** | `ShipmentID (PK, FK)`, `OrderDetailID (PK, FK)`, `Quantity (NN)` |
| **ShipmentHistory** | `ShipmentHistoryID (PK)`, `ShipmentID (FK, NN)`, `Status (NN)`, `UpdatedAt (NN)` |

#### J. Return & Audit (3 bảng)
| Bảng | Các thuộc tính chính |
| :--- | :--- |
| **ReturnRequests** | `ReturnID (PK)`, `OrderID (FK, NN)`, `CustomerID (FK, NN)`, `Reason (NN)`, `Status (NN)` |
| **ReturnDetails** | `ReturnDetailID (PK)`, `ReturnID (FK, NN)`, `OrderDetailID (FK, NN)`, `Quantity (NN)`, `Condition (NN)` |
| **AuditLogs** | `AuditLogID (PK)`, `UserID (FK)`, `Action (NN)`, `EntityName (NN)`, `EntityID (NN)`, `CreatedAt (NN)` |

---

### 5.2. Phương pháp luận Chuẩn hóa Dữ liệu (Normalization Analysis)

Dự án yêu cầu thực hiện phân tích chuẩn hóa thực chất bằng lý thuyết cơ sở dữ liệu quan hệ, không chỉ tuyên bố cảm tính rằng "tất cả các bảng đã đạt chuẩn 3NF".

#### Minh họa bài toán phân rã lược đồ chưa chuẩn:
Giả sử có lược đồ chưa chuẩn hóa:
$$\text{Order}(\text{OrderID}, \text{CustomerID}, \text{CustomerName}, \text{CustomerPhone}, \text{ProductID}, \text{ProductName}, \text{Quantity}, \text{UnitPrice})$$

**Các dị thường dữ liệu (Anomalies) gặp phải:**
* Một đơn hàng có nhiều sản phẩm dẫn đến dư thừa lặp lại thông tin đơn hàng và khách hàng.
* Dị thường cập nhật (Update Anomaly) khi khách đổi số điện thoại hoặc sản phẩm đổi tên.
* Dị thường thêm/xóa (Insertion/Deletion Anomaly) khi chưa có đơn hàng hoặc khi xóa đơn hàng duy nhất.

**Thực hiện phân rã thành các lược đồ chuẩn hóa:**
1. $\text{Customer}(\underline{\text{CustomerID}}, \text{CustomerName}, \text{CustomerPhone})$
2. $\text{Product}(\underline{\text{ProductID}}, \text{ProductName})$
3. $\text{Orders}(\underline{\text{OrderID}}, \text{CustomerID}, \text{OrderDate})$
4. $\text{OrderDetail}(\underline{\text{OrderID}, \text{ProductID}}, \text{Quantity}, \text{UnitPrice})$

**Quy trình 6 bước chứng minh bắt buộc cho các cụm bảng chính trong Deliverable 02:**
1. Xác định tập các phụ thuộc hàm $\mathcal{F}$ (Functional Dependencies).
2. Tìm tất cả khóa ứng viên (Candidate Keys) thông qua bao đóng thuộc tính $X^+$.
3. Chọn khóa chính (Primary Key).
4. Kiểm tra và xác định dạng chuẩn hiện tại ($1\text{NF}, 2\text{NF}, 3\text{NF}, \text{BCNF}$) của từng lược đồ.
5. Chứng minh phép phân rã có **bảo toàn thông tin (Lossless-join Decomposition)** hay không.
6. Chứng minh phép phân rã có **bảo toàn phụ thuộc hàm (Dependency Preservation)** hay không.

> **Lưu ý chuyên môn về Snapshot và Chuẩn hóa:**
> Việc lưu `UnitPrice` hay `ProductNameSnapshot` trong bảng `OrderDetails` **không phải là vi phạm chuẩn hóa (Denormalization)**. Về mặt ngữ nghĩa, `Price` trong `ProductVariants` là *giá niêm yết hiện tại*, trong khi `UnitPrice` trong `OrderDetails` là *giá giao dịch lịch sử tại thời điểm xác lập hợp đồng mua bán*. Hai thuộc tính này mang ý nghĩa nghiệp vụ khác nhau và phụ thuộc hàm vào hai khóa khác nhau:
> * $\text{VariantID} \rightarrow \text{CurrentPrice}$
> * $(\text{OrderID}, \text{VariantID}) \rightarrow \text{TransactionUnitPrice}$

---

### 5.3. Các bài toán thiết kế Logic nâng cao cần giải quyết

| Chủ đề | Câu hỏi thiết kế cần giải quyết trong M3 |
| :--- | :--- |
| **EAV (Entity–Attribute–Value)** | Có nên dùng mô hình `Attribute – Value` cho biến thể sản phẩm, hay kết hợp JSON/Cột cố định để tối ưu truy vấn lọc? |
| **Hierarchical Data** | Quản lý cây danh mục nhiều cấp (`Categories`) bằng danh sách kề (`ParentCategoryID`) kết hợp Recursive CTE hay dùng `Closure Table` / `HierarchyID`? |
| **Historical Data** | Ngoài snapshot trong đơn hàng, có cần thiết kế thêm bảng `ProductPriceHistory` để theo dõi lịch sử biến động giá niêm yết hay không? |
| **Inventory Accounting** | Cơ chế đồng bộ giữa 3 chỉ số: Tồn thực tế (`OnHandQty`), Tồn đang giữ (`ReservedQty`) và Tồn khả dụng (`AvailableQty`). |
| **Order Snapshot** | Xác định danh sách chính xác các trường cần chụp lại (Snapshot) trên đơn hàng và địa chỉ giao hàng. |
| **Promotion Engine** | Mô hình hóa cấu trúc dữ liệu biểu diễn điều kiện áp dụng và quy tắc kết hợp nhiều chương trình khuyến mãi. |
| **Payment & Refund** | Mô hình hóa quan hệ khi một đơn hàng thanh toán nhiều lần (Partial/Retry Payment) và hoàn tiền nhiều đợt (Partial Refund). |