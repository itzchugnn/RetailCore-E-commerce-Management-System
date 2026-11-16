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
| **BR-PR10** | Mỗi sản phẩm phải có tối đa một danh mục chính (`IsPrimary = 1`) trong số các danh mục mà sản phẩm đang tham gia. Danh mục chính được sử dụng để xác định Breadcrumb mặc định và hỗ trợ báo cáo doanh thu. |
| **BR-PR11** | Thuộc tính biến thể được định nghĩa linh hoạt theo từng sản phẩm. Không mặc định mọi sản phẩm đều có cùng tập thuộc tính như Size, Color hoặc Material. |
| **BR-PR12** | Một SKU phải có một tổ hợp giá trị thuộc tính biến thể xác định trong phạm vi sản phẩm. Không được tồn tại hai SKU có cùng tổ hợp thuộc tính trong cùng một sản phẩm. |

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

## 6.9. Shipping (`BR-SH`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-SH01** | Một đơn hàng có thể được chia thành nhiều đợt giao (từ nhiều kho hoặc nhiều đợt đóng gói). |
| **BR-SH02** | Mỗi vận đơn phải có mã theo dõi phù hợp. |
| **BR-SH03** | Số lượng giao không được vượt số lượng đặt mua còn lại. |
| **BR-SH04** | Lịch sử trạng thái vận chuyển phải được bảo toàn. |
| **BR-SH05** | Một đơn hàng chỉ được hoàn tất khi đáp ứng điều kiện giao hàng và thanh toán theo phương thức tương ứng. |

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
