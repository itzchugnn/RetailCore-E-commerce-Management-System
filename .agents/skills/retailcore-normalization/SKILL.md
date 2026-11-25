---
name: retailcore-normalization
description: >-
  Use when performing formal relational normalization analysis (1NF, 2NF, 3NF, BCNF) on the RetailCore database schema (Milestone 3 / M3.02).
  Guides computing functional dependencies, attribute closures, candidate keys, lossless-join proofs, dependency preservation, and technical trade-offs for controlled normalization.
---

# RetailCore Mathematical Normalization Analysis Skill

Bộ quy chuẩn hướng dẫn thực hiện phân tích chuẩn hóa toán học cho hệ thống **RetailCore Enterprise E-Commerce Management System** (Milestone 3 / M3.02).

---

## 1. NGUYÊN TẮC CỐT LÕI (CONTROLLED NORMALIZATION POLICY)

Dự án RetailCore yêu cầu phân tích chuẩn hóa thực chất bằng lý thuyết cơ sở dữ liệu quan hệ trên toàn bộ **79 bảng quan hệ (`E01` – `E79`)**, **tuyệt đối không tuyên bố cảm tính**:
1. **100% bảng trong hệ thống phải đạt chuẩn Dạng chuẩn 3 (3NF)** (ngoại trừ các thuộc tính dẫn xuất/khóa hình thoi có kiểm soát được liệt kê tường minh và bảo vệ nguyên tử dưới đây).
2. **Đánh giá BCNF & Ba Nhóm Đánh đổi Kỹ thuật có Kiểm soát (Controlled Normalization Policy):**
   * **Nhóm A – Bất biến Hợp đồng theo Thời gian (Temporal Snapshots):** Các thuộc tính snapshot lịch sử (`OrderLine.UnitPrice`, `VatRate`, `TaxAmount`, `OrderAddress`, `ShipmentLine.OutboundUnitCost`, `WarehouseTransferLine.UnitCost`) phụ thuộc hàm vào khóa của chứng từ tại thời điểm giao dịch chứ không phụ thuộc vào thực thể danh mục hiện hành, đạt chuẩn 3NF/BCNF theo ngữ nghĩa thời gian.
   * **Nhóm B – Bảo toàn Phụ thuộc Hàm & Khóa Ngoại Hình Thoi (Dependency Preservation & Diamond-FK):** Bảng `VariantAttributeValue` giữ 3NF để bảo toàn cả hai phụ thuộc hàm `(VariantID, AttributeID) -> AttributeValueID` và `AttributeValueID -> AttributeID` mà không cần JOIN; bảng `ProductAttributeValue` giữ siêu khóa `(ProductID, AttributeID, AttributeValueID)` để khai báo trực tiếp hai khóa ngoại phức hợp hình thoi ngay tại tầng Declarative DDL (khóa ứng viên tối thiểu là `(ProductID, AttributeValueID)`).
   * **Nhóm C – Thuộc tính Đệm & Đồng bộ Trạng thái có Kiểm soát (Controlled Cache Attributes):** Các cột `Category.CategoryLevel`, `ProductVariant.IsActive` (ràng buộc `CK_*_StatusActiveConsistency`), `OrderLine.CancelledQty`/`RefundedQty`/`AccumulatedRefundedAmount`, `Promotion.PendingCount`/`UsedCount`, `Voucher.PendingCount`/`UsedCount` và `PromotionProduct.FlashSaleSoldCount` bắt buộc có văn bản **Giải trình Kỹ thuật (Technical Trade-off)** tại `M3.02` kèm cơ chế cưỡng chế nguyên tử bằng `CHECK` Constraint / Boundary Stored Procedure tại `M3.04`.

---

## 2. QUY TRÌNH 6 BƯỚC PHÂN TÍCH CHUẨN HÓA BẮT BUỘC

Với từng bảng quan hệ (hoặc cụm bảng quan hệ then chốt), Agent phải thực thi và ghi nhận đầy đủ 6 bước:

### Bước 1: Xác định Tập Phụ thuộc Hàm đầy đủ ($\mathcal{F}$)
* Liệt kê tất cả các phụ thuộc hàm có ý nghĩa nghiệp vụ giữa các tập thuộc tính: $X \to Y$.
* Phân biệt rõ phụ thuộc hàm tầm thường (Trivial) và không tầm thường (Non-trivial).
* Xác định rõ ngữ nghĩa thời gian: 
  * $\text{VariantID} \to \text{CurrentSellingPrice}$ (Giá niêm yết hiện hành trong Catalog).
  * $(\text{OrderID}, \text{VariantID}) \to \text{TransactionUnitPrice}$ (Giá giao dịch bất biến tại thời điểm đặt đơn).

### Bước 2: Tìm tất cả Khóa ứng viên (Candidate Keys) qua Bao đóng Thuộc tính ($X^+$)
* Áp dụng thuật toán tính bao đóng thuộc tính dưới tập phụ thuộc hàm $\mathcal{F}$:
  1. Khởi tạo $X^{(0)} = X$.
  2. Lặp: Với mỗi $Y \to Z \in \mathcal{F}$, nếu $Y \subseteq X^{(i)}$ thì $X^{(i+1)} = X^{(i)} \cup Z$.
  3. Dừng khi $X^{(i+1)} = X^{(i)}$; $X^+ = X^{(i)}$.
* Tập thuộc tính $K$ là Siêu khóa (Superkey) nếu $K^+ = R$.
* Tập $K$ là Khóa ứng viên (Candidate Key) nếu $K$ là siêu khóa tối thiểu (không tồn tại $K' \subset K$ mà $K'^+ = R$).

### Bước 3: Chọn Khóa chính (Primary Key) & Phân loại Thuộc tính
* Chọn 1 Khóa ứng viên làm Khóa chính ($PK$): ưu tiên Surrogate Key `BIGINT / INT IDENTITY` để tối ưu kích thước B-Tree Index.
* Phân loại:
  * **Thuộc tính khóa (Prime Attributes):** Thuộc tính nằm trong ít nhất một Khóa ứng viên.
  * **Thuộc tính không khóa (Non-prime Attributes):** Thuộc tính không nằm trong bất kỳ Khóa ứng viên nào.

### Bước 4: Kiểm tra và Đánh giá Dạng chuẩn (1NF $\to$ 2NF $\to$ 3NF $\to$ BCNF)
1. **Dạng chuẩn 1 (1NF):**
   * Mọi thuộc tính đều mang giá trị nguyên tố (Atomic values), không chứa tập hợp (Repeating groups).
   * *Lưu ý về JSON:* Cột `VariantAttributesSnapshot` lưu JSON Array là dữ liệu bán cấu trúc có mục đích kiểm toán; ở tầng quan hệ được coi là chuỗi đơn nhất (Single atomic string), hợp chuẩn 1NF.
2. **Dạng chuẩn 2 (2NF):**
   * Đạt 1NF và không tồn tại phụ thuộc hàm bộ phận: Mọi thuộc tính không khóa phải phụ thuộc hàm đầy đủ vào mỗi khóa ứng viên (không phụ thuộc vào tập con thực sự của bất kỳ khóa ứng viên nào).
3. **Dạng chuẩn 3 (3NF):**
   * Đạt 2NF và không có phụ thuộc bắc cầu giữa các thuộc tính không khóa: Với mọi phụ thuộc hàm không tầm thường $X \to Y \in \mathcal{F}$, hoặc $X$ là Siêu khóa, hoặc $Y$ là Thuộc tính khóa (Prime attribute).
4. **Dạng chuẩn Boyce-Codd (BCNF):**
   * Với mọi phụ thuộc hàm không tầm thường $X \to Y \in \mathcal{F}$, $X$ bắt buộc phải là Siêu khóa (Superkey).

### Bước 5: Chứng minh Phép Phân rã Không Mất Mát Thông Tin (Lossless-Join Decomposition)
* Khi một lược đồ $R$ được phân rã thành $(R_1, R_2)$, chứng minh phép phân rã là Lossless Join nếu và chỉ nếu:
  $$(R_1 \cap R_2) \to R_1 \quad \text{hoặc} \quad (R_1 \cap R_2) \to R_2 \in \mathcal{F}^+$$
  *(Nghĩa là thuộc tính chung giữa hai bảng phải là Siêu khóa của ít nhất một trong hai bảng).*

### Bước 6: Đánh giá Tính Bảo toàn Phụ thuộc Hàm (Dependency Preservation)
* Kiểm tra xem $(\mathcal{F}_1 \cup \mathcal{F}_2 \cup \dots \cup \mathcal{F}_k)^+ = \mathcal{F}^+$ hay không.
* Nếu phép phân rã lên BCNF làm mất phụ thuộc hàm quan trọng buộc phải kiểm tra bằng JOIN liên bảng tốn kém, RetailCore lựa chọn dừng ở 3NF để bảo toàn phụ thuộc hàm.

---

## 3. DANH MỤC TÀI LIỆU CẦN ĐÓNG GÓI CHO M3.02

Tài liệu kết quả được lưu trữ tại `docs/Milestones/M3-Logical-Design/02-Normalization-Analysis/`:
1. `01-Normalization-Methodology.md`: Khung lý thuyết và công thức toán học.
2. `02-Functional-Dependencies-Catalog.md`: Danh mục tập FDs của toàn bộ các bảng trong 11 Subject Areas.
3. `03-Candidate-Keys-and-Normal-Forms.md`: Bảng bao đóng $X^+$, Candidate Keys và đánh giá 3NF/BCNF.
4. `04-Controlled-Normalization-Tradeoffs.md`: Văn bản giải trình kỹ thuật cho các trường hợp giữ 3NF Snapshot.
5. `05-Lossless-Join-and-Preservation-Proofs.md`: Chứng minh toán học Lossless-Join và Dependency Preservation.

