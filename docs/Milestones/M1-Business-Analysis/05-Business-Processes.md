# 05. BUSINESS PROCESSES – SƠ ĐỒ QUY TRÌNH NGHIỆP VỤ

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

---

## 5.1. Quy trình tổng thể bán hàng (End-to-End Sales Process)

Đây là quy trình quan trọng nhất của RetailCore, kết nối từ lúc khách hàng duyệt sản phẩm, kiểm tra khuyến mãi/tồn kho, giữ hàng, thanh toán cho đến khi kho xuất giao và hoàn tất đơn hàng.

~~~mermaid
flowchart TD
    Start(["Start"]) --> Browse["Customer browses products"]
    Browse --> AddCart["Add products to cart"]
    AddCart --> IsCheckout{"Checkout?"}
    IsCheckout -- "No" --> Browse
    IsCheckout -- "Yes" --> SelectAddr["Select shipping address"]
    SelectAddr --> SelectPay["Select payment method"]
    SelectPay --> ValidateOrder["Validate voucher and pricing"]
    ValidateOrder --> IsValidOrder{"Valid order?"}
    IsValidOrder -- "No" --> ShowErr["Display validation errors"]
    ShowErr --> SelectAddr
    IsValidOrder -- "Yes" --> CheckInv["Check inventory"]
    CheckInv --> EnoughStock{"Enough stock?"}
    EnoughStock -- "No" --> NotifyStock["Notify insufficient stock"]
    NotifyStock --> AddCart
    EnoughStock -- "Yes" --> CreateReserve["Create order and reserve stock"]
    CreateReserve --> PayMethod{"Payment method"}

    PayMethod -- "COD" --> AwaitFulfill["Await fulfillment"]
    PayMethod -- "Online" --> ProcessOnline["Process online payment"]
    ProcessOnline --> PaySuccess{"Payment successful?"}
    PaySuccess -- "Yes" --> AwaitFulfill
    PaySuccess -- "No" --> RecordFail["Record failed attempt"]
    RecordFail --> RetryAllow{"Retry allowed?"}
    RetryAllow -- "Yes" --> ProcessOnline
    RetryAllow -- "No" --> CancelExpire["Cancel or expire order"]
    CancelExpire --> EndNode(["End"])

    AwaitFulfill --> PrepGoods["Warehouse prepares goods"]
    PrepGoods --> CreateShip["Create shipment"]
    CreateShip --> Deliver["Deliver order"]
    Deliver --> DeliverSuccess{"Delivery successful?"}
    DeliverSuccess -- "No" --> HandleEx["Handle delivery exception"]
    HandleEx --> EndNode
    DeliverSuccess -- "Yes" --> CompleteOrder["Complete order"]
    CompleteOrder --> EndNode
~~~

---

## 5.2. Quy trình quản lý biến động tồn kho (Inventory Event & Accounting Process)

Mọi sự kiện nghiệp vụ tác động đến hàng hóa trong kho đều phải đi qua quy trình chuẩn hóa dưới đây để cập nhật đúng chỉ số `OnHand`, `Reserved`, tính toán lại tồn khả dụng và ghi nhận bản ghi `Stock Movement` trong cùng một Transaction.

~~~mermaid
flowchart TD
    Event(["Inventory Event"]) --> EventType{"Event Type"}

    EventType -- "Purchase Receipt" --> IncOnHand1["Increase OnHand"]
    EventType -- "Customer Reservation" --> IncReserved["Increase Reserved"]
    EventType -- "Release Reservation" --> DecReserved["Decrease Reserved"]
    EventType -- "Shipment" --> DecBoth["Decrease OnHand and Reserved"]
    EventType -- "Return Accepted" --> IncOnHand2["Increase OnHand"]
    EventType -- "Stock Transfer" --> Transfer["Transfer between warehouses"]
    EventType -- "Stock Adjustment" --> Adjust["Record inventory adjustment"]

    IncOnHand1 --> CreateMove["Create Stock Movement"]
    IncReserved --> CreateMove
    DecReserved --> CreateMove
    DecBoth --> CreateMove
    IncOnHand2 --> CreateMove
    Transfer --> CreateMove
    Adjust --> CreateMove

    CreateMove --> Recalc["Recalculate available stock"]
    Recalc --> CommitTx["Commit transaction"]
    CommitTx --> EndNode(["End"])
~~~

---

## 5.3. Các quy trình nghiệp vụ chuyên biệt cần lưu ý khi sang M2

Ngoài hai sơ đồ tổng thể trên, khi tiến hành xác định thực thể, quan hệ và ràng buộc ở M2, cần đối chiếu chặt chẽ với các quy trình chi tiết sau:
* Quy trình nhập hàng từ nhà cung cấp (`Purchase Order` $\rightarrow$ `Stock Receipt`).
* Quy trình chuyển kho nội bộ 2 pha (`Source Warehouse Out` $\rightarrow$ `In-Transit` $\rightarrow$ `Destination Warehouse In`).
* Quy trình xử lý đơn hàng và chia đợt giao hàng (`Order Fulfillment & Split Shipment`).
* Quy trình đối soát thanh toán và xử lý Webhook/Callback bất đồng bộ.
* Quy trình hủy đơn hàng / hết hạn giữ hàng và giải phóng tồn kho (`Release Inventory`).
* Quy trình đổi trả, kiểm định chất lượng hàng hoàn và hoàn tiền (`Return Inspection & Refund`).