# M1.11 - Business Decision Register (Bảng Chốt Quyết định Nghiệp vụ & Tác động CSDL M2)

## 1. Tổng quan (Executive Summary)
Tài liệu này ghi nhận **7 quyết định nghiệp vụ cốt lõi (Core Business Decisions - BD-01 đến BD-07)** được chốt sau giai đoạn phân tích và review thiết kế. Các quyết định này đóng vai trò kim chỉ nam và là ràng buộc tiên quyết cho việc xây dựng Mô hình Dữ liệu Quan niệm (ERD/EER) tại **Milestone M2**.

---

## 2. Bảng Tổng hợp 7 Quyết định Nghiệp vụ (Decisions Matrix)

| Mã | Bài toán / Quyết định | Phương án chốt | Tác động trực tiếp lên Mô hình CSDL (M2) |
| :---: | :--- | :--- | :--- |
| **BD-01** | **Quan hệ Sản phẩm – Danh mục** | **Hỗ trợ Đa danh mục ($N-N$)** kèm cờ `IsPrimary` | Bảng liên kết `Product_Category(product_id, category_id, is_primary)`. Hỗ trợ Breadcrumb và Báo cáo danh mục chính. |
| **BD-02** | **Chính sách Đặt hàng Khách vãng lai (Guest Checkout)** | **Bắt buộc đăng nhập tài khoản** | `Orders.customer_id` bắt buộc `NOT NULL`, liên kết chặt chẽ với `Users(customer_id)`. Quản lý chuẩn `Voucher_Usages` và Đổi trả. |
| **BD-03** | **Xử lý Đơn hàng từ Nhiều kho (Split Shipment)** | **Hỗ trợ tách kiện / giao nhiều kho** | Tách rời `Order` khỏi `WarehouseID`. `WarehouseID` gắn vào `Stock_Reservation` và `Shipment(shipment_id, order_id, warehouse_id)`. |
| **BD-04** | **Cơ chế Giữ tồn kho khi Đặt hàng (Inventory Reservation)** | **Giữ hàng ngay khi tạo đơn**, có thời hạn (`expires_at` = 15 phút) | Bảng `Inventory_Reservation` với trạng thái `ACTIVE / EXPIRED / FULFILLED / RELEASED`. Ngăn chặn bán âm kho (Overselling). |
| **BD-05** | **Chính sách Kết hợp Khuyến mãi (Promotion Stacking)** | **Cho phép kết hợp có kiểm soát** theo tầng: 1 Product Promo + 1 Order Voucher (`is_stackable = 1`) | Bảng `Order_Promotions` và `Order_Item.discount_amount`. Phân định rõ chiết khấu dòng sản phẩm và chiết khấu toàn đơn. |
| **BD-06** | **Mô hình Thuộc tính Biến thể Sản phẩm** | **Mô hình Thuộc tính động có kiểm soát** (Attribute – AttributeValue) | `Attribute`, `Attribute_Value`, `Product_Variant_Attribute_Value`. Linh hoạt cho cả Thời trang (Size/Màu) và Gia dụng (Kích thước/Chất liệu). |
| **BD-07** | **Quy trình Khấu trừ Tồn kho Vật lý** | **Khấu trừ 2 bước:** Giữ kho khi tạo đơn $\rightarrow$ Trừ kho vật lý khi xuất kho bàn giao | Công thức: $\text{Available} = \text{OnHand} - \text{Reserved}$. Ghi nhật ký `Inventory_Transaction` loại `SHIPMENT_OUT` và `DELIVERY_RETURN_IN`. |

---

## 3. Phân tích Chi tiết Tác động Kỹ thuật lên Milestone M2

### BD-01: Sản phẩm – Danh mục ($N-N$ với `IsPrimary`)
* **Bối cảnh:** Bán lẻ thời trang và gia dụng có các nhóm danh mục chức năng (Áo nam, Quần jean, Bàn làm việc) và danh mục chiến dịch/bộ sưu tập (Thu Đông 2026, Hàng mới về, Sale sinh nhật).
* **Quyết định:** Một sản phẩm có thể thuộc nhiều danh mục. Trong đó, có đúng 1 danh mục mang cờ `is_primary = TRUE`.
* **Thiết kế M2:**
  ```sql
  -- Entity: ProductCategory
  (product_id FK, category_id FK, is_primary BIT DEFAULT 0, PRIMARY KEY (product_id, category_id))
  ```
* **Giá trị đem lại:**
  * Giải quyết đường dẫn điều hướng (Breadcrumb) và URL thân thiện dựa trên danh mục chính.
  * Báo cáo doanh thu (DR-02) không bị tính trùng lặp doanh số của sản phẩm khi phân tích theo danh mục.

---

### BD-02: Bắt buộc Đăng nhập khi Mua hàng (No Guest Checkout)
* **Bối cảnh:** Cho phép khách vãng lai (Guest) mua hàng tiện lợi nhưng gây rủi ro lớn trong kiểm soát gian lận Voucher, thất lạc lịch sử đổi trả và trùng lặp dữ liệu khách hàng.
* **Quyết định:** Bắt buộc khách hàng tạo tài khoản / đăng nhập trước khi tiến hành Checkout (Guest chỉ được xem và thêm giỏ hàng).
* **Thiết kế M2:**
  * Bảng `Orders`: Cột `customer_id BIGINT NOT NULL FOREIGN KEY REFERENCES Users(user_id)`.
  * Bảng `Voucher_Usages`: Ràng buộc `UNIQUE(voucher_id, customer_id, order_id)` giúp kiểm soát triệt để giới hạn số lần sử dụng trên mỗi khách hàng.
  * Bảng `Return_Requests`: Gắn liền với định danh khách hàng đã mua đơn, bảo đảm quy định đổi trả trong vòng 7 ngày.

---

### BD-03: Chia Đơn Giao Nhiều Kho (Split Shipment)
* **Bối cảnh:** Doanh nghiệp sở hữu nhiều kho vật lý phân tán. Đơn hàng của khách có thể chứa cả áo (kho A) và ghế (kho B).
* **Quyết định:** Không gán cứng `warehouse_id` vào cấp `Orders`. Một đơn hàng có thể phân rã thành một hoặc nhiều `Shipments` xuất phát từ các kho khác nhau.
* **Thiết kế M2:**
  * `Orders` chỉ quản lý thông tin thương mại, tổng tiền và trạng thái thanh toán.
  * Bảng `Shipments`: Quản lý việc giao vận: `shipment_id`, `order_id (FK)`, `warehouse_id (FK)`, `shipping_carrier`, `tracking_code`, `status`, `shipping_fee`.
  * Bảng `Shipment_Items`: `(shipment_id FK, order_item_id FK, quantity INT)`.
  * Bảng `Stock_Reservations`: Gắn với `warehouse_id` và `order_item_id`.

---

### BD-04: Giữ Tồn Kho Ngay Khi Đặt Hàng (Time-bound Inventory Reservation)
* **Bối cảnh:** Trong các đợt Flash Sale hoặc sản phẩm hot, nhiều khách hàng cùng bấm mua đồng thời. Nếu chờ thanh toán thành công hoặc nhân viên duyệt mới giữ hàng sẽ gây ra Overselling (bán âm kho).
* **Quyết định:** Giữ tồn kho ngay khi tạo đơn với thời hạn tạm giữ là **15 phút** (`expires_at`).
* **Thiết kế M2:**
  * Bảng `Inventory_Reservations`: `(reservation_id, order_id, order_item_id, variant_id, warehouse_id, reserved_quantity, expires_at, status)`.
  * Trạng thái Reservation: `ACTIVE` $\rightarrow$ `FULFILLED` (khi thanh toán thành công) hoặc `RELEASED` (khi khách hủy / quá hạn 15 phút).
  * Công thức tính tồn kho khả dụng tức thời:
    $$\text{Available\_Quantity} = \text{Quantity\_On\_Hand} - \text{Reserved\_Quantity}$$

---

### BD-05: Phân Tầng Khuyến Mãi (Stackable Promotions)
* **Bối cảnh:** Doanh nghiệp muốn hỗ trợ cả giảm giá trực tiếp trên sản phẩm (Flash Sale / Khuyến mãi ngành hàng) và Voucher giảm giá giỏ hàng do khách tự nhập.
* **Quyết định:** Cho phép áp dụng tối đa 1 Khuyến mãi dòng sản phẩm (Product/Item Promotion) kết hợp với tối đa 1 Voucher cấp đơn hàng (Order Voucher) nếu Voucher đó có cờ `is_stackable = TRUE`.
* **Thiết kế M2:**
  * Bảng `Promotions`: Khuyến mãi chung (theo sản phẩm, danh mục, flash sale) $\rightarrow$ Giảm trực tiếp vào `Order_Items.discount_amount`.
  * Bảng `Vouchers`: Mã giảm giá toàn đơn do khách nhập.
  * Bảng `Order_Promotions`: `(order_id, promotion_id NULL, voucher_id NULL, discount_amount, layer_type)`.
  * Lưu trữ tách biệt: `Order.subtotal`, `Order.item_discount_amount`, `Order.voucher_discount_amount`, `Order.shipping_fee`, `Order.total_amount`.

---

### BD-06: Thuộc Tính Biến Thể Động (Attribute – AttributeValue)
* **Bối cảnh:** Thời trang cần Size, Màu sắc, Chất liệu vải; Đồ gia dụng cần Chiều dài, Chiều rộng, Công suất, Chất liệu gỗ/kim loại. Cố định cột sẽ gây `NULL` và phá vỡ tính mở rộng.
* **Quyết định:** Áp dụng mô hình thuộc tính động có kiểm soát (Normalized Dynamic Attributes).
* **Thiết kế M2:**
  * `Attributes`: `(attribute_id, name, code, category_id NULL)` -> Ví dụ: Color, Size, Material, Dimension.
  * `Attribute_Values`: `(value_id, attribute_id, value)` -> Ví dụ: Red, XL, Cotton, 120x60cm.
  * `Product_Variant_Attribute_Values`: `(variant_id, value_id)` -> Bảng liên kết xác định cấu hình cụ thể của một SKU.

---

### BD-07: Chính Sách Khấu Trừ Tồn Kho Vật Lý (Two-phase Stock Movement)
* **Bối cảnh:** Hàng hóa đã đặt nhưng chưa xuất kho thì trên thực tế vẫn đang nằm trên giá kệ của kho. Nếu trừ ngay tồn vật lý sẽ làm lệch số liệu kiểm kê thực tế.
* **Quyết định:** Khấu trừ 2 bước:
  1. **Bước 1 (Khi tạo đơn):** Tăng `Reserved_Quantity`, giữ nguyên `Quantity_On_Hand`.
  2. **Bước 2 (Khi đóng gói xuất kho bàn giao vận chuyển):** Giảm `Reserved_Quantity`, giảm `Quantity_On_Hand`. Ghi bản ghi `Inventory_Transactions` với loại `SHIPMENT_OUT`.
  3. **Xử lý Hàng hoàn (Delivery Failed / Return):** Khi hàng quay về kho và được kiểm tra tiếp nhận, ghi `Inventory_Transactions` với loại `DELIVERY_RETURN_IN`, tăng lại `Quantity_On_Hand`.
* **Thiết kế M2:**
  * Bảng `Inventory`: `(warehouse_id, variant_id, quantity_on_hand, reserved_quantity)`.
  * Bảng `Inventory_Transactions`: `(transaction_id, warehouse_id, variant_id, transaction_type, quantity_changed, balance_after, reference_type, reference_id, created_at)`.
  * Đảm bảo tính toàn vẹn: `quantity_on_hand >= 0` và `reserved_quantity >= 0`.

