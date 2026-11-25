# 01. PROJECT CHARTER – RETAILCORE E-COMMERCE MANAGEMENT SYSTEM (v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Canonical SSOT Baseline
* **Phạm vi:** Medium-to-Enterprise Retail & E-Commerce Information System
* **Đầu ra tiếp theo:** M2 – Conceptual Database Design (**79 Thực thể `E01` – `E79`**)

> **Nguyên tắc trọng tâm:**
> Hệ thống được xây dựng theo triết lý **Database-First / Database-Centric**, đặt Microsoft SQL Server 2022+ làm nguồn sự thật duy nhất (Single Source of Truth) bảo vệ tính toàn vẹn toán học, kiểm soát đồng thời cao, định giá tồn kho bình quân gia quyền liên tục (Moving WAC) và lưu vết kiểm toán bất biến.

---

## 1.1. Thông tin Dự án

| Thuộc tính | Nội dung Chuẩn hóa |
| :--- | :--- |
| **Project Name** | RetailCore – E-commerce Management System |
| **Project Type** | Multi-Warehouse B2C Retail & E-Commerce Information System |
| **Domain** | Retail / Omnichannel-Ready B2C E-Commerce (Thị trường Việt Nam) |
| **Database Platform** | Microsoft SQL Server 2022+ (`Developer Edition` cho Dev/CI/Test; `Enterprise Edition` cho Production HA) |
| **Schema Migration** | Flyway (kết hợp mô hình Dual Database Login: `flyway_migrator` & `retailcore_app`) |
| **Backend** | Java 21+ / Spring Boot 3.x (Hybrid Spring Data JPA `@Immutable` + Spring `JdbcClient`) |
| **Frontend** | React + TypeScript + Tailwind CSS (Customer Storefront & Back-Office Admin Portal) |
| **Architecture** | Modular Monolith, Layered Database-Centric Architecture |
| **Target Users** | Customer, Sales Staff, Customer Support (`CSKH`), Warehouse Staff, QC/KCS Inspector, Marketing Staff, Finance/Accountant, System Administrator |

---

## 1.2. Bối cảnh Dự án

Doanh nghiệp bán lẻ vận hành nhiều kho hàng vật lý cần quản trị tập trung và nhất quán toàn bộ chuỗi giá trị khép kín:
* Danh mục thương hiệu (`Brand`), cây phân loại đa cấp (`Category`), sản phẩm thương mại (`Product`) và biến thể bán hàng (`ProductVariant` – SKU) kèm thuộc tính động hai lớp.
* Mạng lưới nhiều kho vật lý, kiểm soát số dư tồn kho theo 3 phẩm chất (`SELLABLE`, `QUARANTINED`, `DAMAGED`), giữ hàng có thời hạn (`StockReservation`) và tính giá vốn bình quân gia quyền liên tục (`Moving WAC`).
* Mua hàng nhà cung cấp (`PurchaseOrder`), kiểm nhận QC nhiều đợt (`GoodsReceipt`) và xuất trả nhà cung cấp (`SupplierReturn`).
* Giỏ hàng tập trung, đặt hàng nguyên tử chống gửi lặp (`IdempotencyRecord`), định giá bán lẻ đã gồm thuế GTGT (VAT-Inclusive) kèm snapshot thông tin xuất hóa đơn B2B.
* Khuyến mãi phân tầng 3 lớp (`PRODUCT`, `ORDER` Voucher phân bổ LRM, và `SHIPPING` Freeship Voucher).
* Phân bổ giao hàng đa kho (`Split Shipment`), đối soát thu hộ tiền mặt COD 3 lớp theo kiện, xử lý kiện giao thất bại hoàn kho (`DeliveryReturnReceipt`).
* Đổi trả hàng hóa (RMA) truy vết theo kiện giao, giám định chất lượng KCS (`ReturnLineAssessment`) độc lập với quyền hoàn tiền/đổi bù và nhật ký kiểm toán WORM (`AuditLog`).

---

## 1.3. Mục tiêu Dự án (Project Objectives)

| ID | Mục tiêu Cốt lõi |
| :--- | :--- |
| **OBJ-01** | Quản lý chuẩn hóa thương hiệu, danh mục đa tầng, sản phẩm và biến thể SKU với mô hình thuộc tính động có kiểm soát Whitelist. |
| **OBJ-02** | Quản lý tồn kho tập trung đa kho theo 3 trạng thái phẩm chất vật lý, khóa giữ tồn kho All-or-Nothing chống bán âm kho (0% Overselling) trong Flash Sale. |
| **OBJ-03** | Vận hành trọn vẹn vòng đời đơn hàng với cơ chế Snapshot bất biến (giá, thuế GTGT, thuộc tính JSON, địa chỉ giao & thông tin hóa đơn) và 4 máy trạng thái độc lập. |
| **OBJ-04** | Quản lý khuyến mãi 3 tầng (`PRODUCT`, `ORDER`, `SHIPPING`), phân bổ chiết khấu nguyên số VNĐ bằng thuật toán LRM và kiểm soát hạn mức đồng thời cao. |
| **OBJ-05** | Quản lý thanh toán trực tuyến chống replay Webhook, đối soát công nợ COD 3 lớp và xử lý RMA/hoàn tiền lũy tiến $\Delta F$ không lệch 1 đồng VNĐ. |
| **OBJ-06** | Cưỡng chế toàn vẹn dữ liệu trực tiếp tại tầng CSDL bằng Declarative Constraints, Filtered Unique Indexes, Boundary Stored Procedures và `REVOKE DML`. |
| **OBJ-07** | Tối ưu hóa lưu trữ vật lý SQL Server 2022 (8KB Page Sizing, Partition-Aligned Indexing, RCSI) đáp ứng tải đồng thời cao và mục tiêu `RPO < 5 phút`, `RTO < 30 phút`. |
| **OBJ-08** | Cung cấp hệ thống báo cáo vận hành trực tiếp, bản chụp chốt kỳ kế toán (`ReportSnapshot`) và nhật ký kiểm toán an ninh bất biến (`AuditLog`). |

---

## 1.4. Phạm vi Hệ thống (System Scope)

### In Scope – 14 Phân nhóm Chức năng Triển khai (`SA01` – `SA11`)
1. Xác thực người dùng (Email/Phone, OTP) và kế thừa tài khoản TPT (`Account` XOR `Customer` / `Employee`).
2. Phân quyền quản trị động dựa trên vai trò và quyền hạn (`Role` – `Permission` RBAC).
3. Quản lý thương hiệu (`Brand`), cây danh mục đệ quy (`Category`) và sản phẩm (`Product`).
4. Quản lý biến thể `SKU`, giá bán lẻ nguyên đồng VNĐ (đã gồm GTGT), quy cách vận chuyển và thuộc tính động 2 lớp.
5. Quản lý mạng lưới kho, tồn kho 3 phẩm chất, giữ hàng có thời hạn, kiểm kê kho và chuyển kho 2 bước.
6. Quản lý nhà cung cấp, đơn mua hàng (PO), phiếu nhập kho QC nhiều đợt, xuất trả NCC và định giá vốn `Moving WAC`.
7. Quản lý hồ sơ khách hàng và sổ địa chỉ nhận hàng cá nhân.
8. Quản lý giỏ hàng trực tuyến đồng bộ tập trung trên CSDL.
9. Quản lý hợp đồng đơn hàng (`SalesOrder`), hủy thiếu hàng (`CancelledQty`) và đơn đổi bù (`EXCHANGE_REPLACEMENT`).
10. Quản lý khuyến mãi 3 tầng (`PRODUCT` / Flash Sale, `ORDER` Voucher, `SHIPPING` Freeship Voucher).
11. Quản lý thanh toán trực tuyến, lưu vết Webhook bền vững, hoàn tiền truy vết giao dịch gốc và đối soát COD 3 lớp.
12. Quản lý vận chuyển đa kho (`Split Shipment`), giới hạn 3 lần giao và biên bản hoàn kho khi giao thất bại.
13. Quản lý yêu cầu đổi trả (RMA) theo kiện giao và biên bản giám định KCS phiên bản hóa (`ReturnLineAssessment`).
14. Quản lý nhật ký kiểm toán an ninh (`AuditLog`) và báo cáo quản trị / chốt kỳ số liệu (`ReportSnapshot`).

### Out of Scope – Ngoài Phạm vi Phiên bản Hiện tại
* Sàn thương mại điện tử đa nhà bán độc lập (Multi-vendor Marketplace).
* Ứng dụng di động gốc (Native iOS / Android Apps).
* Đồng bộ đơn hàng hai chiều với sàn ngoài (Shopee, Lazada, TikTok Shop).
* Hệ thống ERP tổng thể và phần mềm kế toán sổ cái tổng hợp bên ngoài *(lưu ý: hệ thống vẫn thực hiện đầy đủ tính giá vốn Moving WAC, bóc tách snapshot thuế GTGT đầu ra và đối soát công nợ thu hộ COD nội bộ)*.
* Hệ thống gợi ý sản phẩm bằng AI / Machine Learning.
* Chương trình tích điểm thành viên / đổi điểm thưởng (**Loyalty Points hoàn toàn nằm ngoài phạm vi dự án**).
* Kiến trúc vi dịch vụ phân tán (Distributed Microservices).

---

## 1.5. Giả định và Ràng buộc Nền tảng (Assumptions & Constraints)

| ID | Nội dung Giả định & Ràng buộc |
| :--- | :--- |
| **ASM-01** | Hệ thống phục vụ một pháp nhân doanh nghiệp bán lẻ duy nhất vận hành nhiều kho vật lý trên toàn quốc. |
| **ASM-02** | Cấm Guest Checkout (`BD-02`): Mọi khách hàng bắt buộc đăng nhập tài khoản xác thực trước khi đặt đơn. |
| **ASM-03** | Đơn vị tiền tệ duy nhất là **VNĐ** (`DECIMAL(19,0)` số nguyên đồng cho giao dịch thanh toán; `DECIMAL(19,4)` cho đơn giá và giá vốn WAC; cấm `FLOAT`/`MONEY`). |
| **ASM-04** | Giá bán lẻ niêm yết trên Website là giá **đã bao gồm thuế GTGT** theo chuẩn bán lẻ B2C Việt Nam; thuế GTGT được bóc tách nội bộ trên từng dòng đơn. |
| **ASM-05** | Mọi giao dịch tồn kho, đặt đơn và hoàn tiền đều gắn trực tiếp với đơn vị `ProductVariant` (SKU), kể cả sản phẩm đơn chiếc. |
| **ASM-06** | Cổng thanh toán trực tuyến (VNPAY/MoMo) và hãng vận chuyển (GHN/GHTK/ViettelPost) tích hợp qua môi trường Sandbox/Webhook chuẩn hóa. |
| **ASM-07** | Các thao tác nhạy cảm có rủi ro gian lận (duyệt PO, duyệt kiểm kê kho, giám định KCS vs duyệt chi hoàn tiền) bắt buộc tuân thủ nguyên tắc Phân tách Trách nhiệm (Segregation of Duties). |

---

## 1.6. Tiêu chí Thành công (Success Criteria)

* **Toàn vẹn mô hình `1:1`:** Ánh xạ chính xác 79 Thực thể Quan niệm (`E01` – `E79`) sang 79 Bảng Quan hệ đạt chuẩn 3NF/BCNF có kiểm soát.
* **0% Overselling & 0 Deadlock:** Vượt qua bài kiểm thử chịu tải đồng thời 200 tài khoản độc lập cùng đặt mua SKU Flash Sale hoặc tranh chấp lượt voucher cuối.
* **Chính xác Tài chính Tuyệt đối:** Phân bổ chiết khấu LRM và hoàn tiền lũy tiến nhiều đợt $\Delta F = F(a+b) - F(a)$ khớp chính xác từng đồng VNĐ, không phát sinh sai số làm tròn.
* **Bảo toàn Lịch sử & Chống Xóa Cứng:** 100% đơn hàng bảo toàn nguyên vẹn giá, thuế, thuộc tính JSON và địa chỉ tại giây phút mua; thu hồi quyền `DELETE` vật lý trên toàn bộ bảng chứng từ và nhật ký.
