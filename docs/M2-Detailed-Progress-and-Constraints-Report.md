# BÁO CÁO TIẾN ĐỘ DỰ ÁN RETAILCORE & THUYẾT MINH CHI TIẾT CỘT MỐC M2
## CHI TIẾT TOÀN DIỆN MÔ HÌNH DỮ LIỆU KHÁI NIỆM (M2), BẢN ĐỒ ERD, HỆ THỐNG RÀNG BUỘC KỸ THUẬT & PHƯƠNG HƯỚNG M3

* **Hệ thống Quản trị:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu Mục tiêu:** Microsoft SQL Server Enterprise Edition
* **Cột mốc Trọng tâm:** Đánh giá Chi tiết Milestone 2 (Conceptual Design) & Bản lề chuyển tiếp Milestone 3 (Logical Design)
* **Phiên bản:** v1.0 (Comprehensive Architecture Review)
* **Ngày lập báo cáo:** 23/11/2026
* **Chủ trì thẩm định:** Lead Enterprise Database Engineer & Solution Architect

---

## 1. TỔNG QUAN TIẾN ĐỘ DỰ ÁN (PROJECT PROGRESS OVERVIEW)

Dự án RetailCore được triển khai theo triết lý **Database-First / Database-Centric**, chia thành 8 Cột mốc (Milestones). Dưới đây là bảng định vị tiến độ thực tế đến ngày 23/11/2026:

| Cột mốc (Milestone) | Trọng số | Trạng thái Thực tế | Tiến độ | Kết quả Nghiệm thu Kỹ thuật & Bàn giao |
| :--- | :---: | :---: | :---: | :--- |
| **M1: Phân tích Nghiệp vụ & Yêu cầu** *(Business Analysis)* | 20% | **HOÀN THÀNH (100%)** | 100% | • 10 Phân hệ nghiệp vụ cốt lõi.<br>• 112 Quy tắc nghiệp vụ (`BR-01` đến `BR-112`) được mã hóa.<br>• Đóng gói 11 tài liệu đặc tả chuẩn mực (Charter, Use Cases, Process Flows, Traceability Matrix). |
| **M2: Thiết kế Cơ sở Dữ liệu Mức Quan niệm** *(Conceptual Design)* | 30% | **NGHIỆM THU CÓ ĐIỀU KIỆN (Conditional Approval)** | 100% | • Chuẩn hóa 72 thực thể quan niệm (**E01 – E72**) trên 11 Vùng nghiệp vụ (**SA01 – SA11**).<br>• Hoàn thành 12 bộ sơ đồ Mermaid ERD cú pháp chuẩn `PK`, `FK`, `UK` và ảnh render PNG.<br>• Ban hành Bản ghi hiệu chỉnh kiến trúc `M2.08` giải quyết 10 lỗi nền tảng (CORR-01..10). |
| **Giai đoạn Bản lề Pre-M3 (Current Gate)** | — | **ĐÃ GIẢI QUYẾT 100% (Signed Off)** | 100% | • Hoàn thành Báo cáo chuyển giao `Project-Progress-Report-Pre-M3.md` (v2.1).<br>• Tháo gỡ triệt để 10 Open Issues kiến trúc tại `docs/M3-Issues-Resolution.md`.<br>• Đã Push commit `b60172b` lên nhánh `origin/main` trên GitHub. |
| **M3: Thiết kế Cơ sở Dữ liệu Mức Logic** *(Logical Design)* | 25% | **KHỞI ĐỘNG (Starting M3.01)** | 0% | • Sẵn sàng ánh xạ lược đồ quan hệ (Schema Mapping) cho phân hệ đầu tiên: `SA01 – Product Catalog`.<br>• Lộ trình gồm 4 công đoạn: M3.01 (Mapping) $\to$ M3.02 (3NF/BCNF) $\to$ M3.03 (Data Types/Sizing) $\to$ M3.04 (Constraints/SPs). |
| **M4: Thiết kế Vật lý & Tối ưu Hiệu năng** *(Physical Design)* | 15% | **CHỜ THỰC HIỆN** | 0% | • Phân vùng bảng (Partitioning), Chỉ mục chuyên sâu (Filtered/Columnstore), Chiến lược HA/DR. |
| **M5: DDL Triển khai & Backend** | 10% | **CHỜ THỰC HIỆN** | 0% | • Script T-SQL hoàn chỉnh, Seed Data 1.000.000 dòng tải thử nghiệm, Spring Boot API. |
| **M6: Frontend** | — | **CHỜ THỰC HIỆN** | 0% | • Ứng dụng giao diện React, TypeScript, Tailwind CSS. |
| **M7: Testing & Debugging** | — | **CHỜ THỰC HIỆN** | 0% | • Kiểm thử toàn vẹn dữ liệu, Concurrency stress test, Transaction integrity test. |
| **M8: Đóng gói & Triển khai** | — | **CHỜ THỰC HIỆN** | 0% | • Docker hóa, CI/CD pipeline, Giám sát hệ thống. |

---

## 2. BÁO CÁO CHI TIẾT MILESTONE 2: MÔ HÌNH DỮ LIỆU KHÁI NIỆM (M2)

M2 đóng vai trò xác lập ranh giới nghiệp vụ (Bounded Contexts), định danh thực thể dữ liệu hạt nhân và phân định bản số (cardinality) giữa các mối quan hệ.

### 2.1. Danh mục 72 Thực thể Quan niệm (E01 – E72) qua 11 Vùng nghiệp vụ (Subject Areas)

Hệ thống RetailCore bao phủ toàn bộ chuỗi giá trị bán lẻ đa kênh thông qua 72 thực thể được phân bố trên 11 Subject Areas (SA):

1. **SA01 – Product Catalog (Quản lý Danh mục & Biến thể Sản phẩm):**
   * **Thực thể (E01 – E09):** `Category` (E01), `Product` (E02), `ProductCategory` (E03), `ProductVariant` (E04 - Hạt nhân SKU), `Attribute` (E05), `AttributeValue` (E06), `ProductAttribute` (E07), `ProductAttributeValue` (E08), `VariantAttributeValue` (E09).
   * **Đặc điểm:** Tách biệt cấu trúc sản phẩm cha (`Product`) và biến thể thương mại bán lẻ (`ProductVariant`). Quản lý cây danh mục đa tầng đệ quy và từ điển thuộc tính động theo ma trận 2 lớp.
2. **SA02 – Warehouse & Inventory (Kho bãi & Tồn kho Đa kho):**
   * **Thực thể (E10 – E20):** `Warehouse` (E10), `InventoryBalance` (E11 - Sổ số dư kho), `OrderWarehouseAllocation` (E13 - Phân bổ kho), `StockReservation` (E14), `StockReservationLine` (E15), `InventoryTransaction` (E16 - Sổ cái biến động kho), `InventoryTransactionLine` (E17), `StockCount` (E18 - Phiếu kiểm kê), `StockCountLine` (E19), `WarehouseTransfer` (E20), `WarehouseTransferLine` (E12).
   * **Đặc điểm:** Tách bạch 4 trạng thái tồn kho; quản lý phân bổ đa kho phục vụ Split Shipment; lưu vết giao dịch kép cho mọi biến động vật lý.
3. **SA03 – Procurement (Chuỗi Cung ứng & Mua hàng):**
   * **Thực thể (E21 – E28):** `Supplier` (E21), `SupplierContact` (E22), `SupplierProduct` (E23), `PurchaseOrder` (E24 - Đơn PO), `PurchaseOrderLine` (E25), `GoodsReceipt` (E26 - Biên bản nhập kho), `GoodsReceiptLine` (E27), `SupplierReturn` (E28), `SupplierReturnLine` (Kế thừa E28b).
   * **Đặc điểm:** Kiểm soát đơn đặt hàng mua có kho nhận duy nhất, tiếp nhận hàng nhiều đợt (Partial Receipt), tách biệt hàng kiểm nhận đạt/không đạt và quản lý xuất trả NCC.
4. **SA04 – Customer & Identity (Khách hàng & Định danh):**
   * **Thực thể (E29 – E37):** `Account` (E29 - Hạt nhân định danh), `Customer` (E30), `Employee` (E31), `Role` (E32), `Permission` (E33), `AccountRole` (E34), `RolePermission` (E35), `CustomerAddress` (E36), `CustomerGroup` (E37).
   * **Đặc điểm:** Kiến trúc phân cấp kế thừa TPT (Table-Per-Type) giữa `Account` và `Customer`/`Employee` với ràng buộc loại trừ XOR.
5. **SA05 – Shopping (Trải nghiệm Mua sắm & Giỏ hàng):**
   * **Thực thể (E38 – E39):** `Cart` (E38), `CartItem` (E39).
   * **Đặc điểm:** Lưu giữ giỏ hàng trực tuyến của khách; chuyển đổi nguyên tử sang dòng đơn hàng khi checkout.
6. **SA06 – Sales & Order Management (Bán hàng & Đơn hàng):**
   * **Thực thể (E40 – E43):** `SalesOrder` (E40 - Đơn hàng hạt nhân), `OrderLine` (E41), `OrderAddress` (E42 - Snapshot địa chỉ bất biến), `OrderPromotion` (E43).
   * **Đặc điểm:** Snapshot 100% giá bán, thuế VAT, địa chỉ giao hàng tại thời điểm đặt; độc lập hoàn toàn với việc sửa đổi dữ liệu sau đó.
7. **SA07 – Fulfillment & Shipping (Đóng gói & Giao vận):**
   * **Thực thể (E44 – E48):** `Shipment` (E44 - Bưu kiện vật lý), `Carrier` (E45), `ShipmentLine` (E46), `ShipmentStatusHistory` (E47), `DeliveryReturnReceipt` (E48).
   * **Đặc điểm:** Quản lý chia tách bưu kiện (Split Shipment); theo dõi trạng thái vận chuyển từ bưu cục đến khách; xử lý giao thất bại quay đầu độc lập với kho.
8. **SA08 – Promotion & Voucher (Khuyến mãi & Mã giảm giá):**
   * **Thực thể (E49 – E54):** `Promotion` (E49), `Voucher` (E50), `PromotionRule` (E51), `PromotionReward` (E52), `PromotionUsage` (E53), `VoucherCustomer` (E54).
   * **Đặc điểm:** Hỗ trợ quy tắc khuyến mãi đa tầng (Order / Item / Shipping), quản lý mã voucher cá nhân hóa và giới hạn tần suất sử dụng.
9. **SA09 – Payment & Settlement (Thanh toán & Đối soát):**
   * **Thực thể (E55 – E59, E71):** `Payment` (E55 - Hồ sơ nghĩa vụ thanh toán đơn), `PaymentTransaction` (E56 - Giao dịch cổng), `Refund` (E57 - Chứng từ hoàn tiền), `RefundTransaction` (E58), `RefundAllocation` (E59), `PaymentWebhookLog` (E71 - Nhật ký Webhook cổng thanh toán).
   * **Đặc điểm:** Quan hệ 1:1 với Đơn hàng; 1:N với các lần thanh toán thử; lưu trữ bền vững Webhook chống tấn công Replay.
10. **SA10 – Return & Exchange (Hậu cần Ngược & Giám định):**
    * **Thực thể (E60 – E64, E72):** `ReturnRequest` (E60 - Yêu cầu trả hàng), `ReturnRequestLine` (E61), `ReturnReceipt` (E62 - Phiếu nhận hàng trả), `ReturnReceiptLine` (E63), `ReturnShipmentAllocation` (Kế thừa E64b), `ReturnLineAssessment` (E72 - Giám định phân định trách nhiệm KCS).
    * **Đặc điểm:** Tách biệt quyền hoàn tiền khỏi vị trí phân loại tồn kho; giám định chi tiết từng sản phẩm theo phiên bản kiểm toán.
11. **SA11 – Audit & Reporting (Kiểm toán & Quản trị Hệ thống):**
    * **Thực thể (E65 – E70):** `AuditLog` (E65), `SystemConfig` (E66), `Notification` (E67), `NotificationTemplate` (E68), `DailySalesSummary` (E69), `InventoryDailySnapshot` (E70).
    * **Đặc điểm:** Lưu vết toàn bộ biến động quan trọng; bảng tổng hợp định kỳ phục vụ báo cáo quản trị BI tốc độ cao.

---

## 3. DANH MỤC CÁC TÀI LIỆU SƠ ĐỒ MERMAID ERD ĐÃ CÓ SẴN TRONG PROJECT

Để tối ưu hiệu năng render và tránh làm chậm trình xem file của extension, dự án đã tổ chức toàn bộ sơ đồ ERD thành các file chuyên biệt theo từng phân hệ. Có thể tra cứu trực tiếp tại các đường dẫn sau:

| STT | Phân hệ nghiệp vụ | File Sơ đồ Mermaid ERD | Ảnh Render Trực quan (PNG) |
| :---: | :--- | :--- | :--- |
| **00** | **Khung xương Liên miền Toàn hệ thống** | [Cross-Domain-Backbone.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Cross-Domain-Backbone.md) | [Cross-Domain-Backbone.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Cross-Domain-Backbone.png) |
| **01** | **Product Catalog** | [Product-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Product-Catalog.md) | [Product-Catalog.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Product-Catalog.png) |
| **02** | **Warehouse & Inventory** | [Warehouse-Inventory.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Warehouse-Inventory.md) | [Warehouse-Inventory.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Warehouse-Inventory.png) |
| **03** | **Procurement** | [Procurement.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Procurement.md) | [Procurement.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Procurement.png) |
| **04** | **Customer & Identity** | [Customer-Identity.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Customer-Identity.md) | [Customer-Identity.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Customer-Identity.png) |
| **05** | **Shopping** | [Shopping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Shopping.md) | [Shopping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Shopping.png) |
| **06** | **Sales & Order Management** | [Sales.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Sales.md) | [Sales.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Sales.png) |
| **07** | **Fulfillment & Shipping** | [Fulfillment-Shipping.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Fulfillment-Shipping.md) | [Fulfillment-Shipping.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Fulfillment-Shipping.png) |
| **08** | **Promotion & Voucher** | [Promotion-Voucher.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Promotion-Voucher.md) | [Promotion-Voucher.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Promotion-Voucher.png) |
| **09** | **Payment & Refund** | [Payment-Refund.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Payment-Refund.md) | [Payment-Refund.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Payment-Refund.png) |
| **10** | **Return & Exchange** | [Return-Exchange.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Return-Exchange.md) | [Return-Exchange.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Return-Exchange.png) |
| **11** | **Audit & Reporting** | [Audit-Reporting.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/mermaid/Audit-Reporting.md) | [Audit-Reporting.png](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/ERD/images/Audit-Reporting.png) |

---

## 4. CHI TIẾT CÁC RÀNG BUỘC KỸ THUẬT & QUY TẮC TOÀN VẸN (TECHNICAL CONSTRAINTS)

Hệ thống thiết lập các ràng buộc khống chế toán học chặt chẽ nhằm bảo vệ dữ liệu ở mức tầng CSDL:

### 4.1. Khống chế Số dư Tồn kho & 5 CHECK Constraints Toán học (CORR-02)
Trên thực thể số dư kho `InventoryBalance` (E11), dữ liệu được ràng buộc bởi phương trình cân bằng tồn kho:
$$\text{SellableOnHandQty} = \text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty}$$
$$\text{AvailableQty} = \text{SellableOnHandQty} - \text{ReservedQty} \ge 0$$

5 Ràng buộc toàn vẹn khai báo (Declarative CHECK Constraints):
1. `CK_InventoryBalance_OnHand_Positive`: $\text{OnHandQty} \ge 0$
2. `CK_InventoryBalance_Quarantined`: $0 \le \text{QuarantinedQty} \le \text{OnHandQty}$
3. `CK_InventoryBalance_Damaged`: $0 \le \text{DamagedQty} \le \text{OnHandQty}$
4. `CK_InventoryBalance_TotalDefect`: $\text{QuarantinedQty} + \text{DamagedQty} \le \text{OnHandQty}$
5. `CK_InventoryBalance_Reserved`: $0 \le \text{ReservedQty} \le (\text{OnHandQty} - \text{QuarantinedQty} - \text{DamagedQty})$

*Quy tắc biến động chất lượng:* Thống nhất 3 trạng thái chất lượng (`SELLABLE`, `QUARANTINED`, `DAMAGED`) và nghiệp vụ xuất hủy phế phẩm (`SCRAP_OUT`). Cấm tuyệt đối coi `RESERVED` là trạng thái chất lượng tồn kho.

### 4.2. Ràng buộc Khóa giữ Tồn kho Đa kho 1 : 0..N (CORR-10)
* Một đơn hàng có thể phân bổ từ nhiều kho (`OrderWarehouseAllocation`).
* Mỗi phân bổ kho có thể trải qua nhiều lượt giữ hàng lịch sử, nhưng tại một thời điểm chỉ có tối đa duy nhất **1 lượt giữ hàng có hiệu lực (`ACTIVE`)**.
* Cài đặt kỹ thuật tại SQL Server:
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UQ_StockReservationLine_ActiveAllocation 
  ON dbo.StockReservationLine (AllocationID) 
  WHERE ReservationStatus = 'ACTIVE';
  ```

### 4.3. Ràng buộc Phân bổ Chiết khấu LRM & Triệt tiêu Sai số Tiền tệ VNĐ (CORR-03 / B4)
* Áp dụng thuật toán **Hare-Niemeyer (Largest Remainder Method - LRM)** phân bổ giảm giá voucher đơn hàng trên tập các dòng sản phẩm đủ điều kiện (`EligibleBase`).
* Tiền tệ giao dịch VNĐ là số nguyên đồng sử dụng `DECIMAL(19,0)` (hoặc `BIGINT`). Đơn giá, tỷ lệ chiết khấu và giá vốn WAC sử dụng `DECIMAL(19,4)`. Cấm dùng `MONEY` hoặc `FLOAT`.
* Khi hoàn tiền một phần (Partial Return), số tiền hoàn được tính bằng hàm nguyên lũy tiến $F(q)$ bảo đảm khi hoàn hết toàn bộ số lượng thì số tiền hoàn trả đúng bằng tổng giá trị ròng thực trả, triệt tiêu hoàn toàn sai số phân số đồng lẻ.

### 4.4. Thực thi Ràng buộc Bắt buộc 1:1 trong Microsoft SQL Server (CORR-04)
* Microsoft SQL Server **không hỗ trợ** Deferrable Constraints (`DEFERRABLE INITIALLY DEFERRED`).
* Để thực thi quan hệ bắt buộc 1:1 (như giữa `SalesOrder` và `OrderAddress`, hoặc `SalesOrder` và `Payment`), hệ thống áp dụng chiến lược phòng thủ 2 lớp:
  1. Khóa ngoại có chỉ mục duy nhất `UNIQUE NONCLUSTERED INDEX (OrderID)` tại bảng con.
  2. Thu hồi toàn bộ quyền DML trực tiếp (`INSERT`, `UPDATE`, `DELETE`) trên các bảng này khỏi tài khoản ứng dụng; bắt buộc mọi thao tác khởi tạo phải đi qua Stored Procedure giao dịch nguyên tử có thiết lập `SET XACT_ABORT ON; BEGIN TRANSACTION ... COMMIT`.

### 4.5. Phân cấp Tài khoản TPT (Table-Per-Type) & Ràng buộc Loại trừ XOR (CORR-05)
* Cấu trúc kế thừa: `Account` (cha) $\to$ `Customer` (con 1) / `Employee` (con 2).
* Ràng buộc loại trừ (Disjoint): Một tài khoản không được phép vừa là `Customer` vừa là `Employee`. Kiểm soát qua cột loại tài khoản `AccountType` (`'CUSTOMER'`, `'EMPLOYEE'`) kết hợp Stored Procedure đăng ký và Trigger kiểm chứng.

### 4.6. Lưu trữ Webhook Bền vững & Chống Replay (CORR-06 / B7)
* Thiết lập bảng `PaymentWebhookLog` (E71) kiểm soát tính Idempotent bằng ràng buộc duy nhất:
  `UNIQUE NONCLUSTERED (ProviderID, MerchantAccountID, ProviderEventID, Environment)`
* Nguyên tắc giao dịch: Phản hồi HTTP 200 chỉ được trả về cho cổng thanh toán sau khi dữ liệu webhook đã được ghi nhận bền vững xuống đĩa CSDL qua Stored Procedure `usp_ReceivePaymentWebhook`.

### 4.7. Độc lập 4 Máy Trạng thái (CORR-07)
Tách biệt hoàn toàn máy trạng thái của 4 trục nghiệp vụ:
$$\text{OrderStatus} \ne \text{PaymentStatus} \ne \text{ShipmentStatus} \ne \text{ReturnStatus}$$
Loại bỏ hoàn toàn trạng thái xử lý trả hàng (`RMA_PROCESSING`) ra khỏi `OrderStatus`. Trạng thái đơn hàng tổng hợp dựa trên số lượng còn hiệu lực của các dòng sản phẩm. Tách biệt rõ 3 mốc thời gian: `DeliveredAt` (giao kiện cuối), `ReturnEligibleUntil` (hạn khiếu nại), và `ClosedAt` (đóng hồ sơ tài chính).

### 4.8. Truy vết Bưu kiện Đổi trả qua `ReturnShipmentAllocation` (Issue 1 - M3 Baseline)
Khi đơn hàng bị tách nhiều kiện giao lệch ngày (Split Shipment), hạn đổi trả được tính độc lập theo từng kiện `Shipment.DeliveredAt`. Ràng buộc số lượng trả:
$$\sum \text{AllocatedReturnQty} \le \text{ShipmentLine.ShipmentQty} - \text{PreviouslyReturnedQty}$$

### 4.9. Mô hình Đối soát COD 3 Lớp (Issue 2 - M3 Baseline)
Tách bạch dòng tiền thu hộ COD qua 3 bảng:
1. `CODCollection`: Shipper thu tiền khách theo từng kiện khi phát hàng (cập nhật đơn hàng thành `PARTIALLY_PAID`).
2. `CODRemittance`: Bưu cục/Hãng vận chuyển đối soát chuyển tiền về định kỳ.
3. `CODReconciliation`: Khớp lệnh đối chiếu bảng kê, cước phí trừ và chênh lệch phát sinh.

### 4.10. Audit Trail Giám định KCS E72 (Issue 5 - M3 Baseline)
Bảng `ReturnLineAssessment` áp dụng mô hình phiên bản hóa (`IsCurrent`, `AssessmentVersion`, `AssessmentStatus`: DRAFT, SUBMITTED, FINAL, SUPERSEDED) kèm Filtered Unique Index bảo đảm chỉ có duy nhất 1 bản ghi `IsCurrent = 1` cho mỗi đợt nhận hàng, cấm xóa hoặc sửa đè lịch sử giám định.

---

## 5. CÁC VẤN ĐỀ CÒN TỒN ĐỌNG & PHƯƠNG HƯỚNG GIẢI QUYẾT TẠI MILESTONE 3

Mặc dù toàn bộ 10 vấn đề kiến trúc tiền đề (Pre-M3 Issues) đã được giải quyết về mặt giải pháp và DDL mẫu tại `docs/M3-Issues-Resolution.md`, trong quá trình chuyển dịch sang **Milestone 3 (Logical Database Design)**, đội ngũ kỹ thuật cần tiếp tục xử lý các hạng mục kỹ thuật thực tế sau:

### Vấn đề 1: Hiện thực hóa Ánh xạ Lược đồ Quan hệ (M3.01) cho 11 Phân hệ
* **Thực trạng:** Thư mục `docs/Milestones/M3-Logical-Design/` hiện mới chỉ có `.gitkeep`. Cần chuyển đổi toàn bộ 72 thực thể quan niệm sang các bảng quan hệ SQL Server chính thức.
* **Phương hướng giải quyết:**
  Triển khai theo quy trình 8 bước đã phê duyệt tại `docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/`:
  1. `01-Mapping-Methodology.md` (Xác lập quy tắc ánh xạ).
  2. `02-Entity-to-Relation-Mapping.md` (Ánh xạ 72 thực thể sang quan hệ).
  3. `03-Primary-and-Candidate-Keys.md` (Xác định Surrogate Keys và Natural Keys).
  4. `04-Foreign-Key-Mapping.md` (Xác định khóa ngoại và hành vi toàn vẹn tham chiếu).
  5. `05-Relationship-Transformation.md` (Xử lý 1:1, 1:N, N:N, TPT Subtypes, Cây đệ quy).
  6. `06-Cross-Area-Relations.md` (Kiểm soát 40 liên kết liên vùng nghiệp vụ).
  7. `07-Open-Design-Decisions.md` (Đặc tả chi tiết COD 3 lớp và `ReturnShipmentAllocation`).
  8. `08-Mapping-Validation.md` (Biên bản nghiệm thu M3.01).
  *Trình tự thực hiện:* Bắt đầu ngay từ **Ưu tiên 1: `SA01 – Product Catalog` (E01 – E09)** do tính độc lập cao.

### Vấn đề 2: Đánh giá Chuẩn hóa Toán học 3NF & BCNF (M3.02)
* **Thực trạng:** Một số bảng như `OrderLine`, `ShipmentLine`, `GoodsReceiptLine` chứa các trường Snapshot (như `UnitPriceSnapshot`, `VatRateSnapshot`, `MovingAverageCostSnapshot`) có nguy cơ vi phạm BCNF nếu nhìn thuần túy theo lý thuyết phụ thuộc hàm.
* **Phương hướng giải quyết:**
  Áp dụng **Controlled Normalization Policy (Chính sách Chuẩn hóa có Kiểm soát)**:
  * 100% các bảng bắt buộc đạt chuẩn 3NF.
  * Với các trường hợp duy trì ở 3NF mà không phân rã lên BCNF để lưu vết lịch sử giao dịch bất biến (Point-in-time Snapshots), kỹ sư thiết kế bắt buộc phải lập biên bản giải trình kỹ thuật (Technical Trade-off Justification) chứng minh việc không gây ra dị thường cập nhật (Update Anomalies) do các bảng này là bảng bất biến (`Append-only / Immutable`).

### Vấn đề 3: Định kiểu Dữ liệu Vật lý Chi tiết & Ước tính Dung lượng CSDL (M3.03)
* **Thực trạng:** Cần quy hoạch chi tiết kiểu dữ liệu của từng cột cho 72+ bảng trên SQL Server và dự phóng dung lượng lưu trữ cho bài toán 1.000.000 đơn hàng/năm.
* **Phương hướng giải quyết:**
  * Thống nhất triệt để: Kiểu tiền tệ `DECIMAL(19,0)` cho số nguyên VNĐ; `DECIMAL(19,4)` cho đơn giá/tỷ lệ; Surrogate Keys dùng `BIGINT IDENTITY(1,1)` cho bảng giao dịch và `INT`/`TINYINT` cho bảng danh mục/cấu hình.
  * Chuỗi văn bản dùng `NVARCHAR` có chặn độ dài tối đa xác định, cấm dùng `NVARCHAR(MAX)` tràn lan.
  * Múi giờ chuẩn hóa `DATETIMEOFFSET(7)` (+07:00).
  * Lập bảng tính toán dung lượng 3 năm (Capacity Planning): Tính kích thước trung bình mỗi dòng (Row Size), dung lượng bảng (Data Space), và dung lượng chỉ mục (Index Space).

### Vấn đề 4: Hoàn thiện DDL Khai báo & Đặc tả Stored Procedures Giao dịch Cốt lõi (M3.04)
* **Thực trạng:** Các ràng buộc phức tạp (như bắt buộc 1:1, kiểm soát số dư kho khi bán hàng, xử lý webhook bất đồng bộ, đối soát COD) không thể chỉ dựa vào khai báo DDL thông thường mà đòi hỏi các Stored Procedures giao dịch chặt chẽ.
* **Phương hướng giải quyết:**
  * Đặc tả đầy đủ script DDL: `PRIMARY KEY`, `FOREIGN KEY` (với tùy chọn an toàn `ON DELETE NO ACTION`), `CHECK Constraints`, `DEFAULT Constraints`.
  * Thiết kế chi tiết chữ ký (signatures) và mã nguồn mẫu cho các Stored Procedures huyết mạch:
    1. `usp_CreateSalesOrderWithAddress` (Tạo đơn hàng và snapshot địa chỉ bắt buộc 1:1).
    2. `usp_AllocateAndReserveStock` (Phân bổ đa kho và khóa giữ tồn kho an toàn).
    3. `usp_PostInventoryTransaction` (Ghi sổ cái biến động kho và cập nhật số dư nguyên tử).
    4. `usp_ReceivePaymentWebhook` (Lưu vết webhook bền vững và phân nhánh xử lý).
    5. `usp_AssessReturnReceiptLine` (Phiên bản hóa biên bản giám định KCS E72).

---

## 6. KẾT LUẬN & ĐỀ XUẤT HÀNH ĐỘNG TIẾP THEO

* **Kết luận:** Milestone 2 (Thiết kế Khái niệm) và giai đoạn chuyển giao tiền đề Pre-M3 đã hoàn thành xuất sắc, được chuẩn hóa kỹ thuật và kiểm soát rủi ro triệt để. CSDL RetailCore đã có một nền tảng vững chắc, sẵn sàng 100% để bước vào thiết kế mức Logic.
* **Hành động đề xuất:**
  * **Hành động 1:** Tiến hành khởi tạo cấu trúc tài liệu công đoạn **M3.01 (Relational Schema Mapping)** trong thư mục `docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/`.
  * **Hành động 2:** Bắt tay triển khai ánh xạ quan hệ chi tiết cho phân hệ đầu tiên: **`SA01 – Product Catalog` (E01 – E09)**.

*(Báo cáo được lưu trữ chính thức tại `docs/M2-Detailed-Progress-and-Constraints-Report.md`)*

