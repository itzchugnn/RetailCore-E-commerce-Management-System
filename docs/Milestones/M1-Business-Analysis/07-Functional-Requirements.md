# 07. FUNCTIONAL REQUIREMENTS – ĐẶC TẢ YÊU CẦU CHỨC NĂNG (BASELINE v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis (Đồng bộ 100% mã truy vết với M1.10 & M2.06)
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT
* **Tổng số yêu cầu chức năng:** **55 yêu cầu chuẩn hóa** trên 11 Phân hệ Nghiệp vụ

---

## Danh mục 55 Yêu cầu Chức năng Chuẩn hóa

| Mã Phân hệ | Mã FR | Tên Yêu cầu Chức năng | Tiêu chí Nghiệp vụ Cốt lõi |
| :--- | :--- | :--- | :--- |
| **01. Catalog** | **FR-CAT-01** | Quản lý cây danh mục đa cấp | Tạo/sửa/ẩn danh mục đệ quy `Category`, kiểm soát `Slug` duy nhất và chống vòng lặp cha-con. |
| | **FR-CAT-02** | Quản lý thương hiệu & sản phẩm | Quản lý danh mục `Brand`, hồ sơ `Product`, thuế suất `VatRate` và gán danh mục chính `IsPrimary` (`BD-01`). |
| | **FR-CAT-03** | Quản lý thuộc tính động & Whitelist | Cấu hình từ điển `Attribute`, `AttributeValue`, phân vai `VARIANT_DEFINING` / `DESCRIPTIVE` và Whitelist (`BD-06`). |
| | **FR-CAT-04** | Quản lý biến thể SKU, giá & quy cách vận chuyển | Quản lý `ProductVariant` (`SKU` duy nhất, `SellingPrice` nguyên đồng VNĐ, khối lượng/kích thước vật lý). |
| | **FR-CAT-05** | Tìm kiếm & Lọc đa chiều (Faceted Search) | Phân trang, sắp xếp và lọc phía Server theo `Category`, `Brand`, khoảng giá và `AttributeValue`. |
| **02. Warehouse** | **FR-INV-01** | Quản lý mạng lưới kho vật lý | Quản lý thông tin, địa chỉ và trạng thái hoạt động của các kho `Warehouse`. |
| | **FR-INV-02** | Quản lý số dư tồn kho theo phẩm chất & WAC | Tra cứu tức thời `OnHandQty`, `ReservedQty`, `QuarantinedQty`, `DamagedQty`, `AvailableQty` và giá vốn `CurrentAverageCost` (`CORR-02`, `CORR-08`). |
| | **FR-INV-03** | Khóa giữ tồn kho có thời hạn & Timeout | Khóa giữ All-or-Nothing qua `StockReservation` / `StockReservationLine` (15 phút chuẩn / 5 phút Flash Sale) và tự động nhả tồn khi hết hạn (`BD-04`, `CORR-10`). |
| | **FR-INV-04** | Điều chuyển kho 2 bước (In-Transit) | Xuất chuyển `TRANSFER_OUT` tại kho nguồn (kèm giá vốn WAC) và xác nhận thực nhận `TRANSFER_IN` tại kho đích. |
| | **FR-INV-05** | Kiểm kê kho & Phê duyệt điều chỉnh sổ cái | Lập phiếu `StockCount`, phê duyệt phân tách trách nhiệm (`ApprovedBy <> CreatedBy`) và ghi sổ cái `InventoryTransaction`. |
| **03. Procurement** | **FR-PO-01** | Quản lý nhà cung cấp & danh mục cung ứng | Quản lý hồ sơ `Supplier` và bảng giá/lead-time tham chiếu theo SKU `SupplierProduct`. |
| | **FR-PO-02** | Lập & Phê duyệt đơn mua hàng (PO) | Tạo `PurchaseOrder` chỉ định 1 kho nhận cố định, phê duyệt nội bộ trước khi gửi NCC. |
| | **FR-PO-03** | Tiếp nhận hàng mua nhiều đợt | Lập phiếu nhập `GoodsReceipt` đối soát số lượng lũy kế không vượt số lượng đặt trên `PurchaseOrderLine`. |
| | **FR-PO-04** | Kiểm định chất lượng hàng nhập (QC) | Tách bạch `AcceptedQty` (nhập kho tính WAC) và `RejectedQty` (từ chối nhận tại bến) trên `GoodsReceiptLine`. |
| | **FR-PO-05** | Xuất trả hàng nhà cung cấp & Hạch toán chênh lệch | Lập `SupplierReturn` xuất kho theo WAC hiện hành và hạch toán chênh lệch giá PO vào `CostVarianceAccountCode` (`CORR-08`). |
| **04. Identity** | **FR-AUTH-01** | Đăng ký, Đăng nhập & Xác minh OTP | Xác thực qua Email/Phone, băm mật khẩu an toàn và quản lý lịch sử mã xác thực `AccountVerification`. |
| | **FR-AUTH-02** | Quản lý hồ sơ khách hàng & Sổ địa chỉ | Khởi tạo TPT `Customer`, quản lý nhiều địa chỉ nhận hàng `CustomerAddress` với tối đa 1 địa chỉ mặc định. |
| | **FR-AUTH-03** | Quản lý hồ sơ nhân sự nội bộ | Khởi tạo TPT `Employee` loại trừ XOR hoàn toàn với `Customer` (`CORR-04`). |
| | **FR-AUTH-04** | Phân quyền vai trò động (RBAC) | Quản lý `Role`, `Permission`, gán vai trò `AccountRole` và ma trận `RolePermission`. |
| **05. Shopping** | **FR-CRT-01** | Khởi tạo & Đồng bộ giỏ hàng tập trung | Duy trì tối đa 1 `ShoppingCart` trạng thái `ACTIVE` cho mỗi khách hàng trên mọi thiết bị. |
| | **FR-CRT-02** | Quản lý mặt hàng trong giỏ | Thêm/cập nhật/xóa `CartItem`, tự động cộng dồn số lượng khi chọn trùng SKU. |
| | **FR-CRT-03** | Kiểm tra tính khả dụng thời gian thực | Hiển thị giá hiện hành và cảnh báo/chặn đặt hàng nếu SKU trong giỏ ngừng bán hoặc không đủ tồn khả dụng. |
| | **FR-CRT-04** | Đóng băng chuyển đổi giỏ hàng khi đặt đơn | Chuyển trạng thái giỏ sang `CONVERTED` nguyên tử ngay khi đơn hàng và giữ kho thành công. |
| **06. Sales** | **FR-ORD-01** | Đặt hàng nguyên tử & Chống gửi lặp | Thực thi Checkout All-or-Nothing có bảo vệ `IdempotencyRecord` (E74) (`BD-02`, `BD-04`, `CORR-05`). |
| | **FR-ORD-02** | Đóng băng Snapshot hợp đồng bán hàng | Lưu bất biến địa chỉ & thông tin hóa đơn B2B `OrderAddress`, đơn giá đã gồm GTGT, `TaxAmount` bóc tách và mảng JSON `VariantAttributesSnapshot` trên `OrderLine` (`CORR-09`). |
| | **FR-ORD-03** | Quản lý máy trạng thái đơn hàng độc lập | Vận hành `OrderStatus` độc lập với thanh toán/giao vận và ghi vết bất biến `OrderStatusHistory` (`CORR-06`). |
| | **FR-ORD-04** | Định tuyến & Phân bổ đơn hàng đa kho | Phân tách dòng đơn hàng về các kho xuất phù hợp qua `OrderWarehouseAllocation` (`BD-03`). |
| | **FR-ORD-05** | Hủy đơn, Hủy thiếu hàng & Đơn đổi bù | Hỗ trợ hủy toàn bộ hoặc hủy thiếu hàng (`CancelledQty`), hoàn trả `ReservedQty`/voucher và sinh đơn đổi bù `EXCHANGE_REPLACEMENT`. |
| | **FR-ORD-06** | Bảo vệ bất biến chứng từ bán hàng | Cấm xóa vật lý và thu hồi quyền DML trực tiếp trên bảng đơn hàng (`CORR-04`). |
| **07. Shipping** | **FR-SHP-01** | Đóng gói kiện hàng theo kho xuất | Tạo `Shipment` và `ShipmentLine` xuất phát từ đúng 1 kho vật lý khớp với phân bổ kho. |
| | **FR-SHP-02** | Điều phối hãng vận chuyển & Xuất bãi | Gán `Carrier`, mã vận đơn `TrackingNumber`, snapshot `OutboundUnitCost` và ghi sổ `SHIPMENT_OUT` khi bàn giao. |
| | **FR-SHP-03** | Theo dõi giao nhận & Giới hạn lần giao | Ghi nhận tối đa 3 lần thử `DeliveryAttempt`, tự động chuyển hoàn `RETURNED_TO_SENDER` khi vượt ngưỡng. |
| | **FR-SHP-04** | Tiếp nhận kiện hàng giao thất bại hoàn kho | Thủ kho ký biên bản `DeliveryReturnReceipt` (E75) thực nhận hàng hoàn trước khi cộng lại `OnHandQty` (`BD-07`). |
| | **FR-SHP-05** | Nhật ký hành trình vận đơn | Cập nhật và tra cứu lịch sử trạng thái kiện hàng `ShipmentStatusHistory` từ Webhook và nhân viên. |
| **08. Promotion** | **FR-PRM-01** | Quản lý chương trình khuyến mãi & Phiên bản | Cấu hình `Promotion`, điều kiện `PromotionCondition` và quản lý phiên bản bất biến khi đổi chính sách. |
| | **FR-PRM-02** | Khuyến mãi giảm giá theo SKU & Flash Sale | Thiết lập biến thể hưởng ưu đãi Tầng 1 kèm trần `FlashSaleStockLimit` và `MaxQtyPerCustomer` qua `PromotionProduct`. |
| | **FR-PRM-03** | Phát hành Voucher đơn hàng & Freeship | Quản lý mã `Voucher` (`ORDER` / `SHIPPING`) công khai hoặc cá nhân hóa và nhóm khách hàng `PromotionTargetCustomer`. |
| | **FR-PRM-04** | Tính toán Stacking 3 tầng & Phân bổ LRM | Kiểm soát cờ `IsStackable` giữa 3 tầng (`PRODUCT`, `ORDER`, `SHIPPING` theo `BD-05`) và phân bổ giảm giá cấp đơn bằng thuật toán LRM (`CORR-03`). |
| | **FR-PRM-05** | Kiểm soát hạn mức & Sổ cái tiêu thụ ưu đãi | Chống vượt trần `MaxUsage` / `MaxUsagePerCustomer` đồng thời và lưu vết `PromotionUsage`. |
| **09. Payment** | **FR-PAY-01** | Quản lý danh mục phương thức thanh toán | Cấu hình `PaymentMethod` (COD, VNPAY, MoMo, Thẻ, Chuyển khoản). |
| | **FR-PAY-02** | Quản lý hồ sơ nghĩa vụ thanh toán đơn hàng | Khởi tạo hồ sơ `Payment` ($1:1$ với `SalesOrder`) và theo dõi tiến độ thu/hoàn tiền. |
| | **FR-PAY-03** | Xử lý giao dịch cổng & Lưu vết Webhook bền vững | Ghi nhận từng lần thử `PaymentTransaction` và chống replay callback qua `PaymentWebhookLog` (E71) (`CORR-05`). |
| | **FR-PAY-04** | Phê duyệt & Thực thi hoàn tiền truy vết gốc | Quản lý `Refund` và `RefundTransaction` liên kết `OriginalPaymentTransactionID` (`CORR-08`). |
| | **FR-PAY-05** | Đối soát công nợ thu hộ COD 3 lớp | Quản lý thu tiền bưu tá, bảng kê chuyển trả và khớp chênh lệch qua `CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79) (`CORR-07`). |
| **10. Return** | **FR-RET-01** | Tạo yêu cầu đổi trả truy vết theo kiện giao | Tiếp nhận `ReturnRequest` / `ReturnRequestLine` gắn với kiện hàng qua `ReturnShipmentAllocation` (E76), kiểm tra hạn đổi trả theo `Shipment.DeliveredAt` (`CORR-10`). |
| | **FR-RET-02** | Xét duyệt RMA & Chỉ định kho tiếp nhận | CSKH duyệt yêu cầu trước khi gửi hàng và ấn định 1 kho nhận duy nhất. |
| | **FR-RET-03** | Tiếp nhận bưu kiện hoàn trả vào cách ly | Lập phiếu thực nhận `ReturnReceipt` / `ReturnReceiptLine` tại bến nhập, ghi `RETURN_RECEIPT_IN` đưa hàng vào bể `QUARANTINED`. |
| | **FR-RET-04** | Giám định KCS, Hoàn tiền $\Delta F$ & Đổi bù | Phiên bản hóa `ReturnLineAssessment` (E72), tách quyền hoàn tiền khỏi kho (`CORR-01`), tính tiền hoàn $\Delta F = F(a+b) - F(a)$ (kèm Clawback Voucher) hoặc sinh đơn đổi bù `EXCHANGE_REPLACEMENT`. |
| | **FR-RET-05** | Nhật ký vòng đời xử lý khiếu nại RMA | Lưu vết toàn bộ lịch sử chuyển đổi trạng thái trong `ReturnStatusHistory`. |
| **11. Audit & BI** | **FR-AUD-01** | Nhật ký kiểm toán an ninh bất biến (WORM) | Ghi nhận chọn lọc thao tác nhạy cảm vào `AuditLog` và `AuditLogDetail` (cấm sửa/xóa). |
| | **FR-AUD-02** | Quản lý hạn lưu trữ & Phân vùng Archive Log | Quản lý `RetentionUntil` và chuyển phân vùng nhật ký cũ sang lưu trữ lạnh. |
| | **FR-REP-01** | Quản lý danh mục mẫu báo cáo | Định nghĩa cấu hình các mẫu báo cáo vận hành và tài chính trong `ReportDefinition`. |
| | **FR-REP-02** | Thực thi truy vấn & Kiểm toán xuất dữ liệu | Chạy báo cáo bất đồng bộ/trực tiếp và lưu nhật ký truy xuất `ReportExecution`. |
| | **FR-REP-03** | Chốt số liệu báo cáo kỳ kế toán cố định | Đóng băng số liệu cuối kỳ bất biến qua `ReportSnapshot`. |
| | **FR-REP-04** | Phân quyền ma trận truy cập báo cáo | Kiểm soát chi tiết quyền Xem / Xuất file / Chốt kỳ theo vai trò qua `ReportPermission`. |
