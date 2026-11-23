---
name: retailcore-tsql-standards
description: >-
  Use when designing and writing SQL Server 2022+ T-SQL DDL, declarative constraints, indexes, and stored procedures for RetailCore (Milestone 3 / M3.03, M3.04 & Milestone 4/5).
  Enforces strict named constraints, data type standards, concurrency patterns, and transaction templates.
---

# RetailCore SQL Server 2022+ T-SQL Enterprise Standards Skill

Bộ quy chuẩn lập trình CSDL T-SQL dành cho **Microsoft SQL Server Enterprise Edition** trong hệ thống **RetailCore**.

---

## 1. TIÊU CHUẨN KIỂU DỮ LIỆU BẮT BUỘC (DATA TYPE STANDARDS)

| Mục đích dữ liệu | Kiểu dữ liệu chuẩn | Ràng buộc & Ghi chú |
| :--- | :--- | :--- |
| **Tiền tệ giao dịch VNĐ** | `DECIMAL(19,0)` | Tiền VNĐ thanh toán thực tế là số nguyên đồng, không lưu số lẻ xu. |
| **Đơn giá, Tỷ lệ %, Giá vốn WAC** | `DECIMAL(19,4)` | Giữ 4 chữ số thập phân để triệt tiêu sai số làm tròn khi nhân/chia. |
| **Kiểu cấm sử dụng** | **TUYỆT ĐỐI CẤM** | Cấm `MONEY`, `SMALLMONEY` (lỗi làm tròn 4 chữ số ngầm định), cấm `FLOAT`, `REAL` (sai số dấu phẩy động). |
| **Khóa chính giao dịch lớn** | `BIGINT IDENTITY(1,1)` | Bảng giao dịch: `SalesOrder`, `OrderLine`, `InventoryTransactionLine`, `PaymentTransaction`, `AuditLog`. |
| **Khóa chính danh mục master** | `INT IDENTITY(1,1)` | Bảng danh mục vừa và nhỏ: `Category`, `Brand`, `Warehouse`, `Carrier`, `Role`. |
| **Chuỗi Unicode tiếng Việt** | `NVARCHAR(len)` | Tên sản phẩm `NVARCHAR(200)`, Họ tên `NVARCHAR(100)`, Địa chỉ `NVARCHAR(255)`. |
| **Chuỗi mã định danh ASCII** | `VARCHAR(len)` | `SKU VARCHAR(50)`, `OrderNumber VARCHAR(30)`, `VoucherCode VARCHAR(30)`. |
| **Thời gian hệ thống** | `DATETIME2(0)` hoặc `DATETIMEOFFSET(7)` | `DATETIME2(0)` cho mốc giây thông thường; `DATETIMEOFFSET(7)` cho các giao dịch đa múi giờ. |
| **Cờ logic (Boolean)** | `BIT NOT NULL` | Kèm `DEFAULT 0` hoặc `DEFAULT 1`. |
| **Dữ liệu bán cấu trúc** | `NVARCHAR(MAX)` | Phải có Check Constraint `ISJSON(...) = 1`. |

---

## 2. QUY ƯỚC ĐẶT TÊN NAMED CONSTRAINTS (NAMING CONVENTIONS)

Tất cả ràng buộc CSDL bắt buộc phải được đặt tên rõ ràng (Named Constraints), tuyệt đối không để SQL Server tự sinh tên ngẫu nhiên:

* **Primary Key:** `PK_<TableName>`
  ```sql
  CONSTRAINT PK_SalesOrder PRIMARY KEY CLUSTERED (OrderID)
  ```
* **Foreign Key:** `FK_<ChildTable>_<ParentTable>[_<Purpose>]`
  ```sql
  CONSTRAINT FK_OrderLine_SalesOrder FOREIGN KEY (OrderID) 
      REFERENCES dbo.SalesOrder(OrderID) ON DELETE NO ACTION
  ```
  *(Lưu ý: Mọi FK trên bảng giao dịch bắt buộc khai báo `ON DELETE NO ACTION` để ngăn Cascade Delete làm mất dữ liệu lịch sử).*
* **Unique Constraint / Index:** `UQ_<TableName>_<Columns>` hoặc `UX_<TableName>_<Columns>`
  ```sql
  CONSTRAINT UQ_ProductVariant_SKU UNIQUE (SKU)
  ```
* **Check Constraint:** `CK_<TableName>_<RuleDescription>`
  ```sql
  CONSTRAINT CK_InventoryBalance_OnHand_Positive CHECK (OnHandQty >= 0)
  ```
* **Default Constraint:** `DF_<TableName>_<ColumnName>`
  ```sql
  CONSTRAINT DF_SalesOrder_CreatedAt DEFAULT SYSDATETIME()
  ```

---

## 3. CHỈ MỤC CHUYÊN SÂU (ADVANCED INDEXING PATTERNS)

### 3.1. Filtered Unique Indexes (Cưỡng chế nghiệp vụ trên tập con dữ liệu)
* **Giỏ hàng Active duy nhất cho mỗi khách:**
  ```sql
  CREATE UNIQUE INDEX UX_ShoppingCart_Customer_Active 
  ON dbo.ShoppingCart(CustomerID) 
  WHERE CartStatus = 'ACTIVE';
  ```
* **Chỉ đúng 1 danh mục chính cho sản phẩm khi Published:**
  ```sql
  CREATE UNIQUE INDEX UX_ProductCategory_Primary 
  ON dbo.ProductCategory(ProductID) 
  WHERE IsPrimary = 1;
  ```
* **Chỉ 1 dòng giữ hàng ACTIVE cho mỗi dòng phân bổ kho:**
  ```sql
  CREATE UNIQUE INDEX UX_StockReservationLine_ActiveAlloc 
  ON dbo.StockReservationLine(OrderWarehouseAllocationID) 
  WHERE ReservationLineStatus = 'ACTIVE';
  ```
* **Địa chỉ mặc định duy nhất cho mỗi khách:**
  ```sql
  CREATE UNIQUE INDEX UX_CustomerAddress_Default 
  ON dbo.CustomerAddress(CustomerID) 
  WHERE IsDefault = 1;
  ```
* **Chống Webhook Replay từ cổng thanh toán:**
  ```sql
  CREATE UNIQUE INDEX UX_PaymentWebhookLog_ProviderEvent 
  ON dbo.PaymentWebhookLog(PaymentMethodID, ProviderEventID, Environment) 
  WHERE ProviderEventID IS NOT NULL;
  ```

---

## 4. KHUÔN MẪU STORED PROCEDURE GIAO DỊCH NGUYÊN TỬ (TRANSACTION TEMPLATE)

Mọi Stored Procedure thực thi nghiệp vụ đa bảng trong RetailCore bắt buộc tuân thủ khuôn mẫu:

```sql
CREATE OR ALTER PROCEDURE dbo.usp_ExampleTransaction
    @Param1 BIGINT,
    @Param2 INT,
    @CurrentUserID BIGINT,
    @ReturnCode INT OUTPUT,
    @ReturnMessage NVARCHAR(500) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON; -- Bắt buộc: Tự động Rollback toàn bộ khi gặp run-time error

    BEGIN TRY
        BEGIN TRANSACTION;

        -- 1. Sắp xếp tài nguyên khóa để chống Deadlock (Lock Ordering)
        -- Ví dụ khi khóa hàng loạt SKU: ORDER BY VariantID ASC, WarehouseID ASC

        -- 2. Thao tác nghiệp vụ
        -- INSERT / UPDATE ...

        COMMIT TRANSACTION;
        SET @ReturnCode = 0;
        SET @ReturnMessage = N'Thao tác thành công.';
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        SET @ReturnCode = ERROR_NUMBER();
        SET @ReturnMessage = ERROR_MESSAGE();

        -- Ghi log lỗi vào bảng chuyên dụng nếu cần
        THROW;
    END CATCH
END;
GO
```

---

## 5. DANH MỤC 5 STORED PROCEDURES CỐT LÕI CỦA M3.04

1. `usp_CreateSalesOrderWithAddress`: Tạo nguyên tử `SalesOrder`, snapshot địa chỉ `OrderAddress` và hồ sơ nghĩa vụ `Payment` trong cùng 1 boundary transaction.
2. `usp_AllocateAndReserveStock`: Phân bổ đa kho theo thứ tự sắp xếp và tạo `StockReservationLine`, cập nhật `ReservedQty` nguyên tử.
3. `usp_PostInventoryTransaction`: Ghi sổ cái bất biến `InventoryTransaction` / `InventoryTransactionLine` và cập nhật các delta trên `InventoryBalance`.
4. `usp_ReceivePaymentWebhook`: Lưu vết bảng đệm `PaymentWebhookLog`, kiểm tra chữ ký và idempotency trước khi xử lý chuyển trạng thái `Payment`.
5. `usp_AssessReturnReceiptLine`: Phiên bản hóa biên bản giám định KCS `ReturnLineAssessment` (E72), cập nhật `IsCurrent` và kích hoạt hồ sơ `Refund`.

