# M2.04 - MASTER CONCEPTUAL ERD DIAGRAM (SƠ ĐỒ QUAN HỆ KHÁI NIỆM TỔNG THỂ)

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.0 – Official Release (Master Integration)
* **Hệ quản trị CSDL mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Kế thừa trực tiếp:**
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (70 Thực thể E01 – E70)
  * [02-Relationship-Analysis.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/02-Relationship-Analysis.md) (Phân tích bản số Crow's Foot 11 Phân hệ)
  * [05-Design-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/05-Design-Decisions.md) (3 Nguyên tắc bất biến & Quyết định thiết kế)
  * Quyết định kiến trúc [11-Business-Decisions.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M1-Business-Analysis/11-Business-Decisions.md) (BD-01 đến BD-07)

---

## 1. TỔNG QUAN KIẾN TRÚC MÔ HÌNH DỮ LIỆU KHÁI NIỆM (M2 ARCHITECTURE)

Hệ thống **RetailCore – E-commerce Management System** được mô hình hóa ở tầng Quan niệm (Conceptual Design) bao gồm **70 Thực thể chuẩn hóa (E01 đến E70)** phân bổ trên **11 Phân hệ nghiệp vụ (Subject Areas)**. 

Mô hình tuân thủ triệt để phương pháp luận Crow's Foot Notation, xác định rõ bản số ($1:1$, $1:N$, $N:N$), tính tham gia bắt buộc/tùy chọn (Total/Partial Participation), bảo đảm tính toàn vẹn tham chiếu khái niệm và chuẩn bị nền tảng hoàn hảo cho bước chuyển đổi sang Mô hình Luận lý (Logical Schema) chuẩn 3NF/BCNF trên Microsoft SQL Server.

### Bản đồ 11 Phân hệ Nghiệp vụ và 70 Thực thể:
```
+----------------------------------------------------------------------------------------------------+
|                                 RETAILCORE CONCEPTUAL DOMAINS                                      |
+----------------------------------------------------------------------------------------------------+
|  01. Product Catalog (E01..E09)     | 05. Shopping (E38..E39)       | 09. Payment & Refund (E55..E59)  |
|  02. Warehouse & Inv. (E10..E20)   | 06. Sales (E40..E43)          | 10. Return & Exchange (E60..E64)|
|  03. Procurement (E21..E28)         | 07. Fulfillment (E44..E48)    | 11. Audit & Reporting (E65..E70) |
|  04. Customer & Identity (E29..E37) | 08. Promotion (E49..E54)      |                                  |
+----------------------------------------------------------------------------------------------------+
```

---

## 2. NĂM BẤT BIẾN KIẾN TRÚC BẮT BUỘC TRONG THIẾT KẾ ERD

Mọi mối quan hệ trong sơ đồ ERD tổng thể phải thỏa mãn tuyệt đối 5 bất biến kiến trúc:
1. **Phân tách 4 khái niệm Tồn kho (BD-04, WI-Q01..06):**
   * `OrderWarehouseAllocation` (E12 - Kế hoạch phân bổ kho).
   * `StockReservation` & `StockReservationLine` (E13, E14 - Khóa giữ logic có thời hạn).
   * `ShipmentLine` (E46 - Đóng gói thực tế xuất bãi).
   * `InventoryTransaction` & `InventoryTransactionLine` (E15, E16 - Sổ cái ghi sổ hai chiều Dual-Delta `OnHandDelta`/`ReservedDelta`).
2. **Độc lập 3 Máy trạng thái Giao dịch (SALES-Q02, PAY-Q08, SHIP-Q06):**
   * `SalesOrder.OrderStatus` $\ne$ `Payment.PaymentStatus` $\ne$ `Shipment.ShipmentStatus`.
3. **Nguyên tử hóa Tạo đơn & Giữ hàng (All-or-Nothing, SALES-Q01, BD-04):**
   * `ShoppingCart` chỉ chuyển `CONVERTED` thành `SalesOrder` khi `StockReservation` khóa thành công 100% tồn kho trên tất cả các kho được phân bổ.
4. **Bất biến Dữ liệu Lịch sử & Cấm Xóa cứng (Append-Only / No Hard Delete):**
   * `OrderLine` (E41) snapshot nguyên trạng SKU, tên, đơn giá và `VariantAttributesSnapshot` JSON.
   * `OrderAddress` (E42) snapshot độc lập với sổ địa chỉ `CustomerAddress` (E36).
   * Chứng từ giao dịch (`SalesOrder`, `Payment`, `Shipment`, `ReturnRequest`, `AuditLog`) tuyệt đối không xóa vật lý.
5. **Cấu trúc Khuyến mãi Hai tầng (BD-05, PROMO-Q01..08):**
   * Tối đa 1 khuyến mãi cấp sản phẩm liên kết với `OrderLine` và tối đa 1 voucher cấp đơn liên kết với `SalesOrder`. Ghi nhận lịch sử chiết khấu snapshot vào `PromotionUsage` (E53).

---

## 3. MASTER CROSS-DOMAIN BACKBONE ERD (SƠ ĐỒ TRỤC XƯƠNG SỐNG LIÊN PHÂN HỆ)

Sơ đồ biểu diễn **Trục xương sống liên kết liên miền (Cross-Domain Backbone)** kết nối các Thực thể Trung tâm (Hub Entities) đại diện cho 11 phân hệ:

> [!NOTE]
> **Tham chiếu Sơ đồ & Mã nguồn:**  
> * **Mã nguồn Mermaid:** [Cross-Domain-Backbone.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Cross-Domain-Backbone.md)  
> * **Sơ đồ ảnh Render:** [Cross-Domain-Backbone.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Cross-Domain-Backbone.png)

![Cross-Domain Backbone](ERD/images/Cross-Domain-Backbone.png)

---

## 4. MA TRẬN QUAN HỆ LIÊN MIỀN (CROSS-SUBJECT-AREA RELATIONSHIPS MATRIX)

Bảng dưới đây chuẩn hóa toàn bộ các liên kết vượt biên giới Subject Area, xác định bản số Crow's Foot, tính tham gia (Participation) và quy tắc nghiệp vụ/quyết định kiến trúc chịu trách nhiệm:

| STT | Thực thể Nguồn (Miền A) | Ký hiệu Crow's Foot | Thực thể Đích (Miền B) | Bản số | Tính tham gia | Vai trò nghiệp vụ & Căn cứ kiến trúc |
| :---: | :--- | :---: | :--- | :---: | :---: | :--- |
| **CR-01** | `ProductVariant` (01) | `|| ─── }o` | `InventoryBalance` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Một SKU có thể có số dư tồn kho tại nhiều kho vật lý. Mỗi bản ghi số dư gắn với đúng 1 SKU (WI-Q01). |
| **CR-02** | `ProductVariant` (01) | `|| ─── }o` | `OrderWarehouseAllocation` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Phân bổ xuất kho xác định chính xác SKU cần xuất lấy từ kho vật lý (BD-03, WI-Q03). |
| **CR-03** | `ProductVariant` (01) | `|| ─── }o` | `StockReservationLine` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng khóa giữ hàng xác định rõ SKU nào đang bị phong tỏa (BD-04, WI-Q02). |
| **CR-04** | `ProductVariant` (01) | `|| ─── }o` | `InventoryTransactionLine` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng sổ cái ghi nhận mức biến động tồn kho `OnHandDelta`/`ReservedDelta` cho đúng 1 SKU (WI-Q02). |
| **CR-05** | `ProductVariant` (01) | `|| ─── }o` | `StockCountLine` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kiểm kê đối soát số lượng thực tế của từng SKU tại kho (WI-Q05). |
| **CR-06** | `ProductVariant` (01) | `|| ─── }o` | `WarehouseTransferLine` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Điều chuyển hàng giữa hai kho vật lý theo từng SKU (WI-Q04). |
| **CR-07** | `ProductVariant` (01) | `|| ─── }o` | `SupplierProduct` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | SKU được cung cấp bởi danh sách các nhà cung cấp hợp lệ (PR-Q01). |
| **CR-08** | `ProductVariant` (01) | `|| ─── }o` | `PurchaseOrderLine` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng đặt mua hàng từ nhà cung cấp theo mã SKU cụ thể. |
| **CR-09** | `ProductVariant` (01) | `|| ─── }o` | `GoodsReceiptLine` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kiểm nhận hàng nhập kho theo từng SKU cụ thể (PR-Q03, PR-Q04). |
| **CR-10** | `ProductVariant` (01) | `|| ─── }o` | `SupplierReturnLine` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Xuất trả hàng lỗi/kém chất lượng cho NCC theo SKU (PR-Q05). |
| **CR-11** | `ProductVariant` (01) | `|| ─── }o` | `CartItem` (05) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Mặt hàng SKU khách hàng chọn thêm vào giỏ mua sắm (SHOP-Q03). |
| **CR-12** | `ProductVariant` (01) | `|| ─── }o` | `OrderLine` (06) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | SKU bán trong đơn hàng; `OrderLine` snapshot dữ liệu để độc lập với Catalog (SALES-Q05). |
| **CR-13** | `ProductVariant` (01) | `|| ─── }o` | `PromotionProduct` (08) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Danh sách SKU thuộc diện được chiết khấu theo chương trình khuyến mãi cấp sản phẩm (PROMO-Q01). |
| **CR-14** | `Warehouse` (02) | `|| ─── }o` | `PurchaseOrder` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kho vật lý được chỉ định tiếp nhận toàn bộ hàng hóa của đơn đặt mua PO (PR-Q01). |
| **CR-15** | `Warehouse` (02) | `|| ─── }o` | `GoodsReceipt` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kho tiếp nhận thực tế đợt giao hàng; phải khớp với kho nhận của PO (PR-Q01, PR-Q02). |
| **CR-16** | `Warehouse` (02) | `|| ─── }o` | `SupplierReturn` (03) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kho vật lý xuất trả hàng cho nhà cung cấp (PR-Q05). |
| **CR-17** | `Warehouse` (02) | `|| ─── }o` | `Shipment` (07) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Mỗi kiện hàng giao bắt buộc chỉ xuất phát từ duy nhất 1 kho vật lý (SHIP-Q01, BD-03). |
| **CR-18** | `Warehouse` (02) | `|| ─── }o` | `ReturnRequest` (10) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kho vật lý được chỉ định tiếp nhận hàng đổi trả của yêu cầu RMA (RET-Q06). |
| **CR-19** | `Warehouse` (02) | `|| ─── }o` | `ReturnReceipt` (10) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Kho tiếp nhận kiện hàng khách gửi trả thực tế tại bến nhập (RET-Q06). |
| **CR-20** | `OrderLine` (06) | `|| ─── }|` | `OrderWarehouseAllocation` (02) | $1:1..N$ | Nguồn: **Total**<br>Đích: **Total** | Một dòng đơn hàng bắt buộc phân bổ cho ít nhất một kho xuất hàng (Split Shipment, BD-03, WI-Q03). |
| **CR-21** | `SalesOrder` (06) | `|| ─── }o` | `StockReservation` (02) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Đơn hàng tạo các phiếu khóa giữ tồn kho tạm thời trước khi xuất bãi (BD-04, WI-Q02). |
| **CR-22** | `Customer` (04) | `|| ─── }o` | `ShoppingCart` (05) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Khách hàng sở hữu giỏ mua sắm; tối đa 1 giỏ `ACTIVE` tại một thời điểm (SHOP-Q01). |
| **CR-23** | `Customer` (04) | `|| ─── }o` | `SalesOrder` (06) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Khách hàng ký hợp đồng mua bán; bắt buộc đăng nhập (BD-02, cấm Guest Checkout). |
| **CR-24** | `Customer` (04) | `o| ─── }o` | `Voucher` (08) | $0..1:0..N$ | Nguồn: Partial<br>Đích: Partial | Voucher có thể cấp riêng cho khách hàng VIP (`AssignedCustomerID`) hoặc dùng chung toàn sàn (PROMO-Q03). |
| **CR-25** | `Customer` (04) | `|| ─── }o` | `PromotionTargetCustomer` (08) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Khách hàng thuộc danh sách thụ hưởng chương trình ưu đãi đặc quyền (PROMO-Q03). |
| **CR-26** | `Customer` (04) | `|| ─── }o` | `ReturnRequest` (10) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Khách hàng là chủ thể sở hữu và gửi yêu cầu đổi trả hàng (RET-Q01). |
| **CR-27** | `ShoppingCart` (05) | `|| ─── o|` | `SalesOrder` (06) | $1:0..1$ | Nguồn: Partial<br>Đích: **Total** | Giỏ hàng chuyển đổi sang đơn hàng thành công thì đóng băng `CONVERTED` (SHOP-Q05, SALES-Q01). |
| **CR-28** | `SalesOrder` (06) | `|| ─── }o` | `Shipment` (07) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Đơn hàng có thể phân tách thành nhiều kiện hàng xuất giao từ các kho khác nhau (BD-03, SHIP-Q01). |
| **CR-29** | `OrderLine` (06) | `|| ─── }o` | `ShipmentLine` (07) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng đơn hàng được đóng gói trong các lô giao; tổng xuất không vượt đặt (SHIP-Q01). |
| **CR-30** | `OrderWarehouseAllocation` (02) | `|| ─── }o` | `ShipmentLine` (07) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng lô giao hàng hiện thực hóa kế hoạch phân bổ của kho xuất (SHIP-Q01, BR-FS07). |
| **CR-31** | `SalesOrder` (06) | `|| ─── }o` | `PromotionUsage` (08) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Đơn hàng ghi nhận lịch sử tiêu thụ voucher cấp đơn hàng (BD-05, PROMO-Q05). |
| **CR-32** | `OrderLine` (06) | `o| ─── }o` | `PromotionUsage` (08) | $0..1:0..N$ | Nguồn: Partial<br>Đích: Partial | Dòng đơn hàng ghi nhận ưu đãi chiết khấu trực tiếp cấp sản phẩm (BD-05, PROMO-Q01). |
| **CR-33** | `SalesOrder` (06) | `|| ─── ||` | `Payment` (09) | $1:1$ | Nguồn: **Total**<br>Đích: **Total** | Mỗi đơn hàng bắt buộc có đúng 1 hồ sơ nghĩa vụ thanh toán (`Payment.OrderID` UNIQUE, PAY-Q01). |
| **CR-34** | `SalesOrder` (06) | `|| ─── }o` | `ReturnRequest` (10) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Đơn hàng đã giao thành công có thể phát sinh nhiều yêu cầu đổi trả từng phần (RET-Q01, RET-Q03). |
| **CR-35** | `OrderLine` (06) | `|| ─── }o` | `ReturnRequestLine` (10) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Dòng sản phẩm trong đơn được yêu cầu trả lại; tổng trả không vượt số lượng đã mua (RET-Q03). |
| **CR-36** | `ReturnRequest` (10) | `|| ─── }o` | `Refund` (09) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Yêu cầu đổi trả sau khi kho kiểm định đạt chuẩn sẽ tự động kích hoạt chứng từ hoàn tiền (RET-Q07). |
| **CR-37** | `Account` (04) | `o| ─── }o` | `AuditLog` (11) | $0..1:0..N$ | Nguồn: Partial<br>Đích: Partial | Tài khoản người dùng thực hiện thao tác nhạy cảm; sự kiện hệ thống tự động để NULL (AR-Q01). |
| **CR-38** | `Account` (04) | `|| ─── }o` | `ReportExecution` (11) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Tài khoản nhân viên yêu cầu xuất báo cáo hoặc thực thi truy vấn phân tích (AR-Q06). |
| **CR-39** | `Account` (04) | `o| ─── }o` | `ReportSnapshot` (11) | $0..1:0..N$ | Nguồn: Partial<br>Đích: Partial | Người dùng hoặc job hệ thống tạo bản chụp dữ liệu chốt kỳ báo cáo (AR-Q05). |
| **CR-40** | `Role` (04) | `|| ─── }o` | `ReportPermission` (11) | $1:0..N$ | Nguồn: Partial<br>Đích: **Total** | Vai trò được phân quyền truy cập, xuất file hoặc chốt snapshot báo cáo quản trị (AR-Q07). |

---

## 5. SƠ ĐỒ CHI TIẾT CÁC PHÂN HỆ VÀ ĐIỂM TIẾP GIÁP LIÊN MIỀN (DOMAIN BOUNDARIES)

Toàn bộ mã nguồn Mermaid chi tiết và sơ đồ ảnh render chất lượng cao của 11 phân hệ đã được bóc tách và quản lý tập trung trong thư mục [ERD/](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/). Dưới đây là đặc tả ranh giới tiếp giáp liên miền (Boundary Interfaces) và liên kết tham chiếu trực tiếp:

### 5.1. Phân hệ 01: Product Catalog (E01 – E09)
* **Giao diện liên miền:** 
  * `ProductVariant` (E04) là đầu mối kết nối đến: Inventory (02), Procurement (03), Shopping (05), Sales (06), Promotion (08).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Product-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Product-Catalog.md)
  * **Sơ đồ ảnh Render:** [Product-Catalog.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Product-Catalog.png)

![Product Catalog](ERD/images/Product-Catalog.png)

---

### 5.2. Phân hệ 02: Warehouse & Inventory (E10 – E20)
* **Giao diện liên miền:** 
  * `Warehouse` (E10) nhận liên kết từ PO/GR (03), Shipment (07), RMA (10).
  * `OrderWarehouseAllocation` (E12) nhận từ `OrderLine` (06) và cấp cho `ShipmentLine` (07).
  * `StockReservation` (E13) nhận từ `SalesOrder` (06).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Warehouse-Inventory.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Warehouse-Inventory.md)
  * **Sơ đồ ảnh Render:** [Warehouse-Inventory.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Warehouse-Inventory.png)

![Warehouse & Inventory](ERD/images/Warehouse-Inventory.png)

---

### 5.3. Phân hệ 03: Procurement (E21 – E28)
* **Giao diện liên miền:** Nhận SKU từ Catalog (01) và chỉ định nhập/xuất kho tại Warehouse (02).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Procurement.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Procurement.md)
  * **Sơ đồ ảnh Render:** [Procurement.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Procurement.png)

![Procurement](ERD/images/Procurement.png)

---

### 5.4. Phân hệ 04: Customer & Identity (E29 – E37)
* **Giao diện liên miền:** 
  * `Customer` (E30) liên kết với Shopping (05), Sales (06), Promotion (08), RMA (10).
  * `Account` (E29) và `Role` (E32) liên kết với Audit & Reporting (11).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Customer-Identity.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Customer-Identity.md)
  * **Sơ đồ ảnh Render:** [Customer-Identity.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Customer-Identity.png)

![Customer & Identity](ERD/images/Customer-Identity.png)

---

### 5.5. Phân hệ 05: Shopping (E38 – E39)
* **Giao diện liên miền:** Nhận `CustomerID` từ (04), `VariantID` từ (01); Chuyển đổi thành `SalesOrder` (06).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Shopping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Shopping.md)
  * **Sơ đồ ảnh Render:** [Shopping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Shopping.png)

![Shopping](ERD/images/Shopping.png)

---

### 5.6. Phân hệ 06: Sales (E40 – E43)
* **Giao diện liên miền:** 
  * Trung tâm điều phối toàn bộ chuỗi giá trị: Nhận `CartID` (05), `CustomerID` (04), `VariantID` (01).
  * Phát sinh phân bổ chia kho (02), giữ tồn kho (02), giao hàng (07), thanh toán (09), chiết khấu (08), khiếu nại RMA (10).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Sales.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Sales.md)
  * **Sơ đồ ảnh Render:** [Sales.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Sales.png)

![Sales](ERD/images/Sales.png)

---

### 5.7. Phân hệ 07: Fulfillment & Shipping (E44 – E48)
* **Giao diện liên miền:** Nhận `OrderID` và `OrderLineID` từ (06), `WarehouseID` và `AllocationID` từ (02).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Fulfillment-Shipping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Fulfillment-Shipping.md)
  * **Sơ đồ ảnh Render:** [Fulfillment-Shipping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Fulfillment-Shipping.png)

![Fulfillment & Shipping](ERD/images/Fulfillment-Shipping.png)

---

### 5.8. Phân hệ 08: Promotion & Voucher (E49 – E54)
* **Giao diện liên miền:** 
  * Chiết khấu SKU từ Catalog (01).
  * Tiêu thụ và snapshot giảm giá trên `SalesOrder` và `OrderLine` (06).
  * Chỉ định khách VIP từ Customer (04).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Promotion-Voucher.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Promotion-Voucher.md)
  * **Sơ đồ ảnh Render:** [Promotion-Voucher.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Promotion-Voucher.png)

![Promotion & Voucher](ERD/images/Promotion-Voucher.png)

---

### 5.9. Phân hệ 09: Payment & Refund (E55 – E59)
* **Giao diện liên miền:** 
  * Liên kết $1:1$ với `SalesOrder` (06).
  * Tiếp nhận yêu cầu kích hoạt hoàn tiền từ `ReturnRequest` (10).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Payment-Refund.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Payment-Refund.md)
  * **Sơ đồ ảnh Render:** [Payment-Refund.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Payment-Refund.png)

![Payment & Refund](ERD/images/Payment-Refund.png)

---

### 5.10. Phân hệ 10: Return & Exchange (E60 – E64)
* **Giao diện liên miền:** 
  * Tham chiếu đơn hàng gốc (`SalesOrder`, `OrderLine` từ 06).
  * Tiếp nhận tại kho hàng (`Warehouse` từ 02).
  * Kích hoạt hoàn tiền sang `Refund` (09).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Return-Exchange.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Return-Exchange.md)
  * **Sơ đồ ảnh Render:** [Return-Exchange.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Return-Exchange.png)

![Return & Exchange](ERD/images/Return-Exchange.png)

---

### 5.11. Phân hệ 11: Audit & Reporting (E65 – E70)
* **Giao diện liên miền:** 
  * `AuditLog` (E65) ghi nhận thao tác của mọi `Account` (04).
  * `ReportExecution` (E68) ghi nhận yêu cầu chạy báo cáo của `Account` (04).
  * `ReportPermission` (E70) phân quyền truy cập báo cáo theo `Role` (04).
* **Tham chiếu sơ đồ:**
  * **Mã nguồn Mermaid:** [Audit-Reporting.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Audit-Reporting.md)
  * **Sơ đồ ảnh Render:** [Audit-Reporting.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Audit-Reporting.png)

![Audit & Reporting](ERD/images/Audit-Reporting.png)

---

## 6. KIỂM TRA TÍNH TOÀN VẸN & KHÉP KÍN MÔ HÌNH (INTEGRITY CHECK)

1. **Đầy đủ 100% 70 Thực thể:** Toàn bộ 70 thực thể (E01 đến E70) đều xuất hiện trong sơ đồ và có liên kết quan hệ xác thực, không có bất kỳ thực thể nào bị cô lập (No Orphan Entities).
2. **Khép kín ranh giới nghiệp vụ:**
   * Mọi quan hệ $N:N$ đều được giải quyết thông qua Thực thể Kết hợp (Associative Entity) mang đầy đủ thuộc tính trạng thái và thời gian.
   * Tất cả các chứng từ giao dịch (SalesOrder, Payment, Shipment, GoodsReceipt, ReturnRequest, InventoryTransaction) đều có thực thể chi tiết dòng (Line/Detail) và nhật ký trạng thái (History) tương ứng.
3. **Sẵn sàng chuyển đổi M3:** Sơ đồ Conceptual ERD này là căn cứ tối hậu để chuyển dịch thành công sang Mô hình Luận lý (Logical Relational Schema) tại Milestone 3.
