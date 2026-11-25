# M3.01.02 - ENTITY-TO-RELATION MAPPING: SUBJECT AREA 01 (PRODUCT CATALOG)
## ĐẶC TẢ ÁNH XẠ THỰC THỂ SANG BẢNG QUAN HỆ: PHÂN HỆ PRODUCT CATALOG (10 BẢNG)

* **Dự án:** RetailCore Enterprise E-Commerce Management System
* **Cơ sở dữ liệu mục tiêu:** Microsoft SQL Server 2022+ (Developer / Enterprise Edition)
* **Cột mốc:** Milestone 3 (Logical Database Design) – Công đoạn M3.01
* **Phiên bản:** v2.0 – Canonical SSOT Relational Mapping Baseline
* **Phân hệ:** `Subject Area 01 – Product Catalog` (`E01` đến `E09` và `E73` $\to$ **10 Bảng Quan hệ**)
* **Kế thừa trực tiếp:**
  * [GEMINI.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/GEMINI.md) (`BD-01`, `BD-06`, `CORR-09`)
  * [01-Entity-Catalog.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M2-Conceptual-Design/01-Entity-Catalog.md) (`E01` – `E09`, `E73`)
  * [01-Mapping-Methodology.md](file:///c:/Users/ADMIN/OneDrive/Desktop/RetailCore%20%E2%80%93%20E-commerce%20Management%20System/docs/Milestones/M3-Logical-Design/01-Relational-Schema-Mapping/01-Mapping-Methodology.md)

---

## 1. TỔNG QUAN PHÂN HỆ PRODUCT CATALOG (`SA01`)

Phân hệ **Product Catalog** là hạt nhân quản lý danh mục thương hiệu, cây phân loại hàng hóa đa cấp, thông tin thương mại sản phẩm, mức thuế suất GTGT, từ điển thuộc tính động hai lớp (`VARIANT_DEFINING` / `DESCRIPTIVE`), và đơn vị biến thể bán/lưu kho trung tâm (`ProductVariant` – SKU) kèm quy cách vật lý tính cước giao vận.

### Ma trận chuyển đổi `1:1` từ 10 thực thể quan niệm sang 10 bảng quan hệ:
| Mã Thực thể | Tên Thực thể Quan niệm | Loại hình Thực thể (Conceptual) | Tên Bảng Quan hệ (`dbo.*`) | Khóa Chính (Primary Key) | Khóa Tự nhiên / Siêu khóa Ứng viên (Unique / Superkeys) |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **E73** | `Brand` | Strong / Master Entity | `Brand` | `BrandID (INT IDENTITY)` | `BrandCode (VARCHAR(50))`, `Slug (VARCHAR(120))` |
| **E02** | `Category` | Strong Entity (Cây đệ quy) | `Category` | `CategoryID (INT IDENTITY)` | `Slug (VARCHAR(120))` |
| **E01** | `Product` | Strong Entity | `Product` | `ProductID (INT IDENTITY)` | `ProductCode (VARCHAR(50))` |
| **E03** | `ProductCategory` | Associative Entity ($N:N$) | `ProductCategory` | `(ProductID, CategoryID)` | `(ProductID) WHERE IsPrimary = 1` |
| **E04** | `ProductVariant` | Core SKU Entity | `ProductVariant` | `VariantID (BIGINT IDENTITY)` | `SKU (VARCHAR(50))`, `(ProductID, VariantID)` |
| **E05** | `Attribute` | Strong Entity | `Attribute` | `AttributeID (INT IDENTITY)` | `AttributeCode (VARCHAR(50))` |
| **E06** | `AttributeValue` | Weak Entity | `AttributeValue` | `AttributeValueID (INT IDENTITY)` | `(AttributeID, ValueCode)`, `(AttributeID, Value)`, `(AttributeID, AttributeValueID)` |
| **E07** | `ProductAttribute` | Associative Entity ($N:N$) | `ProductAttribute` | `(ProductID, AttributeID)` | — |
| **E08** | `ProductAttributeValue` | Associative Entity (Dual-Role) | `ProductAttributeValue` | `(ProductID, AttributeID, AttributeValueID)` | CK tối thiểu: `(ProductID, AttributeValueID)` |
| **E09** | `VariantAttributeValue` | Associative Entity (SKU Spec) | `VariantAttributeValue` | `(VariantID, AttributeID)` | `(VariantID, AttributeValueID)` |

---

## 2. SƠ ĐỒ LƯỢC ĐỒ QUAN HỆ LOGIC (LOGICAL RELATIONAL ERD)

* **Mã nguồn Mermaid chuẩn hóa:** [SA01-Product-Catalog.md](../mermaid/SA01-Product-Catalog.md)
* **Sơ đồ ảnh Render:** `../images/SA01-Product-Catalog.png`

![SA01 Product Catalog Logical ERD](../images/SA01-Product-Catalog.png)

---

## 3. ĐẶC TẢ CHI TIẾT LƯỢC ĐỒ 10 BẢNG QUAN HỆ

### 3.1. Bảng `dbo.Brand` (`E73` - Danh mục Thương hiệu Chuẩn hóa)
* **Ý nghĩa:** Quản lý tập trung danh mục thương hiệu/nhãn hàng sản xuất, triệt tiêu hoàn toàn dị thường nhập liệu chuỗi tự do và cung cấp khóa lọc chuẩn cho Faceted Filter trên Storefront.
* **Đặc tính:** Strong Master Entity.
* **Khóa chính:** `BrandID INT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `BrandID` | `INT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate định danh thương hiệu |
| `BrandCode` | `VARCHAR(50)` | **NO** | — | Mã thương hiệu viết hoa không dấu (ví dụ: `NIKE`, `LOCKNLOCK`) |
| `BrandName` | `NVARCHAR(100)` | **NO** | — | Tên thương hiệu hiển thị chính thức |
| `Slug` | `VARCHAR(120)` | **NO** | — | Đường dẫn URL chuẩn SEO phục vụ trang thương hiệu và bộ lọc |
| `CountryOfOrigin` | `NVARCHAR(100)` | YES | `NULL` | Quốc gia xuất xứ thương hiệu |
| `LogoURL` | `VARCHAR(500)` | YES | `NULL` | Đường dẫn ảnh logo thương hiệu trên CDN |
| `Description` | `NVARCHAR(500)` | YES | `NULL` | Giới thiệu ngắn về thương hiệu |
| `IsActive` | `BIT` | **NO** | `1` | Cờ trạng thái hoạt động: `1` (Đang kinh doanh), `0` (Ẩn/Ngừng hợp tác) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm tạo bản ghi |
| `UpdatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cập nhật lần cuối |

#### Named Constraints & Indexes:
* `PK_Brand`: `PRIMARY KEY CLUSTERED (BrandID)`
* `UQ_Brand_BrandCode`: `UNIQUE NONCLUSTERED (BrandCode)`
* `UQ_Brand_Slug`: `UNIQUE NONCLUSTERED (Slug)`
* `DF_Brand_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_Brand_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `DF_Brand_UpdatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR UpdatedAt`

---

### 3.2. Bảng `dbo.Category` (`E02` - Danh mục Sản phẩm Đa tầng)
* **Ý nghĩa:** Biểu diễn cây phân loại hàng hóa đa cấp (ví dụ: Thời trang Nam $\to$ Áo $\to$ Áo Sơ mi).
* **Đặc tính:** Strong Entity đệ quy (Adjacency List). Cột `CategoryLevel` là thuộc tính dẫn xuất có kiểm soát (Controlled Hierarchical Cache) được duy trì nguyên tử bởi `usp_UpsertCategory` / Trigger khi thay đổi nút cha nhằm tăng tốc render menu điều hướng (ghi nhận giải trình kỹ thuật tại M3.02).
* **Khóa chính:** `CategoryID INT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `CategoryID` | `INT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate định danh danh mục |
| `CategoryName` | `NVARCHAR(100)` | **NO** | — | Tên hiển thị của danh mục (Unicode tiếng Việt) |
| `Slug` | `VARCHAR(120)` | **NO** | — | Chuỗi URL thân thiện chuẩn SEO, duy nhất toàn hệ thống |
| `Description` | `NVARCHAR(500)` | YES | `NULL` | Mô tả nội dung danh mục |
| `ParentCategoryID` | `INT` | YES | `NULL` | Khóa ngoại đệ quy trỏ danh mục cha (`NULL` đối với danh mục gốc cấp 1) |
| `CategoryLevel` | `TINYINT` | **NO** | `1` | Độ sâu phân tầng: `1` (Root), `2..10` (Danh mục con) |
| `DisplayOrder` | `INT` | **NO** | `0` | Thứ tự sắp xếp trên thanh điều hướng cùng cấp |
| `IsActive` | `BIT` | **NO** | `1` | Trạng thái hiển thị: `1` (Hoạt động), `0` (Ẩn) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm tạo bản ghi |
| `UpdatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cập nhật lần cuối |

#### Named Constraints & Indexes:
* `PK_Category`: `PRIMARY KEY CLUSTERED (CategoryID)`
* `UQ_Category_Slug`: `UNIQUE NONCLUSTERED (Slug)`
* `FK_Category_Category_Parent`: `FOREIGN KEY (ParentCategoryID) REFERENCES dbo.Category(CategoryID) ON DELETE NO ACTION`
* `CK_Category_NoSelfParent`: `CHECK (ParentCategoryID IS NULL OR ParentCategoryID <> CategoryID)` (Chặn tự tham chiếu trực tiếp; chu trình nhiều bước được chặn bởi CTE kiểm tra trong `usp_UpsertCategory`)
* `CK_Category_RootLevelConsistency`: `CHECK ((ParentCategoryID IS NULL AND CategoryLevel = 1) OR (ParentCategoryID IS NOT NULL AND CategoryLevel BETWEEN 2 AND 10))` (Ngăn chặn triệt để lỗi gán danh mục con mang `CategoryLevel = 1` hoặc danh mục gốc mang `CategoryLevel > 1`)
* `DF_Category_CategoryLevel`: `DEFAULT (1) FOR CategoryLevel`
* `DF_Category_DisplayOrder`: `DEFAULT (0) FOR DisplayOrder`
* `DF_Category_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_Category_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `DF_Category_UpdatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR UpdatedAt`
* `IX_Category_ParentCategoryID_DisplayOrder`: `CREATE NONCLUSTERED INDEX IX_Category_ParentCategoryID_DisplayOrder ON dbo.Category(ParentCategoryID, DisplayOrder) INCLUDE (CategoryName, Slug, CategoryLevel, IsActive)`

---

### 3.3. Bảng `dbo.Product` (`E01` - Sản phẩm Thương mại)
* **Ý nghĩa:** Đại diện cho sản phẩm ở cấp độ thương mại chung, liên kết thương hiệu `Brand`, quản lý bài viết mô tả và mức thuế suất GTGT (`VatRate`) phục vụ snapshot hóa đơn đơn hàng.
* **Đặc tính:** Strong Entity quản lý vòng đời (`DRAFT`, `PUBLISHED`, `ARCHIVED`) đồng bộ chặt chẽ với cờ `IsActive`.
* **Khóa chính:** `ProductID INT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `ProductID` | `INT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate định danh sản phẩm |
| `ProductCode` | `VARCHAR(50)` | **NO** | — | Mã sản phẩm tự nhiên duy nhất (ví dụ: `PRD-NK-001`) |
| `BrandID` | `INT` | YES | `NULL` | Khóa ngoại trỏ về `Brand.BrandID` (`NULL` nếu sản phẩm không thương hiệu/nháp) |
| `ProductName` | `NVARCHAR(255)` | **NO** | — | Tên thương mại chính thức của sản phẩm |
| `Description` | `NVARCHAR(MAX)` | YES | `NULL` | Bài viết giới thiệu chi tiết, hướng dẫn sử dụng và bảo quản |
| `VatRate` | `DECIMAL(5,2)` | **NO** | `10.00` | Thuế suất GTGT áp dụng (`0.00`, `5.00`, `8.00`, `10.00` %) phục vụ snapshot `OrderLine` |
| `ProductStatus` | `VARCHAR(30)` | **NO** | `'DRAFT'` | Trạng thái vòng đời: `'DRAFT'`, `'PUBLISHED'`, `'ARCHIVED'` |
| `IsActive` | `BIT` | **NO** | `1` | Cờ hoạt động đồng bộ trạng thái: `1` (`DRAFT`/`PUBLISHED`), `0` (`DRAFT` khóa tạm / `ARCHIVED`) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm tạo sản phẩm |
| `UpdatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cập nhật lần cuối |

#### Named Constraints & Indexes:
* `PK_Product`: `PRIMARY KEY CLUSTERED (ProductID)`
* `UQ_Product_ProductCode`: `UNIQUE NONCLUSTERED (ProductCode)`
* `FK_Product_Brand`: `FOREIGN KEY (BrandID) REFERENCES dbo.Brand(BrandID) ON DELETE NO ACTION`
* `CK_Product_VatRate`: `CHECK (VatRate IN (0.00, 5.00, 8.00, 10.00))`
* `CK_Product_ProductStatus`: `CHECK (ProductStatus IN ('DRAFT', 'PUBLISHED', 'ARCHIVED'))`
* `CK_Product_StatusActiveConsistency`: `CHECK (NOT (ProductStatus = 'PUBLISHED' AND IsActive = 0) AND NOT (ProductStatus = 'ARCHIVED' AND IsActive = 1))` (Triệt tiêu trạng thái mâu thuẫn giữa `ProductStatus` và `IsActive`)
* `DF_Product_VatRate`: `DEFAULT (10.00) FOR VatRate`
* `DF_Product_ProductStatus`: `DEFAULT ('DRAFT') FOR ProductStatus`
* `DF_Product_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_Product_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `DF_Product_UpdatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR UpdatedAt`
* `IX_Product_BrandID_Status`: `CREATE NONCLUSTERED INDEX IX_Product_BrandID_Status ON dbo.Product(BrandID, ProductStatus) INCLUDE (ProductCode, ProductName, VatRate, IsActive)`

---

### 3.4. Bảng `dbo.ProductCategory` (`E03` - Liên kết Sản phẩm & Danh mục)
* **Ý nghĩa:** Giải quyết quan hệ nhiều-nhiều ($N:N$) giữa `Product` và `Category`, mang cờ danh mục chính `IsPrimary` theo **BD-01**.
* **Đặc tính:** Associative Entity.
* **Khóa chính:** `(ProductID, CategoryID)` (Composite Clustered Primary Key).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `ProductID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Product.ProductID` |
| `CategoryID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Category.CategoryID` |
| `IsPrimary` | `BIT` | **NO** | `0` | Cờ danh mục chính: `1` (Danh mục chính duy nhất), `0` (Danh mục phụ) |
| `AssignedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm gán sản phẩm vào danh mục |

#### Named Constraints & Indexes:
* `PK_ProductCategory`: `PRIMARY KEY CLUSTERED (ProductID, CategoryID)`
* `FK_ProductCategory_Product`: `FOREIGN KEY (ProductID) REFERENCES dbo.Product(ProductID) ON DELETE NO ACTION`
* `FK_ProductCategory_Category`: `FOREIGN KEY (CategoryID) REFERENCES dbo.Category(CategoryID) ON DELETE NO ACTION`
* `DF_ProductCategory_IsPrimary`: `DEFAULT (0) FOR IsPrimary`
* `DF_ProductCategory_AssignedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR AssignedAt`
* **Cưỡng chế đầy đủ hai chiều của `BD-01` ($\le 1$ và $\ge 1$ khi `PUBLISHED`):**
  1. *Cận trên ($\le 1$ — Declarative Filtered Unique Index):*
     ```sql
     CREATE UNIQUE NONCLUSTERED INDEX UX_ProductCategory_Primary
     ON dbo.ProductCategory(ProductID)
     WHERE IsPrimary = 1;
     ```
  2. *Cận dưới ($\ge 1$ khi `PUBLISHED` — Boundary SP & Trigger):* Thủ tục `usp_PublishProduct` và Trigger `trg_ProductCategory_GuardPublishedPrimary` bắt buộc sản phẩm ở trạng thái `'PUBLISHED'` phải luôn duy trì đúng 1 dòng `IsPrimary = 1` và ít nhất 1 `ProductVariant` đang `'ACTIVE'` (`PC-Q03`), đồng thời chặn xóa/hạ cờ `IsPrimary` khi sản phẩm đang `'PUBLISHED'`.
* **Chỉ mục chiều ngược phục vụ duyệt sản phẩm theo danh mục:**
  ```sql
  CREATE NONCLUSTERED INDEX IX_ProductCategory_CategoryID_ProductID
  ON dbo.ProductCategory(CategoryID, ProductID)
  INCLUDE (IsPrimary);
  ```

---

### 3.5. Bảng `dbo.ProductVariant` (`E04` - Hạt nhân Biến thể Bán, Tồn kho & Vận chuyển SKU)
* **Ý nghĩa:** Thực thể giao dịch trung tâm (Core SKU Entity) biểu diễn đơn vị hàng hóa cụ thể có thể định giá, lưu kho và tính cước vận chuyển.
* **Đặc tính:** Core SKU Entity (`BIGINT IDENTITY`). Bổ sung siêu khóa `(ProductID, VariantID)` và quy cách khối lượng/kích thước vật lý phục vụ tính cước giao nhận Zero-Trust. Cột `IsActive` là thuộc tính cờ đồng bộ có kiểm soát (**Controlled State-Flag Cache** do `VariantStatus -> IsActive` qua `CK_ProductVariant_StatusActiveConsistency`), giữ lại để tối ưu vị từ lọc đồng nhất toàn danh mục và giải trình kỹ thuật tại `M3.02`.
* **Khóa chính:** `VariantID BIGINT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `VariantID` | `BIGINT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate chịu tải tham chiếu giao dịch lớn |
| `ProductID` | `INT` | **NO** | — | Khóa ngoại trỏ về sản phẩm cha `Product.ProductID` |
| `SKU` | `VARCHAR(50)` | **NO** | — | Mã quản lý tồn kho tự nhiên duy nhất toàn hệ thống |
| `VariantName` | `NVARCHAR(255)` | **NO** | — | Tên hiển thị đầy đủ của biến thể (ví dụ: `"Áo Thun Cổ Tròn - Trắng / XL"`) |
| `SellingPrice` | `DECIMAL(19,4)` | **NO** | — | Giá bán lẻ niêm yết hiện hành **đã bao gồm thuế GTGT (VAT-Inclusive)**, nguyên đồng VNĐ (`CORR-03`) |
| `CompareAtPrice` | `DECIMAL(19,4)` | YES | `NULL` | Giá gốc so sánh trước giảm đã gồm VAT (nếu có, nguyên đồng VNĐ) |
| `WeightGrams` | `INT` | **NO** | `100` | Khối lượng đóng gói tính bằng gram (`> 0`) để tính phí ship server-side |
| `LengthCm` | `DECIMAL(8,2)` | YES | `NULL` | Chiều dài kiện quy đổi (cm) phục vụ hàng gia dụng/cồng kềnh |
| `WidthCm` | `DECIMAL(8,2)` | YES | `NULL` | Chiều rộng kiện quy đổi (cm) |
| `HeightCm` | `DECIMAL(8,2)` | YES | `NULL` | Chiều cao kiện quy đổi (cm) |
| `VariantStatus` | `VARCHAR(30)` | **NO** | `'ACTIVE'` | Trạng thái kinh doanh: `'ACTIVE'`, `'INACTIVE'`, `'DISCONTINUED'` |
| `IsActive` | `BIT` | **NO** | `1` | Cờ hoạt động đồng bộ: `1` (`ACTIVE`), `0` (`INACTIVE` / `DISCONTINUED`) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm khởi tạo SKU |
| `UpdatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cập nhật giá/trạng thái gần nhất |

#### Named Constraints & Indexes:
* `PK_ProductVariant`: `PRIMARY KEY CLUSTERED (VariantID)`
* `UQ_ProductVariant_SKU`: `UNIQUE NONCLUSTERED (SKU)`
* `UQ_ProductVariant_Product_Variant`: `UNIQUE NONCLUSTERED (ProductID, VariantID)`
* `FK_ProductVariant_Product`: `FOREIGN KEY (ProductID) REFERENCES dbo.Product(ProductID) ON DELETE NO ACTION`
* `CK_ProductVariant_SellingPrice_IntegerVND`: `CHECK (SellingPrice >= 0 AND SellingPrice = FLOOR(SellingPrice))` (Bảo đảm `OrderedQty * SellingPrice` luôn ra số nguyên đồng VNĐ khớp `DECIMAL(19,0)` trên `OrderLine`)
* `CK_ProductVariant_CompareAtPrice`: `CHECK (CompareAtPrice IS NULL OR (CompareAtPrice >= SellingPrice AND CompareAtPrice = FLOOR(CompareAtPrice)))`
* `CK_ProductVariant_WeightGrams`: `CHECK (WeightGrams > 0)`
* `CK_ProductVariant_Dimensions`: `CHECK ((LengthCm IS NULL OR LengthCm > 0) AND (WidthCm IS NULL OR WidthCm > 0) AND (HeightCm IS NULL OR HeightCm > 0))`
* `CK_ProductVariant_VariantStatus`: `CHECK (VariantStatus IN ('ACTIVE', 'INACTIVE', 'DISCONTINUED'))`
* `CK_ProductVariant_StatusActiveConsistency`: `CHECK ((VariantStatus = 'ACTIVE' AND IsActive = 1) OR (VariantStatus IN ('INACTIVE', 'DISCONTINUED') AND IsActive = 0))`
* `DF_ProductVariant_WeightGrams`: `DEFAULT (100) FOR WeightGrams`
* `DF_ProductVariant_VariantStatus`: `DEFAULT ('ACTIVE') FOR VariantStatus`
* `DF_ProductVariant_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_ProductVariant_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `DF_ProductVariant_UpdatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR UpdatedAt`
* `IX_ProductVariant_ProductID_Status`: `CREATE NONCLUSTERED INDEX IX_ProductVariant_ProductID_Status ON dbo.ProductVariant(ProductID, VariantStatus) INCLUDE (SKU, VariantName, SellingPrice, CompareAtPrice, WeightGrams, IsActive)`

---

### 3.6. Bảng `dbo.Attribute` (`E05` - Từ điển Thuộc tính Toàn cục)
* **Ý nghĩa:** Từ điển chuẩn hóa các loại thuộc tính dùng chung toàn sàn (`SIZE`, `COLOR`, `MATERIAL`, `ORIGIN`).
* **Đặc tính:** Strong Master Entity.
* **Khóa chính:** `AttributeID INT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `AttributeID` | `INT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate định danh thuộc tính |
| `AttributeCode` | `VARCHAR(50)` | **NO** | — | Mã thuộc tính chuẩn hóa dùng trong `CORR-09` JSON Snapshot (`SIZE`, `COLOR`) |
| `AttributeName` | `NVARCHAR(100)` | **NO** | — | Tên hiển thị tiếng Việt (`"Kích cỡ"`, `"Màu sắc"`, `"Chất liệu"`) |
| `DataType` | `VARCHAR(30)` | **NO** | `'STRING'` | Kiểu biểu diễn giá trị: `'STRING'`, `'NUMBER'`, `'BOOLEAN'`, `'COLOR_CODE'` |
| `AttributeDescription` | `NVARCHAR(500)` | YES | `NULL` | Ghi chú quy ước nhập liệu hoặc đơn vị đo |
| `IsActive` | `BIT` | **NO** | `1` | Trạng thái sử dụng: `1` (Đang mở), `0` (Khóa gán mới) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm tạo thuộc tính |

#### Named Constraints & Indexes:
* `PK_Attribute`: `PRIMARY KEY CLUSTERED (AttributeID)`
* `UQ_Attribute_AttributeCode`: `UNIQUE NONCLUSTERED (AttributeCode)`
* `CK_Attribute_DataType`: `CHECK (DataType IN ('STRING', 'NUMBER', 'BOOLEAN', 'COLOR_CODE'))`
* `DF_Attribute_DataType`: `DEFAULT ('STRING') FOR DataType`
* `DF_Attribute_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_Attribute_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`

---

### 3.7. Bảng `dbo.AttributeValue` (`E06` - Từ điển Giá trị Thuộc tính)
* **Ý nghĩa:** Danh mục các giá trị chuẩn hóa thuộc về một `Attribute` cụ thể.
* **Đặc tính:** Weak Entity phụ thuộc toàn phần vào `Attribute`. Khai báo siêu khóa `UQ_AttributeValue_Attribute_ValueID (AttributeID, AttributeValueID)` làm đích cho các **Composite Foreign Keys** của `E08` và `E09`.
* **Khóa chính:** `AttributeValueID INT IDENTITY(1,1)` (Surrogate Key Clustered).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `AttributeValueID` | `INT IDENTITY(1,1)` | **NO** | Auto increment | Khóa chính surrogate định danh giá trị thuộc tính |
| `AttributeID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Attribute.AttributeID` |
| `ValueCode` | `VARCHAR(50)` | **NO** | — | Mã giá trị chuẩn hóa phục vụ `CORR-09` JSON Snapshot (`XL`, `NAVY`, `COTTON`) |
| `Value` | `NVARCHAR(255)` | **NO** | — | Giá trị kỹ thuật chuẩn (`#000080`, `XL`, `100% Cotton`) |
| `DisplayValue` | `NVARCHAR(255)` | **NO** | — | Nhãn hiển thị thân thiện cho khách hàng (`"Xanh Navy"`, `"Size XL"`) |
| `DisplayOrder` | `INT` | **NO** | `0` | Thứ tự hiển thị trên bộ chọn biến thể và bộ lọc |
| `IsActive` | `BIT` | **NO** | `1` | Trạng thái kích hoạt: `1` (Hoạt động), `0` (Khóa) |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm tạo giá trị |

#### Named Constraints & Indexes:
* `PK_AttributeValue`: `PRIMARY KEY CLUSTERED (AttributeValueID)`
* `UQ_AttributeValue_Attribute_ValueID`: `UNIQUE NONCLUSTERED (AttributeID, AttributeValueID)` *(Siêu khóa bắt buộc cho Composite FK chống nhiễm chéo thuộc tính)*
* `UQ_AttributeValue_Attribute_ValueCode`: `UNIQUE NONCLUSTERED (AttributeID, ValueCode)`
* `UQ_AttributeValue_Attribute_Value`: `UNIQUE NONCLUSTERED (AttributeID, Value)`
* `FK_AttributeValue_Attribute`: `FOREIGN KEY (AttributeID) REFERENCES dbo.Attribute(AttributeID) ON DELETE NO ACTION`
* `DF_AttributeValue_DisplayOrder`: `DEFAULT (0) FOR DisplayOrder`
* `DF_AttributeValue_IsActive`: `DEFAULT (1) FOR IsActive`
* `DF_AttributeValue_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`

---

### 3.8. Bảng `dbo.ProductAttribute` (`E07` - Khai báo Thuộc tính & Vai trò trên Sản phẩm)
* **Ý nghĩa:** Khai báo sản phẩm sử dụng những thuộc tính nào và ấn định vai trò loại trừ lẫn nhau (`'VARIANT_DEFINING'` hoặc `'DESCRIPTIVE'`) theo **BD-06**.
* **Đặc tính:** Associative Entity ($N:N$ giữa `Product` và `Attribute`).
* **Khóa chính:** `(ProductID, AttributeID)` (Composite Clustered Primary Key).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `ProductID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Product.ProductID` |
| `AttributeID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Attribute.AttributeID` |
| `AttributeRole` | `VARCHAR(30)` | **NO** | — | Vai trò loại trừ: `'VARIANT_DEFINING'` (sinh SKU) hoặc `'DESCRIPTIVE'` (mô tả SP) |
| `IsRequired` | `BIT` | **NO** | `0` | `1`: Bắt buộc gán giá trị khi công bố sản phẩm / tạo biến thể |
| `DisplayOrder` | `INT` | **NO** | `0` | Thứ tự hiển thị thuộc tính trên trang chi tiết sản phẩm |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cấu hình thuộc tính |

#### Named Constraints & Indexes:
* `PK_ProductAttribute`: `PRIMARY KEY CLUSTERED (ProductID, AttributeID)`
* `FK_ProductAttribute_Product`: `FOREIGN KEY (ProductID) REFERENCES dbo.Product(ProductID) ON DELETE NO ACTION`
* `FK_ProductAttribute_Attribute`: `FOREIGN KEY (AttributeID) REFERENCES dbo.Attribute(AttributeID) ON DELETE NO ACTION`
* `CK_ProductAttribute_Role`: `CHECK (AttributeRole IN ('VARIANT_DEFINING', 'DESCRIPTIVE'))`
* `DF_ProductAttribute_IsRequired`: `DEFAULT (0) FOR IsRequired`
* `DF_ProductAttribute_DisplayOrder`: `DEFAULT (0) FOR DisplayOrder`
* `DF_ProductAttribute_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `IX_ProductAttribute_AttributeID`: `CREATE NONCLUSTERED INDEX IX_ProductAttribute_AttributeID ON dbo.ProductAttribute(AttributeID, ProductID) INCLUDE (AttributeRole)`

---

### 3.9. Bảng `dbo.ProductAttributeValue` (`E08` - Tập Giá trị Thuộc tính Sản phẩm: Whitelist & Đặc tả)
* **Ý nghĩa (Ngữ nghĩa Hai Chiều theo `AttributeRole` của `BD-06`):**
  1. Khi `ProductAttribute.AttributeRole = 'VARIANT_DEFINING'`: Đóng vai trò **Bộ lọc Whitelist** các giá trị được phép dùng khi khởi tạo SKU của sản phẩm (ví dụ: Áo mẫu A chỉ cho phép chọn màu Đen và Trắng).
  2. Khi `ProductAttribute.AttributeRole = 'DESCRIPTIVE'`: Lưu **Giá trị đặc tả chung cấp Sản phẩm** (ví dụ: Chất liệu = Cotton, Xuất xứ = Việt Nam), hỗ trợ trực tiếp Faceted Search có chỉ mục mà không bị trôi nổi EAV phi cấu trúc.
* **Đặc tính:** Associative Entity với **Diamond Composite Foreign Keys** khóa chặt cả hai nhánh `ProductAttribute` và `AttributeValue`. Do phụ thuộc hàm liên bảng `AttributeValueID -> AttributeID`, khóa ứng viên tối thiểu (Minimal Candidate Key) là `(ProductID, AttributeValueID)`; khóa chính Clustered `(ProductID, AttributeID, AttributeValueID)` là siêu khóa chủ đích giữ lại `AttributeID` (**Controlled Diamond-FK Attribute**) nhằm cưỡng chế trực tiếp cả hai khóa ngoại phức hợp `(ProductID, AttributeID)` và `(AttributeID, AttributeValueID)` ngay ở tầng Declarative DDL mà không cần Trigger kiểm tra chéo (ghi nhận giải trình kỹ thuật tại `M3.02`).
* **Khóa chính:** `(ProductID, AttributeID, AttributeValueID)` (Composite Clustered Primary Key / Superkey).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `ProductID` | `INT` | **NO** | — | Thành phần khóa ngoại trỏ `ProductAttribute` |
| `AttributeID` | `INT` | **NO** | — | Thành phần khóa ngoại chung kết nối `ProductAttribute` và `AttributeValue` |
| `AttributeValueID` | `INT` | **NO** | — | Thành phần khóa ngoại phức hợp trỏ `AttributeValue` |
| `AssignedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm gán giá trị vào sản phẩm |

#### Named Constraints & Indexes:
* `PK_ProductAttributeValue`: `PRIMARY KEY CLUSTERED (ProductID, AttributeID, AttributeValueID)`
* `FK_ProductAttributeValue_ProductAttribute`: `FOREIGN KEY (ProductID, AttributeID) REFERENCES dbo.ProductAttribute(ProductID, AttributeID) ON DELETE NO ACTION`
* `FK_ProductAttributeValue_AttributeValue`: **`FOREIGN KEY (AttributeID, AttributeValueID) REFERENCES dbo.AttributeValue(AttributeID, AttributeValueID) ON DELETE NO ACTION`** *(Chặn 100% lỗi gán nhầm `AttributeID = SIZE` với `AttributeValueID = Màu Đỏ`)*
* `DF_ProductAttributeValue_AssignedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR AssignedAt`
* `IX_ProductAttributeValue_Attr_Val_Product`: `CREATE NONCLUSTERED INDEX IX_ProductAttributeValue_Attr_Val_Product ON dbo.ProductAttributeValue(AttributeID, AttributeValueID, ProductID)` *(Phục vụ Faceted Filter theo thuộc tính mô tả)*

---

### 3.10. Bảng `dbo.VariantAttributeValue` (`E09` - Tổ hợp Giá trị Thuộc tính Cấu thành SKU)
* **Ý nghĩa:** Xác định tổ hợp giá trị thuộc tính `'VARIANT_DEFINING'` tạo nên một biến thể SKU cụ thể (ví dụ: SKU `AT-TR-XL` có `(COLOR = Trắng, SIZE = XL)`).
* **Đặc tính:** Associative Entity đạt chuẩn 3NF bảo toàn phụ thuộc hàm `(VariantID, AttributeID) -> AttributeValueID` (không đưa `ProductID` dư thừa vào bảng nhằm tránh vi phạm phụ thuộc bộ phận 2NF `VariantID -> ProductID`).
* **Khóa chính:** `(VariantID, AttributeID)` (Composite Clustered Primary Key).

#### Column Definitions:
| Tên Cột (Column Name) | Kiểu Dữ liệu SQL Server | Nullable | Giá trị Mặc định | Ý nghĩa & Mô tả Nghiệp vụ |
| :--- | :--- | :---: | :--- | :--- |
| `VariantID` | `BIGINT` | **NO** | — | Khóa ngoại trỏ về biến thể `ProductVariant.VariantID` |
| `AttributeID` | `INT` | **NO** | — | Khóa ngoại trỏ về `Attribute.AttributeID` |
| `AttributeValueID` | `INT` | **NO** | — | Thành phần khóa ngoại phức hợp trỏ `AttributeValue(AttributeID, AttributeValueID)` |
| `CreatedAt` | `DATETIMEOFFSET(7)` | **NO** | `SYSDATETIMEOFFSET()` | Thời điểm cấu thành biến thể |

#### Named Constraints & Indexes:
* `PK_VariantAttributeValue`: `PRIMARY KEY CLUSTERED (VariantID, AttributeID)`
* `UQ_VariantAttributeValue_Variant_AttrVal`: `UNIQUE NONCLUSTERED (VariantID, AttributeValueID)` *(Khóa ứng viên thứ hai phục vụ chứng minh 3NF tại M3.02)*
* `FK_VariantAttributeValue_ProductVariant`: `FOREIGN KEY (VariantID) REFERENCES dbo.ProductVariant(VariantID) ON DELETE NO ACTION`
* `FK_VariantAttributeValue_Attribute`: `FOREIGN KEY (AttributeID) REFERENCES dbo.Attribute(AttributeID) ON DELETE NO ACTION`
* `FK_VariantAttributeValue_AttributeValue`: **`FOREIGN KEY (AttributeID, AttributeValueID) REFERENCES dbo.AttributeValue(AttributeID, AttributeValueID) ON DELETE NO ACTION`** *(Chặn 100% lỗi nhiễm chéo giá trị khác loại thuộc tính ngay ở tầng Declarative DDL)*
* `DF_VariantAttributeValue_CreatedAt`: `DEFAULT (SYSDATETIMEOFFSET()) FOR CreatedAt`
* `IX_VariantAttributeValue_Attr_Val_Variant`: `CREATE NONCLUSTERED INDEX IX_VariantAttributeValue_Attr_Val_Variant ON dbo.VariantAttributeValue(AttributeID, AttributeValueID) INCLUDE (VariantID)` *(Tăng tốc Faceted Filter theo Size/Màu trên 400.000 dòng bằng Index Seek)*

> [!IMPORTANT]
> **Ba Bất biến Toàn vẹn Sống còn trên `VariantAttributeValue`:**
> 1. **Đơn trị trên từng chiều thuộc tính (`PK_VariantAttributeValue`):** Khóa chính `(VariantID, AttributeID)` bảo đảm một SKU không bao giờ mang 2 giá trị cho cùng một thuộc tính (không thể vừa Size L vừa Size XL).
> 2. **Tuân thủ Whitelist & Chỉ nhận `VARIANT_DEFINING` (`BR-PR13`, `BR-PR14`):** Thủ tục `usp_UpsertProductVariant` và Trigger `trg_VariantAttributeValue_EnforceWhitelistAndRole` kiểm tra thông qua JOIN với `ProductVariant(VariantID)` rằng cặp `(ProductID, AttributeID, AttributeValueID)` bắt buộc tồn tại trong `ProductAttributeValue` **và** có `ProductAttribute.AttributeRole = 'VARIANT_DEFINING'`.
> 3. **Cấm trùng lặp tổ hợp thuộc tính SKU trong cùng sản phẩm (`BR-PR12`):** Cấm tồn tại hai `VariantID` thuộc cùng một `ProductID` có tập các cặp `(AttributeID, AttributeValueID)` trùng khớp hoàn toàn (cưỡng chế qua chữ ký băm tổ hợp hoặc kiểm tra tập hợp trong `usp_UpsertProductVariant`).

---

## 4. MA TRẬN LIÊN KẾT LIÊN MIỀN NGHIỆP VỤ (CROSS-DOMAIN RELATIONS)

Phân hệ `SA01 – Product Catalog` cung cấp Hạt nhân SKU (`ProductVariant` – `E04`) cho toàn bộ các phân hệ hạ nguồn theo đúng mã chuẩn `E01` – `E79`:

| Bảng Nguồn (`SA01`) | Cột Nguồn | Bảng Đích Phụ thuộc | Phân hệ Đích | Cột Khóa Ngoại | Mục đích Nghiệp vụ & Tham chiếu |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ProductVariant` (E04) | `VariantID` | `InventoryBalance` (**E11**) | **SA02** (Inventory) | `VariantID` | Số dư tồn kho 3 phẩm chất và giá vốn WAC theo kho (`CR-01`) |
| `ProductVariant` (E04) | `VariantID` | `OrderWarehouseAllocation` (**E12**) | **SA02** (Inventory) | `VariantID` | Phân bổ xuất kho cho dòng đơn hàng Split Shipment (`CR-02`) |
| `ProductVariant` (E04) | `VariantID` | `StockReservationLine` (**E14**) | **SA02** (Inventory) | `VariantID` | Khóa giữ tồn kho tạm thời có thời hạn `ExpiresAt` (`CR-03`) |
| `ProductVariant` (E04) | `VariantID` | `InventoryTransactionLine` (**E16**) | **SA02** (Inventory) | `VariantID` | Dòng sổ cái biến động tồn kho 4-Delta (`CR-04`) |
| `ProductVariant` (E04) | `VariantID` | `StockCountLine` (**E18**) | **SA02** (Inventory) | `VariantID` | Kiểm đếm tồn kho thực tế theo SKU (`CR-05`) |
| `ProductVariant` (E04) | `VariantID` | `WarehouseTransferLine` (**E20**) | **SA02** (Inventory) | `VariantID` | Luân chuyển nội bộ SKU giữa hai kho vật lý (`CR-06`) |
| `ProductVariant` (E04) | `VariantID` | `SupplierProduct` (**E22**) | **SA03** (Procurement) | `VariantID` | Danh mục nguồn cung ứng và giá mua tham khảo (`CR-07`) |
| `ProductVariant` (E04) | `VariantID` | `PurchaseOrderLine` (**E24**) | **SA03** (Procurement) | `VariantID` | Đặt mua hàng hóa từ nhà cung cấp theo PO (`CR-08`) |
| `ProductVariant` (E04) | `VariantID` | `GoodsReceiptLine` (**E26**) | **SA03** (Procurement) | `VariantID` | Kiểm nhận hàng thực tế nhập kho theo đợt (`CR-09`) |
| `ProductVariant` (E04) | `VariantID` | `SupplierReturnLine` (**E28**) | **SA03** (Procurement) | `VariantID` | Xuất trả hàng lỗi cho nhà cung cấp (`CR-10`) |
| `ProductVariant` (E04) | `VariantID` | `CartItem` (**E39**) | **SA05** (Shopping) | `VariantID` | Mặt hàng khách chọn trong giỏ mua sắm (`CR-11`) |
| `ProductVariant` (E04) | `VariantID` | `OrderLine` (**E41**) | **SA06** (Sales) | `VariantID` | Dòng hợp đồng bán hàng kèm snapshot giá, thuế & JSON (`CR-12`) |
| `ProductVariant` (E04) | `VariantID` | `PromotionProduct` (**E51**) | **SA08** (Promotion) | `VariantID` | Danh sách SKU hưởng khuyến mãi Tầng 1 & hạn mức Flash Sale (`CR-13`) |

---

## 5. DỰ PHÓNG DUNG LƯỢNG THEO TRANG DỮ LIỆU 8KB (8KB PAGE CAPACITY SIZING - SA01)

Áp dụng công thức chuẩn kiến trúc lưu trữ **8KB Data Page** (`8,060 bytes` khả dụng, `2 bytes` slot array mỗi dòng) của SQL Server 2022 trên quy mô mục tiêu 3 năm:
* `Brand`: **500** thương hiệu | `Category`: **500** danh mục | `Product`: **50.000** sản phẩm
* `ProductCategory`: **100.000** dòng | `ProductVariant`: **200.000** SKUs (trung bình 4 SKU/SP)
* `Attribute`: **100** thuộc tính | `AttributeValue`: **2.000** giá trị
* `ProductAttribute`: **150.000** dòng | `ProductAttributeValue`: **300.000** dòng | `VariantAttributeValue`: **400.000** dòng

| Tên Bảng (`dbo.*`) | Kích thước Dòng TB (`AvgRowSize`) | Số Dòng / Trang 8KB (`Rows/Page`) | Số Dòng Dự phóng | Số Trang Dữ liệu (`Data Pages`) | Dung lượng Dữ liệu (`Data MiB`) | Dung lượng Chỉ mục NCI (`Index MiB`) | Tổng Dung lượng (`Total MiB`) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `Brand` | ~291 B | 27 | 500 | 19 | 0.15 MiB | 0.03 MiB | **0.18 MiB** |
| `Category` | ~224 B | 35 | 500 | 15 | 0.12 MiB | 0.05 MiB | **0.17 MiB** |
| `Product` | ~955 B | 8 | 50.000 | 6.250 | 48.83 MiB | 7.27 MiB | **56.10 MiB** |
| `ProductCategory` | 26 B (exact) | 287 | 100.000 | 349 | 2.73 MiB | 2.44 MiB | **5.17 MiB** |
| `ProductVariant` | ~182 B | 43 | 200.000 | 4.652 | 36.34 MiB | 38.67 MiB | **75.01 MiB** |
| `Attribute` | ~135 B | 58 | 100 | 2 | 0.02 MiB | 0.02 MiB | **0.04 MiB** |
| `AttributeValue` | ~87 B | 90 | 2.000 | 23 | 0.18 MiB | 0.16 MiB | **0.34 MiB** |
| `ProductAttribute` | ~50 B | 155 | 150.000 | 968 | 7.56 MiB | 2.62 MiB | **10.18 MiB** |
| `ProductAttributeValue` | 29 B (exact) | 260 | 300.000 | 1.154 | 9.02 MiB | 6.41 MiB | **15.43 MiB** |
| `VariantAttributeValue` | 33 B (exact) | 230 | 400.000 | 1.740 | 13.59 MiB | 19.26 MiB | **32.85 MiB** |
| **TỔNG CỘNG SA01** | — | — | **1.203.100** | **15.172** | **118.54 MiB** | **76.93 MiB** | **195.47 MiB** |

> [!TIP]
> Tổng dung lượng toàn bộ 10 bảng và 19 Nonclustered Indexes (`11 UQ_ + 1 UX_ + 7 IX_`) của phân hệ `Product Catalog` chỉ chiếm **~195.5 MiB** (`15.172` trang dữ liệu 8KB), nằm gọn 100% trong bộ nhớ đệm SQL Server Buffer Pool (RAM) giúp các truy vấn đọc danh mục và Faceted Filter đạt độ trễ dưới `2ms`.

---

## 6. MA TRẬN KIỂM CHỨNG TOÀN VẸN NGHIỆP VỤ (VALIDATION CHECKLIST)

| Mã Quy tắc / QĐ | Nội dung Quy tắc Nghiệp vụ | Cơ chế Cưỡng chế Chính xác trong Schema Logic `SA01` | Trạng thái |
| :---: | :--- | :--- | :---: |
| **BD-01 / BR-PR10** | Sản phẩm thuộc nhiều danh mục; khi `PUBLISHED` bắt buộc có đúng 1 danh mục chính (`IsPrimary = 1`) | Cận trên $\le 1$: `UX_ProductCategory_Primary`; Cận dưới $\ge 1$ khi `PUBLISHED`: `usp_PublishProduct` + `trg_ProductCategory_GuardPublishedPrimary` | **ĐẠT (VERIFIED)** |
| **BD-06 / BR-PR13** | Thuộc tính động 2 lớp loại trừ: `'VARIANT_DEFINING'` vs `'DESCRIPTIVE'` | `CK_ProductAttribute_Role` trên `ProductAttribute`; phân tách ngữ nghĩa Whitelist/Đặc tả trên `ProductAttributeValue` | **ĐẠT (VERIFIED)** |
| **BR-PR01** | Mã sản phẩm `ProductCode` là duy nhất toàn hệ thống | Ràng buộc `UQ_Product_ProductCode` trên bảng `Product` | **ĐẠT (VERIFIED)** |
| **BR-PR02** | Mã `SKU` là duy nhất toàn hệ thống trên tất cả biến thể | Ràng buộc `UQ_ProductVariant_SKU` trên bảng `ProductVariant` | **ĐẠT (VERIFIED)** |
| **BR-PR05** | Cây danh mục đa cấp nhất quán độ sâu và không có chu trình lặp | `CK_Category_NoSelfParent` + `CK_Category_RootLevelConsistency` + kiểm tra đệ quy CTE trong `usp_UpsertCategory` | **ĐẠT (VERIFIED)** |
| **BR-PR06** | Giá bán lẻ `SellingPrice` không âm và là số nguyên đồng VNĐ | `CK_ProductVariant_SellingPrice_IntegerVND` & `CK_ProductVariant_CompareAtPrice` | **ĐẠT (VERIFIED)** |
| **BR-PR11** | Thương hiệu hàng hóa chuẩn hóa không dị thường chuỗi tự do | Bảng Master `Brand` (`E73`) độc lập với `UQ_Brand_BrandCode`, `UQ_Brand_Slug` và `FK_Product_Brand` | **ĐẠT (VERIFIED)** |
| **BR-PR12 / BR-PR14** | Cấm nhiễm chéo thuộc tính, bắt buộc thuộc Whitelist & cấm trùng tổ hợp SKU | Siêu khóa `UQ_AttributeValue_Attribute_ValueID` + Composite FK `(AttributeID, AttributeValueID)` trên `E08`/`E09` + `trg_VariantAttributeValue_EnforceWhitelistAndRole` | **ĐẠT (VERIFIED)** |
| **BR-PR15** | Đầy đủ thuế suất GTGT và khối lượng/kích thước tính cước vận chuyển | `Product.VatRate` (`CK_Product_VatRate`) và `ProductVariant.WeightGrams`/`LengthCm`/`WidthCm`/`HeightCm` | **ĐẠT (VERIFIED)** |
| **CORR-09** | Chuẩn bị đầy đủ định danh tạo JSON Array Snapshot bất biến cho `OrderLine` | Đầy đủ `AttributeID`, `AttributeCode`, `AttributeName`, `AttributeValueID`, `ValueCode`, `DisplayValue` khớp chuẩn JSON Array của `GEMINI.md` | **ĐẠT (VERIFIED)** |
