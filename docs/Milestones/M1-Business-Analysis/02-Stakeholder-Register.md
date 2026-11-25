# 02. STAKEHOLDER REGISTER – DANH SÁCH CÁC BÊN LIÊN QUAN (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT (Đầy đủ vai trò Phân tách Trách nhiệm SoD cho RMA, KCS & Đối soát COD)

---

## 2.1. Danh mục 13 Bên Liên Quan Chuẩn hóa

| ID | Stakeholder | Phân loại Tác nhân | Vai trò Nghiệp vụ | Mối quan tâm Chính trong Hệ thống |
| :--- | :--- | :---: | :--- | :--- |
| **ST-01** | **Business Owner / Executive** | Internal (`EMPLOYEE`) | Chủ doanh nghiệp / Ban giám đốc | Doanh thu ròng, biên lợi nhuận theo giá vốn WAC, vòng quay tồn kho, báo cáo chốt kỳ. |
| **ST-02** | **System Administrator** | Internal (`EMPLOYEE`) | Quản trị viên hệ thống | Quản lý tài khoản TPT, phân quyền RBAC (`Role`/`Permission`), cấu hình hệ thống, `AuditLog`. |
| **ST-03** | **Marketing / Merchandising Staff** | Internal (`EMPLOYEE`) | Nhân viên ngành hàng & Marketing | Quản lý `Brand`, `Category`, `Product`/`SKU`, khuyến mãi 3 tầng, Flash Sale & hiệu quả `Voucher`. |
| **ST-04** | **Sales Operations Staff** | Internal (`EMPLOYEE`) | Nhân viên vận hành đơn hàng | Theo dõi đơn hàng, xác nhận đơn COD, điều phối phân bổ kho và xử lý hủy thiếu hàng (`CancelledQty`). |
| **ST-05** | **Customer Support (`CSKH`)** | Internal (`EMPLOYEE`) | Chăm sóc khách hàng & Xử lý khiếu nại | Tiếp nhận/tạo hộ yêu cầu đổi trả (`ReturnRequest`), xét duyệt điều kiện RMA và chỉ định kho nhận. |
| **ST-06** | **Warehouse Staff / Manager** | Internal (`EMPLOYEE`) | Thủ kho & Quản lý kho vận | Lập/nhận PO (`GoodsReceipt`), xuất kiện (`Shipment`), ký hoàn giao thất bại (`DeliveryReturnReceipt`), chuyển kho, kiểm kê kho. |
| **ST-07** | **QC / KCS Inspector** | Internal (`EMPLOYEE`) | Nhân viên kiểm định chất lượng | Kiểm tra ngoại quan hàng nhập/hàng trả, lập biên bản thẩm định phiên bản hóa `ReturnLineAssessment` (`E72`). |
| **ST-08** | **Finance / Accountant** | Internal (`EMPLOYEE`) | Kế toán thanh toán & Đối soát công nợ | Phê duyệt & chi hoàn tiền (`Refund` / `RefundTransaction`), đối soát COD 3 lớp (`E77`–`E79`), khóa kỳ `ReportSnapshot`. |
| **ST-09** | **Customer** | External (`CUSTOMER`) | Khách hàng mua lẻ B2C / mua sỉ có hóa đơn | Duyệt/lọc sản phẩm, giỏ hàng, đặt đơn, thanh toán an toàn, theo dõi nhiều kiện giao và đổi trả minh bạch. |
| **ST-10** | **Supplier** | External Partner | Nhà cung cấp hàng hóa | Tiếp nhận đơn mua hàng (`PurchaseOrder`), giao hàng theo đợt và đối soát hàng xuất trả (`SupplierReturn`). |
| **ST-11** | **Shipping Provider (`Carrier`)** | External System / Partner | Đối tác vận chuyển (GHN, GHTK, VTP...) | Nhận kiện hàng, cập nhật Webhook hành trình, thu hộ tiền mặt COD và chuyển trả bảng kê đối soát. |
| **ST-12** | **Payment Provider** | External System | Cổng thanh toán điện tử (VNPAY, MoMo...) | Xử lý giao dịch thu tiền trực tuyến, gửi callback Webhook có chữ ký số và thực thi lệnh hoàn tiền. |
| **ST-13** | **Development & DB Team** | Engineering Team | Nhóm kỹ sư CSDL, Backend, Frontend & QA | Toàn vẹn lược đồ 79 bảng, hiệu năng SQL Server 2022, chống Deadlock/Overselling, bảo mật và CI/CD. |

---

## 2.2. Ma trận Mối quan tâm, Quyền hạn & Phân tách Trách nhiệm (Interest & SoD Matrix)

| Stakeholder | Thực thể / Dữ liệu Trọng tâm Tác động | Nguyên tắc Phân tách Trách nhiệm (SoD) | Mức độ Ảnh hưởng |
| :--- | :--- | :--- | :---: |
| **Business Owner** | `ReportDefinition`, `ReportSnapshot`, Dashboard KPI | Chỉ xem báo cáo quản trị; không sửa trực tiếp giao dịch vận hành. | Rất cao |
| **System Administrator** | `Account`, `Employee`, `Role`, `Permission`, `AuditLog` | Cấp quyền RBAC; không được xóa/sửa `AuditLog` (`Append-Only`). | Rất cao |
| **Marketing Staff** | `Brand`, `Category`, `Product`, `ProductVariant`, `Promotion`, `Voucher` | Không được sửa chương trình khuyến mãi đã có `PromotionUsage` (phải tạo `VersionNumber + 1`). | Cao |
| **Sales Staff** | `SalesOrder`, `OrderLine`, `OrderWarehouseAllocation` | Xác nhận/điều phối đơn; **không** tự ý duyệt chi hoàn tiền `Refund`. | Cao |
| **Customer Support (`CSKH`)** | `ReturnRequest`, `ReturnRequestLine`, `ReturnShipmentAllocation` | Duyệt tiếp nhận RMA ban đầu; **không** kiêm nhiệm nhập kho hay giải ngân hoàn tiền. | Cao |
| **Warehouse Staff / Manager** | `GoodsReceipt`, `Shipment`, `DeliveryReturnReceipt`, `ReturnReceipt`, `StockCount` | Nhân viên đếm kho (`CreatedBy`) **không** được tự phê duyệt phiếu kiểm kê (`ApprovedBy <> CreatedBy`). | Cao |
| **QC / KCS Inspector** | `GoodsReceiptLine`, `ReturnLineAssessment` (`E72`) | Độc lập thẩm định `FaultParty` & `InventoryDisposition`; tách biệt với bộ phận giải ngân tài chính. | Cao |
| **Finance / Accountant** | `Payment`, `Refund`, `RefundTransaction`, `CODRemittance`, `CODReconciliation`, `ReportSnapshot` | Phê duyệt & chi hoàn tiền dựa trên kết luận `E72` hợp lệ; khớp bảng kê COD với thực thu ngân hàng. | Rất cao |
| **Customer** | `Customer`, `CustomerAddress`, `ShoppingCart`, `SalesOrder`, `ReturnRequest` | Cách ly hoàn toàn khỏi `Employee` qua TPT XOR; chỉ truy cập dữ liệu đúng `CustomerID` của mình (Anti-IDOR). | Cao |
| **Supplier / Carrier / Gateway** | `PurchaseOrder`, `ShipmentStatusHistory`, `PaymentWebhookLog` | Không có tài khoản `Account` nội bộ; chỉ tương tác qua chứng từ nghiệp vụ hoặc Webhook API xác thực chữ ký. | Trung bình |
| **Development & DB Team** | Toàn bộ 79 bảng, 5 Boundary SPs, Flyway, Docker | Ứng dụng runtime chạy dưới `retailcore_app` (đã `REVOKE DML` trực tiếp trên bảng nhạy cảm). | Rất cao |
