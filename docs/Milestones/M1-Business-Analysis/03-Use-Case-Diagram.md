# 03. USE CASE DIAGRAM – SƠ ĐỒ CA SỬ DỤNG TỔNG THỂ

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

---

## 3.1. Phân nhóm chức năng theo Actor

| Actor | Use Cases tương ứng |
| :--- | :--- |
| **Guest** | Xem sản phẩm, tìm kiếm, lọc sản phẩm, đăng ký, đăng nhập |
| **Customer** | Quản lý hồ sơ, giỏ hàng, đặt hàng, thanh toán, theo dõi đơn, yêu cầu đổi trả |
| **Sales Staff** | Tra cứu khách hàng, quản lý đơn hàng, xác nhận đơn, xử lý hủy đơn |
| **Warehouse Staff** | Quản lý kho, nhập kho, chuyển kho, xuất kho, kiểm kê |
| **Marketing Staff** | Quản lý chương trình khuyến mãi, voucher |
| **Admin** | Quản lý tài khoản, vai trò, phân quyền, cấu hình hệ thống, báo cáo |
| **Supplier** | Cung cấp thông tin hàng hóa và giao hàng theo đơn mua |
| **Shipping Provider** | Nhận vận đơn, cập nhật hành trình |
| **Payment Provider** | Xử lý thanh toán và hoàn tiền |

---

## 3.2. Use Case Diagram Tổng thể (Mermaid)

Sơ đồ dưới đây mô tả ranh giới hệ thống **RetailCore System** cùng mối liên kết giữa các tác nhân nội bộ, khách hàng và các tác nhân bên ngoài:

~~~mermaid
flowchart LR
    %% Primary Actors
    Guest(["👤 Guest"])
    Customer(["🛍️ Customer"])
    Sales(["💼 Sales Staff"])
    Warehouse(["📦 Warehouse Staff"])
    Marketing(["🎯 Marketing Staff"])
    Admin(["🛡️ Administrator"])

    %% External / Supporting Actors
    Supplier(["🏭 Supplier"])
    Shipper(["🚚 Shipping Provider"])
    PaymentGate(["💳 Payment Provider"])

    subgraph RetailCore["RetailCore System"]
        UC_Browse("Browse Products")
        UC_Account("Manage Account")
        UC_Cart("Manage Cart")
        UC_PlaceOrder("Place Order")
        UC_Payment("Process Payment")
        UC_TrackOrder("Track Order")
        UC_Return("Request Return")

        UC_Prod("Manage Products")
        UC_Inv("Manage Inventory")
        UC_PO("Manage Purchase Orders")
        UC_SO("Manage Sales Orders")
        UC_Promo("Manage Promotions")
        UC_UserRole("Manage Users & Roles")
        UC_Report("View Reports")
        UC_Shipment("Manage Shipments")
        UC_Refund("Process Refund")
    end

    %% Guest Interactions
    Guest --> UC_Browse
    Guest --> UC_Account

    %% Customer Interactions
    Customer --> UC_Browse
    Customer --> UC_Account
    Customer --> UC_Cart
    Customer --> UC_PlaceOrder
    Customer --> UC_Payment
    Customer --> UC_TrackOrder
    Customer --> UC_Return

    %% Sales Staff Interactions
    Sales --> UC_Prod
    Sales --> UC_SO
    Sales --> UC_Return
    Sales --> UC_Refund

    %% Warehouse Staff Interactions
    Warehouse --> UC_Inv
    Warehouse --> UC_PO
    Warehouse --> UC_Shipment
    Warehouse --> UC_Return

    %% Marketing Staff Interactions
    Marketing --> UC_Promo
    Marketing --> UC_Report

    %% Administrator Interactions
    Admin --> UC_UserRole
    Admin --> UC_Prod
    Admin --> UC_Inv
    Admin --> UC_SO
    Admin --> UC_Promo
    Admin --> UC_Report

    %% External Actor Associations
    UC_PO --- Supplier
    UC_Shipment --- Shipper
    UC_TrackOrder --- Shipper
    UC_Payment --- PaymentGate
    UC_Refund --- PaymentGate
~~~

---

## 3.3. Quy tắc sử dụng và đặc tả Use Case

Khi đặc tả chi tiết từng Use Case trong tài liệu tiếp theo (`04-Use-Case-Specifications.md`), cần tuân thủ nghiêm ngặt các nguyên tắc:
* **Actor:** Phải có tương tác thực tế với chức năng được mô tả.
* **Preconditions (Tiền điều kiện):** Trạng thái bắt buộc phải tồn tại và đúng trước khi Use Case bắt đầu.
* **Postconditions (Hậu điều kiện):** Trạng thái dữ liệu và hệ thống phải đạt được sau khi Use Case hoàn thành.
* **Main Flow (Luồng chính):** Mô tả kịch bản thành công chuẩn (Happy Path).
* **Alternative Flows (Luồng thay thế):** Mô tả những nhánh xử lý hợp lệ khác trong nghiệp vụ.
* **Exception Flows (Luồng ngoại lệ):** Mô tả những tình huống lỗi hoặc vi phạm điều kiện khiến quy trình không thể tiếp tục và cách hệ thống bảo vệ dữ liệu (ví dụ: Rollback).