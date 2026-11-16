# PHẦN VII, VIII & IX: THIẾT KẾ BACKEND, FRONTEND VÀ CHIẾN LƯỢC KIỂM THỬ

---

## PHẦN VII. THIẾT KẾ BACKEND (DELIVERABLE 04 – REST API)

### 7.1. Kiến trúc Phân lớp (Layered Architecture)
Backend được xây dựng bằng **Java Spring Boot** kết nối trực tiếp tới **SQL Server**, tuân thủ kiến trúc phân lớp rõ ràng:

1. **Controller Layer:** Tiếp nhận HTTP Request, kiểm tra tính hợp lệ dữ liệu đầu vào (DTO Validation), điều phối gọi Service và trả về HTTP Response chuẩn hóa.
2. **Service Layer:** Xử lý logic nghiệp vụ ở tầng ứng dụng, quản lý ranh giới giao dịch (`@Transactional`), tích hợp cổng thanh toán và phân quyền.
3. **Repository / Data Access Layer:** Giao tiếp với cơ sở dữ liệu thông qua Spring Data JPA hoặc Spring JDBC / `JdbcTemplate` (gọi các Stored Procedure phức tạp).
4. **SQL Server (Database Layer):** Đảm nhiệm toàn vẹn dữ liệu cốt lõi (Schema, Constraints, ACID Transactions, Locking, Indexes, Stored Procedures).

#### Cấu trúc Package đề xuất (`com.retailcore`):
~~~text
com.retailcore
├── common
│   ├── exception
│   ├── response
│   └── security
├── auth
├── user
├── product
├── inventory
├── supplier
├── customer
├── cart
├── order
├── promotion
├── payment
├── shipping
├── returns
└── reporting
~~~

---

### 7.2. Thiết kế REST API Tiêu biểu

| HTTP Method | Endpoint | Chức năng nghiệp vụ | Phân quyền |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/products` | Lấy danh sách sản phẩm (tìm kiếm, lọc, phân trang) | Public |
| `GET` | `/api/products/{id}` | Xem chi tiết sản phẩm, các biến thể SKU và tồn kho | Public |
| `POST` | `/api/admin/products` | Tạo mới sản phẩm và biến thể | Admin / Staff |
| `PUT` | `/api/admin/products/{id}` | Cập nhật thông tin sản phẩm | Admin / Staff |
| `GET` | `/api/inventory` | Tra cứu tồn kho theo kho và SKU | Warehouse / Admin |
| `POST` | `/api/inventory/receipts` | Tạo và xác nhận phiếu nhập kho | Warehouse / Admin |
| `GET` | `/api/cart` | Xem giỏ hàng hiện tại của khách hàng | Customer |
| `POST` | `/api/cart/items` | Thêm biến thể sản phẩm (SKU) vào giỏ hàng | Customer |
| `POST` | `/api/orders` | Đặt hàng (Checkout) | Customer |
| `GET` | `/api/orders/{id}` | Xem chi tiết đơn hàng | Customer (Chủ đơn) / Staff |
| `POST` | `/api/orders/{id}/cancel` | Hủy đơn hàng và giải phóng tồn kho đang giữ | Customer / Sales Staff |
| `POST` | `/api/payments` | Khởi tạo giao dịch thanh toán cho đơn hàng | Customer |
| `POST` | `/api/returns` | Gửi yêu cầu đổi trả hàng | Customer |
| `GET` | `/api/admin/reports/sales` | Truy xuất báo cáo thống kê doanh thu | Admin |

---

### 7.3. Tám Nguyên tắc Xử lý Dữ liệu & Bảo mật Bắt buộc tại Backend

1. **Zero-Trust với dữ liệu giá từ Client:** Tuyệt đối không nhận `TotalAmount` hay `UnitPrice` từ Frontend rồi tin tưởng lưu trực tiếp vào Database.
2. **Tính toán tại Server/Database:** Backend và Database phải tự truy xuất giá gốc từ bảng `ProductVariants`, kiểm tra điều kiện khuyến mãi, tính phí vận chuyển và tổng hợp thành tiền.
3. **Bảo vệ tồn kho:** Không cung cấp API cho phép Frontend của khách hàng tự ý gọi cập nhật số lượng tồn kho trực tiếp.
4. **Đảm bảo tính nguyên tử (Atomicity):** Mọi thao tác làm thay đổi dữ liệu trên nhiều bảng phải nằm trong một Transaction được kiểm soát chặt chẽ.
5. **Chống gửi lặp (Idempotency):** Bắt buộc có cơ chế xử lý request lặp (Double-click / Network retry) đối với API Tạo đơn hàng (`POST /api/orders`) và Thanh toán (`POST /api/payments`).
6. **Chống leo thang đặc quyền (Privilege Escalation):** Không cho phép người dùng tự thay đổi `Role` của chính mình thông qua API cập nhật hồ sơ cá nhân.
7. **Bảo mật thông tin nhạy cảm:** Sử dụng DTO/Projection để đảm bảo không bao giờ trả cột `PasswordHash` về phía Frontend.
8. **Chống lỗi IDOR (Insecure Direct Object References):** Kiểm tra quyền sở hữu dữ liệu nghiêm ngặt, không cho phép Khách hàng A truy cập hoặc hủy đơn hàng của Khách hàng B chỉ bằng cách thay đổi `{id}` trên URL.

> **Nguyên tắc vàng về Giao dịch Thanh toán:**
> Phải tách biệt rõ ràng giữa **Database Transaction** và **Lời gọi mạng tới Cổng thanh toán bên ngoài (External Payment Gateway)**. Tuyệt đối không mở một SQL Transaction giữ khóa (Lock) trong suốt thời gian chờ cổng thanh toán bên ngoài phản hồi qua HTTP, vì sẽ làm cạn kiệt Connection Pool và gây treo toàn bộ bảng `Inventory` / `Orders`.

---

## PHẦN VIII. THIẾT KẾ FRONTEND (DELIVERABLE 05 – WEB APPLICATION)

Công nghệ đề xuất: **React + TypeScript**.

### 8.1. Phân hệ Website dành cho Khách hàng (Customer Storefront)
1. **Home (Trang chủ):** Banner chiến dịch, danh mục ngành hàng (Thời trang, Gia dụng), sản phẩm nổi bật, chương trình khuyến mãi đang diễn ra.
2. **Product Listing (Danh sách sản phẩm):** Tìm kiếm theo từ khóa, bộ lọc đa tiêu chí (danh mục, thương hiệu, khoảng giá, thuộc tính màu/size), sắp xếp và phân trang phía Server.
3. **Product Detail (Chi tiết sản phẩm):** Thư viện hình ảnh, bộ chọn biến thể (Size, Color), hiển thị mã SKU tương ứng, giá bán và trạng thái tồn kho khả dụng theo thời gian thực.
4. **Cart (Giỏ hàng):** Quản lý danh sách SKU đã chọn, cập nhật số lượng, tạm tính.
5. **Checkout (Thanh toán):** Chọn/nhập địa chỉ giao hàng, áp dụng mã Voucher, chọn phương thức thanh toán và xác nhận đặt hàng.
6. **Order Tracking & Order History:** Theo dõi tiến trình xử lý vận đơn và tra cứu lịch sử các đơn hàng đã đặt.
7. **Customer Profile & Address Management:** Quản lý thông tin cá nhân và sổ địa chỉ nhận hàng.

---

### 8.2. Phân hệ Quản trị (Admin Dashboard)

| Module quản trị | Chức năng chi tiết |
| :--- | :--- |
| **Dashboard** | Biểu đồ doanh thu theo kỳ, số lượng đơn hàng mới, top sản phẩm/SKU bán chạy, cảnh báo sắp hết hàng. |
| **Products** | Quản lý thông tin sản phẩm gốc, cấu hình thuộc tính và sinh danh sách biến thể SKU kèm giá bán. |
| **Categories** | Quản lý cây danh mục sản phẩm đa cấp và thương hiệu. |
| **Inventory** | Theo dõi tồn thực tế, tồn đang giữ, tồn khả dụng theo từng kho; tra cứu thẻ kho (`StockMovements`). |
| **Procurement** | Quản lý nhà cung cấp, lập và xác nhận phiếu nhập kho (`PurchaseOrders`). |
| **Orders** | Tiếp nhận, xác nhận đơn hàng, tạo vận đơn giao hàng và cập nhật vòng đời đơn hàng. |
| **Promotions** | Thiết lập chương trình khuyến mãi, cấu hình độ ưu tiên và phát hành mã giảm giá (`Vouchers`). |
| **Customers** | Tra cứu hồ sơ khách hàng và lịch sử giao dịch. |
| **Payments** | Theo dõi lịch sử giao dịch thanh toán, đối soát mã tham chiếu và xử lý hoàn tiền (`Refunds`). |
| **Returns** | Tiếp nhận, phê duyệt yêu cầu đổi trả hàng, cập nhật kết quả kiểm định hàng hoàn. |
| **Users & Roles** | Quản lý tài khoản nhân viên, gán vai trò (Roles) và phân quyền chi tiết (Permissions). |

---

### 8.3. Các Tiêu chí Trải nghiệm Người dùng (UX Criteria)
* **Server-side Pagination:** Phân trang, lọc và sắp xếp được xử lý tại Server/Database để đảm bảo hiệu năng khi dữ liệu lớn.
* **Combined Search & Filter:** Kết hợp mượt mà giữa tìm kiếm từ khóa và nhiều bộ lọc thuộc tính.
* **UI States đầy đủ:** Có giao diện rõ ràng cho trạng thái Đang tải (`Loading Skeleton/Spinner`), Không có dữ liệu (`Empty State`) và Lỗi hệ thống (`Error State`).
* **Form Validation:** Kiểm tra dữ liệu đầu vào tức thời ở cả Client-side và hiển thị thông báo lỗi rõ ràng từ Server-side.
* **Confirmation Dialog:** Hộp thoại xác nhận trước khi thực hiện các thao tác quan trọng hoặc nguy hiểm (hủy đơn, khóa sản phẩm, xác nhận nhập kho).
* **Role-based UI Rendering:** Ẩn/hiện menu và nút chức năng theo đúng quyền hạn (`Role`/`Permission`) của người dùng đang đăng nhập.
* **Order Timeline:** Hiển thị trực quan lịch sử trạng thái đơn hàng và vận đơn.
* **Responsive Design:** Tương thích tốt trên cả màn hình Desktop và thiết bị Mobile.

---

## PHẦN IX. CHIẾN LƯỢC KIỂM THỬ VÀ TRIỂN KHAI

### 9.1. Ma trận các Loại Kiểm thử (Testing Strategy)

| Loại kiểm thử | Nội dung kiểm chứng trọng tâm |
| :--- | :--- |
| **Database Testing** | Kiểm tra sự hoạt động chính xác của toàn bộ `PK`, `FK`, `CHECK`, `UNIQUE`, `DEFAULT`, Triggers và Stored Procedures. |
| **Functional Testing** | Kiểm thử đúng đắn các luồng nghiệp vụ theo đặc tả Use Case và Business Rules. |
| **Integration Testing** | Kiểm thử sự tương tác giữa tầng Backend (Spring Boot) và SQL Server. |
| **Transaction Testing** | Kiểm chứng cơ chế `ROLLBACK` toàn phần khi xảy ra lỗi ở bất kỳ bước trung gian nào trong giao dịch đa bảng. |
| **Concurrency Testing** | Giả lập nhiều luồng (Threads/Connections) đồng thời tranh chấp cùng một SKU hoặc Voucher. |
| **Security Testing** | Kiểm thử xác thực (Authentication), phân quyền RBAC (Authorization), chống IDOR và SQL Injection. |
| **Performance Testing** | Đo lường thời gian phản hồi của các truy vấn phức tạp và API, phân tích Execution Plan với dữ liệu giả lập lớn. |
| **Recovery Testing** | Kiểm thử quy trình Backup và phục hồi (Restore) cơ sở dữ liệu khi xảy ra sự cố. |

---

### 9.2. Mười Kịch bản Kiểm thử Quan trọng Bắt buộc Thực hiện (Core Test Scenarios)

Đây là 10 kịch bản kiểm thử cốt lõi chứng minh giá trị kỹ thuật của dự án vượt xa các ứng dụng thêm-sửa-xóa thông thường:

1. **Chống bán vượt tồn kho (Overselling):** Hai khách hàng cùng nhấn mua biến thể SKU cuối cùng trong kho tại cùng một thời điểm $\rightarrow$ Chỉ 1 người thành công, 1 người nhận thông báo hết hàng, tồn kho không bị âm.
2. **Tranh chấp Voucher lượt cuối:** Hai khách hàng cùng áp dụng một mã Voucher chỉ còn đúng $1$ lượt sử dụng (`UsageLimit - UsedCount = 1`) $\rightarrow$ Chỉ 1 đơn áp dụng thành công.
3. **Chống tạo đơn lặp (Idempotency):** Khách hàng nhấn đúp nút "Đặt hàng" hoặc mạng gửi lặp request tạo đơn 2 lần liên tiếp $\rightarrow$ Hệ thống chỉ tạo duy nhất 1 đơn hàng và chỉ giữ kho 1 lần.
4. **Đứt gãy giữa Cổng thanh toán và Backend:** Khách hàng thanh toán thành công trên cổng thanh toán nhưng kết nối gặp sự cố trước khi Backend kịp cập nhật trạng thái đơn $\rightarrow$ Cơ chế đối soát/Webhook xử lý bù trừ chính xác.
5. **Giải phóng tồn kho khi hủy đơn:** Nhân viên hoặc khách hàng hủy đơn hàng sau khi hệ thống đã giữ hàng (`ReservedQty`) $\rightarrow$ Số lượng giữ được giải phóng chính xác về lại `AvailableQty`.
6. **Trả hàng một phần (Partial Return):** Khách hàng mua 3 sản phẩm trong đơn nhưng chỉ gửi yêu cầu trả lại 1 sản phẩm $\rightarrow$ Tính toán số tiền hoàn trả và cập nhật lại kho cho đúng 1 sản phẩm đó.
7. **Đa biến thể – Đa kho:** Một sản phẩm có nhiều SKU được lưu trữ phân tán ở nhiều kho khác nhau $\rightarrow$ Truy vấn tổng tồn kho và phân bổ kho xuất hàng chính xác.
8. **Tính nguyên tử khi nhập kho:** Phiếu nhập kho gồm 10 dòng SKU bị lỗi dữ liệu ở dòng thứ 9 $\rightarrow$ Toàn bộ phiếu nhập bị Rollback, không có dòng SKU nào bị cộng nhầm vào bảng `Inventory`.
9. **Bảo mật phân quyền (RBAC):** Nhân viên kho hoặc Khách hàng cố tình gọi trực tiếp API dành riêng cho Admin/Marketing $\rightarrow$ Hệ thống chặn và trả về mã lỗi `403 Forbidden`.
10. **Bảo toàn lịch sử giá (Snapshot Integrity):** Quản trị viên thay đổi giá bán của SKU từ $500.000\text{ VNĐ}$ lên $650.000\text{ VNĐ}$ sau khi khách đã đặt hàng $\rightarrow$ Chi tiết đơn hàng cũ và báo cáo doanh thu kỳ trước tuyệt đối không bị sai lệch.