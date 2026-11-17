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

## 6.7. Promotion (`BR-PM`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PM01** | Chương trình khuyến mãi phải có thời gian hiệu lực hợp lệ. |
| **BR-PM02** | Voucher phải có mã duy nhất. |
| **BR-PM03** | Voucher không được sử dụng trước thời gian bắt đầu. |
| **BR-PM04** | Voucher không được sử dụng sau thời gian kết thúc. |
| **BR-PM05** | Voucher phải tuân thủ giới hạn lượt sử dụng. |
| **BR-PM06** | Voucher phải thỏa mãn điều kiện áp dụng của chương trình. |
| **BR-PM07** | Mức giảm giá không được vượt giới hạn quy định. |
| **BR-PM08** | Phải có quy tắc xử lý khi nhiều chương trình cùng áp dụng. |
| **BR-PM09** | Việc sử dụng voucher phải được ghi nhận để tránh sử dụng trùng. |
| **BR-PM10** | Một đơn hàng có thể áp dụng tối đa một chương trình khuyến mãi giảm giá trực tiếp trên dòng sản phẩm, nếu chương trình đáp ứng điều kiện áp dụng. |
| **BR-PM11** | Một đơn hàng có thể áp dụng tối đa một voucher cấp đơn hàng. |
| **BR-PM12** | Khuyến mãi cấp sản phẩm và voucher cấp đơn hàng chỉ được kết hợp khi các chương trình liên quan cho phép cộng dồn (`IsStackable = 1`) và không vi phạm quy tắc ưu tiên. |
| **BR-PM13** | Hệ thống phải lưu lại các chương trình khuyến mãi thực tế đã được áp dụng, phạm vi áp dụng, mức giảm và giá trị giảm tại thời điểm đặt hàng. |
| **BR-PM14** | Không được áp dụng nhiều chương trình cùng tầng vượt quá giới hạn đã quy định, kể cả khi các chương trình riêng lẻ đều thỏa điều kiện. |

> **Quy tắc tầng khuyến mãi:**
> 1. **Tầng Product Promotion:** Tối đa một chương trình giảm giá trực tiếp trên mỗi dòng sản phẩm.
> 2. **Tầng Order Voucher:** Tối đa một voucher cấp đơn hàng.
> 3. **Cộng dồn:** Hai tầng có thể kết hợp nếu quy tắc cộng dồn cho phép (`IsStackable = 1`).
> 4. **Kiểm toán:** Hệ thống phải xác định thứ tự tính giảm giá và lưu kết quả tính toán (`OrderPromotions`) để kiểm tra lại giao dịch.

---

## 6.8. Payment (`BR-PA`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-PA01** | Một đơn hàng có thể có nhiều lần thử thanh toán. |
| **BR-PA02** | Mỗi giao dịch thanh toán phải có mã định danh riêng. |
| **BR-PA03** | Một giao dịch thành công phải có thông tin đối soát. |
| **BR-PA04** | Tổng tiền thu thành công không được vượt số tiền phải thanh toán, trừ nghiệp vụ điều chỉnh được phê duyệt. |
| **BR-PA05** | Giao dịch hoàn tiền phải tham chiếu khoản thanh toán gốc. |
| **BR-PA06** | Không được hoàn tiền vượt số tiền thực thu còn được phép hoàn. |
| **BR-PA07** | Callback thanh toán trùng lặp không được tạo ra giao dịch thu tiền trùng. |

---

## 6.9. Shipping & Fulfillment (`BR-SHIP`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SHIP01** | Một đơn hàng có thể được chia thành nhiều đợt giao (từ nhiều kho hoặc nhiều đợt đóng gói). |
| **BR-SHIP02** | Mỗi vận đơn phải có mã theo dõi phù hợp. |
| **BR-SHIP03** | Số lượng giao không được vượt số lượng đặt mua còn lại. |
| **BR-SHIP04** | Lịch sử trạng thái vận chuyển phải được bảo toàn. |
| **BR-SHIP05** | Một đơn hàng chỉ được hoàn tất khi đáp ứng điều kiện giao hàng và thanh toán theo phương thức tương ứng. |

---

## 6.10. Return & Refund (`BR-RT`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-RT01** | Chỉ các đơn hàng đủ điều kiện mới được yêu cầu đổi trả. |
| **BR-RT02** | Số lượng yêu cầu trả không được vượt số lượng còn đủ điều kiện trả. |
| **BR-RT03** | Một yêu cầu trả hàng có thể chứa nhiều dòng sản phẩm. |
| **BR-RT04** | Hàng hoàn chỉ được nhập lại tồn kho khi được kiểm tra và chấp nhận. |
| **BR-RT05** | Hoàn tiền phải tuân thủ số tiền thực thu và chính sách hoàn tiền. |
| **BR-RT06** | Lịch sử xử lý yêu cầu trả hàng phải được bảo toàn. |

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
