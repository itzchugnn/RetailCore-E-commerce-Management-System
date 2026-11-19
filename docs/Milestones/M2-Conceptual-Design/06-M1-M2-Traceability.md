# M2.06 - TRACEABILITY MATRIX & MILESTONE 2 CLOSURE REPORT (MA TRẬN TRUY VẾT & BIÊN BẢN NGHIỆM THU ĐÓNG MILESTONE 2)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.0 – Official Final Release
* **Đơn vị thẩm định:** Lead Enterprise Database Engineer & Solution Architect
* **Hệ quản trị CSDL mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Văn bản căn cứ:**
  * Milestone 1 Baseline: [01-Project-Charter.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/01-Project-Charter.md), [06-Business-Rules.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/06-Business-Rules.md), [07-Functional-Requirements.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/07-Functional-Requirements.md), [10-Traceability-Matrix.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/10-Traceability-Matrix.md), [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md).
  * Milestone 2 Deliverables: [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md), [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md), [03-Cardinality-and-Constraints.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/03-Cardinality-and-Constraints.md), [04-Conceptual-ERD.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/04-Conceptual-ERD.md), [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md).

---

## 1. TỔNG QUAN MỤC TIÊU NGHIỆM THU MILESTONE M2

Tài liệu này xác lập ma trận truy vết hai chiều (Bidirectional Traceability Matrix) hoàn chỉnh 100% giữa các yêu cầu nghiệp vụ, quy tắc nghiệp vụ, quyết định kiến trúc từ **Milestone 1 (Business Analysis)** với toàn bộ **70 Thực thể chuẩn hóa (E01 đến E70)**, sơ đồ quan hệ và các ràng buộc toàn vẹn của **Milestone 2 (Conceptual Database Design)**.

Văn bản này đồng thời đóng vai trò là **Biên bản nghiệm thu kỹ thuật chính thức** xác nhận việc đóng Milestone 2 và chuyển giao toàn diện sang **Milestone 3 (Logical Database Design)**.

---

## 2. TRUY VẾT 7 QUYẾT ĐỊNH NGHIỆP VỤ NỀN TẢNG (BD-01 ĐẾN BD-07)

| Mã QĐ | Nội dung Quyết định nghiệp vụ M1 | Lựa chọn kiến trúc chốt | Thực thể M2 chịu trách nhiệm | Ràng buộc & Cơ chế kỹ thuật M2 | Đánh giá |
| :---: | :--- | :--- | :--- | :--- | :---: |
| **BD-01** | Quan hệ Sản phẩm – Danh mục đa tầng | Hỗ trợ nhiều danh mục ($N:N$), bắt buộc có đúng 1 danh mục chính (`IsPrimary = 1`) khi Published. | `Product` (E01), `Category` (E02), `ProductCategory` (E03). | Quan hệ đệ quy `ParentCategoryID`. Filtered Unique Index: `(ProductID) WHERE IsPrimary = 1`. Quyết định `PC-Q01`, `PC-Q02`. | **100% Đạt** |
| **BD-02** | Bắt buộc Đăng nhập Tài khoản (No Guest Checkout) | Không hỗ trợ Guest Checkout. Khách hàng bắt buộc sở hữu tài khoản xác thực trước khi tạo đơn. | `Account` (E29), `Customer` (E30), `SalesOrder` (E40). | Ràng buộc `SalesOrder.CustomerID NOT NULL`. Quan hệ XOR `Account` $\leftrightarrow$ `Customer`/`Employee`. Quyết định `CID-Q01`, `SALES-Q01`. | **100% Đạt** |
| **BD-03** | Hỗ trợ Chia đơn giao nhiều kho (Split Shipment) | Một đơn hàng có thể phân tách xuất kho từ nhiều kho vật lý khác nhau; mỗi kiện hàng xuất từ 1 kho duy nhất. | `OrderWarehouseAllocation` (E12), `Shipment` (E45), `ShipmentLine` (E46). | Tách `SalesOrder` khỏi `WarehouseID`. Bổ sung thực thể trung gian `OrderWarehouseAllocation`. $\sum \text{AllocatedQty} = \text{OrderedQty}$. Quyết định `SHIP-Q01`, `WI-Q03`. | **100% Đạt** |
| **BD-04** | Chính sách Khóa giữ Tồn kho (Reservation & Timeout) | Giữ tồn logic ngay khi tạo đơn với thời hạn `ExpiresAt` (chuẩn 15 phút). Cơ chế All-or-Nothing. | `StockReservation` (E13), `StockReservationLine` (E14), `InventoryBalance` (E11). | Công thức $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$. Giao dịch tạo đơn và giữ hàng là nguyên tử. Hết hạn tự động Release. Quyết định `WI-Q02`, `SALES-Q01`. | **100% Đạt** |
| **BD-05** | Cấu trúc Khuyến mãi & Stacking Hai tầng | Tối đa 1 khuyến mãi cấp SKU (`OrderLine`) + 1 voucher cấp đơn (`SalesOrder`). Kết hợp khi `IsStackable = 1`. | `Promotion` (E49), `Voucher` (E52), `PromotionProduct` (E51), `PromotionUsage` (E53). | Cấm >1 CTKM cùng tầng. Snapshot mức giảm thực tế vào `PromotionUsage`. Quản lý hạn mức `MaxUsage`, `MaxUsagePerCustomer`. Quyết định `PROMO-Q01..08`. | **100% Đạt** |
| **BD-06** | Mô hình Thuộc tính Biến thể Động có kiểm soát | Phân định rõ vai trò thuộc tính (`DESCRIPTIVE` / `VARIANT_DEFINING`), quản lý whitelist giá trị theo sản phẩm. | `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09). | Từ điển thuộc tính dùng chung. Whitelist giá trị được phép trên từng sản phẩm. Bảo đảm SKU bất biến lịch sử. Quyết định `PC-Q04..06`. | **100% Đạt** |
| **BD-07** | Phân định 4 Trạng thái Tồn kho & Vòng đời 5 bước | Phân tách rạch ròi 4 khái niệm tồn kho. Giao thất bại chỉ nhập lại kho khi thủ kho ký nhận hàng hoàn về thực tế. | `InventoryBalance` (E11), `OrderWarehouseAllocation` (E12), `Shipment` (E45), `InventoryTransaction` (E15), `InventoryTransactionLine` (E16). | Sổ cái ghi sổ hai chiều Dual-Delta (`OnHandDelta`, `ReservedDelta`). Hàng In-Transit và hàng giao thất bại không nằm trong `OnHandQty` của bất kỳ kho nào cho đến khi ký `DELIVERY_RETURN_IN`. Quyết định `WI-Q02`, `SHIP-Q04`. | **100% Đạt** |

---

## 3. TRUY VẾT 55 YÊU CẦU CHỨC NĂNG (FUNCTIONAL REQUIREMENTS)

| Mã Phân hệ | Mã FR | Tên Yêu cầu Chức năng | Use Cases liên quan | Thực thể M2 thực thi | Mối quan hệ & Ràng buộc M2 đáp ứng | Đánh giá |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **01. Catalog** | `FR-CAT-01` | Quản lý danh mục đa cấp | UC Browse Catalog | `Category` (E02) | Quan hệ đệ quy `ParentCategoryID`, `Slug` UK. | **Đạt** |
| | `FR-CAT-02` | Quản lý sản phẩm & gán danh mục | UC Product Mgmt | `Product` (E01), `ProductCategory` (E03) | Quan hệ $N:N$, cờ `IsPrimary`, Filtered Unique Index. | **Đạt** |
| | `FR-CAT-03` | Quản lý thuộc tính động & Whitelist | UC Product Mgmt | `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08) | `AttributeRole` phân định rõ vai trò, Whitelist ràng buộc giá trị hợp lệ. | **Đạt** |
| | `FR-CAT-04` | Quản lý biến thể SKU & giá bán | UC Product Mgmt | `ProductVariant` (E04), `VariantAttributeValue` (E09) | `SKU` Unique toàn hệ thống, snapshot giá bán `SellingPrice`. | **Đạt** |
| | `FR-CAT-05` | Tra cứu & Lọc sản phẩm (Faceted Search)| UC Browse Catalog | `ProductVariant` (E04), `VariantAttributeValue` (E09), `Category` (E02) | Chỉ mục đa chiều theo thuộc tính và danh mục. | **Đạt** |
| **02. Warehouse**| `FR-INV-01` | Quản lý mạng lưới kho vật lý | UC Warehouse Mgmt | `Warehouse` (E10) | `WarehouseCode` UK, quản lý trạng thái kho. | **Đạt** |
| | `FR-INV-02` | Quản lý số dư tồn kho tức thời | UC Stock Inquiry | `InventoryBalance` (E11) | $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} \ge 0$. | **Đạt** |
| | `FR-INV-03` | Khóa giữ tồn kho & Timeout | UC-01 Checkout | `StockReservation` (E13), `StockReservationLine` (E14) | Khóa All-or-Nothing, `ExpiresAt` tự động release. | **Đạt** |
| | `FR-INV-04` | Điều chuyển kho 2 bước In-Transit | UC Transfer Stock | `WarehouseTransfer` (E19), `WarehouseTransferLine` (E20) | Trạng thái In-Transit, cách ly hàng trên đường chuyển. | **Đạt** |
| | `FR-INV-05` | Kiểm kê & Duyệt chênh lệch | UC Stock Count | `StockCount` (E17), `StockCountLine` (E18), `InventoryTransaction` (E15) | Phân tách nhiệm vụ (`ApprovedBy <> CreatedBy`), hạch toán qua sổ cái. | **Đạt** |
| **03. Procurement**| `FR-PO-01` | Quản lý hồ sơ nhà cung cấp | UC Supplier Mgmt | `Supplier` (E21), `SupplierProduct` (E22) | `SupplierCode` UK, ma trận cung ứng SKU `(SupplierID, VariantID)`. | **Đạt** |
| | `FR-PO-02` | Quản lý đơn đặt mua hàng (PO) | UC Create PO | `PurchaseOrder` (E23), `PurchaseOrderLine` (E24) | PO chỉ định 1 kho đích duy nhất (`DestinationWarehouseID`). | **Đạt** |
| | `FR-PO-03` | Tiếp nhận hàng nhiều đợt | UC-02 Goods Receipt | `GoodsReceipt` (E25), `GoodsReceiptLine` (E26) | Kho nhập trùng kho PO; $\sum \text{ReceivedQty} \le \text{OrderedQty}$. | **Đạt** |
| | `FR-PO-04` | Kiểm định chất lượng hàng nhập | UC-02 Goods Receipt | `GoodsReceiptLine` (E26) | $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$. | **Đạt** |
| | `FR-PO-05` | Xuất trả hàng nhà cung cấp | UC Return to Vendor | `SupplierReturn` (E27), `SupplierReturnLine` (E28) | Đối soát chứng từ tiếp nhận gốc `GoodsReceiptLineID`. | **Đạt** |
| **04. Identity** | `FR-AUTH-01`| Đăng ký & Xác thực người dùng | UC Register/Login | `Account` (E29), `AccountVerification` (E37) | Filtered Unique Email/Phone, mã hóa băm `PasswordHash`. | **Đạt** |
| | `FR-AUTH-02`| Quản lý hồ sơ khách hàng | UC Manage Profile | `Customer` (E30), `CustomerAddress` (E36) | Phụ thuộc 1-1 với Account (XOR), tối đa 1 địa chỉ mặc định. | **Đạt** |
| | `FR-AUTH-03`| Quản lý nhân sự nội bộ | UC Manage Employee| `Employee` (E31) | Phụ thuộc 1-1 với Account (XOR), `EmployeeCode` UK. | **Đạt** |
| | `FR-AUTH-04`| Phân quyền vai trò động (RBAC) | UC RBAC Mgmt | `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35) | Ma trận phân quyền động $N:N$ linh hoạt. | **Đạt** |
| **05. Shopping** | `FR-CRT-01` | Tạo & Duy trì giỏ hàng | UC Shopping Cart | `ShoppingCart` (E38) | Duy nhất 1 giỏ `ACTIVE` per Customer (Filtered Index). | **Đạt** |
| | `FR-CRT-02` | Thêm/sửa mặt hàng trong giỏ | UC Shopping Cart | `CartItem` (E39) | Tự động cộng dồn số lượng, không giữ tồn kho trước. | **Đạt** |
| | `FR-CRT-03` | Cảnh báo SKU không khả dụng | UC Shopping Cart | `CartItem` (E39), `InventoryBalance` (E11) | Kiểm tra thời gian thực trước khi tiến hành Checkout. | **Đạt** |
| | `FR-CRT-04` | Chuyển đổi giỏ hàng thành đơn | UC-01 Checkout | `ShoppingCart` (E38), `SalesOrder` (E40) | Đóng băng `CONVERTED`, sinh đơn hàng nguyên tử. | **Đạt** |
| **06. Sales** | `FR-ORD-01` | Tiếp nhận & Xác thực đơn hàng | UC-01 Checkout | `SalesOrder` (E40), `OrderLine` (E41) | Khóa giữ tồn kho All-or-Nothing trước khi tạo đơn. | **Đạt** |
| | `FR-ORD-02` | Snapshot bất biến dữ liệu bán | UC-01 Checkout | `OrderLine` (E41), `OrderAddress` (E42) | Snapshot nguyên trạng giá, tên, thuộc tính JSON và địa chỉ nhận. | **Đạt** |
| | `FR-ORD-03` | Quản lý vòng đời trạng thái đơn | UC Order Lifecycle | `OrderStatusHistory` (E43) | Lưu vết chuyển đổi trạng thái bất biến (Append-Only). | **Đạt** |
| | `FR-ORD-04` | Phân bổ xuất kho chia đơn | UC Order Allocation| `OrderWarehouseAllocation` (E12) | $\sum \text{AllocatedQty} = \text{OrderedQty}$ (Split Shipment). | **Đạt** |
| | `FR-ORD-05` | Hủy đơn & Giải phóng tồn kho | UC Cancel Order | `SalesOrder` (E40), `StockReservation` (E13) | Hoàn trả quota voucher, giải phóng reservation logic. | **Đạt** |
| | `FR-ORD-06` | Cấm xóa vật lý chứng từ đơn | UC Audit Compliance| `SalesOrder` (E40) | Ràng buộc cấm hard delete, chỉ cập nhật `CANCELLED`. | **Đạt** |
| **07. Shipping** | `FR-SHP-01` | Tạo kiện hàng xuất từ 1 kho | UC-05 Shipping | `Shipment` (E45), `ShipmentLine` (E46) | Mỗi Shipment chỉ xuất phát từ 1 kho vật lý duy nhất. | **Đạt** |
| | `FR-SHP-02` | Gán đối tác vận chuyển & Tracking | UC-05 Shipping | `Carrier` (E44), `Shipment` (E45) | Lưu vết `TrackingNumber` và đối soát cước vận chuyển. | **Đạt** |
| | `FR-SHP-03` | Giới hạn số lần giao & Chuyển hoàn| UC Delivery Tracking| `DeliveryAttempt` (E47) | Tối đa 3 lần giao, vượt hạn mức tự động chuyển sang `RETURNING`. | **Đạt** |
| | `FR-SHP-04` | Nhập hoàn kho khi giao thất bại | UC-05 Shipping | `Shipment` (E45), `InventoryTransaction` (E15) | Chỉ tăng tồn khi kho thực nhận hàng hoàn (`DELIVERY_RETURN_IN`). | **Đạt** |
| | `FR-SHP-05` | Lưu vết hành trình kiện hàng | UC Delivery Tracking| `ShipmentStatusHistory` (E48) | Cập nhật tự động qua Webhook và nhân viên bưu tá. | **Đạt** |
| **08. Promotion**| `FR-PRM-01`| Cấu hình chương trình khuyến mãi | UC-06 Promotion | `Promotion` (E49), `PromotionCondition` (E50) | Cấu hình điều kiện định sẵn, không dùng biểu thức tự do. | **Đạt** |
| | `FR-PRM-02`| Chiết khấu sản phẩm theo SKU | UC-06 Promotion | `PromotionProduct` (E51) | Áp dụng ưu đãi chính xác theo danh sách biến thể `ProductVariant`. | **Đạt** |
| | `FR-PRM-03`| Phát hành & Quản lý Voucher | UC-06 Promotion | `Voucher` (E52) | Phân định voucher công khai và voucher gán riêng cho khách. | **Đạt** |
| | `FR-PRM-04`| Kiểm soát Stacking & Thứ tự ưu tiên| UC-06 Promotion | `Promotion` (E49), `PromotionUsage` (E53) | Phân tầng 2 lớp, cờ `IsStackable`, thứ tự `Priority`. | **Đạt** |
| | `FR-PRM-05`| Quản lý hạn mức sử dụng & Hoàn mã | UC-06 Promotion | `Promotion` (E49), `Voucher` (E52), `PromotionUsage` (E53) | Quản lý `MaxUsage`, `MaxUsagePerCustomer`, hoàn quota khi hủy đơn. | **Đạt** |
| **09. Payment** | `FR-PAY-01` | Quản lý phương thức thanh toán | UC-04 Payment | `PaymentMethod` (E55) | Hỗ trợ đa dạng: COD, Thẻ, Chuyển khoản, Ví điện tử. | **Đạt** |
| | `FR-PAY-02` | Ghi nhận hồ sơ thanh toán đơn | UC-04 Payment | `Payment` (E56) | Quan hệ $1:1$ bắt buộc với `SalesOrder`. | **Đạt** |
| | `FR-PAY-03` | Nhật ký giao dịch cổng & Idempotency| UC-04 Payment | `PaymentTransaction` (E57) | Append-only, chống trùng lặp Webhook qua `TransactionReference`. | **Đạt** |
| | `FR-PAY-04` | Xử lý yêu cầu hoàn tiền (Refund)| UC-04 Refund | `Refund` (E58), `RefundTransaction` (E59) | Phê duyệt nội bộ, $\sum \text{Refund} \le \text{Paid} - \text{Refunded}$. | **Đạt** |
| | `FR-PAY-05` | Đối soát COD và phí giao dịch | UC-04 Settlement | `PaymentTransaction` (E57) | Lưu chi tiết phí cổng, hạch toán đối soát với bên thứ ba. | **Đạt** |
| **10. Return** | `FR-RET-01` | Tiếp nhận yêu cầu đổi trả (RMA) | UC-07 Return RMA | `ReturnRequest` (E60), `ReturnRequestLine` (E61) | Khách tự tạo hoặc CSKH tạo thay, trong thời hạn chính sách. | **Đạt** |
| | `FR-RET-02` | Phê duyệt RMA & Chỉ định kho nhận | UC-07 Return RMA | `ReturnRequest` (E60) | Quy trình duyệt 2 bước, chỉ định duy nhất 1 kho nhận cố định. | **Đạt** |
| | `FR-RET-03` | Tiếp nhận & Kiểm định hàng trả | UC-07 Return RMA | `ReturnReceipt` (E62), `ReturnReceiptLine` (E63) | Phân loại `Disposition`: SELLABLE, QUARANTINE, DISPOSED. | **Đạt** |
| | `FR-RET-04` | Kích hoạt hoàn tiền sau kiểm định | UC-07 Return RMA | `ReturnReceipt` (E62), `Refund` (E58) | Chỉ hoàn tiền khi kho đã ký nhận và hàng đạt chuẩn SELLABLE. | **Đạt** |
| | `FR-RET-05` | Lưu vết lịch sử xử lý khiếu nại | UC-07 Return RMA | `ReturnStatusHistory` (E64) | Lưu vết thời gian xử lý phục vụ kiểm toán và SLA. | **Đạt** |
| **11. Audit** | `FR-AUD-01` | Ghi nhật ký kiểm toán chọn lọc | UC Audit Trail | `AuditLog` (E65), `AuditLogDetail` (E66) | Append-only, ghi log hành động nhạy cảm, không lưu data mật. | **Đạt** |
| | `FR-AUD-02` | Chính sách lưu trữ & Archive log | UC Compliance | `AuditLog` (E65) | Cấu hình `RetentionUntil`, phân vùng lưu trữ theo năm. | **Đạt** |
| | `FR-REP-01` | Định nghĩa & Quản lý mẫu báo cáo | UC Reporting | `ReportDefinition` (E67) | Danh mục báo cáo nghiệp vụ và quản trị. | **Đạt** |
| | `FR-REP-02` | Thực thi truy vấn & Xuất file | UC Reporting | `ReportExecution` (E68) | Lưu vết tham số lọc, tài khoản thực thi và đường dẫn file. | **Đạt** |
| | `FR-REP-03` | Chốt kỳ báo cáo cố định (Snapshot)| UC Financial Period | `ReportSnapshot` (E69) | Bản chụp số liệu kế toán không bị biến động hồi tố. | **Đạt** |
| | `FR-REP-04` | Phân quyền báo cáo đa cấp | UC Report Security | `ReportPermission` (E70) | Phân quyền ma trận theo Role: `CanView`, `CanExport`, `CanSnapshot`. | **Đạt** |

---

## 4. TRUY VẾT CÁC QUY TẮC NGHIỆP VỤ CỐT LÕI (CORE BUSINESS RULES TRACEABILITY)

Hệ thống đã rà soát và ánh xạ toàn bộ gần 200 Business Rules vào cấu trúc mô hình dữ liệu:

* **Catalog (`BR-PR01` đến `BR-PR12`):** Ánh xạ trực tiếp vào `Product`, `Category`, `ProductVariant`, `ProductAttribute`, `ProductAttributeValue`, `VariantAttributeValue`. Bắt buộc `IsPrimary` danh mục chính, cấm SKU mồ côi, duy nhất SKU toàn hệ thống.
* **Inventory (`BR-IN01` đến `BR-IN21`):** Ánh xạ vào `Warehouse`, `InventoryBalance`, `OrderWarehouseAllocation`, `StockReservation`, `InventoryTransaction`. Công thức tồn khả dụng, Dual-Delta ledger, kiểm kê kho phân tách trách nhiệm.
* **Procurement (`BR-PO01` đến `BR-PO06`):** Ánh xạ vào `Supplier`, `PurchaseOrder`, `GoodsReceipt`, `SupplierReturn`. Một PO một kho nhận cố định, $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$.
* **Identity & Access (`BR-ID01` đến `BR-ID14`):** Ánh xạ vào `Account`, `Customer`, `Employee`, `Role`, `Permission`, `CustomerAddress`. XOR loại trừ giữa Customer và Employee, Filtered Unique Email/Phone, tối đa 1 địa chỉ mặc định.
* **Shopping (`BR-SH01` đến `BR-SH15`):** Ánh xạ vào `ShoppingCart`, `CartItem`. Duy nhất 1 giỏ `ACTIVE`, cộng dồn số lượng, không giữ tồn kho trước, đóng băng `CONVERTED`.
* **Sales Order (`BR-SA01` đến `BR-SA18`):** Ánh xạ vào `SalesOrder`, `OrderLine`, `OrderAddress`, `OrderStatusHistory`. Tạo đơn nguyên tử (All-or-Nothing), snapshot bất biến dữ liệu bán, độc lập máy trạng thái, cấm xóa cứng (No Hard Delete).
* **Shipping (`BR-FS01` đến `BR-FS22`):** Ánh xạ vào `Carrier`, `Shipment`, `ShipmentLine`, `DeliveryAttempt`, `ShipmentStatusHistory`. Mỗi kiện từ 1 kho, tối đa 3 lần giao, giao thất bại không tự tăng tồn.
* **Promotion (`BR-PRM01` đến `BR-PRM24`):** Ánh xạ vào `Promotion`, `PromotionCondition`, `PromotionProduct`, `Voucher`, `PromotionUsage`. Cấu trúc khuyến mãi 2 tầng, kiểm soát hạn mức `MaxUsage`, `MaxUsagePerCustomer`.
* **Payment (`BR-PY01` đến `BR-PY22`):** Ánh xạ vào `PaymentMethod`, `Payment`, `PaymentTransaction`, `Refund`, `RefundTransaction`. Mỗi đơn 1 hồ sơ thanh toán ($1:1$), đối soát cổng, hạn mức hoàn tiền $\le \text{Paid} - \text{Refunded}$.
* **Return (`BR-RT01` đến `BR-RT23`):** Ánh xạ vào `ReturnRequest`, `ReturnRequestLine`, `ReturnReceipt`, `ReturnReceiptLine`, `ReturnStatusHistory`. Duyệt 2 bước, chỉ định 1 kho nhận, chỉ hoàn tiền sau khi kiểm định đạt SELLABLE.
* **Audit & Reporting (`BR-AR01` đến `BR-AR20`):** Ánh xạ vào `AuditLog`, `AuditLogDetail`, `ReportDefinition`, `ReportExecution`, `ReportSnapshot`, `ReportPermission`. Nhật ký Append-Only, phân quyền ma trận báo cáo.

---

## 5. TRUY VẾT 63 QUYẾT ĐỊNH THIẾT KẾ CHI TIẾT M2

Toàn bộ 63 quyết định kỹ thuật từ tài liệu [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md) đã được hiện thực hóa đầy đủ:

1. **`PC-Q01` đến `PC-Q06` (Catalog):** Bắt buộc danh mục chính qua Filtered Index; cây đệ quy Category; sản phẩm đơn sinh SKU mặc định; phân định `AttributeRole` loại trừ lẫn nhau; từ điển thuộc tính dùng chung; SKU bất biến.
2. **`WI-Q01` đến `WI-Q06` (Warehouse):** Số dư tồn kho tức thời độc lập; công thức AvailableQty; phân bổ kho trung gian; chuyển kho 2 bước In-Transit; kiểm kê không tự cập nhật tồn; phân tách trách nhiệm duyệt kiểm kê.
3. **`PR-Q01` đến `PR-Q06` (Procurement):** Một PO một kho nhận cố định; cho phép nhận nhiều đợt; kiểm định phân loại chất lượng; hoàn trả NCC theo chứng từ nhận gốc; phân tách trách nhiệm duyệt PO.
4. **`CID-Q01` đến `CID-Q06` (Identity):** Ràng buộc XOR Account; RBAC động; định danh linh hoạt Email/Phone qua Filtered Index; bảo mật băm mật khẩu; lịch sử OTP; tối đa 1 địa chỉ mặc định.
5. **`SHOP-Q01` đến `SHOP-Q06` (Shopping):** Tối đa 1 giỏ Active; tự cộng dồn SKU; cảnh báo SKU không khả dụng; không khóa giá/không giữ tồn trong giỏ; đồng bộ giỏ hàng đa thiết bị; đóng băng giỏ khi chuyển đơn.
6. **`SALES-Q01` đến `SALES-Q07` (Sales):** Tạo đơn & giữ hàng nguyên tử; máy trạng thái đơn độc lập; bảo toàn bất biến dòng hàng; snapshot địa chỉ đơn hàng; cấm xóa vật lý chứng từ; nhật ký trạng thái đầy đủ.
7. **`SHIP-Q01` đến `SHIP-Q07` (Shipping):** Một shipment xuất từ 1 kho; chỉ tạo khi đơn confirmed; đề xuất carrier tối ưu; tối đa 3 lần giao; bằng chứng giao hàng; cập nhật trạng thái đa kênh; hạch toán cước phí.
8. **`PROMO-Q01` đến `PROMO-Q08` (Promotion):** Khuyến mãi sản phẩm theo SKU; điều kiện cấu hình chuẩn; phân loại voucher chung/riêng; kiểm soát hạn mức 2 cấp; vòng đời voucher; quản lý phiên bản chương trình.
9. **`PAY-Q01` đến `PAY-Q09` (Payment):** Một phương thức thanh toán cho mỗi đơn; hỗ trợ song song COD và Online; thanh toán lại không làm vỡ đơn; hoàn tiền nhiều lần; kiểm soát duyệt hoàn tiền; lưu vết đối soát.
10. **`RET-Q01` đến `RET-Q08` (Return):** Khách tự tạo hoặc CSKH tạo hộ; giới hạn thời gian đổi trả; cho phép trả nhiều lần; phê duyệt 2 bước; chỉ định 1 kho nhận; phân loại disposition; kích hoạt hoàn tiền an toàn.
11. **`AR-Q01` đến `AR-Q07` (Audit):** Ghi log chọn lọc nghiệp vụ trọng yếu; lưu old/new value; nhật ký chỉ ghi cấm sửa xóa; chính sách archive; báo cáo vận hành song song snapshot kế toán; phân quyền ma trận.

---

## 6. KIỂM TRA TIÊU CHÍ NGHIỆM THU MILESTONE 2 (EXIT CRITERIA AUDIT)

| Mã Tiêu chí | Nội dung Tiêu chí Nghiệm thu M2 | Minh chứng & Tài liệu bàn giao | Kết quả Thẩm định |
| :---: | :--- | :--- | :---: |
| **EC-M2-01** | Chuẩn hóa danh mục 70 Thực thể qua 11 phân hệ | [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md): Đầy đủ 70 thực thể E01..E70 kèm phân loại, ý nghĩa, thuộc tính và khóa PK/UK. | **ĐẠT (100%)** |
| **EC-M2-02** | Phân tích quan hệ và bản số Crow's Foot chi tiết | [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md): Toàn bộ các cặp quan hệ đều có bản số và tính tham gia xác thực. | **ĐẠT (100%)** |
| **EC-M2-03** | Xác lập ma trận bản số và các ràng buộc nghiệp vụ bậc cao | [03-Cardinality-and-Constraints.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/03-Cardinality-and-Constraints.md): Ma trận bản số toàn diện, 5 bất biến kiến trúc, ràng buộc tài chính và Filtered Indexes. | **ĐẠT (100%)** |
| **EC-M2-04** | Master Conceptual ERD Diagram hợp nhất 70 thực thể | [04-Conceptual-ERD.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/04-Conceptual-ERD.md): Sơ đồ Mermaid Master Backbone và 11 sơ đồ phân hệ chi tiết không có thực thể mồ côi. | **ĐẠT (100%)** |
| **EC-M2-05** | Hệ thống hóa các quyết định thiết kế kiến trúc M2 | [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md): 3 nguyên tắc bất biến (A, B, C) và 63 quyết định kỹ thuật được khóa chính thức. | **ĐẠT (100%)** |
| **EC-M2-06** | Ma trận truy vết hai chiều M1 $\leftrightarrow$ M2 đầy đủ 100% | [06-M1-M2-Traceability.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/06-M1-M2-Traceability.md): Truy vết 100% BD, FR, BR và Quyết định kỹ thuật vào thực thể và quan hệ. | **ĐẠT (100%)** |
| **EC-M2-07** | Bảo đảm 5 Bất biến Kiến trúc sống còn | Cả 5 bất biến (Phân tách tồn kho, Độc lập máy trạng thái, Giữ hàng nguyên tử, Bất biến lịch sử/Cấm xóa cứng, Khuyến mãi 2 tầng) được cài đặt triệt để. | **ĐẠT (100%)** |
| **EC-M2-08** | Sẵn sàng chuyển giao sang Thiết kế Luận lý (Milestone 3) | Toàn bộ định nghĩa thực thể, quan hệ, khóa ứng viên và ràng buộc đã sẵn sàng cho bước chuẩn hóa 3NF/BCNF và định kiểu T-SQL. | **ĐẠT (100%)** |

---

## 7. BIÊN BẢN NGHIỆM THU CHÍNH THỨC & CHUYỂN GIAO MILESTONE 3

### 7.1. Kết luận Thẩm định của Hội đồng Kỹ thuật
Căn cứ trên kết quả thẩm tra chéo và đối soát ma trận truy vết:
* **Tỷ lệ hoàn thành Milestone 2:** **100.0%**.
* **Độ chính xác kỹ thuật:** Hoàn toàn tuân thủ các chuẩn mực quốc tế về Database Engineering, mô hình hóa Crow's Foot và triết lý thiết kế Microsoft SQL Server Enterprise.
* **Tính toàn vẹn nghiệp vụ:** Không phát sinh mâu thuẫn giữa các phân hệ; 100% yêu cầu chức năng và quy tắc nghiệp vụ M1 đều có thực thể chịu trách nhiệm rõ ràng.

### 7.2. Tuyên bố Chính thức Khép lại Milestone 2
> **QUYẾT ĐỊNH:** 
> **CHÍNH THỨC NGHIỆM THU VÀ ĐÓNG MILESTONE 2 (CONCEPTUAL DATABASE DESIGN).**
> 
> Toàn bộ bộ tài liệu M2 từ `01-Entity-Catalog.md` đến `06-M1-M2-Traceability.md` được phê duyệt làm tài liệu nền tảng bất biến (Baseline Architecture) để chuyển giao sang **Milestone 3 – Logical Database Design**.

### 7.3. Kế hoạch Hành động Ngay cho Milestone 3 (Logical Database Design)
Ngay sau khi đóng M2, các hạng mục kỹ thuật của Milestone 3 sẽ được khởi động theo lộ trình:
1. **M3.01 – Relational Schema Mapping:** Ánh xạ 70 thực thể thành 70 quan hệ (Tables), xác định chính xác Primary Key (Surrogate Identity vs Composite Natural Key) và Foreign Keys.
2. **M3.02 – Chuẩn hóa Dữ liệu (3NF / BCNF Normalization):** Chứng minh toán học mọi bảng đều đạt chuẩn 3NF và BCNF, phân tích phụ thuộc hàm (Functional Dependencies) loại trừ hoàn toàn dư thừa dữ liệu bất hợp lý.
3. **M3.03 – SQL Server Data Types & Storage Estimation:** Định nghĩa kiểu dữ liệu tối ưu (`BIGINT`, `NVARCHAR`, `DECIMAL(18,2)`, `DATETIMEOFFSET`, `ROWVERSION`), ước tính dung lượng bảng và Transaction Log cho quy mô lớn.
4. **M3.04 – Check Constraints & Filtered Unique Indexes Specification:** Đặc tả chi tiết mã T-SQL của toàn bộ các ràng buộc nghiệp vụ và chỉ mục có lọc trên SQL Server.

