# M2.08 - ARCHITECTURAL CORRECTION LOG & BASELINE AMENDMENT
## BẢN GHI HIỆU CHỈNH KIẾN TRÚC & ĐIỀU CHỈNH BASELINE MÔ HÌNH DỮ LIỆU KHÁI NIỆM

* **Dự án:** RetailCore – E-commerce Management System
* **Cột mốc:** Milestone 2 (M2 – Conceptual Design) $\to$ Milestone 3 (M3 – Logical Design)
* **Phiên bản:** 1.0 – Official Architecture Correction Log
* **Trạng thái:** Active / Mandatory Baseline for Milestone 3
* **Căn cứ hiệu chỉnh:** Thẩm định phản biện kiến trúc độc lập sau Báo cáo M2.07
* **Tài liệu tham chiếu:**
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md)
  * [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md)
  * [04-Conceptual-ERD.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/04-Conceptual-ERD.md)
  * [07-Pre-M3-Architectural-Review.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/07-Pre-M3-Architectural-Review.md)

---

## 1. NGUYÊN TẮC KIẾN TRÚC CỐT LÕI (CORE ARCHITECTURAL PRINCIPLE)

> [!IMPORTANT]
> **Nguyên tắc Vàng Xuyên suốt:**  
> *"Không chỉ thiết kế để lưu được trạng thái cuối cùng; hệ thống CSDL RetailCore phải được thiết kế để truy vết và tái hiện được toàn bộ quá trình chuyển từ trạng thái này sang trạng thái khác (State Transitions, Financial Auditability & Quality Movement)."*

Mục đích của tài liệu `M2.08` là ghi nhận có hệ thống toàn bộ các hiệu chỉnh kiến trúc cần thiết, ngăn chặn triệt để nguy cơ lan truyền lỗi nghiệp vụ và sai lệch toàn vẹn dữ liệu từ tầng Conceptual (M2) xuống tầng Logical/Physical (M3/M4).

---

## 2. MA TRẬN TỔNG HỢP CÁC KHOẢN MỤC HIỆU CHỈNH (CORRECTION MATRIX)

| Mã | Hạng mục | Mức độ | Phân hệ ảnh hưởng | Tóm tắt Quyết định Hiệu chỉnh |
| :---: | :--- | :---: | :--- | :--- |
| **CORR-01** | Refund Eligibility vs Inventory Disposition | **P0** | 10 (Return), 09 (Payment) | Tách biệt hoàn toàn quyền hoàn tiền của khách khỏi phân loại vật lý kho; bổ sung thực thể `ReturnLineAssessment` theo số lượng thực tế; tách riêng `ResolutionType`. |
| **CORR-02** | Quản lý Tồn kho Cách ly & Biến động Chất lượng | **P0** | 02 (Inventory) | Phân biệt rõ Stock Quality Status (`SELLABLE`, `QUARANTINED`, `DAMAGED`) với Movement Type (`SCRAP_OUT`); bổ sung 5 ràng buộc số dư toàn vẹn. |
| **CORR-03** | Phân bổ Chiết khấu & Hoàn tiền Lũy tiến | **P0** | 06 (Sales), 08 (Promo), 09 | Thuật toán LRM chuẩn mực có chặn trần; công thức hoàn tiền lũy tiến $F(q)$ bảo đảm chia lẻ tiền VND nguyên tuyệt đối không lệch đồng lẻ. |
| **CORR-04** | Thực thi Ràng buộc Mandatory 1:1 | **P1** | 06 (Sales), 09 (Payment) | Minh định SQL Server không hỗ trợ Deferrable FK; thu hồi toàn bộ quyền DML trực tiếp, bảo đảm bắt buộc bằng Boundary Transaction & Stored Procedure. |
| **CORR-05** | Account Subtype XOR & Total Participation | **P1** | 04 (Identity) | Chuẩn hóa mô hình TPT (`Account-Customer-Employee`); kiểm soát Total Participation và Disjoint bằng Transaction Stored Procedure kết hợp Trigger. |
| **CORR-06** | Định danh & Xử lý Webhook Thanh toán | **P1** | 09 (Payment) | Phân biệt `IdempotencyKey` (kèm `RequestHash`), `TransactionReference`, `ProviderEventID`; bổ sung thực thể `PaymentWebhookLog` có trạng thái tiếp nhận vs xử lý. |
| **CORR-07** | Chuẩn hóa 3 State Machine Độc lập | **P1** | 06, 07, 09 | Tách biệt hoàn toàn máy trạng thái; đơn hàng chuyển trạng thái tổng hợp dựa trên số lượng còn hiệu lực; tách trạng thái thử `PaymentTransaction` khỏi `Payment`. |
| **CORR-08** | Truy vết Nguồn tiền Hoàn & Đối soát COD | **P1** | 09 (Payment), 07 (Shipping) | Đặt `OriginalPaymentTransactionID` tại `RefundTransaction`; tách bạch nghĩa vụ thanh toán của khách (`CUSTOMER_PAID`) và công nợ phải thu từ hãng (`CarrierSettlementStatus`). |
| **CORR-09** | Chính sách Giá vốn Bình quân Di động | **P2** | 02 (Inventory), 03 (Procure) | Moving Weighted Average: Sửa quy tắc xuất trả NCC (`SupplierReturn`) giảm tồn kho theo WAC hiện tại, hạch toán chênh lệch vào tài khoản điều chỉnh giá vốn. |
| **CORR-10** | Lịch sử Giữ hàng & Snapshot Thuộc tính | **P2** | 02, 06, 10 | Mở rộng Allocation $\leftrightarrow$ Reservation thành $1:0..N$ có Filtered Unique Index; chuẩn hóa JSON snapshot có định danh thuộc tính; Return Window tính theo từng Shipment. |

---

## 3. ĐẶC TẢ CHI TIẾT CÁC KHOẢN MỤC HIỆU CHỈNH

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-01 (Mức độ: P0 - Khẩn cấp)
TÊN HIỆU CHỈNH: TÁCH BIỆT QUYỀN HOÀN TIỀN KHỎI ĐỊNH ĐOẠT TỒN KHO & BỔ SUNG RETURN_LINE_ASSESSMENT
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Quy định: *"Chỉ sau khi ReturnReceipt có dòng Disposition = SELLABLE được POSTED, hệ thống mới tự động tạo Refund"*. Điều này tước đoạt quyền hoàn tiền của khách khi hàng bị lỗi do nhà sản xuất (áo lỗi đường may) và kho buộc phải tiêu hủy (`DISPOSED`). Ngoài ra, nếu chỉ gắn trạng thái trên `ReturnReceiptLine`, không thể biểu diễn trường hợp một đợt nhận hàng có 2 cái duyệt hoàn tiền và 1 cái phải chờ giám định.
* **Quyết định Kiến trúc Sửa đổi:**
  1. Phân tách rạch ròi 3 khái niệm nghiệp vụ:
     * **Bên chịu trách nhiệm (`FaultParty`):** `VENDOR_FAULT`, `CARRIER_FAULT`, `CUSTOMER_FAULT`, `NO_FAULT`.
     * **Kết quả xét duyệt hoàn tiền (`RefundEligibilityStatus`):** `PENDING_INSPECTION`, `APPROVED`, `PARTIALLY_APPROVED`, `REJECTED`.
     * **Hình thức xử lý bồi hoàn (`ResolutionType`):** `REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION` (Tách rời khỏi trạng thái hoàn tiền, cho phép hỗ trợ đổi hàng/sửa chữa kể cả khi không hoàn tiền).
     * **Định đoạt vật lý tại kho (`InventoryDisposition`):** `SELLABLE`, `QUARANTINE`, `DISPOSED`.
  2. **Bổ sung Thực thể Đánh giá Riêng biệt `ReturnLineAssessment` (E72):**
     * Quan hệ: `ReturnReceiptLine (1) ───< (1..N) ReturnLineAssessment`.
     * Cấu trúc thực thể: `AssessmentID (PK, bigint)`, `ReturnReceiptLineID (FK)`, `AssessedQty (int)`, `FaultParty`, `RefundEligibilityStatus`, `ResolutionType`, `ApprovedRefundAmount (decimal)`, `ShippingRefundAmount (decimal)`, `DisputeReason`, `InspectionNotes`, `AssessedAt`, `AssessedBy`.
     * **Ràng buộc kiểm soát số lượng:** $\sum \text{ReturnLineAssessment.AssessedQty} \le \text{ReturnReceiptLine.ReceivedQty}$.
  3. **Quy tắc Nghiệp vụ Chuẩn hóa:**
     * `BR-RT-AMEND-01`: Hàng bị phân loại `DISPOSED` do lỗi của nhà sản xuất (`VENDOR_FAULT`) hoặc vỡ hỏng do vận chuyển (`CARRIER_FAULT`) vẫn được xét duyệt `RefundEligibilityStatus = 'APPROVED'` và được tạo chứng từ `Refund`.
     * `BR-RT-AMEND-02`: Hàng đưa vào `QUARANTINE` (chờ hãng giám định) bắt buộc giữ `RefundEligibilityStatus = 'PENDING_INSPECTION'`. Tuyệt đối không sinh chứng từ `Refund` khi chưa có kết luận biên bản giám định.
     * `BR-RT-AMEND-03 (Partial Milestone Refund)`: Hỗ trợ tạo `Refund` ngay khi từng dòng đánh giá `ReturnLineAssessment` đạt chuẩn `APPROVED` mà không cần chờ toàn bộ yêu cầu trả hàng (`ReturnRequest`) hoàn tất.
* **Thực thể & Thuộc tính Tác động:**
  * Bổ sung thực thể mới `ReturnLineAssessment` (E72) thuộc Subject Area 10 (Return & Exchange).
  * `ReturnReceiptLine` (E63): lưu `ReceivedQty`, `ConditionDescription`, và liên kết $1:N$ với `ReturnLineAssessment`.

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-02 (Mức độ: P0 - Khẩn cấp)
TÊN HIỆU CHỈNH: TỒN KHO CÁCH LY - PHÂN BIỆT STOCK QUALITY STATUS VỚI MOVEMENT TYPE
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Quản lý tồn kho ở cấp Warehouse nhưng chỉ có `OnHandQty` và `ReservedQty`. Sử dụng `DISPOSED` như một trạng thái tồn kho là sai vì hàng tiêu hủy đã rời kho, không còn tồn tại vật lý.
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Phân biệt hai loại thông tin độc lập:**
     * **A. Trạng thái Chất lượng Tồn kho Vật lý (Stock Quality Status):**  
       Chỉ gồm 3 trạng thái của hàng hóa thực tế còn nằm trong kho: `SELLABLE`, `QUARANTINED`, `DAMAGED`.
     * **B. Loại Nghiệp vụ Biến động Kho (Inventory Movement Type):**  
       `PURCHASE_RECEIPT`, `CUSTOMER_RETURN_IN`, `QUARANTINE_IN`, `QUARANTINE_RELEASE`, `MOVE_TO_QUARANTINE`, `MOVE_TO_DAMAGED`, `SCRAP_OUT` (Tiêu hủy: hàng rời kho $\to$ `ToStockStatus = NULL`), `SHIPMENT_OUT`, `SUPPLIER_RETURN_OUT`.
  2. **Bốn Cột Số dư trên `InventoryBalance` (E11) & 5 Ràng buộc Toàn vẹn (Integrity Constraints):**
     $$\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$$
     $$\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty}$$
     * `CK_InventoryBalance_OnHand_Positive`: $\text{OnHandQty} \ge 0$
     * `CK_InventoryBalance_Quarantined`: $0 \le \text{QuarantinedQty} \le \text{OnHandQty}$
     * `CK_InventoryBalance_Damaged`: $0 \le \text{DamagedQty} \le \text{OnHandQty}$
     * `CK_InventoryBalance_TotalDefect`: $\text{QuarantinedQty} + \text{DamagedQty} \le \text{OnHandQty}$
     * `CK_InventoryBalance_Reserved`: $0 \le \text{ReservedQty} \le (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty})$
  3. **Truy vết Biến động Chuyển trạng thái Chất lượng:**
     * Bổ sung trên `InventoryTransactionLine` (E16): `FromStockStatus` và `ToStockStatus` ($\in$ `{'SELLABLE', 'QUARANTINED', 'DAMAGED', NULL}`).
     * Ràng buộc: `ReservedQty` là trạng thái giữ hàng phục vụ đơn, không phải trạng thái chất lượng $\to$ **Cấm** đưa `RESERVED` vào `FromStockStatus`/`ToStockStatus`.
     * Nghiệp vụ tiêu hủy hàng hỏng: `FromStockStatus = 'DAMAGED'`, `ToStockStatus = NULL` (`OnHandDelta = -X, DamagedDelta = -X`).
     * Toàn bộ thao tác chuyển trạng thái phải được thực thi trong cùng một Transaction nguyên tử.
* **Thực thể & Thuộc tính Tác động:**
  * Sửa đổi `InventoryBalance` (E11): thêm `QuarantinedQty`, `DamagedQty`.
  * Sửa đổi `InventoryTransactionLine` (E16): thêm `FromStockStatus`, `ToStockStatus`, `QuarantinedDelta`, `DamagedDelta`.

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-03 (Mức độ: P0 - Khẩn cấp)
TÊN HIỆU CHỈNH: THUẬT TOÁN LRM CHUẨN XÁC & CÔNG THỨC HOÀN TIỀN LŨY TIẾN TỪNG ĐƠN VỊ
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Dùng hàm `Round()` đơn thuần không phải là Largest Remainder Method; chia số dư lẻ trực tiếp gây ra số thập phân lẻ xu/đồng (ví dụ: 100.001đ / 3 sản phẩm = 33.333,667đ, không thể hạch toán tiền VND nguyên).
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Thuật toán Largest Remainder Method Chuẩn mực (Voucher Cấp Đơn):**
     * Điều kiện tiên quyết: $0 \le \text{ActualVoucherDiscount} \le \text{TotalEligibleBase}$ và $\text{TotalEligibleBase} > 0$.
     * `ActualVoucherDiscount` là số tiền giảm cuối cùng sau khi áp dụng trần giảm tối đa (Discount Cap) và điều kiện tối thiểu của đơn hàng.
     * Cơ sở đủ điều kiện: $\text{EligibleBase}_i = \text{LineGrossAmount}_i - \text{LineProductDiscount}_i$.
     * Tính giá trị lý tưởng: $\text{IdealAllocation}_i = \text{ActualVoucherDiscount} \times \frac{\text{EligibleBase}_i}{\sum \text{EligibleBase}}$.
     * Lấy phần nguyên xuống: $\text{BaseAllocation}_i = \lfloor \text{IdealAllocation}_i \rfloor$.
     * Tính phần dư: $\text{Remainder}_i = \text{IdealAllocation}_i - \text{BaseAllocation}_i$.
     * Phân phối $D = \text{ActualVoucherDiscount} - \sum \text{BaseAllocation}_i$ đồng: Cộng $1$ đồng cho $D$ dòng có $\text{Remainder}_i$ lớn nhất. Phá hòa cố định bằng `OrderLineID ASC`.
     * Bảo đảm: $\sum \text{AllocatedOrderDiscount}_i \equiv \text{ActualVoucherDiscount}$.
  2. **Công thức Hoàn tiền Lũy tiến Từng Đơn vị Sản phẩm (Progressive Unit-Level Allocation):**
     * Đặt:
       $$B = \left\lfloor \frac{\text{LineNetAmount}}{\text{OrderedQty}} \right\rfloor, \quad R = \text{LineNetAmount} - (B \times \text{OrderedQty})$$
     * Tổng giá trị được phân bổ cho $q$ sản phẩm đầu tiên:
       $$F(q) = B \cdot q + \min(q, R)$$
     * Số tiền hoàn cho đợt trả từ sản phẩm thứ $(a+1)$ đến $(a+b)$:
       $$\text{RefundAmount} = F(a+b) - F(a)$$
     * *Ví dụ:* $100.001$đ chia cho 3 sản phẩm ($B = 33.333, R = 1$):
       * Hoàn 1 sản phẩm đầu ($a=0, b=1$): $F(1) - F(0) = 33.334 - 0 = \mathbf{33.334}$đ.
       * Hoàn sản phẩm thứ 2 ($a=1, b=1$): $F(2) - F(1) = 66.668 - 33.334 = \mathbf{33.334}$đ.
       * Hoàn sản phẩm thứ 3 ($a=2, b=1$): $F(3) - F(2) = 100.001 - 66.668 = \mathbf{33.333}$đ.
     * **Bảo toàn dữ liệu:** Số tiền hoàn thực tế được ghi vào `RefundTransaction`. Các cột `RefundedQty` và `AccumulatedRefundedAmount` trên `OrderLine` là dữ liệu tổng hợp (Derived/Cached) được cập nhật nguyên tử để tối ưu hóa truy vấn.
* **Thực thể & Thuộc tính Tác động:**
  * Sửa đổi `OrderLine` (E41): `LineGrossAmount`, `LineProductDiscount`, `AllocatedOrderDiscount`, `LineNetAmount`, `RefundedQty`, `AccumulatedRefundedAmount`.

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-04 (Mức độ: P1 - Nghiêm trọng)
TÊN HIỆU CHỈNH: BẢO ĐẢM TÍNH TOÀN VẸN RÀNG BUỘC MANDATORY 1:1 BẰNG TRANSACTION BOUNDARY
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Tuyên bố khóa ngoại `OrderID UNIQUE` ở bảng con tự bảo đảm tính bắt buộc ($1:1$ Mandatory). Đây là sai lầm lý thuyết vì UNIQUE chỉ bảo đảm $0..1:1$, không ngăn được việc tạo `SalesOrder` mồ côi con.
* **Quyết định Kiến trúc Sửa đổi:**
  1. Thừa nhận giới hạn: SQL Server không hỗ trợ Deferrable Constraints.
  2. **Bảo vệ bằng Transaction Boundary & Kiểm soát Quyền DML Chặt chẽ:**
     * Thu hồi toàn bộ quyền DML trực tiếp (`INSERT`, `UPDATE`, `DELETE`) trên các bảng nghiệp vụ nhạy cảm (`SalesOrder`, `OrderAddress`, `Payment`) đối với tài khoản ứng dụng (`REVOKE INSERT, UPDATE, DELETE ON SalesOrder TO ApplicationUser`).
     * Chỉ cấp quyền `EXECUTE` trên các Stored Procedure được kiểm soát chặt chẽ.
     * Quy trình Checkout bắt buộc phải được đóng gói trong Stored Procedure `usp_Checkout_CreateOrder`.
     * Thực thi `INSERT` đồng thời vào `SalesOrder`, `OrderAddress`, và `Payment` trong cùng một khối `BEGIN TRANSACTION ... COMMIT`. Rollback toàn bộ nếu bất kỳ thao tác nào thất bại.
  3. **Phân biệt Rạch ròi:** Thực thể `Payment` (hồ sơ nghĩa vụ tài chính) được tạo ngay khi checkout; điều này hoàn toàn tách biệt với việc có giao dịch thanh toán thành công (`PaymentTransaction`) hay chưa.
* **Thực thể Tác động:**
  * Quy chuẩn hóa giao diện lập trình của `SalesOrder` (E40), `OrderAddress` (E42), `Payment` (E56).

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-05 (Mức độ: P1 - Nghiêm trọng)
TÊN HIỆU CHỈNH: CHUẨN HÓA MÔ HÌNH ACCOUNT SUBTYPE XOR VÀ BẢO ĐẢM TOTAL PARTICIPATION
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Check constraint giả định bảng `Account` có 2 cột `CustomerID` và `EmployeeID` là sai lệch với thiết kế SA04 (nơi `Customer` và `Employee` là bảng subtype trỏ FK về `AccountID`). Ngoài ra, trigger trên bảng con chỉ ngăn sai loại chứ không bảo đảm Total Participation (vẫn có thể sinh ra Account mồ côi không có subtype).
* **Quyết định Kiến trúc Sửa đổi:**
  1. Thống nhất mô hình Table-Per-Type (TPT):
     * `Account`: `AccountID (PK)`, `AccountType` (`'CUSTOMER'` hoặc `'EMPLOYEE'`).
     * `Customer`: `AccountID (PK, FK trỏ Account.AccountID)`.
     * `Employee`: `AccountID (PK, FK trỏ Account.AccountID)`.
  2. **Giải pháp Đa tầng Bảo đảm Disjoint XOR & Total Participation:**
     * *Tầng 1 (Stored Procedure Boundary):* Khởi tạo tài khoản bắt buộc qua `usp_Identity_CreateCustomerAccount` hoặc `usp_Identity_CreateEmployeeAccount` (tạo `Account` và bản ghi subtype trong cùng một Transaction). Thu hồi quyền INSERT trực tiếp.
     * *Tầng 2 (Trigger Kiểm tra Tính Tương thích):* Trigger trên bảng con ngăn cản chèn `AccountID` có `AccountType` không tương thích, và từ chối chèn nếu `AccountID` đã tồn tại ở bảng subtype kia.
     * *Tầng 3 (Ràng buộc Bất biến):* Khóa không cho phép thay đổi `AccountType` tùy ý sau khi tạo, và ngăn cản thao tác xóa bản ghi subtype khi `Account` vẫn còn tồn tại.
* **Thực thể Tác động:**
  * Chuẩn hóa cấu trúc của `Account` (E29), `Customer` (E30), `Employee` (E31).

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-06 (Mức độ: P1 - Nghiêm trọng)
TÊN HIỆU CHỈNH: PHÂN BIỆT RÕ 3 KHÓA THANH TOÁN & BỔ SUNG THỰC THỂ PAYMENT WEBHOOK LOG
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Đánh đồng `TransactionReference` với `IdempotencyKey`. Chưa có cơ chế chống xử lý trùng Webhook lặp lại từ cổng thanh toán.
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Phân biệt rạch ròi 3 khái niệm định danh:**
     * `IdempotencyKey`: UUID do Client/Ứng dụng sinh ra, có phạm vi `UNIQUE (AccountID, OperationType, IdempotencyKey)`. Bổ sung trường `RequestHash NVARCHAR(64)` (băm SHA-256 payload request), `ProcessingStatus` (`IN_PROGRESS`, `SUCCEEDED`, `FAILED`), và `ResponseSnapshot NVARCHAR(MAX)` để trả lại kết quả tức thì khi client retry. Báo lỗi `409 Conflict` nếu trùng Key nhưng khác Hash.
     * `TransactionReference`: Mã tham chiếu giao dịch nội bộ của RetailCore phục vụ đối soát kế toán.
     * `ProviderEventID`: Mã sự kiện/ID giao dịch do Cổng thanh toán (VNPAY, MoMo, Stripe) gửi sang trong Webhook payload.
  2. **Bổ sung Thực thể Hỗ trợ `PaymentWebhookLog` (E71 - Associative/Support Entity):**
     * Thuộc tính: `WebhookLogID (PK)`, `PaymentMethodID (FK)`, `MerchantAccountID`, `ProviderEventID`, `Environment`, `PayloadJSON`, `SignatureVerified (BIT)`, `ProcessingStatus` (`RECEIVED`, `PROCESSED`, `FAILED`, `IGNORED`), `ReceivedAt`, `ProcessedAt`, `ErrorMessage`.
     * Chỉ mục duy nhất chống xử lý trùng: `CREATE UNIQUE INDEX UX_WebhookLog_Event ON PaymentWebhookLog(PaymentMethodID, ProviderEventID, Environment)`.
     * Tách bạch lỗi xác thực chữ ký (Signature Verification Failed) với lỗi xử lý nghiệp vụ. Chỉ cập nhật `Payment` sau khi chữ ký đã được xác thực hợp lệ.
* **Thực thể Tác động:**
  * Bổ sung thực thể mới `PaymentWebhookLog` vào Subject Area 09 (Payment & Refund).

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-07 (Mức độ: P1 - Nghiêm trọng)
TÊN HIỆU CHỈNH: CHUẨN HÓA 3 STATE MACHINE ĐỘC LẬP & TÁCH RỜI TRẠNG THÁI GIAO DỊCH
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Đưa `IN_TRANSIT` và `DELIVERED` vào `SalesOrder.OrderStatus`. Trộn lẫn trạng thái của một lần thử thanh toán thất bại vào trạng thái tổng thể của `Payment`. Đặt `FAILED_DELIVERY` làm trạng thái đóng của `Shipment`.
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Máy Trạng thái Đơn hàng (`SalesOrder.OrderStatus`):**
     * Trạng thái tổng hợp dựa trên số lượng còn hiệu lực:
       `PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `PARTIALLY_DELIVERED` $\to$ `DELIVERED`.
     * Trạng thái hủy: `CANCELLED` (hủy toàn bộ), `PARTIALLY_CANCELLED` (hủy phần số lượng chưa giao).
     * Định nghĩa chuẩn: `DELIVERED` được kích hoạt khi $100\%$ số lượng hàng còn hiệu lực phải giao đã được bưu tá giao thành công.
  2. **Máy Trạng thái Kiện hàng (`Shipment.ShipmentStatus`):**
     * Chuỗi hợp lệ: `PREPARING` $\to$ `PACKED` $\to$ `DISPATCHED` $\to$ `IN_TRANSIT` $\to$ `OUT_FOR_DELIVERY` $\to$ `DELIVERED`.
     * Phân tách: `DeliveryAttempt` ghi nhận kết quả từng lần giao (`FAILED_ATTEMPT`, `CUSTOMER_UNREACHABLE`). Chỉ khi hết số lần giao lại (tối đa 3 lần), `Shipment` mới chuyển `RETURNED_TO_SENDER` (đang chuyển trả).
     * Khi hàng vật lý về đến bến nhập của kho, ghi nhận sự kiện `RETURN_RECEIVED_AT_WAREHOUSE`.
  3. **Máy Trạng thái Thanh toán (`Payment.PaymentStatus`) vs Giao dịch Thử (`PaymentTransactionStatus`):**
     * **`Payment.PaymentStatus` (Hồ sơ tổng thể):** `PENDING` $\to$ `AUTHORIZED` (chỉ áp dụng khi cổng có tính năng 2 pha) $\to$ `PAID` $\to$ `PARTIALLY_REFUNDED` $\to$ `REFUNDED` $\to$ `EXPIRED` / `CANCELLED`.
     * **`PaymentTransaction.TransactionStatus` (Từng lần thử):** `INITIATED`, `PENDING`, `SUCCEEDED`, `FAILED`, `EXPIRED`, `CANCELLED`.
     * Khách thanh toán thất bại lần 1 thì `PaymentTransaction.TransactionStatus = 'FAILED'`, nhưng `Payment.PaymentStatus` vẫn giữ nguyên là `'PENDING'` cho phép khách thử thanh toán lại.
* **Thực thể Tác động:**
  * Cập nhật quy tắc chuyển trạng thái của `SalesOrder` (E40), `Shipment` (E45), `Payment` (E56), `PaymentTransaction` (E57).

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-08 (Mức độ: P1 - Nghiêm trọng)
TÊN HIỆU CHỈNH: TRUY VẾT NGUỒN TIỀN HOÀN TẠI REFUND_TRANSACTION & TÁCH BẠCH 2 ĐẦU ĐỐI SOÁT COD
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Đặt `OriginalPaymentTransactionID` ở `Refund` làm mất khả năng xử lý đơn hoàn tiền giải ngân từ nhiều giao dịch capture khác nhau. Không phân biệt nghĩa vụ thanh toán của khách với khoản phải thu đối soát từ hãng vận chuyển.
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Đặt Khóa Ngoại Truy Vết:**
     * Đặt `OriginalPaymentTransactionID` tại thực thể **`RefundTransaction`** (E59), trỏ về `PaymentTransaction` (E57).
     * Ràng buộc: `PaymentTransaction` được tham chiếu phải là giao dịch thu tiền thành công (`SUCCEEDED`). Tổng tiền hoàn từ một giao dịch gốc $\le$ số tiền thực thu còn lại có thể hoàn.
  2. **Tách bạch 2 Đầu Đối soát COD:**
     * **Góc nhìn Khách hàng (`Payment.PaymentStatus`):** Khi bưu tá quét mã xác nhận đã thu tiền mặt $\to$ Chuyển `PaymentStatus = 'CUSTOMER_PAID'` (hoặc `'PAID'`). Khách hàng đã hoàn thành $100\%$ nghĩa vụ, mở khóa xuất hóa đơn VAT điện tử và tích điểm.
     * **Góc nhìn Đối soát Công nợ Hãng (`Payment.CarrierSettlementStatus`):**
       * `NOT_COLLECTED`: Hàng đang giao, bưu tá chưa thu tiền.
       * `COLLECTED_BY_CARRIER`: Bưu tá đã thu tiền mặt từ khách.
       * `REMITTED`: Hãng vận chuyển chuyển khoản tiền COD về tài khoản công ty kèm bảng kê đối soát.
       * `RECONCILED`: Kế toán công ty khớp bảng kê đối soát thành công.
     * Chỉ khi đạt `CarrierSettlementStatus = 'RECONCILED'`, hệ thống mới sinh `PaymentTransaction` loại `COD_SETTLEMENT` ghi nhận tăng tiền gửi ngân hàng.
* **Thực thể Tác động:**
  * Sửa đổi `RefundTransaction` (E59): thêm `OriginalPaymentTransactionID (FK)`.
  * Sửa đổi `Payment` (E56): thêm thuộc tính `CarrierSettlementStatus`.

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-09 (Mức độ: P2 - Chi tiết Kỹ thuật)
TÊN HIỆU CHỈNH: CHÍNH SÁCH GIÁ VỐN WAC - HẠCH TOÁN CHUẨN XÁC XUẤT TRẢ NHÀ CUNG CẤP
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Chưa xác định phương pháp tính giá vốn. Quy định hàng trả nhà cung cấp giảm tồn kho theo giá mua PO là vi phạm nguyên tắc kế toán hàng tồn kho bình quân di động.
* **Quyết định Kiến trúc Sửa đổi:**
  1. Thống nhất áp dụng phương pháp **Bình quân Gia quyền Liên tục (Moving Weighted Average Costing)** cho RetailCore.
  2. Bổ sung trường `CurrentAverageCost` trên `InventoryBalance` (E11).
  3. **Bốn Nguyên tắc Hạch toán Giá vốn:**
     * *Nguyên tắc 1 (Nhập kho PO):* `GoodsReceiptLine.UnitCost` bao gồm giá mua thực tế cộng chi phí mua/vận chuyển phân bổ hợp lệ. Khi biên bản nhập kho được POST:
       $$\text{NewAvgCost} = \frac{(\text{CurrentOnHand} \times \text{CurrentAvgCost}) + (\text{ReceivedQty} \times \text{ReceiptUnitCost})}{\text{CurrentOnHand} + \text{ReceivedQty}}$$
     * *Nguyên tắc 2 (Trả hàng NCC - Supplier Return):* Giá trị ghi sổ hàng tồn kho **giảm theo giá vốn bình quân hiện tại (WAC at return time)**. Giá trị NCC hoàn lại tính theo giá mua PO gốc. Phần chênh lệch được hạch toán vào tài khoản **Chênh lệch/Điều chỉnh giá vốn hàng bán (Cost Variance / Inventory Adjustment)**.
       *(Ví dụ: Mua 100k, WAC hiện tại 120k. Xuất trả 1 cái $\to$ Tồn kho giảm 120k; Phải thu NCC là 100k; Chênh lệch 20k ghi nhận chi phí điều chỉnh giá vốn).*
     * *Nguyên tắc 3 (Khách trả lại hàng - Customer Return):* Hàng `SELLABLE` nhập lại kho được ghi nhận đúng theo giá vốn xuất kho ban đầu của đơn hàng đó, sau đó tính lại WAC cho kho nhận.
     * *Nguyên tắc 4 (Kiểm kê thừa/thiếu & Reset):* Ghi nhận theo giá vốn bình quân hiện hành tại thời điểm duyệt kiểm kê. Khi tồn kho $\text{OnHandQty} = 0$, giá trị tồn kho bằng $0$, giữ `CurrentAverageCost` làm mốc tham chiếu cho đến khi có lô hàng nhập kho PO mới.
* **Thực thể Tác động:**
  * Bổ sung thuộc tính `CurrentAverageCost` trên `InventoryBalance` (E11).
  * Quy chuẩn hóa cột `UnitCost` trên `InventoryTransactionLine` (E16).

---

```
======================================================================================================
MÃ HIỆU CHỈNH: CORR-10 (Mức độ: P2 - Chi tiết Kỹ thuật)
TÊN HIỆU CHỈNH: LỊCH SỬ GIỮ HÀNG, JSON SCHEMA CÓ ĐỊNH DANH & RETURN WINDOW THEO TỪNG KIỆN
======================================================================================================
```
* **Khiếm khuyết cũ trong Báo cáo M2.07:** Thiết kế giữ hàng xem như $1:1$ với dòng phân bổ khiến hệ thống không lưu được lịch sử khi khách hết hạn 15 phút rồi bấm thanh toán lại. Snapshot JSON không có ID thuộc tính dễ gây xung đột tên hiển thị. Tính thời hạn đổi trả theo toàn đơn gây rủi ro thất thoát khi giao nhiều đợt.
* **Quyết định Kiến trúc Sửa đổi:**
  1. **Quan hệ Allocation $\leftrightarrow$ Reservation là $1 : 0..N$:**
     * Một dòng phân bổ kho `OrderWarehouseAllocation` có thể trải qua nhiều lần giữ hàng `StockReservationLine`.
     * Cài đặt Filtered Unique Index:
       ```sql
       CREATE UNIQUE INDEX UX_Allocation_ActiveReservation 
       ON StockReservationLine(AllocationID) 
       WHERE ReservationLineStatus = 'ACTIVE';
       ```
     * Tổng số lượng trên các dòng `ACTIVE` $\le \text{AllocatedQty}$. Khi hết hạn thanh toán, chuyển `EXPIRED` hoặc `RELEASED`. Khách thanh toán lại $\to$ Sinh dòng `StockReservationLine` mới gắn với `AllocationID` cũ.
  2. **JSON Schema Snapshot có Định danh Thuộc tính:**
     * Cột `VariantAttributesSnapshot NVARCHAR(MAX)` trên `OrderLine` kèm Check Constraint `ISJSON(VariantAttributesSnapshot) = 1`.
     * Định dạng lưu trữ chuẩn hóa:
       ```json
       {
         "schemaVersion": 1,
         "attributes": [
           { "attributeId": 10, "attributeName": "Size", "valueId": 101, "valueDisplay": "XL" },
           { "attributeId": 11, "attributeName": "Color", "valueId": 205, "valueDisplay": "Navy Blue" }
         ]
       }
       ```
     * Lưu độc lập $100\%$ khỏi bảng từ điển master, tránh xung đột tên thuộc tính hiển thị thay đổi theo thời gian.
  3. **Return Window Tính theo Từng Kiện Hàng & Đóng Hồ sơ Hành chính:**
     * Áp dụng chính sách tính thời hạn đổi trả theo từng kiện hàng:  
       $$\text{CurrentTime} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$$
     * Bổ sung mốc thời gian `ClosedAt` trên `SalesOrder` để đóng hồ sơ kế toán/hành chính sau khi hết thời hạn khiếu nại, tách biệt khỏi máy trạng thái giao vận.
* **Thực thể Tác động:**
  * Sửa đổi quan hệ của `OrderWarehouseAllocation` (E12), `StockReservationLine` (E14).
  * Sửa đổi thuộc tính của `SalesOrder` (E40), `OrderLine` (E41), `Shipment` (E45).

---

## 4. KẾ HOẠCH ĐỒNG BỘ TÀI LIỆU TOÀN DIỆN MILESTONE 2 (DOCUMENTATION SYNC PLAN)

Để bảo đảm tính nhất quán $100\%$ của toàn bộ bộ tài liệu dự án trước khi chuyển giao sang Milestone 3, các tài liệu sau đây sẽ được đồng bộ cập nhật theo chuẩn của `M2.08`:

1. **[01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md):**
   * Bổ sung thực thể mới `PaymentWebhookLog` (E71) và `ReturnLineAssessment` (E72).
   * Mở rộng thuộc tính cho `InventoryBalance`, `InventoryTransactionLine`, `ReturnReceiptLine`, `OrderLine`, `RefundTransaction`, `Payment`.
2. **[02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md):**
   * Cập nhật bản số quan hệ `OrderWarehouseAllocation` $\leftrightarrow$ `StockReservationLine` thành $1 : 0..N$.
   * Thêm quan hệ `ReturnReceiptLine` $\leftrightarrow$ `ReturnLineAssessment` ($1 : 1..N$).
   * Cập nhật quan hệ `Refund` $\leftrightarrow$ `RefundTransaction` $\leftrightarrow$ `PaymentTransaction`.
3. **[04-Conceptual-ERD.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/04-Conceptual-ERD.md):**
   * Cập nhật sơ đồ Mermaid của Trục xương sống Backbone và các phân hệ 02, 09, 10.
4. **[07-Pre-M3-Architectural-Review.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/07-Pre-M3-Architectural-Review.md):**
   * Ghi nhận trạng thái "Phê duyệt có điều kiện đã được giải quyết triệt để" và tham chiếu bắt buộc đến bản ghi hiệu chỉnh `M2.08`.

---

## 5. BIÊN BẢN NGHIỆM THU KIẾN TRÚC HIỆU CHỈNH (AMENDED ARCHITECTURAL SIGN-OFF)

* Toàn bộ 10 vấn đề khiếm khuyết kỹ thuật (P0, P1, P2) cùng 3 quyết định kiến trúc trọng yếu đã được phân tích thấu đáo, giải quyết triệt để và chốt đặc tả thiết kế chính thức.
* Bản ghi hiệu chỉnh `M2.08` này đóng vai trò là **Đặc tả Thiết kế Bổ sung Bắt buộc (Mandatory Addendum)** cho Milestone 2, là cơ sở dữ liệu đầu vào không thể thay đổi cho Milestone 3 (Logical Database Design).
* **Trạng thái phê duyệt:** **OFFICIALLY CLEARED FOR M3 IMPLEMENTATION**.


