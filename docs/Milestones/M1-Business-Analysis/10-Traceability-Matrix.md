# 10. TRACEABILITY MATRIX, EXIT CRITERIA & BUSINESS DECISION MAPPING

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.1 – Approved Mapping
* **Database Target:** Microsoft SQL Server

---

## 10.1. Full Requirements Traceability Matrix

Ma trận này kết nối xuyên suốt giữa **Miền nghiệp vụ**, **Use Cases**, **Business Rules (v1.1)**, **Functional Requirements**, **Data Requirements (v1.1)**, **Business Decisions (BD-01 đến BD-07)** và **Thực thể CSDL dự kiến ở M2**:

| Miền nghiệp vụ | Use Case liên quan | Business Rules (v1.1) | Functional Requirements | Data Requirements (v1.1) | Business Decisions | Thực thể CSDL dự kiến (M2) |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **Product Catalog** | UC Product Management, UC Browse Catalog | `BR-PR01` $\rightarrow$ `BR-PR12` | `FR-CAT-01` $\rightarrow$ `FR-CAT-05` | `DA-02`, `DR-11` $\rightarrow$ `DR-14`, `HDR-01`, `HDR-13`, `HDR-14` | **BD-01**, **BD-06** | `Product`, `Category`, `ProductCategory`, `ProductVariant`, `Attribute`, `AttributeValue`, `ProductAttribute`, `VariantAttributeValue` |
| **Inventory & Warehouse** | UC-02, UC-03, UC-08 | `BR-IN01` $\rightarrow$ `BR-IN21` | `FR-INV-01` $\rightarrow$ `FR-INV-05` | `DA-03`, `DA-04`, `DR-15`, `DR-16`, `DR-19`, `HDR-05`, `HDR-15`, `HDR-17` | **BD-04**, **BD-07** | `Warehouse`, `InventoryBalance`, `StockReservation`, `StockMovement` |
| **Customer & Cart** | UC Customer & Cart Management | `BR-CU01` $\rightarrow$ `BR-CU07` | `FR-AUTH-01` $\rightarrow$ `FR-AUTH-04`, `FR-ORD-01` | `DA-06`, `DA-07`, `HDR-04` | **BD-02** | `Customer`, `CustomerAddress`, `Cart`, `CartLine` |
| **Sales & Order** | UC-01, UC-05 (Checkout & Order Lifecycle) | `BR-OR01` $\rightarrow$ `BR-OR12`, `BR-SA01` $\rightarrow$ `BR-SA18` | `FR-ORD-01` $\rightarrow$ `FR-ORD-06` | `DA-08`, `DR-15`, `HDR-02`, `HDR-03`, `HDR-06`, `HDR-16` | **BD-02**, **BD-03**, **BD-04**, `SALES-Q01` $\rightarrow$ `SALES-Q07` | `SalesOrder`, `OrderLine`, `OrderAddress`, `OrderStatusHistory`, `OrderAllocation` |
| **Fulfillment & Shipping** | UC-05 (Shipping & Delivery Return) | `BR-FS01` $\rightarrow$ `BR-FS22`, `BR-IN17` $\rightarrow$ `BR-IN21` | `FR-SHP-01` $\rightarrow$ `FR-SHP-03` | `DA-11`, `DR-17`, `DR-18`, `DR-20`, `HDR-10`, `HDR-17` | **BD-03**, **BD-07**, `SHIP-Q01` $\rightarrow$ `SHIP-Q07` | `Carrier`, `Shipment`, `ShipmentLine`, `DeliveryAttempt`, `ShipmentStatusHistory` |
| **Promotion & Voucher** | UC-06 (Apply Promotion & Voucher) | `BR-PRM01` $\rightarrow$ `BR-PRM24` | `FR-PRM-01` $\rightarrow$ `FR-PRM-02` | `DA-09`, `DR-21` $\rightarrow$ `DR-24`, `HDR-09`, `HDR-18` | **BD-05**, `PROMO-Q01` $\rightarrow$ `PROMO-Q08` | `Promotion`, `PromotionCondition`, `PromotionProduct`, `Voucher`, `PromotionUsage`, `PromotionTargetCustomer` |
| **Payment & Refund** | UC-04 (Payment & Refund) | `BR-PY01` $\rightarrow$ `BR-PY22` | `FR-PAY-01` $\rightarrow$ `FR-PAY-05` | `DA-10`, `HDR-07`, `HDR-08` | `PAY-Q01` $\rightarrow$ `PAY-Q09` | `PaymentMethod`, `Payment`, `PaymentTransaction`, `Refund`, `RefundTransaction` |
| **Procurement** | UC-02 (Goods Receipt) | `BR-PO01` $\rightarrow$ `BR-PO06` | `FR-INV-03` | `DA-05` | - | `Supplier`, `PurchaseOrder`, `PurchaseOrderLine`, `GoodsReceipt`, `GoodsReceiptLine` |
| **Return & Refund** | UC-07 (Customer Return Request) | `BR-RT01` $\rightarrow$ `BR-RT23` | `FR-RET-01` $\rightarrow$ `FR-RET-04` | `DA-12`, `HDR-11` | `RET-Q01` $\rightarrow$ `RET-Q08` | `ReturnRequest`, `ReturnRequestLine`, `ReturnReceipt`, `ReturnReceiptLine`, `ReturnStatusHistory` |
| **Identity & Access** | UC Auth & RBAC | `BR-ID01` $\rightarrow$ `BR-ID14` | `FR-AUTH-01` $\rightarrow$ `FR-AUTH-04` | `DA-01`, `DA-06`, `HDR-12` | **BD-02**, `CID-Q01` $\rightarrow$ `CID-Q06` | `Account`, `Customer`, `Employee`, `Role`, `Permission`, `AccountRole`, `RolePermission`, `CustomerAddress`, `AccountVerification` |
| **Audit & Reporting** | UC Audit & Reporting | `BR-AR01` $\rightarrow$ `BR-AR20` | `FR-AUD-01`, `FR-REP-01` $\rightarrow$ `FR-REP-05` | `DA-13`, `DA-14`, `HDR-12` | `AR-Q01` $\rightarrow$ `AR-Q07` | `AuditLog`, `AuditLogDetail`, `ReportDefinition`, `ReportExecution`, `ReportSnapshot`, `ReportPermission` |

---

## 10.2. M1 Exit Criteria – Điều kiện Nghiệm thu Hoàn thành Milestone M1

| ID | Tiêu chí nghiệm thu | File | Đánh giá |
| :--- | :--- | :--- | :---: |
| **EC-01** | Project Charter xác định rõ mục tiêu và phạm vi | 01-Project-Charter.md| **Đạt** |
| **EC-02** | Xác định Stakeholder và ma trận phân quyền RBAC | 02-Stakeholder-Register.md | **Đạt** |
| **EC-03** | Use Case Diagram tổng thể và phân rã chi tiết | 03-Use-Case-Diagram.md | **Đạt** |
| **EC-04** | Đặc tả các Use Case cốt lõi (Checkout, Giữ kho, Nhập kho) | 04-Use-Case-Specifications.md | **Đạt** |
| **EC-05** | Sơ đồ Business Process (Sequence & State Machine) | 05-Business-Processes.md | **Đạt** |
| **EC-06** | Business Rules chuẩn hóa có mã định danh (v1.1) | 06-Business-Rules.md | **Đạt** |
| **EC-07** | Functional Requirements chi tiết theo phân hệ | 07-Functional-Requirements.md | **Đạt** |
| **EC-08** | Non-Functional Requirements (ACID, Concurrency, Security) | 08-Non-Functional-Requirements.md | **Đạt** |
| **EC-09** | Data Requirements (v1.1) & Constraints Baseline | 09-Data-Requirements.md | **Đạt** |
| **EC-10** | Traceability Matrix kết nối toàn diện | 10-Traceability-Matrix.md | **Đạt** |
| **EC-11** | Business Decisions Register (BD-01 đến BD-07) | 11-Business-Decisions.md | **Đạt** |

---

## 10.3. M1 – Bảng Quyết định Nghiệp vụ Đã Chốt Chính thức (Approved Decisions)

| Mã QĐ | Quyết định nghiệp vụ | Lựa chọn chính thức | Tác động cốt lõi đến M2 |
| :---: | :--- | :--- | :--- |
| **BD-01** | Sản phẩm – Danh mục | **Có, hỗ trợ nhiều danh mục ($N-N$)** kèm cờ `IsPrimary`. | Bảng `ProductCategory(ProductID, CategoryID, IsPrimary)`. |
| **BD-02** | Guest Checkout | **Bắt buộc đăng nhập tài khoản**. Không hỗ trợ Guest Checkout. | `Orders.CustomerID` là `NOT NULL`. Quản lý chuẩn `VoucherUsages`. |
| **BD-03** | Split Shipment | **Có, hỗ trợ chia đơn giao nhiều kho**. | `Order` tách khỏi `WarehouseID`. Sử dụng `OrderAllocation` và `Shipment`. |
| **BD-04** | Chính sách giữ hàng (Reservation) | **Giữ hàng ngay khi tạo đơn**, có thời hạn (`ExpiresAt` = 15 phút). | Bảng `StockReservation` (hoặc gắn với `OrderAllocation`) ngăn Overselling. |
| **BD-05** | Chính sách kết hợp khuyến mãi (Stacking) | **Phân tách 2 tầng độc lập:** Tối đa 1 Product Promo + 1 Order Voucher (nếu `IsStackable = 1`). | Cấm >1 chương trình cùng tầng. Bảng `OrderPromotions` lưu snapshot chiết khấu đối soát. |
| **BD-06** | Thuộc tính biến thể sản phẩm | **Mô hình thuộc tính động có kiểm soát** (`Attribute` $\rightarrow$ `AttributeValue` $\rightarrow$ `VariantAttributeValue`). | Linh hoạt cho cả Thời trang (Size/Màu) và Gia dụng (Kích thước/Chất liệu). |
| **BD-07** | Chính sách tồn kho khi vận chuyển & hoàn hàng | **Phân định 4 trạng thái tồn kho; Vòng đời 5 bước:** Xuất kho giảm đồng thời `OnHandQty` và `ReservedQty` (`SHIPMENT_OUT`). Giao thất bại chỉ nhập lại kho khi thực sự quay về (`DELIVERY_RETURN_IN`). | Bảo toàn tính toàn vẹn tồn kho trên kệ: `AvailableQty = OnHandQty - ReservedQty`. Hàng trên đường giao theo dõi qua `Shipment`. |

> **Kết luận chính thức nghiệm thu Milestone M1:** Toàn bộ $11/11$ tiêu chí nghiệm thu đã hoàn tất $100\%$. Milestone M1 chính thức khép lại, bàn giao đầy đủ bộ đặc tả làm đầu vào vững chắc cho **Milestone M2 – Conceptual Database Design**.
