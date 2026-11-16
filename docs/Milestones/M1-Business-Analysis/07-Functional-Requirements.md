# 07. FUNCTIONAL REQUIREMENTS – YÊU CẦU CHỨC NĂNG (BASELINE v1.0)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline
* **Tổng số yêu cầu chức năng:** 55 yêu cầu

---

## Danh mục Yêu cầu Chức năng theo Phân hệ

| ID | Phân hệ | Yêu cầu chức năng |
| :--- | :--- | :--- |
| **FR-AUTH-01** | Authentication | Đăng ký tài khoản |
| **FR-AUTH-02** | Authentication | Đăng nhập và đăng xuất |
| **FR-AUTH-03** | Authentication | Đổi và đặt lại mật khẩu |
| **FR-AUTH-04** | Authorization | Phân quyền theo vai trò |
| **FR-PRO-01** | Product | Tạo, sửa, ngừng kinh doanh sản phẩm |
| **FR-PRO-02** | Product | Quản lý danh mục nhiều cấp |
| **FR-PRO-03** | Product | Quản lý thương hiệu |
| **FR-PRO-04** | Product | Quản lý SKU và biến thể |
| **FR-PRO-05** | Product | Quản lý hình ảnh sản phẩm |
| **FR-PRO-06** | Product | Tìm kiếm, lọc và sắp xếp sản phẩm |
| **FR-INV-01** | Inventory | Quản lý nhiều kho |
| **FR-INV-02** | Inventory | Tra cứu tồn kho theo SKU và kho |
| **FR-INV-03** | Inventory | Tạo và xác nhận phiếu nhập |
| **FR-INV-04** | Inventory | Chuyển hàng giữa các kho |
| **FR-INV-05** | Inventory | Điều chỉnh tồn kho có kiểm soát |
| **FR-INV-06** | Inventory | Xem lịch sử biến động tồn kho |
| **FR-PO-01** | Procurement | Quản lý nhà cung cấp |
| **FR-PO-02** | Procurement | Tạo đơn mua hàng |
| **FR-PO-03** | Procurement | Theo dõi tình trạng nhận hàng |
| **FR-CUS-01** | Customer | Quản lý hồ sơ khách hàng |
| **FR-CUS-02** | Customer | Quản lý nhiều địa chỉ giao hàng |
| **FR-CART-01** | Cart | Thêm sản phẩm vào giỏ |
| **FR-CART-02** | Cart | Cập nhật số lượng sản phẩm |
| **FR-CART-03** | Cart | Xóa sản phẩm khỏi giỏ |
| **FR-CART-04** | Cart | Kiểm tra tính hợp lệ của giỏ hàng |
| **FR-ORD-01** | Order | Tạo đơn hàng |
| **FR-ORD-02** | Order | Tra cứu đơn hàng |
| **FR-ORD-03** | Order | Xác nhận đơn hàng |
| **FR-ORD-04** | Order | Hủy đơn hàng |
| **FR-ORD-05** | Order | Quản lý lịch sử trạng thái |
| **FR-ORD-06** | Order | Lưu snapshot thông tin giao dịch |
| **FR-PROM-01** | Promotion | Quản lý chương trình khuyến mãi |
| **FR-PROM-02** | Promotion | Quản lý voucher |
| **FR-PROM-03** | Promotion | Kiểm tra điều kiện áp dụng |
| **FR-PROM-04** | Promotion | Ghi nhận lịch sử sử dụng voucher |
| **FR-PAY-01** | Payment | Quản lý phương thức thanh toán |
| **FR-PAY-02** | Payment | Khởi tạo thanh toán |
| **FR-PAY-03** | Payment | Ghi nhận kết quả thanh toán |
| **FR-PAY-04** | Payment | Theo dõi lịch sử giao dịch |
| **FR-PAY-05** | Payment | Quản lý hoàn tiền |
| **FR-SHIP-01** | Shipping | Tạo vận đơn |
| **FR-SHIP-02** | Shipping | Cập nhật trạng thái giao hàng |
| **FR-SHIP-03** | Shipping | Tra cứu hành trình |
| **FR-RET-01** | Return | Tạo yêu cầu đổi trả |
| **FR-RET-02** | Return | Phê duyệt hoặc từ chối yêu cầu |
| **FR-RET-03** | Return | Tiếp nhận và kiểm tra hàng hoàn |
| **FR-RET-04** | Return | Xử lý hoàn tiền |
| **FR-REP-01** | Reporting | Báo cáo doanh thu |
| **FR-REP-02** | Reporting | Báo cáo đơn hàng |
| **FR-REP-03** | Reporting | Báo cáo tồn kho |
| **FR-REP-04** | Reporting | Báo cáo sản phẩm bán chạy |
| **FR-REP-05** | Reporting | Báo cáo hiệu quả voucher |
| **FR-AUD-01** | Audit | Ghi nhận các hoạt động quản trị quan trọng |