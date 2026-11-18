# 06. BUSINESS RULES – BỘ QUY TẮC NGHIỆP VỤ (PHIÊN BẢN v1.1)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.1 – Approved Business Decisions (BD-01 đến BD-07)
* **Database target:** Microsoft SQL Server

Bộ quy tắc nghiệp vụ dưới đây được chuẩn hóa theo 10 nhóm có mã định danh duy nhất để truy vết trực tiếp từ nghiệp vụ sang ERD (M2), Relational Schema (M3) và Database Constraints / Stored Procedures (M4).

---

## 6.1. Identity & Access (`BR-ID`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-ID01** | Email đăng nhập phải duy nhất trong hệ thống. |
| **BR-ID02** | Một người dùng có thể được gán nhiều vai trò. |
| **BR-ID03** | Một vai trò có thể được gán cho nhiều người dùng. |
| **BR-ID04** | Người dùng chỉ được thực hiện chức năng phù hợp với quyền được cấp. |
| **BR-ID05** | Tài khoản bị vô hiệu hóa không được đăng nhập. |

---

## 6.2. Product Catalog (`BR-PR`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PR01** | Mỗi sản phẩm phải có mã định danh duy nhất. |
| **BR-PR02** | Mỗi SKU phải duy nhất trong toàn hệ thống. |
| **BR-PR03** | Một sản phẩm có thể có nhiều SKU. |
| **BR-PR04** | Mỗi SKU thuộc duy nhất một sản phẩm. |
| **BR-PR05** | Một sản phẩm có thể thuộc nhiều danh mục và một danh mục có thể chứa nhiều sản phẩm ($N-N$). Quan hệ được quản lý thông qua thực thể liên kết giữa Product và Category. |
| **BR-PR06** | Giá bán của SKU không được âm. |
| **BR-PR07** | Sản phẩm đã phát sinh giao dịch không được xóa vật lý tùy tiện. |
| **BR-PR08** | SKU ngừng kinh doanh không được sử dụng trong đơn hàng mới. |
| **BR-PR09** | Thông tin sản phẩm tại thời điểm giao dịch phải được bảo toàn khi cần thiết. |
| **BR-PR10** | Mỗi sản phẩm ở trạng thái Published bắt buộc phải có đúng một danh mục chính (`IsPrimary = 1`). Sản phẩm ở trạng thái Draft có thể chưa có danh mục chính. |
| **BR-PR11** | Thuộc tính biến thể được định nghĩa linh hoạt theo từng sản phẩm. Không mặc định mọi sản phẩm đều có cùng tập thuộc tính như Size, Color hoặc Material. |
| **BR-PR12** | Một SKU phải có một tổ hợp giá trị thuộc tính biến thể xác định trong phạm vi sản phẩm. Không được tồn tại hai SKU có cùng tổ hợp thuộc tính trong cùng một sản phẩm. |
| **BR-PR13** | Một thuộc tính khi được gán cho một sản phẩm chỉ có một vai trò duy nhất: `DESCRIPTIVE` (mô tả chung) hoặc `VARIANT_DEFINING` (định nghĩa biến thể). Không cho phép cùng một thuộc tính vừa làm thông tin mô tả, vừa tham gia phân biệt SKU trong cùng sản phẩm. |

---

## 6.3. Inventory & Warehouse (`BR-IN`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-IN01** | Một doanh nghiệp có thể có nhiều kho hàng vật lý. |
| **BR-IN02** | Tồn kho được quản lý theo từng SKU tại từng kho. |
| **BR-IN03** | Một SKU có thể được lưu trữ tại nhiều kho. |
| **BR-IN04** | Tồn thực tế không được âm (`OnHandQty >= 0`). |
| **BR-IN05** | Tồn giữ không được âm và không được lớn hơn tồn thực tế (`ReservedQty >= 0` và `ReservedQty <= OnHandQty`). |
| **BR-IN06** | Tồn khả dụng bằng tồn thực tế trừ tồn giữ ($\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$). |
| **BR-IN07** | Mọi biến động tồn kho phải có lịch sử bất biến. |
| **BR-IN08** | Phiếu nhập đã xác nhận không được chỉnh sửa trực tiếp. |
| **BR-IN09** | Một đơn hàng không được giữ số lượng vượt tồn khả dụng. |
| **BR-IN10** | Hủy đơn phải giải phóng lượng hàng đã giữ tương ứng. |
| **BR-IN11** | Xuất hàng phải được ghi nhận theo kho thực tế xuất hàng. |
| **BR-IN12** | Chuyển kho phải bảo toàn tổng số lượng hàng trong toàn hệ thống, ngoại trừ hao hụt được ghi nhận riêng. |
| **BR-IN13** | Khi đơn hàng được tạo thành công, hệ thống phải thực hiện giữ hàng ngay cho các SKU được phân bổ từ kho tương ứng. Việc giữ hàng phải được thực hiện có kiểm soát đồng thời để tránh vượt quá tồn khả dụng. |
| **BR-IN14** | Mỗi lượt giữ hàng phải ghi nhận số lượng, SKU, kho, thời điểm tạo và thời điểm hết hạn `ExpiresAt`. |
| **BR-IN15** | Tại một kho, $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$. $\text{AvailableQty}$ không được âm và $\text{ReservedQty}$ không được vượt quá $\text{OnHandQty}$. |
| **BR-IN16** | Khi lượt giữ hàng hết hạn mà chưa được chuyển sang giai đoạn xuất kho, hệ thống phải giải phóng số lượng đã giữ tương ứng. |
| **BR-IN17** | Khi kho xuất hàng và bàn giao cho đơn vị vận chuyển, hệ thống phải giảm đồng thời $\text{OnHandQty}$ và $\text{ReservedQty}$ theo số lượng thực tế xuất. Sự kiện phải được ghi nhận bằng StockMovement loại `SHIPMENT_OUT`. |
| **BR-IN18** | Hàng đã bàn giao cho đơn vị vận chuyển không còn thuộc tồn kho vật lý tại kho xuất. Hàng đang vận chuyển phải được theo dõi thông qua trạng thái giao hàng và dữ liệu Shipment, không được tính vào $\text{OnHandQty}$ của kho. |
| **BR-IN19** | Khi hàng giao thất bại và thực sự được vận chuyển quay về kho, nhân viên kho phải xác nhận nhận hàng. Hệ thống ghi StockMovement loại `DELIVERY_RETURN_IN` và tăng $\text{OnHandQty}$ theo số lượng thực tế nhận lại. |
| **BR-IN20** | Nếu đơn hàng vẫn được tiếp tục giao sau khi hàng quay về kho, hệ thống phải thực hiện phân bổ và giữ hàng lại theo quy tắc tồn kho hiện hành trước khi xuất kho lần tiếp theo. |
| **BR-IN21** | Nếu đơn hàng bị hủy, hệ thống phải giải phóng các lượt giữ hàng còn hiệu lực. Với hàng đã xuất kho và quay về, việc nhập lại kho phải được ghi nhận riêng, không được xử lý như thao tác giải phóng reservation. |
| **BR-IN22** | Mỗi cặp `(WarehouseID, VariantID)` có tối đa một bản ghi `InventoryBalance`. |
| **BR-IN23** | `OnHandQty` và `ReservedQty` không được âm; `ReservedQty` không được vượt `OnHandQty`. |
| **BR-IN24** | Số lượng khả dụng được xác định bằng $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$. |
| **BR-IN25** | Kho ngừng hoạt động không được nhận phân bổ (`OrderWarehouseAllocation`) mới. |
| **BR-IN26** | Tổng số lượng phân bổ (`AllocatedQty`) của một dòng đơn hàng không được vượt số lượng đặt mua của dòng đó. |
| **BR-IN27** | Reservation chỉ được tạo thành công khi hệ thống giữ đủ số lượng cần thiết cho toàn bộ đơn hàng (nguyên tắc All-or-nothing). |
| **BR-IN28** | Reservation hết hạn phải giải phóng số lượng đã giữ, không được làm thay đổi số lượng tồn kho vật lý (`OnHandQty`). |
| **BR-IN29** | Việc bàn giao hàng cho đơn vị vận chuyển (`SHIPMENT_OUT`) phải làm giảm đồng thời cả tồn kho vật lý (`OnHandQty`) và tồn kho đã giữ (`ReservedQty`). |
| **BR-IN30** | Chỉ hàng thực tế được kho tiếp nhận và ký biên bản mới được ghi nhận là hàng nhập lại từ giao thất bại (`DELIVERY_RETURN_IN`). |
| **BR-IN31** | Mọi thay đổi tồn kho vật lý bắt buộc phải có chứng từ giao dịch tồn kho hợp lệ (`InventoryTransaction`). |
| **BR-IN32** | Chứng từ tồn kho đã `POSTED` tuyệt đối không được chỉnh sửa trực tiếp; sai lệch phải được xử lý bằng giao dịch điều chỉnh hoặc đảo giao dịch. |
| **BR-IN33** | Điều chỉnh tăng/giảm tồn kho phải có lý do, người thực hiện và cơ chế phê duyệt trước khi ghi sổ. |
| **BR-IN34** | Chuyển kho phải ghi nhận xuất tại kho nguồn và nhập tại kho đích; không được tạo hoặc xóa hàng mà không có đối ứng. |
| **BR-IN35** | Kiểm kê phải lưu lại số lượng hệ thống (`SystemQty`), số lượng thực tế (`CountedQty`) và chênh lệch (`VarianceQty`) tại thời điểm kiểm kê. |

> **Quy ước quan trọng về trạng thái giữ hàng:**
> * `ReservedQty` chỉ đại diện cho hàng đang được giữ tại kho nhưng chưa xuất khỏi kho.
> * Khi hàng được xuất kho, reservation tại kho được tiêu thụ và giải phóng tương ứng.
> * Nếu hàng đang trên đường giao, cam kết cung ứng cho đơn hàng được theo dõi qua Shipment và các bản ghi phân bổ giao hàng, không tiếp tục cộng vào `ReservedQty`.
> * Nếu giao thất bại, hàng chỉ được cộng lại vào tồn kho khi kho thực sự tiếp nhận hàng.
> 
> *Như vậy, yêu cầu “giữ hàng đến khi giao thành công hoặc hủy” được hiểu là **duy trì cam kết xử lý đơn hàng**, không phải duy trì `ReservedQty` sau khi hàng đã rời khỏi kho.*

---

## 6.4. Procurement (`BR-PO`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PO01** | Một nhà cung cấp có thể có nhiều đơn mua hàng. |
| **BR-PO02** | Một đơn mua hàng có ít nhất một dòng hàng. |
| **BR-PO03** | Số lượng đặt mua phải lớn hơn $0$. |
| **BR-PO04** | Số lượng thực nhận không được âm. |
| **BR-PO05** | Không được nhận vượt số lượng còn được phép nhận nếu không có phê duyệt ngoại lệ. |
| **BR-PO06** | Phiếu nhập đã xác nhận phải được bảo toàn lịch sử. |

---

## 6.5. Customer & Cart (`BR-CU`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-CU01** | Một khách hàng có thể có nhiều địa chỉ giao hàng. |
| **BR-CU02** | Một khách hàng có thể có nhiều đơn hàng. |
| **BR-CU03** | Một giỏ hàng có thể chứa nhiều SKU. |
| **BR-CU04** | Mỗi dòng sản phẩm trong giỏ hàng phải có số lượng lớn hơn $0$. |
| **BR-CU05** | Không cho phép thêm SKU ngừng kinh doanh vào giỏ hàng mới. |
| **BR-CU06** | Giá trong giỏ hàng không phải giá giao dịch cuối cùng; giá phải được xác nhận lại khi Checkout. |
| **BR-CU07** | Khách hàng phải đăng nhập bằng tài khoản hợp lệ trước khi tạo đơn hàng. Không hỗ trợ Guest Checkout. |

---

## 6.6. Sales Order (`BR-OR`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-OR01** | Mỗi đơn hàng phải có mã đơn hàng duy nhất. |
| **BR-OR02** | Một đơn hàng phải có ít nhất một dòng hàng. |
| **BR-OR03** | Một dòng hàng tham chiếu một SKU. |
| **BR-OR04** | Số lượng đặt mua phải lớn hơn $0$. |
| **BR-OR05** | Giá bán tại thời điểm đặt hàng phải được bảo toàn (Snapshot). |
| **BR-OR06** | Thông tin giao hàng tại thời điểm đặt phải được bảo toàn. |
| **BR-OR07** | Tổng tiền phải được tính dựa trên các thành phần tiền hợp lệ của đơn hàng. |
| **BR-OR08** | Đơn hàng phải tuân thủ quy trình chuyển trạng thái hợp lệ. |
| **BR-OR09** | Mọi thay đổi trạng thái quan trọng phải có lịch sử. |
| **BR-OR10** | Không được hủy đơn đã hoàn tất theo quy trình hủy thông thường. |
| **BR-OR11** | Mỗi đơn hàng phải thuộc về một khách hàng đã đăng nhập. |
| **BR-OR12** | Một đơn hàng có thể chứa các dòng sản phẩm được phân bổ từ nhiều kho khác nhau (Split Shipment). Không gắn cứng một `WarehouseID` duy nhất vào Order. |

---

## 6.7. Promotion (`BR-PM` / `BR-PRM`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PRM01** | Mỗi Promotion phải có mã chương trình, tên, phạm vi (`PRODUCT`/`ORDER`), thời gian hiệu lực và trạng thái. |
| **BR-PRM02** | Promotion có phạm vi `PRODUCT` (giảm giá trên dòng SKU) hoặc `ORDER` (voucher cấp đơn hàng). |
| **BR-PRM03** | Promotion cấp sản phẩm chỉ áp dụng theo SKU (`ProductVariant`, PROMO-Q01). |
| **BR-PRM04** | Promotion cấp sản phẩm chỉ được áp dụng cho SKU nằm trong danh sách `PromotionProduct` được chỉ định. |
| **BR-PRM05** | Promotion phải thỏa mãn các điều kiện cấu hình sẵn trước khi được áp dụng. |
| **BR-PRM06** | Điều kiện khuyến mãi chỉ sử dụng các loại điều kiện (`ConditionType`) và toán tử được hệ thống hỗ trợ, cấm biểu thức tự do (PROMO-Q02). |
| **BR-PRM07** | Promotion có thể có nhiều điều kiện áp dụng kết hợp (`PromotionCondition`). |
| **BR-PRM08** | Một Promotion cấp đơn hàng có thể phát hành một hoặc nhiều mã Voucher. |
| **BR-PRM09** | Voucher dùng chung không gắn với Customer cụ thể (`AssignedCustomerID IS NULL`). |
| **BR-PRM10** | Voucher riêng chỉ được sử dụng bởi Customer được chỉ định (`AssignedCustomerID NOT NULL`, PROMO-Q03). |
| **BR-PRM11** | Promotion có thể giới hạn đối tượng khách hàng bằng danh sách `PromotionTargetCustomer` (PROMO-Q07). |
| **BR-PRM12** | Tổng lượt sử dụng thành công toàn sàn không vượt `MaxUsage`. |
| **BR-PRM13** | Lượt sử dụng của mỗi Customer không vượt giới hạn mỗi khách hàng `MaxUsagePerCustomer` (PROMO-Q04). |
| **BR-PRM14** | Chỉ ghi nhận lượt sử dụng chính thức (`CONSUMED`, tăng `UsedCount`) khi thanh toán thành công (PROMO-Q05). |
| **BR-PRM15** | Mỗi `OrderLine` chỉ được áp dụng tối đa một Promotion cấp sản phẩm (BD-05). |
| **BR-PRM16** | Mỗi `SalesOrder` chỉ được áp dụng tối đa một Voucher cấp đơn hàng (BD-05). |
| **BR-PRM17** | Ưu đãi cấp sản phẩm và cấp đơn hàng chỉ được kết hợp khi đáp ứng quy tắc cộng dồn (`IsStackable = 1`) và thứ tự ưu tiên `Priority` (BD-05). |
| **BR-PRM18** | Số tiền giảm thực tế phải được snapshot bất biến vào `PromotionUsage.DiscountAmount`. |
| **BR-PRM19** | Khi đơn bị hủy hoặc hoàn trả, lượt sử dụng được hoàn lại (`REVERSED`) nếu voucher còn trong thời hạn hiệu lực (PROMO-Q06). |
| **BR-PRM20** | Không được vượt giới hạn sử dụng khi có nhiều yêu cầu áp dụng ưu đãi đồng thời (kiểm tra nguyên tử trong giao dịch). |
| **BR-PRM21** | Tuyệt đối không được sửa thông tin ảnh hưởng kết quả ưu đãi của Promotion đã có lượt sử dụng. |
| **BR-PRM22** | Khi cần thay đổi logic ưu đãi, phải tạo Promotion mới hoặc phiên bản mới (`VersionNumber += 1`, PROMO-Q08). |
| **BR-PRM23** | `PromotionUsage` phải tham chiếu đúng chương trình và phiên bản đã áp dụng tại thời điểm giao dịch. |
| **BR-PRM24** | Tuyệt đối không được xóa vật lý lịch sử sử dụng ưu đãi đã phát sinh (`No Hard Delete` trên `PromotionUsage`). |

> **Quy tắc tầng khuyến mãi (BD-05):**
> 1. **Tầng Product Promotion:** Tối đa một chương trình giảm giá trực tiếp trên mỗi dòng sản phẩm (`OrderLine`).
> 2. **Tầng Order Voucher:** Tối đa một voucher cấp đơn hàng (`SalesOrder`).
> 3. **Cộng dồn:** Hai tầng có thể kết hợp nếu quy tắc cộng dồn cho phép (`IsStackable = 1`).
> 4. **Kiểm toán:** Hệ thống snapshot toàn bộ kết quả tính toán chiết khấu vào `PromotionUsage` để phục vụ đối soát và kiểm toán tài chính.

---

## 6.8. Payment & Refund (`BR-PA` / `BR-PY`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PY01** | Mỗi đơn hàng `SalesOrder` có đúng một hồ sơ thanh toán `Payment` (`OrderID` UNIQUE, PAY-Q01). |
| **BR-PY02** | Mỗi `Payment` chỉ sử dụng một phương thức thanh toán `PaymentMethod` hợp lệ và đang hoạt động. |
| **BR-PY03** | Hệ thống hỗ trợ song song thanh toán COD và thanh toán trực tuyến qua cổng thanh toán (PAY-Q02). |
| **BR-PY04** | Mỗi lần thử thanh toán phải tạo một giao dịch `PaymentTransaction` mới độc lập (PAY-Q03). |
| **BR-PY05** | Tuyệt đối không ghi đè hoặc xóa các giao dịch thanh toán đã phát sinh; lưu nhật ký bất biến để đối soát. |
| **BR-PY06** | Giao dịch thanh toán phải lưu mã tham chiếu nội bộ `TransactionReference` và thông tin cổng để đối soát (PAY-Q07). |
| **BR-PY07** | Lưu phí giao dịch `TransactionFee` nếu cổng thanh toán cung cấp để xác định số tiền thực nhận (PAY-Q09). |
| **BR-PY08** | Chỉ ghi nhận thanh toán thành công khi có kết quả xác thực chữ ký/trạng thái hợp lệ từ cổng thanh toán. |
| **BR-PY09** | Đơn hàng thanh toán trực tuyến chỉ chuyển sang `CONFIRMED` khi thanh toán thành công (`AmountPaid == AmountDue`, PAY-Q08). |
| **BR-PY10** | Đơn hàng COD được xác nhận theo quy trình COD; việc xác nhận đơn không đồng nghĩa với việc đã thu tiền (tiền chỉ được thu khi giao hàng thành công). |
| **BR-PY11** | Một hồ sơ `Payment` có thể có nhiều lần thử giao dịch `PaymentTransaction` (cho phép thử lại khi thất bại). |
| **BR-PY12** | Tổng số tiền thanh toán thành công không được vượt số tiền phải thu của đơn hàng: `AmountPaid <= AmountDue`. |
| **BR-PY13** | Mỗi yêu cầu hoàn tiền `Refund` phải thuộc một `Payment` đã phát sinh khoản tiền thực thu (`AmountPaid > 0`). |
| **BR-PY14** | Yêu cầu hoàn tiền phải được nhân viên có thẩm quyền phê duyệt (`ApprovedBy NOT NULL`, PAY-Q05). |
| **BR-PY15** | Hỗ trợ hoàn tiền toàn phần, một phần và hoàn nhiều lần theo đợt trả hàng (PAY-Q04). |
| **BR-PY16** | Một hồ sơ `Payment` có thể phát sinh nhiều yêu cầu hoàn tiền `Refund`. |
| **BR-PY17** | Một yêu cầu `Refund` có thể được thực hiện thông qua nhiều giao dịch hoàn tiền `RefundTransaction`. |
| **BR-PY18** | Tổng số tiền hoàn thành công không vượt số tiền đã thu trừ đi số tiền đã hoàn trước đó: $\sum RefundAmount \le RemainingRefundable = AmountPaid - AmountRefunded$. |
| **BR-PY19** | Chỉ các giao dịch hoàn tiền `RefundTransaction` thành công mới được cộng vào tổng số tiền đã hoàn `Payment.AmountRefunded`. |
| **BR-PY20** | Hoàn tiền cho đơn COD được thực hiện qua chuyển khoản ngân hàng hoặc phương thức hoàn tiền riêng (PAY-Q06). |
| **BR-PY21** | Giao dịch hoàn tiền phải lưu thông tin mã giao dịch cổng và chứng từ ủy nhiệm chi để đối soát. |
| **BR-PY22** | Tuyệt đối không xóa vật lý (`No Hard Delete`) hồ sơ thanh toán, yêu cầu hoàn tiền và các giao dịch đã phát sinh. |

---

## 6.9. Fulfillment & Shipping (`BR-FS` / `BR-SHIP`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-FS01** | Một `Shipment` thuộc về đúng một `SalesOrder`. |
| **BR-FS02** | Một `Shipment` chỉ xuất phát từ đúng một `Warehouse` (`WarehouseID NOT NULL`, SHIP-Q01). |
| **BR-FS03** | Một `SalesOrder` có thể được chia thành nhiều `Shipment` (Split Shipment theo kho theo BD-03). |
| **BR-FS04** | Chỉ tạo `Shipment` khi đơn hàng đã được xác nhận (`CONFIRMED`) và đủ điều kiện xuất kho (`PROCESSING`). |
| **BR-FS05** | Mỗi `Shipment` phải có ít nhất một dòng hàng `ShipmentLine`. |
| **BR-FS06** | Mỗi `ShipmentLine` tham chiếu đúng một dòng đơn hàng `OrderLine` và một dòng phân bổ kho `OrderWarehouseAllocation`. |
| **BR-FS07** | Tổng số lượng `ShipmentQty` thuộc một Allocation không được vượt quá `AllocatedQty`. |
| **BR-FS08** | Tổng số lượng được đưa vào giao của một `OrderLine` không được vượt quá `OrderedQty`. |
| **BR-FS09** | `Warehouse` của `Shipment` phải trùng với `Warehouse` của `OrderWarehouseAllocation` liên quan. |
| **BR-FS10** | Mỗi `Shipment` được thực hiện bởi một đơn vị vận chuyển `Carrier` đã được xác nhận. |
| **BR-FS11** | Hệ thống đề xuất `Carrier` tối ưu, nhân viên/điều phối viên xác nhận lựa chọn cuối cùng (SHIP-Q03). |
| **BR-FS12** | Một `Shipment` có thể trải qua nhiều lần giao hàng `DeliveryAttempt`. |
| **BR-FS13** | Số lần giao lại bị giới hạn theo chính sách vận hành (tối đa 3 lần, SHIP-Q04). |
| **BR-FS14** | Khi vượt giới hạn giao lại thất bại, `Shipment` tự động chuyển sang quy trình chuyển hoàn (`RETURNING`). |
| **BR-FS15** | Mỗi lần giao phải lưu thời gian và kết quả; giao thất bại phải ghi nhận lý do cụ thể (`FailureReason`). |
| **BR-FS16** | Khi giao thành công, bắt buộc lưu người thực tế nhận hàng (`RecipientName`) và thời điểm giao (`DeliveredAt`). |
| **BR-FS17** | Trạng thái vận chuyển có thể được cập nhật thủ công bởi nhân viên hoặc tự động qua tích hợp API/Webhook đối tác (SHIP-Q06). |
| **BR-FS18** | Mỗi lần thay đổi trạng thái `Shipment` phải được ghi nhận vào lịch sử `ShipmentStatusHistory`. |
| **BR-FS19** | `SalesOrder` lưu tổng phí vận chuyển tính cho khách; `Shipment` lưu phí vận chuyển thực tế phân bổ (SHIP-Q07). |
| **BR-FS20** | Tổng phí phân bổ cho các Shipment phải nhất quán với phí vận chuyển của đơn hàng theo chính sách đối soát. |
| **BR-FS21** | Bàn giao hàng cho `Carrier` (`DISPATCHED`) phải kích hoạt ghi nhận xuất kho sổ cái `InventoryTransaction` loại `SHIPMENT_OUT` (`OnHandDelta = -Qty`, `ReservedDelta = -Qty`). |
| **BR-FS22** | Chỉ ghi nhận hàng chuyển hoàn vào tồn kho (`DELIVERY_RETURN_IN`) khi thủ kho thực tế nhận lại và kiểm đếm hàng tại kho xuất ban đầu (BD-07). |

---

## 6.10. Return & Exchange (`BR-RT`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-RT01** | Yêu cầu trả hàng `ReturnRequest` phải thuộc một đơn hàng `SalesOrder` hợp lệ của một `Customer`. |
| **BR-RT02** | Khách hàng hoặc nhân viên CSKH được phép tạo yêu cầu trả hàng thay cho khách hàng (RET-Q01). |
| **BR-RT03** | Tài khoản tạo yêu cầu `CreatedByAccountID` phải được ghi nhận bất biến để phục vụ truy vết. |
| **BR-RT04** | Chỉ được yêu cầu trả hàng trong thời hạn chính sách cấu hình kể từ khi đơn giao thành công (RET-Q02). |
| **BR-RT05** | Một yêu cầu trả hàng bắt buộc phải có ít nhất một dòng chi tiết sản phẩm `ReturnRequestLine`. |
| **BR-RT06** | Dòng hàng yêu cầu trả phải thuộc các dòng hàng `OrderLine` của đơn hàng gốc. |
| **BR-RT07** | Số lượng yêu cầu trả `RequestedQty` không được vượt số lượng còn đủ điều kiện trả của dòng đơn hàng. |
| **BR-RT08** | Cho phép khách hàng yêu cầu trả một phần số lượng của một dòng hàng (RET-Q03). |
| **BR-RT09** | Cho phép tạo nhiều yêu cầu trả hàng cho cùng một dòng hàng nếu vẫn còn số lượng đủ điều kiện. |
| **BR-RT10** | Nhân viên CSKH/Kiểm duyệt bắt buộc phải phê duyệt yêu cầu (`APPROVED`) trước khi khách gửi hàng (RET-Q04). |
| **BR-RT11** | Mỗi yêu cầu trả hàng chỉ định ĐÚNG MỘT kho tiếp nhận `WarehouseID` (RET-Q06). |
| **BR-RT12** | Tất cả các phiếu tiếp nhận `ReturnReceipt` của một yêu cầu phải xử lý tại đúng kho đã chỉ định. |
| **BR-RT13** | Một yêu cầu trả hàng có thể được kho tiếp nhận qua nhiều đợt phiếu nhận. |
| **BR-RT14** | Tổng số lượng tiếp nhận thực tế không được vượt quá số lượng đã được phê duyệt `ApprovedQty`. |
| **BR-RT15** | Hàng nhận về kho bắt buộc phải được kiểm định và phân loại thành `SELLABLE`, `QUARANTINE` hoặc `DISPOSED` (RET-Q05). |
| **BR-RT16** | Chỉ hàng đạt chuẩn `SELLABLE` và được ký duyệt (`PostedAt`) mới được tạo `CUSTOMER_RETURN_IN` nhập lại tồn kho bán. |
| **BR-RT17** | Hàng phân loại `QUARANTINE` (chờ thẩm định) tuyệt đối không được tính vào tồn kho bán được. |
| **BR-RT18** | Hàng phân loại `DISPOSED` (hư hỏng, loại bỏ) tuyệt đối không được tính vào tồn kho bán được. |
| **BR-RT19** | Mọi biến động tồn kho do hàng trả về kho phải được ghi nhận bằng chứng từ sổ cái `InventoryTransaction`. |
| **BR-RT20** | Chỉ xác định số tiền đủ điều kiện hoàn và kích hoạt tạo `Refund` sau khi kho tiếp nhận và kiểm định hàng (RET-Q07). |
| **BR-RT21** | Yêu cầu hoàn tiền được thực thi thông qua phân hệ Subject Area 09 – Payment & Refund. |
| **BR-RT22** | Mọi lần thay đổi trạng thái yêu cầu trả hàng phải được ghi nhận bất biến trong `ReturnStatusHistory` (RET-Q08). |
| **BR-RT23** | Tuyệt đối không xóa vật lý (`No Hard Delete`) hồ sơ yêu cầu, phiếu tiếp nhận và kết quả kiểm tra hàng trả. |

---

## 6.11. Identity & Access Control (`BR-ID`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-ID01** | `Account` phải có ít nhất một trong hai thông tin `Email` hoặc `Phone`. |
| **BR-ID02** | `Email` nếu được cung cấp phải duy nhất toàn hệ thống. |
| **BR-ID03** | `Phone` nếu được cung cấp phải duy nhất toàn hệ thống. |
| **BR-ID04** | Mỗi `Account` chỉ được liên kết với `Customer` hoặc `Employee`, không được đồng thời cả hai (Ràng buộc loại trừ XOR). |
| **BR-ID05** | Một `Account` có thể được cấp nhiều `Role`. |
| **BR-ID06** | Không được cấp trùng lặp một `Role` cho cùng một `Account`. |
| **BR-ID07** | Một `Role` có thể bao gồm nhiều `Permission` và không được cấu hình trùng lặp. |
| **BR-ID08** | `Customer` có thể sở hữu nhiều địa chỉ giao hàng (`CustomerAddress`). |
| **BR-ID09** | Mỗi `Customer` có tối đa một địa chỉ mặc định (`IsDefault = 1`). |
| **BR-ID10** | Địa chỉ đầu tiên của `Customer` tự động được đặt làm địa chỉ mặc định. |
| **BR-ID11** | Khi địa chỉ mặc định bị vô hiệu hóa, phải chọn địa chỉ mặc định mới nếu còn địa chỉ sử dụng được. |
| **BR-ID12** | Mọi yêu cầu xác minh (`AccountVerification`) phải gắn với một `Account` và có thời hạn hiệu lực `ExpiresAt`. |
| **BR-ID13** | Không được sử dụng lại yêu cầu xác minh đã hết hạn hoặc đã hoàn thành. |
| **BR-ID14** | Thay đổi `Email` hoặc `Phone` cần thực hiện quy trình xác minh thông tin mới trước khi sử dụng làm thông tin đăng nhập đã xác minh. |

---

## 6.12. Shopping (`BR-SH`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SH01** | Một Customer có thể có nhiều ShoppingCart trong lịch sử nhưng tối đa một giỏ ACTIVE tại một thời điểm. |
| **BR-SH02** | Mỗi ShoppingCart thuộc đúng một Customer. |
| **BR-SH03** | Một ShoppingCart có thể chứa nhiều CartItem. |
| **BR-SH04** | Một CartItem tham chiếu đúng một ProductVariant. |
| **BR-SH05** | Một SKU chỉ xuất hiện tối đa một lần trong cùng một ShoppingCart. |
| **BR-SH06** | Khi thêm SKU đã tồn tại, hệ thống cộng dồn Quantity. |
| **BR-SH07** | Quantity của CartItem phải lớn hơn 0. |
| **BR-SH08** | SKU không khả dụng vẫn được giữ trong CartItem và hệ thống phải cảnh báo khách hàng. |
| **BR-SH09** | Không lưu giá bán chính thức trong CartItem; lấy giá hiện hành khi hiển thị giỏ hàng. |
| **BR-SH10** | Giá bán được kiểm tra lại và snapshot khi tạo đơn hàng. |
| **BR-SH11** | ShoppingCart được lưu tập trung trên server và đồng bộ giữa các thiết bị của cùng khách hàng. |
| **BR-SH12** | Khi chuyển đổi thành đơn hàng thành công, ShoppingCart chuyển sang CONVERTED và được bảo toàn. |
| **BR-SH13** | Khi khách hàng tiếp tục mua sắm sau khi giỏ hàng đã CONVERTED, hệ thống tạo ShoppingCart mới. |
| **BR-SH14** | Thêm sản phẩm vào giỏ không làm thay đổi OnHandQty hoặc ReservedQty. |
| **BR-SH15** | Không cho phép chỉnh sửa CartItem thuộc ShoppingCart đã CONVERTED. |

---

## 6.13. Sales Order (`BR-SA`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SA01** | Mỗi đơn hàng phải thuộc về đúng một khách hàng đã đăng nhập (`CustomerID` NOT NULL, BD-02: cấm Guest Checkout). |
| **BR-SA02** | Mỗi đơn hàng được tạo từ đúng một giỏ hàng (`CartID` NOT NULL). |
| **BR-SA03** | Một giỏ hàng chỉ được chuyển đổi thành tối đa một đơn hàng (`CartID` UNIQUE trong `SalesOrder`). |
| **BR-SA04** | Một đơn hàng phải có ít nhất một dòng hàng (cấm tạo đơn hàng rỗng). |
| **BR-SA05** | Mỗi dòng hàng phải tham chiếu đúng một SKU (`ProductVariant`). |
| **BR-SA06** | Số lượng đặt mua (`OrderedQty`) của mỗi dòng hàng phải lớn hơn 0. |
| **BR-SA07** | Khi tạo đơn, hệ thống phải giữ đủ toàn bộ hàng trong một giao dịch nguyên tử (Atomic); nếu giữ hàng thất bại, không tạo đơn chính thức. |
| **BR-SA08** | Trạng thái ban đầu của đơn hàng là `PENDING_PAYMENT`. |
| **BR-SA09** | Đơn hàng chỉ chuyển sang `CONFIRMED` khi đáp ứng điều kiện xác nhận thanh toán (thanh toán online thành công hoặc COD hợp lệ). |
| **BR-SA10** | Sau khi đơn hàng được xác nhận (`CONFIRMED`), tuyệt đối không cho phép sửa trực tiếp các dòng hàng (`OrderLine`). |
| **BR-SA11** | Thông tin sản phẩm, SKU, đơn giá và toàn bộ thuộc tính biến thể phải được snapshot tại thời điểm đặt hàng. |
| **BR-SA12** | Đơn hàng phải lưu snapshot đầy đủ địa chỉ giao hàng tại thời điểm tạo đơn (`OrderAddress`), độc lập với sổ địa chỉ `CustomerAddress`. |
| **BR-SA13** | Mỗi đơn hàng có đúng một địa chỉ giao hàng snapshot (`SalesOrder 1 ─── 1 OrderAddress`). |
| **BR-SA14** | Mỗi lần thay đổi trạng thái đơn hàng phải được ghi nhận vào lịch sử trạng thái (`OrderStatusHistory`) kèm thời điểm, tác nhân và lý do. |
| **BR-SA15** | Tuyệt đối không xóa vật lý (`No Hard Delete`) đơn hàng và dòng hàng đã phát sinh. |
| **BR-SA16** | Việc hủy đơn phải tuân thủ chính sách thời gian tự hủy (khi chưa đóng gói) và quy trình xử lý của nhân viên (khi đã đóng gói/giao vận). |
| **BR-SA17** | Không được chuyển giỏ hàng sang `CONVERTED` nếu quá trình tạo đơn và giữ hàng thất bại. |
| **BR-SA18** | Tổng tiền đơn hàng phải nhất quán với tiền hàng, giảm giá và phí vận chuyển đã snapshot: $TotalAmount = SubtotalAmount - DiscountAmount + ShippingFee$. |

---

## 6.14. Audit & Reporting (`BR-AR`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-AR01** | Hệ thống chỉ ghi `AuditLog` đối với các thao tác quan trọng và nhạy cảm đã được xác định (AR-Q01). |
| **BR-AR02** | Mỗi `AuditLog` phải ghi nhận loại hành động, đối tượng tác động (`EntityName`, `RecordID`) và thời điểm thực hiện. |
| **BR-AR03** | `AuditLog` phải ghi nhận tài khoản thực hiện `AccountID` nếu hành động được thực hiện bởi người dùng. |
| **BR-AR04** | Các thao tác do hệ thống tự động chạy ngầm có thể không có `AccountID` nhưng phải xác định được nguồn thực hiện. |
| **BR-AR05** | Các trường thuộc danh sách theo dõi phải ghi nhận giá trị trước `OldValue` và giá trị sau `NewValue` trong `AuditLogDetail` khi có thay đổi (AR-Q02). |
| **BR-AR06** | Tuyệt đối cấm ghi dữ liệu bí mật, mật khẩu, mã OTP, token hoặc thông tin thẻ ngân hàng vào `AuditLog` và `AuditLogDetail`. |
| **BR-AR07** | Tuyệt đối cấm chỉnh sửa hoặc xóa `AuditLog` qua bất kỳ chức năng nghiệp vụ nào (`Append-Only / WORM`, AR-Q03). |
| **BR-AR08** | `AuditLog` có thời hạn lưu trữ hoạt động được cấu hình qua `RetentionUntil` (AR-Q04). |
| **BR-AR09** | `AuditLog` hết hạn được chuyển vào kho lưu trữ lạnh (Cold Archive) theo chính sách lưu trữ của doanh nghiệp. |
| **BR-AR10** | Mỗi báo cáo được cung cấp cho người dùng phải có `ReportDefinition` tương ứng trong danh mục. |
| **BR-AR11** | Mỗi lần chạy hoặc xuất báo cáo ra file bắt buộc phải tạo một bản ghi `ReportExecution` mới (AR-Q06). |
| **BR-AR12** | `ReportExecution` phải lưu tài khoản yêu cầu, bộ tham số lọc và trạng thái thực thi. |
| **BR-AR13** | Người dùng chỉ được chạy báo cáo khi có quyền phù hợp được cấu hình trong `ReportPermission` (AR-Q07). |
| **BR-AR14** | Báo cáo vận hành hàng ngày được phép truy vấn trực tiếp từ dữ liệu nghiệp vụ qua các view/index tối ưu (AR-Q05). |
| **BR-AR15** | Báo cáo tài chính/kế toán cần chốt số liệu kỳ phải sử dụng `ReportSnapshot` để bảo đảm số liệu cố định bất biến. |
| **BR-AR16** | `ReportSnapshot` phải lưu kỳ báo cáo (`PeriodStart`, `PeriodEnd`) và thời điểm chốt số liệu. |
| **BR-AR17** | Snapshot đã chốt tuyệt đối không được thay đổi dữ liệu trực tiếp. |
| **BR-AR18** | Một báo cáo có thể được cấp quyền cho nhiều `Role` thông qua `ReportPermission`. |
| **BR-AR19** | Quyền xem (`CanView`), quyền xuất file (`CanExport`), và quyền chốt kỳ (`CanCreateSnapshot`) được phân biệt độc lập. |
| **BR-AR20** | Không tạo dữ liệu snapshot trùng lặp nếu không có nhu cầu nghiệp vụ hoặc tối ưu hiệu năng rõ ràng. |
