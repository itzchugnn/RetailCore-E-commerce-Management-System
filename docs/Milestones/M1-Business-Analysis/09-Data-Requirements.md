# 09. DATA REQUIREMENTS – ĐẶC TẢ YÊU CẦU DỮ LIỆU (PHIÊN BẢN v1.1)

* **Milestone:** M1 – Business Analysis (Deliverable đầu vào quan trọng cho M2)
* **Phiên bản tài liệu:** 1.1 – Approved Business Decisions (BD-01 đến BD-07)
* **Database target:** Microsoft SQL Server

Nếu `Functional Requirements` mô tả hệ thống cần làm gì, thì `Data Requirements` mô tả hệ thống phải lưu giữ những thông tin gì, cần bảo toàn lịch sử nào và cần ràng buộc dữ liệu ra sao. Đây là cơ sở trực tiếp để xây dựng ERD/EER ở M2 và Relational Schema ở M3.

---

## 9.1. Data Subject Areas (Các miền chủ đề dữ liệu)

| ID | Nhóm dữ liệu | Nội dung cần quản lý |
| :--- | :--- | :--- |
| **DA-01** | Identity | Tài khoản, vai trò, quyền |
| **DA-02** | Product | Sản phẩm, SKU, thuộc tính, hình ảnh |
| **DA-03** | Warehouse | Kho hàng, vị trí kho |
| **DA-04** | Inventory | Tồn kho, hàng giữ, biến động tồn |
| **DA-05** | Procurement | Nhà cung cấp, đơn mua, phiếu nhập |
| **DA-06** | Customer | Hồ sơ và địa chỉ khách hàng |
| **DA-07** | Shopping | Giỏ hàng, chi tiết giỏ |
| **DA-08** | Sales | Đơn hàng, chi tiết đơn, lịch sử trạng thái |
| **DA-09** | Promotion | Chương trình giảm giá, voucher |
| **DA-10** | Payment | Giao dịch thanh toán, hoàn tiền |
| **DA-11** | Shipping | Vận đơn, lịch sử vận chuyển |
| **DA-12** | Return | Yêu cầu đổi trả, chi tiết hàng hoàn |
| **DA-13** | Audit | Nhật ký hoạt động |
| **DA-14** | Reporting | Dữ liệu phục vụ thống kê |

---

## 9.2. Yêu cầu Quản lý Dữ liệu Chi tiết theo Nghiệp vụ (Data Requirements)

### 9.2.1. Product & Category
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-11** | Product–Category Assignment | Sản phẩm, danh mục được gán và cờ `IsPrimary`. |
| **DR-12** | Product Attribute Definition | Các thuộc tính biến thể được định nghĩa cho từng sản phẩm. |
| **DR-13** | Attribute Value | Các giá trị hợp lệ của từng thuộc tính, ví dụ: Size = M, Color = Black, Material = Denim. |
| **DR-14** | Variant Attribute Combination | Tổ hợp giá trị thuộc tính xác định từng SKU duy nhất trong sản phẩm. |

### 9.2.2. Order & Warehouse Fulfillment
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-15** | Order Warehouse Allocation | Dòng đơn hàng, kho được phân bổ, SKU và số lượng được phân bổ từ kho đó. |
| **DR-16** | Stock Reservation | SKU, kho, số lượng giữ, thời điểm tạo, thời điểm hết hạn (`ExpiresAt`) và trạng thái reservation. |
| **DR-17** | Shipment | Đơn hàng, kho xuất, đơn vị vận chuyển, trạng thái và thông tin giao hàng. |
| **DR-18** | Shipment Line | Dòng đơn hàng, SKU, số lượng thực tế xuất và số lượng thuộc từng lần giao. |
| **DR-19** | Inventory Movement | Loại biến động (`SHIPMENT_OUT`, `DELIVERY_RETURN_IN`, v.v.), số lượng, kho, SKU, thời điểm, chứng từ nguồn và người thực hiện. |
| **DR-20** | Delivery Return Receipt | Lần giao thất bại, hàng thực tế quay về, số lượng nhận lại, kho tiếp nhận và thời điểm xác nhận. |

### 9.2.3. Promotion & Voucher
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-21** | Applied Order Promotions | Các chương trình khuyến mãi thực tế được áp dụng trên đơn hàng. |
| **DR-22** | Promotion Application Scope | Phạm vi áp dụng: dòng sản phẩm (Product/Item) hoặc toàn đơn hàng (Order Voucher). |
| **DR-23** | Promotion Calculation Snapshot | Giá trị trước giảm, giá trị giảm, giá trị sau giảm và thứ tự áp dụng. |
| **DR-24** | Promotion Stacking Rules | Quy tắc cộng dồn (`IsStackable`), tầng khuyến mãi, độ ưu tiên và giới hạn số chương trình được kết hợp. |

---

## 9.3. Historical Data Requirements (Yêu cầu bảo toàn dữ liệu lịch sử)

> **Nguyên tắc thiết kế:** Dữ liệu lịch sử phải có khả năng truy vết độc lập theo thời gian. Không phải cứ thêm một cột `UpdatedAt` ghi đè trên bảng chính là giải quyết được bài toán lịch sử.

| ID | Dữ liệu lịch sử | Yêu cầu bảo toàn |
| :--- | :--- | :--- |
| **HDR-01** | Giá sản phẩm | Lưu lịch sử thay đổi giá khi cần truy vấn giá theo thời điểm |
| **HDR-02** | Giá bán trong đơn | Không thay đổi khi giá sản phẩm hiện tại thay đổi |
| **HDR-03** | Thông tin sản phẩm trong đơn | Bảo toàn snapshot tại thời điểm giao dịch |
| **HDR-04** | Địa chỉ giao hàng | Bảo toàn thông tin tại thời điểm đặt hàng |
| **HDR-05** | Tồn kho | Lưu lịch sử nhập, xuất và điều chỉnh |
| **HDR-06** | Trạng thái đơn hàng | Lưu lịch sử chuyển trạng thái |
| **HDR-07** | Thanh toán | Lưu từng lần thử và kết quả giao dịch |
| **HDR-08** | Hoàn tiền | Lưu từng lần hoàn tiền và số tiền |
| **HDR-09** | Khuyến mãi | Lưu thông tin chương trình được áp dụng và mức giảm |
| **HDR-10** | Vận chuyển | Lưu lịch sử hành trình và trạng thái |
| **HDR-11** | Đổi trả | Lưu lịch sử xử lý yêu cầu |
| **HDR-12** | Audit | Lưu các thao tác quản trị quan trọng |
| **HDR-13** | Phân mục sản phẩm | Lịch sử gán sản phẩm vào danh mục, bao gồm danh mục chính và thời gian hiệu lực |
| **HDR-14** | Định nghĩa thuộc tính | Lịch sử thay đổi định nghĩa thuộc tính và giá trị thuộc tính của sản phẩm |
| **HDR-15** | Giữ hàng (Reservation) | Lịch sử giữ hàng, gia hạn, giải phóng và tiêu thụ reservation |
| **HDR-16** | Phân bổ kho cho đơn | Lịch sử phân bổ dòng đơn hàng cho các kho |
| **HDR-17** | Vận chuyển & Trả hàng | Lịch sử xuất kho, bàn giao vận chuyển, giao thất bại và nhập lại kho |
| **HDR-18** | Tính toán khuyến mãi | Lịch sử áp dụng khuyến mãi, bao gồm thứ tự áp dụng và kết quả tính giảm giá |

---

## 9.4. Các ràng buộc dữ liệu cần lưu ý khi thiết kế (Constraints Baseline cho M2–M4)

Đây là các ràng buộc cần được chuyển hóa thành khóa, ràng buộc duy nhất, CHECK, chỉ mục lọc hoặc logic giao dịch ở giai đoạn M3–M4.

| Đối tượng | Ràng buộc dự kiến | Tầng thực thi khuyến nghị |
| :--- | :--- | :--- |
| **ProductCategory** | Mỗi cặp ProductID–CategoryID chỉ xuất hiện một lần. | Database: `PRIMARY KEY (ProductID, CategoryID)` |
| **ProductCategory** | Tối đa một dòng `IsPrimary = 1` cho mỗi ProductID. | Database: `UNIQUE FILTERED INDEX` (WHERE IsPrimary = 1) |
| **ProductVariant** | SKU phải duy nhất toàn hệ thống. | Database: `UNIQUE CONSTRAINT (SKU)` |
| **VariantAttributeValue** | Không trùng một AttributeID trong cùng một SKU. | Database: `UNIQUE (VariantID, AttributeID)` |
| **ProductVariant** | Không có hai SKU trong cùng sản phẩm mang tổ hợp thuộc tính giống nhau. | Database Trigger / Stored Procedure validation |
| **StockReservation** | Số lượng reservation phải dương; không được giữ vượt quá lượng khả dụng. | Database `CHECK (ReservedQty > 0)` & Stored Procedure |
| **InventoryBalance** | `OnHandQty >= 0`, `ReservedQty >= 0`, `ReservedQty <= OnHandQty`. | Database: `CHECK CONSTRAINT` |
| **Shipment** | Một đơn hàng có thể có nhiều Shipment; mỗi Shipment thuộc một kho xuất cụ thể. | Database: `FOREIGN KEY (OrderID), FOREIGN KEY (WarehouseID)` |
| **ShipmentLine** | Số lượng xuất không được vượt quá số lượng còn được phân bổ cho dòng đơn hàng. | Stored Procedure / Application Transaction |
| **OrderPromotion** | Giới hạn số chương trình được áp dụng theo từng tầng (tối đa 1 Product Promo, 1 Order Voucher). | Database: Unique Filtered Index / Transaction validation |
| **StockMovement** | Mỗi biến động phải có loại giao dịch, chứng từ nguồn và thông tin truy vết bất biến. | Database: `CHECK (MovementType IN (...))` & No UPDATE/DELETE |

> **Lưu ý nguyên tắc thiết kế:** Không phải mọi ràng buộc đều nên triển khai bằng một `CHECK CONSTRAINT`. Những quy tắc cần kiểm tra nhiều dòng hoặc nhiều bảng, chẳng hạn tổng số lượng đã phân bổ, điều kiện cộng dồn khuyến mãi và kiểm soát tồn kho đồng thời, cần được thiết kế bằng transaction, stored procedure hoặc logic ứng dụng có kiểm soát.
