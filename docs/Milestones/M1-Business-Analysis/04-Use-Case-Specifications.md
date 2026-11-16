# 04. USE CASE SPECIFICATIONS – ĐẶC TẢ CHI TIẾT CA SỬ DỤNG

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

Dưới đây là đặc tả chi tiết cho 8 Use Case cốt lõi có ảnh hưởng trực tiếp nhất đến cấu trúc dữ liệu, giao dịch (Transaction) và kiểm soát đồng thời của RetailCore.

---

## UC-01: Place Order – Đặt hàng

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-01` |
| **Primary Actor** | Customer |
| **Supporting Actor** | Payment Provider |
| **Trigger** | Khách hàng chọn chức năng đặt hàng (Checkout) |
| **Priority** | Critical |
| **Preconditions** | - Khách hàng đã đăng nhập.<br>- Giỏ hàng có ít nhất một sản phẩm.<br>- Các SKU đang được phép kinh doanh. |
| **Main Flow** | 1. Khách hàng truy cập Checkout.<br>2. Hệ thống lấy thông tin giỏ hàng.<br>3. Khách hàng chọn địa chỉ giao hàng.<br>4. Khách hàng chọn phương thức thanh toán.<br>5. Khách hàng nhập voucher nếu có.<br>6. Hệ thống kiểm tra tính hợp lệ của voucher.<br>7. Hệ thống kiểm tra tồn kho khả dụng.<br>8. Hệ thống tính tổng tiền đơn hàng.<br>9. Hệ thống hiển thị thông tin xác nhận.<br>10. Khách hàng xác nhận đặt hàng.<br>11. Hệ thống tạo đơn hàng và các chi tiết đơn hàng.<br>12. Hệ thống thực hiện giữ tồn kho.<br>13. Hệ thống ghi nhận thông tin thanh toán ban đầu.<br>14. Hệ thống trả về mã đơn hàng. |
| **Alternative Flows** | - **A1:** Khách hàng không sử dụng voucher, hệ thống bỏ qua bước kiểm tra voucher.<br>- **A2:** Khách hàng chọn COD, hệ thống tạo đơn với phương thức thanh toán khi nhận hàng.<br>- **A3:** Khách hàng sử dụng voucher hợp lệ, hệ thống tính lại tổng tiền. |
| **Exception Flows** | - **E1:** Giỏ hàng trống $\rightarrow$ Không cho phép đặt hàng.<br>- **E2:** SKU hết hàng $\rightarrow$ Thông báo sản phẩm không đủ tồn kho.<br>- **E3:** Voucher hết hạn $\rightarrow$ Yêu cầu thay voucher.<br>- **E4:** Tồn kho thay đổi trong lúc đặt hàng $\rightarrow$ Thực hiện kiểm tra lại trong transaction.<br>- **E5:** Lỗi Database $\rightarrow$ Rollback toàn bộ giao dịch tạo đơn và giữ hàng. |
| **Postconditions** | - Đơn hàng được tạo hợp lệ.<br>- Chi tiết đơn hàng được lưu.<br>- Giá bán và địa chỉ giao hàng được bảo toàn (Snapshot).<br>- Tồn kho được giữ tương ứng.<br>- Đơn hàng có trạng thái ban đầu phù hợp. |

---

## UC-02: Receive Purchase Order – Nhập kho

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-02` |
| **Primary Actor** | Warehouse Staff |
| **Supporting Actor** | Supplier |
| **Priority** | High |
| **Preconditions** | - Nhân viên có quyền nhập kho.<br>- Kho đang hoạt động.<br>- Đơn mua hàng tồn tại nếu nhập theo đơn mua. |
| **Main Flow** | 1. Nhân viên truy cập chức năng nhập kho.<br>2. Chọn kho nhận hàng.<br>3. Chọn nhà cung cấp.<br>4. Chọn đơn mua hàng hoặc tạo phiếu nhập phù hợp.<br>5. Nhập các SKU và số lượng thực nhận.<br>6. Hệ thống kiểm tra dữ liệu nhập.<br>7. Nhân viên xác nhận phiếu nhập.<br>8. Hệ thống cập nhật tồn kho.<br>9. Hệ thống ghi nhận Stock Movement.<br>10. Hệ thống hoàn tất phiếu nhập. |
| **Exception Flows** | - **E1:** SKU không tồn tại $\rightarrow$ Từ chối dòng hàng.<br>- **E2:** Số lượng không hợp lệ $\rightarrow$ Yêu cầu nhập lại.<br>- **E3:** Phiếu nhập đã xác nhận $\rightarrow$ Không cho phép xác nhận lần hai.<br>- **E4:** Lỗi cập nhật tồn kho $\rightarrow$ Rollback toàn bộ giao dịch. |
| **Postconditions** | - Phiếu nhập được xác nhận.<br>- Tồn kho được tăng tương ứng.<br>- Lịch sử biến động tồn kho được ghi nhận. |

---

## UC-03: Reserve Inventory – Giữ hàng

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-03` |
| **Primary Actor** | System |
| **Supporting Actor** | Customer |
| **Priority** | Critical |
| **Preconditions** | - Có yêu cầu đặt hàng hợp lệ.<br>- SKU tồn tại và đang kinh doanh. |
| **Main Flow** | 1. Hệ thống xác định SKU và số lượng cần giữ.<br>2. Kiểm tra tồn khả dụng.<br>3. Khóa hoặc kiểm soát bản ghi tồn kho phù hợp.<br>4. Kiểm tra lại số lượng khả dụng.<br>5. Tăng số lượng `ReservedQty`.<br>6. Tạo thông tin giữ hàng.<br>7. Thiết lập thời hạn giữ hàng.<br>8. Hoàn tất giao dịch. |
| **Exception Flows** | - **E1:** Không đủ tồn kho $\rightarrow$ Từ chối giữ hàng.<br>- **E2:** Yêu cầu giữ hàng trùng lặp $\rightarrow$ Kiểm tra tính idempotent.<br>- **E3:** Lỗi Database $\rightarrow$ Rollback. |
| **Postconditions** | - Tồn giữ được cập nhật chính xác.<br>- Lịch sử giữ hàng được ghi nhận.<br>- Không vượt quá số lượng tồn thực tế. |

---

## UC-04: Process Payment – Thanh toán

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-04` |
| **Primary Actor** | Customer |
| **Supporting Actor** | Payment Provider |
| **Priority** | Critical |
| **Preconditions** | - Đơn hàng tồn tại.<br>- Đơn hàng được phép thanh toán.<br>- Phương thức thanh toán hợp lệ. |
| **Main Flow** | 1. Khách hàng chọn phương thức thanh toán.<br>2. Hệ thống tạo yêu cầu thanh toán.<br>3. Hệ thống gửi thông tin đến Payment Provider nếu thanh toán trực tuyến.<br>4. Payment Provider xử lý giao dịch.<br>5. Payment Provider trả kết quả hoặc gửi thông báo xác nhận.<br>6. Hệ thống xác minh thông tin giao dịch.<br>7. Hệ thống cập nhật trạng thái thanh toán.<br>8. Hệ thống ghi nhận mã tham chiếu giao dịch.<br>9. Hệ thống thông báo kết quả cho khách hàng. |
| **Exception Flows** | - **E1:** Thanh toán thất bại $\rightarrow$ Ghi nhận giao dịch thất bại.<br>- **E2:** Khách hàng hủy thanh toán $\rightarrow$ Ghi nhận trạng thái hủy nếu có kết quả xác nhận.<br>- **E3:** Callback bị gửi nhiều lần $\rightarrow$ Không ghi nhận thanh toán trùng.<br>- **E4:** Timeout $\rightarrow$ Truy vấn lại trạng thái từ nhà cung cấp, không mặc định là thất bại. |
| **Postconditions** | - Giao dịch thanh toán được ghi nhận.<br>- Đơn hàng có trạng thái thanh toán tương ứng.<br>- Có thể truy vết giao dịch với nhà cung cấp. |

---

## UC-05: Process Sales Order – Xử lý đơn hàng

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-05` |
| **Primary Actor** | Sales Staff |
| **Supporting Actor** | Warehouse Staff |
| **Priority** | High |
| **Main Flow** | 1. Nhân viên xem danh sách đơn chờ xử lý.<br>2. Chọn đơn hàng.<br>3. Kiểm tra thông tin khách hàng và thanh toán.<br>4. Xác nhận đơn hàng.<br>5. Chuyển yêu cầu chuẩn bị hàng đến kho.<br>6. Kho thực hiện lấy hàng và đóng gói.<br>7. Nhân viên xác nhận bàn giao.<br>8. Hệ thống tạo hoặc cập nhật vận đơn.<br>9. Đơn hàng được theo dõi cho đến khi giao thành công. |
| **Exception Flows** | - **E1:** Đơn hàng đã bị hủy $\rightarrow$ Không cho phép xử lý.<br>- **E2:** Thanh toán không hợp lệ $\rightarrow$ Không cho phép chuyển sang trạng thái cần thanh toán thành công.<br>- **E3:** Không thể chuẩn bị hàng $\rightarrow$ Chuyển sang quy trình xử lý ngoại lệ.<br>- **E4:** Giao hàng thất bại $\rightarrow$ Ghi nhận trạng thái giao thất bại và xử lý tiếp. |

---

## UC-06: Apply Promotion – Áp dụng khuyến mãi

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-06` |
| **Primary Actor** | Customer |
| **Supporting Actor** | Marketing Staff |
| **Priority** | High |
| **Main Flow** | 1. Khách hàng nhập voucher.<br>2. Hệ thống kiểm tra mã voucher.<br>3. Kiểm tra thời gian hiệu lực.<br>4. Kiểm tra giới hạn sử dụng.<br>5. Kiểm tra điều kiện đơn hàng.<br>6. Kiểm tra phạm vi sản phẩm áp dụng.<br>7. Tính mức giảm giá.<br>8. Trả về số tiền được giảm. |
| **Exception Flows** | - **E1:** Voucher không tồn tại.<br>- **E2:** Voucher hết hạn.<br>- **E3:** Voucher hết lượt sử dụng.<br>- **E4:** Không đạt giá trị đơn hàng tối thiểu.<br>- **E5:** Voucher không áp dụng cho sản phẩm trong giỏ. |

---

## UC-07: Request Return – Yêu cầu đổi trả

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-07` |
| **Primary Actor** | Customer |
| **Supporting Actor** | Sales Staff, Warehouse Staff |
| **Priority** | High |
| **Main Flow** | 1. Khách hàng chọn đơn hàng đã giao.<br>2. Chọn sản phẩm cần trả.<br>3. Nhập số lượng trả và lý do.<br>4. Gửi yêu cầu.<br>5. Hệ thống kiểm tra điều kiện đổi trả.<br>6. Nhân viên xem xét yêu cầu.<br>7. Nhân viên phê duyệt hoặc từ chối.<br>8. Nếu được duyệt, hệ thống tiếp nhận hàng hoàn.<br>9. Kho kiểm tra tình trạng sản phẩm.<br>10. Hệ thống cập nhật kết quả xử lý.<br>11. Hệ thống thực hiện hoàn tiền nếu đủ điều kiện. |
| **Exception Flows** | - **E1:** Đơn hàng không đủ điều kiện đổi trả.<br>- **E2:** Số lượng trả vượt số lượng được phép.<br>- **E3:** Yêu cầu trùng lặp.<br>- **E4:** Hàng hoàn không đạt điều kiện nhập lại kho. |

---

## UC-08: Transfer Inventory – Chuyển kho

| Thuộc tính | Mô tả |
| :--- | :--- |
| **ID** | `UC-08` |
| **Primary Actor** | Warehouse Staff |
| **Priority** | High |
| **Main Flow** | 1. Nhân viên chọn kho nguồn và kho đích.<br>2. Chọn SKU và số lượng chuyển.<br>3. Hệ thống kiểm tra tồn khả dụng tại kho nguồn.<br>4. Tạo phiếu chuyển kho.<br>5. Xác nhận xuất khỏi kho nguồn.<br>6. Ghi nhận hàng đang vận chuyển nội bộ.<br>7. Kho đích xác nhận nhận hàng.<br>8. Cập nhật tồn kho tại kho đích.<br>9. Hoàn tất phiếu chuyển. |
| **Exception Flows** | - **E1:** Kho nguồn không đủ hàng.<br>- **E2:** Kho nguồn và kho đích trùng nhau.<br>- **E3:** Kho đích từ chối số lượng thực nhận.<br>- **E4:** Giao dịch bị gián đoạn giữa hai kho. |