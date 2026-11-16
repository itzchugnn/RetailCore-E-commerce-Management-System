# 06. BUSINESS RULES – BỘ QUY TẮC NGHIỆP VỤ (BASELINE v1.0)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

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
| **BR-PR05** | Một sản phẩm có thể thuộc nhiều danh mục. |
| **BR-PR06** | Giá bán của SKU không được âm. |
| **BR-PR07** | Sản phẩm đã phát sinh giao dịch không được xóa vật lý tùy tiện. |
| **BR-PR08** | SKU ngừng kinh doanh không được sử dụng trong đơn hàng mới. |
| **BR-PR09** | Thông tin sản phẩm tại thời điểm giao dịch phải được bảo toàn khi cần thiết. |

---

## 6.3. Inventory & Warehouse (`BR-IN`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-IN01** | Một doanh nghiệp có thể có nhiều kho. |
| **BR-IN02** | Tồn kho được quản lý theo từng SKU tại từng kho. |
| **BR-IN03** | Một SKU có thể được lưu trữ tại nhiều kho. |
| **BR-IN04** | Tồn thực tế không được âm. |
| **BR-IN05** | Tồn giữ không được âm và không được lớn hơn tồn thực tế. |
| **BR-IN06** | Tồn khả dụng bằng tồn thực tế trừ tồn giữ ($\text{Available} = \text{OnHand} - \text{Reserved}$). |
| **BR-IN07** | Mọi biến động tồn kho phải có lịch sử. |
| **BR-IN08** | Phiếu nhập đã xác nhận không được chỉnh sửa trực tiếp. |
| **BR-IN09** | Một đơn hàng không được giữ số lượng vượt tồn khả dụng. |
| **BR-IN10** | Hủy đơn phải giải phóng lượng hàng đã giữ tương ứng. |
| **BR-IN11** | Xuất hàng phải được ghi nhận theo kho thực tế xuất hàng. |
| **BR-IN12** | Chuyển kho phải bảo toàn tổng số lượng hàng trong toàn hệ thống, ngoại trừ hao hụt được ghi nhận riêng. |

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

---

## 6.6. Sales Order (`BR-OR`)

| ID | Quy tắc nghiệp vụ |
| :--- | :--- |
| **BR-OR01** | Mỗi đơn hàng phải có mã đơn hàng duy nhất. |
| **BR-OR02** | Một đơn hàng phải có ít nhất một dòng hàng. |
| **BR-OR03** | Một dòng hàng tham chiếu một SKU. |
| **BR-OR04** | Số lượng đặt mua phải lớn hơn $0$. |
| **BR-OR05** | Giá bán tại thời điểm đặt hàng phải được bảo toàn. |
| **BR-OR06** | Thông tin giao hàng tại thời điểm đặt phải được bảo toàn. |
| **BR-OR07** | Tổng tiền phải được tính dựa trên các thành phần tiền hợp lệ của đơn hàng. |
| **BR-OR08** | Đơn hàng phải tuân thủ quy trình chuyển trạng thái hợp lệ. |
| **BR-OR09** | Mọi thay đổi trạng thái quan trọng phải có lịch sử. |
| **BR-OR10** | Không được hủy đơn đã hoàn tất theo quy trình hủy thông thường. |

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
| **BR-SH01** | Một đơn hàng có thể được chia thành nhiều đợt giao. |
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

> **Lưu ý kỹ thuật cho M3 & M4:** Những quy tắc trên là baseline, chưa phải toàn bộ đặc tả chi tiết. Các quy tắc liên quan đến số lượng, trạng thái và tính đồng thời sẽ được chuyển thành các cơ chế kiểm soát phù hợp ở M3/M4 (PK/FK/UNIQUE/CHECK Constraints, Triggers, Stored Procedures với Locking, hoặc Service Layer Validation). Không phải tất cả Business Rules đều có thể biểu diễn đơn thuần bằng một `CHECK Constraint`.