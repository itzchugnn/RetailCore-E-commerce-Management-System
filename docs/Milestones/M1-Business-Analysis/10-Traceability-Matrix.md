# 10. TRACEABILITY MATRIX, EXIT CRITERIA & BUSINESS DECISION MAPPING (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis (Đồng bộ 2 chiều với 79 Thực thể M2)
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT Mapping
* **Database Target:** Microsoft SQL Server 2022+

---

## 10.1. Ma trận Truy vết Yêu cầu Toàn diện (Full Requirements Traceability Matrix)

Ma trận kết nối xuyên suốt 100% giữa **11 Miền nghiệp vụ**, **Use Cases**, **215 Business Rules (`M1.06`)**, **55 Functional Requirements (`M1.07`)**, **30 Data Requirements (`M1.09`)**, **Quyết định `BD-01..07` / `CORR-01..10`** và **79 Thực thể M2 (`E01` – `E79`)**:

| Miền nghiệp vụ | Use Cases | Business Rules (`M1.06`) | Functional Requirements (`M1.07`) | Data Requirements (`M1.09`) | Quyết định & Hiệu chỉnh | Thực thể Chuẩn M2 (`E01` – `E79`) |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **SA01. Product Catalog** | UC Catalog & SKU Mgmt | `BR-PR01` $\to$ `BR-PR15` | `FR-CAT-01` $\to$ `FR-CAT-05` | `DR-02` $\to$ `DR-06`, `DR-11` $\to$ `DR-14`, `HDR-01`, `HDR-02` | `BD-01`, `BD-06`, `PC-Q01..06` | `Product` (E01), `Category` (E02), `ProductCategory` (E03), `ProductVariant` (E04), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09), `Brand` (E73) |
| **SA02. Warehouse & Inventory** | UC-02, UC-03, UC-08 | `BR-IN01` $\to$ `BR-IN25` | `FR-INV-01` $\to$ `FR-INV-05` | `DR-15`, `DR-16`, `DR-19`, `DR-25`, `DR-26`, `HDR-05`, `HDR-06` | `BD-03`, `BD-04`, `BD-07`, `CORR-02`, `CORR-08`, `CORR-10` | `Warehouse` (E10), `InventoryBalance` (E11), `OrderWarehouseAllocation` (E12), `StockReservation` (E13), `StockReservationLine` (E14), `InventoryTransaction` (E15), `InventoryTransactionLine` (E16), `StockCount` (E17), `StockCountLine` (E18), `WarehouseTransfer` (E19), `WarehouseTransferLine` (E20) |
| **SA03. Procurement** | UC-02 (PO & Goods Receipt) | `BR-PO01` $\to$ `BR-PO10` | `FR-PO-01` $\to$ `FR-PO-05` | `DR-27`, `HDR-04` | `PR-Q01..06`, `CORR-08` | `Supplier` (E21), `SupplierProduct` (E22), `PurchaseOrder` (E23), `PurchaseOrderLine` (E24), `GoodsReceipt` (E25), `GoodsReceiptLine` (E26), `SupplierReturn` (E27), `SupplierReturnLine` (E28) |
| **SA04. Customer & Identity** | UC Auth, Profile & RBAC | `BR-ID01` $\to$ `BR-ID15` | `FR-AUTH-01` $\to$ `FR-AUTH-04` | `DR-01`, `DR-07` $\to$ `DR-09` | `BD-02`, `CORR-04`, `CID-Q01..06` | `Account` (E29), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `AccountVerification` (E37) |
| **SA05. Shopping** | UC Shopping Cart | `BR-SH01` $\to$ `BR-SH15` | `FR-CRT-01` $\to$ `FR-CRT-04` | `DR-10` | `SHOP-Q01..06` | `ShoppingCart` (E38), `CartItem` (E39) |
| **SA06. Sales & Order** | UC-01, UC-05 | `BR-SA01` $\to$ `BR-SA20` | `FR-ORD-01` $\to$ `FR-ORD-06` | `DR-23`, `HDR-01` $\to$ `HDR-03`, `HDR-07` | `BD-02..04`, `CORR-03`, `CORR-04`, `CORR-06`, `CORR-09` | `SalesOrder` (E40), `OrderLine` (E41), `OrderAddress` (E42), `OrderStatusHistory` (E43) |
| **SA07. Fulfillment & Shipping** | UC-05 (Shipping & Return In) | `BR-FS01` $\to$ `BR-FS22` | `FR-SHP-01` $\to$ `FR-SHP-05` | `DR-17`, `DR-18`, `DR-20`, `HDR-04`, `HDR-07` | `BD-03`, `BD-07`, `CORR-06`, `CORR-08` | `Carrier` (E44), `Shipment` (E45), `ShipmentLine` (E46), `DeliveryAttempt` (E47), `ShipmentStatusHistory` (E48), `DeliveryReturnReceipt` (E75) |
| **SA08. Promotion & Voucher** | UC-06 (Promotion & Voucher) | `BR-PRM01` $\to$ `BR-PRM24` | `FR-PRM-01` $\to$ `FR-PRM-05` | `DR-21` $\to$ `DR-24`, `HDR-10` | `BD-05`, `CORR-03`, `PROMO-Q01..08` | `Promotion` (E49), `PromotionCondition` (E50), `PromotionProduct` (E51), `Voucher` (E52), `PromotionUsage` (E53), `PromotionTargetCustomer` (E54) |
| **SA09. Payment & Settlement** | UC-04, UC-09 (Payment, COD & Refund) | `BR-PY01` $\to$ `BR-PY25` | `FR-PAY-01` $\to$ `FR-PAY-05` | `DR-28`, `HDR-08`, `HDR-09` | `CORR-04` $\to$ `CORR-08`, `PAY-Q01..10` | `PaymentMethod` (E55), `Payment` (E56), `PaymentTransaction` (E57), `Refund` (E58), `RefundTransaction` (E59), `PaymentWebhookLog` (E71), `IdempotencyRecord` (E74), `CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79) |
| **SA10. Return & Exchange** | UC-07 (RMA & KCS Assessment) | `BR-RT01` $\to$ `BR-RT24` | `FR-RET-01` $\to$ `FR-RET-05` | `DR-29`, `HDR-07`, `HDR-11` | `CORR-01`, `CORR-03`, `CORR-10`, `RET-Q01..08` | `ReturnRequest` (E60), `ReturnRequestLine` (E61), `ReturnReceipt` (E62), `ReturnReceiptLine` (E63), `ReturnStatusHistory` (E64), `ReturnLineAssessment` (E72), `ReturnShipmentAllocation` (E76) |
| **SA11. Audit & Reporting** | UC Audit & Reporting | `BR-AR01` $\to$ `BR-AR20` | `FR-AUD-01`, `FR-AUD-02`, `FR-REP-01` $\to$ `FR-REP-04` | `DR-30`, `HDR-12` | `AR-Q01..07` | `AuditLog` (E65), `AuditLogDetail` (E66), `ReportDefinition` (E67), `ReportExecution` (E68), `ReportSnapshot` (E69), `ReportPermission` (E70) |

---

## 10.2. Điều kiện Nghiệm thu Hoàn thành Milestone M1 (Exit Criteria)

| ID | Tiêu chí nghiệm thu | Tài liệu bàn giao | Đánh giá |
| :--- | :--- | :--- | :---: |
| **EC-01** | Project Charter xác định rõ mục tiêu và phạm vi | `01-Project-Charter.md` | **Đạt** |
| **EC-02** | Xác định Stakeholder và ma trận trách nhiệm | `02-Stakeholder-Register.md` | **Đạt** |
| **EC-03** | Use Case Diagram tổng thể và phân rã chi tiết | `03-Use-Case-Diagram.md` | **Đạt** |
| **EC-04** | Đặc tả các Use Case cốt lõi (Checkout, Giữ kho, Nhập kho) | `04-Use-Case-Specifications.md` | **Đạt** |
| **EC-05** | Sơ đồ luồng quy trình nghiệp vụ cốt lõi (Flowcharts) | `05-Business-Processes.md` | **Đạt** |
| **EC-06** | Bộ 215 Business Rules định danh duy nhất không trùng lặp (v2.0) | `06-Business-Rules.md` | **Đạt** |
| **EC-07** | Bộ 55 Functional Requirements đồng bộ mã định danh | `07-Functional-Requirements.md` | **Đạt** |
| **EC-08** | Non-Functional Requirements định lượng (ACID, Concurrency, RPO/RTO) | `08-Non-Functional-Requirements.md` | **Đạt** |
| **EC-09** | Bộ 30 Data Requirements (`DR-01..DR-30`) & Constraints Baseline | `09-Data-Requirements.md` | **Đạt** |
| **EC-10** | Traceability Matrix hai chiều không đứt gãy mã tham chiếu | `10-Traceability-Matrix.md` | **Đạt** |
| **EC-11** | Business Decisions Register (`BD-01..07` & Reverse-Sync `CORR-01..10`) | `11-Business-Decisions.md` | **Đạt** |

---

## 10.3. Bảng Tổng hợp 7 Quyết định Nghiệp vụ Đã Chốt (`BD-01` – `BD-07`)

| Mã QĐ | Quyết định nghiệp vụ | Lựa chọn chính thức | Thực thể & Cơ chế Cốt lõi ở M2–M3 |
| :---: | :--- | :--- | :--- |
| **BD-01** | Sản phẩm – Danh mục | Hỗ trợ nhiều danh mục ($N:N$), bắt buộc đúng 1 danh mục chính khi `PUBLISHED`. | `ProductCategory(ProductID, CategoryID, IsPrimary)` + `UX_ProductCategory_Primary` + SP/Trigger Publish. |
| **BD-02** | Guest Checkout | Cấm Guest Checkout; bắt buộc đăng nhập tài khoản trước khi đặt đơn. | `SalesOrder.CustomerID NOT NULL` tham chiếu `Customer(AccountID)`. |
| **BD-03** | Split Shipment | Hỗ trợ chia đơn/dòng đơn xuất từ nhiều kho; tách doanh thu phí ship khách trả với cước hãng. | `OrderWarehouseAllocation` (E12), `Shipment` (E45), `ShipmentLine` (E46). |
| **BD-04** | Stock Reservation | Giữ tồn logic All-or-Nothing ngay khi tạo đơn với thời hạn `ExpiresAt` (15p chuẩn / 5p Flash Sale). | `StockReservation` (E13), `StockReservationLine` (E14) ($1:0..N$ với `AllocationID`), `PromotionProduct` (E51). |
| **BD-05** | Promotion Stacking | Tối đa 1 ưu đãi `PRODUCT` trên dòng + 1 `Voucher` `ORDER` + 1 `Voucher` `SHIPPING` (Freeship) trên đơn khi `IsStackable = 1`. | `Promotion` (E49), `Voucher` (E52), `PromotionUsage` (E53) + phân bổ LRM (`CORR-03`). |
| **BD-06** | Thuộc tính động 2 lớp | Phân tách vai trò `'VARIANT_DEFINING'` và `'DESCRIPTIVE'` trên từng sản phẩm. | `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09). |
| **BD-07** | Tồn kho giao vận & giao thất bại | Xuất bãi giảm đồng thời `OnHandQty` & `ReservedQty`; giao thất bại chỉ cộng tồn khi ký `DeliveryReturnReceipt` (E75). | `InventoryBalance` (E11) (3 phẩm chất `SELLABLE`, `QUARANTINED`, `DAMAGED`), `InventoryTransactionLine` (E16), `DeliveryReturnReceipt` (E75). |
