# 09. DATA REQUIREMENTS – ĐẶC TẢ YÊU CẦU DỮ LIỆU (PHIÊN BẢN v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis (Đầu vào chuẩn hóa cho 79 Thực thể M2)
* **Phiên bản tài liệu:** 2.0 – Full DR-01..DR-30 Restoration & CORR Sync
* **Database target:** Microsoft SQL Server 2022+

---

## 9.1. Mười một Miền Chủ đề Dữ liệu Chuẩn hóa (11 Subject Areas)

| Mã Miền | Tên Miền Dữ liệu (Subject Area) | Nội dung Dữ liệu Cốt lõi Quản lý |
| :--- | :--- | :--- |
| **SA01** | Product Catalog | Thương hiệu (`Brand`), cây danh mục (`Category`), sản phẩm (`Product`), SKU (`ProductVariant`), thuộc tính động (`Attribute`..`VariantAttributeValue`). |
| **SA02** | Warehouse & Inventory | Kho hàng, số dư tồn kho 3 phẩm chất & WAC, phân bổ kho, giữ hàng có hạn, sổ cái biến động, kiểm kê, chuyển kho. |
| **SA03** | Procurement | Nhà cung cấp, danh mục cung ứng, đơn mua PO, phiếu nhập kho QC, phiếu xuất trả NCC. |
| **SA04** | Customer & Identity | Tài khoản xác thực TPT (`Account`, `Customer`, `Employee`), RBAC (`Role`, `Permission`), sổ địa chỉ, lịch sử OTP. |
| **SA05** | Shopping | Giỏ hàng trực tuyến tập trung và chi tiết dòng giỏ hàng. |
| **SA06** | Sales & Order Management | Hợp đồng đơn hàng, dòng đơn hàng (kèm LRM, thuế GTGT & JSON snapshot), địa chỉ nhận snapshot, lịch sử trạng thái đơn. |
| **SA07** | Fulfillment & Shipping | Hãng vận chuyển, vận đơn kiện hàng, chi tiết đóng gói, lịch sử lần giao, biên bản hoàn kho giao thất bại (`DeliveryReturnReceipt`). |
| **SA08** | Promotion & Voucher | Chương trình khuyến mãi phân tầng, điều kiện, SKU áp dụng, mã voucher, khách VIP mục tiêu, sổ cái sử dụng. |
| **SA09** | Payment & Settlement | Phương thức TT, hồ sơ nghĩa vụ TT, giao dịch cổng, hoàn tiền, Webhook log, Idempotency record, đối soát COD 3 lớp. |
| **SA10** | Return & Exchange | Yêu cầu đổi trả RMA, phân bổ kiện trả (`ReturnShipmentAllocation`), phiếu nhận hàng hoàn, giám định KCS (`ReturnLineAssessment`), lịch sử RMA. |
| **SA11** | Audit & Reporting | Nhật ký an ninh bất biến (`AuditLog`, `AuditLogDetail`), định nghĩa báo cáo, lịch sử chạy, snapshot chốt kỳ, phân quyền báo cáo. |

---

## 9.2. Danh mục 30 Yêu cầu Quản lý Dữ liệu Chi tiết (`DR-01` đến `DR-30`)

### 9.2.1. Identity, Customer & Shopping (`DR-01`, `DR-07` – `DR-10`)
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-01** | Account & TPT Profile | Định danh đăng nhập (`Email`/`Phone`), `PasswordHash`, `AccountType` (`CUSTOMER` XOR `EMPLOYEE`) và hồ sơ nghiệp vụ tương ứng (`CORR-04`). |
| **DR-07** | Dynamic RBAC Matrix | Danh mục `Role`, `Permission` và bảng gán quyền đa chiều `AccountRole`, `RolePermission`. |
| **DR-08** | Account Verification Log | Lịch sử yêu cầu xác thực OTP/Email kèm mã băm, trạng thái và thời điểm hết hạn `ExpiresAt`. |
| **DR-09** | Customer Address Book | Sổ địa chỉ nhận hàng cá nhân của khách kèm cờ địa chỉ mặc định duy nhất `IsDefault`. |
| **DR-10** | Persistent Shopping Cart | Giỏ hàng tập trung (`ACTIVE` / `CONVERTED` / `ABANDONED`) và các dòng `CartItem` theo SKU. |

### 9.2.2. Product Catalog (`DR-02` – `DR-06`, `DR-11` – `DR-14`)
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-02** | Primary Category & Hierarchy | Cây danh mục đa cấp (`ParentCategoryID`, `Slug`) và quy tắc đúng 1 danh mục chính (`IsPrimary = 1`) khi sản phẩm `PUBLISHED` (`BD-01`). |
| **DR-03** | Brand Master Dictionary | Danh mục thương hiệu độc lập (`BrandCode`, `BrandName`, `Slug`, `CountryOfOrigin`) phục vụ chuẩn hóa dữ liệu và Faceted Filter. |
| **DR-04** | Commercial Product & VAT | Thông tin sản phẩm thương mại (`ProductCode`, `ProductName`, `BrandID`, `VatRate` GTGT, `ProductStatus`). |
| **DR-05** | Core SKU & Pricing | Biến thể bán hàng (`SKU` duy nhất, `SellingPrice` nguyên đồng VNĐ, `CompareAtPrice`, `VariantStatus`). |
| **DR-06** | SKU Logistics Dimensions | Khối lượng (`WeightGrams`) và kích thước vật lý (`LengthCm`, `WidthCm`, `HeightCm`) của từng SKU phục vụ tính cước vận chuyển. |
| **DR-11** | Product–Category Assignment | Bảng liên kết $N:N$ giữa sản phẩm và danh mục kèm cờ `IsPrimary` và thời điểm gán. |
| **DR-12** | Product Attribute Role | Khai báo thuộc tính áp dụng cho sản phẩm kèm vai trò loại trừ `'VARIANT_DEFINING'` hoặc `'DESCRIPTIVE'` (`BD-06`). |
| **DR-13** | Global Attribute Dictionary | Từ điển `Attribute` (`AttributeCode`) và `AttributeValue` (`ValueCode`, `Value`, `DisplayValue`) dùng chung toàn hệ thống. |
| **DR-14** | Whitelist & SKU Composition | Tập giá trị `ProductAttributeValue` (Whitelist biến thể / Giá trị mô tả sản phẩm) và tổ hợp cấu thành SKU `VariantAttributeValue`. |

### 9.2.3. Warehouse, Inventory & Procurement (`DR-15` – `DR-20`, `DR-25` – `DR-27`)
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-15** | Order Warehouse Allocation | Kế hoạch phân bổ số lượng của từng `OrderLine` về các kho vật lý xuất hàng (`BD-03`). |
| **DR-16** | Timed Stock Reservation | Phiếu và dòng giữ hàng tạm thời gắn với `AllocationID` ($1:0..N$), thời hạn `ExpiresAt` và trạng thái `ACTIVE` duy nhất (`BD-04`, `CORR-10`). |
| **DR-17** | Shipment Header | Vận đơn kiện hàng xuất từ đúng 1 kho, đối tác vận chuyển `Carrier`, mã `TrackingNumber`, cước phí thực tế của hãng. |
| **DR-18** | Shipment Line & Outbound Cost | Chi tiết SKU đóng gói theo `AllocationID`, số lượng xuất và snapshot giá vốn `OutboundUnitCost` (`CORR-08`). |
| **DR-19** | Dual-Delta Inventory Ledger | Chứng từ `InventoryTransaction` và dòng chi tiết 4 delta (`OnHandDelta`, `ReservedDelta`, `QuarantinedDelta`, `DamagedDelta`), `FromStockStatus \to ToStockStatus` và `UnitCost` (`CORR-02`). |
| **DR-20** | Delivery Return Receipt | Biên bản thủ kho xác nhận thực nhận kiện hàng giao thất bại quay về kho (`DeliveryReturnReceipt` E75) trước khi cộng lại tồn (`BD-07`). |
| **DR-25** | Stock Count & Two-Step Transfer | Phiếu kiểm kê kho (`SystemQty`, `CountedQty`, `VarianceQty`, phê duyệt độc lập) và phiếu điều chuyển kho 2 bước (`WarehouseTransfer` kèm `UnitCost`). |
| **DR-26** | Multi-Quality Inventory Balance | Số dư tức thời theo kho & SKU tách 3 phẩm chất (`SELLABLE`, `QUARANTINED`, `DAMAGED`), `ReservedQty` và giá vốn bình quân `CurrentAverageCost` (`CORR-02`, `CORR-08`). |
| **DR-27** | Procurement & WAC Variance | Hồ sơ `Supplier`, `PurchaseOrder`, kiểm nhận `GoodsReceiptLine` (`AcceptedQty`/`RejectedQty`) và xuất trả `SupplierReturn` kèm tài khoản chênh lệch giá vốn `CostVarianceAccountCode`. |

### 9.2.4. Sales, Promotion, Payment, Return & Audit (`DR-21` – `DR-24`, `DR-28` – `DR-30`)
| ID | Data Requirement | Dữ liệu cần quản lý |
| :--- | :--- | :--- |
| **DR-21** | Promotion & Voucher Catalog | Chương trình ưu đãi phiên bản hóa (`Promotion`), điều kiện (`PromotionCondition`), danh sách SKU (`PromotionProduct`), mã `Voucher` và khách VIP mục tiêu. |
| **DR-22** | Promotion Scope & Stacking | Phạm vi 3 tầng độc lập (`PRODUCT`, `ORDER`, `SHIPPING`), cờ kết hợp `IsStackable` và độ ưu tiên `Priority` (`BD-05`). |
| **DR-23** | LRM Discount & Order Snapshot | Giá đã gồm GTGT, giảm giá dòng, giảm giá voucher LRM (`AllocatedOrderDiscount`), giảm phí vận chuyển (`ShippingDiscountAmount`), hủy thiếu hàng (`CancelledQty`), địa chỉ & thông tin hóa đơn B2B `OrderAddress` và JSON `VariantAttributesSnapshot` (`CORR-03`, `CORR-09`). |
| **DR-24** | Promotion Usage Ledger | Nhật ký tiêu thụ/hoàn trả lượt ưu đãi `PromotionUsage` (`PENDING`, `CONSUMED`, `REVERSED`). |
| **DR-28** | Payment, Webhook, Idempotency & 3-Layer COD | Hồ sơ `Payment` ($1:1$), `PaymentTransaction`, `Refund`, `RefundTransaction` (`OriginalPaymentTransactionID`), `PaymentWebhookLog` (E71), `IdempotencyRecord` (E74) và đối soát COD 3 lớp `CODCollection` (E77) $\to$ `CODRemittance` (E78) $\to$ `CODReconciliation` (E79) (`CORR-05`, `CORR-07`, `CORR-08`). |
| **DR-29** | RMA, Parcel Trace & Versioned KCS Assessment | Hồ sơ `ReturnRequest`, cầu nối kiện giao `ReturnShipmentAllocation` (E76), phiếu nhận cách ly `ReturnReceipt`, biên bản giám định phiên bản hóa `ReturnLineAssessment` (E72) và đơn đổi bù `EXCHANGE_REPLACEMENT` (`CORR-01`, `CORR-10`). |
| **DR-30** | WORM Audit Trail & Financial Period Snapshots | Nhật ký an ninh chỉ ghi `AuditLog`/`AuditLogDetail`, danh mục `ReportDefinition`, lịch sử chạy `ReportExecution`, bản chụp chốt kỳ `ReportSnapshot` và ma trận phân quyền `ReportPermission` (`SA11`). |

---

## 9.3. Yêu cầu Bảo toàn Dữ liệu Lịch sử (`HDR-01` – `HDR-12`)

| ID | Dữ liệu lịch sử | Cơ chế bảo toàn bắt buộc |
| :--- | :--- | :--- |
| **HDR-01** | Giá bán & Thuế trong đơn | Snapshot bất biến `UnitPrice`, `VatRate`, `TaxAmount` trên `OrderLine`. |
| **HDR-02** | Định danh biến thể trong đơn | Snapshot `SKU`, `ProductName`, `VariantName` và JSON Array `VariantAttributesSnapshot` (`CORR-09`). |
| **HDR-03** | Địa chỉ nhận hàng | Snapshot độc lập sang bảng `OrderAddress` ($1:1$ với `SalesOrder`). |
| **HDR-04** | Giá vốn hàng mua & xuất kho | Snapshot `GoodsReceiptLine.UnitCost`, `ShipmentLine.OutboundUnitCost`, `WarehouseTransferLine.UnitCost`. |
| **HDR-05** | Sổ cái biến động tồn kho | Ghi sổ Append-Only trên `InventoryTransaction` và `InventoryTransactionLine`. |
| **HDR-06** | Lịch sử giữ hàng theo phân bổ | Lưu lịch sử $1:0..N$ trên `StockReservationLine` (`ACTIVE`, `RELEASED`, `EXPIRED`, `CONSUMED`). |
| **HDR-07** | Lịch sử trạng thái chứng từ | Bảng lịch sử chuyên biệt: `OrderStatusHistory`, `ShipmentStatusHistory`, `ReturnStatusHistory`. |
| **HDR-08** | Lịch sử thanh toán & Webhook | Nhật ký bất biến `PaymentTransaction`, `RefundTransaction`, `PaymentWebhookLog`, `IdempotencyRecord`. |
| **HDR-09** | Đối soát công nợ thu hộ COD | Lưu vết đầy đủ 3 bước `CODCollection` $\to$ `CODRemittance` $\to$ `CODReconciliation`. |
| **HDR-10** | Phiên bản khuyến mãi & sử dụng | Phiên bản hóa `Promotion.VersionNumber` và sổ cái `PromotionUsage`. |
| **HDR-11** | Phiên bản giám định KCS hàng trả | Phiên bản hóa `ReturnLineAssessment` (`AssessmentVersion`, `IsCurrent`) cấm ghi đè kết luận cũ. |
| **HDR-12** | Kiểm toán an ninh & Báo cáo kỳ | Nhật ký WORM `AuditLog`/`AuditLogDetail` và bản chụp chốt kỳ `ReportSnapshot`. |

---

## 9.4. Baseline Ràng buộc Kỹ thuật Chuyển giao M2–M4

| Đối tượng | Ràng buộc Cốt lõi | Cơ chế Cưỡng chế trên SQL Server |
| :--- | :--- | :--- |
| **ProductCategory** | Tối đa 1 dòng `IsPrimary = 1` mỗi `ProductID`; bắt buộc đúng 1 khi `PUBLISHED`. | `UX_ProductCategory_Primary` + Boundary SP/Trigger (`BD-01`). |
| **AttributeValue & E08/E09** | Cấm gán chéo giá trị khác loại thuộc tính. | `UQ (AttributeID, AttributeValueID)` + Composite FK `(AttributeID, AttributeValueID)`. |
| **InventoryBalance** | Tồn kho 3 phẩm chất không âm và tổng không vượt `OnHandQty`. | 5 Declarative `CHECK` Constraints (`CORR-02`). |
| **StockReservationLine** | Tối đa 1 dòng giữ hàng `ACTIVE` trên mỗi `AllocationID`. | `UX_StockReservationLine_ActiveAlloc` (`CORR-10`). |
| **SalesOrder / Account** | Quan hệ Mandatory $1:1$ và TPT Disjoint XOR. | Boundary Stored Procedures + `REVOKE DML` trực tiếp (`CORR-04`). |
| **PaymentWebhookLog / E74** | Chống xử lý trùng lặp callback và request đặt đơn/thanh toán. | `UX_PaymentWebhookLog_ProviderEvent` & `UQ_IdempotencyRecord_ScopeKey` (`CORR-05`). |
| **ReturnLineAssessment** | Bảo toàn tách lô $1:N$ và phiên bản hiện hành duy nhất mỗi lô. | `UX_ReturnLineAssessment_Current ON (ReturnReceiptLineID, AssessmentGroupNo) WHERE IsCurrent = 1`. |
