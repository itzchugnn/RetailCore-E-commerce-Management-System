# M1.11 - Business Decisions Register (Chính thức hóa BD-01 đến BD-07)

* **Milestone:** M1 – Business Analysis
* **Phiên bản:** 1.1 – Approved Business Decisions
* **Database Target:** Microsoft SQL Server
* **Trạng thái:** Đã phê duyệt chính thức (Approved)

---

## 1. Bảng Tổng hợp 7 Quyết định Nghiệp vụ Chính thức

| ID | Quyết định | Kết luận chính thức | Tác động trực tiếp lên Mô hình CSDL (M2–M4) |
| :---: | :--- | :--- | :--- |
| **BD-01** | **Sản phẩm – Danh mục** | Một sản phẩm có thể thuộc nhiều danh mục ($N-N$). Có một danh mục chính. | Thực thể liên kết `ProductCategory(ProductID, CategoryID, IsPrimary)`. Hỗ trợ Breadcrumb và báo cáo doanh thu không trùng lặp. |
| **BD-02** | **Guest Checkout** | Bắt buộc đăng nhập trước khi đặt hàng. Không hỗ trợ Guest Checkout. | `Orders.CustomerID` bắt buộc `NOT NULL`. Quản lý chuẩn `VoucherUsages`, chính sách đổi trả (7 ngày) và hồ sơ khách hàng. |
| **BD-03** | **Split Shipment** | Một đơn hàng có thể được xử lý và giao từ nhiều kho. | Tách rời `Order` khỏi `WarehouseID`. `OrderLine` được chia thành các `OrderAllocation`, xuất kho qua nhiều `Shipment`. |
| **BD-04** | **Reservation** | Giữ hàng ngay khi tạo đơn, có thời hạn `ExpiresAt` (15 phút). | Thực thể `StockReservation` (hoặc gắn với `OrderAllocation`) với thời hạn tạm giữ, ngăn chặn bán âm kho (Overselling) trong Flash Sale. |
| **BD-05** | **Promotion Stacking** | Cho phép tối đa một khuyến mãi cấp sản phẩm và một voucher cấp đơn hàng, nếu quy tắc cho phép cộng dồn (`IsStackable = 1`). | Tách bạch 2 tầng khuyến mãi; cấm áp dụng 2 chương trình cùng tầng; lưu vết chi tiết tính toán trong `OrderPromotions`. |
| **BD-06** | **Variant Attributes** | Sử dụng mô hình thuộc tính linh hoạt theo từng sản phẩm (Dynamic Attributes). | Bộ ba thực thể chuẩn hóa `Attribute`, `AttributeValue`, `ProductAttribute`, `VariantAttributeValue`. Không hardcode cột Size/Color. |
| **BD-07** | **Inventory on Delivery & Failed Delivery** | Khi xuất kho, giảm đồng thời tồn vật lý (`OnHandQty`) và lượng giữ (`ReservedQty`). Hàng giao thất bại chỉ được nhập lại kho khi thực sự quay về. | Phân định rạch ròi 4 trạng thái tồn kho và tuân thủ vòng đời 5 bước: Tạo đơn $\rightarrow$ Xuất kho $\rightarrow$ Đang giao $\rightarrow$ Tiếp nhận hoàn kho $\rightarrow$ Giao lại/Hủy. |

---

## 2. Chi tiết 2 Quyết định Trọng tâm: BD-05 & BD-07

### 2.1. BD-05: Chính sách Khuyến mãi Phân tầng & Điều kiện Cộng dồn (Promotion Stacking)

#### Hai tầng khuyến mãi độc lập:
1. **Tầng 1 – Product Promotion (Khuyến mãi cấp sản phẩm / Dòng hàng):**
   * Áp dụng trực tiếp trên từng dòng sản phẩm (`OrderLine`).
   * Ví dụ: Flash Sale giảm 15% cho Áo thun Polo, Giảm 50.000đ cho Đèn bàn.
   * **Giới hạn:** Tối đa **1 chương trình** giảm giá trực tiếp trên mỗi dòng sản phẩm.
2. **Tầng 2 – Order Voucher (Voucher cấp đơn hàng):**
   * Áp dụng trên toàn bộ giá trị đơn hàng sau khi đã trừ chiết khấu dòng sản phẩm.
   * Ví dụ: Nhập mã `SUMMER2026` giảm 10% tối đa 100.000đ cho đơn từ 500.000đ.
   * **Giới hạn:** Tối đa **1 voucher** cấp đơn hàng trên mỗi đơn.

#### Quy tắc cộng dồn (Stacking Rules):
* Hai tầng **chỉ được phép kết hợp** khi và chỉ khi:
  1. Voucher cấp đơn hàng có cờ `IsStackable = 1` (hoặc thuộc tính `CanCombineWithProductDiscount = TRUE`).
  2. Chương trình khuyến mãi cấp sản phẩm không có điều khoản cấm kết hợp mã giảm giá.
* **Quy tắc chặn tuyệt đối:** Không được áp dụng nhiều chương trình cùng tầng vượt quá giới hạn (ví dụ: không được nhập 2 voucher đơn hàng, không được áp 2 chương trình flash sale trên 1 dòng sản phẩm), kể cả khi các chương trình riêng lẻ đều thỏa điều kiện áp dụng.

#### Lưu vết đối soát tài chính (`OrderPromotions` & `OrderLine`):
* `OrderLine.UnitPrice`: Giá niêm yết snapshot tại thời điểm đặt.
* `OrderLine.DiscountAmount`: Số tiền giảm trực tiếp từ Tầng 1 (Product Promotion).
* `Order.VoucherDiscountAmount`: Số tiền giảm từ Tầng 2 (Order Voucher).
* Bảng `OrderPromotions`: Lưu toàn bộ danh sách các chương trình đã áp dụng, mức giảm, thứ tự áp dụng (Thứ tự 1: Tính giảm dòng $\rightarrow$ Thứ tự 2: Tính giảm toàn đơn) và giá trị trước/sau giảm để phục vụ đối soát kế toán bất biến.

---

### 2.2. BD-07: Chính sách Quản lý Tồn kho khi Vận chuyển và Giao thất bại

#### Quy ước bản chất về các trạng thái tồn kho:
* **Hàng đang giữ tại kho (`ReservedQty`):** Chỉ đại diện cho lượng hàng đang nằm trên giá kệ vật lý của một kho cụ thể, đã có cam kết cho đơn hàng nhưng **chưa xuất kho**.
* **Hàng đã xuất kho:** Khi hàng được đóng gói và bàn giao cho đơn vị vận chuyển, reservation tại kho được tiêu thụ và kết thúc.
* **Hàng đang trên đường giao (In-Transit):** Hàng đã rời kho vật lý thì **không còn thuộc `OnHandQty` của kho xuất**, và cũng **không được duy trì trong `ReservedQty`**. Cam kết cung ứng lúc này được theo dõi thông qua thực thể `Shipment`, `ShipmentLine` và trạng thái vận đơn.
* **Hàng giao thất bại:** Việc giao hàng thất bại **không đồng nghĩa hàng đã quay về kho**. Hàng chỉ được cộng lại vào tồn kho khi nhân viên kho thực sự nhận hàng, kiểm tra và ký xác nhận phiếu tiếp nhận.

#### Vòng đời 5 bước của tồn kho:
1. **Bước 1: Tạo đơn hàng thành công (Place Order / Hold)**
   * `OnHandQty` giữ nguyên; `ReservedQty` tăng theo số lượng đặt mua.
   * Công thức tồn khả dụng: $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$.
2. **Bước 2: Kho đóng gói & Bàn giao vận chuyển (Shipment Handover – `SHIPMENT_OUT`)**
   * Hàng thực sự rời khỏi kho vật lý.
   * Giảm đồng thời cả `OnHandQty` và `ReservedQty` theo số lượng thực tế xuất.
   * Ghi một bản ghi vào `StockMovement` loại `SHIPMENT_OUT`.
3. **Bước 3: Giao hàng thất bại (Delivery Failed)**
   * Đơn vị vận chuyển báo giao thất bại (khách không nghe máy, sai địa chỉ).
   * Hệ thống cập nhật trạng thái `Shipment` thành `DELIVERY_FAILED`.
   * **Tuyệt đối chưa tăng tồn kho**, vì hàng vẫn đang nằm trên xe hoặc tại bưu cục của đơn vị vận chuyển.
4. **Bước 4: Kho tiếp nhận hàng hoàn về (Delivery Return Receipt – `DELIVERY_RETURN_IN`)**
   * Hàng thực tế được xe chở quay trở lại kho xuất (hoặc kho chỉ định).
   * Nhân viên kho kiểm tra tình trạng hàng hóa, ký biên bản `DeliveryReturnReceipt`.
   * Hệ thống ghi `StockMovement` loại `DELIVERY_RETURN_IN` và tăng `OnHandQty` theo số lượng thực nhận lại.
5. **Bước 5: Giao lại hoặc Hủy đơn (Re-delivery or Order Cancellation)**
   * *Nếu giao lại:* Hệ thống tiến hành phân bổ kho (`OrderAllocation`) và giữ hàng lại (`StockReservation`) theo quy tắc tồn kho hiện hành trước khi xuất kho lần 2.
   * *Nếu hủy đơn:* Kích hoạt quy trình hoàn tiền (`Refund`), giải phóng các nghĩa vụ liên quan và đóng đơn hàng.
---

## 3. Chính thức hóa Quyết định Thiết kế Subject Area 01: Product Catalog (PC-Q01 đến PC-Q06)

| ID | Vấn đề thiết kế | Quyết định chốt chính thức | Tác động CSDL & Nghiệp vụ |
| :---: | :--- | :--- | :--- |
| **PC-Q01** | Bắt buộc Danh mục chính khi Published? | **Bắt buộc đúng một danh mục chính (`IsPrimary = 1`) khi Published.** | Cài đặt `UNIQUE FILTERED INDEX (ProductID) WHERE IsPrimary = 1`. Cần thiết để render Breadcrumb, SEO và báo cáo doanh thu không trùng lặp (DR-02). |
| **PC-Q02** | Cấu trúc phân cấp Danh mục? | **Hỗ trợ nhiều cấp cha – con.** | Khóa ngoại đệ quy `Category.ParentCategoryID REFERENCES Category(CategoryID)` không tạo chu trình. |
| **PC-Q03** | Sản phẩm không có biến thể thuộc tính (Simple Product)? | **Tạo 1 SKU mặc định, tổ hợp thuộc tính rỗng.** | Mọi sản phẩm đủ điều kiện kinh doanh đều có đơn vị bán và quản lý tồn kho ở cấp SKU. CSDL không bị Polymorphic Foreign Key. |
| **PC-Q04** | Vai trò của thuộc tính khi gán cho sản phẩm? | **Một thuộc tính chỉ có duy nhất 1 vai trò: `DESCRIPTIVE` hoặc `VARIANT_DEFINING`.** | Cột `ProductAttribute.AttributeRole` phân định rạch ròi: mô tả chung (không sinh SKU) hoặc định nghĩa biến thể (sinh SKU). |
| **PC-Q05** | Dùng chung định nghĩa `Attribute`? | **Dùng chung từ điển `Attribute` toàn cục.** | Chuẩn hóa danh mục thuộc tính toàn hệ thống, phục vụ bộ lọc Faceted Filter trên Website. |
| **PC-Q06** | Bảo toàn SKU khi đổi thuộc tính? | **Bảo toàn tuyệt đối SKU và dữ liệu lịch sử.** | Không sửa đổi ngữ nghĩa hoặc xóa vật lý SKU đã phát sinh giao dịch. Áp dụng `IsActive = 0` (Soft Delete). |
