---
name: retailcore-tsql-standards
description: >-
  Use when designing and writing SQL Server 2022+ T-SQL DDL, declarative constraints, indexes, and stored procedures for RetailCore (Milestone 3 / M3.03, M3.04 & Milestone 4/5).
  Enforces strict named constraints, data type standards, concurrency patterns, and transaction templates.
---

# RetailCore SQL Server 2022+ T-SQL Enterprise Standards Skill

Bộ quy chuẩn lập trình CSDL T-SQL dành cho **Microsoft SQL Server 2022+** trong hệ thống **RetailCore**.

---

## 1. TIÊU CHUẨN KIỂU DỮ LIỆU BẮT BUỘC (DATA TYPE STANDARDS)

| Mục đích dữ liệu | Kiểu dữ liệu chuẩn | Ràng buộc & Ghi chú |
| :--- | :--- | :--- |
| **Tiền tệ giao dịch VNĐ** | `DECIMAL(19,0)` | Tiền VNĐ thanh toán/tổng tiền là số nguyên đồng, không lưu số lẻ xu. |
| **Đơn giá, Tỷ lệ %, Giá vốn WAC** | `DECIMAL(19,4)` | Giữ 4 chữ số thập phân cho WAC và tỷ lệ chiết khấu; với giá bán lẻ B2C đã gồm VAT (`SellingPrice`, `CompareAtPrice`) thêm `CHECK (Price = FLOOR(Price))`. |
| **Tỷ lệ thuế GTGT (VAT)** | `DECIMAL(5,2)` | Giá trị chuẩn theo quy định thuế Việt Nam: `0.00`, `5.00`, `8.00`, `10.00`. |
| **Kiểu cấm sử dụng** | **TUYỆT ĐỐI CẤM** | Cấm `MONEY`, `SMALLMONEY` (lỗi làm tròn 4 chữ số trung gian), cấm `FLOAT`, `REAL` (sai số dấu phẩy động). |
| **Khóa chính giao dịch / SKU / Account** | `BIGINT IDENTITY(1,1)` | `ProductVariant`, `Account`, `SalesOrder`, `OrderLine`, `InventoryTransactionLine`, `PaymentTransaction`, `AuditLog`. |
| **Khóa chính danh mục master (12 bảng)** | `INT IDENTITY(1,1)` | `Brand`, `Category`, `Product`, `Attribute`, `AttributeValue`, `Warehouse`, `Supplier`, `Carrier`, `Role`, `Permission`, `PaymentMethod`, `ReportDefinition`. |
| **Chuỗi Unicode tiếng Việt** | `NVARCHAR(len)` | Tên sản phẩm `NVARCHAR(255)`, Họ tên `NVARCHAR(100)`, Địa chỉ `NVARCHAR(255)`. |
| **Chuỗi mã định danh ASCII** | `VARCHAR(len)` | `SKU VARCHAR(50)`, `OrderNumber VARCHAR(30)`, `VoucherCode VARCHAR(30)`. |
| **Thời gian hệ thống** | `DATETIMEOFFSET(7)` | Chuẩn ISO 8601 có múi giờ (`+07:00`), mặc định `SYSDATETIMEOFFSET()`. |
| **Cờ logic (Boolean)** | `BIT NOT NULL` | Kèm `DEFAULT (0)` hoặc `DEFAULT (1)`. |
| **Dữ liệu bán cấu trúc (JSON)** | `NVARCHAR(MAX)` | Bắt buộc kèm Check Constraint `ISJSON(...) = 1`. |

---

## 2. QUY ƯỚC ĐẶT TÊN NAMED CONSTRAINTS & INDEXES

Tất cả ràng buộc và chỉ mục CSDL bắt buộc đặt tên rõ ràng, tuyệt đối không để SQL Server tự sinh tên ngẫu nhiên:

* **Primary Key:** `PK_<TableName>`
  ```sql
  CONSTRAINT PK_SalesOrder PRIMARY KEY CLUSTERED (OrderID)
  ```
* **Foreign Key:** `FK_<ChildTable>_<ParentTable>[_<Purpose>]` (mặc định `ON DELETE NO ACTION` + kèm Nonclustered Index hỗ trợ chiều truy vấn ngược)
  ```sql
  CONSTRAINT FK_OrderLine_SalesOrder FOREIGN KEY (OrderID)
      REFERENCES dbo.SalesOrder(OrderID) ON DELETE NO ACTION
  ```
* **Unique Constraint / Filtered Index:** `UQ_<TableName>_<Columns>` hoặc `UX_<TableName>_<Purpose>`
  ```sql
  CONSTRAINT UQ_ProductVariant_SKU UNIQUE NONCLUSTERED (SKU)
  ```
* **Nonclustered Search Index:** `IX_<TableName>_<Columns>`
* **Check Constraint:** `CK_<TableName>_<RuleDescription>`
* **Default Constraint:** `DF_<TableName>_<ColumnName>`

---

## 3. FILTERED UNIQUE INDEXES BẮT BUỘC

* **Giỏ hàng Active duy nhất cho mỗi khách:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_ShoppingCart_Active
  ON dbo.ShoppingCart(CustomerID)
  WHERE CartStatus = 'ACTIVE';
  ```
* **Định danh Email / Số điện thoại duy nhất khi khác NULL trên `Account`:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_Account_Email ON dbo.Account(Email) WHERE Email IS NOT NULL;
  CREATE UNIQUE NONCLUSTERED INDEX UX_Account_Phone ON dbo.Account(Phone) WHERE Phone IS NOT NULL;
  ```
* **Tối đa 1 danh mục chính cho mỗi sản phẩm:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_ProductCategory_Primary
  ON dbo.ProductCategory(ProductID)
  WHERE IsPrimary = 1;
  ```
* **Tối đa 1 dòng giữ hàng ACTIVE cho mỗi dòng phân bổ kho:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_StockReservationLine_ActiveAlloc
  ON dbo.StockReservationLine(AllocationID)
  WHERE ReservationLineStatus = 'ACTIVE';
  ```
* **Tối đa 1 phiên bản hiện hành cho mỗi lô giám định hàng trả:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_ReturnLineAssessment_Current
  ON dbo.ReturnLineAssessment(ReturnReceiptLineID, AssessmentGroupNo)
  WHERE IsCurrent = 1;
  ```
* **Chuyển đổi giỏ hàng sang đơn hàng (cho phép Direct-Buy `CartID IS NULL`):**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_SalesOrder_Cart
  ON dbo.SalesOrder(CartID)
  WHERE CartID IS NOT NULL;
  ```
* **Địa chỉ mặc định duy nhất cho mỗi khách:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_CustomerAddress_Default
  ON dbo.CustomerAddress(CustomerID)
  WHERE IsDefault = 1;
  ```
* **Chống Webhook Replay từ cổng thanh toán:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_PaymentWebhookLog_ProviderEvent
  ON dbo.PaymentWebhookLog(PaymentMethodID, ProviderEventID, Environment)
  WHERE ProviderEventID IS NOT NULL;
  ```
* **Mã vận đơn duy nhất theo hãng vận chuyển và liên kết hoàn tiền 1:1 theo biên bản KCS:**
  ```sql
  CREATE UNIQUE NONCLUSTERED INDEX UX_Shipment_TrackingNumber ON dbo.Shipment(CarrierID, TrackingNumber) WHERE TrackingNumber IS NOT NULL;
  CREATE UNIQUE NONCLUSTERED INDEX UX_Refund_Assessment ON dbo.Refund(AssessmentID) WHERE AssessmentID IS NOT NULL;
  ```

---

## 4. KHUÔN MẪU STORED PROCEDURE GIAO DỊCH AN TOÀN (SAVEPOINT-AWARE TEMPLATE)

Mọi Boundary Stored Procedure thực thi nghiệp vụ đa bảng bắt buộc hỗ trợ cả gọi độc lập lẫn gọi lồng nhau qua `@@TRANCOUNT` + `SAVE TRANSACTION`:

```sql
CREATE OR ALTER PROCEDURE dbo.usp_ExampleTransaction
    @Param1 BIGINT,
    @Param2 INT,
    @CurrentAccountID BIGINT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    DECLARE @TranCounter INT = @@TRANCOUNT;
    DECLARE @SavePointName SYSNAME = N'SP_' + CAST(@@PROCID AS NVARCHAR(20));

    BEGIN TRY
        IF @TranCounter = 0
            BEGIN TRANSACTION;
        ELSE
            SAVE TRANSACTION @SavePointName;

        -- 1. Khóa tài nguyên theo thứ tự cố định chống Deadlock (Lock Ordering)
        -- Ví dụ: ORDER BY VariantID ASC, WarehouseID ASC với gợi ý WITH (UPDLOCK, HOLDLOCK)

        -- 2. Thực thi thao tác nghiệp vụ & kiểm tra bất biến

        IF @TranCounter = 0
            COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF XACT_STATE() = -1 AND @TranCounter = 0
            ROLLBACK TRANSACTION;
        ELSE IF XACT_STATE() = 1
        BEGIN
            IF @TranCounter = 0
                ROLLBACK TRANSACTION;
            ELSE
                ROLLBACK TRANSACTION @SavePointName;
        END;

        THROW; -- Ném mã lỗi chuẩn (50001+) để tầng Backend @RestControllerAdvice ánh xạ HTTP Status
    END CATCH
END;
GO
```

---

## 5. DANH MỤC 5 BOUNDARY STORED PROCEDURES CỐT LÕI CỦA M3.04

1. `usp_Checkout_CreateOrder`: Thực thi nguyên tử All-or-Nothing toàn bộ quy trình kiểm tra `IdempotencyRecord` (E74), tạo `SalesOrder` (E40), snapshot `OrderAddress` (E42, kèm hóa đơn B2B tùy chọn), tính giá đã gồm VAT / bóc tách `TaxAmount` / khuyến mãi 3 tầng (`PRODUCT`, `ORDER` LRM, `SHIPPING` Freeship) trên `OrderLine` (E41), phân bổ đa kho `OrderWarehouseAllocation` (E12), khóa giữ tồn kho `StockReservationLine` (E14, TTL 15m / 5m Flash Sale) và khởi tạo `Payment` (E56).
2. `usp_Identity_RegisterAccount`: Tạo nguyên tử `Account` (E29) và hồ sơ con `Customer` (E30) hoặc `Employee` (E31) bảo đảm Disjoint XOR & Total Participation (`CORR-04`).
3. `usp_PostInventoryTransaction`: Ghi sổ cái bất biến `InventoryTransaction` (E15) / `InventoryTransactionLine` (E16, bao gồm `RETURN_RECEIPT_IN` vào `QUARANTINED`), cập nhật 4 delta trên `InventoryBalance` (E11) và tính lại giá vốn bình quân WAC (`CORR-08`).
4. `usp_ReceivePaymentWebhook`: Lưu bền vững `PaymentWebhookLog` (E71), xác thực chữ ký và xử lý idempotent chuyển trạng thái `PaymentTransaction` (E57) / `Payment` (E56).
5. `usp_AssessReturnReceiptLine`: Phiên bản hóa biên bản giám định KCS `ReturnLineAssessment` (E72), tính tiền hoàn lũy tiến $\Delta F = F(a+b) - F(a)$ dùng chung trên `CancelledQty + RefundedQty` với `UPDLOCK` trên `OrderLine`, kiểm tra thu hồi voucher (**Clawback**) khi lỗi khách làm rớt `MinOrderAmount` (`CORR-03`), và sinh `Refund` (E58) hoặc đơn đổi hàng `SalesOrder` (`EXCHANGE_REPLACEMENT`, `CORR-01`).
