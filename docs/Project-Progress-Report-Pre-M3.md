# BÁO CÁO TIẾN ĐỘ DỰ ÁN RETAILCORE & THUYẾT MINH KIẾN TRÚC PRE-M3 (PHIÊN BẢN v2.1)
## PROJECT PROGRESS REPORT & ARCHITECTURAL BASELINE FOR MILESTONE 3 - CONDITIONAL GATE REVIEW

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Quy mô Dự phóng:** 1.000.000 đơn hàng/năm, 10.000.000 biến động kho/năm, tính sẵn sàng 99.99%
* **Phiên bản Báo cáo:** v2.1 (Cập nhật và hoàn thiện toàn diện theo Đánh giá Kiến trúc Chuyên sâu ngày 21/11/2026)
* **Trạng thái Cổng M2:** **CONDITIONAL APPROVAL FOR M3 (Nghiệm thu Chuyển giao Có Điều kiện)**
* **Phê duyệt:** Lead Enterprise Database Engineer & Solution Architect

---

## 1. TỔNG QUAN ĐIỀU HÀNH & ĐÁNH GIÁ CỔNG M2 (EXECUTIVE SUMMARY & GATE REVIEW)

### 1.1. Tinh thần Quản trị Kiến trúc và Nguyên tắc Chuyển giao
Phiên bản báo cáo **v2.1** tiếp tục nâng cao tính chính xác kỹ thuật của hệ thống RetailCore. Đội ngũ kỹ thuật xác lập nguyên tắc quản trị kiến trúc thực chứng:
1. **M2 Baseline là nền tảng quan niệm vững chắc nhưng mở:** M2 đóng vai trò xác lập ranh giới nghiệp vụ (bounded contexts), định danh thực thể cốt lõi và các mối quan hệ nghiệp vụ liên miền. 
2. **Milestone 3 (M3 - Logical Database Design) là giai đoạn kiểm chứng và phát hiện:** Quá trình chuyển từ mô hình khái niệm sang mô hình quan hệ toán học (Candidate Keys, Functional Dependencies, 3NF/BCNF, Physical Data Types, Declarative Constraints) sẽ tiếp tục thử thách, kiểm chứng và hoàn thiện các quyết định của M2.
3. **Chấp thuận Cổng có Điều kiện (Conditional Gate Approval):** Dự án được phép bắt đầu ngay công đoạn **M3.01 (Relational Schema Mapping)**; đồng thời giữ tính linh hoạt và mở đối với các công đoạn **M3.02 (Normalization)**, **M3.03 (Data Types & Sizing)**, và **M3.04 (Constraints DDL & SPs)** cho đến khi các quy tắc chi tiết về kiểm toán, đối soát COD đa kiện và phân bổ dòng giao hàng được chốt hạ hoàn toàn.

---

### 1.2. Bảng theo dõi tiến độ tổng thể các Cột mốc (Milestone Scorecard)

| Cột mốc (Milestone) | Trọng số | Trạng thái Hiện tại | Kết quả Nghiệm thu Kỹ thuật | Ghi chú Quản trị Cổng |
| :--- | :---: | :---: | :--- | :--- |
| **M1: Phân tích Nghiệp vụ & Yêu cầu** *(Business Analysis)* | 20% | **HOÀN THÀNH (100%)** | • 10 Phân hệ nghiệp vụ cốt lõi.<br>• 112 Quy tắc nghiệp vụ (BR) được mã hóa.<br>• Từ điển dữ liệu nghiệp vụ chuẩn hóa. | Đã ký nghiệm thu Baseline M1. |
| **M2: Thiết kế Cơ sở Dữ liệu Mức Quan niệm** *(Conceptual Design)* | 30% | **NGHIỆM THU CÓ ĐIỀU KIỆN (Conditional Approval)** | • 72 Thực thể quan niệm (E01 – E72) chuẩn hóa.<br>• 11 Vùng nghiệp vụ (Subject Areas).<br>• 12 Lược đồ Mermaid ERD chuẩn hóa cú pháp (`PK`, `FK`, `UK`).<br>• 10 Quyết định kiến trúc xử lý lỗi gốc (P0–P2). | Phê duyệt bắt đầu M3.01; mở các hạng mục truy vết đa kiện và đối soát chi tiết tại M3. |
| **M3: Thiết kế Cơ sở Dữ liệu Mức Logic** *(Logical Design)* | 25% | **KHỞI ĐỘNG (Starting M3.01)** | • Ánh xạ Lược đồ Quan hệ (Schema Mapping).<br>• Chuẩn hóa dữ liệu toán học (3NF / BCNF Evaluation).<br>• Định cỡ dữ liệu vật lý SQL Server & Sizing.<br>• Đặc tả Ràng buộc DDL & Stored Procedures giao dịch. | Triển khai từ M3.01 với SA01 - Product Catalog. |
| **M4: Thiết kế Vật lý & Tối ưu Hiệu năng** *(Physical & Performance)* | 15% | **CHỜ THỰC HIỆN (Pending)** | • Partitioning Scheme (theo quý/năm).<br>• Chỉ mục chuyên sâu (Filtered, Columnstore, Covering).<br>• Kế hoạch sao lưu phục hồi (HA/DR, Availability Groups). | Triển khai sau khi M3 được nghiệm thu. |
| **M5: DDL Triển khai & Kiểm thử Toàn diện** *(Implementation & Testing)* | 10% | **CHỜ THỰC HIỆN (Pending)** | • Script T-SQL tạo Schema hoàn chỉnh.<br>• Bộ dữ liệu Seed/Mock 1.000.000 dòng kiểm thử tải.<br>• Kiểm thử tự động tính toàn vẹn (Stress test / Concurrency). | Giai đoạn bàn giao cuối cùng. |

---

## 2. PHÂN LOẠI QUẢN TRỊ KIẾN TRÚC PRE-M3 (GOVERNANCE TAXONOMY)

Toàn bộ các nội dung kiến trúc được phân loại theo 4 nhóm chuẩn mực:

```mermaid
graph TD
    subgraph Architecture_Governance["Quản trị Kiến trúc RetailCore"]
        A["Quyết định đã chốt<br>(Finalized Decisions)"]
        B["Quyết định mở / Nghiên cứu tại M3<br>(Open / M3 Investigation)"]
        C["Giả định Nghiệp vụ<br>(Business Assumptions)"]
        D["Rủi ro & Biện pháp Giảm thiểu<br>(Risks & Controls)"]
    end
    A --> A1["10 Quyết định Nền tảng (CORR-01..10)<br>Chuẩn hóa Index giữ hàng theo AllocationID<br>Chuẩn hóa LRM theo EligibleBase<br>Độc lập OrderStatus khỏi RMA"]
    B --> B1["Truy vết ReturnShipmentAllocation trong RMA<br>Mô hình 3 lớp COD Split Shipment<br>Lịch sử Tái thẩm định Assessment Audit Trail"]
    C --> C1["WAC là mô hình định giá đơn giản hóa cho OnHand<br>Chính sách Đổi trả/Hoàn tiền do Policy Engine cấu hình<br>Tài khoản hạch toán Chênh lệch PPV do Kế toán quy định"]
    D --> D1["Lệch làm tròn tiền tệ VNĐ (Đã triệt tiêu qua LRM & F(q))<br>Race conditions khi giữ kho (Kiểm soát qua Filtered Index)<br>Bảo toàn 1:1 (Kiểm soát qua Phân quyền DML & SP Transaction)"]
```

### 2.1. Nhóm 1: Các Quyết định Kiến trúc Đã Chốt (Finalized Decisions)
1. **Tách biệt Đánh giá Giám định và Tồn kho:** Sử dụng `ReturnLineAssessment` (E72) làm thực thể trung gian phân định trách nhiệm lỗi (`FaultParty`) và hướng xử lý (`ResolutionType`), tách khỏi vị trí vật lý trong kho.
2. **Mô hình Số dư Tồn kho & 5 CHECK Constraints:** Khống chế toán học nghiêm ngặt trên `InventoryBalance` (E11) với:
   $$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$$
3. **Chuẩn hóa Chất lượng Tồn kho & Ghi sổ Biến động:** Thống nhất 3 trạng thái chất lượng (`SELLABLE`, `QUARANTINED`, `DAMAGED`) và nghiệp vụ xuất hủy phế phẩm (`SCRAP_OUT`). Mọi biến động số dư phải ghi sổ giao dịch kép qua `InventoryTransactionLine` (E16) với tính nguyên tử (atomic) tuyệt đối.
4. **Thuật toán Phân bổ Giảm giá LRM trên Tập Đủ điều kiện (`EligibleBase`):** Áp dụng phương pháp Số dư Lớn nhất (Hare-Niemeyer) trên giá trị ròng đủ điều kiện của từng dòng, có quy tắc phá hòa (`OrderLineID ASC`), triệt tiêu hoàn toàn sai số phân số.
5. **Công thức Hoàn tiền Nguyên số Tích luỹ $F(q)$:** Sử dụng hàm nguyên xác định số tiền hoàn lũy tiến theo số lượng, đảm bảo khi hoàn tất cả số lượng thì số tiền hoàn đúng bằng tổng giá trị ròng thực trả. Phí vận chuyển được quản lý qua trường bồi hoàn riêng (`ShippingRefundAmount`).
6. **Thực thi Quan hệ Bắt buộc 1:1 qua Stored Procedure & Phân quyền DML:** Phối hợp `UNIQUE FK` tại DDL, thu hồi quyền DML trực tiếp trên bảng cha/con khỏi tài khoản ứng dụng, bắt buộc thực thi qua Stored Procedure giao dịch (`BEGIN TRANSACTION ... COMMIT` với `SET XACT_ABORT ON`). Triển khai Table-Per-Type (TPT) cho cấu trúc `Account` $\to$ `Customer` / `Employee`.
7. **Độc lập Bốn Máy Trạng thái:** Phân định độc lập tuyệt đối giữa $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$. Loại bỏ hoàn toàn trạng thái xử lý đổi trả (`RMA_PROCESSING`) khỏi `OrderStatus`.
8. **Phân định Mốc Đóng hồ sơ Đơn hàng:** Tách biệt rõ `DeliveredAt` (thời điểm giao kiện cuối cùng), `ReturnEligibleUntil` (thông tin tổng hợp hạn đổi trả cấp đơn), và `ClosedAt` (đóng hồ sơ tài chính/lưu trữ). Quyết định đổi trả của từng món hàng được xác thực nghiêm ngặt theo hạn của từng kiện `Shipment.DeliveredAt`.
9. **Định giá Xuất trả NCC theo WAC:** Xuất trả NCC ghi giảm tồn kho theo giá vốn bình quân di động hiện hành `CurrentAverageCost`.
10. **Quản lý Giữ hàng $1 : 0..N$ qua Filtered Unique Index trên `AllocationID`:** Cho phép một dòng đơn hàng phân bổ và giữ hàng đồng thời ở nhiều kho khác nhau (Split Allocation). Mỗi phân bổ chỉ có duy nhất một dòng giữ hàng `ACTIVE`.

### 2.2. Nhóm 2: Quyết định Mở / Nghiên cứu Chuyên sâu (Đã Giải Quyết Toàn Diện tại M3-Issues-Resolution.md)
Toàn bộ 7 quyết định mở đã được giải quyết dứt điểm và đặc tả giải pháp kỹ thuật, DDL chuẩn hóa tại [M3-Issues-Resolution.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/M3-Issues-Resolution.md):
1. **Cơ chế Truy vết Dòng giao hàng qua `ReturnShipmentAllocation` (Đã chuẩn hóa):** Thiết kế thực thể phân bổ `ReturnShipmentAllocation` liên kết `ReturnRequestLineID` và `ShipmentLineID` kèm số lượng trả cụ thể (`AllocatedReturnQty`), khống chế bởi ràng buộc:
   $$\sum \text{ReturnQty}_{\text{đã duyệt}} \le \text{DeliveredQty} - \text{PreviouslyReturnedQty}$$
2. **Mô hình Lịch sử Giám định (Assessment Audit Trail - Đã chuẩn hóa):** Cấu trúc phiên bản hóa (`IsCurrent`, `AssessmentVersion`, `AssessmentStatus`: DRAFT, SUBMITTED, FINAL, SUPERSEDED) cho E72 kèm Filtered Unique Index bảo đảm duy nhất 1 bản ghi `IsCurrent = 1` cho mỗi dòng nhận hàng.
3. **Mô hình Đối soát COD 3 Lớp (Split Shipment COD - Đã chuẩn hóa):** Thiết kế 3 bảng giao dịch COD cấp kiện hàng/giao dịch:
   * **Lớp 1 (Customer Collection):** Thu tiền khách theo từng kiện (`CODCollection` gắn với `ShipmentID`).
   * **Lớp 2 (Carrier Remittance):** Khoản tiền hãng chuyển về theo từng đợt (`CODRemittance`).
   * **Lớp 3 (Reconciliation):** Đối chiếu chênh lệch bảng kê và khoản thực nhận (`CODReconciliation`).
   `Payment` cấp đơn hàng phản ánh thanh toán một phần (`AmountPaid`, `PaymentStatus = 'PARTIALLY_PAID'`).
4. **Khung Chuẩn hóa 3NF/BCNF Thực tế (Đã ban hành chính sách):** Áp dụng Controlled Normalization Policy: 100% bảng đạt 3NF; các trường hợp phi chuẩn hóa hoặc giữ 3NF không đạt BCNF (như `OrderLine`, `ShipmentLine`) phải có văn bản giải trình kỹ thuật bảo toàn phụ thuộc hàm.
5. **Thống nhất Kiểu Dữ liệu Tiền tệ VNĐ (Đã chuẩn hóa):** Sử dụng `DECIMAL(19,0)` (hoặc `BIGINT`) cho các trường tiền tệ giao dịch nguyên số đồng; và `DECIMAL(19,4)` cho đơn giá, tỷ lệ chiết khấu, giá vốn bình quân WAC. Tuyệt đối không dùng `MONEY` hay `FLOAT`.
6. **Mô hình Định giá Tồn kho WAC (Đã xác nhận):** `CurrentAverageCost` là giá vốn bình quân áp dụng cho toàn bộ `OnHand` của SKU tại kho; tính toán tự động sau mỗi `GoodsReceipt`.
7. **Triệt tiêu Sai số Làm tròn (Đã thiết lập Test Specs):** Thuật toán LRM Hare-Niemeyer trên `EligibleBase` và hàm tích lũy nguyên $F(q)$ bảo đảm triệt tiêu sai số làm tròn phân số.

### 2.3. Nhóm 3: Giả định Nghiệp vụ Đã Được Xác Nhận (Business Assumptions Confirmed)
1. **Mô hình Định giá Tồn kho Đơn giản hóa:** `CurrentAverageCost` được xác nhận là giá vốn bình quân áp dụng cho toàn bộ lượng tồn vật lý `OnHand` của SKU tại kho. Các khoản trích lập dự phòng giảm giá cho hàng `DAMAGED` sẽ được xử lý tại tầng hạch toán kế toán khi xuất hủy `SCRAP_OUT`.
2. **Chính sách Hoàn phí Vận chuyển & Tỷ lệ Bồi hoàn:** Tách bạch trường `ApprovedRefundAmount` (tiền hàng) và `ShippingRefundAmount` (tiền ship). Tỷ lệ hoàn và việc hoàn phí giao hàng do **Policy Engine** cấu hình và truyền vào, không gắn cứng ràng buộc ở tầng CSDL.
3. **Tài khoản Hạch toán Chênh lệch Giá vốn:** Khoản chênh lệch $CostVariance = SupplierRefundAmount - InventoryValueDecrease$ được lưu vết kỹ thuật kèm cột `CostVarianceAccountCode` tham chiếu tài khoản kế toán doanh nghiệp (TK 632 / TK 156 chênh lệch).
4. **Lưu trữ Bền vững Webhook:** Phản hồi HTTP 200 chỉ được gửi cho cổng thanh toán sau khi payload webhook đã được ghi nhận bền vững (durably persisted) vào cơ sở dữ liệu `PaymentWebhookLog` qua Stored Procedure giao dịch `usp_ReceivePaymentWebhook`. Khóa duy nhất kiểm soát Idempotency chống tấn công Replay.

### 2.4. Nhóm 4: Rủi ro Kỹ thuật & Biện pháp Giảm thiểu (Risks & Controls)
1. **Rủi ro Sai số Làm tròn:** Triệt tiêu hoàn toàn nhờ việc kết hợp LRM, công thức tích luỹ nguyên $F(q)$ và kiểu dữ liệu `DECIMAL(19,0)`.
2. **Rủi ro Tranh chấp Ghi sổ Tồn kho & Giữ hàng Đa kho:** Kiểm soát qua Filtered Unique Index trên `StockReservationLine(AllocationID)` kết hợp Transaction Isolation Level `SNAPSHOT` hoặc `READ COMMITTED SNAPSHOT (RCSI)`.
3. **Rủi ro Hiệu năng JSON Snapshot:** Không quét trực tiếp `OPENJSON` hay index theo vị trí mảng `$[0]`; thiết kế Persisted Computed Columns theo mã thuộc tính xác định (`attributeCode`) hoặc trích xuất bảng phân tích chuẩn hóa cho báo cáo BI.
4. **Rủi ro Phá vỡ Ràng buộc 1:1:** Kiểm soát bằng cách thu hồi quyền DML trực tiếp và cưỡng chế thực thi qua Stored Procedure giao dịch có `SET XACT_ABORT ON`.

---

## 3. KIẾN TRÚC CÁC LUỒNG NGHIỆP VỤ HUYẾT MẠCH & BẢN ĐỒ THỰC THỂ HẠT NHÂN (END-TO-END SYSTEM FLOWS & ENTITY MAPPING)

Toàn bộ hệ thống RetailCore được vận hành thông qua **8 Luồng nghiệp vụ huyết mạch** kết nối 72 thực thể quan niệm trên 11 Subject Areas.

```mermaid
flowchart TD
    subgraph Core_Supply_Chain["Chuỗi Cung Ứng & Bán Hàng RetailCore"]
        F1["1. Chuỗi Mua Hàng & Quản Lý Nhập Kho (Inbound)"]
        F2["2. Mua Sắm, Giỏ Hàng & Khởi Tạo Đơn Hàng (Shopping to Order)"]
        F3["3. Khuyến Mãi, Voucher & Phân Bổ LRM (Promotions)"]
        F4["4. Phân Bổ Đa Kho & Khóa Giữ Tồn Kho (Allocation & Reservation)"]
        F5["5. Đóng Gói Bưu Kiện, Split Shipment & Giao Vận (Fulfillment)"]
        F6["6. Thanh Toán Trực Tuyến & Đối Soát COD (Payment & Settlement)"]
        F7["7. Hậu Cần Ngược, Giám Định KCS & Hoàn Tiền (Reverse Logistics)"]
        F8["8. Kiểm Toán Hệ Thống & Báo Cáo Quản Trị (Audit & Reporting)"]
    end
    F1 -->|Tạo tồn khả dụng| F2
    F2 -->|Kích hoạt tính giá| F3
    F3 -->|Xác nhận giữ kho| F4
    F4 -->|Điều phối xuất kho| F5
    F5 -->|Xác thực công nợ| F6
    F6 -.->|Phát sinh khiếu nại| F7
    F1 & F2 & F4 & F5 & F6 & F7 ==>|Ghi nhật ký kiểm tra| F8
```

---

### 3.1. Luồng 1: Chuỗi Mua Hàng & Quản Lý Nhập Kho (Inbound Procurement & Inventory Storing Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Đảm bảo hàng hóa từ các Nhà cung cấp được mua vào theo đúng hợp đồng thương mại, được kiểm đếm ngoại quan tại cửa kho (KCS), cập nhật tức thời vào sổ cái tồn kho vật lý và tính toán giá vốn bình quân di động (Moving Weighted Average Cost - WAC).

```mermaid
sequenceDiagram
    autonumber
    actor ProcUser as Nhân viên Mua hàng
    participant Sup as Supplier (E21)
    participant PO as PurchaseOrder (E24/E25)
    participant GR as GoodsReceipt (E26/E27)
    participant Inv as InventoryBalance (E11)
    participant Tx as InventoryTransaction (E15/E16)
    
    ProcUser->>PO: Tạo đơn mua hàng PO (Chỉ định Kho, SKU, Đơn giá mua)
    PO->>Sup: Gửi đơn đặt hàng chính thức
    Sup-->>GR: Giao hàng tới cửa kho (Biên bản giao hàng)
    GR->>GR: Thủ kho kiểm đếm thực tế (ReceivedQty vs AcceptedQty)
    GR->>Tx: Post biên bản -> Ghi sổ cái Transaction (Loại PURCHASE_RECEIPT)
    Tx->>Inv: Tăng OnHandQty, Cập nhật CurrentAverageCost (WAC)
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E21** | `Supplier` | Strong Entity | Đại diện cho đối tác cung ứng hàng hóa, theo dõi công nợ và xếp hạng đối tác. | `SupplierID`, `SupplierCode`, `SupplierName`, `TaxCode` |
| **E22** | `SupplierContact` | Weak Entity | Đầu mối liên hệ trực tiếp phụ trách các đơn hàng mua. | `ContactID`, `SupplierID`, `ContactName`, `Phone`, `Email` |
| **E23** | `PurchaseAgreement` | Document Entity | Thỏa thuận khung hoặc hợp đồng nguyên tắc quy định giá mua cố định và kỳ hạn. | `AgreementID`, `SupplierID`, `StartDate`, `EndDate`, `Terms` |
| **E24** | `PurchaseOrder` | Strong / Document | Hồ sơ đơn mua hàng chính thức gửi NCC (PO Header). Quản lý trạng thái mua. | `PurchaseOrderID`, `PONumber`, `SupplierID`, `WarehouseID`, `POStatus` |
| **E25** | `PurchaseOrderLine` | Weak / Detail | Chi tiết từng SKU cần mua, số lượng đặt hàng và đơn giá cam kết mua. | `POLineID`, `PurchaseOrderID`, `VariantID`, `OrderedQty`, `UnitPrice` |
| **E26** | `GoodsReceipt` | Strong / Document | Biên bản tiếp nhận hàng hóa tại bến nhập kho (GRN Header). | `ReceiptID`, `ReceiptNumber`, `PurchaseOrderID`, `WarehouseID`, `PostedAt` |
| **E27** | `GoodsReceiptLine` | Weak / Detail | Chi tiết số lượng thực nhận, số lượng đạt chuẩn (Accepted) và số lượng từ chối. | `ReceiptLineID`, `ReceiptID`, `POLineID`, `ReceivedQty`, `AcceptedQty`, `UnitCost` |
| **E28** | `SupplierReturn` | Strong / Document | Phiếu xuất trả hàng cho nhà cung cấp khi hàng lỗi phát hiện sau nhập kho. | `ReturnID`, `SupplierID`, `WarehouseID`, `CostVariance`, `ReturnStatus` |
| **E10** | `Warehouse` | Strong Entity | Kho hàng vật lý tiếp nhận lưu trữ hàng hóa. | `WarehouseID`, `WarehouseCode`, `WarehouseName` |
| **E11** | `InventoryBalance` | Associative Entity | Bảng số dư tức thời của SKU tại kho. Tăng `OnHandQty` và cập nhật WAC. | `WarehouseID`, `VariantID`, `OnHandQty`, `CurrentAverageCost` |
| **E15** | `InventoryTransaction` | Document Entity | Chứng từ sổ cái biến động kho loại `PURCHASE_RECEIPT` hoặc `SUPPLIER_RETURN_OUT`. | `TransactionID`, `TransactionType`, `ReferenceID`, `PostedAt` |
| **E16** | `InventoryTransactionLine` | Weak Entity | Dòng biến động số dư: `OnHandDelta > 0`, ghi nhận giá mua thực tế vào `UnitCost`. | `TransactionLineID`, `TransactionID`, `VariantID`, `OnHandDelta`, `UnitCost` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Tính toán WAC tức thời:** Khi biên bản `GoodsReceipt` được chốt (`PostedAt` NOT NULL), giá vốn bình quân di động mới của tồn kho thương phẩm được tính theo công thức chuẩn hóa:
  $$\text{WAC}_{\text{mới}} = \frac{(\text{OnHandQty}_{\text{cũ}} \times \text{WAC}_{\text{cũ}}) + (\text{AcceptedQty} \times \text{UnitCost}_{\text{PO}})}{\text{OnHandQty}_{\text{cũ}} + \text{AcceptedQty}}$$
* **Xuất trả NCC (SupplierReturn):** Giảm tồn kho theo `CurrentAverageCost` hiện hành, chênh lệch so với giá PO được tính vào `CostVariance` để phục vụ phòng Kế toán hạch toán.

---

### 3.2. Luồng 2: Trải Nghiệm Mua Sắm, Chuyển Đổi Giỏ Hàng & Khởi Tạo Đơn Hàng (Shopping, Cart Conversion & Order Inception Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Chuyển hóa nhu cầu mua sắm từ Giỏ hàng trực tuyến thành một Hợp đồng mua bán chính thức (`SalesOrder`). Toàn bộ thông tin thời điểm mua được đóng băng bất biến qua **Cơ chế Snapshot 3 Điểm** nhằm bảo vệ hệ thống trước sự biến động dữ liệu sau này.

```mermaid
sequenceDiagram
    autonumber
    actor Cust as Khách Hàng (Customer)
    participant Cart as ShoppingCart / CartItem (E38/E39)
    participant Order as SalesOrder / OrderLine (E40/E41)
    participant Addr as OrderAddress (E42)
    participant Hist as OrderStatusHistory (E43)
    
    Cust->>Cart: Thêm biến thể SKU vào Giỏ hàng (CartItem)
    Cust->>Order: Bấm "Đặt Hàng" (Checkout)
    Note over Cart,Order: Thực thi Giao dịch Nguyên tử (All-or-Nothing)
    Cart->>Cart: CartStatus chuyển 'CONVERTED'
    Order->>Order: Tạo SalesOrder (PENDING_PAYMENT)
    Order->>Order: Tạo OrderLine (Đóng băng UnitPrice & VariantAttributesSnapshot JSON)
    Order->>Addr: Tạo OrderAddress (Sao chép hoàn toàn từ CustomerAddress)
    Order->>Hist: Ghi nhận OrderStatusHistory (Khởi tạo đơn hàng)
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E29** | `Account` | Strong Entity | Tài khoản định danh đăng nhập dùng chung (TPT Base). | `AccountID`, `Username`, `Email`, `AccountType` |
| **E30** | `Customer` | Subtype Entity | Hồ sơ khách hàng thương mại, lưu cấp độ trung thành (Loyalty Tier). | `AccountID (PK_FK)`, `CustomerCode`, `LoyaltyTierID` |
| **E32** | `CustomerAddress` | Weak Entity | Sổ địa chỉ cá nhân của khách hàng (có thể bị sửa hoặc xóa bất kỳ lúc nào). | `AddressID`, `CustomerID`, `RecipientName`, `StreetAddress` |
| **E01** | `Product` | Strong Entity | Thông tin chung của sản phẩm thương mại. | `ProductID`, `ProductName`, `Brand` |
| **E04** | `ProductVariant` | Core SKU Entity | Biến thể vật lý cụ thể (màu, size) mang mã SKU duy nhất để bán. | `VariantID`, `ProductID`, `SKU`, `SellingPrice` |
| **E38** | `ShoppingCart` | Strong Entity | Giỏ hàng trực tuyến lưu trạng thái mua sắm của khách. | `CartID`, `CustomerID`, `CartStatus` (ACTIVE / CONVERTED) |
| **E39** | `CartItem` | Weak Entity | Dòng hàng trong giỏ, lưu số lượng tạm chọn của từng SKU. | `CartItemID`, `CartID`, `VariantID`, `Quantity` |
| **E40** | `SalesOrder` | Strong / Document | Hợp đồng mua bán chính thức giữa Khách hàng và RetailCore (Header). | `OrderID`, `OrderNumber`, `CustomerID`, `OrderStatus`, `TotalAmount` |
| **E41** | `OrderLine` | Weak / Detail | Chi tiết từng mặt hàng được chốt mua, lưu snapshot giá và thuộc tính. | `OrderLineID`, `OrderID`, `VariantID`, `UnitPrice`, `VariantAttributesSnapshot` |
| **E42** | `OrderAddress` | Snapshot Entity | Bản sao bất biến địa chỉ giao hàng, độc lập hoàn toàn với `CustomerAddress`. | `OrderAddressID`, `OrderID`, `RecipientName`, `RecipientPhone`, `StreetAddress` |
| **E43** | `OrderStatusHistory` | Audit Entity | Nhật ký vết lưu chuyển trạng thái của đơn hàng phục vụ kiểm soát SLA. | `StatusHistoryID`, `OrderID`, `PreviousStatus`, `NewStatus`, `ChangedAt` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Cơ chế Snapshot 3 Điểm:**
  1. `OrderAddress` không có Foreign Key trỏ về `CustomerAddress`. Khi khách hàng sửa sổ địa chỉ cá nhân, địa chỉ đơn hàng đã đặt hoàn toàn không bị ảnh hưởng.
  2. `OrderLine.UnitPrice` lưu cứng giá bán tại thời điểm bấm đặt hàng. Mọi thay đổi giá niêm yết trong danh mục sau đó không làm thay đổi giá trị đơn hàng.
  3. `OrderLine.VariantAttributesSnapshot` lưu trữ mảng JSON chuẩn hoá (`attributeId`, `attributeCode`, `valueCode`, `displayValue`), đảm bảo tính toàn vẹn dữ liệu lịch sử lâu dài.

---

### 3.3. Luồng 3: Áp Mã Khuyến Mãi, Xếp Chồng Voucher & Phân Bổ Chiết Khấu LRM (Promotion Stacking & Discount Allocation Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Xác thực tính hợp lệ của mã giảm giá (`Voucher`) và chương trình khuyến mãi (`Promotion`). Phân bổ chiết khấu tổng đơn về từng dòng hàng đủ điều kiện bằng **Thuật toán Số dư Lớn nhất (Largest Remainder Method - LRM)** để đảm bảo số tiền giảm giá và thanh toán ròng là các số nguyên đồng VNĐ chính xác tuyệt đối.

```mermaid
sequenceDiagram
    autonumber
    participant Order as SalesOrder / OrderLine (E40/E41)
    participant Promo as Promotion / Voucher (E49/E52)
    participant Rule as PromotionCondition (E50)
    participant Usage as PromotionUsage (E53)
    
    Order->>Promo: Gửi mã Voucher và Danh sách OrderLine
    Promo->>Rule: Kiểm tra điều kiện (MinOrderAmount, TargetCustomer, Hạn dùng)
    Rule-->>Promo: Hợp lệ -> Tính tổng tiền giảm giá (DiscountAmount)
    Promo->>Order: Xác định tập EligibleBase = GrossAmount - ProductDiscount
    Promo->>Order: Áp dụng LRM phân bổ DiscountAmount về các dòng đủ điều kiện
    Note over Promo,Order: Không làm tròn số lẻ - Phân bổ chính xác từng đồng VNĐ
    Order->>Usage: Ghi sổ cái tiêu dùng PromotionUsage (Bất biến)
    Order->>Order: Tính LineNetAmount = LineGrossAmount - ProductDiscount - AllocatedOrderDiscount
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E49** | `Promotion` | Strong / Document | Định nghĩa chương trình ưu đãi tổng thể (Header, có quản lý phiên bản). | `PromotionID`, `PromotionCode`, `DiscountType`, `DiscountValue`, `MaxDiscountAmount` |
| **E50** | `PromotionCondition` | Weak Entity | Các quy tắc điều kiện đơn hàng/khách hàng phải thỏa mãn để được hưởng ưu đãi. | `ConditionID`, `PromotionID`, `ConditionType`, `ComparisonOperator`, `ConditionValue` |
| **E51** | `PromotionProduct` | Associative Entity | Danh sách các SKU thuộc diện được áp dụng ưu đãi cấp sản phẩm. | `PromotionID`, `VariantID` |
| **E52** | `Voucher` | Document Entity | Mã phiếu giảm giá cụ thể mà khách hàng nhập khi checkout (Public hoặc Cá nhân). | `VoucherID`, `PromotionID`, `VoucherCode`, `MaxUsage`, `UsedCount`, `EndAt` |
| **E53** | `PromotionUsage` | Associative / Ledger | Sổ cái ghi nhận lịch sử áp dụng khuyến mại bất biến vào đơn hàng. | `PromotionUsageID`, `PromotionID`, `OrderID`, `OrderLineID`, `VoucherID`, `DiscountAmount` |
| **E54** | `PromotionTargetCustomer` | Associative Entity | Phân quyền chương trình ưu đãi dành riêng cho nhóm khách hàng thân thiết/VIP. | `PromotionID`, `CustomerID` |
| **E41** | `OrderLine` | Detail Entity | Tiếp nhận giá trị chiết khấu phân bổ vào trường `AllocatedOrderDiscount`. | `AllocatedOrderDiscount`, `LineNetAmount`, `LineGrossAmount` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Xác định Tập Đủ điều kiện (`EligibleBase`):**
  $$\text{EligibleBase}_i = \text{LineGrossAmount}_i - \text{LineProductDiscount}_i$$
  Chỉ các dòng thỏa mãn điều kiện áp dụng voucher mới tham gia mẫu số phân bổ:
  $$D_i = D \times \frac{\text{EligibleBase}_i}{\sum_{j \in \text{Eligible}} \text{EligibleBase}_j}$$
* **Triệt tiêu Sai số Làm tròn (LRM):** Thuật toán Hare-Niemeyer phân chia phần nguyên đồng trước, phần dư phân số được ưu tiên cộng 1 đồng cho các dòng có phần dư lớn nhất. Trường hợp hòa phần dư (tie-breaker), ưu tiên theo `OrderLineID ASC`. Tổng chiết khấu phân bổ luôn bằng đúng 100% chiết khấu đơn:
  $$\sum_{i=1}^N \text{AllocatedOrderDiscount}_i = \text{DiscountAmount}_{\text{Order}}$$

---

### 3.4. Luồng 4: Phân Bổ Đa Kho & Khóa Giữ Tồn Kho Đa Kiện (Multi-Warehouse Allocation & Stock Reservation Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Đối với đơn hàng có nhiều mặt hàng nằm rải rác ở nhiều kho vật lý khác nhau, hệ thống tự động định tuyến xuất hàng (`OrderWarehouseAllocation`) và khóa giữ tạm thời tồn kho khả dụng (`StockReservationLine`) gắn liền với từng `AllocationID` trong thời gian 15 phút, ngăn chặn triệt để tình trạng bán vượt tồn (Overselling) trong đơn hàng đa kho (Split Order).

```mermaid
sequenceDiagram
    autonumber
    participant Order as SalesOrder / OrderLine (E40/E41)
    participant Alloc as OrderWarehouseAllocation (E12)
    participant Res as StockReservation / Line (E13/E14)
    participant Inv as InventoryBalance (E11)
    
    Order->>Alloc: Chạy thuật toán định tuyến kho (Tối ưu vị trí & Tồn kho)
    Alloc->>Alloc: Tạo bản ghi Allocation (Phân bổ SKU về từng WarehouseID)
    Alloc->>Res: Tạo StockReservation (Header) & StockReservationLine (theo từng AllocationID)
    Res->>Res: Khống chế 1 dòng duy nhất ACTIVE cho mỗi AllocationID
    Res->>Inv: Tăng ReservedQty, Giảm AvailableQty (OnHandQty giữ nguyên)
    Note over Inv: AvailableQty = OnHandQty - ReservedQty - QuarantinedQty - DamagedQty
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E12** | `OrderWarehouseAllocation` | Associative Entity | Bản ghi định tuyến: Quyết định lấy bao nhiêu sản phẩm của dòng đơn từ kho nào. | `AllocationID`, `OrderLineID`, `WarehouseID`, `VariantID`, `AllocatedQty`, `AllocationStatus` |
| **E13** | `StockReservation` | Document Entity | Phiếu khóa giữ hàng tạm thời gắn liền với đơn hàng (Header, quản lý thời gian hết hạn). | `ReservationID`, `OrderID`, `ReservationStatus`, `CreatedAt`, `ExpiresAt`, `ReleasedAt` |
| **E14** | `StockReservationLine` | Weak Entity | Chi tiết số lượng SKU bị khóa gắn với từng `AllocationID` cụ thể tại một kho. | `ReservationLineID`, `ReservationID`, `AllocationID`, `WarehouseID`, `ReservedQty`, `ReservationLineStatus` |
| **E11** | `InventoryBalance` | Associative Entity | Bảng số dư kho: Tiếp nhận biến động tăng `ReservedQty`. | `WarehouseID`, `VariantID`, `ReservedQty`, `OnHandQty` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Hỗ trợ Giữ hàng Đa kho Đồng thời (Split Allocation):** Một dòng đơn hàng `OrderLine` (ví dụ: mua 10 áo) có thể phân bổ thành 2 `Allocation`: Allocation A (6 áo tại Kho 1) và Allocation B (4 áo tại Kho 2). Do đó, ràng buộc duy nhất dòng giữ hàng `ACTIVE` phải được đặt trên **`AllocationID`**, tuyệt đối không đặt trên `OrderLineID`:
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAllocation
  ON StockReservationLine(AllocationID)
  WHERE ReservationLineStatus = 'ACTIVE';
  ```
* **Bảo toàn Tồn kho Vật lý:** Thao tác khóa giữ kho **tuyệt đối không làm thay đổi `OnHandQty`**. Chỉ có `ReservedQty` tăng lên, dẫn đến `AvailableQty` giảm tức thời.

---

### 3.5. Luồng 5: Đóng Gói Bưu Kiện, Phân Tách Kiện Hàng & Giao Vận (Fulfillment, Split Shipment & Carrier Dispatch Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Thực thi đóng gói bưu phẩm tại từng kho hàng theo đúng phân bổ. Nếu đơn hàng lấy từ nhiều kho, hệ thống tự động tách thành nhiều kiện độc lập (**Split Shipment**). Khi đơn vị vận chuyển tiếp nhận kiện (`IN_TRANSIT`), hệ thống mới chính thức xuất sổ trừ tồn kho vật lý và cập nhật tracking.

```mermaid
sequenceDiagram
    autonumber
    participant Alloc as OrderWarehouseAllocation (E12)
    participant Ship as Shipment / ShipmentLine (E45/E46)
    participant Carrier as Carrier / Tracking (E44/E48)
    participant Tx as InventoryTransaction (E15/E16)
    participant Inv as InventoryBalance (E11)
    
    Alloc->>Ship: Tạo Shipment theo từng Warehouse (READY_FOR_HANDOVER)
    Ship->>Ship: Tạo ShipmentLine gắn trực tiếp với AllocationID
    Ship->>Carrier: Bàn giao bưu tá (Ghi nhận TrackingNumber)
    Carrier->>Ship: Trạng thái chuyển 'IN_TRANSIT' (Xuất bãi)
    Ship->>Tx: Ghi sổ cái Transaction (Loại SHIPMENT_OUT)
    Tx->>Inv: Trừ OnHandQty (-ShipmentQty) & Giải phóng ReservedQty (-ShipmentQty)
    Carrier->>Ship: Giao hàng thành công -> Cập nhật DeliveredAt của kiện
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E44** | `Carrier` | Strong Entity | Đối tác giao vận (GHTK, GHN, Viettel Post, Shopee Xpress...). | `CarrierID`, `CarrierCode`, `CarrierName`, `ContactPhone` |
| **E45** | `Shipment` | Strong / Document | Kiện hàng giao vận thực tế được đóng gói từ một kho xuất phát (Header). | `ShipmentID`, `ShipmentNumber`, `OrderID`, `WarehouseID`, `CarrierID`, `TrackingNumber`, `ShipmentStatus`, `DeliveredAt` |
| **E46** | `ShipmentLine` | Associative / Detail | Chi tiết số lượng mặt hàng nằm trong kiện hàng, gắn liền với phân bổ kho. | `ShipmentLineID`, `ShipmentID`, `OrderLineID`, `AllocationID`, `ShipmentQty` |
| **E47** | `DeliveryAttempt` | Weak Entity | Ghi nhận các lần bưu tá phát hàng thực tế (lần 1, 2, 3 kèm nguyên nhân nếu thất bại). | `AttemptID`, `ShipmentID`, `AttemptNumber`, `AttemptStatus`, `FailureReason` |
| **E48** | `ShipmentStatusHistory` | Audit Entity | Nhật ký vết chuyển đổi trạng thái của kiện hàng phục vụ đối soát SLA giao vận. | `ShipmentStatusHistoryID`, `ShipmentID`, `PreviousStatus`, `NewStatus`, `ChangedAt` |
| **E15** | `InventoryTransaction` | Document Entity | Chứng từ xuất kho loại `SHIPMENT_OUT`. | `TransactionID`, `TransactionType`, `ReferenceID`, `PostedAt` |
| **E16** | `InventoryTransactionLine` | Weak Entity | Dòng biến động kép: `OnHandDelta = -Qty`, `ReservedDelta = -Qty`. | `TransactionLineID`, `TransactionID`, `VariantID`, `OnHandDelta`, `ReservedDelta` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Ghi sổ Kép Xuất Kho (Atomic Dual-Delta):** Khi kiện hàng chuyển sang `IN_TRANSIT`, hệ thống ghi sổ:
  $$\text{OnHandDelta} = -\text{ShipmentQty}, \quad \text{ReservedDelta} = -\text{ShipmentQty}$$
  Thao tác này đồng thời giảm tồn kho vật lý và giải phóng khóa giữ hàng trong cùng một Database Transaction.
* **Độc lập Mốc Giao theo Kiện:** Trong Split Shipment, mỗi `Shipment` có thời điểm giao thành công `DeliveredAt` hoàn toàn độc lập, làm căn cứ xác thực hạn đổi trả riêng cho từng kiện.

---

### 3.6. Luồng 6: Thanh Toán Trực Tuyến, Xử Lý Webhook & Đối Soát COD 3 Lớp (Payment Settlement, Webhook & COD Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Quản lý nghĩa vụ tài chính của đơn hàng thông qua mô hình trạng thái phân nhánh. Hỗ trợ thanh toán trực tuyến với bộ đệm Webhook lưu trữ bền vững chống Replay attack, đồng thời vận hành đối soát COD 3 lớp bảo vệ dòng tiền thu hộ trong đơn hàng chia kiện (Split Shipment).

```mermaid
sequenceDiagram
    autonumber
    actor Cust as Khách Hàng
    participant Pay as Payment (E56)
    participant Tx as PaymentTransaction (E57)
    participant Log as PaymentWebhookLog (E71)
    participant Carrier as Bưu Cục COD (Carrier)
    
    alt Thanh toán Điện tử (Online Gateway)
        Pay->>Tx: Khởi tạo lần thử thanh toán (INITIATED)
        Tx->>Log: Cổng thanh toán gửi Webhook kết quả
        Log->>Log: Lưu bền vững Payload -> Trả HTTP 200 -> Kiểm tra Idempotency
        Log->>Tx: Xử lý nghiệp vụ -> Cập nhật Tx = SUCCEEDED
        Tx->>Pay: Cập nhật PaymentStatus = PAID
    else Thanh toán Tiền mặt khi Nhận hàng (COD Split Shipment)
        Cust->>Carrier: Trả tiền mặt cho Shipper lúc nhận Kiện 1
        Carrier-->>Pay: Lớp 1 (Collection): Ghi nhận CODCollection theo ShipmentID
        Pay->>Pay: Cập nhật AmountPaid tăng thêm, PaymentStatus = PARTIALLY_PAID
        Note over Pay: Chỉ chuyển PAID khi toàn bộ các kiện được thu tiền đủ
        Carrier->>Pay: Lớp 2 (Remittance): Hãng chuyển khoản tiền thu hộ theo đợt
        Pay->>Pay: Lớp 3 (Reconciliation): Khớp bảng kê đối soát với thực nhận
    end
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E55** | `PaymentMethod` | Strong Entity | Danh mục phương thức thanh toán: COD, VNPAY, MOMO, ZALOPAY, CREDIT_CARD. | `PaymentMethodID`, `MethodCode`, `MethodName`, `MethodType` |
| **E56** | `Payment` | Strong / Document | Hồ sơ nghĩa vụ tài chính tổng thể của đơn hàng ($1:1$ với `SalesOrder`). | `PaymentID`, `OrderID`, `PaymentMethodID`, `PaymentStatus`, `AmountDue`, `AmountPaid` |
| **E57** | `PaymentTransaction` | Weak / Event | Nhật ký từng lần thử thu tiền qua cổng thanh toán (hỗ trợ thử lại nhiều lần). | `PaymentTransactionID`, `PaymentID`, `GatewayTransactionID`, `TransactionStatus`, `Amount` |
| **E71** | `PaymentWebhookLog` | Support / Audit | Bộ đệm raw payload webhook từ cổng thanh toán, ngăn chặn tấn công Replay. | `WebhookLogID`, `ProviderID`, `MerchantAccountID`, `ProviderEventID`, `Environment`, `ProcessingStatus`, `Payload` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Lưu trữ Bền vững Webhook trước khi Phản hồi HTTP 200:** Hệ thống chỉ trả mã HTTP 200 sau khi toàn bộ raw payload đã được ghi nhận bền vững vào bảng `PaymentWebhookLog`. Xử lý nghiệp vụ diễn ra tuần tự hoặc bất đồng bộ với cơ chế retry an toàn.
* **Khắc phục Sai lệch Trạng thái COD trong Split Shipment:** Khi đơn hàng có 2 kiện (Kiện A: 400.000đ; Kiện B: 600.000đ):
  * Kiện A giao thành công và thu tiền $\implies$ Ghi nhận bản ghi `CODCollection` gắn với `Shipment A`. Cập nhật `Payment.AmountPaid = 400.000đ`, trạng thái chuyển sang **`PARTIALLY_PAID`**. Tuyệt đối không đánh dấu `PAID` cho toàn đơn hàng.
  * Chỉ khi Kiện B giao và thu đủ 600.000đ còn lại $\implies$ `Payment.AmountPaid = 1.000.000đ`, `Payment.PaymentStatus` mới chính thức chuyển sang **`PAID`**.

---

### 3.7. Luồng 7: Hậu Cần Ngược, Giám Định Lỗi & Hoàn Tiền Bảo Toàn Tài Chính (Reverse Logistics, Assessment & Refund Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Xử lý khiếu nại đổi trả hàng (RMA) sau bán hàng. Truy vết số lượng trả về từng kiện giao hàng cụ thể thông qua `ReturnShipmentAllocation`, thực hiện giám định độc lập lỗi KCS thông qua `ReturnLineAssessment` (E72) và hoàn trả tiền chính xác theo công thức lũy tiến số nguyên $F(q)$.

```mermaid
sequenceDiagram
    autonumber
    actor Cust as Khách Hàng
    participant RMA as ReturnRequest / Line (E60/E61)
    participant Alloc as ReturnShipmentAllocation (M3)
    participant Rec as ReturnReceipt / Line (E62/E63)
    participant Ass as ReturnLineAssessment (E72)
    participant Inv as Inventory (E11/E16)
    participant Ref as Refund / RefundTx (E58/E59)
    
    Cust->>RMA: Mở yêu cầu RMA
    RMA->>Alloc: Truy vết số lượng trả về từng ShipmentLine (Kiểm tra DeliveredAt của kiện)
    Alloc->>Rec: Tiếp nhận hàng tại bến nhập kho
    Rec->>Ass: Giám định độc lập (FaultParty, ResolutionType, ApprovedRefundAmount)
    par Xử lý Kho Vật lý (Tùy theo hiện trạng chất lượng)
        Ass->>Inv: Hàng đạt chuẩn -> Nhập lại kho SELLABLE (CUSTOMER_RETURN_IN)
        Ass->>Inv: Hàng lỗi tiêu huỷ -> Xuất hủy phế phẩm (SCRAP_OUT)
    and Xử lý Tài chính (Độc lập Kho)
        Ass->>Ref: Tạo phiếu hoàn tiền Refund
        Ref->>Ref: Thực thi RefundTransaction (Liên kết OriginalPaymentTransactionID)
    end
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E60** | `ReturnRequest` | Strong / Document | Yêu cầu đổi trả hàng chính thức do khách hoặc CSKH lập (RMA Header). | `ReturnRequestID`, `ReturnNumber`, `OrderID`, `CustomerID`, `WarehouseID`, `ReturnStatus` |
| **E61** | `ReturnRequestLine` | Weak / Detail | Chi tiết từng SKU và số lượng yêu cầu trả, gắn với dòng đơn hàng gốc. | `ReturnRequestLineID`, `ReturnRequestID`, `OrderLineID`, `RequestedQty`, `ApprovedQty` |
| *M3* | `ReturnShipmentAllocation` | Associative Entity | Phân bổ số lượng hàng trả về từng dòng giao hàng cụ thể để kiểm tra hạn đổi trả. | `AllocationID`, `ReturnRequestLineID`, `ShipmentLineID`, `AllocatedReturnQty` |
| **E62** | `ReturnReceipt` | Strong / Document | Biên bản tiếp nhận bưu kiện hoàn trả thực tế tại kho được chỉ định. | `ReturnReceiptID`, `ReceiptNumber`, `ReturnRequestID`, `WarehouseID`, `PostedAt` |
| **E63** | `ReturnReceiptLine` | Weak / Detail | Chi tiết số lượng thực tế mở gói tiếp nhận tại bến nhập kho. | `ReturnReceiptLineID`, `ReturnReceiptID`, `ReturnRequestLineID`, `ReceivedQty` |
| **E64** | `ReturnStatusHistory` | Audit Entity | Nhật ký vết chuyển đổi trạng thái của yêu cầu đổi trả phục vụ đối soát SLA. | `ReturnStatusHistoryID`, `ReturnRequestID`, `PreviousStatus`, `NewStatus`, `ChangedAt` |
| **E72** | `ReturnLineAssessment` | Assessment Entity | Bản ghi thẩm định trách nhiệm lỗi, điều kiện hoàn tiền và lưu vết Audit Trail. | `AssessmentID`, `ReturnReceiptLineID`, `AssessedQty`, `FaultParty`, `ResolutionType`, `ApprovedRefundAmount`, `IsCurrent` |
| **E58** | `Refund` | Strong / Document | Hồ sơ phê duyệt lệnh hoàn tiền gắn với hồ sơ thanh toán thực thu ban đầu. | `RefundID`, `PaymentID`, `RefundReference`, `RefundAmount`, `RefundStatus` |
| **E59** | `RefundTransaction` | Weak / Event | Giao dịch thực thi chuyển tiền hoàn qua cổng, liên kết giao dịch gốc. | `RefundTransactionID`, `RefundID`, `OriginalPaymentTransactionID`, `Amount`, `TransactionStatus` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Kiểm soát Hạn và Số lượng Trả qua `ReturnShipmentAllocation`:**
  Mỗi phân bổ hàng trả về dòng kiện hàng phải thỏa mãn điều kiện thời gian và số lượng:
  $$\text{RequestDate} \le \text{Shipment.DeliveredAt} + \text{PolicyDays}$$
  $$\sum \text{AllocatedReturnQty} \le \text{ShipmentLine.ShipmentQty} - \text{PreviouslyReturnedQty}$$
* **Công thức Hoàn tiền Tích luỹ Nguyên số $F(q)$:** Triệt tiêu hoàn toàn sai số phân số khi khách trả hàng lẻ từng đợt, đảm bảo số tiền hoàn tích lũy bằng đúng số tiền ròng thực trả.
* **Kiểm soát Ngân quỹ Hoàn tiền:** Mọi `RefundTransaction` bắt buộc trỏ về `OriginalPaymentTransactionID` thành công và được kiểm soát không hoàn vượt quá số tiền thực thu.

---

### 3.8. Luồng 8: Quản Trị Hệ Thống, Kiểm Toán Vết & Báo Cáo Doanh Nghiệp (System Audit & Enterprise Reporting Flow)

#### 1. Mục tiêu & Hành trình nghiệp vụ:
Ghi vết bất biến toàn bộ các thao tác nghiệp vụ nhạy cảm (thay đổi giá, can thiệp tồn kho, điều chỉnh hoàn tiền) để đáp ứng chuẩn mực kiểm toán SOX/ISO. Cung cấp cơ chế đóng băng dữ liệu báo cáo tài chính/quản trị (`ReportSnapshot`) theo kỳ kế toán cố định, giảm tải truy vấn trực tiếp lên cơ sở dữ liệu giao dịch OLTP.

```mermaid
sequenceDiagram
    autonumber
    actor AdminUser as Quản trị viên / Kế toán
    participant App as Ứng Dụng RetailCore
    participant Audit as AuditLog / Detail (E65/E66)
    participant Rep as ReportDefinition / Exec (E67/E68)
    participant Snap as ReportSnapshot (E69)
    
    AdminUser->>App: Thực hiện thao tác nghiệp vụ nhạy cảm (Sửa giá / Phê duyệt hoàn tiền)
    App->>Audit: Ghi AuditLog (AccountID, IP, ActionType, EntityName, RecordID)
    App->>Audit: Ghi AuditLogDetail (FieldName, OldValue, NewValue)
    
    AdminUser->>Rep: Yêu cầu chạy báo cáo doanh thu / tồn kho cuối tháng
    Rep->>Rep: Ghi ReportExecution (Parameters, RequestedAt)
    Rep->>Snap: Chốt số liệu kỳ kế toán -> Tạo ReportSnapshot (Đóng băng dữ liệu)
```

#### 2. Bảng Tra cứu Thực thể Tham gia & Ý nghĩa Chi tiết:

| Mã | Tên Thực thể | Phân loại | Vai trò & Ý nghĩa Nghiệp vụ trong Luồng | Thuộc tính Trọng yếu |
| :---: | :--- | :--- | :--- | :--- |
| **E65** | `AuditLog` | Audit Entity | Nhật ký ghi nhận các thao tác nghiệp vụ và sự kiện an ninh quan trọng (Header). | `AuditLogID`, `AccountID`, `ActionType`, `EntityName`, `RecordID`, `ActionAt`, `IPAddress` |
| **E66** | `AuditLogDetail` | Weak / Detail | Chi tiết các thuộc tính bị thay đổi (giá trị trước và sau khi chỉnh sửa). | `AuditLogDetailID`, `AuditLogID`, `FieldName`, `OldValue`, `NewValue` |
| **E67** | `ReportDefinition` | Catalog Entity | Danh mục định nghĩa các mẫu báo cáo quản trị và nghiệp vụ trong hệ thống. | `ReportDefinitionID`, `ReportCode`, `ReportName`, `DataSource`, `ReportType` |
| **E68** | `ReportExecution` | Event Entity | Nhật ký theo dõi mỗi lần người dùng kích hoạt chạy hoặc tải xuất báo cáo. | `ReportExecutionID`, `ReportDefinitionID`, `AccountID`, `ExecutionStatus`, `CompletedAt` |
| **E69** | `ReportSnapshot` | Document / Snapshot | Ảnh chụp dữ liệu báo cáo được chốt cố định theo kỳ kế toán, bảo đảm bất biến. | `ReportSnapshotID`, `ReportDefinitionID`, `SnapshotCode`, `PeriodStart`, `PeriodEnd`, `DataLocation` |
| **E70** | `ReportPermission` | Associative Entity | Phân quyền chi tiết cho phép nhóm vai trò (`Role`) được xem hoặc xuất báo cáo. | `ReportDefinitionID`, `RoleID`, `CanView`, `CanExport`, `CanCreateSnapshot` |

#### 3. Bất biến Kiến trúc & Nguyên tắc Chốt:
* **Ghi vết Kiểm toán Bất biến (Tamper-evident Audit):** Bản ghi `AuditLog` và `AuditLogDetail` chỉ cho phép quyền `INSERT`. Thu hồi quyền `UPDATE` và `DELETE` trên bảng audit khỏi toàn bộ các tài khoản kết nối ứng dụng.
* **Đóng băng Ảnh chụp Báo cáo (Report Freezing):** Số liệu báo cáo tài chính cuối tháng được chốt vào `ReportSnapshot`. Các truy vấn phân tích của ban giám đốc sẽ đọc từ snapshot này, ngăn chặn nguy cơ các giao dịch mới phát sinh làm sai lệch số liệu của kỳ kế toán đã khóa sổ.

---

## 4. THUYẾT MINH CHI TIẾT 10 PHƯƠNG ÁN KIẾN TRÚC (ĐÃ CHUẨN HOÁ THEO REVIEW v2.1)

---

### Phương án 1 (P0): Mô hình Giám định `ReturnLineAssessment` (E72) & Lịch sử Đánh giá (Audit Trail)

#### 1. Bối cảnh & Điểm cần chuẩn hoá:
Trong thiết kế ban đầu, việc hoàn tiền bị ràng buộc cứng vào việc hàng đã nhập lại kho hay chưa (`Restocked`). Khi hàng bị hư hỏng do vận chuyển hoặc lỗi sản xuất nghiêm trọng phải tiêu hủy tại chỗ (`SCRAP_OUT`), nếu không có cơ chế phân tách, hệ thống sẽ chặn hoàn tiền hoặc ép buộc nhập hàng ảo vào kho.
Bên cạnh đó, việc giám định thực tế thường trải qua nhiều giai đoạn: duyệt một phần, chờ kiểm tra từ hãng, từ chối, hoặc đổi hàng thay vì hoàn tiền.

#### 2. Quyết định kiến trúc chuẩn hoá:
* Tách độc lập quyết định tài chính khỏi định đoạt kho hàng bằng thực thể trung gian **`ReturnLineAssessment` (E72)**.
* **Tách bạch Thuộc tính Trách nhiệm và Hướng Xử lý:**
  * `FaultParty`: VENDOR, CARRIER, CUSTOMER, NO_FAULT, SHARED.
  * `ResolutionType`: REFUND, EXCHANGE, REPAIR, NO_ACTION.
  * `RefundEligibilityStatus`: PENDING, APPROVED, PARTIALLY_APPROVED, REJECTED.
  * `ApprovedRefundAmount`: Số tiền hàng được duyệt hoàn.
  * `ShippingRefundAmount`: Phí vận chuyển được duyệt hoàn (theo chính sách).
* **Loại bỏ quy tắc cứng 100%:** Trách nhiệm của VENDOR/CARRIER không mặc định sinh ra quyền hoàn tiền 100% kèm phí ship ở tầng DDL; đây là tham số cấu hình của Policy Engine.
* **Mô hình Lịch sử Đánh giá (Audit Trail):** E72 hỗ trợ lưu vết phiên bản (`AssessmentVersion`, `IsCurrent`, `AssessmentStatus`: DRAFT, SUBMITTED, FINAL, SUPERSEDED). Khi cần tái thẩm định, bản ghi cũ chuyển `IsCurrent = 0` và bản ghi mới ghi nhận kết quả điều chỉnh.
* **Bộ ràng buộc toán học:**
  $$\sum \text{AssessedQty} \le \text{ReceivedQty}$$
  $$\text{ApprovedRefundQty} \le \text{AssessedEligibleQty}$$
  $$\text{ApprovedRefundAmount} \le \text{AllocatedNetAmount}(\text{ApprovedRefundQty})$$

```mermaid
flowchart TD
    RL["ReturnReceiptLine (E63)<br>ReceivedQty = 5"] --> A1["Assessment 1 (E72)<br>AssessedQty = 3<br>FaultParty: VENDOR<br>Resolution: REFUND<br>Status: APPROVED<br>ApprovedRefundAmount = 300.000đ"]
    RL --> A2["Assessment 2 (E72)<br>AssessedQty = 2<br>FaultParty: CUSTOMER<br>Resolution: NO_ACTION<br>Status: REJECTED<br>Reason: Rách tem niêm phong"]
    A1 --> INV["Kho vật lý: SCRAP_OUT<br>(Tiêu huỷ phế phẩm)"]
    A1 --> FIN["Kế toán: Lập RefundTransaction<br>(Hoàn tiền đợt 1: 300.000đ)"]
```

---

### Phương án 2 (P0): Quản lý Tồn kho Cách ly & Hư hỏng - Chuẩn hoá Ngữ nghĩa & Ghi sổ Biến động

#### 1. Bối cảnh & Chuẩn hoá Ngữ nghĩa:
Tồn kho vật lý trong kho bao gồm nhiều trạng thái chất lượng khác nhau. Cần tránh nhập nhằng giữa **Trạng thái chất lượng tồn kho** và **Loại giao dịch xuất kho**.
* **Chuẩn hóa Chất lượng Tồn kho:**
  * `SELLABLE`: Hàng thương phẩm đạt chuẩn, sẵn sàng kinh doanh.
  * `QUARANTINED`: Hàng cách ly chờ kiểm tra, thẩm định chất lượng hoặc tranh chấp.
  * `DAMAGED`: Hàng xác định hư hỏng, lỗi, không thể bán thương mại.
* **Chuẩn hóa Nghiệp vụ Xuất hủy:**
  * `SCRAP_OUT` là **Loại nghiệp vụ giao dịch (Transaction Type)** ghi nhận việc đưa hàng phế phẩm ra khỏi kho để tiêu hủy. Hàng sau khi `SCRAP_OUT` đã rời khỏi kho nên **KHÔNG** tồn tại trạng thái tồn kho vật lý mang tên "DISPOSED".

#### 2. Cấu trúc Số dư & Bộ 5 Declarative CHECK Constraints trên `InventoryBalance` (E11):
Tất cả các đại lượng trong bảng số dư phải thỏa mãn hệ tiên đề:
1. `OnHandQty >= 0` (Tổng tồn kho vật lý không âm).
2. `ReservedQty >= 0` (Số lượng giữ hàng cho đơn chưa xuất không âm).
3. `QuarantinedQty >= 0` (Số lượng cách ly kiểm định không âm).
4. `DamagedQty >= 0` (Số lượng hư hại xác định không âm).
5. `OnHandQty >= (ReservedQty + QuarantinedQty + DamagedQty)` (Tổng các trạng thái thành phần không được vượt quá tổng tồn kho vật lý).

Công thức Khả dụng Thương mại (Available to Promise):
$$\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$$

#### 3. Nguyên tắc Ghi sổ Biến động Kép trên `InventoryTransactionLine` (E16):
Mọi thay đổi số dư kho bắt buộc phải gắn liền với một bản ghi `InventoryTransactionLine`. Giao dịch ghi sổ và cập nhật `InventoryBalance` phải được thực thi trong cùng một Database Transaction nguyên tử:
* **Chuyển trạng thái nội bộ (VD: Chuyển 5 sản phẩm từ QUARANTINED sang DAMAGED):** Không làm thay đổi tổng tồn kho vật lý `OnHand`.
  $$\text{OnHandDelta} = 0, \quad \text{QuarantinedDelta} = -5, \quad \text{DamagedDelta} = +5$$
* **Xuất hủy phế phẩm (SCRAP_OUT 5 sản phẩm DAMAGED):** Giảm trực tiếp tồn kho vật lý và giảm trạng thái nguồn tương ứng.
  $$\text{OnHandDelta} = -5, \quad \text{DamagedDelta} = -5, \quad \text{QuarantinedDelta} = 0$$

---

### Phương án 3 (P0): Thuật toán Phân bổ Chiết khấu Đơn hàng LRM trên Tập Đủ Điều kiện & Công thức Hoàn tiền Nguyên số VND

#### 1. Bối cảnh & Thống nhất Kiểu Dữ liệu Tiền tệ:
Khi khách hàng áp mã giảm giá tổng đơn (Order-level Discount), khoản chiết khấu phải được phân bổ về từng dòng hàng (`OrderLine`) đủ điều kiện để phục vụ tính giá vốn, thuế và hoàn tiền.
Do đơn vị tiền tệ giao dịch của RetailCore là Việt Nam Đồng (VND) - loại tiền tệ không có phần thập phân lẻ, việc chia tỷ lệ rất dễ phát sinh phần dư.
* **Quy chuẩn Kiểu dữ liệu Tiền tệ:**
  * Toàn bộ các trường tiền tệ giao dịch nghiệp vụ (`LineGrossAmount`, `LineNetAmount`, `AllocatedOrderDiscount`, `ApprovedRefundAmount`, `TotalAmount`) sử dụng kiểu **`DECIMAL(19,0)`** (hoặc `BIGINT`).
  * Các trường đơn giá trung gian, tỷ lệ chiết khấu, giá vốn bình quân di động (`UnitPrice`, `CurrentAverageCost`) sử dụng kiểu **`DECIMAL(19,4)`** để bảo toàn độ chính xác trong các phép tính toán học.

#### 2. Thuật toán Largest Remainder Method (Hare-Niemeyer) trên `EligibleBase`:
Chỉ những dòng hàng thỏa mãn điều kiện áp dụng voucher mới tham gia mẫu số phân bổ:
$$\text{EligibleBase}_i = \text{LineGrossAmount}_i - \text{LineProductDiscount}_i$$
1. Tính phần giảm giá cơ sở nguyên (Base Integer Discount):
   $$B_i = \left\lfloor \frac{\text{EligibleBase}_i \times D}{\sum_{j \in \text{Eligible}} \text{EligibleBase}_j} \right\rfloor$$
2. Tính phần dư phân số (Fractional Remainder):
   $$R_i = \left(\frac{\text{EligibleBase}_i \times D}{\sum_{j \in \text{Eligible}} \text{EligibleBase}_j}\right) - B_i$$
3. Tính tổng số tiền đồng nguyên còn thiếu:
   $$K = D - \sum_{i=1}^N B_i$$
4. Sắp xếp các dòng theo $R_i$ giảm dần; nếu $R_i$ bằng nhau thì ưu tiên theo `OrderLineID ASC` (Deterministic Tie-breaker). Cộng thêm $1$ VNĐ vào $B_i$ cho $K$ dòng đầu tiên.
5. **Quy tắc biên:** Dòng không đủ điều kiện nhận $AllocatedOrderDiscount = 0$. Giới hạn giảm giá tối đa của voucher (MaxDiscountAmount) được áp dụng trước khi phân bổ. Tổng phân bổ bảo đảm bằng đúng $D$.

#### 3. Công thức Hoàn tiền Tích luỹ Nguyên số $F(q)$ cho Hoàn tiền Từng phần:
Khi dòng đơn hàng có số lượng $Q$, tổng số tiền thực trả sau phân bổ là $A$ (`LineNetAmount`). Khách hàng trả lại $q$ món ($1 \le q \le Q$).
Số tiền hoàn lũy tiến cho $q$ món được tính bằng hàm nguyên đơn điệu:
$$F(q) = \left\lfloor \frac{A}{Q} \right\rfloor \cdot q + \min\left(q,\; A \pmod Q\right)$$

Số tiền duyệt hoàn cho lần trả thứ $k$ với số lượng $\Delta q_k = q_k - q_{k-1}$:
$$\Delta \text{Refund}_k = F(q_k) - F(q_{k-1})$$

* **Bảo toàn Tài chính:** $F(0) = 0$ và $F(Q) = A$ (hoàn đủ 100% khi trả hết hàng).
* **Tách bạch Phí Vận chuyển:** $F(q)$ chỉ áp dụng cho tiền hàng ròng $A$. Phí vận chuyển được hoàn (nếu có) được tính riêng vào `ShippingRefundAmount` theo chính sách bảo hành.
* **Phân biệt Duyệt hoàn và Thực thi:** `ApprovedRefundAmount` là số tiền được cấp thẩm quyền duyệt trên E72; giao dịch tiền mặt/ngân hàng chỉ hoàn tất khi bản ghi `RefundTransaction` ghi nhận trạng thái `SUCCEEDED`.

---

### Phương án 4 (P1): Ràng buộc Bắt buộc 1:1 qua Stored Procedure Giao dịch & Phân quyền Bảo mật DML

#### 1. Giới hạn Kỹ thuật của DDL trong Ràng buộc Bắt buộc 1:1:
Trong chuẩn SQL, một quan hệ 1:1 tùy chọn ở một đầu có thể khống chế bằng `FOREIGN KEY ... UNIQUE`. Tuy nhiên, DDL thuần túy **chỉ khống chế cận trên $\le 1$, không thể khống chế cận dưới $\ge 1$** (tính bắt buộc). Nếu không có cơ chế kiểm soát truy cập và giao dịch, dòng cha có thể tồn tại vĩnh viễn mà không có dòng con tương ứng (Orphan Parent).

#### 2. Giải pháp Kiến trúc Phối hợp DDL, Stored Procedure và Phân quyền Bảo mật:
* **Tầng DDL:** Khai báo khóa ngoại kèm ràng buộc duy nhất để chặn việc gán trùng lặp:
  `Customer (AccountID PK_FK REFERENCES Account(AccountID) UNIQUE)`
* **Tầng Phân quyền (Security & Access Control):** Thu hồi toàn bộ quyền `INSERT`, `UPDATE`, `DELETE` trực tiếp trên các bảng `Account`, `Customer`, `Employee` khỏi tài khoản kết nối ứng dụng (Application DB User). Bắt buộc mọi thao tác phải thông qua Stored Procedure được ủy quyền (`EXECUTE`).
* **Tầng Xử lý Giao dịch:** Bắt buộc toàn bộ thao tác tạo mới thực thể 1:1 phải thông qua **Stored Procedure nghiệp vụ nguyên tử** với `SET XACT_ABORT ON`:
  ```sql
  CREATE PROCEDURE dbo.usp_CreateCustomerAccount
      @Username NVARCHAR(50),
      @Email NVARCHAR(100),
      @CustomerCode VARCHAR(20),
      @LoyaltyTierID INT
  AS
  BEGIN
      SET NOCOUNT ON;
      SET XACT_ABORT ON;
      BEGIN TRANSACTION;
          DECLARE @NewAccountID BIGINT;
          
          -- 1. Tạo bản ghi Account (Parent)
          INSERT INTO Account (Username, Email, AccountType, CreatedAt)
          VALUES (@Username, @Email, 'CUSTOMER', SYSDATETIMEOFFSET());
          
          SET @NewAccountID = SCOPE_IDENTITY();

          -- 2. Tạo bản ghi Customer (Child subtype) bắt buộc trong cùng Transaction
          INSERT INTO Customer (AccountID, CustomerCode, LoyaltyTierID)
          VALUES (@NewAccountID, @CustomerCode, @LoyaltyTierID);
      COMMIT TRANSACTION;
  END;
  ```
* **Kế thừa Table-Per-Type (TPT):** Phân định `Account` làm bảng cha lưu thông tin xác thực dùng chung; `Customer` và `Employee` kế thừa thông qua `AccountID` vừa là Primary Key vừa là Foreign Key (`PK_FK`). Ràng buộc loại trừ (Subtype Exclusivity) được kiểm soát để một Account không thể đồng thời là Customer và Employee.

---

### Phương án 5 (P1): Quản lý Webhook Bền vững, Idempotency & Phân nhánh Trạng thái Thanh toán

#### 1. Chuẩn hóa Định danh & Thuộc tính `PaymentWebhookLog` (E71):
Để hỗ trợ môi trường đa tài khoản cổng thanh toán (Multi-Merchant Accounts) và tách biệt môi trường Sandbox/Production, bảng E71 được chuẩn hóa:
* **Khóa duy nhất:**
  $$\text{UNIQUE}(\text{ProviderID},\; \text{MerchantAccountID},\; \text{ProviderEventID},\; \text{Environment})$$
* **Danh mục Thuộc tính Đầy đủ:** `WebhookLogID`, `ProviderID` (đối tác thanh toán: VNPAY, MOMO), `MerchantAccountID`, `ProviderEventID`, `Environment` (SANDBOX / PRODUCTION), `ProcessingStatus` (RECEIVED, PENDING, PROCESSED, FAILED), `ReceivedAt`, `ProcessedAt`, `Payload` (NVARCHAR(MAX) lưu trữ bền vững), `ErrorMessage`, `RetryCount`.

#### 2. Tách biệt Webhook Receipt và Webhook Processing:
* **Webhook Receipt:** Tiếp nhận raw payload, lưu log bền vững vào cơ sở dữ liệu `PaymentWebhookLog`. Chỉ sau khi bản ghi được `INSERT` thành công mới phản hồi HTTP 200 cho đối tác.
* **Webhook Processing:** Thực thi nghiệp vụ cập nhật trạng thái giao dịch. Khi có lỗi nghiệp vụ dẫn đến `FAILED`, cho phép cơ chế retry replay idempotent xử lý lại thay vì bỏ qua vô điều kiện.

#### 3. Mô hình Cây Trạng thái Thanh toán (Payment State Tree):
Loại bỏ sơ đồ chuyển trạng thái tuyến tính sai lệch. Trạng thái `Payment` được phân nhánh chuẩn hóa:

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> AUTHORIZED: Cổng hỗ trợ 2-phase auth
    PENDING --> PAID: Thu tiền 1 đợt thành công
    PENDING --> PARTIALLY_PAID: Thu tiền từng kiện COD (Split Shipment)
    PENDING --> FAILED: Thanh toán thất bại
    PENDING --> EXPIRED: Hết hạn phiên thanh toán
    PENDING --> CANCELLED: Khách hủy đơn

    AUTHORIZED --> PAID: Capture thành công
    AUTHORIZED --> CANCELLED: Void ủy quyền

    PARTIALLY_PAID --> PAID: Thu đủ 100% các kiện
    PARTIALLY_PAID --> PARTIALLY_REFUNDED: Hoàn tiền kiện đã nhận

    PAID --> PARTIALLY_REFUNDED: Hoàn tiền một phần
    PAID --> REFUNDED: Hoàn tiền toàn bộ

    PARTIALLY_REFUNDED --> PARTIALLY_REFUNDED: Hoàn tiền thêm các đợt sau
    PARTIALLY_REFUNDED --> REFUNDED: Hoàn hết 100% số tiền thực thu
```

#### 4. Khóa Ngoại Tự Tham chiếu Hoàn tiền (`OriginalPaymentTransactionID`):
Mỗi bản ghi giao dịch hoàn tiền (`TransactionType = 'REFUND'`) bắt buộc phải lưu `OriginalPaymentTransactionID` trỏ về giao dịch thu tiền gốc với các quy tắc:
* Giao dịch gốc phải có trạng thái `SUCCEEDED`.
* Giao dịch hoàn phải thuộc cùng hồ sơ `Payment` cha.
* Ràng buộc kiểm soát ngân quỹ:
  $$\sum \text{RefundTransaction.Amount} \le \text{OriginalPaymentTransaction.Amount} - \sum \text{PriorRefunds}$$

---

### Phương án 6 (P1): Độc lập Bốn Máy Trạng thái & Phân định Mốc Đóng Đơn hàng

#### 1. Độc lập Bốn Máy Trạng thái Nghiệp vụ:
Tách rời hoàn toàn các chiều vận hành:
$$\text{OrderStatus (Thương mại)} \ne \text{PaymentStatus (Tài chính)} \ne \text{ShipmentStatus (Hậu cần)} \ne \text{ReturnStatus (Đổi trả)}$$

* **Loại bỏ trạng thái RMA khỏi Đơn hàng:** `RMA_PROCESSING` thuộc về `ReturnRequest.ReturnStatus`, **tuyệt đối không phải là trạng thái của `SalesOrder`**. Một đơn hàng đã giao thành công vẫn giữ trạng thái thương mại ổn định trong khi khách khiếu nại đổi trả độc lập.

#### 2. Vòng đời Thương mại của `SalesOrder`:
```mermaid
stateDiagram-v2
    [*] --> DRAFT
    DRAFT --> CONFIRMED: Khách đặt hàng thành công
    CONFIRMED --> PROCESSING: Điều phối kho & Đóng gói
    PROCESSING --> DELIVERED: Tất cả Shipment giao thành công (DeliveredAt ghi nhận)
    DELIVERED --> CLOSED: Quá hạn ReturnEligibleUntil & Không có RMA mở (ClosedAt ghi nhận)
    DRAFT --> CANCELLED: Khách hủy đặt hàng
    CONFIRMED --> CANCELLED: Hủy trước khi xuất kho
    CLOSED --> [*]
    CANCELLED --> [*]
```

#### 3. Phân định Ba Mốc Thời gian trên `SalesOrder`:
1. **`DeliveredAt`:** Thời điểm hoàn tất giao hàng (cập nhật bằng thời điểm giao thành công của kiện hàng cuối cùng).
2. **`ReturnEligibleUntil`:** Hạn cuối cùng tổng hợp mang tính thông tin cấp đơn hàng ($\max(\text{Shipment.DeliveredAt} + \text{PolicyDays})$). Quyết định hợp lệ của từng sản phẩm cụ thể bắt buộc phải đối chiếu với `Shipment.DeliveredAt` của chính kiện hàng chứa sản phẩm đó.
3. **`ClosedAt`:** Thời điểm chính thức đóng hồ sơ lưu trữ đơn hàng sau khi hết hạn đổi trả và không còn yêu cầu RMA nào mở.

---

### Phương án 7 (P1): Mô hình Đối soát COD 3 Lớp trong Đơn hàng Split Shipment

#### 1. Bản chất Nghiệp vụ COD 3 Lớp:
Để tránh tình trạng đánh dấu `PAID` sớm cho toàn bộ đơn hàng khi mới chỉ có một kiện được giao và thu tiền:
* **Lớp 1 (Customer Collection):** Số tiền Shipper thực thu của khách theo từng kiện hàng cụ thể (`CODCollection` gắn liền với `ShipmentID`). Khi nhận tiền, nghĩa vụ của khách với kiện đó hoàn tất.
* **Lớp 2 (Carrier Remittance):** Khoản tiền hãng vận chuyển chuyển khoản về tài khoản ngân hàng của RetailCore theo từng đợt đối soát bưu phẩm (`CODRemittance`).
* **Lớp 3 (Reconciliation):** Đối chiếu chênh lệch giữa số tiền Shipper đã thu và số tiền hãng thực tế chuyển về sau khi trừ cước vận chuyển (`CODReconciliation`).

#### 2. Quản lý Trạng thái Thanh toán Cấp Đơn hàng:
* Trong đơn hàng chia nhiều kiện (`Shipment A`: 400.000đ; `Shipment B`: 600.000đ):
  * Khi Kiện A giao thành công và thu 400.000đ: Cập nhật `Payment.AmountPaid = 400.000đ`, trạng thái chuyển sang **`PARTIALLY_PAID`**.
  * Khi Kiện B giao thành công và thu tiếp 600.000đ: `Payment.AmountPaid = 1.000.000đ`, trạng thái chính thức chuyển sang **`PAID`**.
* Cơ chế này đảm bảo dòng tiền tài chính phản ánh trung thực nghĩa vụ nợ, không tạo ra ngộ nhận đã thu đủ tiền khi còn kiện hàng đang lưu thông.

---

### Phương án 8 (P2): Giá vốn Bình quân Di động (WAC) khi Xuất trả NCC & Định giá Tồn kho Độc lập

#### 1. Nguyên lý Định giá khi Xuất trả Nhà Cung cấp (Supplier Return):
Khi xuất trả hàng lỗi hoặc tồn kho cho Nhà cung cấp, kế toán kho bắt buộc phải ghi giảm giá trị tồn kho theo **Giá vốn Bình quân Di động hiện hành (`CurrentAverageCost`)**, tuyệt đối không ghi giảm theo giá mua trên đơn PO gốc.

#### 2. Công thức Hạch toán Chênh lệch (Cost Variance):
Khoản chênh lệch giữa giá trị nhà cung cấp hoàn lại (thường theo giá mua PO) và giá trị giảm tồn kho WAC:
$$\text{InventoryValueDecrease} = \text{ReturnedQty} \times \text{CurrentAverageCost}$$
$$\text{SupplierRefundAmount} = \text{ReturnedQty} \times \text{POLine.UnitPrice}$$
$$\text{CostVariance} = \text{SupplierRefundAmount} - \text{InventoryValueDecrease}$$

* **Giả định Định giá Tồn kho RetailCore:** `CurrentAverageCost` phản ánh giá vốn bình quân áp dụng cho toàn bộ lượng tồn vật lý `OnHand` của SKU tại kho. Hàng `QUARANTINED` hay `DAMAGED` vẫn mang giá trị sổ sách kế toán cho đến khi có quyết định xử lý bằng chứng từ xuất hủy `SCRAP_OUT` (chuyển sang tài khoản chi phí tổn thất hàng hỏng). Các phân bổ trích lập tổn thất chi tiết sẽ được xử lý tại tầng kế toán tài chính.
* **Hàng Khách Trả Lại:** Khôi phục theo giá vốn xuất kho ban đầu (Original COGS) của đơn hàng đó nếu truy vết được; trường hợp không thể truy vết mới nhập theo WAC hiện hành.

---

### Phương án 9 (P2): Chuẩn hoá JSON Snapshot Biến thể & Chuẩn mực Lập chỉ mục SQL Server

#### 1. Cấu trúc JSON Snapshot Chuẩn hóa:
Để bảo đảm tính bất biến khi thuộc tính sản phẩm trong danh mục thay đổi sau 5-10 năm, `OrderLine.VariantAttributesSnapshot` lưu trữ mảng JSON cấu trúc chặt chẽ:
```json
[
  {
    "attributeId": 101,
    "attributeCode": "COLOR",
    "attributeName": "Màu sắc",
    "valueId": 502,
    "valueCode": "NAVY_BLUE",
    "displayValue": "Xanh Navy"
  },
  {
    "attributeId": 102,
    "attributeCode": "SIZE",
    "attributeName": "Kích cỡ",
    "valueId": 604,
    "valueCode": "XL",
    "displayValue": "XL"
  }
]
```

#### 2. Chuẩn mực Kỹ thuật về Truy vấn & Indexing trên SQL Server:
* **Loại bỏ Index theo Vị trí Mảng (`$[0]`):** Tuyệt đối không tạo chỉ mục dựa trên thứ tự phần tử JSON như `JSON_VALUE(VariantAttributesSnapshot, '$[0].valueCode')` vì thứ tự thuộc tính trong mảng JSON không mang tính định danh nghiệp vụ.
* **Nguyên tắc Indexing JSON:** Nếu có nhu cầu lọc hoặc tìm kiếm thường xuyên trên một thuộc tính ổn định, hệ thống sẽ trích xuất thành cột dữ liệu riêng hoặc sử dụng bảng dữ liệu phân tích chuẩn hóa (Star Schema / Fact-Dimension), không quét trực tiếp JSON trên bảng giao dịch `OrderLine`.

---

### Phương án 10 (P0/P2): Quản lý Giữ hàng Đa kho qua `AllocationID` & Truy vết Đổi trả qua `ReturnShipmentAllocation`

#### 1. Sửa lỗi Chỉ mục Giữ hàng Đang hoạt động:
Một dòng đơn hàng `OrderLine` có thể phân bổ xuất từ nhiều kho (Split Allocation). Mỗi phân bổ chỉ có duy nhất một dòng giữ hàng `ACTIVE`. Chỉ mục được định nghĩa chuẩn xác trên bảng **`StockReservationLine`**:
```sql
CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAllocation
ON StockReservationLine(AllocationID)
WHERE ReservationLineStatus = 'ACTIVE';
```

#### 2. Cơ chế Truy vết Đổi trả Đa Kiện qua `ReturnShipmentAllocation`:
Trong đơn hàng phân tách kiện (Split Shipment), mỗi kiện hàng có lộ trình và thời điểm giao thực tế khác nhau. Thời hạn đổi trả được tính độc lập cho từng kiện dựa trên **`Shipment.DeliveredAt`**.
Hệ thống thiết lập quan hệ truy vết:
$$\text{ReturnRequestLine} \longrightarrow \text{ReturnShipmentAllocation} \longrightarrow \text{ShipmentLine}$$

Ràng buộc toàn vẹn số lượng và thời gian:
$$\text{RequestedAt} \le \text{Shipment.DeliveredAt} + \text{PolicyDays}$$
$$\sum \text{AllocatedReturnQty} \le \text{ShipmentLine.ShipmentQty} - \text{PreviouslyReturnedQty}$$

Cơ chế này giải quyết triệt để bài toán: Một dòng đơn hàng mua 5 sản phẩm, giao làm 2 đợt lệch ngày, khách trả 2 sản phẩm thì hệ thống xác định chính xác 2 sản phẩm đó thuộc kiện nào và còn trong hạn đổi trả hay không.

---

## 5. LỘ TRÌNH ĐIỀU CHỈNH MILESTONE 3 (LOGICAL DATABASE DESIGN)

Mục tiêu và phương pháp luận triển khai Milestone 3 được tổ chức thành 4 phân kỳ kỹ thuật rõ ràng:

```mermaid
flowchart LR
    M301["M3.01: Relational Schema Mapping<br>(72 Thực thể Khái niệm -> Schema Quan hệ Tối ưu)"] --> M302["M3.02: Chuẩn hoá Dữ liệu Toán học<br>(Candidate Keys -> FDs -> 3NF -> BCNF Test)"]
    M302 --> M303["M3.03: SQL Server Sizing & Data Types<br>(DECIMAL 19,0 cho VNĐ, 19,4 cho Đơn giá/WAC)"]
    M303 --> M304["M3.04: Declarative Constraints & SPs<br>(DDL CHECK, Filtered Index, Transactional SPs)"]
```

### Bước 1 (M3.01) – Relational Schema Mapping (Ánh xạ Lược đồ Quan hệ)
* **Mục tiêu:** Chuyển đổi toàn diện 72 thực thể quan niệm thành Lược đồ Quan hệ (Relational Schema) tương thích hoàn hảo với SQL Server.
* **Không ép buộc cứng 72 bảng:** Số lượng bảng vật lý được quyết định dựa trên yêu cầu chuẩn hóa, lưu vết kiểm toán và tối ưu truy vấn:
  * Ánh xạ 1:1 độc lập thành bảng riêng.
  * Tách bảng phục vụ lưu vết lịch sử (Audit Trail / Versions cho E72).
  * Bổ sung bảng liên kết truy vết giao vận đa kiện (`ReturnShipmentAllocation`).
  * Gộp thực thể nếu các phụ thuộc hàm và quan hệ cho phép mà không làm phát sinh dị thường dữ liệu.

### Bước 2 (M3.02) – Chuẩn hóa Dữ liệu Toán học (3NF & BCNF Evaluation)
* **Quy trình chuẩn hóa 6 bước nghiêm ngặt:**
  1. Xác định toàn bộ Khóa ứng viên (Candidate Keys) của từng quan hệ.
  2. Xác định tập Phụ thuộc hàm đầy đủ (Full Functional Dependencies - FDs).
  3. Kiểm tra và chứng minh 1NF, 2NF và 3NF.
  4. Kiểm tra điều kiện Boyce-Codd Normal Form (BCNF): Với mọi phụ thuộc hàm $X \to Y$, $X$ bắt buộc phải là Superkey.
  5. Đánh giá tính Bảo toàn Phụ thuộc hàm (Dependency Preservation) và Nối không mất mát thông tin (Lossless Join) khi phân rã.
  6. Trường hợp giữ lại thiết kế 3NF không đạt BCNF hoặc phi chuẩn hóa có chủ đích (Aggregates, Denormalized Snapshots), bắt buộc phải lập biên bản đánh đổi kỹ thuật (Technical Trade-off Justification).

### Bước 3 (M3.03) – Định kiểu Dữ liệu SQL Server & Ước tính Dung lượng (Sizing)
* **Chuẩn hóa Kiểu Tiền tệ & Định lượng:**
  * Tiền tệ giao dịch VNĐ: `DECIMAL(19,0)` hoặc `BIGINT`.
  * Đơn giá chi tiết, Tỷ lệ chiết khấu, Giá vốn bình quân: `DECIMAL(19,4)`.
  * Khóa chính (Surrogate Keys): `BIGINT IDENTITY(1,1)` cho các bảng giao dịch lớn; `INT` cho danh mục trung bình; `TINYINT` cho bảng trạng thái/loại.
  * Chuỗi văn bản Unicode: `NVARCHAR` có giới hạn độ dài xác định (`NVARCHAR(50)`, `NVARCHAR(255)`, `NVARCHAR(500)`), tránh sử dụng tràn lan `NVARCHAR(MAX)`.
  * Thời gian toàn cầu: `DATETIMEOFFSET(7)` để lưu trữ chính xác múi giờ Việt Nam (+07:00) và đồng bộ hệ thống.
* **Tính toán Dung lượng 3 năm (Capacity Planning):** Tính kích thước dòng trung bình (Row Size), dung lượng bảng (Data Space), dung lượng chỉ mục (Index Space) dựa trên tải 1.000.000 đơn hàng/năm.

### Bước 4 (M3.04) – Đặc tả Chi tiết Ràng buộc Kỹ thuật & Stored Procedures
* Đặc tả đầy đủ mã nguồn DDL cho toàn bộ các ràng buộc khai báo: `PRIMARY KEY`, `FOREIGN KEY` (kèm hành vi `ON DELETE NO ACTION`), `CHECK Constraints`, `DEFAULT Constraints`.
* Đặc tả các chỉ mục có lọc (`Filtered Indexes`) giải quyết các bài toán trạng thái duy nhất.
* Đặc tả chữ ký và luồng thực thi của các Stored Procedures giao dịch cốt lõi: Đặt hàng, Giữ kho, Nhập/Xuất kho, Ghi nhận Webhook, Giám định đổi trả.

---

## 6. GATE CHECKLIST KIỂM CHỨNG & BẰNG CHỨNG THỰC CHỨNG (GATE VERIFICATION CHECKLIST)

### 6.1. Bảng Tiêu chí Kiểm chứng Cổng Chuyển giao Đầy đủ Bằng chứng:

| STT | Tiêu chí Kiểm chứng Kiến trúc | Kết quả Đánh giá | Bằng chứng / Tài liệu Kiểm chứng (Evidence Artifact) | Phương pháp Kiểm chứng (Verification Method) | Trạng thái Chuyển tiếp M3 |
| :---: | :--- | :---: | :--- | :--- | :--- |
| **01** | **Bảo tồn Ranh giới 72 Thực thể** | **ĐẠT (Passed)** | `docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md` | Đối chiếu danh mục 72 thực thể (E01-E72) với 112 Business Rules M1. | Cho phép chuyển sang M3.01 Schema Mapping. |
| **02** | **Đồng bộ Sơ đồ Mermaid ERD** | **ĐẠT (Passed)** | 12 files tại `docs/Milestones/M2-Conceptual-Design/ERD/mermaid/` | Cú pháp hóa thuần `PK`, `FK`, `UK`; re-render ảnh PNG không lỗi. | Sơ đồ sẵn sàng làm tài liệu trực quan cho M3. |
| **03** | **Độc lập Giám định vs Kho (P0-01)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.1) & `08-Architectural-Correction-Log.md` | Tách bạch `FaultParty` khỏi `ResolutionType`; mô hình Audit Trail E72. | Hoàn thiện cấu trúc Audit Trail tại M3.01. |
| **04** | **Toán học Tồn kho & 5 CHECK (P0-02)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.2) & `03-Cardinality-and-Constraints.md` | Chứng minh toán học $\text{AvailableQty} \ge 0$; chuẩn hóa `SELLABLE`, `QUARANTINED`, `DAMAGED`, `SCRAP_OUT`. | Sẵn sàng chuyển thành DDL CHECK Constraints tại M3.04. |
| **05** | **Triệt tiêu Sai số Làm tròn (P0-03)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.3) & Kịch bản Test LRM | Thuật toán Hare-Niemeyer trên `EligibleBase` + Công thức tích luỹ nguyên $F(q)$ + `DECIMAL(19,0)`. | Chuyển thành đặc tả kiểm thử logic (Test Specs) tại M3. |
| **06** | **Thực thi Bắt buộc 1:1 (P1-04)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.4) & T-SQL SP Template | Thu hồi quyền DML trực tiếp + Bắt buộc qua Stored Procedure giao dịch `XACT_ABORT ON`. | Đặc tả kịch bản SP tại M3.04. |
| **07** | **Webhook Bền vững & Idempotency (P1-05)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.5) & Schema E71 | `UNIQUE(ProviderID, MerchantAccountID, ProviderEventID, Environment)`; Lưu bền vững trước khi HTTP 200. | M3 thiết kế bảng trạng thái Webhook và luồng retry. |
| **08** | **Độc lập 4 Máy Trạng thái (P1-06)** | **ĐẠT (Passed)** | Báo cáo v2.1 (Mục 4.6) & State Transition Matrix | Loại bỏ `RMA_PROCESSING` khỏi `OrderStatus`; tách `DeliveredAt`, `ReturnEligibleUntil`, `ClosedAt`. | Thiết kế cột thời gian và State Transition Matrix tại M3. |
| **09** | **Đối soát COD 3 Lớp Split Shipment (P1-07)** | **ĐÃ GIẢI QUYẾT (Resolved)** | Báo cáo v2.1 (Mục 4.7) & `docs/M3-Issues-Resolution.md` | Chuẩn hóa 3 bảng: `CODCollection`, `CODRemittance`, `CODReconciliation`. Hỗ trợ `PARTIALLY_PAID`. | Sẵn sàng ánh xạ quan hệ tại M3.01. |
| **10** | **Truy vết Kiện hàng khi RMA (P2-10)** | **ĐÃ GIẢI QUYẾT (Resolved)** | Báo cáo v2.1 (Mục 4.10) & `docs/M3-Issues-Resolution.md` | Bổ sung `ReturnShipmentAllocation` kiểm tra hạn theo từng kiện và khống chế số lượng trả. | Sẵn sàng ánh xạ quan hệ tại M3.01. |

---

### 6.2. Quyết định Nghiệm thu Cổng Chuyển giao (Gate Sign-off Decision)

```
========================================================================================
                      GATE REVIEW SIGN-OFF: CONDITIONAL APPROVAL
========================================================================================
Dự án:                 RetailCore Enterprise E-Commerce Management System
Giai đoạn Đánh giá:    Kết thúc Milestone 2 (M2) -> Chuyển giao Milestone 3 (M3)
Quyết định:            CONDITIONAL APPROVAL FOR M3 (Chấp thuận Có Điều kiện)
----------------------------------------------------------------------------------------
ĐIỀU KIỆN THỰC THI:
1. ĐƯỢC PHÉP KHỞI ĐỘNG: 
   - Tiến hành ngay công đoạn M3.01 (Relational Schema Mapping) bắt đầu từ SA01 - Product Catalog.
2. NGUYÊN TẮC GIỮ MỞ (NON-BLOCKING ITERATION):
   - Không áp đặt cứng nhắc số lượng 72 bảng vật lý nếu thiết kế chuẩn hóa và kiểm toán 
     đòi hỏi tách/gộp hợp lý.
   - Không áp đặt 100% bảng phải đạt BCNF nếu việc phân rã làm mất phụ thuộc hàm hoặc 
     làm suy giảm nghiêm trọng hiệu năng nghiệp vụ.
   - Thiết kế chi tiết cấu trúc ReturnShipmentAllocation và đối soát COD 3 lớp trong quá trình
     triển khai M3.
3. TIẾP TỤC BẢO TRÌ VÀ CẬP NHẬT:
   - Mọi điều chỉnh cấu trúc phát sinh trong M3 sẽ được đồng bộ ngược (Reverse-Sync) vào
     tài liệu M2 để bảo đảm tính nhất quán xuyên suốt vòng đời dự án.
========================================================================================
```

---

## 7. KẾ HOẠCH KHỞI ĐỘNG M3.01 – RELATIONAL SCHEMA MAPPING

Theo đúng khuyến nghị kiến trúc, M3.01 sẽ không bắt đầu bằng việc viết DDL tạo bảng vội vã, mà thiết lập một bộ tài liệu **Ánh xạ Quan hệ có Kiểm soát** theo cấu trúc thư mục chuẩn:

```
docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/
├── 01-Mapping-Methodology.md          # Phương pháp luận ánh xạ thực thể sang quan hệ
├── 02-Entity-to-Relation-Mapping.md   # Bảng ánh xạ chi tiết 72 thực thể sang các quan hệ
├── 03-Primary-and-Candidate-Keys.md  # Xác định Surrogate Keys, Natural Keys & Candidate Keys
├── 04-Foreign-Key-Mapping.md          # Đặc tả khóa ngoại và hành vi toàn vẹn tham chiếu
├── 05-Relationship-Transformation.md  # Chuyển đổi quan hệ 1:1, 1:N, N:N, Subtypes TPT, Recursive
├── 06-Cross-Area-Relations.md         # Kiểm soát 40 liên kết liên vùng nghiệp vụ
├── 07-Open-Design-Decisions.md        # Ghi nhận các quyết định mở (COD 3 lớp, ReturnShipmentAllocation)
└── 08-Mapping-Validation.md           # Biên bản kiểm chứng chéo và đánh giá sẵn sàng sang M3.02
```

### Trình tự Thực hiện theo 11 Subject Areas:
1. **Ưu tiên 1 (Khởi động chuẩn hóa phương pháp):** `SA01 – Product Catalog` (E01 – E09) – Ít phụ thuộc liên miền, chuẩn hóa cấu trúc phân cấp đệ quy, biến thể SKU và giá trị thuộc tính.
2. **Ưu tiên 2:** `SA02 – Warehouse & Inventory` (E10 – E20).
3. **Ưu tiên 3:** `SA03 – Procurement` (E21 – E28).
4. **Ưu tiên 4:** `SA04 – Customer & Identity` (E29 – E37).
5. **Ưu tiên 5:** `SA05 – Shopping` (E38 – E39).
6. **Ưu tiên 6:** `SA06 – Sales` (E40 – E43).
7. **Ưu tiên 7:** `SA07 – Fulfillment & Shipping` (E44 – E48).
8. **Ưu tiên 8:** `SA08 – Promotion & Voucher` (E49 – E54).
9. **Ưu tiên 9:** `SA09 – Payment & Refund` (E55 – E57, E58, E59, E71).
10. **Ưu tiên 10:** `SA10 – Return & Exchange` (E60 – E64, E72).
11. **Ưu tiên 11:** `SA11 – Audit & Reporting` (E65 – E70).
12. **Rà soát Toàn diện & Nghiệm thu M3.01.**
