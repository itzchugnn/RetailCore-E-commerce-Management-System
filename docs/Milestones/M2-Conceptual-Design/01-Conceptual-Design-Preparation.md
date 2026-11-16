# M2.01 - KICK-OFF & CONCEPTUAL DATABASE DESIGN PREPARATION

* **Milestone:** M2 – Conceptual Database Design (Thiết kế CSDL Quan niệm)
* **Phiên bản:** 1.0
* **Trạng thái:** In Progress
* **Đầu vào kế thừa:** 7 Quyết định nghiệp vụ đã chốt (BD-01 đến BD-07) & Bộ Business Rules v1.1

---

## 1. Mục tiêu Milestone M2
Milestone M2 thực hiện chuyển hóa toàn bộ các yêu cầu nghiệp vụ đã xác nhận ở M1 thành **Mô hình Dữ liệu Khái niệm (Conceptual Data Model - ERD/EER)**.

M2 tập trung giải quyết 6 câu hỏi bản chất:
1. Hệ thống cần quản lý những loại thực thể nghiệp vụ nào?
2. Mỗi thực thể có ý nghĩa gì trong thế giới thực tế của doanh nghiệp bán lẻ?
3. Các thực thể liên hệ với nhau như thế nào?
4. Bản số quan hệ (Cardinality & Participation Constraints) là bao nhiêu ($1-1, 1-N, N-N$, Toàn phần hay Một phần)?
5. Thực thể nào cần là Thực thể Kết hợp (Associative Entity), Thực thể Yếu (Weak Entity) hay Kế thừa/Chuyên biệt hóa (Generalization / Specialization)?
6. Những ràng buộc nghiệp vụ nào không thể hiện đầy đủ bằng Cardinality?

> **Lưu ý phương pháp luận:** M2 **chưa phải** giai đoạn xác định kiểu dữ liệu SQL Server, Index, Stored Procedure hay tối ưu truy vấn. Những nội dung đó thuộc phạm vi của M3 (Logical Design) và M4 (Physical Design).

---

## 2. Danh mục Thực thể Ứng viên theo 14 Miền Chủ đề (Candidate Entities)

| Miền chủ đề (Subject Area) | Thực thể ứng viên | Ý nghĩa & Vai trò nghiệp vụ |
| :--- | :--- | :--- |
| **1. Identity & Access** | `User`, `Role`, `Permission`, `UserRole`, `RolePermission` | Người dùng nội bộ, khách hàng và cấu trúc phân quyền RBAC. |
| **2. Product Catalog** | `Product`, `Category`, `ProductCategory`, `ProductVariant`, `Attribute`, `AttributeValue`, `ProductAttribute`, `VariantAttributeValue` | Sản phẩm, cây danh mục đa cấp, SKU biến thể và cấu trúc thuộc tính động. |
| **3. Warehouse** | `Warehouse`, `InventoryBalance`, `StockMovement` | Kho hàng vật lý, số dư tồn kho thực tế và nhật ký biến động kho bất biến. |
| **4. Inventory Reservation** | `StockReservation` | Cơ chế giữ tồn kho tạm thời có thời hạn cho đơn hàng checkout. |
| **5. Procurement** | `Supplier`, `PurchaseOrder`, `PurchaseOrderLine`, `GoodsReceipt`, `GoodsReceiptLine` | Nhà cung cấp, đơn đặt mua hàng và quy trình tiếp nhận nhập kho. |
| **6. Customer** | `Customer`, `CustomerAddress` | Hồ sơ khách hàng và sổ địa chỉ nhận hàng cá nhân. |
| **7. Shopping** | `Cart`, `CartLine` | Giỏ hàng mua sắm (hỗ trợ lưu tạm thời trước khi checkout). |
| **8. Sales** | `Order`, `OrderLine`, `OrderStatusHistory` | Đơn hàng thương mại, chi tiết dòng hàng và lịch sử chuyển đổi trạng thái. |
| **9. Fulfillment** | `OrderAllocation`, `Shipment`, `ShipmentLine`, `DeliveryReturnReceipt` | Phân bổ đơn hàng cho nhiều kho, kiện hàng giao vận và xử lý hàng giao thất bại. |
| **10. Promotion** | `Promotion`, `PromotionRule`, `Voucher`, `VoucherUsage`, `OrderPromotion` | Chiến dịch khuyến mãi, mã voucher và ma trận lưu vết chiết khấu phân tầng. |
| **11. Payment** | `Payment`, `PaymentAttempt`, `Refund` | Các giao dịch thanh toán trực tuyến, đối soát COD và lịch sử hoàn tiền. |
| **12. Shipping** | `ShippingProvider`, `ShipmentTrackingHistory` | Đơn vị vận chuyển liên kết và nhật ký hành trình vận đơn. |
| **13. Return** | `ReturnRequest`, `ReturnLine`, `ReturnStatusHistory` | Yêu cầu đổi trả sau mua, thẩm định chất lượng và lịch sử xử lý khiếu nại. |
| **14. Audit & Reporting** | `AuditLog` | Nhật ký kiểm toán thao tác hệ thống phục vụ an toàn thông tin. |

---

## 3. Phân tích 6 Nhóm Quan hệ Trọng yếu trong M2

### 3.1. Product – Category
* Quan hệ: $N-N$.
* Thực thể kết hợp: `ProductCategory` sở hữu thuộc tính `IsPrimary`.
* Ràng buộc: Mỗi sản phẩm có thể tham gia nhiều danh mục nhưng chỉ có duy nhất một danh mục mang `IsPrimary = 1`.

### 3.2. Product – Variant – Attribute
* `Product` 1 — $N$ `ProductVariant`.
* `Product` định nghĩa các thuộc tính áp dụng thông qua thực thể kết hợp `ProductAttribute`.
* `Attribute` 1 — $N$ `AttributeValue`.
* `ProductVariant` $N — N$ `AttributeValue` thông qua `VariantAttributeValue`. Tổ hợp giá trị thuộc tính phải duy nhất trong phạm vi sản phẩm đó.

### 3.3. Order – Warehouse – Shipment (Xử lý Split Shipment)
* `Order` 1 — $N$ `OrderLine`.
* Mỗi `OrderLine` được chia thành một hoặc nhiều `OrderAllocation` tương ứng với các kho tiếp nhận.
* `Warehouse` 1 — $N$ `Shipment`.
* `Order` 1 — $N$ `Shipment`.
* `Shipment` 1 — $N$ `ShipmentLine`. Mỗi `ShipmentLine` tiêu thụ số lượng từ đúng `OrderAllocation` của kho đó.

### 3.4. Order – StockReservation
* Một đơn hàng phát sinh nhiều `StockReservation`.
* Mỗi `StockReservation` chỉ định rõ SKU, kho và số lượng giữ kèm mốc `ExpiresAt` (15 phút).
* Lượt giữ kho gắn với vòng đời xử lý đơn: Tiêu thụ thành xuất kho (`FULFILLED`) hoặc giải phóng lại tồn khả dụng (`RELEASED`).

### 3.5. Shipment – Delivery Return
* Một `Shipment` có thể trải qua nhiều lần thử giao hàng (`ShipmentTrackingHistory`).
* Khi giao thất bại và hàng thực tế quay về kho, kho lập `DeliveryReturnReceipt` để ghi nhận số lượng thực tế nhận lại. Tuyệt đối không tự động cộng lại tồn kho khi chưa có phiếu nhập kiểm tra.

### 3.6. Order – Promotion
* Đơn hàng áp dụng tối đa 1 khuyến mãi cấp sản phẩm (trừ vào `OrderLine`) và 1 voucher cấp đơn hàng (nếu `IsStackable = 1`).
* Thực thể `OrderPromotion` lưu snapshot chiết khấu và thứ tự áp dụng để đối soát tài chính bất biến.

---

## 4. Giải quyết 10 Câu hỏi Thiết kế Trọng tâm (M2-Q01 đến M2-Q10)

| ID | Câu hỏi thiết kế | Phân tích & Đề xuất giải pháp của Database Engineer |
| :---: | :--- | :--- |
| **M2-Q01** | `ProductVariant` có phải là thực thể độc lập không? | **Có.** `ProductVariant` (SKU) có surrogate key riêng (`VariantID`), mang mã định danh nghiệp vụ duy nhất toàn hệ thống (`SKU`). Mọi quan hệ về giá bán, tồn kho, giữ hàng và dòng đơn hàng đều gắn trực tiếp với `ProductVariant`, không gắn với `Product`. |
| **M2-Q02** | `ProductAttribute` có cần tách khỏi `Attribute` không? | **Cần tách.** `Attribute` là từ điển thuộc tính toàn cục (Color, Size, Material). `ProductAttribute` xác định sản phẩm cụ thể áp dụng thuộc tính nào. Điều này ngăn chặn việc gán sai ngữ nghĩa (ví dụ: gán Size quần áo cho Đèn bàn). |
| **M2-Q03** | `OrderAllocation` có cần tồn tại độc lập không? | **Bắt buộc có.** Khi 1 dòng đơn hàng (`OrderLine`) đặt số lượng lớn (ví dụ: 5 cái) nhưng phải lấy từ 2 kho khác nhau (Kho A lấy 3, Kho B lấy 2), `OrderAllocation` chính là cầu nối trung gian đại diện cho cam kết cung ứng của từng kho. |
| **M2-Q04** | `StockReservation` liên hệ với `OrderLine` hay `OrderAllocation`? | **Liên hệ với `OrderAllocation`** (hoặc cặp `(OrderLine, Warehouse)`). Tồn kho vật lý luôn gắn với một kho cụ thể, do đó lượt giữ hàng phải gắn đích danh vào kho đã phân bổ. |
| **M2-Q05** | `ShipmentLine` liên hệ với `OrderLine` hay `OrderAllocation`? | **Liên hệ với `OrderAllocation`.** Mỗi `Shipment` xuất phát từ một kho xác định và sẽ tiêu thụ trực tiếp lượng hàng đã phân bổ và giữ ở kho đó. |
| **M2-Q06** | `StockMovement` có cần tham chiếu chứng từ nguồn không? | **Bắt buộc có.** `StockMovement` phải lưu `ReferenceType` (`GOODS_RECEIPT`, `SHIPMENT_OUT`, `TRANSFER`, `STOCK_ADJUST`, `DELIVERY_RETURN_IN`) và `ReferenceID` để đảm bảo nguyên tắc đối soát kế toán kho bất biến. |
| **M2-Q07** | `Promotion` có cần chuyên biệt hóa theo tầng không? | **Có.** Sử dụng mô hình cha - con (Generalization): Bảng cha `Promotion` quản lý thời gian, trạng thái, phân tầng (`PRODUCT_LEVEL` vs `ORDER_LEVEL`); Bảng con `Voucher` quản lý mã code, hạn mức dùng, điều kiện tối thiểu và cờ `IsStackable`. |
| **M2-Q08** | `ReturnRequest` có liên hệ với `Shipment` không? | **Không bắt buộc gắn cứng.** Giao thất bại được xử lý qua `Shipment` và `DeliveryReturnReceipt`. Còn `ReturnRequest` phục vụ nghiệp vụ khách hàng khiếu nại đổi trả sau khi đã nhận hàng thành công (liên kết với `OrderLine`). |
| **M2-Q09** | `CustomerAddress` có cần snapshot trong `Order` không? | **Bắt buộc snapshot 100%.** Thông tin người nhận (Tên, SĐT, Địa chỉ, Phường/Xã, Quận/Huyện, Tỉnh/Thành) phải được sao chép nguyên văn vào `Order` tại thời điểm đặt hàng. Việc khách sửa sổ địa chỉ sau này không được làm thay đổi đơn hàng cũ. |
| **M2-Q10** | `Payment` và `Order` có quan hệ 1–1 hay 1–N? | **Quan hệ 1–N.** Một đơn hàng có thể có nhiều lần thử thanh toán (`PaymentAttempt` thất bại rồi thử lại) và có thể phát sinh nhiều lần thanh toán/hoàn tiền (`Refund`). |

---

## 5. Tiêu chí Hoàn thành Milestone M2 (Definition of Done)

- [ ] Xác định và mô tả đầy đủ các thực thể nghiệp vụ (Strong, Weak, Associative Entities).
- [ ] Xác định định danh nghiệp vụ và thuộc tính của từng thực thể.
- [ ] Xác định quan hệ giữa các thực thể và bản số (Cardinality & Participation Constraints: $1-1, 1-N, N-N$, Total/Partial).
- [ ] Mô hình hóa các quan hệ kế thừa/chuyên biệt hóa (Generalization / Specialization) cho Promotion và Identity.
- [ ] Mô hình hóa chính xác 7 quyết định nghiệp vụ (BD-01 đến BD-07).
- [ ] Hoàn thành sơ đồ Conceptual ERD/EER trực quan (Mermaid / PlantUML / DBML).
- [ ] Đối soát chéo ERD với toàn bộ Use Case (M1), Business Rules (BR v1.1) và Data Requirements (DR v1.1).
- [ ] Lập biên bản giải trình các quyết định thiết kế CSDL quan niệm.

