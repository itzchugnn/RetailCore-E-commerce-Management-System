# M1.11 - Business Decisions Register (Chính thức hóa BD-01 đến BD-07)

* **Milestone:** M1 – Business Analysis
* **Phiên bản:** 1.1 – Approved Business Decisions
* **Database Target:** Microsoft SQL Server
* **Trạng thái:** Đã phê duyệt chính thức (Approved)

---

## 1. Bảng Tổng hợp 7 Quyết định Nghiệp vụ Chính thức

| ID | Quyết định | Kết luận chính thức | Tác động trực tiếp lên Mô hình CSDL (M2–M4) |
| :---: | :--- | :--- | :--- |
| **BD-01** | **Sản phẩm – Danh mục** | Một sản phẩm có thể thuộc nhiều danh mục ($N-N$). Có một danh mục chính. | Thực thể liên kết `ProductCategory(ProductID, CategoryID, IsPrimary)`. Hỗ trợ Breadcrumb và báo cáo doanh thu không trùng lặp. |
| **BD-02** | **Guest Checkout** | Bắt buộc đăng nhập trước khi đặt hàng. Không hỗ trợ Guest Checkout. | `Orders.CustomerID` bắt buộc `NOT NULL`. Quản lý chuẩn `VoucherUsages`, chính sách đổi trả (7 ngày) và hồ sơ khách hàng. |
| **BD-03** | **Split Shipment** | Một đơn hàng có thể được xử lý và giao từ nhiều kho. | Tách rời `Order` khỏi `WarehouseID`. `OrderLine` được chia thành các `OrderAllocation`, xuất kho qua nhiều `Shipment`. |
| **BD-04** | **Reservation** | Giữ hàng ngay khi tạo đơn, có thời hạn `ExpiresAt` (15 phút). | Thực thể `StockReservation` (hoặc gắn với `OrderAllocation`) với thời hạn tạm giữ, ngăn chặn bán âm kho (Overselling) trong Flash Sale. |
| **BD-05** | **Promotion Stacking** | Cho phép tối đa một khuyến mãi cấp sản phẩm và một voucher cấp đơn hàng, nếu quy tắc cho phép cộng dồn (`IsStackable = 1`). | Tách bạch 2 tầng khuyến mãi; cấm áp dụng 2 chương trình cùng tầng; lưu vết chi tiết tính toán trong `OrderPromotions`. |
| **BD-06** | **Variant Attributes** | Sử dụng mô hình thuộc tính linh hoạt theo từng sản phẩm (Dynamic Attributes). | Bộ ba thực thể chuẩn hóa `Attribute`, `AttributeValue`, `ProductAttribute`, `VariantAttributeValue`. Không hardcode cột Size/Color. |
| **BD-07** | **Inventory on Delivery & Failed Delivery** | Khi xuất kho, giảm đồng thời tồn vật lý (`OnHandQty`) và lượng giữ (`ReservedQty`). Hàng giao thất bại chỉ được nhập lại kho khi thực sự quay về. | Phân định rạch ròi 4 trạng thái tồn kho và tuân thủ vòng đời 5 bước: Tạo đơn $\rightarrow$ Xuất kho $\rightarrow$ Đang giao $\rightarrow$ Tiếp nhận hoàn kho $\rightarrow$ Giao lại/Hủy. |

---

## 2. Chi tiết 2 Quyết định Trọng tâm: BD-05 & BD-07

### 2.1. BD-05: Chính sách Khuyến mãi Phân tầng & Điều kiện Cộng dồn (Promotion Stacking)

#### Hai tầng khuyến mãi độc lập:
1. **Tầng 1 – Product Promotion (Khuyến mãi cấp sản phẩm / Dòng hàng):**
   * Áp dụng trực tiếp trên từng dòng sản phẩm (`OrderLine`).
   * Ví dụ: Flash Sale giảm 15% cho Áo thun Polo, Giảm 50.000đ cho Đèn bàn.
   * **Giới hạn:** Tối đa **1 chương trình** giảm giá trực tiếp trên mỗi dòng sản phẩm.
2. **Tầng 2 – Order Voucher (Voucher cấp đơn hàng):**
   * Áp dụng trên toàn bộ giá trị đơn hàng sau khi đã trừ chiết khấu dòng sản phẩm.
   * Ví dụ: Nhập mã `SUMMER2026` giảm 10% tối đa 100.000đ cho đơn từ 500.000đ.
   * **Giới hạn:** Tối đa **1 voucher** cấp đơn hàng trên mỗi đơn.

#### Quy tắc cộng dồn (Stacking Rules):
* Hai tầng **chỉ được phép kết hợp** khi và chỉ khi:
  1. Voucher cấp đơn hàng có cờ `IsStackable = 1` (hoặc thuộc tính `CanCombineWithProductDiscount = TRUE`).
  2. Chương trình khuyến mãi cấp sản phẩm không có điều khoản cấm kết hợp mã giảm giá.
* **Quy tắc chặn tuyệt đối:** Không được áp dụng nhiều chương trình cùng tầng vượt quá giới hạn (ví dụ: không được nhập 2 voucher đơn hàng, không được áp 2 chương trình flash sale trên 1 dòng sản phẩm), kể cả khi các chương trình riêng lẻ đều thỏa điều kiện áp dụng.

#### Lưu vết đối soát tài chính (`OrderPromotions` & `OrderLine`):
* `OrderLine.UnitPrice`: Giá niêm yết snapshot tại thời điểm đặt.
* `OrderLine.DiscountAmount`: Số tiền giảm trực tiếp từ Tầng 1 (Product Promotion).
* `Order.VoucherDiscountAmount`: Số tiền giảm từ Tầng 2 (Order Voucher).
* Bảng `OrderPromotions`: Lưu toàn bộ danh sách các chương trình đã áp dụng, mức giảm, thứ tự áp dụng (Thứ tự 1: Tính giảm dòng $\rightarrow$ Thứ tự 2: Tính giảm toàn đơn) và giá trị trước/sau giảm để phục vụ đối soát kế toán bất biến.

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
   * Công thức tồn khả dụng: $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$.
2. **Bước 2: Kho đóng gói & Bàn giao vận chuyển (Shipment Handover – `SHIPMENT_OUT`)**
   * Hàng thực sự rời khỏi kho vật lý.
   * Giảm đồng thời cả `OnHandQty` và `ReservedQty` theo số lượng thực tế xuất.
   * Ghi một bản ghi vào `StockMovement` loại `SHIPMENT_OUT`.
3. **Bước 3: Giao hàng thất bại (Delivery Failed)**
   * Đơn vị vận chuyển báo giao thất bại (khách không nghe máy, sai địa chỉ).
   * Hệ thống cập nhật trạng thái `Shipment` thành `DELIVERY_FAILED`.
   * **Tuyệt đối chưa tăng tồn kho**, vì hàng vẫn đang nằm trên xe hoặc tại bưu cục của đơn vị vận chuyển.
4. **Bước 4: Kho tiếp nhận hàng hoàn về (Delivery Return Receipt – `DELIVERY_RETURN_IN`)**
   * Hàng thực tế được xe chở quay trở lại kho xuất (hoặc kho chỉ định).
   * Nhân viên kho kiểm tra tình trạng hàng hóa, ký biên bản `DeliveryReturnReceipt`.
   * Hệ thống ghi `StockMovement` loại `DELIVERY_RETURN_IN` và tăng `OnHandQty` theo số lượng thực nhận lại.
5. **Bước 5: Giao lại hoặc Hủy đơn (Re-delivery or Order Cancellation)**
   * *Nếu giao lại:* Hệ thống tiến hành phân bổ kho (`OrderAllocation`) và giữ hàng lại (`StockReservation`) theo quy tắc tồn kho hiện hành trước khi xuất kho lần 2.
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
| **SHOP-Q04** | Cơ chế lưu trữ và tính toán giá bán? | **Chỉ lấy giá hiện hành từ SKU; Snapshot giá khi tạo đơn hàng.** | Không lưu giá cố định trong `CartItem`. Giá bán chỉ được chốt bất biến tại `OrderLine.UnitPrice` khi tạo đơn hàng (`BR-CU06`, `BR-OR05`). |
| **SHOP-Q05** | Đồng bộ giỏ hàng trên nhiều thiết bị? | **Đồng bộ tập trung trên Database Server theo `CustomerID`.** | Giỏ hàng lưu trữ bền vững trên CSDL, đồng bộ liền mạch giữa Web, Mobile App và Tablet khi khách hàng đăng nhập. |
| **SHOP-Q06** | Trạng thái giỏ hàng sau khi chuyển đổi thành đơn hàng? | **Chuyển thành `CONVERTED`, đóng băng lịch sử và tạo giỏ mới khi mua tiếp.** | Khép kín vòng đời phễu chuyển đổi (Funnel Conversion Tracking), bảo toàn dữ liệu phục vụ đối soát và phân tích hành vi mua sắm. |

---

## 8. Chính thức hóa Quyết định Thiết kế Subject Area 06: Sales (SALES-Q01 đến SALES-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **SALES-Q01** | Thời điểm tạo đơn hàng và giữ hàng tồn kho? | **Tạo đơn và giữ toàn bộ hàng nguyên tử (Atomic); thất bại thì không tạo đơn chính thức.** | Tuân thủ BD-04 và WI-Q02 (All-or-nothing). Ngăn chặn tình trạng tạo đơn thiếu hàng. Giỏ hàng giữ nguyên `ACTIVE` nếu giữ hàng thất bại. |
| **SALES-Q02** | Vòng đời trạng thái đơn hàng (Order State Machine)? | **`PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `COMPLETED` (có `CANCELLED`).** | Trạng thái ban đầu là `PENDING_PAYMENT`. Chỉ chuyển sang `CONFIRMED` khi thanh toán thành công (hoặc COD hợp lệ). Kho chỉ đóng gói đơn `CONFIRMED`. |
| **SALES-Q03** | Chính sách hủy đơn hàng? | **Kết hợp: Khách tự hủy khi chưa đóng gói (`PENDING_PAYMENT`, `CONFIRMED`); sau đó cần nhân viên xử lý.** | Bảo vệ vận hành kho khỏi việc hủy đơn khi hàng đã đóng gói hoặc đã lên xe giao vận. Hủy đơn tự động kích hoạt nhả tồn kho giữ. |
| **SALES-Q04** | Thay đổi dòng hàng sau khi xác nhận đơn? | **Tuyệt đối không cho phép sửa trực tiếp `OrderLine` sau khi đơn đã xác nhận.** | Tránh vỡ thuật toán phân bổ kho, vỡ logic voucher khuyến mãi và tính toán lại công nợ. Khách muốn đổi món buộc phải hủy đơn cũ và đặt lại. |
| **SALES-Q05** | Mức độ snapshot thông tin sản phẩm trên `OrderLine`? | **Snapshot toàn bộ: SKU, tên sản phẩm, tên biến thể, đơn giá và `VariantAttributesSnapshot` (JSON).** | Bảo toàn 100% hợp đồng thương mại và hóa đơn theo luật thuế. Đơn hàng cũ hiển thị độc lập hoàn toàn với các biến động của Catalog sau này. |
| **SALES-Q06** | Quản lý địa chỉ giao hàng trong đơn? | **Mỗi đơn hàng có ĐÚNG MỘT địa chỉ giao hàng snapshot (`SalesOrder 1 ─── 1 OrderAddress`).** | Split shipment chỉ là chia nguồn xuất kho, toàn bộ kiện hàng đều giao về cùng 1 địa chỉ nhận của đơn. Độc lập với sổ địa chỉ `CustomerAddress`. |
| **SALES-Q07** | Chính sách lưu trữ đơn hàng và dòng hàng? | **Tuyệt đối không xóa vật lý (`No Hard Delete`) đơn hàng và dòng hàng đã phát sinh.** | Đơn hàng là chứng từ kế toán, pháp lý. Đơn hủy giữ nguyên trạng thái `CANCELLED` cùng nhật ký `OrderStatusHistory` phục vụ kiểm toán và báo cáo tỷ lệ hủy. |
