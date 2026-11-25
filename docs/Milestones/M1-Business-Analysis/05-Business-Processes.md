# 05. BUSINESS PROCESSES – SƠ ĐỒ QUY TRÌNH NGHIỆP VỤ CHUẨN HÓA (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT (Đồng bộ vòng đời Tồn kho, Giao thất bại `E75`, Giám định RMA `E72` & Đối soát COD 3 lớp `E77`–`E79`)

---

## 5.1. Quy trình Bán hàng, Giao vận Đa kho & Xử lý Giao thất bại (End-to-End Order & Fulfillment Process)

Quy trình khép kín từ lúc khách hàng đặt đơn có kiểm soát `IdempotencyRecord` (E74), khóa giữ tồn kho All-or-Nothing, thanh toán, xuất kho Split Shipment cho đến nhánh xử lý giao thất bại tối đa 3 lần và ký biên bản hoàn kho `DeliveryReturnReceipt` (E75).

~~~mermaid
flowchart TD
    Start(["Khách hàng tại trang Checkout"]) --> IdempCheck["Kiểm tra khóa IdempotencyRecord (E74)"]
    IdempCheck --> ValidOrder{"Dữ liệu, Giá Zero-Trust & Khuyến mãi 3 tầng hợp lệ?"}
    ValidOrder -- "Không" --> ShowErr["Trả lỗi xác thực chi tiết"]
    ShowErr --> Start

    ValidOrder -- "Có" --> LockStock["Phân bổ kho (E12) & Khóa InventoryBalance theo thứ tự VariantID, WarehouseID"]
    LockStock --> EnoughStock{"Đủ 100% AvailableQty cho mọi SKU (All-or-Nothing)?"}
    EnoughStock -- "Không" --> RollbackStock["Rollback toàn bộ, giữ ShoppingCart = ACTIVE"]
    RollbackStock --> Start

    EnoughStock -- "Có" --> CreateDocs["Tạo nguyên tử SalesOrder, OrderAddress, OrderLine (LRM + VAT), StockReservation (15p/5p) & Payment"]
    CreateDocs --> PayType{"Phương thức thanh toán?"}

    PayType -- "COD" --> ConfirmOrder["Chuyển OrderStatus = CONFIRMED (PaymentStatus = PENDING)"]
    PayType -- "Online" --> CallGateway["Gọi cổng thanh toán ngoài DB Transaction & chờ PaymentWebhookLog (E71)"]
    CallGateway --> PayResult{"Thanh toán thành công trong hạn giữ hàng?"}
    PayResult -- "Có" --> MarkPaid["PaymentStatus = PAID & OrderStatus = CONFIRMED"]
    PayResult -- "Không / Quá hạn ExpiresAt" --> ReleaseHold["Nhả ReservedQty, hoàn lượt Voucher & chuyển OrderStatus = CANCELLED"]
    ReleaseHold --> EndCancel(["Kết thúc đơn hủy"])

    ConfirmOrder --> PackShip["Từng kho đóng gói Shipment (E45) & ShipmentLine (E46)"]
    MarkPaid --> PackShip
    PackShip --> Dispatch["Bàn giao Carrier (DISPATCHED): Snapshot OutboundUnitCost & ghi SHIPMENT_OUT (-OnHandQty, -ReservedQty)"]
    Dispatch --> AttemptDeliver["Bưu tá giao hàng & ghi nhận DeliveryAttempt (E47)"]
    AttemptDeliver --> IsDelivered{"Giao thành công?"}

    IsDelivered -- "Có" --> DeliveredFlow["Shipment = DELIVERED; nếu COD thì ghi CODCollection (E77) & cập nhật tiến độ PAID"]
    DeliveredFlow --> EndSuccess(["Hoàn tất giao hàng"])

    IsDelivered -- "Không" --> CheckRetry{"Số lần thử < 3?"}
    CheckRetry -- "Còn lượt (< 3)" --> AttemptDeliver
    CheckRetry -- "Đã thất bại 3 lần" --> ReturnSender["Chuyển Shipment = RETURNED_TO_SENDER (Tuyệt đối CHƯA cộng tồn kho)"]
    ReturnSender --> DockReturn["Kiện hoàn về bến kho: Thủ kho kiểm đếm & ký DeliveryReturnReceipt (E75 - POSTED)"]
    DockReturn --> PostReturnIn["Ghi sổ DELIVERY_RETURN_IN (+OnHandQty theo OutboundUnitCost) & Hủy/Hoàn tiền hoặc Giao lại"]
    PostReturnIn --> EndReturnIn(["Kết thúc xử lý chuyển hoàn"])
~~~

---

## 5.2. Quy trình Tác động Tồn kho: Phân tách Giữ hàng Logic vs Sổ cái Vật lý (`CORR-02` & `CORR-08`)

> [!IMPORTANT]
> **Nguyên tắc Bất biến Sổ cái Kho:**
> * Các sự kiện **Giữ hàng / Nhả giữ hàng logic** (`StockReservation` / `StockReservationLine`) **chỉ** cập nhật cột `InventoryBalance.ReservedQty`, **tuyệt đối không** sinh bản ghi `InventoryTransaction` vật lý.
> * Chỉ các sự kiện làm thay đổi số lượng vật lý hoặc chuyển đổi giữa 3 trạng thái phẩm chất (`SELLABLE`, `QUARANTINED`, `DAMAGED`) mới ghi sổ bất biến vào `InventoryTransaction` (E15) & `InventoryTransactionLine` (E16) và tính lại giá vốn `Moving WAC`.

~~~mermaid
flowchart TD
    InvEvent(["Sự kiện Tác động Tồn kho"]) --> Branch{"Phân loại Bản chất Nghiệp vụ"}

    %% Logical Reservation Branch
    Branch -- "1. Khóa / Nhả giữ hàng Logic" --> ResAction{"Loại thao tác Reservation"}
    ResAction -- "Đặt đơn mới" --> HoldQty["Tăng ReservedQty & tạo StockReservationLine = ACTIVE"]
    ResAction -- "Hủy đơn / Hết hạn 15p" --> RelQty["Giảm ReservedQty & chuyển StockReservationLine = RELEASED / EXPIRED"]
    HoldQty --> Check5Rules["Kiểm tra 5 CHECK Constraints trên InventoryBalance (E11)"]
    RelQty --> Check5Rules

    %% Physical / Quality Ledger Branch
    Branch -- "2. Biến động Vật lý / Phẩm chất" --> LedgerType{"Loại chứng từ nguồn (POSTED)"}
    LedgerType -- "GoodsReceipt (AcceptedQty)" --> TxPO["PURCHASE_RECEIPT: +OnHandQty (SELLABLE) & Tính lại Moving WAC"]
    LedgerType -- "Shipment Handover" --> TxShip["SHIPMENT_OUT: -OnHandQty, -ReservedQty & Snapshot OutboundUnitCost"]
    LedgerType -- "DeliveryReturnReceipt (E75)" --> TxDelRet["DELIVERY_RETURN_IN: +OnHandQty & Khôi phục WAC theo OutboundUnitCost"]
    LedgerType -- "ReturnReceipt (E62) & KCS RMA (E72)" --> TxRMA["Bến nhập ghi RETURN_RECEIPT_IN (+OnHandQty, +QuarantinedQty) -> KCS chuyển SELLABLE (-QuarantinedQty, tính WAC) hoặc SCRAP_OUT (-OnHandQty, -QuarantinedQty)"]
    LedgerType -- "WarehouseTransfer (2 bước)" --> TxTrans["TRANSFER_OUT (-Kho nguồn, snapshot WAC) -> TRANSFER_IN (+Kho đích, tính lại WAC)"]
    LedgerType -- "StockCount / SupplierReturn" --> TxAdj["STOCK_ADJUSTMENT (ApprovedBy <> CreatedBy) hoặc SUPPLIER_RETURN_OUT (hạch toán CostVariance)"]

    TxPO --> WriteLedger["Ghi bất biến InventoryTransaction (E15) & InventoryTransactionLine 4-Delta (E16)"]
    TxShip --> WriteLedger
    TxDelRet --> WriteLedger
    TxRMA --> WriteLedger
    TxTrans --> WriteLedger
    TxAdj --> WriteLedger
    WriteLedger --> Check5Rules
    Check5Rules --> CommitInv(["Commit ACID Transaction"])
~~~

---

## 5.3. Quy trình Đổi trả Hàng hóa (RMA), Giám định KCS Độc lập & Hoàn tiền / Đổi bù (`CORR-01`, `CORR-03`, `CORR-10`)

~~~mermaid
flowchart TD
    RMAStart(["Khách hàng / CSKH tạo ReturnRequest (E60)"]) --> CheckWindow["Đối chiếu ReturnShipmentAllocation (E76): RequestedAt <= Shipment.DeliveredAt + PolicyDays"]
    CheckWindow --> ValidRMA{"Đủ điều kiện & CSKH duyệt?"}
    ValidRMA -- "Từ chối" --> RejectRMA["ReturnStatus = REJECTED & Lưu ReturnStatusHistory"]
    ValidRMA -- "Duyệt (APPROVED)" --> AssignWH["Chỉ định 1 Kho tiếp nhận cố định (WarehouseID)"]
    AssignWH --> DockRecv["Kho chốt ReturnReceipt (E62 - POSTED): Ghi RETURN_RECEIPT_IN (+OnHandQty, +QuarantinedQty)"]
    DockRecv --> KCSAssess["Nhân viên KCS lập ReturnLineAssessment (E72 - Phiên bản hóa IsCurrent = 1)"]

    KCSAssess --> PhysPath["Nhánh 1 (Định đoạt Vật lý - InventoryDisposition): SELLABLE (-QuarantinedQty, tính WAC) / QUARANTINED / DISPOSED (SCRAP_OUT: -OnHandQty, -QuarantinedQty)"]
    KCSAssess --> FinPath{"Nhánh 2 (Quyền lợi Khách - RefundEligibilityStatus & ResolutionType)"}

    FinPath -- "PENDING_INSPECTION (Cách ly chờ hãng)" --> WaitReassess["Chờ kết luận giám định -> Tạo AssessmentVersion + 1 mới"]
    FinPath -- "REJECTED / NO_ACTION" --> CloseNoRefund["Từ chối hoàn tiền & thông báo lý do"]
    FinPath -- "APPROVED + REFUND" --> CalcDeltaF["Khóa UPDLOCK OrderLine: Tính tiền hoàn lũy tiến ΔF = F(a+b) - F(a) & kiểm tra Clawback Voucher"]
    CalcDeltaF --> ExecRefund["Kế toán duyệt Refund (E58) & chi RefundTransaction (E59 trỏ OriginalPaymentTransactionID)"]
    FinPath -- "APPROVED + EXCHANGE" --> CreateExOrder["Sinh đơn đổi bù SalesOrder (OrderType = EXCHANGE_REPLACEMENT, ParentOrderID, TotalAmount = 0)"]

    PhysPath --> DoneRMA(["Khép kín hồ sơ RMA"])
    ExecRefund --> DoneRMA
    CreateExOrder --> DoneRMA
    CloseNoRefund --> DoneRMA
~~~

---

## 5.4. Quy trình Đối soát Công nợ Thu hộ COD Hai Đầu & Ba Lớp (`CORR-07`)

~~~mermaid
flowchart LR
    ShipDelivered(["Bưu tá giao kiện COD thành công"]) --> Layer1["Lớp 1: Ghi nhận CODCollection (E77) theo ShipmentID"]
    Layer1 --> CustObligation["Đầu 1 (Nghĩa vụ Khách): Cập nhật PaymentStatus = PARTIALLY_PAID / PAID"]
    Layer1 --> CarrierDebt["Đầu 2 (Công nợ Hãng): CarrierSettlementStatus = COLLECTED_BY_CARRIER"]
    CarrierDebt --> Layer2["Lớp 2: Hãng chuyển khoản & gửi bảng kê -> Kế toán lập CODRemittance (E78)"]
    Layer2 --> Layer3["Lớp 3: Khớp từng kiện qua CODReconciliation (E79) & kiểm tra VarianceAmount"]
    Layer3 --> Reconciled{"Khớp đủ 100%?"}
    Reconciled -- "Có" --> PostSettleTx["Sinh PaymentTransaction (COD_SETTLEMENT) & CarrierSettlementStatus = RECONCILED"]
    Reconciled -- "Lệch số tiền" --> FlagDiscrepancy["Đánh dấu DISCREPANCY để truy thu hãng vận chuyển (Không ảnh hưởng trạng thái PAID của khách)"]
~~~
