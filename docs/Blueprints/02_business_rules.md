# PHẦN III. BỘ QUY TẮC NGHIỆP VỤ (BUSINESS RULES – VERSION 1.0)

Bộ quy tắc nghiệp vụ dưới đây đóng vai trò là căn cứ pháp lý nội bộ của hệ thống để kiểm tra tính đúng đắn của sơ đồ ERD, lược đồ quan hệ (Relational Schema), các ràng buộc toàn vẹn (Constraints) trong SQL Server cũng như logic xử lý ở tầng Application.

---

## 3.1. Nhóm Sản phẩm (Product Catalog Rules)

| ID | Quy tắc nghiệp vụ (Business Rule) | Định hướng kiểm soát dự kiến |
| :--- | :--- | :--- |
| **BR-P01** | Một sản phẩm thuộc ít nhất một danh mục kinh doanh. | Application Logic & Bảng liên kết `ProductCategories` |
| **BR-P02** | Một sản phẩm có thể có nhiều SKU (biến thể). | Quan hệ $1 - N$ giữa `Products` và `ProductVariants` |
| **BR-P03** | Mỗi SKU thuộc duy nhất một sản phẩm. | `ProductID NOT NULL` (FK) trong bảng `ProductVariants` |
| **BR-P04** | SKU phải duy nhất trong toàn hệ thống. | `UNIQUE Constraint` trên cột `SKU` |
| **BR-P05** | Mỗi SKU có thể được lưu trữ tại nhiều kho. | Quan hệ $N - N$ qua bảng `Inventory (WarehouseID, VariantID)` |
| **BR-P06** | Giá bán không được âm ($\text{Price} \ge 0$). | `CHECK Constraint (Price >= 0)` |
| **BR-P07** | Không được xóa vật lý sản phẩm đã phát sinh giao dịch; chỉ được chuyển trạng thái ngừng kinh doanh. | Soft Delete / Status Flag + `FOREIGN KEY (NO ACTION)` |

---

## 3.2. Nhóm Kho hàng (Warehouse & Inventory Rules)

| ID | Quy tắc nghiệp vụ (Business Rule) | Định hướng kiểm soát dự kiến |
| :--- | :--- | :--- |
| **BR-I01** | Một doanh nghiệp có thể có nhiều kho hàng. | Bảng `Warehouses` độc lập |
| **BR-I02** | Mỗi kho quản lý nhiều SKU. | Quan hệ $1 - N$ từ `Warehouses` sang `Inventory` |
| **BR-I03** | Tồn kho được quản lý định danh theo cặp `(Kho – SKU)`. | `PRIMARY KEY (WarehouseID, VariantID)` trên bảng `Inventory` |
| **BR-I04** | Công thức tính tồn khả dụng: $\text{Tồn khả dụng} = \text{Tồn thực tế} - \text{Tồn đang giữ}$. | Computed Column `AvailableQty AS (OnHandQty - ReservedQty) PERSISTED` |
| **BR-I05** | Không cho phép tồn kho âm ($\text{OnHandQty} \ge 0$, $\text{AvailableQty} \ge 0$). | `CHECK Constraint` trên `OnHandQty` và `ReservedQty <= OnHandQty` |
| **BR-I06** | Mọi biến động tồn kho (nhập, xuất, điều chỉnh) đều phải được ghi nhận lịch sử. | Bảng `StockMovements` bắt buộc ghi trong cùng Transaction cập nhật `Inventory` |
| **BR-I07** | Phiếu nhập/xuất đã xác nhận hoàn tất không được phép sửa trực tiếp; mọi điều chỉnh phải thông qua giao dịch bù trừ mới. | Kiểm soát trạng thái phiếu bằng Stored Procedure / Trigger |

---

## 3.3. Nhóm Đơn hàng (Sales Order Rules)

| ID | Quy tắc nghiệp vụ (Business Rule) | Định hướng kiểm soát dự kiến |
| :--- | :--- | :--- |
| **BR-O01** | Một khách hàng có thể có nhiều đơn hàng. | Quan hệ $1 - N$ giữa `Customers` và `Orders` |
| **BR-O02** | Một đơn hàng phải có ít nhất một dòng chi tiết sản phẩm. | Kiểm soát trong Transaction tạo đơn (`usp_CreateOrder`) |
| **BR-O03** | Một chi tiết đơn hàng phải tham chiếu chính xác đến một SKU (`VariantID`). | `VariantID NOT NULL` (FK) trong bảng `OrderDetails` |
| **BR-O04** | Đơn hàng bắt buộc lưu lại giá bán tại thời điểm đặt hàng (Snapshot). | Cột `UnitPrice`, `ProductNameSnapshot`, `SKUSnapshot` trong `OrderDetails` |
| **BR-O05** | Đơn hàng bắt buộc lưu lại thông tin giao hàng tại thời điểm đặt. | Bảng `OrderAddresses` độc lập với sổ địa chỉ hiện tại của khách |
| **BR-O06** | Không được xác nhận/đặt đơn vượt quá số lượng tồn khả dụng tại thời điểm giữ hàng. | Kiểm soát đồng thời (Concurrency Locking) + `CHECK Constraint` |
| **BR-O07** | Đơn hàng đã ở trạng thái hoàn tất (Completed) hoặc đã hủy (Cancelled) không được chuyển ngược về trạng thái đang xử lý. | State Machine validation trong Stored Procedure / Service Layer |
| **BR-O08** | Mỗi lần thay đổi trạng thái quan trọng của đơn hàng phải có bản ghi lịch sử. | Bảng `OrderStatusHistory` lưu `OldStatus`, `NewStatus`, `ChangedAt`, `ChangedBy` |

---

## 3.4. Nhóm Khuyến mãi (Promotion & Voucher Rules)

| ID | Quy tắc nghiệp vụ (Business Rule) | Định hướng kiểm soát dự kiến |
| :--- | :--- | :--- |
| **BR-PM01** | Một chương trình khuyến mãi phải có thời gian bắt đầu (`StartDate`) và kết thúc (`EndDate`) hợp lệ ($\text{EndDate} > \text{StartDate}$). | `CHECK Constraint (EndDate > StartDate)` |
| **BR-PM02** | Một chương trình khuyến mãi có thể áp dụng cho nhiều sản phẩm hoặc nhiều SKU. | Bảng `PromotionProducts` |
| **BR-PM03** | Một đơn hàng có thể sử dụng voucher nếu thỏa mãn đầy đủ điều kiện của chương trình. | Validation logic tại thời điểm tính giá & tạo đơn |
| **BR-PM04** | Mỗi Voucher có giới hạn tổng số lượt sử dụng (`UsageLimit`) và số lượt đã dùng (`UsedCount`). | `CHECK Constraint (UsedCount <= UsageLimit)` |
| **BR-PM05** | Không được sử dụng voucher đã hết hạn hoặc đã hết lượt sử dụng. | Kiểm tra đồng thời với khóa `UPDLOCK` khi áp dụng voucher vào đơn |
| **BR-PM06** | Phải xác định rõ quy tắc kết hợp (Stacking/Exclusivity) giữa các chương trình khuyến mãi. | Thuộc tính `Priority`, `IsStackable` và quy tắc giải quyết xung đột khuyến mãi |

> **Điểm cần nghiên cứu sâu (BR-PM06):**
> Không nên mặc định mọi khuyến mãi đều được cộng dồn hoặc mọi khuyến mãi đều loại trừ nhau tuyệt đối. Trong giai đoạn M1 và M2, cần định nghĩa rõ ma trận kết hợp:
> * **Mức độ ưu tiên (`Priority`):** Chương trình nào được tính trước?
> * **Khả năng kết hợp (`IsStackable`):** Khuyến mãi giảm giá trực tiếp trên SKU có được dùng chung với Voucher giảm giá toàn đơn hàng hay không?
> * **Phạm vi áp dụng:** Áp dụng trên giá gốc hay giá sau khi đã trừ khuyến mãi trước đó?

---

## 3.5. Nhóm Thanh toán (Payment & Refund Rules)

| ID | Quy tắc nghiệp vụ (Business Rule) | Định hướng kiểm soát dự kiến |
| :--- | :--- | :--- |
| **BR-PA01** | Một đơn hàng có thể có nhiều lần thử thanh toán (Payment Attempts) nếu các lần trước thất bại hoặc thanh toán từng phần. | Quan hệ $1 - N$ giữa `Orders` và `Payments` |
| **BR-PA02** | Tổng số tiền thu thành công của một đơn hàng không được vượt quá tổng số tiền phải thanh toán của đơn hàng đó. | Kiểm tra nghiệp vụ trước và sau khi ghi nhận trạng thái `Payments` |
| **BR-PA03** | Mọi giao dịch thanh toán thành công đều phải được ghi nhận đầy đủ vào hệ thống. | Bảng `Payments` với lịch sử thời gian và trạng thái bất biến |
| **BR-PA04** | Giao dịch thanh toán phải có mã tham chiếu (`TransactionReference`) duy nhất theo từng nhà cung cấp cổng thanh toán. | `Filtered Unique Index` trên `(PaymentMethodID, TransactionReference) WHERE TransactionReference IS NOT NULL` |
| **BR-PA05** | Mọi giao dịch hoàn tiền (`Refund`) bắt buộc phải tham chiếu đến một giao dịch thanh toán gốc đã thành công. | `PaymentID NOT NULL` (FK) trong bảng `Refunds` và tổng tiền hoàn $\le$ số tiền đã thanh toán |