# 10. TRACEABILITY MATRIX, EXIT CRITERIA & BUSINESS DECISION REGISTER

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

---

## 10.1. Ma trận Truy vết Yêu cầu (Requirements Traceability Matrix)

Ma trận này chứng minh tính liên kết chặt chẽ và nhất quán giữa các miền nghiệp vụ, Use Cases, Business Rules, Functional Requirements và Data Requirements:

| Business Area | Use Case | Business Rules | Functional Requirements | Data Requirements |
| :--- | :--- | :--- | :--- | :--- |
| **Product** | UC Product Management | `BR-PR01` $\rightarrow$ `BR-PR09` | `FR-PRO-01` $\rightarrow$ `FR-PRO-06` | `DA-02`, `HDR-01` |
| **Inventory** | `UC-02`, `UC-03`, `UC-08` | `BR-IN01` $\rightarrow$ `BR-IN12` | `FR-INV-01` $\rightarrow$ `FR-INV-06` | `DA-03`, `DA-04`, `HDR-05` |
| **Procurement** | `UC-02` | `BR-PO01` $\rightarrow$ `BR-PO06` | `FR-PO-01` $\rightarrow$ `FR-PO-03` | `DA-05` |
| **Customer & Cart** | UC Customer & Cart Management | `BR-CU01` $\rightarrow$ `BR-CU06` | `FR-CUS-01` $\rightarrow$ `FR-CUS-02`, `FR-CART-01` $\rightarrow$ `FR-CART-04` | `DA-06`, `DA-07` |
| **Order** | `UC-01`, `UC-05` | `BR-OR01` $\rightarrow$ `BR-OR10` | `FR-ORD-01` $\rightarrow$ `FR-ORD-06` | `DA-08`, `HDR-02` $\rightarrow$ `HDR-04`, `HDR-06` |
| **Promotion** | `UC-06` | `BR-PM01` $\rightarrow$ `BR-PM09` | `FR-PROM-01` $\rightarrow$ `FR-PROM-04` | `DA-09`, `HDR-09` |
| **Payment** | `UC-04` | `BR-PA01` $\rightarrow$ `BR-PA07` | `FR-PAY-01` $\rightarrow$ `FR-PAY-05` | `DA-10`, `HDR-07`, `HDR-08` |
| **Shipping** | `UC-05` | `BR-SH01` $\rightarrow$ `BR-SH05` | `FR-SHIP-01` $\rightarrow$ `FR-SHIP-03` | `DA-11`, `HDR-10` |
| **Return** | `UC-07` | `BR-RT01` $\rightarrow$ `BR-RT06` | `FR-RET-01` $\rightarrow$ `FR-RET-04` | `DA-12`, `HDR-11` |
| **Audit & IAM** | Administrative Activities | `BR-ID01` $\rightarrow$ `BR-ID05` | `FR-AUTH-01` $\rightarrow$ `FR-AUTH-04`, `FR-AUD-01` | `DA-01`, `DA-13`, `HDR-12` |

---

## 10.2. M1 Exit Criteria – Điều kiện Nghiệm thu Hoàn thành Milestone M1

| ID | Tiêu chí nghiệm thu | Trạng thái |
| :--- | :--- | :--- |
| **EC-01** | Project Charter xác định rõ mục tiêu và phạm vi | **Đạt** |
| **EC-02** | Xác định Stakeholder và mối quan tâm | **Đạt** |
| **EC-03** | Có Use Case Diagram tổng quan | **Đạt** |
| **EC-04** | Đặc tả các Use Case cốt lõi | **Đạt** |
| **EC-05** | Có Business Process tổng quan | **Đạt** |
| **EC-06** | Business Rules có mã định danh | **Đạt** |
| **EC-07** | Có Functional Requirements | **Đạt** |
| **EC-08** | Có Non-functional Requirements | **Đạt** |
| **EC-09** | Có Data Requirements | **Đạt** |
| **EC-10** | Có Traceability Matrix | **Đạt** |

> **Kết luận nghiệm thu M1:** Đạt $10/10$ tiêu chí ở mức Baseline v1.0. Đủ cơ sở tiến hành chốt các quyết định kiến trúc nghiệp vụ (Business Decision Register) và chuyển sang **Milestone M2 – Conceptual Database Design**.

---

## 10.3. M1 – Business Decision Register (Cần chốt trước khi vẽ ERD ở M2)

Bảy quyết định nghiệp vụ dưới đây tác động trực tiếp đến Cardinality, Khóa, Thực thể kết hợp (Associative Entities) và Ràng buộc toàn vẹn ở M2:

| Mã QĐ | Câu hỏi quyết định nghiệp vụ | Phương án A (Đề xuất mặc định cho Medium Enterprise) | Phương án B (Đơn giản hóa) | Trạng thái / Lựa chọn |
| :--- | :--- | :--- | :--- | :--- |
| **BD-01** | Một sản phẩm có thể thuộc nhiều danh mục không? | **Có, hỗ trợ nhiều danh mục** (Quan hệ $N-N$, phục vụ cả phân loại gốc và danh mục chiến dịch/bộ sưu tập) | Không, mỗi sản phẩm chỉ thuộc duy nhất một danh mục ($1-N$) | Chờ xác nhận |
| **BD-02** | Khách hàng có được đặt hàng không cần tài khoản không? | **Bắt buộc đăng nhập** (Đảm bảo truy vết lịch sử đơn hàng, voucher theo khách hàng và đổi trả chặt chẽ) | Cho phép Guest Checkout (`CustomerID` nullable hoặc tạo hồ sơ khách vãng lai) | Chờ xác nhận |
| **BD-03** | Một đơn hàng có thể được giao từ nhiều kho không? | **Có, hỗ trợ chia đơn giao nhiều kho (Split Shipment)** (Một `Order` có nhiều `Shipments`, mỗi `Shipment` xuất từ 1 `Warehouse`) | Không, một đơn hàng chỉ được xuất từ duy nhất một kho | Chờ xác nhận |
| **BD-04** | Chính sách giữ hàng khi đặt hàng? | **Giữ ngay khi tạo đơn, có thời hạn (`ExpiresAt`)** (Chống bán lố ngay tại thời điểm Checkout) | Chỉ giữ hàng khi nhân viên bán hàng xác nhận đơn | Chờ xác nhận |
| **BD-05** | Chính sách khuyến mãi trên một đơn hàng? | **Cho phép kết hợp nếu thỏa quy tắc ưu tiên (`Priority` & `IsStackable`)** (VD: Khuyến mãi giảm giá trực tiếp trên sản phẩm + 1 Voucher đơn hàng) | Mỗi đơn hàng chỉ được áp dụng duy nhất một chương trình khuyến mãi | Chờ xác nhận |
| **BD-06** | Quản lý thuộc tính biến thể sản phẩm? | **Mô hình thuộc tính linh hoạt theo sản phẩm / danh mục** (Hỗ trợ cả Thời trang: Size/Color và Gia dụng: Chất liệu/Công suất/Kích thước) | Chỉ hỗ trợ các cột thuộc tính cố định (`Size`, `Color`) trên bảng Variant | Chờ xác nhận |
| **BD-07** | Chính sách tồn kho khi bàn giao vận chuyển và khi giao thất bại? | **Giữ hàng (`ReservedQty`) cho đến khi xuất kho/giao thành công hoặc hủy** (Tách bạch rõ trạng thái đang giữ, đang giao và hoàn nhập kho khi giao thất bại) | Trừ đứt tồn kho ngay khi tạo đơn | Chờ xác nhận |