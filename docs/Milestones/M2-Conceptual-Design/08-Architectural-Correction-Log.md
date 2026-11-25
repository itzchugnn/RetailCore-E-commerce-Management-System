# M2.08 - ARCHITECTURAL CORRECTION LOG (BẢN GHI HIỆU CHỈNH KIẾN TRÚC CHUẨN HÓA PRE-M3)

* **Dự án:** RetailCore – E-commerce Management System
* **Cột mốc:** Milestone 2 (M2 – Conceptual Design) $\to$ Milestone 3 (M3 – Logical Design)
* **Phiên bản:** 2.1 – Canonical SSOT Baseline (**79 Thực thể `E01` – `E79`**, đồng bộ 100% với [GEMINI.md](../../../GEMINI.md))
* **Trạng thái:** **APPROVED & FULLY SYNCHRONIZED**

---

## 1. MỤC ĐÍCH TÀI LIỆU

Văn bản này ghi nhận chính thức **10 Hiệu chỉnh Kiến trúc Bắt buộc (`CORR-01` đến `CORR-10`)** sau các đợt rà soát phản biện độc lập trước cổng Milestone 3, chính thức hóa danh mục chuẩn toàn hệ thống gồm **79 Thực thể (`E01` – `E79`)** để ánh xạ `1:1` sang 79 Bảng Quan hệ ở M3.

---

## 2. MA TRẬN TỔNG HỢP 10 HIỆU CHỈNH KIẾN TRÚC CHUẨN HÓA (`CORR-01` – `CORR-10`)

| Mã | Hạng mục Hiệu chỉnh | Mức độ | Phân hệ ảnh hưởng | Quyết định Kiến trúc Đã Khóa (Canonical SSOT) |
| :---: | :--- | :---: | :--- | :--- |
| **CORR-01** | Tách quyền hoàn tiền khỏi định đoạt kho, Phiên bản hóa KCS & Đổi hàng `EXCHANGE` | **P0** | SA10, SA09, SA06 | Tách `RefundEligibilityStatus` khỏi `InventoryDisposition`; bổ sung `ReturnLineAssessment` (E72) tách lô (`AssessmentGroupNo`) & phiên bản hóa (`IsCurrent`); `EXCHANGE` sinh đơn đổi bù `EXCHANGE_REPLACEMENT` ($0\text{đ}$). |
| **CORR-02** | Tồn kho cách ly 3 phẩm chất, Nhập bến RMA vào `QUARANTINED` & 5 Check Constraints | **P0** | SA02, SA10 | Chỉ có 3 phẩm chất vật lý (`SELLABLE`, `QUARANTINED`, `DAMAGED`); bến nhập `ReturnReceipt` ghi `RETURN_RECEIPT_IN` vào `QUARANTINED` trước khi KCS chuyển `SELLABLE` hoặc xuất hủy `SCRAP_OUT`; 5 CHECK constraints trên `InventoryBalance` (E11). |
| **CORR-03** | Định giá Đã Gồm GTGT, LRM, Hoàn tiền lũy tiến $\Delta F$ & Clawback Voucher | **P0** | SA06, SA08, SA09, SA10 | Giá bán lẻ đã gồm GTGT (`TaxAmount` bóc tách nội hàm); phân bổ voucher `'ORDER'` bằng LRM phá hòa `OrderLineID ASC`; tính tiền hoàn/hủy thiếu hàng bằng $\Delta\text{Refund} = F(a+b) - F(a)$ kèm Clawback Voucher khi lỗi khách vi phạm `MinOrderAmount`. |
| **CORR-04** | Ràng buộc Mandatory 1:1, Revoke DML & TPT Account | **P1** | SA04, SA06, SA09 | Thu hồi quyền DML trực tiếp của `retailcore_app`, thực thi qua Boundary SP nguyên tử; chuẩn hóa TPT `Account` (E29) XOR `Customer` (E30) / `Employee` (E31) dùng `AccountID PK, FK`. |
| **CORR-05** | Idempotency Record & Payment Webhook Log | **P1** | SA06, SA09 | Phân biệt `IdempotencyKey` (bảng `IdempotencyRecord` E74), `TransactionReference`, `ProviderEventID` (bảng `PaymentWebhookLog` E71 lưu bền vững trước HTTP 200). |
| **CORR-06** | Độc lập 4 Máy trạng thái Giao dịch | **P1** | SA06, SA07, SA09, SA10 | $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$; tách `Payment.PaymentStatus` khỏi từng lần thử `PaymentTransaction.TransactionStatus`. |
| **CORR-07** | Đối soát COD 2 đầu & Mô hình COD 3 lớp | **P1** | SA07, SA09 | Tách nghĩa vụ khách hàng (`PARTIALLY_PAID` / `PAID`) với công nợ hãng vận chuyển qua `CarrierSettlementStatus` và cụm 3 bảng `CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79); bổ sung `DeliveryReturnReceipt` (E75). |
| **CORR-08** | Nguồn tiền hoàn & Định giá tồn kho Moving WAC | **P2** | SA02, SA03, SA07, SA09 | Đặt `OriginalPaymentTransactionID` tại `RefundTransaction` (E59); tính WAC trên `AcceptedQty` của bể `SELLABLE`, snapshot `OutboundUnitCost` trên `ShipmentLine` & `UnitCost` trên `WarehouseTransferLine`, hạch toán chênh lệch trả NCC vào `CostVarianceAccountCode`. |
| **CORR-09** | Chuẩn hóa JSON Array Snapshot biến thể & Danh mục Brand | **P2** | SA01, SA06 | Chuẩn hóa `OrderLine.VariantAttributesSnapshot` dạng JSON Array có `ISJSON(...) = 1`; tách bảng danh mục `Brand` (E73) và khóa phức hợp thuộc tính động. |
| **CORR-10** | Lịch sử giữ hàng $1:0..N$ & Hạn đổi trả theo từng kiện | **P2** | SA02, SA07, SA10 | `StockReservationLine` (E14) liên kết $1:0..N$ với `AllocationID` kèm Filtered Unique Index `ACTIVE`; bổ sung `ReturnShipmentAllocation` (E76) kiểm tra hạn RMA theo `Shipment.DeliveredAt`. |

---

## 3. ĐẶC TẢ CHI TIẾT CÁC HIỆU CHỈNH TRỌNG TÂM

### 3.1. `CORR-01` (P0): Tách Quyền Hoàn Tiền Khỏi Định Đoạt Tồn Kho & Phiên Bản Hóa `ReturnLineAssessment` (E72)
* **Vấn đề gốc:** Quy định cũ chỉ tạo `Refund` khi hàng nhập lại kho đạt `SELLABLE`, làm mất quyền hoàn tiền của khách khi hàng bị lỗi từ NSX (`VENDOR_FAULT`) hoặc vỡ hỏng do vận chuyển (`CARRIER_FAULT`) buộc phải tiêu hủy. Ngoài ra, gắn `Disposition` đơn trị trên `ReturnReceiptLine` không biểu diễn được trường hợp nhận 3 sản phẩm trong đó 2 sản phẩm duyệt ngay và 1 sản phẩm chờ giám định.
* **Quyết định chuẩn hóa:**
  1. Bỏ cột `Disposition` khỏi `ReturnReceiptLine` (E63) và chuyển toàn bộ thông tin kết luận sang thực thể **`ReturnLineAssessment` (E72)** ($1:N$ với `ReturnReceiptLine`).
  2. Phân tách 4 chiều nghiệp vụ độc lập trên `ReturnLineAssessment`:
     * `FaultParty`: `'VENDOR_FAULT'`, `'CARRIER_FAULT'`, `'CUSTOMER_FAULT'`, `'NO_FAULT'`.
     * `RefundEligibilityStatus`: `'PENDING_INSPECTION'`, `'APPROVED'`, `'PARTIALLY_APPROVED'`, `'REJECTED'`.
     * `ResolutionType`: `'REFUND'`, `'EXCHANGE'`, `'REPAIR'`, `'NO_ACTION'` (với `'EXCHANGE'`, hệ thống sinh đơn hàng đổi bù `SalesOrder` mang `OrderType = 'EXCHANGE_REPLACEMENT'`, `ParentOrderID` trỏ đơn gốc và `TotalAmount = 0` để đi qua đúng luồng phân bổ – giữ kho – xuất kiện mà không vi phạm trần `ShipmentQty <= OrderedQty`).
     * `InventoryDisposition`: `'SELLABLE'`, `'QUARANTINED'`, `'DISPOSED'`.
  3. **Tách lô giám định kết hợp Audit Trail phiên bản hóa:**
     * Mỗi lô số lượng trên cùng `ReturnReceiptLineID` được đánh số `AssessmentGroupNo` ($1, 2, \dots$), phiên bản hóa qua `AssessmentVersion` và cờ `IsCurrent BIT`.
     * Cài đặt Filtered Unique Index bảo toàn đồng thời tách lô $1:N$ và duy nhất 1 bản hiện hành cho mỗi lô:
       ```sql
       CREATE UNIQUE NONCLUSTERED INDEX UX_ReturnLineAssessment_Current
       ON dbo.ReturnLineAssessment(ReturnReceiptLineID, AssessmentGroupNo)
       WHERE IsCurrent = 1;
       ```
     * Bất biến số lượng: Tổng `AssessedQty` của các bản ghi `IsCurrent = 1` không vượt quá `ReturnReceiptLine.ReceivedQty`.

---

### 3.2. `CORR-02` (P0): Tồn Kho Cách Ly 3 Phẩm Chất & 5 Declarative CHECK Constraints
* **Ba trạng thái phẩm chất tồn kho vật lý:** `SELLABLE`, `QUARANTINED`, `DAMAGED`. (`SCRAP_OUT` là loại giao dịch xuất tiêu hủy khỏi kho với `ToStockStatus = NULL`, không phải trạng thái tồn kho).
* **Công thức & 5 Ràng buộc trên `InventoryBalance` (E11):**
  $$\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$$
  $$\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty}$$
  1. `CK_InventoryBalance_OnHand_NonNegative`: `OnHandQty >= 0`
  2. `CK_InventoryBalance_Reserved_NonNegative`: `ReservedQty >= 0`
  3. `CK_InventoryBalance_Quarantined_NonNegative`: `QuarantinedQty >= 0`
  4. `CK_InventoryBalance_Damaged_NonNegative`: `DamagedQty >= 0`
  5. `CK_InventoryBalance_Capacity`: `ReservedQty + QuarantinedQty + DamagedQty <= OnHandQty`

---

### 3.3. `CORR-03` (P0): Định Giá Đã Gồm GTGT, Phân Bổ LRM, Hoàn Tiền Lũy Tiến $\Delta F$ & Thu Hồi Voucher (Clawback)
1. **Chuẩn hóa công thức giá bán lẻ B2C đã bao gồm thuế GTGT (VAT-Inclusive) & Hóa đơn B2B:**
   * Đơn giá bán lẻ niêm yết `ProductVariant.SellingPrice` đã bao gồm thuế GTGT: `LineGrossAmount = OrderedQty * UnitPrice`, `LineNetAmount = LineGrossAmount - LineProductDiscount - AllocatedOrderDiscount`.
   * Thuế GTGT bóc tách nội hàm phục vụ kê khai/hóa đơn điện tử: $\text{OrderLine.TaxAmount} = \text{LineNetAmount} - \text{ROUND}(\text{LineNetAmount} / (1 + \text{VatRate}/100.0), 0)$.
   * Tổng thanh toán đơn hàng: $\text{SalesOrder.TotalAmount} = \text{SubtotalAmount} - \text{DiscountAmount} + (\text{ShippingFee} - \text{ShippingDiscountAmount})$. Thông tin pháp nhân nhận hóa đơn B2B tùy chọn (`BuyerCompanyName`, `BuyerTaxCode`, `BuyerCompanyAddress`, `BuyerInvoiceEmail`) được snapshot độc lập trên `OrderAddress` (E42).
2. **Largest Remainder Method (LRM):**
   * Tính trên $\text{EligibleBase}_i = \text{LineGrossAmount}_i - \text{LineProductDiscount}_i$.
   * $\text{BaseAllocation}_i = \lfloor \text{ActualVoucherDiscount} \times \text{EligibleBase}_i / \sum \text{EligibleBase} \rfloor$.
   * Phần dư $D = \text{ActualVoucherDiscount} - \sum \text{BaseAllocation}_i$ đồng được cộng thêm $1\text{đ}$ cho $D$ dòng có phần thập phân dư lớn nhất, phá hòa cố định theo `OrderLineID ASC`.
3. **Công thức hoàn tiền lũy tiến từng đơn vị $\Delta F$ (dùng chung cho Hủy thiếu hàng `CancelledQty` & Đổi trả RMA) và Quy tắc Clawback Voucher:**
   * Với dòng hàng mua số lượng $Q = \text{OrderedQty}$ có giá trị ròng `LineNetAmount`:
     $$B = \left\lfloor \frac{\text{LineNetAmount}}{Q} \right\rfloor, \qquad R = \text{LineNetAmount} - (B \times Q) = \text{LineNetAmount} \pmod{Q}$$
   * Hàm tích lũy cho $q$ đơn vị đầu tiên ($0 \le q \le Q$):
     $$F(q) = B \cdot q + \min(q, R)$$
   * Khi hủy thiếu hàng trước xuất kho (`CancelledQty`) hoặc duyệt hoàn thêm $b$ đơn vị RMA sau khi dòng đơn hàng đã có $a = \text{OrderLine.RefundedQty}$ đơn vị được hoàn trước đó (khóa dòng `OrderLine` bằng `WITH (UPDLOCK, HOLDLOCK)`):
     $$\Delta\text{Refund}(a \to a+b) = F(a + b) - F(a)$$
   * **Quy tắc Clawback Voucher khi lỗi/đổi ý từ phía khách:** Nếu `FaultParty IN ('VENDOR_FAULT', 'CARRIER_FAULT')` (hoặc kho hủy thiếu hàng `CancelledQty`), khách hàng giữ nguyên quyền lợi voucher trên phần hàng còn lại. Ngược lại, nếu `FaultParty IN ('CUSTOMER_FAULT', 'NO_FAULT')` làm tổng `EligibleBase` của phần hàng khách giữ lại giảm xuống dưới ngưỡng `Promotion.MinOrderAmount` của voucher cấp `'ORDER'`, hệ thống thu hồi phần chiết khấu voucher không còn đủ điều kiện: $\text{EffectiveRefund} = \max(0, \Delta\text{Refund}(a \to a+b) - \text{UnrecoveredOrderVoucherBenefit})$.
   * **Ví dụ kiểm chứng toán học chính xác:** $\text{LineNetAmount} = 100.001\text{đ}$, $Q = 3 \implies B = 33.333\text{đ}$ và $R = 100.001 - 99.999 = \mathbf{2}\text{đ}$:
     * Đợt 1 trả 1 sp ($a = 0 \to 1$): $F(1) - F(0) = (33.333 \times 1 + \min(1, 2)) - 0 = \mathbf{33.334\text{đ}}$.
     * Đợt 2 trả 1 sp ($a = 1 \to 2$): $F(2) - F(1) = (33.333 \times 2 + \min(2, 2)) - 33.334 = 66.668 - 33.334 = \mathbf{33.334\text{đ}}$.
     * Đợt 3 trả 1 sp ($a = 2 \to 3$): $F(3) - F(2) = (33.333 \times 3 + \min(3, 2)) - 66.668 = 100.001 - 66.668 = \mathbf{33.333\text{đ}}$.
     * Tổng cộng 3 đợt: $33.334 + 33.334 + 33.333 = \mathbf{100.001\text{đ}}$ (khớp tuyệt đối từng đồng VNĐ).

---

### 3.4. `CORR-04` (P1): Ràng Buộc Mandatory $1:1$, Thu Hồi Quyền DML & Mô Hình TPT `Account`
* Thu hồi quyền `INSERT, UPDATE, DELETE` trực tiếp của tài khoản ứng dụng `retailcore_app` trên `SalesOrder`, `OrderAddress`, `Payment`, `Account`, `Customer`, `Employee` (và thu hồi `UPDATE, DELETE` trên `AuditLog`). Mọi thao tác khởi tạo đi qua Boundary Stored Procedure (`usp_Checkout_CreateOrder`, `usp_Identity_RegisterAccount`).
* Chuẩn hóa TPT thuần khiết: `Account(AccountID PK, AccountType)` XOR giữa `Customer(AccountID PK, FK)` và `Employee(AccountID PK, FK, EmployeeCode UK)`. Các bảng nghiệp vụ dùng cột khóa ngoại vai trò `CustomerID` tham chiếu `Customer(AccountID)`.

---

### 3.5. `CORR-05` (P1): Bảng `IdempotencyRecord` (E74) & `PaymentWebhookLog` (E71)
* **Bổ sung thực thể `IdempotencyRecord` (E74):** Lưu `IdempotencyRecordID (PK)`, `AccountID (FK)`, `OperationType`, `IdempotencyKey`, `RequestHash` (SHA-256), `ProcessingStatus` (`IN_PROGRESS`, `SUCCEEDED`, `FAILED`), `HttpStatusCode`, `ResponseSnapshot`, `ResourceType`, `ResourceID`, `CreatedAt`, `ExpiresAt` với `UNIQUE (AccountID, OperationType, IdempotencyKey)`.
* **Thực thể `PaymentWebhookLog` (E71):** Lưu bền vững mọi callback từ cổng thanh toán trước khi trả HTTP 200 với chỉ mục duy nhất `UX_PaymentWebhookLog_ProviderEvent ON (PaymentMethodID, ProviderEventID, Environment) WHERE ProviderEventID IS NOT NULL`.

---

### 3.6. `CORR-06` (P1): Độc Lập 4 Máy Trạng Thái Giao Dịch
* `SalesOrder.OrderStatus`: `PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `PARTIALLY_DELIVERED` $\to$ `DELIVERED` (hủy: `PARTIALLY_CANCELLED`, `CANCELLED`; đóng hành chính qua `ClosedAt`).
* `Shipment.ShipmentStatus`: `PREPARING` $\to$ `PACKED` $\to$ `DISPATCHED` $\to$ `IN_TRANSIT` $\to$ `OUT_FOR_DELIVERY` $\to$ `DELIVERED` (hoàn: `RETURNED_TO_SENDER` $\to$ `RETURN_RECEIVED_AT_WAREHOUSE`).
* `Payment.PaymentStatus`: `PENDING` $\to$ `AUTHORIZED` $\to$ `PARTIALLY_PAID` $\to$ `PAID` $\to$ `PARTIALLY_REFUNDED` $\to$ `REFUNDED` $\to$ `EXPIRED` / `CANCELLED` (tách rời khỏi từng lần thử `PaymentTransaction.TransactionStatus`).
* `ReturnRequest.ReturnStatus`: `PENDING_APPROVAL` $\to$ `APPROVED` $\to$ `AWAITING_RETURN` $\to$ `RECEIVED` $\to$ `INSPECTING` $\to$ `COMPLETED` / `REJECTED` / `CANCELLED`.

---

### 3.7. `CORR-07` (P1): Đối Soát COD 2 Đầu, Cụm COD 3 Lớp (`E77`–`E79`) & `DeliveryReturnReceipt` (`E75`)
* Nghĩa vụ khách hàng tách biệt với khoản phải thu từ Carrier (`Payment.CarrierSettlementStatus`).
* Bổ sung 3 thực thể đối soát COD chi tiết cho đơn hàng đa kiện Split Shipment:
  * **`CODCollection` (E77):** Ghi nhận tiền mặt bưu tá thu trên từng kiện `Shipment`.
  * **`CODRemittance` (E78):** Bảng kê đối soát và khoản tiền hãng vận chuyển chuyển trả về ngân hàng doanh nghiệp.
  * **`CODReconciliation` (E79):** Khớp đối soát từng `CODCollection` vào `CODRemittance`, theo dõi `VarianceAmount` và liên kết `PaymentTransaction` loại `COD_SETTLEMENT`.
* Bổ sung thực thể **`DeliveryReturnReceipt` (E75)** tại SA07 ghi nhận biên bản thủ kho thực nhận kiện hàng giao thất bại trước khi sinh giao dịch `DELIVERY_RETURN_IN` (`BD-07`).

---

### 3.8. `CORR-08` (P2): Nguồn Tiền Hoàn & Chính Sách Giá Vốn Bình Quân Liên Tục (Moving WAC)
* Đặt `OriginalPaymentTransactionID` tại `RefundTransaction` (E59) tham chiếu `PaymentTransaction` (E57) gốc; đồng thời trên `Refund` (E58) cho phép `ReturnLineAssessmentID` là tùy chọn (`NULL` khi hoàn tiền hủy đơn trả trước trước giao hàng, `NOT NULL` khi hoàn từ RMA).
* **Quy tắc tính Moving WAC chính xác:**
  1. *Nhập kho PO (`GoodsReceiptLine`):* Tính trên lượng hàng đạt chuẩn **`AcceptedQty`** (không tính `RejectedQty`) và chỉ tính trên bể tồn kho **`SELLABLE`** (`SellableOnHandQty = OnHandQty - QuarantinedQty - DamagedQty`) để hàng `DAMAGED`/`QUARANTINED` không làm sai lệch giá vốn lô mới:
     $$\text{NewAvgCost} = \frac{(\text{CurrentSellableQty} \times \text{CurrentAvgCost}) + (\text{AcceptedQty} \times \text{ReceiptUnitCost})}{\text{CurrentSellableQty} + \text{AcceptedQty}}$$
  2. *Xuất bán & Điều chuyển kho:* Snapshot giá vốn xuất kho hiện hành vào `ShipmentLine.OutboundUnitCost` (E46) và `WarehouseTransferLine.UnitCost` (E20).
  3. *Hoàn hàng bán / Giao thất bại (`CUSTOMER_RETURN_IN`, `DELIVERY_RETURN_IN`) & Nhận chuyển kho (`TRANSFER_IN`):* Nhập lại bể `SELLABLE` theo đúng `OutboundUnitCost` / `UnitCost` đã snapshot và tính lại `CurrentAverageCost` tại kho nhận.
  4. *Trả hàng NCC (`SupplierReturn` E27):* Giảm giá trị kho theo `CurrentAverageCost` hiện tại, phần chênh lệch với giá mua PO gốc hạch toán vào `CostVarianceAccountCode`.

---

### 3.9. `CORR-09` (P2): Chuẩn Hóa JSON Array Snapshot Biến Thể & Danh Mục `Brand` (`E73`)
* Bổ sung thực thể danh mục **`Brand` (E73)** tại SA01 (`Product.BrandID FK`) và ràng buộc khóa phức hợp `(AttributeID, AttributeValueID)` ngăn nhiễm chéo giá trị thuộc tính.
* Cột `OrderLine.VariantAttributesSnapshot NVARCHAR(MAX)` có `CHECK (ISJSON(VariantAttributesSnapshot) = 1)` và thống nhất **1 cấu trúc JSON Array duy nhất** trên toàn hệ thống:
  ```json
  [
    {
      "attributeId": 1,
      "attributeCode": "SIZE",
      "attributeName": "Kích cỡ",
      "valueId": 105,
      "valueCode": "XL",
      "displayValue": "Size XL"
    },
    {
      "attributeId": 2,
      "attributeCode": "COLOR",
      "attributeName": "Màu sắc",
      "valueId": 204,
      "valueCode": "NAVY",
      "displayValue": "Xanh Navy"
    }
  ]
  ```

---

### 3.10. `CORR-10` (P2): Lịch Sử Giữ Hàng $1:0..N$ & Truy Vết Hạn Đổi Trả Qua `ReturnShipmentAllocation` (`E76`)
* `StockReservationLine` (E14) liên kết $1:0..N$ với `OrderWarehouseAllocation` (E12) qua cột `AllocationID`, cài đặt chỉ mục lọc:
  `CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAlloc ON dbo.StockReservationLine(AllocationID) WHERE ReservationLineStatus = 'ACTIVE';`
* Bổ sung thực thể cầu nối **`ReturnShipmentAllocation` (E76)** tại SA10 nối `ReturnRequestLine` (E61) với `ShipmentLine` (E46), cưỡng chế kiểm tra thời hạn đổi trả theo từng kiện: $\text{RequestedAt} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$ và bảo đảm $\sum \text{AllocatedReturnQty} \le \text{ShipmentQty} - \text{PreviouslyReturnedQty}$.
