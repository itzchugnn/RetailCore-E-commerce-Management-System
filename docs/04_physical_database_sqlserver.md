# PHẦN VI. THIẾT KẾ CƠ SỞ DỮ LIỆU VẬT LÝ – SQL SERVER
**Deliverable 03 – SQL Server Database**

Đây là phần trọng tâm nhất của dự án (chiếm tổng cộng $60\%$ trọng số đánh giá gồm Thiết kế CSDL và Hiệu năng SQL Server).

---

## 6.1. Môi trường Triển khai

| Thành phần | Lựa chọn kỹ thuật |
| :--- | :--- |
| **DBMS** | Microsoft SQL Server 2022 (hoặc phiên bản mới hơn tương thích) |
| **IDE / Công cụ** | SQL Server Management Studio (SSMS) / Azure Data Studio |
| **Tên Database** | `RetailCoreDB` |
| **Encoding** | `NVARCHAR` cho dữ liệu chuỗi Unicode (tiếng Việt) |
| **Storage Architecture** | Tổ chức Filegroup tách biệt: Primary (`MDF`), Secondary Data/Indexes (`NDF`), Transaction Log (`LDF`) |
| **Chiến lược Backup** | Full Backup + Differential Backup + Transaction Log Backup |

---

## 6.2. Quy ước Thiết kế Kiểu Dữ liệu (Data Type Conventions)

Mọi lựa chọn kiểu dữ liệu phải tối ưu bộ nhớ trang (8KB Data Page) và tốc độ truy xuất B-Tree Index:

* **`BIGINT IDENTITY(1,1)`:** Dùng làm khóa chính thay thế (Surrogate Key) ở các bảng giao dịch lớn hoặc có tần suất tăng trưởng cao (`Orders`, `OrderDetails`, `ProductVariants`, `StockMovements`, `Payments`, `AuditLogs`).
* **`INT IDENTITY(1,1)`:** Dùng cho các bảng danh mục hoặc thực thể có số lượng bản ghi vừa và nhỏ (`Products`, `Categories`, `Brands`, `Warehouses`, `Users`).
* **`DECIMAL(19,4)`:** Bắt buộc dùng cho mọi giá trị tiền tệ để đảm bảo độ chính xác tuyệt đối, tránh sai số dấu phẩy động.
* **`DATETIME2(0)` hoặc `DATETIME2(3)`:** Dùng cho các cột thời điểm (tiết kiệm dung lượng và chính xác hơn `DATETIME` truyền thống).
* **`NVARCHAR(n)`:** Dùng cho nội dung văn bản có dấu tiếng Việt (Tên sản phẩm, Họ tên, Địa chỉ). Dùng `VARCHAR(n)` cho các mã định danh chỉ chứa ký tự ASCII (`SKU`, `OrderCode`, `VoucherCode`).
* **`BIT`:** Dùng cho các thuộc tính kiểu Boolean (`IsDefault`, `IsStackable`).
* **`TINYINT` hoặc `SMALLINT`:** Dùng cho các cột mã trạng thái (`Status`, `MovementType`) có miền giá trị cố định.
* **`UNIQUEIDENTIFIER`:** Chỉ sử dụng khi thực sự có nhu cầu định danh toàn cục (Global ID) hoặc tích hợp hệ thống phân tán; **không** mặc định dùng UUID cho mọi khóa chính Clustered Index để tránh phân mảnh trang (Index Fragmentation) và phình kích thước khóa ngoại.

---

## 6.3. Ví dụ Triển khai DDL Mẫu trên SQL Server

*Lưu ý: Ba đoạn script dưới đây minh họa phong cách viết DDL chuẩn mực với đầy đủ tên ràng buộc (Named Constraints). Bộ script chính thức sẽ được hoàn thiện sau khi chốt toàn bộ lược đồ logic ở Milestone M3.*

### Ví dụ 1: Khởi tạo Database và Bảng `Products`
~~~sql
CREATE DATABASE RetailCoreDB;
GO

USE RetailCoreDB;
GO

CREATE TABLE dbo.Products
(
    ProductID INT IDENTITY(1,1) NOT NULL,
    BrandID INT NULL,
    ProductName NVARCHAR(200) NOT NULL,
    Description NVARCHAR(MAX) NULL,
    Status TINYINT NOT NULL
        CONSTRAINT DF_Products_Status DEFAULT 1,
    CreatedAt DATETIME2(0) NOT NULL
        CONSTRAINT DF_Products_CreatedAt DEFAULT SYSDATETIME(),

    CONSTRAINT PK_Products
        PRIMARY KEY CLUSTERED (ProductID),

    CONSTRAINT CK_Products_Status
        CHECK (Status IN (0, 1, 2))
);
GO
~~~

### Ví dụ 2: Bảng Biến thể Sản phẩm (`ProductVariants`)
~~~sql
CREATE TABLE dbo.ProductVariants
(
    VariantID BIGINT IDENTITY(1,1) NOT NULL,
    ProductID INT NOT NULL,
    SKU VARCHAR(50) NOT NULL,
    Price DECIMAL(19,4) NOT NULL,
    Status TINYINT NOT NULL
        CONSTRAINT DF_Variants_Status DEFAULT 1,

    CONSTRAINT PK_ProductVariants
        PRIMARY KEY CLUSTERED (VariantID),

    CONSTRAINT UQ_ProductVariants_SKU
        UNIQUE (SKU),

    CONSTRAINT FK_Variants_Products
        FOREIGN KEY (ProductID)
        REFERENCES dbo.Products(ProductID),

    CONSTRAINT CK_Variants_Price
        CHECK (Price >= 0)
);
GO
~~~

### Ví dụ 3: Bảng Tồn kho (`Inventory`) với Computed Column Persisted
~~~sql
CREATE TABLE dbo.Inventory
(
    WarehouseID INT NOT NULL,
    VariantID BIGINT NOT NULL,

    OnHandQty INT NOT NULL
        CONSTRAINT DF_Inventory_OnHandQty DEFAULT 0,
    ReservedQty INT NOT NULL
        CONSTRAINT DF_Inventory_ReservedQty DEFAULT 0,

    AvailableQty AS (OnHandQty - ReservedQty) PERSISTED,

    CONSTRAINT PK_Inventory
        PRIMARY KEY CLUSTERED (WarehouseID, VariantID),

    CONSTRAINT CK_Inventory_OnHand
        CHECK (OnHandQty >= 0),

    CONSTRAINT CK_Inventory_Reserved
        CHECK (
            ReservedQty >= 0
            AND ReservedQty <= OnHandQty
        )
);
GO
~~~

---

## 6.4. Những Kỹ thuật SQL Server Bắt buộc Nghiên cứu & Áp dụng

| Kỹ thuật | Bài toán áp dụng trong RetailCore |
| :--- | :--- |
| **Primary Key / Foreign Key** | Đảm bảo toàn vẹn thực thể (Entity Integrity) và toàn vẹn tham chiếu (Referential Integrity). |
| **UNIQUE / CHECK Constraints** | Chặn dữ liệu sai quy tắc nghiệp vụ ngay tại tầng lưu trữ (VD: giá âm, tồn kho âm, trùng SKU). |
| **DEFAULT Constraints** | Thiết lập giá trị mặc định tự động cho trạng thái bản ghi và dấu thời gian hệ thống. |
| **Clustered Index** | Định hình cấu trúc vật lý của bảng dữ liệu (B-Tree) tối ưu cho các truy vấn phạm vi hoặc khóa chính. |
| **Nonclustered Index** | Tăng tốc các truy vấn tìm kiếm, lọc và kết nối (JOIN) thường xuyên. |
| **Covering Index (`INCLUDE`)** | Bao phủ toàn bộ các cột trong `SELECT` để loại bỏ hoàn toàn chi phí `Key Lookup`. |
| **Filtered Index (`WHERE`)** | Đánh chỉ mục trên tập con dữ liệu có điều kiện (VD: đơn hàng đang xử lý, mã giao dịch `NOT NULL`). |
| **View / Indexed View** | Phục vụ truy vấn báo cáo doanh thu và tổng hợp số liệu phức tạp. |
| **Stored Procedure** | Đóng gói các giao dịch nghiệp vụ quan trọng cần tính nguyên tử và hiệu năng cao. |
| **Trigger (`AFTER`)** | Kiểm soát các ràng buộc liên bảng phức tạp hoặc tự động ghi nhận nhật ký kiểm toán (Audit/History) khi phù hợp. |
| **Transaction Management** | Đảm bảo tính chất ACID (đặc biệt là Atomicity và Consistency) cho chuỗi thao tác nhiều bảng. |
| **Isolation Level & Locking** | Xử lý bài toán truy cập đồng thời (Concurrency Control), chống Lost Update và Overselling. |
| **Execution Plan Analysis** | Đọc hiểu Actual Execution Plan (`Scan` vs `Seek`, `Hash Match`, `Nested Loops`, `Key Lookup`) để chứng minh hiệu quả tối ưu. |
| **Backup & Restore** | Xây dựng quy trình sao lưu và diễn tập phục hồi dữ liệu đến một thời điểm cụ thể (Point-in-Time Recovery). |

---

## 6.5. Danh mục 13 Stored Procedures Trọng tâm Cần Xây dựng

> *Nguyên tắc: Không biến mọi thao tác CRUD đơn giản thành Stored Procedure. Chỉ ưu tiên xây dựng Stored Procedure cho những nghiệp vụ đòi hỏi tính nguyên tử (Atomicity), kiểm soát đồng thời (Concurrency) chặt chẽ hoặc xử lý dữ liệu tập trung tại Database.*

| Tên Stored Procedure | Nghiệp vụ xử lý |
| :--- | :--- |
| `usp_CreateProduct` | Khởi tạo sản phẩm gốc cùng danh mục và các biến thể SKU ban đầu trong một giao dịch. |
| `usp_UpdateProduct` | Cập nhật thông tin sản phẩm và đồng bộ trạng thái các biến thể liên quan. |
| `usp_ReceivePurchaseOrder` | Xác nhận phiếu nhập kho: cập nhật trạng thái phiếu, cộng `OnHandQty` vào `Inventory` và ghi `StockMovements`. |
| `usp_TransferInventory` | Chuyển hàng giữa hai kho: trừ kho xuất, cộng kho nhập và ghi nhận hai dòng lịch sử biến động. |
| `usp_ReserveInventory` | Giữ hàng trong kho khi đặt đơn (kiểm tra `AvailableQty` và tăng `ReservedQty` an toàn đồng thời). |
| `usp_ReleaseInventory` | Giải phóng lượng hàng đang giữ khi đơn hàng bị hủy hoặc hết hạn thanh toán. |
| `usp_CreateOrder` | Tạo đơn hàng hoàn chỉnh: kiểm tra giá/voucher, tạo `Orders`, `OrderDetails` (snapshot), `OrderAddresses` và giữ tồn kho. |
| `usp_ConfirmOrder` | Nhân viên xác nhận đơn hàng và ghi nhận lịch sử chuyển trạng thái `OrderStatusHistory`. |
| `usp_ProcessPayment` | Ghi nhận kết quả giao dịch thanh toán, kiểm tra tổng tiền thu và cập nhật trạng thái thanh toán của đơn. |
| `usp_ApplyPromotion` | Kiểm tra điều kiện, tính toán mức giảm giá và ghi nhận lượt sử dụng Voucher (`VoucherUsages`) an toàn đồng thời. |
| `usp_CreateReturnRequest` | Khởi tạo yêu cầu đổi trả hàng kèm danh sách sản phẩm và số lượng hoàn trả hợp lệ so với đơn gốc. |
| `usp_ProcessRefund` | Xử lý hoàn tiền tham chiếu giao dịch gốc và cập nhật nhập lại tồn kho nếu hàng đạt chuẩn. |
| `usp_GetSalesReport` | Truy vấn thống kê doanh thu, lợi nhuận, số đơn hàng theo khoảng thời gian, danh mục và kho. |

---

## 6.6. Bài toán Transaction và Kiểm soát Đồng thời (Concurrency Control)

Đây là điểm tạo nên sự khác biệt cốt lõi giữa dự án kỹ thuật dữ liệu thực thụ và các đồ án CRUD thông thường.

### Kịch bản điển hình: Cạnh tranh SKU cuối cùng trong kho (Race Condition / Overselling)
Giả sử trong kho chỉ còn đúng $1$ sản phẩm (`AvailableQty = 1`). Hai khách hàng A và B nhấn nút "Đặt hàng" tại cùng một mili-giây:

| Thời điểm | Khách hàng A | Khách hàng B | Trạng thái `Inventory` mong muốn |
| :--- | :--- | :--- | :--- |
| $t_1$ | Gửi yêu cầu đặt $1$ sản phẩm | Gửi yêu cầu đặt $1$ sản phẩm | `OnHand = 1`, `Reserved = 0`, `Available = 1` |
| $t_2$ | Giao dịch A khóa dòng và giữ hàng thành công | Giao dịch B phải chờ khóa (Lock Wait) | `OnHand = 1`, `Reserved = 1`, `Available = 0` |
| $t_3$ | **Kết quả:** Đặt hàng thành công | **Kết quả:** Báo lỗi "Không đủ tồn kho" | Không xảy ra tình trạng bán lố (`Reserved = 2`) |

### Các kỹ thuật bắt buộc nghiên cứu và áp dụng:
1. Quản lý giao dịch tường minh: `BEGIN TRANSACTION`, `COMMIT TRANSACTION`, `ROLLBACK TRANSACTION` kết hợp khối `TRY...CATCH`.
2. Bật `SET XACT_ABORT ON` trong các Stored Procedure giao dịch để đảm bảo SQL Server tự động rollback và giải phóng khóa khi phát sinh lỗi runtime.
3. Sử dụng Table Hints như `UPDLOCK`, `ROWLOCK`, `HOLDLOCK` khi đọc kiểm tra số lượng tồn kho hoặc lượt dùng Voucher trong giao dịch cập nhật.
4. Cấu hình và hiểu rõ cơ chế hoạt động của `READ_COMMITTED_SNAPSHOT` (RCSI) cho các truy vấn đọc thông thường nhằm tránh việc đọc chặn ghi (Readers blocking Writers).
5. Thiết kế thứ tự truy cập tài nguyên nhất quán giữa các giao dịch để phòng tránh **Deadlock**, đồng thời xử lý thử lại (Retry logic) khi phát hiện Deadlock (Error 1205).
6. Thiết kế cơ chế **Idempotency (Tính lũy đẳng)** bằng `IdempotencyKey` để chống tạo trùng đơn hàng hoặc thanh toán kép khi mạng chập chờn khiến API gửi yêu cầu lặp.
7. **Cập nhật nguyên tử có điều kiện (Atomic Conditional Update):** Việc giữ hàng phải thực hiện ngay trong câu lệnh `UPDATE ... WHERE AvailableQty >= @RequestedQty`, tuyệt đối không tách thành hai bước đọc `SELECT` rồi mới `UPDATE` mà không có khóa bảo vệ.

---

## 6.7. Chiến lược Đánh Chỉ mục (Indexing Strategy)

Không tạo Index một cách máy móc trên mọi cột. Mỗi Index thêm vào đều tiêu tốn dung lượng đĩa/RAM (Buffer Pool) và làm chậm các thao tác `INSERT`, `UPDATE`, `DELETE`. Chỉ tạo Index dựa trên các mẫu truy vấn (Query Patterns) thực tế:

| Truy vấn nghiệp vụ phổ biến | Chiến lược Index cần xem xét |
| :--- | :--- |
| Tìm kiếm biến thể sản phẩm theo mã `SKU` | `UNIQUE NONCLUSTERED INDEX` trên cột `SKU` (tự động tạo qua `UQ_ProductVariants_SKU`). |
| Duyệt danh sách sản phẩm theo Danh mục | Index trên `ProductCategories(CategoryID, ProductID)` kết hợp Covering Index trên `Products(Status) INCLUDE (ProductName, BrandID)`. |
| Khách hàng xem lịch sử đơn hàng của mình | Nonclustered Index trên `Orders(CustomerID, OrderDate DESC) INCLUDE (OrderCode, Status, TotalAmount)`. |
| Nhân viên lọc và xử lý đơn hàng theo trạng thái | Nonclustered Index (hoặc Filtered Index cho các trạng thái đang mở) trên `Orders(Status, OrderDate ASC)`. |
| Kiểm tra và cập nhật tồn kho theo Kho và SKU | Đã được tối ưu bởi `PRIMARY KEY CLUSTERED (WarehouseID, VariantID)` trên bảng `Inventory`. |
| Tra cứu đối soát lịch sử biến động tồn kho | Nonclustered Index trên `StockMovements(WarehouseID, VariantID, CreatedAt DESC)` và trên `(ReferenceID, MovementType)`. |