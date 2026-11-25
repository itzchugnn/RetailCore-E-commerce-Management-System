# M1.11 - Business Decisions Register (Chính thức hóa BD-01 đến BD-07)

* **Milestone:** M1 – Business Analysis
* **Phiên bản:** 2.0 – Canonical SSOT Approved Business Decisions
* **Database Target:** Microsoft SQL Server 2022+
* **Trạng thái:** Đã phê duyệt chính thức (Approved & Reverse-Synced `CORR-01..10`)

---

## 1. Bảng Tổng hợp 7 Quyết định Nghiệp vụ Chính thức

| ID | Quyết định | Kết luận chính thức | Tác động trực tiếp lên Mô hình CSDL (M2–M4) |
| :---: | :--- | :--- | :--- |
| **BD-01** | **Sản phẩm – Danh mục** | Một sản phẩm có thể thuộc nhiều danh mục ($N-N$). Có một danh mục chính. | Thực thể liên kết `ProductCategory(ProductID, CategoryID, IsPrimary)`. Hỗ trợ Breadcrumb và báo cáo doanh thu không trùng lặp. |
| **BD-02** | **Guest Checkout** | Bắt buộc đăng nhập trước khi đặt hàng. Không hỗ trợ Guest Checkout. | `SalesOrder.CustomerID` bắt buộc `NOT NULL` (tham chiếu `Customer.AccountID`). Quản lý chuẩn `PromotionUsage`, chính sách đổi trả và hồ sơ khách hàng. |
| **BD-03** | **Split Shipment** | Một đơn hàng có thể được xử lý và giao từ nhiều kho. | Tách rời `SalesOrder` khỏi `WarehouseID`. `OrderLine` được chia thành các `OrderWarehouseAllocation` (E12), xuất kho qua nhiều `Shipment` (E45). |
| **BD-04** | **Reservation** | Giữ hàng ngay khi tạo đơn, có thời hạn `ExpiresAt` (15 phút chuẩn; 5 phút cho đơn Flash Sale). | Thực thể `StockReservation` (E13) và `StockReservationLine` (E14) gắn với `AllocationID` ($1:0..N$), kết hợp hạn mức `PromotionProduct` (E51) chống bán âm kho. |
| **BD-05** | **Promotion Stacking** | Phân tách 3 tầng ưu đãi độc lập (`PRODUCT`, `ORDER`, `SHIPPING`); cho phép kết hợp khác tầng khi `IsStackable = 1`. | Cấm áp dụng quá 1 chương trình trong cùng tầng; lưu vết tại `PromotionUsage` (E53), phân bổ LRM xuống `OrderLine` và lưu giảm ship tại `SalesOrder.ShippingDiscountAmount`. |
| **BD-06** | **Variant Attributes** | Sử dụng mô hình thuộc tính linh hoạt theo từng sản phẩm (Dynamic Attributes). | Cụm 5 thực thể chuẩn hóa `Attribute`, `AttributeValue`, `ProductAttribute`, `ProductAttributeValue`, `VariantAttributeValue`. Không hardcode cột Size/Color. |
| **BD-07** | **Inventory on Delivery & Failed Delivery** | Khi xuất kho, giảm đồng thời tồn vật lý (`OnHandQty`) và lượng giữ (`ReservedQty`). Hàng giao thất bại chỉ được nhập lại kho khi thực sự quay về. | Phân định 3 phẩm chất tồn kho vật lý (`SELLABLE`, `QUARANTINED`, `DAMAGED`) và ký biên bản `DeliveryReturnReceipt` (E75) khi thực nhận hoàn kho. |

---

## 2. Chi tiết 2 Quyết định Trọng tâm: BD-05 & BD-07

### 2.1. BD-05: Chính sách Khuyến mãi Phân tầng 3 Lớp & Điều kiện Cộng dồn (Promotion Stacking)

#### Ba tầng khuyến mãi độc lập (`Promotion.PromotionScope`):
1. **Tầng 1 – Product Promotion (`'PRODUCT'` – Khuyến mãi cấp SKU / Dòng hàng):**
   * Áp dụng trực tiếp trên từng dòng sản phẩm (`OrderLine`) thông qua danh sách `PromotionProduct` (`E51`, hỗ trợ giới hạn chiến dịch `FlashSaleStockLimit` và `MaxQtyPerCustomer`).
   * **Giới hạn:** Tối đa **1 chương trình** giảm giá trực tiếp trên mỗi dòng `OrderLine`.
2. **Tầng 2 – Order Voucher (`'ORDER'` – Voucher giảm giá tiền hàng cấp đơn):**
   * Áp dụng trên tổng giá trị tiền hàng đủ điều kiện sau khi đã trừ chiết khấu Tầng 1 ($\sum \text{EligibleBase}$).
   * **Giới hạn:** Tối đa **1 mã voucher** cấp `'ORDER'` trên mỗi `SalesOrder`.
3. **Tầng 3 – Shipping Voucher (`'SHIPPING'` – Voucher Freeship miễn/giảm phí vận chuyển):**
   * Áp dụng trực tiếp vào phí vận chuyển thu khách của đơn hàng (`SalesOrder.ShippingDiscountAmount <= SalesOrder.ShippingFee`), không làm méo giá trị hàng hóa hay thuế GTGT dòng hàng.
   * **Giới hạn:** Tối đa **1 mã voucher** cấp `'SHIPPING'` trên mỗi `SalesOrder`.

#### Quy tắc cộng dồn (Stacking Rules):
* Các chương trình thuộc **khác tầng** chỉ được kết hợp đồng thời khi cờ `IsStackable = 1` cho phép.
* **Quy tắc chặn tuyệt đối:** Tuyệt đối cấm áp dụng quá 1 chương trình trong cùng một tầng (không nhập 2 voucher tiền hàng, không nhập 2 mã Freeship, không áp 2 chương trình giảm giá SKU trên cùng 1 dòng đơn hàng).

#### Lưu vết đối soát tài chính chuẩn hóa (`PromotionUsage`, `OrderLine` & `SalesOrder`):
* `OrderLine.UnitPrice`: Đơn giá bán lẻ niêm yết (đã gồm GTGT) snapshot tại thời điểm đặt.
* `OrderLine.LineProductDiscount`: Số tiền giảm trực tiếp từ Tầng 1 (`'PRODUCT'`).
* `SalesOrder.DiscountAmount` & `OrderLine.AllocatedOrderDiscount`: Tổng giảm giá hàng hóa và phần chiết khấu Tầng 2 (`'ORDER'`) phân bổ nguyên số VNĐ xuống từng dòng theo thuật toán **Largest Remainder Method (LRM)** (`CORR-03`).
* `SalesOrder.ShippingDiscountAmount`: Số tiền miễn/giảm phí vận chuyển từ Tầng 3 (`'SHIPPING'`).
* Thực thể sổ cái `PromotionUsage` (`E53`): Lưu toàn bộ danh sách chương trình/voucher đã áp dụng (`PromotionID`, `OrderID`, `OrderLineID`, `VoucherID`, `DiscountAmount`, `UsageStatus`) phục vụ đối soát kế toán bất biến.

---

### 2.2. BD-07: Chính sách Quản lý Tồn kho khi Vận chuyển và Giao thất bại

#### Quy ước bản chất về các trạng thái tồn kho:
* **Hàng đang giữ tại kho (`ReservedQty`):** Chỉ đại diện cho lượng hàng đang nằm trên giá kệ vật lý của một kho cụ thể, đã có cam kết cho đơn hàng nhưng **chưa xuất kho**.
* **Hàng đã xuất kho:** Khi hàng được đóng gói và bàn giao cho đơn vị vận chuyển, reservation tại kho được tiêu thụ và kết thúc.
* **Hàng đang trên đường giao (In-Transit):** Hàng đã rời kho vật lý thì **không còn thuộc `OnHandQty` của kho xuất**, và cũng **không được duy trì trong `ReservedQty`**. Cam kết cung ứng lúc này được theo dõi thông qua thực thể `Shipment`, `ShipmentLine` và trạng thái vận đơn.
* **Hàng giao thất bại:** Việc giao hàng thất bại **không đồng nghĩa hàng đã quay về kho**. Hàng chỉ được cộng lại vào tồn kho khi nhân viên kho thực sự nhận hàng, kiểm tra và ký xác nhận phiếu tiếp nhận.

#### Vòng đời 5 bước của tồn kho:
1. **Bước 1: Tạo đơn hàng thành công (Place Order / Hold)**
   * `OnHandQty` giữ nguyên; `ReservedQty` tăng theo số lượng đặt mua.
   * Công thức tồn khả dụng (`CORR-02`): $\text{AvailableQty} = (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}) - \text{ReservedQty}$.
2. **Bước 2: Kho đóng gói & Bàn giao vận chuyển (Shipment Handover – `SHIPMENT_OUT`)**
   * Hàng thực sự rời khỏi kho vật lý.
   * Giảm đồng thời cả `OnHandQty` và `ReservedQty` theo số lượng thực tế xuất.
   * Ghi một bản ghi vào `InventoryTransaction` / `InventoryTransactionLine` loại `SHIPMENT_OUT`.
3. **Bước 3: Giao hàng thất bại (Delivery Failed)**
   * Đơn vị vận chuyển báo giao thất bại (khách không nghe máy, sai địa chỉ) qua `DeliveryAttempt` (tối đa 3 lần, sau đó chuyển `RETURNED_TO_SENDER`).
   * **Tuyệt đối chưa tăng tồn kho**, vì hàng vẫn đang nằm trên xe hoặc tại bưu cục của đơn vị vận chuyển.
4. **Bước 4: Kho tiếp nhận hàng hoàn về (Delivery Return Receipt – `DELIVERY_RETURN_IN`)**
   * Hàng thực tế được xe chở quay trở lại kho xuất (hoặc kho chỉ định).
   * Nhân viên kho kiểm tra tình trạng hàng hóa, ký biên bản `DeliveryReturnReceipt` (E75).
   * Hệ thống ghi `InventoryTransaction` loại `DELIVERY_RETURN_IN` và tăng `OnHandQty` theo số lượng thực nhận lại.
5. **Bước 5: Giao lại hoặc Hủy đơn (Re-delivery or Order Cancellation)**
   * *Nếu giao lại:* Hệ thống tiến hành phân bổ kho (`OrderWarehouseAllocation`) và giữ hàng lại (`StockReservationLine`) theo quy tắc tồn kho hiện hành trước khi xuất kho lần 2.
   * *Nếu hủy đơn:* Kích hoạt quy trình hoàn tiền (`Refund`), giải phóng các nghĩa vụ liên quan và đóng đơn hàng.
---

## 3. Chính thức hóa Quyết định Thiết kế Subject Area 01: Product Catalog (PC-Q01 đến PC-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **PC-Q01** | Bắt buộc Danh mục chính khi Published? | **Bắt buộc đúng một danh mục chính (`IsPrimary = 1`) khi Published.** | Cài đặt `UNIQUE FILTERED INDEX (ProductID) WHERE IsPrimary = 1`. Cần thiết để render Breadcrumb, SEO và báo cáo doanh thu không trùng lặp (DR-02). |
| **PC-Q02** | Cấu trúc phân cấp Danh mục? | **Hỗ trợ nhiều cấp cha – con.** | Khóa ngoại đệ quy `Category.ParentCategoryID REFERENCES Category(CategoryID)` không tạo chu trình. |
| **PC-Q03** | Sản phẩm không có biến thể thuộc tính (Simple Product)? | **Tạo 1 SKU mặc định, tổ hợp thuộc tính rỗng.** | Mọi sản phẩm đủ điều kiện kinh doanh đều có đơn vị bán và quản lý tồn kho ở cấp SKU. CSDL không bị Polymorphic Foreign Key. |
| **PC-Q04** | Vai trò của thuộc tính khi gán cho sản phẩm? | **Một thuộc tính chỉ có duy nhất 1 vai trò: `DESCRIPTIVE` hoặc `VARIANT_DEFINING`.** | Cột `ProductAttribute.AttributeRole` phân định rạch ròi: mô tả chung (không sinh SKU) hoặc định nghĩa biến thể (sinh SKU). |
| **PC-Q05** | Dùng chung định nghĩa `Attribute`? | **Dùng chung từ điển `Attribute` toàn cục.** | Chuẩn hóa danh mục thuộc tính toàn hệ thống, phục vụ bộ lọc Faceted Filter trên Website. |
| **PC-Q06** | Bảo toàn SKU khi đổi thuộc tính? | **Bảo toàn tuyệt đối SKU và dữ liệu lịch sử.** | Không sửa đổi ngữ nghĩa hoặc xóa vật lý SKU đã phát sinh giao dịch. Áp dụng `IsActive = 0` (Soft Delete). |

---

## 4. Chính thức hóa Quyết định Thiết kế Subject Area 02: Warehouse & Inventory (WI-Q01 đến WI-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **WI-Q01** | Quản lý tồn kho theo vị trí kệ (`Location`)? | **Không, chỉ quản lý theo cấp Kho (`Warehouse`) ở phiên bản này.** | Giữ mô hình tinh gọn, sẵn sàng mở rộng sau mà không phá vỡ mô hình hiện tại. |
| **WI-Q02** | Giữ hàng một phần khi thiếu tồn kho? | **Không, áp dụng nguyên tắc All-or-nothing cho toàn bộ đơn hàng.** | Đơn hàng chỉ được xác nhận khi hệ thống giữ đủ 100% số lượng cần thiết cho toàn bộ đơn hàng. |
| **WI-Q03** | Phương thức phân bổ kho? | **Tự động phân bổ, cho phép nhân viên điều chỉnh thủ công.** | Tối ưu nguồn hàng và khoảng cách giao vận; hỗ trợ can thiệp khi kho gặp sự cố thực tế. |
| **WI-Q04** | Theo dõi hàng chuyển kho (`In-Transit`)? | **Có, theo dõi quy trình chuyển kho 2 bước độc lập.** | Bổ sung thực thể `WarehouseTransfer` và `WarehouseTransferLine`. Giảm `OnHand` kho nguồn $\to$ trạng thái `IN_TRANSIT` $\to$ tăng `OnHand` kho đích theo thực nhận. |
| **WI-Q05** | Phê duyệt phiếu kiểm kê trước khi cân chỉnh? | **Có, bắt buộc được cấp quản lý phê duyệt.** | Phiếu kiểm kê hoàn tất kiểm đếm (`COMPLETED`) không tự động sửa `InventoryBalance`. Chỉ sau khi phê duyệt mới sinh `InventoryTransaction`. |
| **WI-Q06** | Cơ chế phê duyệt điều chỉnh tồn kho? | **Có, áp dụng cơ chế phê duyệt với các phiếu điều chỉnh tồn kho.** | Mọi giao dịch điều chỉnh tăng/giảm tồn kho phải lưu vết `ApprovedBy`, `ApprovedAt` và lý do điều chỉnh (`AdjustmentReason`). |

---

## 5. Chính thức hóa Quyết định Thiết kế Subject Area 03: Procurement (PR-Q01 đến PR-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **PR-Q01** | Một PO có thể nhận hàng tại nhiều kho không? | **Không, một PO chỉ có đúng một kho nhận dự kiến.** | `PurchaseOrder.DestinationWarehouseID` xác định duy nhất kho nhận của PO; `GoodsReceipt.WarehouseID` bắt buộc trùng khớp. |
| **PR-Q02** | Cho phép nhận hàng nhiều đợt (Partial Receipt)? | **Có, cho phép nhận hàng nhiều đợt qua nhiều `GoodsReceipt`.** | Quan hệ `PurchaseOrder` $1:N$ `GoodsReceipt` và `PurchaseOrderLine` $1:N$ `GoodsReceiptLine`. |
| **PR-Q03** | Nhà cung cấp giao vượt số lượng đặt mua? | **Mặc định không cho phép nhận vượt số lượng đặt.** | Ràng buộc $\sum \text{GoodsReceiptLine.ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$. |
| **PR-Q04** | Hàng không đạt chất lượng (QC) khi tiếp nhận? | **Ghi nhận `RejectedQty` tại phiếu nhận và xử lý riêng.** | Phân tách $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$. Chỉ `AcceptedQty` mới tạo `PURCHASE_RECEIPT` và tăng `OnHandQty`. |
| **PR-Q05** | Có cần quản lý trả hàng cho nhà cung cấp? | **Có, quản lý qua cặp `SupplierReturn` và `SupplierReturnLine`.** | Trả hàng sau nhập kho sinh `SUPPLIER_RETURN_OUT` làm giảm `OnHandQty`. Dòng trả đối soát về `GoodsReceiptLineID` gốc. |
| **PR-Q06** | PO có bắt buộc được phê duyệt trước khi gửi NCC? | **Có, bắt buộc phê duyệt (`ApprovedBy`, `ApprovedAt`).** | Tuân thủ nguyên tắc phân tách trách nhiệm (SoD). Chỉ PO ở trạng thái `APPROVED` hoặc `SENT` mới có hiệu lực pháp lý. |

---

## 6. Chính thức hóa Quyết định Thiết kế Subject Area 04: Customer & Identity (CID-Q01 đến CID-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **CID-Q01** | Account có thể đồng thời là Customer và Employee? | **Không, phân tách loại trừ lẫn nhau (Disjoint Subtypes / Ràng buộc XOR).** | Ngăn chặn leo thang đặc quyền, cách ly tài khoản quản trị Back-office khỏi tài khoản mua sắm công cộng. |
| **CID-Q02** | Một Account có thể có nhiều Role không? | **Có, hỗ trợ nhiều Role qua thực thể kết hợp `AccountRole` ($N:N$).** | Phục vụ nhân sự kiêm nhiệm nhiều vị trí và phân hạng khách hàng VIP. |
| **CID-Q03** | Phương thức đăng nhập và định danh Account? | **Hỗ trợ Email hoặc Số điện thoại + Mật khẩu.** | Phù hợp thói quen mua sắm B2C di động. CSDL cài đặt Filtered Unique Index cho cả `Email` và `Phone`. |
| **CID-Q04** | Thông tin liên hệ bắt buộc khi đăng ký? | **Ít nhất một trong hai (Email hoặc Phone).** | Giảm rào cản chuyển đổi khi đăng ký. Ràng buộc `CHECK (Email IS NOT NULL OR Phone IS NOT NULL)`. Khi Checkout bắt buộc có SĐT nhận hàng. |
| **CID-Q05** | Có cần lưu lịch sử xác minh không? | **Có, lưu lịch sử bằng thực thể `AccountVerification`.** | Chống spam OTP (Rate Limiting), quản lý thời hạn `ExpiresAt`, phục vụ đổi SĐT/Email và điều tra an ninh. |
| **CID-Q06** | Quy tắc quản lý địa chỉ giao hàng mặc định? | **Tối đa 1 địa chỉ mặc định; địa chỉ đầu tiên tự động làm mặc định.** | Cài đặt `UNIQUE FILTERED INDEX (CustomerID) WHERE IsDefault = 1` trên SQL Server. Khi xóa địa chỉ mặc định, buộc chọn địa chỉ mới nếu còn. |

---

## 7. Chính thức hóa Quyết định Thiết kế Subject Area 05: Shopping (SHOP-Q01 đến SHOP-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **SHOP-Q01** | Số lượng giỏ hàng của một khách hàng? | **Một giỏ hàng ACTIVE, có thể có nhiều giỏ hàng lịch sử.** | Cài đặt `UNIQUE FILTERED INDEX (CustomerID) WHERE CartStatus = 'ACTIVE'`. Lưu vết lịch sử giỏ hàng chuyển đổi và bỏ rơi phục vụ Marketing. |
| **SHOP-Q02** | Xử lý khi thêm cùng một SKU nhiều lần? | **Cộng dồn số lượng (`Quantity`) vào dòng `CartItem` hiện có.** | Ràng buộc duy nhất `UNIQUE (CartID, VariantID)`. Tránh phân mảnh dòng sản phẩm trong giỏ hàng. |
| **SHOP-Q03** | Xử lý khi SKU hết hàng hoặc ngừng kinh doanh? | **Giữ `CartItem` và hiển thị cảnh báo; khóa nút Checkout.** | Khách hàng chủ động nắm được tình trạng sản phẩm; ngăn chặn tạo đơn hàng không thể đáp ứng giao vận. |
| **SHOP-Q04** | Cơ chế lưu trữ và tính toán giá bán? | **Chỉ lấy giá hiện hành từ SKU; Snapshot giá khi tạo đơn hàng.** | Không lưu giá cố định trong `CartItem`. Giá bán chỉ được chốt bất biến tại `OrderLine.UnitPrice` khi tạo đơn hàng (`BR-SH09`, `BR-SH10`, `BR-SA05`). |
| **SHOP-Q05** | Đồng bộ giỏ hàng trên nhiều thiết bị? | **Đồng bộ tập trung trên Database Server theo `CustomerID`.** | Giỏ hàng lưu trữ bền vững trên CSDL, đồng bộ liền mạch giữa Web, Mobile App và Tablet khi khách hàng đăng nhập. |
| **SHOP-Q06** | Trạng thái giỏ hàng sau khi chuyển đổi thành đơn hàng? | **Chuyển thành `CONVERTED`, đóng băng lịch sử và tạo giỏ mới khi mua tiếp.** | Khép kín vòng đời phễu chuyển đổi (Funnel Conversion Tracking), bảo toàn dữ liệu phục vụ đối soát và phân tích hành vi mua sắm. |

---

## 8. Chính thức hóa Quyết định Thiết kế Subject Area 06: Sales (SALES-Q01 đến SALES-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **SALES-Q01** | Thời điểm tạo đơn hàng và giữ hàng tồn kho? | **Tạo đơn và giữ toàn bộ hàng nguyên tử (Atomic); thất bại thì không tạo đơn chính thức.** | Tuân thủ BD-04 và WI-Q02 (All-or-nothing). Ngăn chặn tình trạng tạo đơn thiếu hàng. Giỏ hàng giữ nguyên `ACTIVE` nếu giữ hàng thất bại. |
| **SALES-Q02** | Vòng đời trạng thái đơn hàng (Order State Machine)? | **`PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `PARTIALLY_DELIVERED` $\to$ `DELIVERED` (có `PARTIALLY_CANCELLED` / `CANCELLED`, đóng hồ sơ hành chính qua `ClosedAt`).** | Trạng thái ban đầu của đơn trả trước là `PENDING_PAYMENT`. Đơn COD hợp lệ chuyển `CONFIRMED` ngay khi đặt (`CORR-06`). Kho chỉ đóng gói đơn `CONFIRMED`. |
| **SALES-Q03** | Chính sách hủy đơn hàng? | **Kết hợp: Khách tự hủy khi chưa đóng gói (`PENDING_PAYMENT`, `CONFIRMED`); sau đó cần nhân viên xử lý.** | Bảo vệ vận hành kho khỏi việc hủy đơn khi hàng đã đóng gói hoặc đã lên xe giao vận. Hủy đơn tự động kích hoạt nhả tồn kho giữ. |
| **SALES-Q04** | Thay đổi dòng hàng sau khi xác nhận đơn? | **Tuyệt đối không cho phép sửa trực tiếp `OrderLine` sau khi đơn đã xác nhận.** | Tránh vỡ thuật toán phân bổ kho, vỡ logic voucher khuyến mãi và tính toán lại công nợ. Khách muốn đổi món buộc phải hủy đơn cũ và đặt lại. |
| **SALES-Q05** | Mức độ snapshot thông tin sản phẩm trên `OrderLine`? | **Snapshot toàn bộ: SKU, tên sản phẩm, tên biến thể, đơn giá và `VariantAttributesSnapshot` (JSON).** | Bảo toàn 100% hợp đồng thương mại và hóa đơn theo luật thuế. Đơn hàng cũ hiển thị độc lập hoàn toàn với các biến động của Catalog sau này. |
| **SALES-Q06** | Quản lý địa chỉ giao hàng trong đơn? | **Mỗi đơn hàng có ĐÚNG MỘT địa chỉ giao hàng snapshot (`SalesOrder 1 ─── 1 OrderAddress`).** | Split shipment chỉ là chia nguồn xuất kho, toàn bộ kiện hàng đều giao về cùng 1 địa chỉ nhận của đơn. Độc lập với sổ địa chỉ `CustomerAddress`. |
| **SALES-Q07** | Chính sách lưu trữ đơn hàng và dòng hàng? | **Tuyệt đối không xóa vật lý (`No Hard Delete`) đơn hàng và dòng hàng đã phát sinh.** | Đơn hàng là chứng từ kế toán, pháp lý. Đơn hủy giữ nguyên trạng thái `CANCELLED` cùng nhật ký `OrderStatusHistory` phục vụ kiểm toán và báo cáo tỷ lệ hủy. |

---

## 9. Chính thức hóa Quyết định Thiết kế Subject Area 07: Fulfillment & Shipping (SHIP-Q01 đến SHIP-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **SHIP-Q01** | Số lượng kho xuất hàng cho một Shipment? | **Đúng một kho (`WarehouseID NOT NULL`).** | Tuân thủ BD-03 (Split Shipment theo kho). Mỗi kiện hàng vật lý có 1 mã vận đơn (`TrackingNumber`) xuất phát từ duy nhất 1 kho. Không gom hàng từ nhiều kho vào chung 1 Shipment. |
| **SHIP-Q02** | Thời điểm tạo lô giao hàng (Shipment)? | **Sau khi đơn được xác nhận (`CONFIRMED`) và kho chuẩn bị xuất hàng (`PROCESSING`).** | Không tạo Shipment khi đơn đang `PENDING_PAYMENT` hoặc đã bị hủy. Chỉ tạo khi kho bắt đầu nhặt hàng (Picking) để in Pick-list và Pack-slip. |
| **SHIP-Q03** | Cách chọn đơn vị vận chuyển (Carrier)? | **Hệ thống đề xuất dựa trên cấu hình, nhân viên/điều phối viên xác nhận.** | Tầng ứng dụng gợi ý Carrier tối ưu theo vùng miền, cước phí và SLA. CSDL lưu `Shipment.CarrierID` (FK), hỗ trợ linh hoạt cả tự động và điều chỉnh thủ công. |
| **SHIP-Q04** | Xử lý khi giao hàng thất bại? | **Cho phép giao lại tối đa $N$ lần (chuẩn là 3 lần); vượt giới hạn thì chuyển hoàn (`RETURNED_TO_SENDER`).** | Chuẩn hóa theo ngành bưu chính viễn thông Việt Nam. Ghi nhận chi tiết từng lần thử vào `DeliveryAttempt`. Vượt quá hạn mức tự động kích hoạt luồng chuyển hoàn. |
| **SHIP-Q05** | Lưu trữ bằng chứng giao hàng (Proof of Delivery)? | **Lưu người nhận, thời gian và ghi chú giao hàng; mở rộng ảnh/chữ ký sau.** | Mức khái niệm lưu `RecipientName`, `AttemptedAt`, `DeliveryNote` trong `DeliveryAttempt`. Mức Logical/Physical có thể mở rộng cột `ProofOfDeliveryURL` mà không vỡ schema. |
| **SHIP-Q06** | Cơ chế cập nhật trạng thái vận chuyển? | **Hỗ trợ cả thủ công và tích hợp tự động qua API/Webhook đối tác.** | `ShipmentStatusHistory` ghi nhận cả sự kiện webhook tự động từ Carrier API (GHTK/GHN) và các thao tác cập nhật của điều phối viên (`ChangedBy` ghi rõ nguồn cập nhật). |
| **SHIP-Q07** | Phí vận chuyển được quản lý như thế nào? | **`SalesOrder` lưu tổng phí tính cho khách; `Shipment` lưu phần phí phân bổ thực tế.** | Phân định ranh giới doanh thu vận chuyển (`SalesOrder.ShippingFee`) và chi phí logistics thực tế trả cho đối tác (`Shipment.ShippingFee`), phục vụ đối soát công nợ. |

---

## 10. Chính thức hóa Quyết định Thiết kế Subject Area 08: Promotion (PROMO-Q01 đến PROMO-Q08)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **PROMO-Q01** | Phạm vi ưu đãi cấp sản phẩm? | **Áp dụng theo SKU (`ProductVariant`) qua bảng liên kết `PromotionProduct`.** | Chuẩn hóa CSDL cao nhất, kiểm soát chính xác chiết khấu từng biến thể, tránh nhập nhằng định giá giữa các biến thể của sản phẩm. |
| **PROMO-Q02** | Biểu diễn điều kiện khuyến mãi? | **Sử dụng các loại điều kiện cấu hình sẵn (`ConditionType`), cấm biểu thức tự do.** | Tối ưu hóa tốc độ đánh giá bằng Index SQL Server, loại bỏ rủi ro bảo mật injection và nghẽn hiệu năng của bộ dynamic expression parsing. |
| **PROMO-Q03** | Cơ chế cấp voucher cấp đơn hàng? | **Hỗ trợ cả Voucher dùng chung toàn sàn và Voucher riêng cho từng khách hàng.** | CSDL linh hoạt: Voucher dùng chung để trống `AssignedCustomerID`; Voucher cá nhân gán `AssignedCustomerID` bắt buộc. |
| **PROMO-Q04** | Giới hạn lượt sử dụng voucher? | **Giới hạn ở cả cấp tổng chương trình (`MaxUsage`) và cấp mỗi khách hàng (`MaxUsagePerCustomer`).** | Ngăn chặn gian lận (Voucher Abuse) và tài khoản bot gom vét sạch mã ưu đãi của chương trình. |
| **PROMO-Q05** | Thời điểm ghi nhận lượt sử dụng? | **Chỉ ghi nhận chính thức (`CONSUMED`, tăng `UsedCount`) khi thanh toán thành công.** | Giữ chỗ tạm thời (`PENDING`) khi đơn ở `PENDING_PAYMENT` để tránh bán vượt mã. Chỉ chốt tiêu thụ khi đơn sang `CONFIRMED`. |
| **PROMO-Q06** | Xử lý voucher khi đơn hủy/hoàn? | **Hoàn lại lượt sử dụng (`UsageStatus = 'REVERSED'`) nếu voucher còn trong thời hạn.** | Bảo vệ quyền lợi khách hàng khi đơn bị hủy hợp lệ trước khi đóng gói hoặc hoàn trả toàn bộ. Hết hạn chương trình thì không hoàn lại. |
| **PROMO-Q07** | Chương trình dành riêng cho khách? | **Hỗ trợ qua thực thể kết hợp `PromotionTargetCustomer` ($N:N$).** | Phục vụ các chiến dịch chăm sóc khách hàng VIP, ưu đãi ngày sinh nhật, hoặc phát voucher đền bù dịch vụ cho từng tài khoản. |
| **PROMO-Q08** | Chính sách sửa chương trình đã dùng? | **Tuyệt đối không sửa thông tin ảnh hưởng ưu đãi; quản trị viên tạo phiên bản mới.** | Bảo toàn bất biến kiểm toán tài chính (Financial Auditability). CSDL quản lý qua `VersionNumber` và `SupersedesPromotionID`. |

---

## 11. Chính thức hóa Quyết định Thiết kế Subject Area 09: Payment & Refund (PAY-Q01 đến PAY-Q09)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **PAY-Q01** | Số lượng phương thức thanh toán trên một đơn? | **Một phương thức duy nhất cho mỗi đơn hàng.** | Đơn giản hóa đối soát công nợ, chuẩn hóa quan hệ $1:1$ giữa `SalesOrder` và `Payment`. Tránh chia nhỏ dòng tiền gây phân mảnh và phức tạp đối soát. |
| **PAY-Q02** | Có hỗ trợ phương thức COD không? | **Có, hỗ trợ song song COD và cổng thanh toán trực tuyến.** | Phù hợp thói quen TMĐT Việt Nam; phân định luồng thanh toán trước (Online) và thanh toán sau (COD). |
| **PAY-Q03** | Cơ chế thanh toán lại khi giao dịch thất bại? | **Cho phép thanh toán lại; mỗi lần thử tạo một `PaymentTransaction` mới.** | Đơn giữ `PENDING_PAYMENT` trong thời gian timeout, không bắt khách đặt lại đơn mới khi rớt mạng hoặc sai OTP. Giao dịch cũ lưu vết phục vụ đối soát. |
| **PAY-Q04** | Hỗ trợ hoàn tiền một phần (Partial Refund)? | **Có, hỗ trợ hoàn tiền toàn phần, một phần và hoàn nhiều lần.** | Ràng buộc bất biến: $\sum RefundAmount \le RemainingRefundable = AmountPaid - AmountRefunded$. Phục vụ đổi trả từng món trong đơn nhiều sản phẩm. |
| **PAY-Q05** | Thẩm quyền phê duyệt yêu cầu hoàn tiền? | **Nhân viên có thẩm quyền phê duyệt (`ApprovedBy NOT NULL`).** | Kiểm soát nội bộ (Internal Control), chống thất thoát dòng tiền ra khỏi doanh nghiệp. Không tự động hoàn tiền khi chưa thẩm định. |
| **PAY-Q06** | Quy trình hoàn tiền cho đơn hàng COD? | **Hoàn qua chuyển khoản ngân hàng (Bank Transfer).** | Thu thập thông tin STK khách hàng, thực hiện lệnh chi và lưu mã giao dịch đối soát tại `RefundTransaction`. |
| **PAY-Q07** | Lưu trữ phản hồi từ cổng thanh toán? | **Lưu chi tiết mã phản hồi, mã tham chiếu và thông điệp cổng.** | Làm bằng chứng pháp lý đối soát cuối ngày và giải quyết khiếu nại/tra soát (Chargeback). CSDL lưu `GatewayTransactionID`, `GatewayResponseCode`. |
| **PAY-Q08** | Điều kiện chuyển đơn sang `CONFIRMED`? | **Đơn online: khi thanh toán thành công; Đơn COD: khi duyệt hợp lệ.** | Đồng bộ triệt để với `SALES-Q02`. Đảm bảo kho chỉ xuất hàng khi tiền đã thu chắc chắn hoặc đơn COD tin cậy. Xác nhận đơn COD không đồng nghĩa với đã thu tiền. |
| **PAY-Q09** | Ghi nhận phí dịch vụ của cổng thanh toán? | **Có, lưu trường `TransactionFee` trên `PaymentTransaction`.** | Hỗ trợ đối soát số tiền thực nhận (Net Settlement) và tính toán chính xác biên lợi nhuận ròng của đơn hàng. |
| **PAY-Q10** | Đối soát công nợ thu hộ COD trên đơn đa kiện như thế nào? | **Tách nghĩa vụ khách hàng khỏi công nợ hãng vận chuyển qua mô hình COD 3 lớp (`CORR-07`).** | Bưu tá thu theo kiện ghi `CODCollection` (E77, cập nhật khách trả `PARTIALLY_PAID`/`PAID`), hãng chuyển trả bảng kê `CODRemittance` (E78) và kế toán khớp từng kiện qua `CODReconciliation` (E79). |

---

## 12. Chính thức hóa Quyết định Thiết kế Subject Area 10: Return & Exchange (RET-Q01 đến RET-Q08)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **RET-Q01** | Đối tượng được phép tạo yêu cầu trả hàng? | **Khách hàng tự tạo hoặc Nhân viên (CSKH) tạo thay khách hàng.** | CSDL lưu `CustomerID` (chủ đơn) và `CreatedByAccountID` (tài khoản người tạo), hỗ trợ tiếp nhận RMA đa kênh từ tổng đài. |
| **RET-Q02** | Thời hạn được phép gửi yêu cầu trả hàng? | **Cấu hình theo số ngày kể từ thời điểm giao thành công của từng kiện hàng (`Shipment.DeliveredAt`).** | Truy vết kiện giao qua `ReturnShipmentAllocation` (E76): `RequestedAt <= Shipment.DeliveredAt + PolicyReturnDays` (`CORR-10`). Quá hạn hệ thống khóa tạo RMA cho kiện đó. |
| **RET-Q03** | Có cho phép trả một phần số lượng dòng hàng? | **Có, cho phép trả một phần và trả nhiều lần cho cùng dòng hàng.** | Ràng buộc bất biến: $\sum ApprovedQty \le OrderedQty - CancelledQty - AlreadyReturnedQty$. Tính tiền hoàn nguyên số qua công thức hiệu lũy tiến $\Delta F = F(a+b) - F(a)$ kèm Clawback Voucher nếu lỗi khách (`CORR-03`). |
| **RET-Q04** | Quy trình phê duyệt yêu cầu trả hàng? | **Nhân viên CSKH/Kiểm duyệt phải phê duyệt trước khi khách gửi hàng.** | Ngăn chặn việc gửi trả hàng sai chính sách, lãng phí chi phí logistics ngược. Trạng thái bắt buộc: `PENDING_APPROVAL` $\to$ `APPROVED`. |
| **RET-Q05** | Định đoạt vật lý hàng trả về tại kho (`InventoryDisposition`)? | **Bến nhập đưa vào `QUARANTINED`; KCS định đoạt 3 hướng: `SELLABLE`, `QUARANTINED`, hoặc `DISPOSED`.** | Phiếu `ReturnReceipt` (E62) chốt `RETURN_RECEIPT_IN` (`+OnHandQty`, `+QuarantinedQty`); sau đó `ReturnLineAssessment` (E72) chuyển `SELLABLE` (`-QuarantinedQty`, tính WAC), giữ `QUARANTINED`, hoặc xuất hủy `SCRAP_OUT` (`-OnHandQty`, `-QuarantinedQty`) (`CORR-02`). |
| **RET-Q06** | Số lượng kho tiếp nhận cho một yêu cầu? | **Mỗi yêu cầu RMA chỉ định ĐÚNG MỘT kho tiếp nhận (`WarehouseID NOT NULL`).** | Tránh chia tách phân tán hàng hoàn. Mọi phiếu `ReturnReceipt` của yêu cầu phải tiếp nhận tại đúng kho đã chỉ định. |
| **RET-Q07** | Thời điểm tạo yêu cầu hoàn tiền (`Refund`)? | **Tách rời hoàn toàn khỏi trạng thái nhập kho bán được; kích hoạt ngay khi biên bản thẩm định `ReturnLineAssessment` (E72) được duyệt `APPROVED` (`CORR-01`).** | Hàng bị tiêu hủy `DISPOSED` do lỗi nhà sản xuất (`VENDOR_FAULT`) hoặc vận chuyển (`CARRIER_FAULT`) vẫn được duyệt hoàn tiền 100%. Hỗ trợ hoàn tiền từng mốc theo số lượng thực nhận. |
| **RET-Q08** | Lưu trữ lịch sử trạng thái yêu cầu trả hàng? | **Có, lưu vết toàn diện qua thực thể `ReturnStatusHistory`.** | Đảm bảo tính kiểm toán và đo lường SLA xử lý RMA: lưu `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedByAccountID` và `ChangeReason`. |

---

## 13. Chính thức hóa Quyết định Thiết kế Subject Area 11: Audit & Reporting (AR-Q01 đến AR-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **AR-Q01** | Phạm vi ghi nhật ký kiểm toán (AuditLog)? | **Chỉ ghi các thao tác quan trọng và nhạy cảm; không ghi log mọi CRUD/đọc.** | Bảo vệ hiệu năng I/O cho OLTP. Các hành động theo dõi: Duyệt đơn, hủy đơn, kiểm kê kho, chỉnh tồn, duyệt hoàn tiền, đổi giá SKU, cấp quyền Role/Permission. |
| **AR-Q02** | Mức độ lưu chi tiết thay đổi dữ liệu? | **Lưu OldValue và NewValue cho các trường được theo dõi (`AuditLogDetail`).** | Tối ưu hóa dung lượng lưu trữ, phục vụ hiển thị so sánh (Diff Viewer). Tuyệt đối cấm lưu mật khẩu, OTP, token bảo mật vào nội dung log. |
| **AR-Q03** | Có cho phép chỉnh sửa hoặc xóa AuditLog? | **Tuyệt đối cấm chỉnh sửa/xóa qua chức năng nghiệp vụ (`Append-Only / WORM`).** | Tuân thủ nguyên tắc an ninh kiểm toán SOX/PCI-DSS/ISO 27001. Dữ liệu chỉ ghi, cấm sửa, cấm xóa bởi bất kỳ người dùng hay quản trị viên nào. |
| **AR-Q04** | Thời gian lưu trữ AuditLog? | **Cấu hình thời hạn lưu trữ hoạt động (12–24 tháng), sau đó chuyển vào Cold Archive.** | Sử dụng Table Partitioning trên SQL Server để duy trì hiệu năng chỉ mục tra cứu nhanh, sau đó di chuyển log cũ sang kho lưu trữ lạnh. |
| **AR-Q05** | Cơ chế xây dựng báo cáo quản trị? | **Kết hợp: Báo cáo vận hành truy vấn trực tiếp; Báo cáo tài chính dùng `ReportSnapshot`.** | Báo cáo vận hành đọc trực tiếp từ Views tối ưu; báo cáo tài chính/kế toán dùng snapshot để chốt cố định số liệu kỳ kế toán, tránh biến động hồi tố. |
| **AR-Q06** | Lưu vết lịch sử chạy và xuất báo cáo? | **Có, ghi nhận lịch sử qua thực thể `ReportExecution`.** | Phục vụ kiểm toán an ninh dữ liệu (ai đã trích xuất dữ liệu ra file) và hỗ trợ tác vụ xuất file dung lượng lớn bất đồng bộ lưu tại `ResultLocation`. |
| **AR-Q07** | Phân quyền truy cập và sử dụng báo cáo? | **Phân quyền ma trận qua thực thể kết hợp `ReportPermission` (Role - Report).** | Tích hợp trực tiếp với RBAC của Subject Area 04, phân tách rõ 3 quyền hạn: `CanView` (xem), `CanExport` (xuất file) và `CanCreateSnapshot` (chốt kỳ). |
