# PHẦN I & II: XÁC ĐỊNH PHẠM VI HỆ THỐNG VÀ PHÂN TÍCH NGHIỆP VỤ

---

## PHẦN I. XÁC ĐỊNH PHẠM VI HỆ THỐNG

### 1.1. Bối cảnh bài toán
Hệ thống được xây dựng cho một doanh nghiệp bán lẻ quy mô vừa chuyên kinh doanh hai nhóm ngành hàng chủ lực:
* **Thời trang:** Áo, quần, giày, phụ kiện.
* **Đồ gia dụng:** Nội thất, đồ trang trí, thiết bị gia đình.

**Đặc điểm vận hành của doanh nghiệp:**
* Sản phẩm có nhiều biến thể (Variants): kích thước, màu sắc, chất liệu.
* Sở hữu và vận hành nhiều kho hàng (Multi-warehouse).
* Bán hàng trực tiếp qua kênh Website thương mại điện tử của doanh nghiệp.
* Có chương trình khuyến mãi đa dạng và chính sách khách hàng thân thiết.

> **Lý do lựa chọn mô hình:** Đây là mô hình lý tưởng để nghiên cứu thiết kế cơ sở dữ liệu vì một sản phẩm không đơn thuần là một dòng dữ liệu tĩnh. Nó bao gồm nhiều biến thể (SKU), nhiều mức giá theo thời điểm, được lưu trữ ở nhiều vị trí kho khác nhau và có lịch sử biến động giao dịch phức tạp.

---

### 1.2. Phạm vi chức năng (Functional Scope)

#### Danh mục phân hệ
| Phân hệ (Module) | Chức năng chính | Mức độ ưu tiên |
| :--- | :--- | :--- |
| **Authentication** | Đăng ký, đăng nhập, phân quyền (RBAC) | Bắt buộc |
| **Product Catalog** | Danh mục, thương hiệu, sản phẩm, biến thể (SKU), thuộc tính | Bắt buộc |
| **Inventory** | Quản lý kho, tồn kho, nhập xuất, giữ hàng | Bắt buộc |
| **Customer** | Hồ sơ khách hàng, sổ địa chỉ, lịch sử mua hàng | Bắt buộc |
| **Cart** | Quản lý giỏ hàng | Bắt buộc |
| **Order** | Đặt hàng, xử lý vòng đời đơn hàng | Bắt buộc |
| **Payment** | Thanh toán, đối soát giao dịch, hoàn tiền | Bắt buộc |
| **Promotion** | Chương trình khuyến mãi, Voucher, giảm giá | Bắt buộc |
| **Shipping** | Địa chỉ giao hàng, vận đơn, theo dõi trạng thái vận chuyển | Bắt buộc |
| **Return** | Yêu cầu đổi trả, kiểm định hàng hoàn | Nâng cao |
| **Reporting** | Dashboard quản trị, báo cáo thống kê doanh thu | Bắt buộc |
| **Loyalty** | Điểm thưởng, hạng thành viên | Để mở rộng |

#### Giới hạn phạm vi (Out of Scope / Constraints) để giữ quy mô Medium:
* **Mô hình Single-tenant / Single-vendor:** Phục vụ một doanh nghiệp bán lẻ duy nhất, không phải sàn thương mại điện tử nhiều nhà bán (Marketplace).
* **Quản trị tập trung:** Một hệ thống quản trị duy nhất nhưng hỗ trợ quản lý nhiều kho hàng vật lý.
* **Kênh bán hàng:** Một Website bán hàng trực tuyến.
* **Thanh toán:** Tích hợp cổng thanh toán ở mức mô phỏng (Mock) hoặc môi trường thử nghiệm (Sandbox).
* **Kiến trúc phần mềm:** Monolith phân lớp rõ ràng, không triển khai Microservices.
* **Nền tảng:** Chỉ xây dựng Web Application (Responsive), không xây dựng ứng dụng Mobile Native.
* **Tích hợp bên thứ ba:** Không tích hợp thực tế với Shopee, TikTok Shop hay các nền tảng đa kênh bên ngoài ở phiên bản đầu tiên.

---

### 1.3. Các tác nhân hệ thống (Actors)

| Actor | Vai trò trong hệ thống |
| :--- | :--- |
| **Guest** | Người dùng chưa đăng nhập; tìm kiếm và xem thông tin sản phẩm, danh mục, khuyến mãi. |
| **Customer** | Khách hàng đã có tài khoản; quản lý giỏ hàng, đặt hàng, thanh toán, theo dõi đơn và gửi yêu cầu đổi trả. |
| **Sales Staff** | Nhân viên bán hàng; tiếp nhận, xác nhận và xử lý đơn hàng, hỗ trợ khách hàng. |
| **Warehouse Staff** | Nhân viên kho; quản lý nhập kho, xuất kho, kiểm kê, chuẩn bị hàng và đóng gói bàn giao vận chuyển. |
| **Marketing Staff** | Nhân viên tiếp thị; thiết lập và quản lý các chương trình khuyến mãi, phát hành voucher. |
| **Admin** | Quản trị viên toàn hệ thống; quản lý người dùng, phân quyền, cấu hình hệ thống và xem báo cáo tổng hợp. |

> **Quyết định thiết kế phân quyền:**
> * Một người dùng có thể sở hữu nhiều vai trò (Roles) nếu doanh nghiệp cho phép.
> * **Tuyệt đối không** thiết kế mỗi vai trò thành một bảng User riêng biệt. Hệ thống sử dụng mô hình **RBAC (Role-Based Access Control)**.

---

## PHẦN II. KHẢO SÁT VÀ PHÂN TÍCH NGHIỆP VỤ

> *Nếu khảo sát sai nghiệp vụ, sơ đồ ERD đẹp đến đâu cũng không cứu được hệ thống.*

### 2.1. Các quy trình nghiệp vụ chính (Core Business Processes)

#### Quy trình 1 – Quản lý sản phẩm (Product Management)
1. Nhân viên tạo danh mục sản phẩm (Category) và thương hiệu (Brand).
2. Tạo thông tin sản phẩm gốc (Product) với tên, mô tả, thương hiệu, hình ảnh.
3. Khai báo các thuộc tính biến thể như màu sắc (Color), kích thước (Size), chất liệu.
4. Hệ thống sinh tự động hoặc ghi nhận mã **SKU** riêng biệt cho từng biến thể (Product Variant).
5. Thiết lập giá bán và trạng thái kinh doanh cho từng biến thể.
6. Sản phẩm được công khai trên website khi đáp ứng đủ điều kiện dữ liệu.

#### Quy trình 2 – Nhập kho (Procurement & Stock Receiving)
1. Nhân viên kho tạo phiếu nhập kho (Purchase Order / Stock Receipt).
2. Chọn nhà cung cấp (Supplier) và kho nhận hàng (Warehouse).
3. Thêm danh sách các SKU cùng số lượng nhập và đơn giá nhập.
4. Xác nhận hoàn tất phiếu nhập kho.
5. Hệ thống cập nhật tăng số lượng tồn kho thực tế và ghi nhận bản ghi lịch sử biến động tồn kho (Stock Movement).

#### Quy trình 3 – Đặt hàng (Checkout & Order Placement)
1. Khách hàng thêm biến thể sản phẩm (SKU) vào giỏ hàng.
2. Tiến hành thanh toán: chọn địa chỉ giao hàng và phương thức thanh toán.
3. Hệ thống kiểm tra tính hợp lệ của giá bán hiện hành, điều kiện áp dụng voucher và số lượng tồn kho khả dụng.
4. Hệ thống tính toán tổng tiền đơn hàng (Subtotal, Discount, Shipping Fee, Total Amount).
5. Khởi tạo đơn hàng (Order) và chi tiết đơn hàng (Order Details) kèm dữ liệu snapshot.
6. Thực hiện **giữ hàng trong kho (Inventory Reservation)** để tránh bán vượt tồn kho.
7. Khách hàng thực hiện thanh toán trực tuyến hoặc xác nhận hình thức COD.

#### Quy trình 4 – Xử lý đơn hàng (Order Fulfillment)
1. Nhân viên bán hàng kiểm tra và xác nhận đơn hàng.
2. Bộ phận kho nhận lệnh, lấy hàng và chuẩn bị hàng theo SKU đã giữ.
3. Đóng gói kiện hàng và bàn giao cho đơn vị vận chuyển (tạo Shipment).
4. Cập nhật trạng thái vận đơn và trạng thái giao hàng theo thời gian thực.
5. Xác nhận giao hàng thành công đến tay khách hàng (trừ tồn kho thực tế và giải phóng lượng hàng giữ).
6. Chuyển trạng thái hoàn tất đơn hàng (Completed).

#### Quy trình 5 – Đổi trả và hoàn tiền (Return & Refund)
1. Khách hàng gửi yêu cầu đổi trả hàng kèm lý do và hình ảnh/thông tin chi tiết.
2. Nhân viên kiểm tra điều kiện đổi trả theo chính sách của doanh nghiệp.
3. Phê duyệt hoặc từ chối yêu cầu đổi trả.
4. Kho tiếp nhận hàng hoàn về và thực hiện kiểm tra chất lượng thực tế (Condition).
5. Hệ thống cập nhật tăng lại tồn kho nếu sản phẩm hoàn về đủ điều kiện nhập lại để bán tiếp.
6. Thực hiện giao dịch hoàn tiền (Refund) tham chiếu đến giao dịch thanh toán gốc hoặc xử lý đơn đổi hàng.

---

### 2.2. Danh mục Tài liệu Cần Xây dựng Trước Khi Vẽ ERD

Trước khi bắt tay vào thiết kế ERD ở Milestone M2, dự án bắt buộc phải hoàn thiện các tài liệu phân tích sau:

| Tài liệu | Nội dung trọng tâm |
| :--- | :--- |
| **Project Charter** | Mục tiêu dự án, phạm vi hệ thống, các giới hạn kỹ thuật và nghiệp vụ |
| **Stakeholder List** | Danh sách các bên liên quan và mối quan tâm/nhu cầu thông tin của họ |
| **Use Case Diagram** | Sơ đồ tổng thể các chức năng hệ thống phân bổ theo từng Actor |
| **Use Case Specification** | Đặc tả chi tiết từng Use Case: Tiền điều kiện, Hậu điều kiện, Luồng chính, Luồng nhánh, Luồng ngoại lệ |
| **Business Process** | Sơ đồ quy trình nghiệp vụ (BPMN / Activity Diagram) |
| **Business Rules** | Danh mục quy tắc nghiệp vụ có mã định danh duy nhất (Căn cứ kiểm chứng ERD và Constraints) |
| **Functional Requirements** | Đặc tả yêu cầu chức năng chi tiết theo từng phân hệ |
| **Non-functional Requirements** | Yêu cầu phi chức năng: Hiệu năng, Bảo mật, Toàn vẹn giao dịch, Khả năng chịu tải đồng thời |
| **Data Requirements** | Yêu cầu về dữ liệu cần lưu trữ, lịch sử cần bảo toàn và nhu cầu truy vấn báo cáo |