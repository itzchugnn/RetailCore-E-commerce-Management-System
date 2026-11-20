# BÁO CÁO TIẾN ĐỘ DỰ ÁN RETAILCORE & THUYẾT MINH KIẾN TRÚC PRE-M3
## PROJECT PROGRESS REPORT & ARCHITECTURAL BASELINE FOR MILESTONE 3

* **Tên dự án:** RetailCore – E-commerce Management System
* **Cột mốc báo cáo:** Chuyển giao từ Milestone 2 (Conceptual Design) sang Milestone 3 (Logical Design)
* **Thời điểm lập báo cáo:** 20/11/2026
* **Đơn vị lập báo cáo:** Lead Enterprise Database Engineer & Solution Architect
* **Hệ quản trị CSDL mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Căn cứ pháp lý & kỹ thuật:**
  * Toàn bộ bộ tài liệu Milestone 1: [01-Project-Charter.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/01-Project-Charter.md) đến [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md).
  * Toàn bộ bộ tài liệu Milestone 2: [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) đến [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md).

---

## 1. TỔNG QUAN ĐIỀU HÀNH & TIẾN ĐỘ DỰ ÁN (EXECUTIVE SUMMARY)

Dự án **RetailCore – Hệ thống Quản trị Bán lẻ & Thương mại Điện tử Đa kênh** đã hoàn thành toàn diện giai đoạn Nghiên cứu Phân tích Nghiệp vụ (Milestone 1) và Thiết kế Cơ sở Dữ liệu Quan niệm (Milestone 2). Hệ thống đã trải qua các đợt thẩm định độc lập nghiêm ngặt, rà soát 6 chiều kích thước kỹ thuật (Entity Ownership, Cross-domain Relationships, Cardinality, Business Rule Consistency, Lifecycle, Completeness) và chuẩn hóa dứt điểm 10 khiếm khuyết cốt lõi ở cấp độ nghiệp vụ, cấu trúc dữ liệu và giải thuật toán học.

### Bảng theo dõi tiến độ tổng thể các Cột mốc (Milestone Scorecard)

| Cột mốc (Milestone) | Phạm vi công việc | Khối lượng bàn giao | Tỷ lệ hoàn thành | Đánh giá chất lượng | Trạng thái nghiệm thu |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Milestone 1 (M1)**<br>Business Analysis | Phân tích nghiệp vụ, use cases, quy tắc nghiệp vụ, ma trận truy vết | 11 tài liệu kỹ thuật, 55 Use Cases, ~200 Business Rules, 7 Quyết định nghiệp vụ nền tảng (BD-01..07) | **100%** | Chuẩn IEEE/BABOK | **ĐÃ NGHIỆM THU ĐÓNG** |
| **Milestone 2 (M2)**<br>Conceptual Design | Mô hình hóa thực thể, quan hệ Crow's Foot, ERD, rà soát kiến trúc | 8 tài liệu kỹ thuật (v1.2), **72 Thực thể chuẩn hóa**, 40 liên kết Backbone, 12 sơ đồ Mermaid & PNG render | **100%** | Zero Defect, Zero Drift | **ĐÃ NGHIỆM THU ĐÓNG** *(Kèm M2.08 Amendment)* |
| **Milestone 3 (M3)**<br>Logical Design | Ánh xạ Relational Schema, chuẩn hóa 3NF/BCNF, định kiểu T-SQL, ràng buộc | 72 Tables, ma trận FDs, T-SQL Constraints DDL, Sizing Estimation | **0%** | — | **SẴN SÀNG KHỞI ĐỘNG** |
| **Milestone 4 (M4)**<br>Physical Design | Phân vùng (Partitioning), Indexing strategy, Concurrency, Backup/DR | SQL Server Filegroups, Columnstore, CI/CD DACPAC, High Availability | **0%** | — | Chờ M3 hoàn tất |
| **Milestone 5 – 8**<br>App Dev & Deployment | Backend API, Admin Frontend, Storefront, E2E Testing, UAT, Production Go-Live | Source code NestJS/Next.js, Test Suites, Docker, Monitoring | **0%** | — | Kế hoạch tương lai |

---

## 2. KẾT QUẢ ĐẠT ĐƯỢC TẠI MILESTONE 1 VÀ MILESTONE 2

### 2.1. Thành quả Milestone 1 (Business Analysis)
Milestone 1 đã thiết lập nền móng vững chắc với hệ thống yêu cầu nghiệp vụ được văn bản hóa chi tiết và khóa cứng phạm vi (Scope Baseline):
1. **7 Quyết định Nghiệp vụ Nền tảng (BD-01 đến BD-07):**
   * `BD-01`: Cấu trúc Danh mục sản phẩm đa tầng ($N:N$) bắt buộc đúng 1 danh mục chính (`IsPrimary = 1`).
   * `BD-02`: Bắt buộc đăng nhập xác thực tài khoản (`No Guest Checkout`).
   * `BD-03`: Hỗ trợ chia đơn giao nhiều kho (`Split Shipment`).
   * `BD-04`: Cơ chế khóa giữ tồn kho logic có thời hạn (`Reservation Timeout 15 phút`) theo nguyên tắc All-or-Nothing.
   * `BD-05`: Kiến trúc khuyến mãi & Stacking 2 tầng (cấp SKU và cấp Đơn).
   * `BD-06`: Mô hình thuộc tính biến thể động có kiểm soát (phân định `AttributeRole`, Whitelist giá trị).
   * `BD-07`: Mô hình tồn kho 4 tầng và nguyên tắc cô lập hàng giao thất bại (`DELIVERY_RETURN_IN`).
2. **55 Yêu cầu Chức năng (FR-CAT, FR-INV, FR-PO, FR-AUTH, FR-CRT, FR-ORD, FR-SHP, FR-PRM, FR-PAY, FR-RET, FR-AUD):** Được ánh xạ 100% hai chiều vào các bảng và ràng buộc CSDL.
3. **Bộ Quy tắc Nghiệp vụ (~200 Business Rules):** Định nghĩa tường minh các điều kiện biên của mọi luồng vận hành thương mại điện tử.

### 2.2. Thành quả Milestone 2 (Conceptual Database Design)
Milestone 2 đã chuyển hóa toàn bộ các yêu cầu phi cấu trúc từ M1 thành mô hình khái niệm chuẩn mực quốc tế:
1. **Chuẩn hóa Danh mục 72 Thực thể (E01 đến E72):**
   * Phân bổ đều trên 11 Phân hệ (Subject Areas 01 đến 11).
   * Bổ sung **E71 (`PaymentWebhookLog`)** tại Phân hệ 09 nhằm bảo đảm Idempotency cổng thanh toán.
   * Bổ sung **E72 (`ReturnLineAssessment`)** tại Phân hệ 10 nhằm tách rời hoàn toàn quyền hoàn tiền khỏi phân loại kho vật lý.
2. **Quy hoạch Trục Xương sống Liên miền (Cross-Domain Backbone):**
   * 40 Liên kết liên miền chuẩn hóa (CR-01 đến CR-40) kết nối các Hub Entities hạt nhân theo mô hình Hub-and-Spoke, triệt tiêu phụ thuộc vòng (Circular Dependencies) và ngăn chặn Deadlock.
3. **Hệ thống Sơ đồ Mermaid ERD & Hình ảnh Render:**
   * 12 Sơ đồ Mermaid chi tiết được chuẩn hóa cú pháp tuyệt đối (chỉ sử dụng `PK`, `FK`, `UK` đơn nhất, loại bỏ hoàn toàn các hậu tố gây lỗi render).
   * 12 File hình ảnh PNG render tương ứng được cập nhật đồng bộ trong thư mục `docs/Milestones/M2-Conceptual-Design/ERD/images/`.

---

## 3. THUYẾT MINH CHI TIẾT 10 PHƯƠNG ÁN KIẾN TRÚC ĐÃ CHỐT TRƯỚC M3

Dưới đây là phần thuyết minh chi tiết, chuyên sâu về 10 phương án kỹ thuật đã được Hội đồng Kiến trúc phê duyệt tại [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md), sẵn sàng để hiện thực hóa thành Relational Schema tại Milestone 3:

```
+--------------------------------------------------------------------------------------------------------------------+
|                                    10 KIẾN TRÚC BẢO CHỨNG TOÀN VẸN TRƯỚC M3                                        |
+--------------------------------------------------------------------------------------------------------------------+
| P0-01. Tách Hoàn tiền khỏi Tồn kho     ---> Thực thể E72: ReturnLineAssessment (FaultParty / RefundEligibility)    |
| P0-02. Tồn kho Cách ly & Hư hỏng       ---> QuarantinedQty, DamagedQty, 5 CHECK Constraints trên InventoryBalance |
| P0-03. Thuật toán LRM & Hoàn tiền F(q) ---> Largest Remainder Method, công thức số nguyên VNĐ F(q) trên OrderLine  |
| P1-04. Mandatory 1:1 & TPT Account     ---> Stored Procedure Transaction Atomic, AccountType (CUSTOMER / EMPLOYEE) |
| P1-05. Bảng đệm Webhook & Idempotency  ---> Thực thể E71: PaymentWebhookLog, OriginalPaymentTransactionID          |
| P1-06. Độc lập 3 Máy Trạng thái        ---> SalesOrder, Payment, Shipment độc lập; DeliveredAt, ClosedAt           |
| P1-07. Đối soát COD Hai Chặng          ---> Chặng 1: Khách trả (PAID); Chặng 2: Công nợ Carrier (SettlementStatus) |
| P2-08. Giá vốn WAC khi Trả hàng NCC    ---> SupplierReturn giảm tồn theo CurrentAverageCost, ghi nhận Cost Variance|
| P2-09. Chuẩn hóa JSON Snapshot         ---> VariantAttributesSnapshot lưu mảng JSON định danh attributeId / valueId|
| P2-10. Vòng đời Giữ hàng & Return Win  ---> Allocation-Reservation 1:0..N (Filtered Index ACTIVE), Hạn theo Kiện    |
+--------------------------------------------------------------------------------------------------------------------+
```

---

### Phương án 1 (P0): Tách biệt Tuyệt đối Quyền Hoàn tiền (`RefundEligibility`) khỏi Định đoạt Kho (`InventoryDisposition`) qua `ReturnLineAssessment` (E72)

#### 1. Rủi ro nếu thiết kế non tay:
Trước đây, mô hình quy định: *"Chỉ sau khi ReturnReceipt có dòng Disposition = SELLABLE được POSTED, hệ thống mới tự động tạo Refund"*. Đây là một sai sót nghiệp vụ nghiêm trọng. Xét tình huống khách mua áo bị lỗi đường chỉ từ nhà sản xuất; khi trả hàng về kho, kiểm định xác nhận lỗi và phân loại `DISPOSED` (tiêu hủy). Nếu bắt buộc hàng phải `SELLABLE` mới hoàn tiền thì khách hàng bị tước đoạt quyền lợi hợp pháp, dẫn đến khiếu nại gay gắt và vi phạm luật bảo vệ người tiêu dùng.

#### 2. Quyết định kiến trúc đã chốt:
Tách rời hoàn toàn hai chiều thẩm định độc lập:
* **Định đoạt vật lý tại kho (`InventoryDisposition`):** Xử lý số lượng hàng hóa thực nhận trong kho vật lý: `SELLABLE`, `QUARANTINE`, `DISPOSED`.
* **Xét duyệt điều kiện hoàn tiền (`RefundEligibility`):** Đánh giá nguyên nhân lỗi và chính sách bảo hành của RetailCore: `PENDING`, `APPROVED`, `REJECTED`.

#### 3. Cấu trúc thực thể mới `ReturnLineAssessment` (E72):
Bổ sung thực thể E72 trực thuộc Subject Area 10, đóng vai trò liên kết giữa tiếp nhận hàng trả tại kho (`ReturnReceiptLine`) với hồ sơ hoàn tiền (`Refund`):
* `AssessmentID` (bigint PK)
* `ReturnReceiptLineID` (bigint FK trỏ về E63)
* `AssessedQty` (int, $\sum \text{AssessedQty} \le \text{ReturnReceiptLine.ReceivedQty}$)
* `FaultParty` (varchar: `VENDOR`, `CARRIER`, `CUSTOMER`, `NO_FAULT`)
* `RefundEligibilityStatus` (varchar: `PENDING`, `APPROVED`, `REJECTED`)
* `ResolutionType` (varchar: `REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION`)
* `ApprovedRefundAmount` (decimal(18,2))
* `ShippingRefundAmount` (decimal(18,2))
* `AssessedBy` (bigint FK), `AssessedAt` (datetimeoffset), `AssessmentNotes` (nvarchar)

#### 4. Quy tắc vận hành & Chuyển tiếp M3:
* **Hàng lỗi do Nhà sản xuất (`VENDOR`) hoặc Vận chuyển (`CARRIER`):** Dù `Disposition = 'DISPOSED'` (hàng nát không thể bán lại), `RefundEligibilityStatus` vẫn được duyệt `APPROVED` và hoàn trả 100% tiền hàng kèm phí vận chuyển.
* **Hỗ trợ hoàn tiền từng mốc (Milestone Partial Refund):** Hệ thống kích hoạt tạo chứng từ `Refund` ngay khi có dòng `ReturnLineAssessment` được `APPROVED` cho số lượng thực nhận, không bắt buộc phải chờ toàn bộ bưu kiện của yêu cầu RMA về hết kho mới được hoàn tiền.

---

### Phương án 2 (P0): Quản lý Tồn kho Cách ly (Quarantine) & Hư hỏng (Damaged) với Bộ 5 Ràng buộc CHECK

#### 1. Rủi ro nếu thiết kế non tay:
Ở Subject Area 02, RetailCore quản lý tồn kho ở cấp độ Warehouse (không quản lý chi tiết đến vị trí Kệ/Bin/Pallet). Nếu chỉ có 2 trường `OnHandQty` và `ReservedQty`, toàn bộ hàng lỗi chờ thẩm định hãng hoặc hàng hỏng chờ thanh lý sẽ bị gộp chung vào `OnHandQty`. Kết quả là hàng hỏng vẫn được tính vào `AvailableQty` và hệ thống tiếp tục bán món đồ hỏng đó cho khách hàng mới, gây tê liệt vận hành.

#### 2. Quyết định kiến trúc đã chốt:
Phân rã số dư tồn kho tại từng kho vật lý thành **3 trạng thái chất lượng nội bộ**:
1. `SELLABLE` (Hàng đạt chuẩn, sẵn sàng mở bán).
2. `QUARANTINED` (Hàng đang cách ly, chờ giám định hãng hoặc hàng hoàn trả chưa thẩm định).
3. `DAMAGED` (Hàng xác định hư hỏng, chờ lập thủ tục xuất hủy hoặc thanh lý phụ tùng).

#### 3. Cấu trúc dữ liệu & Bộ 5 Declarative CHECK Constraints trên `InventoryBalance` (E11):
Mở rộng bảng `InventoryBalance` gồm: `OnHandQty`, `ReservedQty`, `QuarantinedQty`, `DamagedQty`, `CurrentAverageCost`.  
Trên SQL Server, cài đặt bộ 5 ràng buộc `CHECK`:
1. `CHECK (OnHandQty >= 0)`
2. `CHECK (ReservedQty >= 0)`
3. `CHECK (QuarantinedQty >= 0)`
4. `CHECK (DamagedQty >= 0)`
5. `CHECK (OnHandQty >= ReservedQty + QuarantinedQty + DamagedQty)`

Công thức tồn khả dụng mở bán an toàn:
$$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$$

#### 4. Ghi sổ kép đa chiều trên `InventoryTransactionLine` (E16):
Bổ sung các cột: `QuarantinedDelta`, `DamagedDelta`, `FromStockStatus`, `ToStockStatus`, `UnitCost`.
* **Luồng chuyển trạng thái nội bộ:** Khi chuyển 5 cái từ Cách ly sang Hỏng: `QuarantinedDelta = -5`, `DamagedDelta = +5`, `OnHandDelta = 0`, `FromStockStatus = 'QUARANTINED'`, `ToStockStatus = 'DAMAGED'`.
* **Luồng xuất hủy (`SCRAP_OUT`):** Khi tiêu hủy 5 cái hỏng khỏi kho: `OnHandDelta = -5`, `DamagedDelta = -5`, `FromStockStatus = 'DAMAGED'`, `ToStockStatus = NULL`.

---

### Phương án 3 (P0): Thuật toán Phân bổ Chiết khấu Đơn hàng LRM & Công thức Hoàn tiền Nguyên số $F(q)$

#### 1. Rủi ro nếu thiết kế non tay:
Đơn hàng áp Voucher cấp đơn (Order-level Voucher). Nếu phân bổ tiền giảm giá xuống từng dòng hàng bằng hàm `Round()` thông thường thì tổng chiết khấu các dòng sẽ lệch từ vài đồng đến vài trăm đồng so với giá trị voucher do lỗi làm tròn (Rounding Error). Hơn nữa, khi khách trả hàng lẻ từng cái (ví dụ: mua 3 cái áo cùng dòng được giảm tổng cộng 50.000đ, nhưng khách chỉ trả 1 cái trước, 2 cái trả sau), việc thiếu công thức toán học xác định sẽ gây tranh chấp số tiền hoàn và làm vỡ sổ cái kế toán.

#### 2. Quyết định kiến trúc đã chốt:
1. **Thuật toán Largest Remainder Method (Hamilton/Hare-Niemeyer):**
   * Tính phần nguyên và phần dư tiền giảm giá voucher cho từng `OrderLine` theo tỷ trọng giá trị.
   * Phân bổ phần nguyên, sau đó xếp hạng phần dư giảm dần và chia bù mỗi dòng 1 VNĐ cho đến khi khớp 100% giá trị voucher:
     $$\sum_{i=1}^{n} \text{AllocatedOrderDiscount}_i \equiv \text{VoucherDiscountAmount}$$
2. **Lưu trữ trực tiếp trên `OrderLine` (E41):**
   * `LineGrossAmount = UnitPrice * OrderedQty`
   * `LineProductDiscount` (giảm giá tầng 1 theo SKU)
   * `AllocatedOrderDiscount` (giảm giá tầng 2 từ voucher phân bổ theo LRM)
   * `LineNetAmount = LineGrossAmount - LineProductDiscount - AllocatedOrderDiscount`
   * `RefundedQty` (số lượng lũy kế đã hoàn trả)
   * `AccumulatedRefundedAmount` (số tiền lũy kế đã hoàn trả)
3. **Công thức Hoàn tiền Lũy tiến Nguyên số (Unit-Level Progressive Refund Formula):**
   Với dòng hàng có số lượng đặt $Q$ và giá trị thanh toán ròng $\text{LineNetAmount}$:
   $$B = \left\lfloor \frac{\text{LineNetAmount}}{Q} \right\rfloor \quad (\text{Mức hoàn cơ bản mỗi cái}), \quad R = \text{LineNetAmount} \pmod{Q} \quad (\text{Số tiền dư lẻ})$$
   Hàm số xác định tổng tiền hoàn khi khách trả $q$ đơn vị ($1 \le q \le Q$):
   $$F(q) = B \cdot q + \min(q, R)$$
   Khi khách trả thêm $b$ đơn vị sau khi đã từng hoàn $a$ đơn vị trước đó:
   $$\Delta \text{Refund} = F(a + b) - F(a)$$

#### 3. Minh chứng toán học:
* Mọi khoản hoàn tiền $\Delta \text{Refund}$ luôn là **số nguyên VNĐ** tuyệt đối.
* Khi khách trả hết hàng ($q = Q$): $F(Q) = B \cdot Q + R \equiv \text{LineNetAmount}$ (bảo đảm hoàn đúng 100% số tiền thực thu, không lệch một xu).

---

### Phương án 4 (P1): Ràng buộc Bắt buộc 1:1 qua Stored Procedure Giao dịch Nguyên tử & Kế thừa Table-Per-Type (TPT)

#### 1. Rủi ro nếu thiết kế non tay:
Nhiều kỹ sư CSDL lầm tưởng rằng chỉ cần khai báo `FOREIGN KEY` kèm `UNIQUE` trên bảng con (ví dụ: `Payment.OrderID UNIQUE REFERENCES SalesOrder(OrderID)`) là đã đảm bảo quan hệ Bắt buộc 1:1 (Mandatory 1:1). Trên thực tế, DDL Declarative chỉ đảm bảo cận trên $\le 1$ (tối đa 1 bản ghi thanh toán cho mỗi đơn), nhưng **không thể ngăn chặn một SalesOrder tồn tại mà không có Payment nào** (cận dưới vẫn là $0$). Tương tự với mô hình chuyên biệt hóa `Account` $\leftrightarrow$ `Customer`/`Employee`.

#### 2. Quyết định kiến trúc đã chốt:
1. **Kiểm soát Mandatory 1:1 qua Stored Procedure giao dịch nguyên tử:**
   * Nghiêm cấm ứng dụng gọi lệnh `INSERT` riêng rẽ vào bảng `SalesOrder` hoặc `Account`.
   * Mọi thao tác tạo đơn hoặc tạo tài khoản bắt buộc phải đi qua Stored Procedure trung tâm (`sp_CreateSalesOrder`, `sp_CreateAccount`).
   * Sử dụng Transaction nguyên tử (`BEGIN TRANSACTION ... COMMIT TRANSACTION`) với `XACT_ABORT ON`: Tạo đồng thời `SalesOrder` + `Payment` + `OrderAddress` trong một transaction duy nhất. Bất kỳ lỗi nào phát sinh sẽ rollback 100%, triệt tiêu nguy cơ đơn hàng mồ côi không có hồ sơ thanh toán.
2. **Mô hình Kế thừa Table-Per-Type (TPT) cho Account:**
   * Bảng cha `Account` chứa thuộc tính phân loại subtype: `AccountType VARCHAR(10) NOT NULL` (`'CUSTOMER'` hoặc `'EMPLOYEE'`).
   * Bảng con `Customer` và `Employee` sử dụng `AccountID PK_FK` làm khóa chính kiêm khóa ngoại trỏ về `Account(AccountID)`.
   * Cài đặt Trigger hoặc Check Constraint loại trừ: Khi `AccountType = 'CUSTOMER'` thì chỉ được phép tồn tại bản ghi ở bảng `Customer`, cấm xuất hiện ở bảng `Employee` và ngược lại.

---

### Phương án 5 (P1): Bảng đệm `PaymentWebhookLog` (E71), Xử lý Idempotent & Tự tham chiếu Hoàn tiền

#### 1. Rủi ro nếu thiết kế non tay:
Các cổng thanh toán trực tuyến (VNPAY, MoMo, ZaloPay, Stripe) luôn gửi Webhook Callback với cơ chế retry (gửi lại nhiều lần nếu mạng chậm). Nếu ghi nhận trực tiếp vào bảng giao dịch mà không có bảng đệm Idempotent, hệ thống sẽ gặp rủi ro xử lý 2 lần (Double-crediting), ghi nhận thừa tiền cho khách hoặc bị hacker tấn công Replay Attack. Ngoài ra, các giao dịch hoàn tiền (`RefundTransaction`) nếu không liên kết với giao dịch thanh toán gốc sẽ không thể đối soát phí cổng khi hoàn tiền.

#### 2. Quyết định kiến trúc đã chốt:
1. **Bổ sung thực thể `PaymentWebhookLog` (E71):**
   * Đóng vai trò là bảng đệm (Staging Buffer) tiếp nhận payload thô trước khi xử lý nghiệp vụ:
     `WebhookLogID` (PK), `PaymentMethodID` (FK), `PaymentProvider`, `EventID`, `EventType`, `RawPayload`, `Signature`, `ProcessedStatus` (`PENDING`, `PROCESSED`, `FAILED`, `IGNORED`), `ReceivedAt`, `ProcessedAt`, `ErrorMessage`.
   * Cài đặt Filtered Unique Index trên SQL Server:
     ```sql
     CREATE UNIQUE INDEX UX_WebhookLog_ProviderEvent 
     ON PaymentWebhookLog(PaymentProvider, EventID) 
     WHERE EventID IS NOT NULL;
     ```
   * Khi Webhook thứ hai có cùng `EventID` gửi đến, lệnh Insert sẽ kích hoạt vi phạm Unique Index ngay lập tức, hệ thống trả về mã HTTP 200 OK cho cổng và bỏ qua việc xử lý lặp lại (Idempotency).
2. **Liên kết giao dịch hoàn tiền gốc (`RefundTransaction.OriginalPaymentTransactionID`):**
   * `RefundTransaction` bổ sung cột `OriginalPaymentTransactionID BIGINT NOT NULL REFERENCES PaymentTransaction(PaymentTransactionID)`.
   * Đảm bảo mọi giao dịch hoàn tiền đều truy vết ngược được chính xác giao dịch thu tiền ban đầu để đối soát hoàn phí cổng (Gateway Fee Reversal).

---

### Phương án 6 (P1): Tách biệt Ba Máy Trạng thái Độc lập & Quản lý Đóng sổ Đơn hàng

#### 1. Rủi ro nếu thiết kế non tay:
Gộp chung trạng thái đơn hàng, trạng thái thanh toán và trạng thái giao hàng vào một cột `OrderStatus` duy nhất khiến hệ thống rơi vào trạng thái bế tắc (State explosion): Không thể mô hình hóa được đơn COD (đã giao nhưng chưa nhận tiền từ shipper), đơn Split Shipment (giao 2 kiện thì 1 kiện thành công, 1 kiện giao thất bại), hoặc đơn giao lại nhiều lần.

#### 2. Quyết định kiến trúc đã chốt:
1. **Tách biệt 3 máy trạng thái giao dịch độc lập tuyệt đối:**
   $$\text{SalesOrder.OrderStatus} \neq \text{Payment.PaymentStatus} \neq \text{Shipment.ShipmentStatus}$$
   * `SalesOrder.OrderStatus`: Phản ánh tiến độ giao kết hợp đồng bán hàng: `PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `DELIVERED` $\to$ `COMPLETED` (hoặc `CANCELLED`).
   * `Payment.PaymentStatus`: Phản ánh nghĩa vụ thanh toán của khách: `PENDING` $\to$ `PAID` $\to$ `PARTIALLY_REFUNDED` $\to$ `REFUNDED` (hoặc `FAILED`).
   * `Shipment.ShipmentStatus`: Phản ánh quá trình vận chuyển bưu kiện: `PREPARING` $\to$ `READY_TO_DISPATCH` $\to$ `IN_TRANSIT` $\to$ `DELIVERED` (hoặc `DELIVERY_FAILED` $\to$ `RETURNING` $\to$ `RETURNED`).
2. **Quy tắc chuyển trạng thái & Mốc thời gian đóng sổ (`DeliveredAt`, `ClosedAt`):**
   * Đơn hàng COD được chuyển sang `CONFIRMED` ngay khi duyệt đơn hợp lệ, không phụ thuộc vào `PaymentStatus`.
   * Cột `SalesOrder.DeliveredAt`: Cập nhật tự động bằng $\max(\text{Shipment.DeliveredAt})$ khi toàn bộ các kiện hàng của đơn đã giao thành công. Mốc này dùng làm căn cứ tính thời hạn đổi trả hàng (Return Window).
   * Cột `SalesOrder.ClosedAt`: Sau khi hết thời hạn khiếu nại (ví dụ: 7 ngày sau `DeliveredAt`), Job tự động chuyển `OrderStatus = 'COMPLETED'` và đóng dấu `ClosedAt = NOW()`. Khi đã `COMPLETED`, khóa vĩnh viễn không cho phép tạo thêm RMA.

---

### Phương án 7 (P1): Mô hình Đối soát COD Hai Chặng Độc lập (Two-Sided Reconciliation)

#### 1. Rủi ro nếu thiết kế non tay:
Nếu coi việc khách trả tiền mặt cho shipper là công ty đã nhận được tiền thì kế toán sẽ bị ghi nhận thừa doanh thu tiền mặt khả dụng. Ngược lại, nếu coi khi nào shipper nộp tiền về công ty thì khách mới được coi là thanh toán thì đơn hàng của khách sẽ bị treo ở trạng thái nợ tiền (`PENDING`) dù khách đã trả tiền đầy đủ, tước đoạt quyền tạo RMA đổi trả của khách.

#### 2. Quyết định kiến trúc đã chốt:
Phân rã chu trình COD thành **2 chặng kế toán độc lập**:
* **Chặng 1 (Nghĩa vụ thanh toán của Người mua):**
  * Ngay khi shipper giao hàng thành công và thu tiền mặt (`ShipmentStatus = 'DELIVERED'`): Người mua đã hoàn tất 100% nghĩa vụ tài chính theo pháp luật thương mại.
  * Hệ thống cập nhật ngay: `Payment.PaymentStatus = 'PAID'` và `Payment.PaidAt = DeliveredAt`. Khách hàng lập tức có đầy đủ quyền lợi đánh giá sản phẩm hoặc yêu cầu đổi trả theo chính sách.
* **Chặng 2 (Công nợ thu hộ giữa RetailCore và Đối tác Vận chuyển):**
  * Khoản tiền mặt đang do shipper/hãng vận chuyển nắm giữ là tài sản thuộc Khoản phải thu (Accounts Receivable) của sàn.
  * Quản lý độc lập qua thuộc tính `Payment.CarrierSettlementStatus`:
    $$\text{NOT\_COLLECTED} \longrightarrow \text{COLLECTED\_BY\_CARRIER} \longrightarrow \text{REMITTED} \longrightarrow \text{RECONCILED}$$
  * Khi đối soát bưu tá định kỳ, kế toán tải bảng kê đối soát lên hệ thống: Ghi nhận bản ghi `PaymentTransaction` loại `COD_SETTLEMENT` và chuyển `CarrierSettlementStatus = 'RECONCILED'`.

---

### Phương án 8 (P2): Giá vốn Bình quân Di động (Moving Weighted Average - WAC) khi Xuất trả Nhà Cung cấp

#### 1. Rủi ro nếu thiết kế non tay:
Khi xuất trả hàng cho nhà cung cấp (`SupplierReturn`), nếu trừ giá trị tồn kho theo giá mua ban đầu trên đơn đặt hàng PO (`PurchaseOrderLine.UnitPrice`), trong khi tồn kho trong kho đang được định giá theo phương pháp Giá vốn Bình quân Di động (WAC), thì tổng giá trị tài sản trên sổ cái kế toán kho sẽ bị lệch nghiêm trọng so với thực tế, gây méo mó báo cáo tài chính và giá vốn hàng bán (COGS).

#### 2. Quyết định kiến trúc đã chốt:
1. **Nguyên tắc hạch toán xuất trả hàng nhà cung cấp:**
   * Khi thực hiện xuất trả hàng cho nhà cung cấp, hệ thống ghi nhận giao dịch xuất kho với đơn giá vốn xuất kho bằng đúng **Giá vốn Bình quân Di động tại thời điểm xuất trả** (`InventoryBalance.CurrentAverageCost`).
   * Giá trị tồn kho bị giảm = $\text{ReturnedQty} \times \text{CurrentAverageCost}$.
2. **Xử lý chênh lệch giá vốn (Cost Variance Adjustment):**
   * Số tiền thực tế nhà cung cấp hoàn trả cho RetailCore tính theo đơn giá mua thỏa thuận ban đầu trên PO (`PurchaseOrderLine.UnitPrice`).
   * Khoản chênh lệch giữa giá trị PO hoàn lại và giá trị tồn kho bị giảm được hạch toán tự động vào tài khoản điều chỉnh chênh lệch giá vốn:
     $$\text{CostVariance} = \text{ReturnedQty} \times (\text{POLine.UnitPrice} - \text{CurrentAverageCost})$$
   * Đảm bảo sổ cái kho luôn cân bằng tuyệt đối với tổng số dư vật lý nhân đơn giá bình quân di động.

---

### Phương án 9 (P2): Chuẩn hóa JSON Snapshot Biến thể có Định danh Cấu trúc

#### 1. Rủi ro nếu thiết kế non tay:
Nhiều hệ thống lưu snapshot thuộc tính biến thể bằng chuỗi văn bản tự do hoặc JSON phi cấu trúc, ví dụ: `{"Size": "L", "Color": "Navy Blue"}`. Thiết kế này có 2 nhược điểm chết người: (1) Nếu danh mục đổi tên nhãn "Color" thành "Màu sắc", hệ thống không thể chạy truy vấn tổng hợp hoặc tìm kiếm thống nhất; (2) Không thể ánh xạ ngược lại `AttributeID` và `AttributeValueID` phục vụ phân tích dữ liệu mua sắm (Faceted Analytics) hoặc tái sinh SKU.

#### 2. Quyết định kiến trúc đã chốt:
Chuẩn hóa trường `OrderLine.VariantAttributesSnapshot NVARCHAR(MAX)` thành **mảng JSON có định danh ID và mã Code cấu trúc chuẩn**:
```json
[
  {
    "attributeId": 1,
    "attributeCode": "SIZE",
    "attributeName": "Kích cỡ",
    "valueId": 105,
    "valueCode": "L",
    "displayValue": "Size L"
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
* **Lợi ích kỹ thuật:** Cho phép lập chỉ mục và truy vấn tốc độ cao trên SQL Server thông qua các hàm tích hợp sẵn như `JSON_VALUE`, `JSON_QUERY` và `OPENJSON()`, phục vụ các báo cáo phân tích xu hướng tiêu dùng theo thuộc tính mà không làm chậm hệ thống.

---

### Phương án 10: Vòng đời Giữ hàng $1 : 0..N$ qua Filtered Index & Hạn Đổi trả theo Từng Kiện hàng

#### 1. Rủi ro nếu thiết kế non tay:
* **Về giữ hàng:** Nếu ép quan hệ giữa `OrderWarehouseAllocation` và `StockReservationLine` là $1:0..1$ bằng ràng buộc `AllocationID UNIQUE`, hệ thống sẽ không thể lưu vết lịch sử nếu khách hàng thanh toán lỗi lần đầu dẫn đến hết hạn giữ hàng (EXPIRED), sau đó khách bấm thanh toán lại và tạo lượt giữ hàng mới.
* **Về thời hạn đổi trả:** Đối với đơn chia nhiều kiện xuất từ nhiều kho (Split Shipment), kiện thứ nhất giao ngày 01/10, kiện thứ hai giao ngày 05/10. Nếu tính thời hạn đổi trả 7 ngày chung cho cả đơn theo ngày giao của kiện cuối cùng thì kiện thứ nhất được hưởng thời hạn trả hàng lên tới 11 ngày; ngược lại nếu tính theo kiện đầu tiên thì kiện thứ hai chỉ còn 3 ngày để đổi trả.

#### 2. Quyết định kiến trúc đã chốt:
1. **Quan hệ $1 : 0..N$ với Filtered Unique Index cho Giữ hàng tồn kho:**
   * Một dòng phân bổ `OrderWarehouseAllocation` có thể liên kết với nhiều dòng giữ hàng `StockReservationLine` theo thời gian ($1 : 0..N$), lưu vết đầy đủ các dòng giữ hàng cũ đã hết hạn (`EXPIRED`) hoặc đã hủy (`RELEASED`).
   * Cài đặt Filtered Unique Index trên SQL Server để bảo đảm **tại một thời điểm chỉ có tối đa 1 dòng giữ hàng đang hoạt động**:
     ```sql
     CREATE UNIQUE INDEX UX_ReservationLine_ActiveAlloc 
     ON StockReservationLine(OrderWarehouseAllocationID) 
     WHERE ReservationLineStatus = 'ACTIVE';
     ```
2. **Tính thời hạn đổi trả hàng theo từng kiện hàng (`Shipment.DeliveredAt`):**
   * Ràng buộc ứng dụng kiểm tra thời hạn đổi trả RMA tính theo ngày giao thành công của chính kiện hàng chứa dòng sản phẩm đó:
     $$\text{DATEDIFF}(\text{day}, \text{Shipment.DeliveredAt}, \text{ReturnRequest.RequestedAt}) \le \text{PolicyDays}$$
   * Đảm bảo sự công bằng tuyệt đối cho khách hàng và tính chặt chẽ trong kiểm soát rủi ro của từng kho xuất hàng.

---

## 4. KẾ HOẠCH HÀNH ĐỘNG CHI TIẾT CHO MILESTONE 3 (LOGICAL DATABASE DESIGN)

Sau khi toàn bộ 10 phương án kiến trúc M2 đã được khóa cứng, các hạng mục kỹ thuật của **Milestone 3 (Logical Database Design)** sẽ được khởi động ngay theo lộ trình 4 bước chuẩn mực:

```
+-------------------------------------------------------------------------------------------------------------+
|                                    LỘ TRÌNH THỰC THI MILESTONE 3                                            |
+-------------------------------------------------------------------------------------------------------------+
| M3.01: Relational Schema Mapping    ---> Chuyển đổi 72 Thực thể thành 72 Bảng quan hệ trên SQL Server       |
| M3.02: 3NF & BCNF Normalization     ---> Chứng minh toán học mọi bảng đạt chuẩn BCNF qua Phụ thuộc hàm (FD) |
| M3.03: SQL Server Types & Sizing    ---> Định kiểu dữ liệu tối ưu & Ước tính dung lượng lưu trữ 1M đơn/năm  |
| M3.04: Declarative Constraints DDL  ---> Đặc tả chi tiết mã T-SQL: CHECK, Filtered Unique Index, Stored Proc|
+-------------------------------------------------------------------------------------------------------------+
```

### Bước 1 (M3.01) – Relational Schema Mapping (Ánh xạ Lược đồ Quan hệ)
* Chuyển đổi 72 thực thể thành 72 bảng dữ liệu quan hệ (Relations/Tables) thuộc schema quản lý chuẩn hóa (ví dụ: `sales.*`, `inventory.*`, `customer.*`, `payment.*`, `catalog.*`, `audit.*`).
* Xác định chính xác chiến lược Khóa chính: Khóa thay thế tự tăng (`BIGINT IDENTITY(1,1)`) cho toàn bộ các chứng từ giao dịch; Khóa tự nhiên duy nhất (`Natural Business Key UK`) cho các mã nghiệp vụ (`OrderNumber`, `SKU`, `WarehouseCode`, `PONumber`).
* Thiết lập hệ thống Khóa ngoại (Foreign Keys) với quy tắc toàn vẹn tham chiếu nghiêm ngặt: Mặc định `ON DELETE NO ACTION` đối với toàn bộ các chứng từ lịch sử và sổ cái tài chính.

### Bước 2 (M3.02) – Chuẩn hóa Dữ liệu Toán học (3NF & BCNF Normalization)
* Xây dựng ma trận Phụ thuộc hàm (Functional Dependencies - FDs) cho từng bảng.
* Chứng minh toán học rằng mọi quan hệ đều đạt **Chuẩn 3 (3NF)** và **Chuẩn Boyce-Codd (BCNF)**: Mọi thuộc tính không khóa đều phụ thuộc đầy đủ và trực tiếp vào khóa chính, loại bỏ hoàn toàn các phụ thuộc bắc cầu và dư thừa dữ liệu bất hợp lý.
* Ghi nhận và bảo vệ các trường Denormalization có kiểm soát (như `CurrentAverageCost` trên `InventoryBalance` hoặc các cột snapshot bất biến trên `OrderLine`) với các cơ chế kiểm soát đồng bộ chặt chẽ.

### Bước 3 (M3.03) – Định kiểu Dữ liệu SQL Server & Ước tính Dung lượng Lưu trữ (Data Types & Sizing)
* Lựa chọn kiểu dữ liệu tối ưu trên Microsoft SQL Server:
  * Khóa chính/Khóa ngoại: `BIGINT` (8 bytes).
  * Tiền tệ & Giá trị tài chính: `DECIMAL(18,2)` (9 bytes) - nghiêm cấm dùng `FLOAT`/`REAL` để tránh sai số dấu phẩy động.
  * Chuỗi văn bản Unicode: `NVARCHAR(n)` (họ tên, địa chỉ, mô tả); Chuỗi mã hệ thống: `VARCHAR(n)` (`Status`, `Code`, `Type`).
  * Thời gian & Múi giờ: `DATETIMEOFFSET` (10 bytes) - đảm bảo chuẩn hóa thời gian đa múi giờ cho hệ thống bán lẻ đa kênh.
  * Kiểm soát Concurrency: Cột `ROWVERSION` (`TIMESTAMP`) cho các bảng giao dịch có tần suất cập nhật cao để hỗ trợ Optimistic Concurrency Control.
* Bảng tính toán dung lượng lưu trữ chi tiết: Ước tính kích thước hàng (Row Size), dung lượng bảng và dung lượng chỉ mục cho quy mô vận hành chuẩn **1.000.000 đơn hàng/năm**, tính toán tốc độ tăng trưởng của Transaction Log và phân bổ Filegroup.

### Bước 4 (M3.04) – Đặc tả Chi tiết Ràng buộc Kỹ thuật & Chỉ mục có lọc (Constraints & Filtered Indexes Specification)
* Soạn thảo chi tiết mã lệnh DDL của:
  * Bộ 5 Declarative CHECK Constraints trên bảng `InventoryBalance`.
  * Bộ 8 Filtered Unique Indexes (Active cart, Active reservation, Webhook raw buffer, Primary category, Unique email/phone).
  * Khung mã lệnh Stored Procedure chuẩn cho giao dịch nguyên tử tạo đơn hàng (Mandatory 1:1 `SalesOrder` + `Payment` + `OrderAddress`).

---

## 5. ĐÁNH GIÁ RỦI RO & TUYÊN BỐ SẴN SÀNG (RISK & READINESS SIGN-OFF)

| Rủi ro kỹ thuật khi chuyển giao M3 | Mức độ | Biện pháp kiểm soát & Giảm thiểu tại M3 |
| :--- | :---: | :--- |
| **1. Tranh chấp khóa giữ hàng (Concurrency Lock Contention)** | Trung bình | Sử dụng Pessimistic Locking với gợi ý khóa `UPDLOCK, ROWLOCK` trong Stored Procedure checkout để tránh Deadlock khi nhiều khách cùng mua 1 SKU cuối cùng. |
| **2. Phân bổ chiết khấu lẻ VNĐ** | Thấp | Giải thuật LRM và công thức $F(q)$ đã được chứng minh toán học khép kín; M3 chỉ cần chuyển dịch nguyên văn thành hàm Scalar User-Defined Function (UDF) hoặc xử lý trong Stored Procedure. |
| **3. Phình to dung lượng Webhook Log** | Thấp | Cấu hình chính sách Retention Policy tại M4: Tự động dọn dẹp hoặc chuyển archive các bản ghi `PaymentWebhookLog` đã xử lý thành công (`PROCESSED`) sau 90 ngày. |

### Tuyên bố Nghiệm thu của Lead Database Engineer
> **KẾT LUẬN CUỐI CÙNG:**
> 
> Giai đoạn **Milestone 2 – Conceptual Database Design** của dự án RetailCore đã hoàn thành xuất sắc, đạt độ hoàn thiện **100% Zero Defect**. Toàn bộ 72 thực thể, 40 liên kết liên miền và 10 phương án kiến trúc cốt lõi đã được kiểm chứng chéo và phê duyệt đồng thuận.
> 
> **Hệ thống chính thức đủ điều kiện đóng Gate và chuyển giao toàn diện sang Milestone 3 – Logical Database Design.**

