# M2.05 - CONCEPTUAL DESIGN DECISIONS & ARCHITECTURAL PRINCIPLES

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 2.0 – Canonical SSOT
* **Trạng thái:** Approved Decisions (79 Entities Baseline)
* **Kế thừa trực tiếp:** BD-01 đến BD-07 (M1) & CORR-01 đến CORR-10 (M2.08)

---

## 1. Ba Quyết định Kỹ thuật Trọng tâm Bước vào M2 (Không thể Vi phạm)

### A. `OrderWarehouseAllocation` là thực thể ứng viên cốt lõi cho Split Shipment
* **Vấn đề thực tế:** Một đơn hàng `ORD001` gồm Áo khoác (Kho Quận 1) và Bàn làm việc (Kho Thủ Đức). Hoặc 1 dòng `OrderLine` đặt 5 cái nhưng Kho A chỉ có 3 và Kho B có 2.
* **Quyết định thiết kế:** Bắt buộc có lớp phân bổ trung gian `OrderWarehouseAllocation` (E12).
* **Cơ chế liên kết:** `OrderWarehouseAllocation` là cầu nối chuẩn xác kết nối giữa:
  * Yêu cầu thương mại của khách hàng (`OrderLine`).
  * Thực tế kho cung ứng (`Warehouse`).
  * Cam kết giữ hàng vật lý (`StockReservationLine`).
  * Kiện hàng thực tế xuất đi (`ShipmentLine`).
* Nếu thiếu `OrderWarehouseAllocation`, mô hình sẽ bị vướng vào bế tắc khi cố gắng nối `OrderLine` trực tiếp với `Warehouse` hoặc `ShipmentLine`.

### B. Tách bạch 4 khái niệm Tồn kho & Vận hành (Không được gộp)
Tuyệt đối không được gộp 4 khái niệm sau thành một thực thể duy nhất:
1. **`OrderWarehouseAllocation` (E12)**: Đơn hàng được **phân bổ** bao nhiêu hàng từ kho nào (Cam kết kế hoạch).
2. **`StockReservation` & `StockReservationLine` (E13, E14)**: Lượng hàng đang được **giữ tạm thời tại kho** trước khi xuất kho (Khóa logic tồn kho có hạn `ExpiresAt`).
3. **`Shipment` & `ShipmentLine` (E45, E46)**: Một **lần giao hàng** vật lý từ một kho cụ thể (Vận đơn, đối tác vận chuyển, tracking).
4. **`InventoryTransaction` & `InventoryTransactionLine` (E15, E16)**: **Sự kiện ghi sổ kế toán kho** làm thay đổi tồn kho vật lý (`OnHandQty`) và phẩm chất kho bất biến.

### C. Giao hàng thất bại KHÔNG đồng nghĩa hàng đã quay về kho
* **Nguyên tắc cốt lõi:** Khi shipper báo giao thất bại, hàng vẫn đang nằm trên thùng xe hoặc tại bưu cục chuyển phát. Hàng **chưa hề** quay lại giá kệ của kho.
* **Vòng đời 5 bước của tồn kho:**
  1. **Tạo đơn hàng:** `OnHandQty` giữ nguyên; `ReservedQty` tăng; $\text{AvailableQty} = (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}) - \text{ReservedQty}$.
  2. **Xuất kho bàn giao vận chuyển (`SHIPMENT_OUT`):** `OnHandQty` giảm; `ReservedQty` giảm. Ghi nhật ký `InventoryTransaction` loại `SHIPMENT_OUT`. Lượt reservation tại kho được tiêu thụ. Hàng rời khỏi kho được theo dõi qua `Shipment`, không nằm trong `OnHandQty` và không nằm trong `ReservedQty`.
  3. **Giao hàng thất bại:** Ghi nhận `DeliveryAttempt`; hết 3 lần giao thì chuyển `ShipmentStatus = 'RETURNED_TO_SENDER'`. **Chưa tăng tồn kho.**
  4. **Kho xác nhận nhận hàng (`DELIVERY_RETURN_IN`):** Nhân viên kho kiểm tra kiện hàng hoàn về, ký biên bản tiếp nhận `DeliveryReturnReceipt` (E75). Hệ thống ghi `InventoryTransaction` loại `DELIVERY_RETURN_IN` và tăng `OnHandQty` theo số lượng thực tế nhận lại.
  5. **Giao lại hoặc Hủy đơn:**
     * *Giao lại:* Tạo `OrderWarehouseAllocation` và `StockReservationLine` mới theo tồn kho hiện hành trước khi xuất lần 2.
     * *Hủy đơn:* Kích hoạt quy trình `Refund` và đóng đơn hàng.

---

## 2. Giải quyết 10 Câu hỏi Thiết kế Trọng yếu (M2-Q01 đến M2-Q10)

| ID | Câu hỏi thiết kế | Kết luận & Quyết định kỹ thuật |
| :---: | :--- | :--- |
| **M2-Q01** | `ProductVariant` có phải là thực thể độc lập không? | **Có.** `ProductVariant` (SKU) có surrogate key riêng (`VariantID`), định danh duy nhất toàn hệ thống bằng `SKU`. Giá bán, tồn kho, giữ kho và dòng đơn hàng đều gắn với `ProductVariant`, không gắn với `Product`. |
| **M2-Q02** | `ProductAttribute` có cần tách khỏi `Attribute` không? | **Cần tách.** `Attribute` là từ điển thuộc tính toàn cục (Color, Size). `ProductAttribute` xác định sản phẩm cụ thể áp dụng thuộc tính nào (ngăn chặn gán Size cho Đồ gia dụng). |
| **M2-Q03** | `OrderWarehouseAllocation` có cần tồn tại độc lập không? | **Bắt buộc có.** Đóng vai trò thực thể trung gian phân bổ chi tiết số lượng của 1 `OrderLine` cho từng kho vật lý cụ thể (phục vụ Split Shipment). |
| **M2-Q04** | `StockReservationLine` liên hệ với `OrderLine` hay `OrderWarehouseAllocation`? | **Liên hệ với `OrderWarehouseAllocation` (`AllocationID`, quan hệ $1:0..N$ theo CORR-10).** Tồn kho vật lý luôn gắn với một kho cụ thể, do đó lượt giữ hàng phải gắn đích danh vào kho đã phân bổ. |
| **M2-Q05** | `ShipmentLine` liên hệ với `OrderLine` hay `OrderWarehouseAllocation`? | **Liên hệ với cả `OrderLine` và `OrderWarehouseAllocation`.** Mỗi `Shipment` xuất phát từ 1 kho và sẽ tiêu thụ trực tiếp lượng hàng đã phân bổ và giữ tại kho đó. |
| **M2-Q06** | `InventoryTransaction` có cần tham chiếu chứng từ nguồn không? | **Bắt buộc có.** Cần `ReferenceType` (`GOODS_RECEIPT`, `SHIPMENT_OUT`, `WAREHOUSE_TRANSFER`, `STOCK_COUNT`, `DELIVERY_RETURN_RECEIPT`, `RETURN_RECEIPT`, `RETURN_LINE_ASSESSMENT`, `SUPPLIER_RETURN`) và `ReferenceID` để đối soát kiểm toán kho bất biến. |
| **M2-Q07** | `Promotion` và `Voucher` được tổ chức như thế nào? | **Tách 2 thực thể liên kết $1:0..N$ theo 3 tầng ưu đãi:** `Promotion` (E49) quản lý chương trình khuyến mãi (`PromotionScope` = `'PRODUCT'`, `'ORDER'` hoặc `'SHIPPING'` Freeship, cờ `IsStackable`); `Voucher` (E52) quản lý mã giảm giá phát hành thuộc chương trình cấp `'ORDER'` hoặc `'SHIPPING'`. |
| **M2-Q08** | `ReturnRequestLine` truy vết kiện hàng giao như thế nào trong Split Shipment? | **Liên kết qua thực thể cầu nối `ReturnShipmentAllocation` (E76) nối `ReturnRequestLine` (E61) và `ShipmentLine` (E46).** Bảo đảm kiểm tra thời hạn đổi trả theo đúng `Shipment.DeliveredAt` của từng kiện và truy vết đúng `OutboundUnitCost` (CORR-08, CORR-10). |
| **M2-Q09** | `CustomerAddress` có cần snapshot trong `SalesOrder`? | **Bắt buộc snapshot 100% sang `OrderAddress` (E42).** Địa chỉ giao hàng tại thời điểm đặt phải được sao chép nguyên văn, không trỏ FK về `CustomerAddress` để tránh việc khách đổi sổ địa chỉ làm sai lệch lịch sử đơn cũ. |
| **M2-Q10** | `Payment` và `SalesOrder` có quan hệ 1–1 hay 1–N? | **Quan hệ 1–1 đối với hồ sơ nghĩa vụ thanh toán (`Payment`, `OrderID` UNIQUE), và 1–N đối với các lần giao dịch cổng (`PaymentTransaction`) cùng các yêu cầu hoàn tiền (`Refund`) (PAY-Q01, PAY-Q04, CORR-04).** |

---

## 3. Quyết định Thiết kế Subject Area 01: Product Catalog (PC-Q01 đến PC-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **PC-Q01** | Bắt buộc Danh mục chính khi Published? | **CÓ, bắt buộc đúng một danh mục chính (`IsPrimary = 1`) khi Published.** | Cài đặt `UNIQUE FILTERED INDEX (ProductID) WHERE IsPrimary = 1` ở SQL Server. Bắt buộc có đúng 1 danh mục chính để render Breadcrumb, Canonical URL và tính doanh thu theo danh mục không trùng lặp (DR-02). |
| **PC-Q02** | Cấu trúc phân cấp Danh mục? | **CÓ, hỗ trợ nhiều cấp cha – con.** | Thiết kế khóa ngoại đệ quy `Category.ParentCategoryID REFERENCES Category(CategoryID)`. Hỗ trợ cây danh mục đa tầng cho Thời trang & Gia dụng. |
| **PC-Q03** | Sản phẩm không có biến thể thuộc tính (Simple Product)? | **TẠO 1 SKU MẶC ĐỊNH, tổ hợp thuộc tính rỗng.** | Không bao giờ cho phép bán trực tiếp từ `Product`. Mọi giao dịch đơn hàng và tồn kho đều trỏ duy nhất đến `ProductVariant.VariantID`. Tránh được lỗi thiết kế Polymorphic Foreign Key. |
| **PC-Q04** | Một thuộc tính vừa mô tả vừa xác định SKU? | **KHÔNG, phân định rõ vai trò qua `AttributeRole` (`DESCRIPTIVE` / `VARIANT_DEFINING`).** | Khai báo rõ ràng trong `ProductAttribute`: `AttributeRole = 'VARIANT_DEFINING'` (thuộc tính biến thể, cấu thành SKU) hoặc `'DESCRIPTIVE'` (thuộc tính mô tả chung, không tham gia phân tách SKU) - hai vai trò độc quyền loại trừ lẫn nhau trên từng sản phẩm (BR-PR13). |
| **PC-Q05** | Dùng chung định nghĩa `Attribute`? | **CÓ, dùng chung từ điển `Attribute` toàn cục.** | Chuẩn hóa toàn hệ thống. Hỗ trợ tính năng Faceted Filter / Bộ lọc thuộc tính trên Website (khách chọn lọc màu Đen sẽ ra tất cả sản phẩm có màu Đen). |
| **PC-Q06** | Bảo toàn SKU khi đổi thuộc tính? | **CÓ, bảo toàn tuyệt đối SKU và dữ liệu lịch sử.** | Không được sửa đổi ngữ nghĩa hoặc xóa vật lý SKU đã từng phát sinh giao dịch. Áp dụng `IsActive = 0` (Soft Delete) khi ngừng kinh doanh thuộc tính/giá trị. |
---

## 4. Quyết định Thiết kế Subject Area 02: Warehouse & Inventory (WI-Q01 đến WI-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **WI-Q01** | Quản lý tồn kho đến cấp kệ, tầng, ô chứa (`Location`)? | **KHÔNG, chỉ quản lý theo cấp Kho (`Warehouse`) ở phiên bản này.** | Giữ mô hình tồn kho tinh gọn, tránh over-engineering; trường `Location` có thể mở rộng ở phase sau mà không phá vỡ mô hình hiện tại. |
| **WI-Q02** | Giữ hàng một phần khi thiếu tồn kho? | **KHÔNG, áp dụng nguyên tắc All-or-nothing cho toàn bộ đơn hàng.** | Tránh tình trạng đơn hàng tạo thành công nhưng chỉ giữ được 1 phần hàng (gây thất vọng cho khách và phức tạp cho việc hủy dở dang). Nếu thiếu bất kỳ SKU nào trong đơn, giao dịch checkout bị từ chối và báo lỗi ngay. |
| **WI-Q03** | Phương thức phân bổ kho (`OrderWarehouseAllocation`)? | **TỰ ĐỘNG phân bổ, cho phép nhân viên điều chỉnh trước khi xuất hàng.** | Thuật toán phân bổ tự động theo ưu tiên (Kho gần khách nhất, Kho có đủ toàn bộ hàng). Nhân viên CSKH/Kho có thể can thiệp điều chỉnh nếu có sự cố thực tế tại kho. |
| **WI-Q04** | Theo dõi hàng chuyển kho (`In-Transit`)? | **CÓ, theo dõi chặt chẽ quy trình Chuyển kho 2 bước.** | Bổ sung thực thể `WarehouseTransfer` và `WarehouseTransferLine`. Khi kho xuất tạo `TRANSFER_OUT` thì giảm `OnHand` tại kho nguồn; hàng ở trạng thái `IN_TRANSIT`; chỉ khi kho đích ký nhận `TRANSFER_IN` thì mới tăng `OnHand` tại kho đích. |
| **WI-Q05** | Phê duyệt phiếu kiểm kê trước khi cân chỉnh tồn kho? | **CÓ, phiếu kiểm kê bắt buộc được phê duyệt mới sinh giao dịch điều chỉnh.** | Phân tách rõ 2 bước: Hoàn tất kiểm đếm (`COMPLETED`) $\rightarrow$ Quản lý duyệt chênh lệch (`APPROVED`) $\rightarrow$ Hệ thống tự sinh `InventoryTransaction` loại `STOCK_ADJUSTMENT` để cập nhật `InventoryBalance`. |
| **WI-Q06** | Cơ chế phê duyệt điều chỉnh tồn kho? | **CÓ, áp dụng cơ chế phê duyệt phân tách nhiệm vụ (SoD).** | Mọi giao dịch điều chỉnh tăng/giảm tồn kho thông qua phiếu kiểm kê `StockCount` (E17) và sổ cái `InventoryTransaction` (E15, `TransactionType = 'STOCK_ADJUSTMENT'`) phải lưu vết `ApprovedBy`, `ApprovedAt` (`ApprovedBy <> CreatedBy`) và lý do điều chỉnh (`Notes`) để đảm bảo tính minh bạch kiểm toán. |

---

## 5. Quyết định Thiết kế Subject Area 03: Procurement (PR-Q01 đến PR-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **PR-Q01** | Một PO có thể nhận hàng tại nhiều kho không? | **KHÔNG, một PO chỉ có ĐÚNG MỘT kho nhận dự kiến (`DestinationWarehouseID`).** | Đơn giản hóa đối soát chứng từ, hóa đơn VAT và phiếu kho. Nếu cần nhập cho nhiều kho, tạo các PO riêng biệt. `GoodsReceipt.WarehouseID` bắt buộc trùng khớp với kho nhận của PO. |
| **PR-Q02** | Có cho phép nhận hàng nhiều đợt (Partial Receipt) cho một PO không? | **CÓ, cho phép nhận hàng nhiều đợt qua nhiều `GoodsReceipt`.** | Phù hợp thực tế vận tải của nhà cung cấp. Quan hệ `PurchaseOrder` $1:N$ `GoodsReceipt` và `PurchaseOrderLine` $1:N$ `GoodsReceiptLine`. Trạng thái PO tự động chuyển từ `PARTIALLY_RECEIVED` sang `FULLY_RECEIVED` khi nhận đủ. |
| **PR-Q03** | Nhà cung cấp giao vượt số lượng đặt mua được xử lý thế nào? | **MẶC ĐỊNH KHÔNG CHO PHÉP NHẬN VƯỢT.** | Ràng buộc: $\sum \text{GoodsReceiptLine.ReceivedQty} \le \text{PurchaseOrderLine.OrderedQty}$. Kiểm soát chặt chẽ ngân sách đã duyệt và công nợ NCC. Nếu muốn nhập thêm phải lập PO bổ sung. |
| **PR-Q04** | Hàng không đạt chất lượng (QC) khi tiếp nhận được xử lý thế nào? | **GHI NHẬN `RejectedQty` TẠI PHIẾU NHẬP VÀ XỬ LÝ RIÊNG.** | Phân tách: $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$. Chỉ `AcceptedQty` mới tạo `PURCHASE_RECEIPT` và tăng `OnHandQty`. `RejectedQty` không đưa vào tồn khả dụng, dùng để đòi giao bù hoặc lập biên bản xuất trả. |
| **PR-Q05** | Có cần quản lý trả hàng cho nhà cung cấp (`SupplierReturn`) không? | **CÓ, quản lý qua cặp `SupplierReturn` và `SupplierReturnLine`.** | Đóng kín chu trình mua hàng. Trả hàng đã nhập kho sẽ sinh giao dịch `SUPPLIER_RETURN_OUT` làm giảm `OnHandQty`. Dòng trả hàng liên kết đối soát về `GoodsReceiptLineID` gốc để kiểm soát số lượng và giá vốn. |
| **PR-Q06** | PO có bắt buộc được phê duyệt trước khi gửi nhà cung cấp không? | **CÓ, BẮT BUỘC PHÊ DUYỆT (`ApprovedBy`, `ApprovedAt`).** | Tuân thủ nguyên tắc phân tách trách nhiệm (SoD). Chỉ các PO ở trạng thái `APPROVED` hoặc `SENT` mới có hiệu lực pháp lý và mới được phép tạo `GoodsReceipt` tiếp nhận hàng. |

---

## 6. Quyết định Thiết kế Subject Area 04: Customer & Identity (CID-Q01 đến CID-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **CID-Q01** | Account có thể đồng thời là Customer và Employee không? | **KHÔNG, phân tách loại trừ lẫn nhau (Disjoint Subtypes / Ràng buộc XOR).** | Tránh nguy cơ leo thang đặc quyền (Privilege Escalation), ngăn chặn xung đột lợi ích nội bộ và cách ly tài khoản quản trị Back-office khỏi tài khoản mua sắm công cộng. |
| **CID-Q02** | Một Account có thể có nhiều Role không? | **CÓ, hỗ trợ nhiều Role qua thực thể kết hợp `AccountRole` ($N:N$).** | Cho phép nhân sự nội bộ kiêm nhiệm nhiều vị trí (vừa soạn kho vừa kiểm kê, hoặc quản lý mua hàng kiêm nhập liệu) và phân cấp nhóm khách hàng VIP. |
| **CID-Q03** | Phương thức đăng nhập và định danh Account? | **HỖ TRỢ EMAIL HOẶC SỐ ĐIỆN THOẠI + MẬT KHẨU.** | Phù hợp thói quen mua sắm thương mại điện tử tại thị trường Việt Nam (đăng nhập bằng SĐT qua OTP SMS). CSDL cài đặt Filtered Unique Index cho cả `Email` và `Phone`. |
| **CID-Q04** | Thông tin liên hệ nào bắt buộc khi đăng ký? | **ÍT NHẤT MỘT TRONG HAI (Email hoặc Phone).** | Giảm rào cản chuyển đổi khi đăng ký tài khoản mới. Ràng buộc kiểm tra: `CHECK (Email IS NOT NULL OR Phone IS NOT NULL)`. Khi tiến hành đặt hàng (Checkout), số điện thoại nhận hàng bắt buộc phải có tại địa chỉ giao hàng. |
| **CID-Q05** | Có cần lưu lịch sử xác minh không? | **CÓ, lưu lịch sử bằng thực thể `AccountVerification`.** | Phục vụ kiểm soát Rate Limiting chống spam OTP, quản lý thời hạn `ExpiresAt`, phục vụ quy trình đổi Email/Phone và điều tra an ninh tài khoản khi có sự cố. |
| **CID-Q06** | Quy tắc quản lý địa chỉ giao hàng mặc định? | **TỐI ĐA 1 ĐỊA CHỈ MẶC ĐỊNH; địa chỉ đầu tiên tự động làm mặc định.** | Cài đặt bằng `UNIQUE FILTERED INDEX (CustomerID) WHERE IsDefault = 1` trên SQL Server. Khi địa chỉ mặc định bị vô hiệu hóa, bắt buộc chọn địa chỉ mới nếu còn. |

---

## 7. Quyết định Thiết kế Subject Area 05: Shopping (SHOP-Q01 đến SHOP-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **SHOP-Q01** | Số lượng giỏ hàng của một khách hàng? | **Một giỏ hàng ACTIVE, có thể có nhiều giỏ hàng lịch sử.** | Cài đặt `UNIQUE FILTERED INDEX (CustomerID) WHERE CartStatus = 'ACTIVE'`. Lưu vết lịch sử giỏ hàng chuyển đổi và bỏ rơi phục vụ Marketing. |
| **SHOP-Q02** | Xử lý khi thêm cùng một SKU nhiều lần? | **Cộng dồn số lượng (`Quantity`) vào dòng `CartItem` hiện có.** | Ràng buộc duy nhất `UNIQUE (CartID, VariantID)`. Tránh phân mảnh dòng sản phẩm trong giỏ hàng. |
| **SHOP-Q03** | Xử lý khi SKU hết hàng hoặc ngừng kinh doanh? | **Giữ `CartItem` và hiển thị cảnh báo; khóa nút Checkout.** | Khách hàng chủ động nắm được tình trạng sản phẩm; ngăn chặn tạo đơn hàng không thể đáp ứng giao vận. |
| **SHOP-Q04** | Cơ chế lưu trữ và tính toán giá bán? | **Chỉ lấy giá hiện hành từ SKU; Snapshot giá khi tạo đơn hàng.** | Không lưu giá cố định trong `CartItem`. Giá bán chỉ được chốt bất biến tại `OrderLine.UnitPrice` khi tạo đơn hàng (`BR-SH09`, `BR-SH10`, `BR-SA05`). |
| **SHOP-Q05** | Đồng bộ giỏ hàng trên nhiều thiết bị? | **Đồng bộ tập trung trên Database Server theo `CustomerID`.** | Giỏ hàng lưu trữ bền vững trên CSDL, đồng bộ liền mạch giữa Web, Mobile App và Tablet khi khách hàng đăng nhập. |
| **SHOP-Q06** | Trạng thái giỏ hàng sau khi chuyển đổi thành đơn hàng? | **Chuyển thành `CONVERTED`, đóng băng lịch sử và tạo giỏ mới khi mua tiếp.** | Khép kín vòng đời phễu chuyển đổi (Funnel Conversion Tracking), bảo toàn dữ liệu phục vụ đối soát và phân tích hành vi mua sắm. |

---

## 8. Quyết định Thiết kế Subject Area 06: Sales (SALES-Q01 đến SALES-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **SALES-Q01** | Thời điểm tạo đơn hàng và giữ hàng tồn kho? | **Tạo đơn và giữ toàn bộ hàng nguyên tử (Atomic); thất bại thì không tạo đơn chính thức.** | Tuân thủ BD-04 và WI-Q02 (All-or-nothing). Ngăn chặn tình trạng tạo đơn thiếu hàng. Giỏ hàng giữ nguyên `ACTIVE` nếu giữ hàng thất bại. |
| **SALES-Q02** | Vòng đời trạng thái đơn hàng (Order State Machine)? | **`PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `PARTIALLY_DELIVERED` $\to$ `DELIVERED` (có `PARTIALLY_CANCELLED` / `CANCELLED`, đóng hành chính qua `ClosedAt`).** | Đơn trả trước khởi tạo ở `PENDING_PAYMENT`; đơn COD hợp lệ chuyển `CONFIRMED` ngay khi đặt (CORR-06). Kho chỉ đóng gói đơn `CONFIRMED`. |
| **SALES-Q03** | Chính sách hủy đơn hàng? | **Kết hợp: Khách tự hủy khi chưa đóng gói (`PENDING_PAYMENT`, `CONFIRMED`); sau đó cần nhân viên xử lý.** | Bảo vệ vận hành kho khỏi việc hủy đơn khi hàng đã đóng gói hoặc đã lên xe giao vận. Hủy đơn tự động kích hoạt nhả tồn kho giữ. |
| **SALES-Q04** | Thay đổi dòng hàng sau khi xác nhận đơn? | **Tuyệt đối không cho phép sửa trực tiếp `OrderLine` sau khi đơn đã xác nhận.** | Tránh vỡ thuật toán phân bổ kho, vỡ logic voucher khuyến mãi và tính toán lại công nợ. Khách muốn đổi món buộc phải hủy đơn cũ và đặt lại. |
| **SALES-Q05** | Mức độ snapshot thông tin sản phẩm trên `OrderLine`? | **Snapshot toàn bộ: SKU, tên sản phẩm, tên biến thể, đơn giá và `VariantAttributesSnapshot` (JSON).** | Bảo toàn 100% hợp đồng thương mại và hóa đơn theo luật thuế. Đơn hàng cũ hiển thị độc lập hoàn toàn với các biến động của Catalog sau này. |
| **SALES-Q06** | Quản lý địa chỉ giao hàng trong đơn? | **Mỗi đơn hàng có ĐÚNG MỘT địa chỉ giao hàng snapshot (`SalesOrder 1 ─── 1 OrderAddress`).** | Split shipment chỉ là chia nguồn xuất kho, toàn bộ kiện hàng đều giao về cùng 1 địa chỉ nhận của đơn. Độc lập với sổ địa chỉ `CustomerAddress`. |
| **SALES-Q07** | Chính sách lưu trữ đơn hàng và dòng hàng? | **Tuyệt đối không xóa vật lý (`No Hard Delete`) đơn hàng và dòng hàng đã phát sinh.** | Đơn hàng là chứng từ kế toán, pháp lý. Đơn hủy giữ nguyên trạng thái `CANCELLED` cùng nhật ký `OrderStatusHistory` phục vụ kiểm toán và báo cáo tỷ lệ hủy. |

---

## 9. Quyết định Thiết kế Subject Area 07: Fulfillment & Shipping (SHIP-Q01 đến SHIP-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **SHIP-Q01** | Số lượng kho xuất hàng cho một Shipment? | **Đúng một kho (`WarehouseID NOT NULL`).** | Tuân thủ BD-03 (Split Shipment). Mỗi kiện hàng vật lý có 1 mã vận đơn (`TrackingNumber`) xuất phát từ duy nhất 1 kho. Không gom hàng từ nhiều kho vào chung 1 Shipment. |
| **SHIP-Q02** | Thời điểm tạo lô giao hàng (Shipment)? | **Sau khi đơn được xác nhận (`CONFIRMED`) và kho chuẩn bị xuất hàng (`PROCESSING`).** | Không tạo Shipment khi đơn đang `PENDING_PAYMENT` hoặc đã bị hủy. Chỉ tạo khi kho bắt đầu nhặt hàng (Picking) để in Pick-list và Pack-slip. |
| **SHIP-Q03** | Cách chọn đơn vị vận chuyển (Carrier)? | **Hệ thống đề xuất dựa trên cấu hình, nhân viên/điều phối viên xác nhận.** | Tầng ứng dụng gợi ý Carrier tối ưu theo vùng miền, cước phí và SLA. CSDL lưu `Shipment.CarrierID` (FK), hỗ trợ linh hoạt cả tự động và điều chỉnh thủ công. |
| **SHIP-Q04** | Xử lý khi giao hàng thất bại? | **Cho phép giao lại tối đa $N$ lần (chuẩn là 3 lần); vượt giới hạn thì chuyển hoàn (`RETURNED_TO_SENDER`).** | Chuẩn hóa theo ngành bưu chính viễn thông Việt Nam. Ghi nhận chi tiết từng lần thử vào `DeliveryAttempt`. Vượt quá hạn mức tự động chuyển `RETURNED_TO_SENDER`; chỉ cộng lại tồn kho khi thủ kho ký `DeliveryReturnReceipt` (E75, `BD-07`). |
| **SHIP-Q05** | Lưu trữ bằng chứng giao hàng (Proof of Delivery)? | **Lưu người nhận, thời gian và ghi chú giao hàng; mở rộng ảnh/chữ ký sau.** | Mức khái niệm lưu `RecipientName`, `AttemptedAt`, `DeliveryNote` trong `DeliveryAttempt`. Mức Logical/Physical có thể mở rộng cột `ProofOfDeliveryURL` mà không vỡ schema. |
| **SHIP-Q06** | Cơ chế cập nhật trạng thái vận chuyển? | **Hỗ trợ cả thủ công và tích hợp tự động qua API/Webhook đối tác.** | `ShipmentStatusHistory` ghi nhận cả sự kiện webhook tự động từ Carrier API (GHTK/GHN) và các thao tác cập nhật của điều phối viên (`ChangedBy` ghi rõ nguồn cập nhật). |
| **SHIP-Q07** | Phí vận chuyển được quản lý như thế nào? | **`SalesOrder` lưu tổng phí tính cho khách; `Shipment` lưu phần phí phân bổ thực tế.** | Phân định ranh giới doanh thu vận chuyển (`SalesOrder.ShippingFee`) và chi phí logistics thực tế trả cho đối tác (`Shipment.ShippingFee`), phục vụ đối soát công nợ. |

---

## 10. Quyết định Thiết kế Subject Area 08: Promotion (PROMO-Q01 đến PROMO-Q08)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **PROMO-Q01** | Phạm vi ưu đãi cấp sản phẩm? | **Áp dụng theo SKU (`ProductVariant`) qua bảng liên kết `PromotionProduct`.** | Chuẩn hóa CSDL cao nhất, kiểm soát chính xác chiết khấu từng biến thể, tránh nhập nhằng định giá giữa các biến thể của sản phẩm. |
| **PROMO-Q02** | Biểu diễn điều kiện khuyến mãi? | **Sử dụng các loại điều kiện cấu hình sẵn (`ConditionType`), cấm biểu thức tự do.** | Tối ưu hóa tốc độ đánh giá bằng Index SQL Server, loại bỏ rủi ro bảo mật injection và nghẽn hiệu năng của bộ dynamic expression parsing. |
| **PROMO-Q03** | Cơ chế cấp voucher cấp đơn / vận chuyển? | **Hỗ trợ cả Voucher dùng chung toàn sàn và Voucher riêng cho từng khách hàng (cấp `'ORDER'` và `'SHIPPING'`).** | CSDL linh hoạt: Voucher dùng chung để trống `AssignedCustomerID`; Voucher cá nhân gán `AssignedCustomerID` bắt buộc. |
| **PROMO-Q04** | Giới hạn lượt sử dụng voucher? | **Giới hạn ở cả cấp tổng chương trình (`MaxUsage`) và cấp mỗi khách hàng (`MaxUsagePerCustomer`).** | Ngăn chặn gian lận (Voucher Abuse) và tài khoản bot gom vét sạch mã ưu đãi của chương trình. |
| **PROMO-Q05** | Thời điểm ghi nhận lượt sử dụng? | **Chỉ ghi nhận chính thức (`CONSUMED`, tăng `UsedCount`) khi thanh toán thành công.** | Giữ chỗ tạm thời (`PENDING`) khi đơn ở `PENDING_PAYMENT` để tránh bán vượt mã. Chỉ chốt tiêu thụ khi đơn sang `CONFIRMED`. |
| **PROMO-Q06** | Xử lý voucher khi đơn hủy/hoàn? | **Hoàn lại lượt sử dụng (`UsageStatus = 'REVERSED'`) nếu voucher còn trong thời hạn.** | Bảo vệ quyền lợi khách hàng khi đơn bị hủy hợp lệ trước khi đóng gói hoặc hoàn trả toàn bộ. Hết hạn chương trình thì không hoàn lại. |
| **PROMO-Q07** | Chương trình dành riêng cho khách? | **Hỗ trợ qua thực thể kết hợp `PromotionTargetCustomer` ($N:N$).** | Phục vụ các chiến dịch chăm sóc khách hàng VIP, ưu đãi ngày sinh nhật, hoặc phát voucher đền bù dịch vụ cho từng tài khoản. |
| **PROMO-Q08** | Chính sách sửa chương trình đã dùng? | **Tuyệt đối không sửa thông tin ảnh hưởng ưu đãi; quản trị viên tạo phiên bản mới.** | Bảo toàn bất biến kiểm toán tài chính (Financial Auditability). CSDL quản lý qua `VersionNumber` và `SupersedesPromotionID`. |

---

## 11. Quyết định Thiết kế Subject Area 09: Payment & Refund (PAY-Q01 đến PAY-Q10)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **PAY-Q01** | Số lượng phương thức thanh toán trên một đơn? | **Một phương thức duy nhất cho mỗi đơn hàng.** | Đơn giản hóa đối soát công nợ, chuẩn hóa quan hệ $1:1$ giữa `SalesOrder` và `Payment`. Tránh chia nhỏ dòng tiền gây phân mảnh và phức tạp đối soát. |
| **PAY-Q02** | Có hỗ trợ phương thức COD không? | **Có, hỗ trợ song song COD và cổng thanh toán trực tuyến.** | Phù hợp thói quen TMĐT Việt Nam; phân định luồng thanh toán trước (Online) và thanh toán sau (COD). |
| **PAY-Q03** | Cơ chế thanh toán lại khi giao dịch thất bại? | **Cho phép thanh toán lại; mỗi lần thử tạo một `PaymentTransaction` mới.** | Đơn giữ `PENDING_PAYMENT` trong thời gian timeout, không bắt khách đặt lại đơn mới khi rớt mạng hoặc sai OTP. Giao dịch cũ lưu vết phục vụ đối soát. |
| **PAY-Q04** | Hỗ trợ hoàn tiền một phần (Partial Refund)? | **Có, hỗ trợ hoàn tiền toàn phần, một phần và hoàn nhiều lần.** | Ràng buộc bất biến: $\sum \text{RefundAmount} \le \text{RemainingRefundable} = \text{AmountPaid} - \text{AmountRefunded}$. Phục vụ đổi trả từng món trong đơn nhiều sản phẩm. |
| **PAY-Q05** | Thẩm quyền phê duyệt yêu cầu hoàn tiền? | **Nhân viên có thẩm quyền phê duyệt (`ApprovedBy NOT NULL`).** | Kiểm soát nội bộ (Internal Control), chống thất thoát dòng tiền ra khỏi doanh nghiệp. Không tự động hoàn tiền khi chưa thẩm định. |
| **PAY-Q06** | Quy trình hoàn tiền cho đơn hàng COD? | **Hoàn qua chuyển khoản ngân hàng (Bank Transfer).** | Thu thập thông tin STK khách hàng, thực hiện lệnh chi và lưu mã giao dịch đối soát tại `RefundTransaction`. |
| **PAY-Q07** | Lưu trữ phản hồi từ cổng thanh toán? | **Lưu chi tiết mã phản hồi, mã tham chiếu và thông điệp cổng.** | Làm bằng chứng pháp lý đối soát cuối ngày và giải quyết khiếu nại/tra soát (Chargeback). CSDL lưu `GatewayTransactionID`, `GatewayResponseCode`. |
| **PAY-Q08** | Điều kiện chuyển đơn sang `CONFIRMED`? | **Đơn online: khi thanh toán thành công; Đơn COD: khi duyệt hợp lệ.** | Đồng bộ triệt để với `SALES-Q02`. Đảm bảo kho chỉ xuất hàng khi tiền đã thu chắc chắn hoặc đơn COD tin cậy. Xác nhận đơn COD không đồng nghĩa với đã thu tiền. |
| **PAY-Q09** | Ghi nhận phí dịch vụ của cổng thanh toán? | **Có, lưu trường `TransactionFee` trên `PaymentTransaction`.** | Hỗ trợ đối soát số tiền thực nhận (Net Settlement) và tính toán chính xác biên lợi nhuận ròng của đơn hàng. |
| **PAY-Q10** | Đối soát tiền thu hộ COD cho đơn đa kiện (Split Shipment)? | **Tách nghĩa vụ khách hàng (`PaymentStatus`) khỏi công nợ hãng vận chuyển qua mô hình COD 3 lớp.** | Ghi nhận thu hộ từng kiện `CODCollection` (E77) $\to$ bảng kê chuyển tiền `CODRemittance` (E78) $\to$ khớp đối soát `CODReconciliation` (E79), đồng bộ `Payment.CarrierSettlementStatus` (`CORR-07`). |

---

## 12. Quyết định Thiết kế Subject Area 10: Return & Exchange (RET-Q01 đến RET-Q08)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **RET-Q01** | Đối tượng được phép tạo yêu cầu trả hàng? | **Khách hàng tự tạo hoặc Nhân viên (CSKH) tạo thay khách hàng.** | CSDL lưu `CustomerID` (chủ đơn) và `CreatedByAccountID` (tài khoản người tạo), hỗ trợ tiếp nhận RMA đa kênh từ tổng đài. |
| **RET-Q02** | Thời hạn được phép gửi yêu cầu trả hàng? | **Cấu hình theo số ngày kể từ thời điểm giao hàng thành công của từng kiện hàng (`Shipment.DeliveredAt`).** | Liên kết qua `ReturnShipmentAllocation` (E76): `DATEDIFF(day, Shipment.DeliveredAt, ReturnRequest.RequestedAt) <= PolicyReturnDays`. Hỗ trợ chính xác đơn Split Shipment giao lệch ngày (CORR-10). |
| **RET-Q03** | Có cho phép trả một phần số lượng dòng hàng? | **Có, cho phép trả một phần và trả nhiều lần cho cùng dòng hàng.** | Ràng buộc bất biến: $\sum \text{ReturnedQty} \le \text{OrderedQty} - \text{CancelledQty}$, dùng chung hàm tích lũy đơn điệu $\Delta F = F(a+b) - F(a)$ để không lệch 1 đồng VNĐ (CORR-03). |
| **RET-Q04** | Quy trình phê duyệt yêu cầu trả hàng? | **Nhân viên CSKH/Kiểm duyệt phải phê duyệt trước khi khách gửi hàng.** | Ngăn chặn việc gửi trả hàng sai chính sách, lãng phí chi phí logistics ngược. Trạng thái bắt buộc: `PENDING_APPROVAL` $\to$ `APPROVED`. |
| **RET-Q05** | Phân loại hàng trả về sau tiếp nhận & kiểm định tại kho? | **Tiếp nhận vào `QUARANTINED`, sau đó định đoạt 3 hướng: `SELLABLE`, `QUARANTINED`, hoặc `DISPOSED`.** | Khi ký `ReturnReceipt` (E62), ghi `RETURN_RECEIPT_IN` đưa hàng vào bể cách ly `QUARANTINED`. Sau giám định `ReturnLineAssessment` (E72): `SELLABLE` nhả sang tồn bán (cập nhật WAC theo `OutboundUnitCost`), `QUARANTINED` giữ cách ly, `DISPOSED` xuất tiêu hủy `SCRAP_OUT` (CORR-01, CORR-02). |
| **RET-Q06** | Số lượng kho tiếp nhận cho một yêu cầu? | **Mỗi yêu cầu RMA chỉ định ĐÚNG MỘT kho tiếp nhận (`WarehouseID NOT NULL`).** | Tránh chia tách phân tán hàng hoàn. Mọi phiếu `ReturnReceipt` của yêu cầu phải tiếp nhận tại đúng kho đã chỉ định. |
| **RET-Q07** | Thời điểm tạo yêu cầu hoàn tiền (`Refund`)? | **Sau khi kiểm định và đánh giá lỗi được phê duyệt qua `ReturnLineAssessment` (E72).** | Tách rời khỏi kiểm định kho: Kích hoạt tạo `Refund` ngay khi có dòng đánh giá (`ReturnLineAssessment.RefundEligibilityStatus = 'APPROVED'`) được phê duyệt. Hỗ trợ hoàn tiền từng mốc theo số lượng thực nhận được chấp thuận (CORR-01). |
| **RET-Q08** | Lưu trữ lịch sử trạng thái yêu cầu trả hàng? | **Có, lưu vết toàn diện qua thực thể `ReturnStatusHistory`.** | Đảm bảo tính kiểm toán và đo lường SLA xử lý RMA: lưu `PreviousStatus`, `NewStatus`, `ChangedAt`, `ChangedByAccountID` và `ChangeReason`. |

---

## 13. Quyết định Thiết kế Subject Area 11: Audit & Reporting (AR-Q01 đến AR-Q07)

| ID | Vấn đề thiết kế | Quyết định chốt | Giải thích kỹ thuật & Tác động CSDL |
| :---: | :--- | :--- | :--- |
| **AR-Q01** | Phạm vi ghi nhật ký kiểm toán (AuditLog)? | **Chỉ ghi các thao tác quan trọng và nhạy cảm; không ghi log mọi CRUD/đọc.** | Bảo vệ hiệu năng I/O cho OLTP. Các hành động theo dõi: Duyệt đơn, hủy đơn, kiểm kê kho, chỉnh tồn, duyệt hoàn tiền, đổi giá SKU, cấp quyền Role/Permission. |
| **AR-Q02** | Mức độ lưu chi tiết thay đổi dữ liệu? | **Lưu OldValue và NewValue cho các trường được theo dõi (`AuditLogDetail`).** | Tối ưu hóa dung lượng lưu trữ, phục vụ hiển thị so sánh (Diff Viewer). Tuyệt đối cấm lưu mật khẩu, OTP, token bảo mật vào nội dung log. |
| **AR-Q03** | Có cho phép chỉnh sửa hoặc xóa AuditLog? | **Tuyệt đối cấm chỉnh sửa/xóa qua chức năng nghiệp vụ (`Append-Only / WORM`).** | Tuân thủ nguyên tắc an ninh kiểm toán SOX/PCI-DSS/ISO 27001. Dữ liệu chỉ ghi, cấm sửa, cấm xóa bởi bất kỳ người dùng hay quản trị viên nào. |
| **AR-Q04** | Thời gian lưu trữ AuditLog? | **Cấu hình thời hạn lưu trữ hoạt động (12–24 tháng), sau đó chuyển vào Cold Archive.** | Sử dụng Table Partitioning trên SQL Server để duy trì hiệu năng chỉ mục tra cứu nhanh, sau đó di chuyển log cũ sang kho lưu trữ lạnh. |
| **AR-Q05** | Cơ chế xây dựng báo cáo quản trị? | **Kết hợp: Báo cáo vận hành truy vấn trực tiếp; Báo cáo tài chính dùng `ReportSnapshot`.** | Báo cáo vận hành đọc trực tiếp từ Views tối ưu; báo cáo tài chính/kế toán dùng snapshot để chốt cố định số liệu kỳ kế toán, tránh biến động hồi tố. |
| **AR-Q06** | Lưu vết lịch sử chạy và xuất báo cáo? | **Có, ghi nhận lịch sử qua thực thể `ReportExecution`.** | Phục vụ kiểm toán an ninh dữ liệu (ai đã trích xuất dữ liệu ra file) và hỗ trợ tác vụ xuất file dung lượng lớn bất đồng bộ lưu tại `ResultLocation`. |
| **AR-Q07** | Phân quyền truy cập và sử dụng báo cáo? | **Phân quyền ma trận qua thực thể kết hợp `ReportPermission` (Role - Report).** | Tích hợp trực tiếp với RBAC của Subject Area 04, phân tách rõ 3 quyền hạn: `CanView` (xem), `CanExport` (xuất file) và `CanCreateSnapshot` (chốt kỳ). |

---

## 14. 10 Quyết định Hiệu chỉnh Kiến trúc Chuyên sâu Pre-M3 (CORR-01 đến CORR-10)

Căn cứ biên bản thẩm định kiến trúc độc lập [08-Architectural-Correction-Log.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/08-Architectural-Correction-Log.md), toàn bộ 10 quyết định sửa đổi nền tảng sau đây đã được phê duyệt và áp dụng chính thức:

| Mã Hiệu chỉnh | Vấn đề Kiến trúc | Quyết định Chuẩn hóa | Tác động CSDL & Thực thi SQL Server |
| :---: | :--- | :--- | :--- |
| **CORR-01** | Tách quyền hoàn tiền khỏi tình trạng kho & Đổi hàng | Quyền hoàn tiền độc lập với `InventoryDisposition`. Bổ sung **`ReturnLineAssessment` (E72)** phiên bản hóa; đổi hàng sinh đơn bù **`EXCHANGE_REPLACEMENT`**. | Đánh giá `FaultParty` (`VENDOR_FAULT`/`CARRIER_FAULT`/`CUSTOMER_FAULT`/`NO_FAULT`), `RefundEligibilityStatus`, `ResolutionType` (`REFUND`/`EXCHANGE`/`REPAIR`/`NO_ACTION`). Filtered Index `(ReturnReceiptLineID, AssessmentGroupNo) WHERE IsCurrent = 1`. |
| **CORR-02** | Quản lý tồn kho cách ly & hư hỏng | Phân loại 3 trạng thái phẩm chất tại kho: `SELLABLE`, `QUARANTINED`, `DAMAGED` (nhận hàng RMA vào `QUARANTINED` qua `RETURN_RECEIPT_IN`, tiêu hủy là `SCRAP_OUT`). | Bộ 5 CHECK constraints trên `InventoryBalance` (E11); `InventoryTransactionLine` (E16) quản lý `FromStockStatus`, `ToStockStatus` và bộ 4 delta. |
| **CORR-03** | Giá đã gồm VAT, LRM, $\Delta F$ & Thu hồi Voucher | Giá B2C **VAT-Inclusive**; phân bổ LRM; công thức $\Delta\text{Refund} = F(a+b) - F(a)$ dùng chung cho `CancelledQty` + `RefundedQty`; thu hồi voucher (**Clawback**) nếu lỗi khách làm rớt `MinOrderAmount`. | Khóa `UPDLOCK` trên `OrderLine` (E41); lưu `LineGrossAmount`, `LineProductDiscount`, `AllocatedOrderDiscount`, `LineNetAmount`, `TaxAmount`, `CancelledQty`, `RefundedQty`, `AccumulatedRefundedAmount`. |
| **CORR-04** | Ràng buộc bắt buộc 1:1, Revoke DML & TPT Account | Thu hồi quyền DML trực tiếp của `retailcore_app`, bắt buộc dùng Boundary SP nguyên tử. Kế thừa `Account` dùng **Table-Per-Type (TPT)**. | `Account(AccountID, AccountType)` XOR giữa `Customer(AccountID PK, FK)` và `Employee(AccountID PK, FK)`. |
| **CORR-05** | Idempotency Record & Payment Webhook Log | Chống gửi lặp qua **`IdempotencyRecord` (E74)** và tiếp nhận callback qua **`PaymentWebhookLog` (E71)**. | `UNIQUE (AccountID, OperationType, IdempotencyKey)` trên E74; `UX_PaymentWebhookLog_ProviderEvent ON (PaymentMethodID, ProviderEventID, Environment)` trên E71. |
| **CORR-06** | Độc lập 4 máy trạng thái giao dịch | Tách bạch tuyệt đối `OrderStatus`, `PaymentStatus`, `ShipmentStatus`, `ReturnStatus` và `PaymentTransaction.TransactionStatus`. | Đơn COD chuyển `CONFIRMED` khi hợp lệ dù `PaymentStatus = 'PENDING'`. Đóng hồ sơ hành chính qua `SalesOrder.ClosedAt`. |
| **CORR-07** | Đối soát COD 2 đầu & Cụm COD 3 lớp | Tách nghĩa vụ khách hàng (`PAID`) khỏi công nợ hãng vận chuyển; bổ sung **`CODCollection` (E77), `CODRemittance` (E78), `CODReconciliation` (E79)** và **`DeliveryReturnReceipt` (E75)**. | Hỗ trợ đối soát thu hộ chi tiết đến từng kiện hàng trong đơn Split Shipment đa kho/đa hãng vận chuyển. |
| **CORR-08** | Nguồn tiền hoàn & Giá vốn Moving WAC | Đặt `OriginalPaymentTransactionID` tại `RefundTransaction` (E59); tính WAC trên `AcceptedQty` của bể `SELLABLE`. | Snapshot `ShipmentLine.OutboundUnitCost` và `WarehouseTransferLine.UnitCost`; chênh lệch trả NCC (`SupplierReturn`) hạch toán vào `CostVarianceAccountCode`. |
| **CORR-09** | Chuẩn hóa JSON Snapshot biến thể & `Brand` | Tách bảng danh mục **`Brand` (E73)**; lưu `OrderLine.VariantAttributesSnapshot` dạng **JSON Array** có `ISJSON(...) = 1`. | Cấu trúc thống nhất: `[{"attributeId": 1, "attributeCode": "SIZE", "attributeName": "Kích cỡ", "valueId": 105, "valueCode": "XL", "displayValue": "Size XL"}]`. |
| **CORR-10** | Lịch sử giữ hàng $1:0..N$ & Hạn RMA theo kiện | `Allocation` $\leftrightarrow$ `ReservationLine` là $1 : 0..N$; bổ sung **`ReturnShipmentAllocation` (E76)** nối dòng RMA với dòng kiện giao. | `UX_StockReservationLine_ActiveAlloc ON (AllocationID) WHERE ReservationLineStatus = 'ACTIVE'`; kiểm tra `RequestedAt <= Shipment.DeliveredAt + PolicyReturnDays`. |
