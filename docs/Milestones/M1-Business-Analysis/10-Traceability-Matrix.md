# 10. TRACEABILITY MATRIX, EXIT CRITERIA & BUSINESS DECISION MAPPING

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.1 – Approved Mapping
* **Database Target:** Microsoft SQL Server

---

## 10.1. Ma trận Truy vết Toàn diện (Full Requirements Traceability Matrix)

Ma trận này kết nối xuyên suốt giữa **Miền nghiệp vụ**, **Use Cases**, **Business Rules (v1.1)**, **Functional Requirements**, **Data Requirements (v1.1)**, **Quyết định nghiệp vụ (BD-01 đến BD-07)** và **Thực thể CSDL dự kiến ở M2**:

| Miền nghiệp vụ | Use Case liên quan | Business Rules (v1.1) | Functional Requirements | Data Requirements (v1.1) | Business Decisions | Thực thể CSDL dự kiến (M2) |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **Product Catalog** | UC Product Management, UC Browse Catalog | `BR-PR01` $\rightarrow$ `BR-PR12` | `FR-CAT-01` $\rightarrow$ `FR-CAT-05` | `DA-02`, `DR-11` $\rightarrow$ `DR-14`, `HDR-01`, `HDR-13`, `HDR-14` | **BD-01**, **BD-06** | `Product`, `Category`, `ProductCategory`, `ProductVariant`, `Attribute`, `AttributeValue`, `ProductAttribute`, `VariantAttributeValue` |
| **Inventory & Warehouse** | UC-02, UC-03, UC-08 | `BR-IN01` $\rightarrow$ `BR-IN21` | `FR-INV-01` $\rightarrow$ `FR-INV-05` | `DA-03`, `DA-04`, `DR-15`, `DR-16`, `DR-19`, `HDR-05`, `HDR-15`, `HDR-17` | **BD-04**, **BD-07** | `Warehouse`, `InventoryBalance`, `StockReservation`, `StockMovement` |
| **Customer & Cart** | UC Customer & Cart Management | `BR-CU01` $\rightarrow$ `BR-CU07` | `FR-AUTH-01` $\rightarrow$ `FR-AUTH-04`, `FR-ORD-01` | `DA-06`, `DA-07`, `HDR-04` | **BD-02** | `Customer`, `CustomerAddress`, `Cart`, `CartLine` |
| **Sales & Order** | UC-01, UC-05 (Checkout & Order Lifecycle) | `BR-OR01` $\rightarrow$ `BR-OR12` | `FR-ORD-02` $\rightarrow$ `FR-ORD-05` | `DA-08`, `DR-15`, `HDR-02`, `HDR-03`, `HDR-06`, `HDR-16` | **BD-02**, **BD-03**, **BD-04** | `Order`, `OrderLine`, `OrderStatusHistory`, `OrderAllocation` |
| **Fulfillment & Shipping** | UC-05 (Shipping & Delivery Return) | `BR-SH01` $\rightarrow$ `BR-SH05`, `BR-IN17` $\rightarrow$ `BR-IN21` | `FR-SHP-01` $\rightarrow$ `FR-SHP-03` | `DA-11`, `DR-17`, `DR-18`, `DR-20`, `HDR-10`, `HDR-17` | **BD-03**, **BD-07** | `OrderAllocation`, `Shipment`, `ShipmentLine`, `ShippingProvider`, `ShipmentTrackingHistory`, `DeliveryReturnReceipt` |
| **Promotion & Voucher** | UC-06 (Apply Promotion & Voucher) | `BR-PM01` $\rightarrow$ `BR-PM14` | `FR-PRM-01` $\rightarrow$ `FR-PRM-02` | `DA-09`, `DR-21` $\rightarrow$ `DR-24`, `HDR-09`, `HDR-18` | **BD-05** | `Promotion`, `PromotionRule`, `Voucher`, `VoucherUsage`, `OrderPromotion` |
| **Payment & Refund** | UC-04 (Payment & Refund) | `BR-PA01` $\rightarrow$ `BR-PA07` | `FR-PAY-01` $\rightarrow$ `FR-PAY-02` | `DA-10`, `HDR-07`, `HDR-08` | - | `Payment`, `PaymentAttempt`, `Refund` |
| **Procurement** | UC-02 (Goods Receipt) | `BR-PO01` $\rightarrow$ `BR-PO06` | `FR-INV-03` | `DA-05` | - | `Supplier`, `PurchaseOrder`, `PurchaseOrderLine`, `GoodsReceipt`, `GoodsReceiptLine` |
| **Return & Refund** | UC-07 (Customer Return Request) | `BR-RT01` $\rightarrow$ `BR-RT06` | `FR-RET-01` $\rightarrow$ `FR-RET-04` | `DA-12`, `HDR-11` | - | `ReturnRequest`, `ReturnLine`, `ReturnStatusHistory` |
| **Identity & IAM** | UC Auth & RBAC | `BR-ID01` $\rightarrow$ `BR-ID05` | `FR-AUTH-01` $\rightarrow$ `FR-AUTH-04` | `DA-01`, `DA-13`, `HDR-12` | **BD-02** | `User`, `Role`, `Permission`, `UserRole`, `RolePermission`, `AuditLog` |

---

## 10.2. M1 Exit Criteria – Điều kiện Nghiệm thu Hoàn thành Milestone M1

| ID | Tiêu chí nghiệm thu | Kiểm chứng thực tế | Đánh giá |
| :--- | :--- | :--- | :---: |
| **EC-01** | Project Charter xác định rõ mục tiêu và phạm vi | Đã hoàn thiện [01-Project-Charter.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/01-Project-Charter.md) | **Đạt** |
| **EC-02** | Xác định Stakeholder và ma trận phân quyền RBAC | Đã hoàn thiện [02-Stakeholder-Register.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/02-Stakeholder-Register.md) | **Đạt** |
| **EC-03** | Use Case Diagram tổng thể và phân rã chi tiết | Đã hoàn thiện [03-Use-Case-Diagram.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/03-Use-Case-Diagram.md) | **Đạt** |
| **EC-04** | Đặc tả các Use Case cốt lõi (Checkout, Giữ kho, Nhập kho) | Đã hoàn thiện [04-Use-Case-Specifications.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/04-Use-Case-Specifications.md) | **Đạt** |
| **EC-05** | Sơ đồ Business Process (Sequence & State Machine) | Đã hoàn thiện [05-Business-Processes.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/05-Business-Processes.md) | **Đạt** |
| **EC-06** | Business Rules chuẩn hóa có mã định danh (v1.1) | Đã cập nhật [06-Business-Rules.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/06-Business-Rules.md) gồm BR-PR10..12, BR-CU07, BR-OR11..12, BR-IN13..21, BR-PM10..14 | **Đạt** |
| **EC-07** | Functional Requirements chi tiết theo phân hệ | Đã hoàn thiện [07-Functional-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/07-Functional-Requirements.md) | **Đạt** |
| **EC-08** | Non-Functional Requirements (ACID, Concurrency, Security) | Đã hoàn thiện [08-Non-Functional-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/08-Non-Functional-Requirements.md) | **Đạt** |
| **EC-09** | Data Requirements (v1.1) & Constraints Baseline | Đã cập nhật [09-Data-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/09-Data-Requirements.md) gồm DR-11..24, HDR-13..18 | **Đạt** |
| **EC-10** | Traceability Matrix kết nối toàn diện | Đã cập nhật ánh xạ 2 chiều đầy đủ tại tài liệu này | **Đạt** |
| **EC-11** | Business Decisions Register (BD-01 đến BD-07) | Đã chính thức hóa tại [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md) với phân tích sâu BD-05 & BD-07 | **Đạt** |

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
