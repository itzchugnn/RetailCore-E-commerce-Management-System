# 09. DATA REQUIREMENTS – ĐẶC TẢ YÊU CẦU DỮ LIỆU

* **Milestone:** M1 – Business Analysis (Deliverable đầu vào quan trọng cho M2)
* **Phiên bản tài liệu:** 1.0 – Baseline

Nếu `Functional Requirements` mô tả hệ thống cần làm gì, thì `Data Requirements` mô tả hệ thống phải lưu giữ những thông tin gì, cần bảo toàn lịch sử nào và cần truy vấn dữ liệu như thế nào. Đây là cơ sở trực tiếp để xây dựng ERD/EER ở M2.

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

## 9.2. Historical Data Requirements (Yêu cầu bảo toàn dữ liệu lịch sử)

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

---

## 9.3. Data Retention (Chính sách lưu giữ dữ liệu)

| Nhóm dữ liệu | Chính sách dự kiến |
| :--- | :--- |
| **Tài khoản** | Vô hiệu hóa thay vì xóa vật lý khi đã có giao dịch |
| **Sản phẩm** | Lưu giữ nếu đã phát sinh giao dịch (chỉ chuyển trạng thái ngừng kinh doanh) |
| **Đơn hàng** | Không xóa vật lý tùy tiện |
| **Thanh toán** | Bảo toàn lịch sử giao dịch bất biến |
| **Kho hàng** | Bảo toàn toàn bộ lịch sử biến động tồn kho |
| **Audit Logs** | Có chính sách lưu giữ và dọn dẹp/lưu trữ (Archive) riêng |
| **Giỏ hàng cũ** | Có thể xóa hoặc lưu trữ định kỳ theo chính sách dọn dẹp |

---

## 9.4. Reporting Requirements (Yêu cầu truy vấn báo cáo)

Hệ thống cơ sở dữ liệu cần được thiết kế để hỗ trợ hiệu quả 10 nhóm báo cáo sau:

| ID | Báo cáo | Tiêu chí phân tích / Lọc |
| :--- | :--- | :--- |
| **DR-01** | Doanh thu theo thời gian | Ngày, tuần, tháng, quý |
| **DR-02** | Doanh thu theo sản phẩm | SKU, sản phẩm, danh mục |
| **DR-03** | Sản phẩm bán chạy | Số lượng bán |
| **DR-04** | Tình hình tồn kho | Kho, SKU, tồn khả dụng |
| **DR-05** | Hàng sắp hết | Ngưỡng tồn tối thiểu (Reorder Level / Low Stock Threshold) |
| **DR-06** | Hiệu quả voucher | Lượt sử dụng, tổng mức giảm giá, doanh thu mang lại |
| **DR-07** | Đơn hàng theo trạng thái | Trạng thái, khoảng thời gian |
| **DR-08** | Khách hàng mua nhiều | Số đơn hoàn tất, tổng giá trị mua |
| **DR-09** | Tình hình hoàn trả | Số lượng trả, giá trị hoàn tiền, tỷ lệ hoàn theo sản phẩm |
| **DR-10** | Hiệu quả nhà cung cấp | Số lượng nhập, tổng giá trị nhập, tiến độ giao |

---

## 9.5. Data Volume Assumptions (Giả định quy mô dữ liệu mục tiêu)

Để có cơ sở thiết kế kiểu dữ liệu (`INT` vs `BIGINT`), phân vùng lưu trữ và benchmark hiệu năng Index/Query ở M4, dự án xác định quy mô dữ liệu mục tiêu giả định như sau:

| Đối tượng dữ liệu | Quy mô bản ghi mục tiêu |
| :--- | :--- |
| **Users** | $5.000$ |
| **Customers** | $10.000$ |
| **Products** | $3.000$ |
| **Product Variants (SKUs)** | $15.000$ |
| **Warehouses** | $5$ |
| **Orders** | $100.000$ |
| **Order Details** | $500.000$ |
| **Stock Movements** | $1.000.000$ |
| **Payments** | $150.000$ |
| **Audit Logs** | $1.000.000$ |