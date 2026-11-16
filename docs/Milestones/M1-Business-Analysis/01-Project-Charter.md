# 01. PROJECT CHARTER – RETAILCORE E-COMMERCE MANAGEMENT SYSTEM

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline
* **Phạm vi:** Medium-scale
* **Đầu ra tiếp theo:** M2 – Conceptual Design

> **Nguyên tắc trọng tâm trong M1:**
> Tài liệu này mô tả hệ thống cần làm gì và phải tuân thủ quy tắc nghiệp vụ nào, chưa quyết định hệ thống sẽ lưu dữ liệu bằng những bảng nào. Các quyết định về bảng, khóa, quan hệ và chuẩn hóa sẽ được thực hiện ở M2 và M3.

---

## 1.1. Thông tin dự án

| Thuộc tính | Nội dung |
| :--- | :--- |
| **Project Name** | RetailCore |
| **Project Type** | E-commerce Management System |
| **Domain** | Retail / E-commerce |
| **Project Scale** | Medium |
| **Primary Objective** | Xây dựng hệ thống thương mại điện tử và quản trị bán lẻ |
| **Database Platform** | Microsoft SQL Server |
| **Backend** | Java Spring Boot |
| **Frontend** | React + TypeScript |
| **Architecture** | Monolithic, Layered Architecture |
| **Target Users** | Customer, Sales Staff, Warehouse Staff, Marketing Staff, Admin |

---

## 1.2. Bối cảnh dự án

Các doanh nghiệp bán lẻ thường phải quản lý nhiều hoạt động liên quan chặt chẽ với nhau:
* Quản lý danh mục và thông tin sản phẩm.
* Quản lý biến thể sản phẩm và mã SKU.
* Quản lý nhiều kho hàng.
* Quản lý nhập hàng và tồn kho.
* Quản lý khách hàng và đơn hàng.
* Quản lý chương trình khuyến mãi.
* Quản lý thanh toán, giao hàng và đổi trả.
* Theo dõi doanh thu và hiệu quả kinh doanh.

> Nếu các hoạt động này được quản lý bằng những hệ thống rời rạc, doanh nghiệp có thể gặp tình trạng dữ liệu không nhất quán, khó kiểm soát tồn kho và thiếu khả năng truy vết giao dịch. **RetailCore** được xây dựng nhằm giải quyết bài toán quản lý tập trung những hoạt động trên.

---

## 1.3. Mục tiêu dự án (Project Objectives)

| ID | Mục tiêu |
| :--- | :--- |
| **OBJ-01** | Xây dựng hệ thống quản lý sản phẩm và biến thể sản phẩm. |
| **OBJ-02** | Quản lý tồn kho tập trung tại nhiều kho hàng. |
| **OBJ-03** | Quản lý toàn bộ quy trình bán hàng từ giỏ hàng đến hoàn tất đơn hàng. |
| **OBJ-04** | Quản lý chương trình khuyến mãi và voucher. |
| **OBJ-05** | Quản lý thanh toán, giao hàng và đổi trả. |
| **OBJ-06** | Đảm bảo tính toàn vẹn và nhất quán dữ liệu. |
| **OBJ-07** | Xây dựng nền tảng Database có khả năng mở rộng và tối ưu hiệu năng. |
| **OBJ-08** | Cung cấp báo cáo phục vụ hoạt động quản trị kinh doanh. |

---

## 1.4. Phạm vi hệ thống (Scope)

### In Scope – Các chức năng được triển khai
1. Authentication và Authorization.
2. Quản lý người dùng và phân quyền.
3. Quản lý danh mục, thương hiệu và sản phẩm.
4. Quản lý SKU và thuộc tính biến thể.
5. Quản lý kho và tồn kho.
6. Quản lý nhà cung cấp và nhập hàng.
7. Quản lý khách hàng và địa chỉ.
8. Quản lý giỏ hàng.
9. Quản lý đơn hàng.
10. Quản lý khuyến mãi và voucher.
11. Quản lý thanh toán và hoàn tiền.
12. Quản lý vận chuyển.
13. Quản lý yêu cầu đổi trả.
14. Quản lý báo cáo và nhật ký hoạt động.

### Out of Scope – Không triển khai trong phiên bản đầu
* Marketplace nhiều nhà bán độc lập.
* Ứng dụng mobile native.
* Đồng bộ đơn hàng với Shopee, TikTok Shop.
* Tích hợp hệ thống ERP bên ngoài.
* Hệ thống kế toán chuyên nghiệp.
* AI recommendation engine.
* Chương trình loyalty với nhiều cấp độ phức tạp.
* Microservices và kiến trúc phân tán.

---

## 1.5. Giả định và giới hạn (Assumptions & Constraints)

| ID | Giả định |
| :--- | :--- |
| **ASM-01** | Hệ thống phục vụ một doanh nghiệp bán lẻ. |
| **ASM-02** | Doanh nghiệp có thể vận hành nhiều kho. |
| **ASM-03** | Khách hàng có thể mua hàng bằng tài khoản. |
| **ASM-04** | Tiền tệ mặc định là VND. |
| **ASM-05** | Mỗi sản phẩm có thể có nhiều SKU. |
| **ASM-06** | Website là kênh bán hàng duy nhất trong phiên bản đầu. |
| **ASM-07** | Thanh toán trực tuyến có thể được mô phỏng hoặc sử dụng sandbox. |
| **ASM-08** | Nhân viên kho và nhân viên bán hàng sử dụng giao diện quản trị. |

---

## 1.6. Tiêu chí thành công (Success Criteria)

* Toàn bộ chức năng trong phạm vi được triển khai.
* Không xảy ra tình trạng tồn kho âm do đặt hàng đồng thời.
* Không mất dữ liệu giao dịch khi xảy ra lỗi giữa chừng.
* Đơn hàng bảo toàn được giá và thông tin giao dịch tại thời điểm mua.
* Có thể truy vết lịch sử nhập, xuất và biến động tồn kho.
* Có bộ kiểm thử nghiệp vụ và kiểm thử đồng thời.
* Có tài liệu thiết kế Database và API đầy đủ.