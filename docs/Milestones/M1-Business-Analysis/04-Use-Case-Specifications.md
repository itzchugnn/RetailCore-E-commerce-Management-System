# 04. USE CASE SPECIFICATIONS – ĐẶC TẢ CHI TIẾT CA SỬ DỤNG CỐT LÕI (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT (Đồng bộ 100% với 79 Thực thể `E01`–`E79`, `BD-01..07` và `CORR-01..10`)

Tài liệu đặc tả chi tiết **9 Use Case trọng yếu** quyết định trực tiếp kiến trúc giao dịch nguyên tử, kiểm soát đồng thời và đối soát tài chính – kho vận của RetailCore.

---

## UC-01: Place Order – Đặt hàng Nguyên tử & Chống Gửi Lặp (`usp_Checkout_CreateOrder`)

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-01` |
| **Primary Actor** | Customer |
| **Supporting Actors** | Payment Provider (nếu chọn thanh toán trả trước) |
| **Priority** | Critical (P0) |
| **Preconditions** | - Khách hàng đã đăng nhập tài khoản hợp lệ (`Customer.AccountID`, `BD-02`).<br>- Có giỏ hàng `ShoppingCart` trạng thái `'ACTIVE'` chứa $\ge 1$ `CartItem`.<br>- Client gửi kèm header `Idempotency-Key` duy nhất cho phiên đặt hàng. |
| **Main Flow** | 1. Khách hàng chọn địa chỉ nhận hàng (kèm thông tin xuất hóa đơn GTGT công ty tùy chọn), phương thức thanh toán và mã `Voucher` (`ORDER` / `SHIPPING`) nếu có.<br>2. Hệ thống kiểm tra/khóa nguyên tử `IdempotencyRecord(AccountID, 'CREATE_ORDER', IdempotencyKey, RequestHash)` (`CORR-05`).<br>3. Hệ thống truy xuất giá niêm yết `SellingPrice` (đã gồm GTGT), `VatRate` và khối lượng/kích thước từ CSDL (Zero-Trust Pricing & Shipping).<br>4. Kiểm tra và tính khuyến mãi 3 tầng (`PRODUCT` / Flash Sale cap, `ORDER` Voucher phân bổ xuống từng dòng bằng thuật toán **LRM**, `SHIPPING` Freeship Voucher).<br>5. Định tuyến phân bổ kho `OrderWarehouseAllocation` (`E12`) và khóa dòng `InventoryBalance` theo thứ tự cố định `VariantID ASC, WarehouseID ASC` (`BR-IN11`).<br>6. Kiểm tra `AvailableQty >= AllocatedQty` cho 100% SKU (All-or-Nothing); tăng `ReservedQty` và tạo `StockReservation` (`E13`) / `StockReservationLine` (`E14`) với `ExpiresAt` (`15 phút` chuẩn hoặc `5 phút` cho đơn Flash Sale).<br>7. Tạo nguyên tử `SalesOrder` (`E40`), bản chụp `OrderAddress` (`E42`), các dòng `OrderLine` (`E41` kèm bóc tách `TaxAmount` nội bộ và JSON Array `VariantAttributesSnapshot`), `OrderStatusHistory` (`E43`), `PromotionUsage` (`E53`) và hồ sơ `Payment` (`E56`).<br>8. Chuyển `ShoppingCart.CartStatus = 'CONVERTED'`, lưu `ResponseSnapshot` vào `IdempotencyRecord` và Commit Transaction.<br>9. Nếu đơn COD hợp lệ $\to$ `OrderStatus = 'CONFIRMED'`; nếu đơn Online $\to$ `OrderStatus = 'PENDING_PAYMENT'` và trả URL cổng thanh toán ngoài phạm vi DB Transaction. |
| **Alternative Flows** | - **A1 (Retry cùng `Idempotency-Key` & khớp `RequestHash`):** Hệ thống trả lại ngay `ResponseSnapshot` mã đơn hàng đã tạo, tuyệt đối không trừ tồn kho lần hai.<br>- **A2 (Đơn đổi bù `EXCHANGE_REPLACEMENT`):** Hệ thống nội bộ tạo đơn liên kết `ParentOrderID` với `TotalAmount = 0` từ kết luận KCS `E72`. |
| **Exception Flows** | - **E1 (Thiếu tồn kho bất kỳ SKU nào / Hết suất Flash Sale):** Rollback 100% giao dịch; giỏ hàng giữ nguyên `'ACTIVE'`.<br>- **E2 (Trùng `Idempotency-Key` nhưng đang xử lý hoặc lệch `RequestHash`):** Trả `409 Conflict`.<br>- **E3 (Voucher hết hạn / hết lượt `PendingCount + UsedCount >= MaxUsage`):** Từ chối giao dịch và yêu cầu khách chọn lại mã. |
| **Postconditions** | - Khởi tạo đồng bộ `SalesOrder` $1:1$ `OrderAddress` $1:1$ `Payment`.<br>- `ReservedQty` tăng tương ứng, `OnHandQty` chưa giảm, **không** sinh `InventoryTransaction` vật lý ở bước giữ hàng. |

---

## UC-02: Receive Purchase Order – Nhập Kho Kiểm Nhận QC & Tính Giá Vốn Moving WAC

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-02` |
| **Primary Actor** | Warehouse Staff, QC / KCS Inspector |
| **Supporting Actor** | Supplier |
| **Priority** | High |
| **Preconditions** | - `PurchaseOrder` (`E23`) đang ở trạng thái `'APPROVED'` hoặc `'SENT'` (`ApprovedBy <> CreatedBy`).<br>- Kho tiếp nhận trùng khớp `PurchaseOrder.DestinationWarehouseID` (`PR-Q01`). |
| **Main Flow** | 1. Thủ kho tạo phiếu `GoodsReceipt` (`E25`) gắn với `PurchaseOrderID` và `WarehouseID`.<br>2. Thủ kho và KCS kiểm đếm từng SKU trên `GoodsReceiptLine` (`E26`): ghi nhận `ReceivedQty = AcceptedQty + RejectedQty` và `UnitCost` theo PO.<br>3. Hệ thống kiểm tra lũy kế $\sum \text{ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$.<br>4. Khi chốt phiếu (`ReceiptStatus = 'POSTED'`), thủ tục `usp_PostInventoryTransaction` khóa `InventoryBalance` của kho nhận.<br>5. Chỉ phần `AcceptedQty > 0` mới sinh `InventoryTransaction` (`PURCHASE_RECEIPT`), cộng tăng `OnHandQty` (bể `SELLABLE`) và cập nhật lại giá vốn bình quân gia quyền `CurrentAverageCost` (`CORR-08`).<br>6. Cập nhật trạng thái PO sang `'PARTIALLY_RECEIVED'` hoặc `'FULLY_RECEIVED'`. |
| **Exception Flows** | - **E1 (`ReceivedQty` vượt số lượng còn lại trên PO):** Từ chối chốt phiếu nhập.<br>- **E2 (`RejectedQty > 0` không kèm `RejectionReason`):** Yêu cầu bổ sung lý do từ chối QC.<br>- **E3 (Phiếu đã `POSTED`):** Chặn xác nhận lặp (chứng từ kho bất biến). |
| **Postconditions** | - `InventoryBalance.OnHandQty` và `CurrentAverageCost` cập nhật chính xác trên lượng `AcceptedQty`; `RejectedQty` được lưu vết để đối soát NCC mà không làm bẩn tồn kho bán. |

---

## UC-03: Reserve & Release Inventory – Khóa Giữ & Giải Phóng Tồn Kho Logic

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-03` |
| **Primary Actor** | System (Kích hoạt từ `UC-01` hoặc Background Expiration Worker) |
| **Priority** | Critical (P0) |
| **Preconditions** | - Các dòng `OrderWarehouseAllocation` (`E12`) đã xác định kho xuất và số lượng cần giữ. |
| **Main Flow** | 1. Sắp xếp tập `(VariantID, WarehouseID)` tăng dần cố định để triệt tiêu Deadlock.<br>2. Khóa bản ghi `InventoryBalance` (`UPDLOCK, ROWLOCK`) và kiểm tra $\text{AvailableQty} = (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}) - \text{ReservedQty} \ge \text{AllocatedQty}$.<br>3. Cộng tăng `ReservedQty` và tạo `StockReservationLine` (`ReservationLineStatus = 'ACTIVE'`, cưỡng chế tối đa 1 dòng `ACTIVE` trên mỗi `AllocationID` qua `UX_StockReservationLine_ActiveAlloc`, `CORR-10`).<br>4. Khi quá hạn `ExpiresAt` mà đơn `PENDING_PAYMENT` chưa thanh toán (hoặc khách hủy đơn trước xuất bãi): Worker chuyển `ReservationLineStatus = 'EXPIRED'`/`'RELEASED'`, giảm `ReservedQty`, hoàn lại suất `PromotionUsage` (`'REVERSED'`) và hủy đơn. |
| **Exception Flows** | - **E1 (Webhook thanh toán đến trễ đúng lúc hết hạn):** Khóa đồng bộ trạng thái đơn; nếu đơn đã `EXPIRED`/`CANCELLED` nhưng cổng thu tiền thành công, tự động sinh yêu cầu `Refund` hoàn tiền 100% cho khách. |
| **Postconditions** | - Bảo đảm 5 CHECK Constraints trên `InventoryBalance` luôn thỏa mãn; `OnHandQty` không bao giờ bị thay đổi bởi thao tác giữ/nhả logic. |

---

## UC-04: Process Payment & Webhook – Xử Lý Thanh Toán Trực Tuyến Bất Đồng Bộ

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-04` |
| **Primary Actor** | Customer, Payment Provider |
| **Priority** | Critical (P0) |
| **Preconditions** | - `SalesOrder` đang ở `'PENDING_PAYMENT'` và `StockReservation` còn hạn `ExpiresAt`. |
| **Main Flow** | 1. Khách hàng gửi yêu cầu thanh toán kèm `Idempotency-Key` (`IdempotencyRecord` loại `'PROCESS_PAYMENT'`).<br>2. Hệ thống tạo bản ghi `PaymentTransaction` (`E57`, trạng thái `'INITIATED'`) và đóng kết nối DB trước khi gọi HTTP sang cổng thanh toán (Connection Pool Safety).<br>3. Khi Payment Provider gửi Webhook callback: hệ thống ghi bền vững ngay vào `PaymentWebhookLog` (`E71`) trước khi trả HTTP 200 (`CORR-05`).<br>4. Thủ tục `usp_ReceivePaymentWebhook` kiểm tra chống replay qua `(PaymentMethodID, ProviderEventID, Environment)` và xác thực chữ ký `SignatureVerified = 1`.<br>5. Cập nhật `PaymentTransaction.TransactionStatus = 'SUCCEEDED'`, `Payment.PaymentStatus = 'PAID'`, `PromotionUsage.UsageStatus = 'CONSUMED'` và chuyển `SalesOrder.OrderStatus = 'CONFIRMED'`. |
| **Exception Flows** | - **E1 (Giao dịch thất bại tại cổng):** Ghi nhận `PaymentTransaction` = `'FAILED'`; `Payment` vẫn giữ `'PENDING'` để khách thử lại tạo `PaymentTransaction` mới trong thời gian còn giữ hàng.<br>- **E2 (Webhook gửi trùng `ProviderEventID`):** Bỏ qua an toàn (idempotent no-op), trả HTTP 200. |
| **Postconditions** | - Lịch sử từng lần thử thanh toán và payload gốc của cổng được bảo toàn bất biến. |

---

## UC-05: Fulfill Order, Split Shipment & Failed Delivery Return – Xuất Giao Đa Kho & Nhập Hoàn Giao Thất Bại

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-05` |
| **Primary Actor** | Warehouse Staff, Sales Staff |
| **Supporting Actor** | Shipping Provider (`Carrier`) |
| **Priority** | High |
| **Main Flow** | 1. Từ đơn hàng `'CONFIRMED'`, từng kho vật lý có phân bổ tạo vận đơn `Shipment` (`E45`, đúng 1 `WarehouseID`) và `ShipmentLine` (`E46` khớp `AllocationID`).<br>2. Kho đóng gói (`'PACKED'`), gán `CarrierID`, `TrackingNumber` và `CODAmountToCollect` (nếu đơn COD).<br>3. Khi bàn giao bưu tá (`ShipmentStatus = 'DISPATCHED'`): hệ thống snapshot giá vốn hiện hành vào `ShipmentLine.OutboundUnitCost`, ghi sổ `InventoryTransaction` loại `'SHIPMENT_OUT'` giảm đồng thời `OnHandQty` và `ReservedQty`, tiêu thụ `StockReservationLine` (`'CONSUMED'`).<br>4. Bưu tá giao thành công (`'DELIVERED'`): ghi nhận `DeliveryAttempt` (`E47`) và cập nhật đơn hàng (`'PARTIALLY_DELIVERED'` / `'DELIVERED'`). |
| **Alternative & Exception Flows** | - **A1 (Hủy thiếu hàng trước xuất bãi – Short-Pick):** Nếu khi nhặt hàng phát hiện SKU hỏng không đủ xuất, nhân viên vận hành ghi nhận `OrderLine.CancelledQty`, giải phóng phần giữ kho thiếu, chuyển đơn sang `'PARTIALLY_CANCELLED'` và hoàn tiền lũy tiến $\Delta F$ cho phần hủy (nếu trả trước).<br>- **E1 (Giao hàng thất bại quá 3 lần `DeliveryAttempt`):** Kiện hàng chuyển sang `'RETURNED_TO_SENDER'` (**tuyệt đối chưa cộng lại tồn kho**). Chỉ khi kiện hàng quay về bến nhập và thủ kho ký biên bản `DeliveryReturnReceipt` (`E75`, `'POSTED'`), hệ thống mới sinh `DELIVERY_RETURN_IN` cộng lại tồn kho theo `OutboundUnitCost` gốc (`BD-07`). |
| **Postconditions** | - Tồn kho vật lý, trạng thái từng kiện hàng và giá vốn xuất kho được đối soát chính xác tuyệt đối. |

---

## UC-06: Apply 3-Tier Promotion & Voucher – Áp Dụng Khuyến Mãi 3 Tầng & Phân Bổ LRM

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-06` |
| **Primary Actor** | Customer |
| **Supporting Actor** | Marketing Staff |
| **Priority** | High |
| **Main Flow** | 1. Hệ thống tự động kiểm tra ưu đãi Tầng 1 (`PromotionScope = 'PRODUCT'` trên `PromotionProduct` `E51`, đối chiếu trần `FlashSaleStockLimit` và `MaxQtyPerCustomer`).<br>2. Khách hàng chọn/nhập tối đa 1 mã `Voucher` Tầng 2 (`'ORDER'`) và tối đa 1 mã `Voucher` Tầng 3 (`'SHIPPING'` Freeship).<br>3. Hệ thống kiểm tra thời hạn, nhóm khách hàng `PromotionTargetCustomer` (`E54`), điều kiện `PromotionCondition` (`E50`), hạn mức `PendingCount + UsedCount < MaxUsage`, `MaxUsagePerCustomer` và quy tắc kết hợp `IsStackable = 1` (`BD-05`).<br>4. Phân bổ chiết khấu Tầng 2 xuống từng `OrderLine.AllocatedOrderDiscount` bằng thuật toán **Largest Remainder Method (LRM)** phá hòa theo `OrderLineID ASC` (`CORR-03`), và áp mức giảm phí vận chuyển vào `SalesOrder.ShippingDiscountAmount <= ShippingFee`. |
| **Exception Flows** | - **E1:** Nhập nhiều hơn 1 chương trình trong cùng tầng hoặc vi phạm `IsStackable = 0` $\to$ Từ chối kết hợp và hướng dẫn khách chọn phương án tối ưu. |
| **Postconditions** | - Tổng `AllocatedOrderDiscount` trên các dòng khớp chính xác 100% số nguyên VNĐ với mức giảm của Voucher cấp đơn. |

---

## UC-07: Request Return, KCS Assessment & Progressive Refund / Exchange – Đổi Trả, Giám Định KCS & Hoàn Tiền $\Delta F$

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-07` |
| **Primary Actor** | Customer |
| **Supporting Actors** | Customer Support (`CSKH`), Warehouse Staff, QC / KCS Inspector, Finance / Accountant |
| **Priority** | Critical (P0) |
| **Preconditions** | - Kiện hàng `Shipment` liên quan đã giao thành công (`'DELIVERED'`).<br>- Thời điểm gửi yêu cầu thỏa $\text{RequestedAt} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$ trên từng kiện qua `ReturnShipmentAllocation` (`E76`, `CORR-10`). |
| **Main Flow** | 1. Khách hàng (hoặc CSKH tạo hộ) gửi `ReturnRequest` (`E60`), `ReturnRequestLine` (`E61`) và phân bổ kiện giao `ReturnShipmentAllocation` (`E76`).<br>2. CSKH thẩm tra chính sách, duyệt `'APPROVED'` và chỉ định duy nhất 1 kho nhận (`WarehouseID`).<br>3. Khi bưu kiện hoàn về kho, thủ kho lập `ReturnReceipt` (`E62`) và `ReturnReceiptLine` (`E63`); khi chốt phiếu (`POSTED`), hệ thống ghi `RETURN_RECEIPT_IN` đưa hàng thực nhận vào bể cách ly (`+OnHandQty`, `+QuarantinedQty`, tồn `SELLABLE` chưa đổi).<br>4. **Nhân viên KCS giám định độc lập (`usp_AssessReturnReceiptLine`):** Lập biên bản phiên bản hóa `ReturnLineAssessment` (`E72`) theo từng lô `AssessmentGroupNo`, xác định độc lập:<br>   - Định đoạt kho `InventoryDisposition`: `'SELLABLE'` ($\to$ giải phóng khỏi cách ly về `SELLABLE`, cập nhật WAC theo `OutboundUnitCost`), `'QUARANTINED'` (giữ trong bể cách ly chờ hãng kiểm tra), hoặc `'DISPOSED'` ($\to$ xuất tiêu hủy `SCRAP_OUT` giảm đồng thời `OnHandQty` và `QuarantinedQty` mà không ảnh hưởng tồn tốt).<br>   - Lỗi & Quyền lợi: `FaultParty` (`'VENDOR_FAULT'`, `'CARRIER_FAULT'`, `'CUSTOMER_FAULT'`, `'NO_FAULT'`), `RefundEligibilityStatus` và `ResolutionType` (`'REFUND'`, `'EXCHANGE'`, `'REPAIR'`, `'NO_ACTION'`).<br>5. Nếu `RefundEligibilityStatus = 'APPROVED'` và `ResolutionType = 'REFUND'`: khóa `UPDLOCK` trên `OrderLine`, tính tiền hoàn lũy tiến $\Delta\text{Refund} = F(a+b) - F(a)$ (khấu trừ Clawback nếu lỗi khách/đổi ý làm giá trị đơn còn lại tụt dưới `MinOrderAmount` của voucher) và sinh yêu cầu `Refund` (`E58`) cho Kế toán phê duyệt giải ngân qua `RefundTransaction` (`E59`, trỏ `OriginalPaymentTransactionID`). |
| **Alternative & Exception Flows** | - **A1 (Hàng tiêu hủy `DISPOSED` do lỗi NSX hoặc vận chuyển — `CORR-01`):** Kể cả khi hàng hỏng phải xuất tiêu hủy `SCRAP_OUT` không nhập lại kho bán, khách hàng **vẫn được duyệt hoàn tiền 100%** (hoặc đổi mới).<br>- **A2 (Đổi sản phẩm mới – `ResolutionType = 'EXCHANGE'`):** Hệ thống khởi tạo đơn hàng đổi bù liên kết `SalesOrder(OrderType = 'EXCHANGE_REPLACEMENT', ParentOrderID, TotalAmount = 0)` để xuất giao kiện mới đúng quy trình kho.<br>- **E1 (Hàng đưa vào `'QUARANTINED'` chờ hãng giám định):** Giữ `RefundEligibilityStatus = 'PENDING_INSPECTION'`; khi có kết luận cuối cùng mới tạo phiên bản `AssessmentVersion + 1` (`IsCurrent = 1`) mà không ghi đè bản cũ. |
| **Postconditions** | - Tách bạch hoàn toàn định đoạt vật lý tồn kho với quyền lợi tài chính của khách hàng; tổng tiền hoàn nhiều đợt không bao giờ vượt hoặc lệch `LineNetAmount`. |

---

## UC-08: Transfer Inventory – Điều Chuyển Kho Nội Bộ Hai Bước (`In-Transit`)

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-08` |
| **Primary Actor** | Warehouse Staff, Warehouse Manager |
| **Priority** | High |
| **Main Flow** | 1. Tạo phiếu `WarehouseTransfer` (`E19`, `SourceWarehouseID <> DestinationWarehouseID`) và `WarehouseTransferLine` (`E20`), trình quản lý phê duyệt.<br>2. **Bước 1 – Xuất kho nguồn (`DISPATCHED`):** Kiểm tra `AvailableQty >= TransferQty`, snapshot giá vốn WAC kho nguồn vào `WarehouseTransferLine.UnitCost`, ghi `InventoryTransaction` loại `'TRANSFER_OUT'` giảm `OnHandQty` kho nguồn; hàng chuyển sang trạng thái đang đi đường (`'IN_TRANSIT'`).<br>3. **Bước 2 – Nhập kho đích (`RECEIVED`):** Kho đích kiểm đếm `ReceivedQty`, ghi nhận `VarianceQty = TransferQty - ReceivedQty`, sinh `'TRANSFER_IN'` tăng `OnHandQty` và tính lại WAC tại kho đích theo `UnitCost` điều chuyển. |
| **Exception Flows** | - **E1 (Hư hỏng/thiếu hụt trên đường vận chuyển nội bộ `VarianceQty > 0`):** Phần thiếu hụt không cộng vào `SELLABLE` kho đích mà lập biên bản kiểm tra trách nhiệm vận chuyển nội bộ. |
| **Postconditions** | - Không xảy ra tình trạng nhân đôi tồn kho hoặc mất dấu hàng đang trung chuyển giữa hai kho. |

---

## UC-09: Reconcile COD Settlement – Đối Soát Công Nợ Thu Hộ COD Ba Lớp

| Thuộc tính | Đặc tả Chi tiết |
| :--- | :--- |
| **ID** | `UC-09` |
| **Primary Actor** | Finance / Accountant |
| **Supporting Actor** | Shipping Provider (`Carrier`) |
| **Priority** | High |
| **Main Flow** | 1. **Lớp 1 – Bưu tá thu tiền theo kiện (`CODCollection` E77):** Khi kiện COD giao thành công, ghi nhận số tiền thực thu từ khách, cập nhật nghĩa vụ khách hàng `Payment.PaymentStatus` sang `'PARTIALLY_PAID'` / `'PAID'` và `CarrierSettlementStatus = 'COLLECTED_BY_CARRIER'`.<br>2. **Lớp 2 – Bảng kê chuyển trả (`CODRemittance` E78):** Định kỳ hãng vận chuyển gửi bảng kê và chuyển khoản ngân hàng; Kế toán nhập `CODRemittance` (`TotalCollectedAmount`, `CarrierFeeDeducted`, `NetRemittedAmount`, `ReceivedBankTxRef`).<br>3. **Lớp 3 – Khớp đối soát từng kiện (`CODReconciliation` E79):** Hệ thống/Kế toán đối chiếu từng `CODCollection` vào `CODRemittance`, tính `VarianceAmount`; khi khớp hoàn tất (`'RECONCILED'`), sinh `PaymentTransaction` loại `'COD_SETTLEMENT'` và chuyển `Payment.CarrierSettlementStatus = 'RECONCILED'`. |
| **Exception Flows** | - **E1 (Lệch tiền thu hộ `VarianceAmount <> 0`):** Đánh dấu dòng `CODReconciliation` ở trạng thái `'DISCREPANCY'` để truy thu đối tác vận chuyển mà không làm ảnh hưởng trạng thái `'PAID'` hợp pháp của khách mua hàng. |
| **Postconditions** | - Tách bạch minh bạch 100% nghĩa vụ thanh toán của người mua và công nợ phải thu từ từng hãng vận chuyển trong đơn Split Shipment. |
