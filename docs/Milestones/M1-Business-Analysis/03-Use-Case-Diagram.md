# 03. USE CASE DIAGRAM – SƠ ĐỒ CA SỬ DỤNG TỔNG THỂ (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT (Chuẩn hóa Phân tách Trách nhiệm SoD giữa Bán hàng, CSKH, Kho, KCS & Kế toán)

---

## 3.1. Phân nhóm Chức năng theo Tác nhân (Actor-to-Use-Case Mapping)

| Actor | Nhóm Use Cases Được Phép Thực Thi | Ràng buộc Phân quyền & SoD |
| :--- | :--- | :--- |
| **Guest** | Xem & tìm kiếm/lọc sản phẩm (`UC_Browse`), Đăng ký tài khoản & xác thực OTP (`UC_Account`). | Cấm đặt hàng nếu chưa đăng nhập (`BD-02`). |
| **Customer** | Duyệt sản phẩm (`UC_Browse`), Quản lý hồ sơ & sổ địa chỉ (`UC_Account`), Quản lý giỏ hàng (`UC_Cart`), Đặt hàng (`UC_PlaceOrder`), Thanh toán (`UC_Payment`), Theo dõi kiện giao (`UC_TrackOrder`), Gửi yêu cầu đổi trả (`UC_Return`). | Chỉ thao tác trên dữ liệu thuộc `CustomerID` của chính mình. |
| **Marketing Staff** | Quản lý thương hiệu, danh mục, sản phẩm & SKU (`UC_Prod`), Quản lý khuyến mãi 3 tầng & phát hành Voucher (`UC_Promo`), Xem báo cáo chiến dịch (`UC_Report`). | Không can thiệp tồn kho vật lý hoặc thanh toán. |
| **Sales Staff** | Tra cứu & xử lý đơn hàng, xác nhận đơn COD, xử lý hủy thiếu hàng (`UC_SO`), Theo dõi tiến độ giao vận (`UC_TrackOrder`). | **Không** sửa danh mục giá gốc Catalog; **không** duyệt chi hoàn tiền. |
| **Customer Support (`CSKH`)** | Tra cứu đơn hàng (`UC_SO`), Tiếp nhận/tạo hộ và xét duyệt yêu cầu đổi trả RMA (`UC_Return`). | Chỉ duyệt điều kiện RMA đầu vào, không tự giải ngân `Refund`. |
| **Warehouse Staff** | Quản lý tồn kho, chuyển kho 2 bước & kiểm kê (`UC_Inv`), Quản lý PO & nhập kho (`UC_PO`), Đóng gói xuất kiện & ký nhận hoàn giao thất bại (`UC_Shipment`), Tiếp nhận bưu kiện RMA tại bến nhập (`UC_ReceiveReturn`). | Người tạo phiếu kiểm kê/PO không được tự phê duyệt (`SoD`). |
| **QC / KCS Inspector** | Kiểm định chất lượng hàng nhập (`UC_PO`) và Giám định lỗi/phẩm chất hàng đổi trả (`UC_AssessReturn` – `ReturnLineAssessment` E72). | Độc lập kết luận lỗi (`FaultParty`) và định đoạt kho (`InventoryDisposition`). |
| **Finance / Accountant** | Phê duyệt & thực thi hoàn tiền (`UC_Refund`), Đối soát công nợ thu hộ COD 3 lớp (`UC_CODReconcile`), Xem báo cáo & chốt kỳ kế toán (`UC_Report`). | Chỉ giải ngân khi có chứng từ hủy hợp lệ hoặc biên bản `E72` được duyệt. |
| **Administrator** | Quản lý tài khoản TPT & phân quyền RBAC (`UC_UserRole`), Cấu hình danh mục nền tảng (`UC_Prod`), Phê duyệt điều chỉnh kiểm kê (`UC_Inv`), Giám sát `AuditLog` & báo cáo (`UC_Report`). | Cấm sửa/xóa nhật ký `AuditLog` (`WORM`). |
| **Supplier** *(External)* | Cung ứng hàng hóa theo `PurchaseOrder` và tiếp nhận hàng xuất trả `SupplierReturn`. | Tác nhân đối tác bên ngoài. |
| **Shipping Provider** *(External)* | Nhận bàn giao kiện hàng, cập nhật Webhook hành trình, thu hộ COD và gửi bảng kê đối soát. | Tích hợp qua API / Webhook đối tác vận chuyển. |
| **Payment Provider** *(External)* | Xử lý thanh toán trực tuyến, gửi Webhook xác nhận giao dịch và thực thi hoàn tiền qua cổng. | Tích hợp qua API / Webhook có xác thực chữ ký số. |

---

## 3.2. Sơ đồ Use Case Tổng thể (Mermaid)

~~~mermaid
flowchart LR
    %% Primary Actors
    Guest(["👤 Guest"])
    Customer(["🛍️ Customer"])
    Marketing(["🎯 Marketing Staff"])
    Sales(["💼 Sales Staff"])
    CSKH(["🎧 Customer Support"])
    Warehouse(["📦 Warehouse Staff"])
    KCS(["🔬 QC / KCS Inspector"])
    Accountant(["💰 Finance / Accountant"])
    Admin(["🛡️ Administrator"])

    %% External / Supporting Actors
    Supplier(["🏭 Supplier"])
    Shipper(["🚚 Shipping Provider"])
    PaymentGate(["💳 Payment Provider"])

    subgraph RetailCore["RetailCore Enterprise System"]
        UC_Browse("Browse & Filter Catalog")
        UC_Account("Register, Login & Manage Profile")
        UC_Cart("Manage Shopping Cart")
        UC_PlaceOrder("Place Order (Atomic Checkout)")
        UC_Payment("Process Online Payment")
        UC_TrackOrder("Track Order & Split Shipments")
        UC_Return("Request & Review RMA")

        UC_Prod("Manage Brands, Categories & SKUs")
        UC_Promo("Manage 3-Tier Promotions & Vouchers")
        UC_SO("Manage Sales Orders & Short-Pick")
        UC_PO("Manage PO, Goods Receipt & Vendor Return")
        UC_Inv("Manage Inventory, Transfer & Stock Count")
        UC_Shipment("Pack Shipments & Delivery Return In")
        UC_ReceiveReturn("Receive Returned Parcels at Dock")
        UC_AssessReturn("Assess Return Quality & Fault (E72)")
        UC_Refund("Approve & Execute Refund")
        UC_CODReconcile("Reconcile 3-Layer COD Settlement")
        UC_UserRole("Manage TPT Accounts & RBAC")
        UC_Report("View Reports, Snapshots & Audit Logs")
    end

    %% Guest & Customer
    Guest --> UC_Browse
    Guest --> UC_Account
    Customer --> UC_Browse
    Customer --> UC_Account
    Customer --> UC_Cart
    Customer --> UC_PlaceOrder
    Customer --> UC_Payment
    Customer --> UC_TrackOrder
    Customer --> UC_Return

    %% Commercial & Operations
    Marketing --> UC_Prod
    Marketing --> UC_Promo
    Marketing --> UC_Report

    Sales --> UC_SO
    Sales --> UC_TrackOrder

    CSKH --> UC_SO
    CSKH --> UC_Return

    %% Warehouse, KCS & Finance (Strict SoD)
    Warehouse --> UC_Inv
    Warehouse --> UC_PO
    Warehouse --> UC_Shipment
    Warehouse --> UC_ReceiveReturn

    KCS --> UC_PO
    KCS --> UC_AssessReturn

    Accountant --> UC_Refund
    Accountant --> UC_CODReconcile
    Accountant --> UC_Report

    %% System Administration
    Admin --> UC_UserRole
    Admin --> UC_Prod
    Admin --> UC_Inv
    Admin --> UC_Report

    %% External Integrations
    UC_PO --- Supplier
    UC_Shipment --- Shipper
    UC_TrackOrder --- Shipper
    UC_CODReconcile --- Shipper
    UC_Payment --- PaymentGate
    UC_Refund --- PaymentGate
~~~

---

## 3.3. Quy tắc Đặc tả Use Case Chuẩn hóa

Mọi đặc tả chi tiết trong [04-Use-Case-Specifications.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/04-Use-Case-Specifications.md) tuân thủ nghiêm ngặt:
1. **Ranh giới Giao dịch (ACID Boundary):** Chỉ rõ bước nào nằm trong Boundary Stored Procedure trên SQL Server và bước nào gọi HTTP ra ngoài (tuyệt đối không giữ khóa CSDL khi chờ mạng ngoài).
2. **Chống gửi lặp (Idempotency):** Xác định rõ cơ chế khóa `IdempotencyRecord` (E74) hoặc `PaymentWebhookLog` (E71) cho các thao tác tạo đơn, thanh toán và nhận callback.
3. **Phân tách Trách nhiệm (SoD):** Đảm bảo không có một tác nhân đơn lẻ nào vừa tự kiểm định kho vừa tự giải ngân tiền hoàn.
