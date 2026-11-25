# 06. BUSINESS RULES – BỘ QUY TẮC NGHIỆP VỤ CHUẨN HÓA (PHIÊN BẢN v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis (Reverse-Synced với M2 & `GEMINI.md`)
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT (**215 Quy tắc Nghiệp vụ** định danh duy nhất trên **11 Nhóm Phân hệ**)
* **Database target:** Microsoft SQL Server 2022+
* **Kế thừa & Đồng bộ:** Quyết định nền tảng `BD-01` đến `BD-07` và 10 Hiệu chỉnh Kiến trúc `CORR-01` đến `CORR-10` ([GEMINI.md](../../../GEMINI.md)).

---

## 6.1. Product Catalog (`BR-PR01` – `BR-PR15`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PR01** | Mỗi sản phẩm thương mại (`Product`) phải có mã định danh tự nhiên `ProductCode` duy nhất toàn hệ thống. |
| **BR-PR02** | Mỗi biến thể bán hàng (`ProductVariant`) phải có mã `SKU` duy nhất toàn hệ thống. |
| **BR-PR03** | Một sản phẩm có thể có một hoặc nhiều SKU; sản phẩm đơn giản không phân biến thể tự động có 1 SKU mặc định rỗng thuộc tính (`PC-Q03`). |
| **BR-PR04** | Mỗi SKU bắt buộc thuộc duy nhất một sản phẩm thương mại (`ProductVariant.ProductID NOT NULL`). |
| **BR-PR05** | Một sản phẩm có thể thuộc nhiều danh mục (`Category`) và một danh mục chứa nhiều sản phẩm ($N:N$ qua `ProductCategory`). Cây danh mục đệ quy không được phép tự tham chiếu hoặc tạo chu trình lặp. |
| **BR-PR06** | Giá bán niêm yết (`SellingPrice`) không được âm và phải là số nguyên đồng VNĐ (`SellingPrice = FLOOR(SellingPrice)`). Giá gốc so sánh (`CompareAtPrice`) nếu có phải lớn hơn hoặc bằng `SellingPrice`. |
| **BR-PR07** | Tuyệt đối không xóa vật lý (`No Hard Delete`) sản phẩm, thương hiệu, danh mục hoặc SKU đã phát sinh giao dịch; chỉ chuyển trạng thái ngừng kinh doanh (`ARCHIVED`/`DISCONTINUED`, `IsActive = 0`). |
| **BR-PR08** | SKU ở trạng thái `INACTIVE` hoặc `DISCONTINUED` không được phép đặt mua trong đơn hàng mới. |
| **BR-PR09** | Thông tin sản phẩm, đơn giá, thuế suất GTGT và thuộc tính biến thể tại thời điểm đặt đơn phải được snapshot bất biến trên `OrderLine`. |
| **BR-PR10** | Mỗi sản phẩm có tối đa một danh mục chính (`IsPrimary = 1`); khi chuyển sang trạng thái `PUBLISHED`, bắt buộc phải có **đúng một** danh mục chính và ít nhất một SKU đang `ACTIVE` (`BD-01`). |
| **BR-PR11** | Thương hiệu hàng hóa được chuẩn hóa qua danh mục `Brand` độc lập (`BrandCode`, `Slug` duy nhất) để phục vụ truy vấn và Faceted Filter không dị thường. |
| **BR-PR12** | Không được tồn tại hai SKU thuộc cùng một `Product` mang tổ hợp giá trị thuộc tính `VARIANT_DEFINING` giống hệt nhau. |
| **BR-PR13** | Một thuộc tính (`Attribute`) khi gán cho sản phẩm qua `ProductAttribute` chỉ nhận duy nhất một vai trò: `'VARIANT_DEFINING'` (cấu thành SKU) hoặc `'DESCRIPTIVE'` (mô tả chung cấp sản phẩm) (`BD-06`). |
| **BR-PR14** | Giá trị thuộc tính gán trên `ProductAttributeValue` và `VariantAttributeValue` bắt buộc thuộc đúng `AttributeID` sở hữu giá trị đó trong từ điển `AttributeValue` (cấm nhiễm chéo thuộc tính). Với `VARIANT_DEFINING`, giá trị của SKU bắt buộc nằm trong tập Whitelist `ProductAttributeValue` của sản phẩm cha. |
| **BR-PR15** | Mỗi sản phẩm khai báo mức thuế suất GTGT hợp lệ (`VatRate` $\in \{0\%, 5\%, 8\%, 10\%\}$) và mỗi SKU khai báo khối lượng vật lý (`WeightGrams > 0`) cùng kích thước đóng gói phục vụ tính phí vận chuyển Zero-Trust. |

---

## 6.2. Warehouse & Inventory (`BR-IN01` – `BR-IN25`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-IN01** | Doanh nghiệp vận hành mạng lưới nhiều kho vật lý (`Warehouse`), định danh duy nhất bằng `WarehouseCode`. |
| **BR-IN02** | Số dư tồn kho tức thời (`InventoryBalance`) được quản lý độc lập theo từng cặp `(WarehouseID, VariantID)`. |
| **BR-IN03** | Chỉ tồn tại đúng 3 trạng thái chất lượng tồn kho vật lý trong kho: `SELLABLE` (đủ chuẩn bán), `QUARANTINED` (cách ly chờ giám định), và `DAMAGED` (hư hỏng chờ xử lý) (`CORR-02`). |
| **BR-IN04** | Tiêu hủy hàng hỏng là nghiệp vụ xuất khỏi kho (`SCRAP_OUT`), tuyệt đối không tồn tại trạng thái chất lượng `DISPOSED` nằm trong số dư kho (`CORR-02`). |
| **BR-IN05** | Toàn bộ 4 cột số lượng trên `InventoryBalance` (`OnHandQty`, `ReservedQty`, `QuarantinedQty`, `DamagedQty`) bắt buộc không âm ($\ge 0$). |
| **BR-IN06** | Bất biến sức chứa vật lý và tồn khả dụng (`CORR-02`): $\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty} \ge 0$ và $\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty} \ge 0$. |
| **BR-IN07** | Kho ngừng hoạt động (`WarehouseStatus <> 'ACTIVE'`) không được nhận phân bổ đơn hàng (`OrderWarehouseAllocation`) mới. |
| **BR-IN08** | Tổng số lượng phân bổ hiệu lực (`AllocatedQty`) qua các kho của một dòng đơn hàng phải bằng đúng số lượng đặt mua còn hiệu lực (`OrderedQty - CancelledQty`) của dòng đó (`BD-03`). |
| **BR-IN09** | Khi tạo đơn hàng, hệ thống bắt buộc khóa giữ tồn kho (`StockReservation` / `StockReservationLine`) theo nguyên tắc **All-or-Nothing**: thiếu bất kỳ SKU nào thì hủy bỏ toàn bộ giao dịch đặt đơn (`BD-04`). |
| **BR-IN10** | Mỗi lượt giữ hàng có thời điểm hết hạn `ExpiresAt` (**15 phút** chuẩn; rút ngắn còn **5 phút** đối với đơn chứa SKU chiến dịch Flash Sale) và gắn với dòng phân bổ `AllocationID` theo quan hệ $1:0..N$; tại một thời điểm mỗi `AllocationID` chỉ có tối đa 1 dòng `ReservationLineStatus = 'ACTIVE'` (`CORR-10`). |
| **BR-IN11** | Để triệt tiêu Deadlock khi giữ hàng đồng thời cao (Flash Sale), thứ tự khóa dòng `InventoryBalance` bắt buộc sắp xếp cố định theo `VariantID ASC, WarehouseID ASC`. |
| **BR-IN12** | Khi lượt giữ hàng hết hạn thanh toán hoặc đơn bị hủy trước xuất kho, hệ thống giải phóng `ReservedQty` và giữ nguyên `OnHandQty` vật lý. |
| **BR-IN13** | Khi bàn giao kiện hàng cho hãng vận chuyển (`SHIPMENT_OUT`), hệ thống giảm đồng thời cả `OnHandQty` và `ReservedQty` theo số lượng thực xuất (`BD-07`). |
| **BR-IN14** | Hàng đang trên đường giao (In-Transit) được theo dõi qua `Shipment`, không thuộc `OnHandQty` hay `ReservedQty` của kho xuất (`BD-07`). |
| **BR-IN15** | Khi giao hàng thất bại, tuyệt đối chưa tăng tồn kho. Chỉ khi kiện hoàn thực tế về bến nhập và thủ kho ký biên bản `DeliveryReturnReceipt` (`POSTED`), hệ thống mới ghi `DELIVERY_RETURN_IN` và tăng `OnHandQty` (`BD-07`). |
| **BR-IN16** | Nếu đơn hàng giao lại sau khi hoàn kho, hệ thống phải tạo phân bổ và giữ hàng mới theo tồn kho hiện hành trước khi xuất lại. |
| **BR-IN17** | Mọi biến động làm thay đổi số dư vật lý hoặc chuyển đổi chất lượng kho bắt buộc ghi nhận qua sổ cái bất biến `InventoryTransaction` và `InventoryTransactionLine` (lưu đủ 4 delta và `FromStockStatus \to ToStockStatus`). |
| **BR-IN18** | Chứng từ sổ cái kho sau khi `POSTED` là bất biến (Append-Only); mọi điều chỉnh sai sót phải thực hiện bằng chứng từ điều chỉnh/đảo mới. |
| **BR-IN19** | Quy trình điều chuyển nội bộ giữa hai kho (`WarehouseTransfer`) thực hiện 2 bước tách biệt: `TRANSFER_OUT` giảm `OnHandQty` kho nguồn (kèm snapshot `UnitCost` theo WAC kho nguồn) và `TRANSFER_IN` tăng `OnHandQty` kho đích theo số lượng thực nhận. |
| **BR-IN20** | Phiếu kiểm kê kho (`StockCount`) hoàn tất đếm (`COMPLETED`) không tự động sửa số dư kho, bắt buộc lưu rõ `SystemQty`, `CountedQty` và `VarianceQty`. |
| **BR-IN21** | Chênh lệch kiểm kê chỉ được hạch toán sinh giao dịch `STOCK_ADJUSTMENT` sau khi cấp quản lý phê duyệt; người tạo phiếu đếm không được tự phê duyệt phiếu của chính mình (`ApprovedBy <> CreatedBy`). |
| **BR-IN22** | Giá vốn tồn kho áp dụng phương pháp Bình quân Gia quyền Liên tục (**Moving WAC**) trên lượng hàng đạt chuẩn `SELLABLE` tại từng kho (`CurrentAverageCost`) (`CORR-08`). |
| **BR-IN23** | Hàng cách ly (`QUARANTINED`) và hư hỏng (`DAMAGED`) được hạch toán riêng để không làm pha loãng giá vốn bình quân của lô hàng `SELLABLE` mới nhập. |
| **BR-IN24** | Khi xuất kho bán hàng hoặc điều chuyển, giá vốn WAC tại thời điểm xuất phải được snapshot vào chứng từ xuất (`ShipmentLine.OutboundUnitCost`, `WarehouseTransferLine.UnitCost`). |
| **BR-IN25** | Khi số lượng tồn kho về $0$, tổng giá trị tồn kho bằng $0$ và giữ `CurrentAverageCost` gần nhất làm mốc tham chiếu cho đến khi phát sinh lô nhập mới. |

---

## 6.3. Procurement (`BR-PO01` – `BR-PO10`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PO01** | Mỗi nhà cung cấp (`Supplier`) có mã `SupplierCode` duy nhất; danh mục SKU cung ứng quản lý qua `SupplierProduct(SupplierID, VariantID)`. |
| **BR-PO02** | Một đơn mua hàng (`PurchaseOrder`) chỉ định **đúng một** kho nhận cố định (`DestinationWarehouseID`) và có ít nhất một dòng `PurchaseOrderLine` với `OrderedQty > 0` (`PR-Q01`). |
| **BR-PO03** | Đơn mua hàng bắt buộc được cấp có thẩm quyền phê duyệt (`ApprovedBy <> CreatedBy`) trước khi gửi nhà cung cấp và tiếp nhận hàng (`PR-Q06`). |
| **BR-PO04** | Một PO có thể tiếp nhận thành nhiều đợt qua nhiều phiếu `GoodsReceipt`; kho tiếp nhận trên `GoodsReceipt` bắt buộc trùng `DestinationWarehouseID` của PO (`PR-Q02`). |
| **BR-PO05** | Trên mỗi dòng tiếp nhận `GoodsReceiptLine`, số lượng kiểm đếm thỏa bất biến $\text{ReceivedQty} = \text{AcceptedQty} + \text{RejectedQty}$ (`PR-Q04`). |
| **BR-PO06** | Lũy kế số lượng tiếp nhận qua các đợt không được vượt số lượng đặt mua trên dòng PO: $\sum \text{ReceivedQty} \le \text{OrderedQty}$ (`PR-Q03`). |
| **BR-PO07** | Chỉ phần số lượng đạt chuẩn `AcceptedQty` trên phiếu nhập đã chốt (`POSTED`) mới sinh giao dịch `PURCHASE_RECEIPT`, tăng `OnHandQty` và cập nhật lại giá vốn bình quân `CurrentAverageCost` (`CORR-08`). |
| **BR-PO08** | Phần `RejectedQty` không đạt QC được ghi nhận kèm `RejectionReason`, tuyệt đối không cộng vào tồn kho bán được. |
| **BR-PO09** | Xuất trả hàng đã nhập kho cho nhà cung cấp (`SupplierReturn` / `SupplierReturnLine`) phải tham chiếu dòng `GoodsReceiptLineID` gốc với tổng số lượng trả không vượt lượng thực nhận còn lại (`PR-Q05`). |
| **BR-PO10** | Khi xuất trả NCC (`SUPPLIER_RETURN_OUT`), giá trị tồn kho giảm theo **WAC hiện hành** tại kho xuất, công nợ thu hồi tính theo giá mua PO gốc, và phần chênh lệch hạch toán vào tài khoản điều chỉnh giá vốn `CostVarianceAccountCode` (`CORR-08`). |

---

## 6.4. Customer & Identity (`BR-ID01` – `BR-ID15`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-ID01** | Mỗi tài khoản `Account` bắt buộc cung cấp ít nhất một trong hai thông tin định danh: `Email` hoặc `Phone`. |
| **BR-ID02** | `Email` nếu được cung cấp phải duy nhất toàn hệ thống (qua Filtered Unique Index `WHERE Email IS NOT NULL`). |
| **BR-ID03** | `Phone` nếu được cung cấp phải duy nhất toàn hệ thống (qua Filtered Unique Index `WHERE Phone IS NOT NULL`). |
| **BR-ID04** | Phân cấp tài khoản tuân thủ mô hình Table-Per-Type (TPT) loại trừ XOR (`CORR-04`): `Account(AccountID, AccountType)` chỉ liên kết với **đúng một** hồ sơ con là `Customer(AccountID)` hoặc `Employee(AccountID)`, cấm kiêm nhiệm cả hai. |
| **BR-ID05** | Khởi tạo tài khoản bắt buộc thực thi nguyên tử qua Boundary Stored Procedure tạo đồng thời `Account` và bảng con tương ứng; thu hồi quyền DML trực tiếp (`CORR-04`). |
| **BR-ID06** | Tài khoản bị khóa hoặc vô hiệu hóa (`AccountStatus <> 'ACTIVE'`) không được phép đăng nhập hay tạo giao dịch mới. |
| **BR-ID07** | Một `Account` có thể được cấp nhiều vai trò (`Role`) qua `AccountRole` không trùng lặp. |
| **BR-ID08** | Một `Role` tập hợp nhiều quyền hạn (`Permission`) qua `RolePermission` không trùng lặp; người dùng chỉ được thực thi thao tác thuộc tập quyền hiệu lực. |
| **BR-ID09** | Hồ sơ nhân sự `Employee` có mã nhân viên `EmployeeCode` duy nhất toàn doanh nghiệp. |
| **BR-ID10** | Mỗi `Customer` có thể lưu nhiều địa chỉ trong sổ địa chỉ `CustomerAddress`. |
| **BR-ID11** | Mỗi `Customer` có tối đa một địa chỉ mặc định (`IsDefault = 1`, cưỡng chế bằng Filtered Unique Index). |
| **BR-ID12** | Địa chỉ hợp lệ đầu tiên của khách hàng tự động được đặt làm mặc định; khi vô hiệu hóa địa chỉ mặc định, phải chỉ định địa chỉ hoạt động khác thay thế nếu còn. |
| **BR-ID13** | Mọi yêu cầu xác thực OTP/link (`AccountVerification`) phải gắn với `Account`, lưu giá trị mã xác thực dạng băm và có thời hạn `ExpiresAt`. |
| **BR-ID14** | Yêu cầu xác minh đã hết hạn hoặc đã sử dụng (`VERIFIED`) không được phép tái sử dụng. |
| **BR-ID15** | Thay đổi `Email` hoặc `Phone` đăng nhập bắt buộc hoàn tất xác minh thông tin mới trước khi chuyển đổi chính thức. |

---

## 6.5. Shopping (`BR-SH01` – `BR-SH15`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SH01** | Một `Customer` có thể có nhiều `ShoppingCart` trong lịch sử nhưng tối đa **đúng một** giỏ hàng ở trạng thái `ACTIVE` tại mọi thời điểm. |
| **BR-SH02** | Mỗi `ShoppingCart` thuộc quyền sở hữu của đúng một `Customer` (`CustomerID NOT NULL`). |
| **BR-SH03** | Một `ShoppingCart` chứa danh sách mặt hàng qua `CartItem`. |
| **BR-SH04** | Mỗi `CartItem` tham chiếu trực tiếp tới đúng một biến thể `ProductVariant` (`VariantID`). |
| **BR-SH05** | Mỗi SKU chỉ xuất hiện tối đa một dòng trong cùng một `ShoppingCart` (`UNIQUE (CartID, VariantID)`). |
| **BR-SH06** | Khi khách thêm trùng SKU đã có trong giỏ `ACTIVE`, hệ thống tự động cộng dồn `Quantity`. |
| **BR-SH07** | Số lượng `Quantity` trên mỗi `CartItem` bắt buộc lớn hơn $0$. |
| **BR-SH08** | Không cho phép thêm mới SKU đã ngừng kinh doanh vào giỏ; nếu SKU đang nằm sẵn trong giỏ mới chuyển sang hết hàng/ngừng bán, giữ dòng `CartItem` kèm cảnh báo và chặn Checkout cho đến khi khách bỏ chọn/xóa. |
| **BR-SH09** | Không lưu cứng giá bán trong `CartItem`; giao diện giỏ hàng luôn hiển thị theo giá niêm yết hiện hành của `ProductVariant`. |
| **BR-SH10** | Đơn giá, khuyến mãi và thuế GTGT được tính toán lại từ CSDL (Zero-Trust) và đóng băng snapshot tại thời điểm tạo đơn hàng. |
| **BR-SH11** | Giỏ hàng lưu tập trung trên CSDL Server, đồng bộ tức thời giữa các thiết bị của cùng khách hàng. |
| **BR-SH12** | Khi giao dịch tạo đơn và giữ tồn kho thành công 100%, `ShoppingCart` chuyển sang trạng thái `CONVERTED` trong cùng giao dịch nguyên tử. |
| **BR-SH13** | Khi khách tiếp tục mua sắm sau khi giỏ cũ đã `CONVERTED`, hệ thống khởi tạo một giỏ `ACTIVE` mới. |
| **BR-SH14** | Thao tác thêm/sửa sản phẩm trong giỏ hàng tuyệt đối không khóa giữ tồn kho (`ReservedQty` không đổi). |
| **BR-SH15** | Cấm mọi thao tác thêm/sửa/xóa `CartItem` trên giỏ hàng đã ở trạng thái `CONVERTED`. |

---

## 6.6. Sales & Order Management (`BR-SA01` – `BR-SA20`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SA01** | Cấm Guest Checkout: Mỗi `SalesOrder` bắt buộc thuộc về một khách hàng đã đăng nhập (`CustomerID NOT NULL`, `BD-02`). |
| **BR-SA02** | Mỗi đơn hàng có mã nghiệp vụ `OrderNumber` duy nhất toàn hệ thống. |
| **BR-SA03** | Một giỏ hàng chỉ được chuyển đổi thành tối đa một đơn hàng (`UX_SalesOrder_Cart` trên `CartID WHERE CartID IS NOT NULL`). |
| **BR-SA04** | Mỗi đơn hàng bắt buộc có ít nhất một dòng `OrderLine` với `OrderedQty > 0` (cấm đơn rỗng). |
| **BR-SA05** | Mỗi `OrderLine` tham chiếu đúng một `ProductVariant` và lưu snapshot bất biến: `SKU`, `ProductName`, `VariantName`, `UnitPrice`, `VatRate`, `TaxAmount` và mảng JSON `VariantAttributesSnapshot` (`CORR-09`). |
| **BR-SA06** | Cột `VariantAttributesSnapshot` bắt buộc thỏa `ISJSON(...) = 1` và tuân thủ cấu trúc mảng JSON định danh chuẩn `[{"attributeId", "attributeCode", "attributeName", "valueId", "valueCode", "displayValue"}]` (`CORR-09`). |
| **BR-SA07** | Mỗi đơn hàng bắt buộc có đúng một bản chụp địa chỉ nhận hàng `OrderAddress` ($1:1$), hoàn toàn độc lập với sổ địa chỉ `CustomerAddress` (`CORR-04`), đồng thời lưu snapshot tùy chọn thông tin pháp nhân xuất hóa đơn GTGT (`BuyerCompanyName`, `BuyerTaxCode`, `BuyerCompanyAddress`, `BuyerInvoiceEmail`). |
| **BR-SA08** | Khởi tạo `SalesOrder`, `OrderAddress`, `OrderLine`, phân bổ kho, giữ hàng và `Payment` phải thực thi nguyên tử trong Boundary Stored Procedure; thu hồi quyền DML trực tiếp trên `SalesOrder`, `OrderAddress`, `Payment` (`CORR-04`). |
| **BR-SA09** | API đặt đơn bắt buộc kiểm tra khóa chống gửi lặp `IdempotencyRecord(AccountID, 'CREATE_ORDER', IdempotencyKey)` kèm băm `RequestHash` SHA-256 (`CORR-05`). |
| **BR-SA10** | Bốn máy trạng thái vận hành độc lập tuyệt đối: $\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$ (`CORR-06`). |
| **BR-SA11** | Vòng đời `OrderStatus`: `PENDING_PAYMENT` (đơn trả trước chờ thanh toán) $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` $\to$ `SHIPPED` $\to$ `PARTIALLY_DELIVERED` $\to$ `DELIVERED` (nhánh hủy: `PARTIALLY_CANCELLED`, `CANCELLED`). Đơn COD hợp lệ chuyển thẳng sang `CONFIRMED` ngay khi đặt. |
| **BR-SA12** | Mốc đóng hồ sơ hành chính/kế toán sau thời hạn đổi trả được lưu tại cột `SalesOrder.ClosedAt`, không trộn vào máy trạng thái giao vận (`CORR-10`). |
| **BR-SA13** | Sau khi đơn đã tạo, cấm sửa trực tiếp danh sách SKU hoặc đơn giá trên `OrderLine`. |
| **BR-SA14** | Giá bán lẻ niêm yết (`UnitPrice`) đã bao gồm thuế GTGT (VAT-Inclusive): trên từng `OrderLine`, $\text{LineGrossAmount} = \text{OrderedQty} \times \text{UnitPrice}$, $\text{LineNetAmount} = \text{LineGrossAmount} - \text{LineProductDiscount} - \text{AllocatedOrderDiscount}$, và thuế GTGT bóc tách nội bộ $\text{TaxAmount} = \text{LineNetAmount} - \text{ROUND}(\text{LineNetAmount} / (1 + \text{VatRate}/100), 0)$. |
| **BR-SA15** | Chiết khấu voucher cấp đơn (`'ORDER'`) được phân bổ xuống từng dòng đủ điều kiện theo thuật toán **Largest Remainder Method (LRM)** phá hòa bằng `OrderLineID ASC`, bảo đảm $\sum \text{AllocatedOrderDiscount} \equiv \text{ActualVoucherDiscount}$ (`CORR-03`). |
| **BR-SA16** | Tổng thanh toán trên `SalesOrder` thỏa bất biến tiền nguyên VNĐ: $\text{TotalAmount} = \text{SubtotalAmount} - \text{DiscountAmount} + (\text{ShippingFee} - \text{ShippingDiscountAmount})$ (thuế `TaxAmount` đã nằm trong giá bán nên chỉ tổng hợp phục vụ kê khai hóa đơn, tuyệt đối không cộng trùng vào tiền khách trả). |
| **BR-SA17** | Khách hàng được tự hủy đơn khi chưa đóng gói (`PENDING_PAYMENT`, `CONFIRMED`); khi kho đã sang `PROCESSING`, việc hủy toàn bộ hoặc hủy một phần thiếu hàng trước xuất kho (`OrderLine.CancelledQty`, trạng thái `PARTIALLY_CANCELLED`) phải qua nhân viên vận hành và hoàn tiền theo công thức $\Delta F$. |
| **BR-SA18** | Mọi thay đổi trạng thái đơn hàng phải lưu vết bất biến vào `OrderStatusHistory` kèm thời điểm, tác nhân và lý do. |
| **BR-SA19** | Không chuyển giỏ hàng sang `CONVERTED` nếu giao dịch đặt đơn hoặc giữ hàng bị Rollback. |
| **BR-SA20** | Tuyệt đối cấm xóa vật lý (`No Hard Delete`) `SalesOrder`, `OrderLine`, `OrderAddress` và `OrderStatusHistory`. |

---

## 6.7. Fulfillment & Shipping (`BR-FS01` – `BR-FS22`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-FS01** | Một kiện giao vận `Shipment` thuộc đúng một `SalesOrder`. |
| **BR-FS02** | Mỗi `Shipment` chỉ xuất phát từ **duy nhất một** kho vật lý (`WarehouseID NOT NULL`, `BD-03`). |
| **BR-FS03** | Một đơn hàng có thể tách thành nhiều `Shipment` xuất từ các kho khác nhau (Split Shipment). |
| **BR-FS04** | Chỉ khởi tạo `Shipment` khi đơn hàng đã ở trạng thái hợp lệ (`CONFIRMED` / `PROCESSING`). |
| **BR-FS05** | Mỗi `Shipment` bắt buộc có ít nhất một dòng `ShipmentLine` với `ShipmentQty > 0`. |
| **BR-FS06** | Mỗi `ShipmentLine` tham chiếu đúng một `OrderLine` và một `OrderWarehouseAllocation` cùng kho xuất. |
| **BR-FS07** | Lũy kế `ShipmentQty` của một `AllocationID` không vượt quá `AllocatedQty`. |
| **BR-FS08** | Lũy kế `ShipmentQty` của một `OrderLineID` không vượt quá phần số lượng đặt mua còn hiệu lực (`OrderedQty - CancelledQty`). |
| **BR-FS09** | `Shipment.WarehouseID` bắt buộc trùng khớp với `OrderWarehouseAllocation.WarehouseID`. |
| **BR-FS10** | Mỗi `Shipment` gắn với một đối tác vận chuyển `Carrier` hợp lệ; cặp `(CarrierID, TrackingNumber)` là duy nhất khi có mã vận đơn. |
| **BR-FS11** | Hệ thống đề xuất `Carrier` theo khu vực và khối lượng kiện hàng, điều phối viên có thể xác nhận/điều chỉnh trước khi xuất bãi. |
| **BR-FS12** | Vòng đời `ShipmentStatus`: `PREPARING` $\to$ `PACKED` $\to$ `DISPATCHED` $\to$ `IN_TRANSIT` $\to$ `OUT_FOR_DELIVERY` $\to$ `DELIVERED` (nhánh thất bại: `RETURNED_TO_SENDER` $\to$ `RETURN_RECEIVED_AT_WAREHOUSE`) (`CORR-06`). |
| **BR-FS13** | Mỗi lần giao hàng thực tế được ghi nhận vào `DeliveryAttempt` với `AttemptNumber` tăng tuần tự (tối đa 3 lần giao). |
| **BR-FS14** | Khi thất bại cả 3 lần giao, `Shipment` chuyển sang `RETURNED_TO_SENDER` để bưu tá chuyển hoàn về kho. |
| **BR-FS15** | Mọi lần giao thất bại phải ghi rõ `FailureReason`; khi giao thành công bắt buộc lưu `RecipientName` và `DeliveredAt`. |
| **BR-FS16** | Trạng thái kiện hàng được lưu vết bất biến trong `ShipmentStatusHistory` (từ cả Webhook hãng vận chuyển và nhân viên kho). |
| **BR-FS17** | Tách bạch tài chính vận chuyển: `SalesOrder.ShippingFee` là phí vận chuyển thu của khách hàng; `Shipment.ShippingFee` là chi phí cước thực tế trả cho hãng vận chuyển trên từng vận đơn (doanh nghiệp chịu phần chênh lệch trợ giá khi Split Shipment tách nhiều kho). |
| **BR-FS18** | Với đơn COD giao nhiều kiện, mỗi `Shipment` xác định số tiền thu hộ nguyên đồng VNĐ (`CODAmountToCollect` tính theo lũy tiến $\Delta F$ của dòng hàng trong kiện cộng phí ship ròng ở kiện đầu) sao cho tổng trên các kiện khớp đúng `Payment.AmountDue`. |
| **BR-FS19** | Khi bàn giao kiện hàng cho bưu tá (`DISPATCHED`), hệ thống snapshot giá vốn bình quân xuất kho hiện hành vào `ShipmentLine.OutboundUnitCost` (`CORR-08`). |
| **BR-FS20** | Sự kiện `DISPATCHED` đồng thời kích hoạt ghi sổ cái `InventoryTransaction` loại `SHIPMENT_OUT` (`OnHandDelta = -Qty`, `ReservedDelta = -Qty`) (`BD-07`). |
| **BR-FS21** | Khi kiện hàng giao thất bại quay về bến nhập, thủ kho kiểm đếm ngoại quan và lập biên bản `DeliveryReturnReceipt` (E75, $1:0..1$ với `Shipment`). |
| **BR-FS22** | Chỉ khi `DeliveryReturnReceipt` được chốt (`POSTED`), hệ thống mới sinh giao dịch `DELIVERY_RETURN_IN` cộng lại tồn kho và tính lại WAC theo `OutboundUnitCost` ban đầu (`BD-07`, `CORR-08`). |

---

## 6.8. Promotion & Voucher (`BR-PRM01` – `BR-PRM24`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PRM01** | Mỗi `Promotion` định danh bằng `(PromotionCode, VersionNumber)`, phạm vi (`PRODUCT` / `ORDER` / `SHIPPING`), thời gian hiệu lực và trạng thái. |
| **BR-PRM02** | Phân tách 3 tầng khuyến mãi độc lập: Tầng 1 `PRODUCT` (giảm trực tiếp trên dòng SKU), Tầng 2 `ORDER` (voucher giảm trên tiền hàng đủ điều kiện) và Tầng 3 `SHIPPING` (voucher Freeship miễn/giảm phí vận chuyển lưu tại `SalesOrder.ShippingDiscountAmount <= ShippingFee`) (`BD-05`). |
| **BR-PRM03** | Khuyến mãi `PRODUCT` áp dụng đích danh theo `ProductVariant` qua bảng `PromotionProduct` (kèm kiểm soát hạn mức chiến dịch `FlashSaleStockLimit`, `FlashSaleSoldCount` và trần mỗi khách `MaxQtyPerCustomer`). |
| **BR-PRM04** | Chỉ các SKU nằm trong `PromotionProduct` của chương trình đang hiệu lực mới được hưởng ưu đãi Tầng 1. |
| **BR-PRM05** | Chương trình chỉ áp dụng khi đơn hàng/khách hàng thỏa mãn toàn bộ điều kiện trong `PromotionCondition`. |
| **BR-PRM06** | `PromotionCondition` chỉ dùng các toán tử và `ConditionType` định sẵn, cấm biểu thức code tự do để chống lỗi bảo mật và tối ưu truy vấn. |
| **BR-PRM07** | Một chương trình có thể kết hợp nhiều điều kiện `PromotionCondition` (ví dụ `MIN_ORDER_AMOUNT`, `MIN_ITEM_QTY`). |
| **BR-PRM08** | Một chương trình phạm vi `ORDER` hoặc `SHIPPING` có thể phát hành một hoặc nhiều mã `Voucher` (`VoucherCode` duy nhất toàn sàn). |
| **BR-PRM09** | Voucher công khai có `AssignedCustomerID IS NULL`. |
| **BR-PRM10** | Voucher cá nhân hóa (`AssignedCustomerID NOT NULL`) chỉ được sử dụng bởi đúng `Customer` được gán. |
| **BR-PRM11** | Chương trình giới hạn nhóm khách hàng VIP kiểm tra quyền thụ hưởng qua bảng `PromotionTargetCustomer`. |
| **BR-PRM12** | Tổng số lượt giữ chỗ đang chờ và đã tiêu thụ (`PendingCount + UsedCount`) không được vượt quá `MaxUsage` toàn chương trình/voucher. |
| **BR-PRM13** | Tổng lượt sử dụng của mỗi khách hàng không được vượt quá `MaxUsagePerCustomer`. |
| **BR-PRM14** | Khi đặt đơn trả trước, `PromotionUsage` khởi tạo ở trạng thái `PENDING` (giữ quota); chỉ chuyển sang `CONSUMED` và cộng `UsedCount` khi thanh toán thành công (hoặc đơn COD được xác nhận). |
| **BR-PRM15** | Mỗi `OrderLine` chỉ được áp dụng tối đa **1** chương trình khuyến mãi cấp `PRODUCT` (`BD-05`). |
| **BR-PRM16** | Mỗi `SalesOrder` chỉ được áp dụng tối đa **1** mã `Voucher` cấp `ORDER` và tối đa **1** mã `Voucher` cấp `SHIPPING` (`BD-05`). |
| **BR-PRM17** | Các tầng ưu đãi khác nhau (`PRODUCT`, `ORDER`, `SHIPPING`) chỉ được cộng dồn trên cùng đơn hàng khi cờ `IsStackable = 1` cho phép (`BD-05`). |
| **BR-PRM18** | Số tiền giảm thực tế được đóng băng snapshot vào `PromotionUsage.DiscountAmount` và phân bổ LRM xuống `OrderLine.AllocatedOrderDiscount` (`CORR-03`). |
| **BR-PRM19** | Khi đơn hàng bị hủy toàn bộ trước giao hàng, `PromotionUsage` chuyển sang `REVERSED` và hoàn lại lượt dùng nếu voucher còn hạn hiệu lực. |
| **BR-PRM20** | Kiểm tra và trừ hạn mức voucher bắt buộc thực hiện nguyên tử có khóa dòng để chống vượt trần `MaxUsage` khi nhiều người bấm cùng lúc. |
| **BR-PRM21** | Cấm chỉnh sửa công thức/mức giảm của `Promotion` đã từng phát sinh bản ghi `PromotionUsage`. |
| **BR-PRM22** | Khi cần thay đổi chính sách giảm giá của chương trình đang chạy, phải phát hành phiên bản mới (`VersionNumber + 1`, trỏ `SupersedesPromotionID`). |
| **BR-PRM23** | `PromotionUsage` luôn trỏ chính xác về `PromotionID` phiên bản đã áp dụng tại giây phút đặt hàng. |
| **BR-PRM24** | Tuyệt đối cấm xóa vật lý (`No Hard Delete`) các bản ghi `PromotionUsage`. |

---

## 6.9. Payment & Settlement (`BR-PY01` – `BR-PY25`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PY01** | Mỗi `SalesOrder` có quan hệ bắt buộc $1:1$ với đúng một hồ sơ nghĩa vụ thanh toán `Payment` (`Payment.OrderID` UNIQUE, `CORR-04`). |
| **BR-PY02** | Mỗi `Payment` gắn với một phương thức thanh toán `PaymentMethod` đang hoạt động (`MethodCode` duy nhất). |
| **BR-PY03** | Hỗ trợ đồng thời thanh toán trả trước qua cổng điện tử và thanh toán thu hộ tiền mặt khi nhận hàng (COD). |
| **BR-PY04** | Tách biệt trạng thái tổng thể `Payment.PaymentStatus` (`PENDING`, `AUTHORIZED`, `PARTIALLY_PAID`, `PAID`, `PARTIALLY_REFUNDED`, `REFUNDED`, `EXPIRED`, `CANCELLED`) khỏi trạng thái từng lần thử `PaymentTransaction.TransactionStatus` (`CORR-06`). |
| **BR-PY05** | Khi khách thử thanh toán trực tuyến thất bại, chỉ bản ghi `PaymentTransaction` mang trạng thái `FAILED`, còn `Payment` giữ `PENDING` để khách thử lại tạo `PaymentTransaction` mới trong thời gian giữ hàng. |
| **BR-PY06** | Mọi yêu cầu khởi tạo thanh toán (`POST /api/payments`) bắt buộc đi qua kiểm soát chống gửi lặp `IdempotencyRecord` (E74) với `UNIQUE (AccountID, OperationType, IdempotencyKey)` và `RequestHash` SHA-256 (`CORR-05`). |
| **BR-PY07** | Mọi sự kiện callback từ cổng thanh toán phải được lưu bền vững vào `PaymentWebhookLog` (E71) trước khi phản hồi HTTP 200, chống xử lý lặp qua `UNIQUE (PaymentMethodID, ProviderEventID, Environment)` (`CORR-05`). |
| **BR-PY08** | Hệ thống chỉ cập nhật trạng thái thu tiền sau khi xác thực chữ ký số (`SignatureVerified = 1`) hợp lệ trên `PaymentWebhookLog`. |
| **BR-PY09** | Mỗi `PaymentTransaction` lưu đầy đủ `TransactionReference` nội bộ, `GatewayTransactionID`, `TransactionFee` và dữ liệu phản hồi cổng để đối soát dòng tiền ròng. |
| **BR-PY10** | Đơn trả trước chỉ chuyển `OrderStatus = 'CONFIRMED'` khi `PaymentStatus = 'PAID'` (`AmountPaid = AmountDue`). |
| **BR-PY11** | Tổng số tiền thu thành công trên các giao dịch không được vượt quá `AmountDue` của hồ sơ `Payment`. |
| **BR-PY12** | Đối soát COD tách bạch hai đầu (`CORR-07`): Nghĩa vụ khách hàng (`PaymentStatus`) độc lập với công nợ phải thu từ hãng vận chuyển (`CarrierSettlementStatus`). |
| **BR-PY13** | Với đơn COD (kể cả Split Shipment đa kiện), mỗi khi bưu tá thu tiền một kiện `Shipment`, hệ thống ghi nhận `CODCollection` (E77) và cập nhật tiến độ thanh toán khách hàng sang `PARTIALLY_PAID` hoặc `PAID`; nếu kiện còn lại giao thất bại và bị hủy phần chưa giao (`PARTIALLY_CANCELLED`), hệ thống giảm `Payment.AmountDue` tương ứng để khép kín trạng thái `PAID` cho phần đã thu. |
| **BR-PY14** | Khi hãng vận chuyển chuyển trả tiền thu hộ theo kỳ, kế toán ghi nhận bảng kê `CODRemittance` (E78) kèm tổng tiền thu, cước khấu trừ và thực nhận. |
| **BR-PY15** | Từng khoản thu kiện hàng `CODCollection` được khớp chi tiết với bảng kê qua `CODReconciliation` (E79), theo dõi mọi khoản lệch `VarianceAmount`. |
| **BR-PY16** | Chỉ khi đối soát khớp (`RECONCILED`), hệ thống mới sinh `PaymentTransaction` loại `COD_SETTLEMENT` và chuyển `Payment.CarrierSettlementStatus = 'RECONCILED'`. |
| **BR-PY17** | Mỗi yêu cầu hoàn tiền `Refund` phải gắn với một hồ sơ `Payment` đã có thực thu (`AmountPaid > 0`). |
| **BR-PY18** | Yêu cầu `Refund` có thể phát sinh từ hủy đơn trả trước trước khi giao (`ReturnLineAssessmentID IS NULL`) hoặc từ biên bản thẩm định RMA được duyệt (`ReturnLineAssessmentID NOT NULL`). |
| **BR-PY19** | Mọi chứng từ `Refund` bắt buộc được cấp có thẩm quyền phê duyệt (`ApprovedBy NOT NULL`) trước khi chi tiền. |
| **BR-PY20** | Hỗ trợ hoàn tiền toàn phần, hoàn một phần và hoàn nhiều đợt trên cùng một đơn hàng. |
| **BR-PY21** | Mỗi giao dịch chi trả hoàn tiền `RefundTransaction` (E59) bắt buộc trỏ `OriginalPaymentTransactionID` về đúng giao dịch thu tiền `SUCCEEDED` gốc trên `PaymentTransaction` (`CORR-08`). |
| **BR-PY22** | Trần hoàn tiền bất biến: $\sum \text{RefundAmount} \le \text{AmountPaid} - \text{AmountRefunded}$. |
| **BR-PY23** | Cột `Payment.AmountRefunded` chỉ được cộng tăng khi `RefundTransaction` thực thi thành công (`SUCCEEDED`). |
| **BR-PY24** | Đơn COD đã thu tiền mặt được hoàn trả qua chuyển khoản ngân hàng (`BANK_TRANSFER`) và lưu mã ủy nhiệm chi trên `RefundTransaction`. |
| **BR-PY25** | Cấm tuyệt đối xóa vật lý (`No Hard Delete`) bất kỳ bản ghi nào thuộc `Payment`, `PaymentTransaction`, `Refund`, `RefundTransaction`, `PaymentWebhookLog`, `IdempotencyRecord` và cụm đối soát COD. |

---

## 6.10. Return & Exchange (`BR-RT01` – `BR-RT24`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-RT01** | Yêu cầu đổi trả `ReturnRequest` phải gắn với một `SalesOrder` hợp lệ thuộc đúng `Customer` sở hữu đơn. |
| **BR-RT02** | Khách hàng tự tạo hoặc nhân viên CSKH tạo hộ yêu cầu đổi trả; luôn lưu vết `CreatedByAccountID` bất biến. |
| **BR-RT03** | Mỗi dòng yêu cầu đổi trả `ReturnRequestLine` tham chiếu `OrderLine` gốc và phải phân bổ cụ thể vào kiện hàng đã giao qua bảng cầu nối `ReturnShipmentAllocation` (E76) (`CORR-10`). |
| **BR-RT04** | Thời hạn đổi trả RMA được kiểm tra độc lập trên từng kiện giao thành công: $\text{RequestedAt} \le \text{Shipment.DeliveredAt} + \text{PolicyReturnDays}$ (`CORR-10`). |
| **BR-RT05** | Mỗi `ReturnRequest` bắt buộc có ít nhất một dòng `ReturnRequestLine` với `RequestedQty > 0`. |
| **BR-RT06** | Lũy kế số lượng phân bổ trả trên từng dòng kiện giao không được vượt số lượng đã giao trừ số lượng đã trả trước đó: $\sum \text{AllocatedReturnQty} \le \text{ShipmentQty} - \text{PreviouslyReturnedQty}$. |
| **BR-RT07** | Hỗ trợ trả hàng một phần số lượng trên dòng đơn và trả làm nhiều đợt RMA khác nhau nếu còn hạn và còn số lượng đủ điều kiện. |
| **BR-RT08** | Lũy kế số lượng duyệt trả trên toàn bộ các đợt RMA của một `OrderLine` thỏa $\sum \text{ApprovedQty} \le \text{OrderedQty} - \text{AlreadyReturnedQty}$. |
| **BR-RT09** | CSKH bắt buộc xét duyệt yêu cầu (`APPROVED`) trước khi khách gửi kiện hàng hoàn về kho. |
| **BR-RT10** | Mỗi `ReturnRequest` chỉ định **đúng một** kho tiếp nhận cố định (`WarehouseID NOT NULL`). |
| **BR-RT11** | Tất cả các phiếu tiếp nhận tại bến nhập `ReturnReceipt` của yêu cầu RMA phải xử lý tại đúng kho đã chỉ định. |
| **BR-RT12** | Một yêu cầu RMA có thể tiếp nhận thành nhiều đợt phiếu `ReturnReceipt`; mỗi phiếu có ít nhất một dòng `ReturnReceiptLine`. |
| **BR-RT13** | Tổng `ReceivedQty` thực nhận qua các phiếu `ReturnReceipt` không vượt quá `ApprovedQty`; khi chốt phiếu bến nhập (`POSTED`), hệ thống ghi `RETURN_RECEIPT_IN` đưa hàng hoàn vào bể cách ly (`+OnHandQty`, `+QuarantinedQty`, tồn `SELLABLE` chưa đổi). |
| **BR-RT14** | Quyền hoàn tiền của khách hàng (`RefundEligibilityStatus`) được tách rời độc lập hoàn toàn khỏi định đoạt vị trí tồn kho (`InventoryDisposition`) thông qua thực thể giám định `ReturnLineAssessment` (E72) (`CORR-01`). |
| **BR-RT15** | Một dòng nhận hàng `ReturnReceiptLine` có thể tách thành nhiều lô kết luận giám định `ReturnLineAssessment` ($1:N$ theo `AssessmentGroupNo`) sao cho tổng `AssessedQty` hiện hành (`IsCurrent = 1`) $\le \text{ReceivedQty}$ (`CORR-01`). |
| **BR-RT16** | Biên bản giám định `ReturnLineAssessment` được phiên bản hóa bất biến (`AssessmentVersion`, `IsCurrent`), cấm ghi đè lịch sử khi tái thẩm định tranh chấp; mỗi `(ReturnReceiptLineID, AssessmentGroupNo)` chỉ có tối đa 1 bản ghi `IsCurrent = 1`. |
| **BR-RT17** | Mỗi dòng giám định phân định rõ 4 chiều: `FaultParty` (`VENDOR_FAULT`, `CARRIER_FAULT`, `CUSTOMER_FAULT`, `NO_FAULT`), `RefundEligibilityStatus` (`PENDING_INSPECTION`, `APPROVED`, `PARTIALLY_APPROVED`, `REJECTED`), `ResolutionType` (`REFUND`, `EXCHANGE`, `REPAIR`, `NO_ACTION`) và `InventoryDisposition` (`SELLABLE`, `QUARANTINED`, `DISPOSED`). |
| **BR-RT18** | Hàng lỗi do nhà sản xuất (`VENDOR_FAULT`) hoặc hư hỏng do vận chuyển (`CARRIER_FAULT`) dù phải tiêu hủy vật lý (`InventoryDisposition = 'DISPOSED' \to \text{SCRAP\_OUT}` giảm đồng thời `OnHandQty` và `QuarantinedQty`) vẫn được duyệt `RefundEligibilityStatus = 'APPROVED'` hoàn tiền đầy đủ (`CORR-01`). |
| **BR-RT19** | Hàng cần giữ lại cách ly chờ hãng giám định (`InventoryDisposition = 'QUARANTINED'`) bắt buộc giữ `RefundEligibilityStatus = 'PENDING_INSPECTION'`, chưa sinh chứng từ `Refund` cho đến khi có phiên bản kết luận cuối cùng. |
| **BR-RT20** | Chỉ phần số lượng được KCS kết luận đạt chuẩn bán lại (`InventoryDisposition = 'SELLABLE'`) mới giải phóng khỏi `QuarantinedQty` sang bể `SELLABLE` và tính lại giá vốn WAC theo đúng `OutboundUnitCost` xuất kho ban đầu của kiện hàng (`CORR-08`). |
| **BR-RT21** | Hỗ trợ hoàn tiền từng phần theo mốc (Partial Milestone Refund): ngay khi một lô `ReturnLineAssessment` được duyệt `APPROVED` với `ResolutionType = 'REFUND'`, hệ thống kích hoạt tạo `Refund` mà không bắt chờ đóng toàn bộ RMA; nếu `ResolutionType = 'EXCHANGE'`, hệ thống khởi tạo đơn đổi bù liên kết `SalesOrder(OrderType = 'EXCHANGE_REPLACEMENT', ParentOrderID, TotalAmount = 0)` (`CORR-01`). |
| **BR-RT22** | Số tiền hoàn hàng hóa cho $b$ đơn vị mới trên `OrderLine` (sau khi khóa `UPDLOCK` với $a = \text{CancelledQty} + \text{RefundedQty}$ đơn vị đã hoàn trước đó) được tính bằng $\Delta\text{Refund} = F(a+b) - F(a)$ với $F(q) = B \cdot q + \min(q, R)$; riêng khi trả hàng do `CUSTOMER_FAULT` / `NO_FAULT` làm giá trị hàng giữ lại tụt dưới `MinOrderAmount` của voucher cấp đơn, hệ thống khấu trừ thu hồi phần ưu đãi voucher không còn đủ điều kiện (`CORR-03`). |
| **BR-RT23** | Phí vận chuyển hoàn lại (`ShippingRefundAmount`) được hạch toán tách biệt với tiền hàng trên `ReturnLineAssessment`. |
| **BR-RT24** | Mọi chuyển đổi trạng thái RMA phải lưu vết trong `ReturnStatusHistory`; cấm tuyệt đối xóa vật lý hồ sơ RMA, phiếu nhận và biên bản giám định. |

---

## 6.11. Audit & Reporting (`BR-AR01` – `BR-AR20`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-AR01** | Hệ thống ghi `AuditLog` chọn lọc cho các thao tác quản trị và nghiệp vụ nhạy cảm (duyệt/hủy đơn, chỉnh tồn, duyệt kiểm kê, đổi giá SKU, duyệt hoàn tiền, thay đổi phân quyền RBAC). |
| **BR-AR02** | Mỗi `AuditLog` bắt buộc lưu `ActionType`, `EntityName`, `RecordID`, `ActionAt`, `Result` và `CorrelationID`. |
| **BR-AR03** | Thao tác do người dùng thực hiện phải ghi nhận `AccountID`; tiến trình tự động chạy nền cho phép `AccountID IS NULL` nhưng phải nêu rõ tác nhân hệ thống. |
| **BR-AR04** | Chi tiết thay đổi từng trường dữ liệu nhạy cảm được lưu cặp `OldValue` / `NewValue` trong `AuditLogDetail`. |
| **BR-AR05** | Tuyệt đối cấm ghi `PasswordHash`, mã OTP, JWT token, khóa bí mật hoặc số thẻ thanh toán đầy đủ vào `AuditLog` / `AuditLogDetail`. |
| **BR-AR06** | Bảng `AuditLog` và `AuditLogDetail` là nhật ký chỉ ghi (`Append-Only / WORM`); thu hồi hoàn toàn quyền `UPDATE` và `DELETE` của ứng dụng. |
| **BR-AR07** | `AuditLog` quản lý hạn lưu trữ nóng qua `RetentionUntil` (12–24 tháng) và phân vùng theo `ActionAt` để chuyển dữ liệu cũ sang lưu trữ lạnh không gây khóa bảng. |
| **BR-AR08** | Danh mục báo cáo quản trị được định nghĩa chuẩn hóa trong `ReportDefinition` (`ReportCode` duy nhất). |
| **BR-AR09** | Mỗi lần người dùng xem hoặc kết xuất báo cáo ra file bắt buộc sinh một bản ghi nhật ký `ReportExecution`. |
| **BR-AR10** | `ReportExecution` lưu đầy đủ `AccountID` người chạy, chuỗi tham số lọc `Parameters`, trạng thái và đường dẫn file kết xuất `ResultLocation`. |
| **BR-AR11** | Chỉ tài khoản mang vai trò (`Role`) được cấp quyền tương ứng trong `ReportPermission` mới được phép truy cập báo cáo. |
| **BR-AR12** | Báo cáo vận hành thời gian thực truy vấn dưới chế độ đọc không khóa (`RCSI` / Read Replica) để không gây nghẽn giao dịch đặt hàng OLTP. |
| **BR-AR13** | Báo cáo tài chính, doanh thu và tồn kho chốt kỳ kế toán bắt buộc sử dụng `ReportSnapshot` để đóng băng số liệu cố định, không bị biến động hồi tố. |
| **BR-AR14** | Mỗi `ReportSnapshot` định danh duy nhất bằng `SnapshotCode`, lưu rõ `PeriodStart`, `PeriodEnd` và `SnapshotAt`. |
| **BR-AR15** | Bản chụp `ReportSnapshot` sau khi chốt (`FINALIZED`) là bất biến, cấm sửa đổi dữ liệu trực tiếp. |
| **BR-AR16** | Nếu phát hiện điều chỉnh kế toán sau khóa sổ, phải phát hành bản chụp phiên bản mới thay vì ghi đè bản cũ. |
| **BR-AR17** | Ma trận phân quyền `ReportPermission(ReportDefinitionID, RoleID)` tách bạch độc lập 3 cờ quyền hạn: `CanView`, `CanExport` và `CanCreateSnapshot`. |
| **BR-AR18** | Nhân viên không có quyền `CanExport = 1` không được tải dữ liệu thô CSV/Excel ra khỏi hệ thống. |
| **BR-AR19** | Chỉ quản trị viên tài chính/kho được cấp `CanCreateSnapshot = 1` mới được phép chốt sổ kỳ báo cáo. |
| **BR-AR20** | Không sinh các bản chụp tự động trùng lặp cùng kỳ và cùng bộ tham số nếu kỳ trước đã chốt thành công. |
