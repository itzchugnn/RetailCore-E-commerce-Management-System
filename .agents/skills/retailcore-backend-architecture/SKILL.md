---
name: retailcore-backend-architecture
description: >-
  Use when developing, structuring, and maintaining the Java Spring Boot 3.x backend REST API, Flyway migrations, and database access layer for RetailCore (Milestone 5).
  Enforces clean layered architecture, hybrid JPA / JdbcClient data access, dual DB users, zero-trust pricing, idempotency, and security standards.
---

# RetailCore Backend Architecture & Spring Boot Engineering Skill

Bộ quy chuẩn phát triển tầng Backend bằng **Java 21+** và **Spring Boot 3.x** tích hợp **Microsoft SQL Server 2022+** trong hệ thống **RetailCore** (Milestone 5).

---

## 1. CẤU TRÚC PACKAGE PHÂN LỚP THEO MIỀN NGHIỆP VỤ

```text
com.retailcore
├── common
│   ├── config          # DatabaseConfig, SecurityConfig, WebConfig, OpenApiConfig
│   ├── dto             # ApiResponse<T>, PageResponse<T>, ErrorResponse
│   ├── exception       # BusinessException, ResourceNotFoundException, GlobalExceptionHandler
│   ├── idempotency     # IdempotencyInterceptor / IdempotencyService (E74)
│   ├── security        # JwtTokenProvider, CustomUserDetails, SecurityUtils
│   └── util            # MoneyUtils, LrmCalculator, HashUtils, DateTimeUtils
├── catalog             # Brand (E73), Category, Product, ProductVariant, Attribute (SA01)
├── inventory           # Warehouse, InventoryBalance, Allocation, Reservation, Ledger (SA02)
├── procurement         # Supplier, PurchaseOrder, GoodsReceipt, SupplierReturn (SA03)
├── identity            # Account, Customer, Employee, Role, Permission (SA04)
├── cart                # ShoppingCart, CartItem (SA05)
├── sales               # SalesOrder, OrderLine, OrderAddress, OrderStatusHistory (SA06)
├── shipping            # Carrier, Shipment, DeliveryAttempt, DeliveryReturnReceipt (SA07)
├── promotion           # Promotion, Voucher, PromotionUsage (SA08)
├── payment             # Payment, PaymentTx, Refund, WebhookLog (E71), COD 3-Layer (E77–E79) (SA09)
├── returns             # ReturnRequest, ReturnReceipt, ReturnLineAssessment (E72), ReturnShipmentAllocation (E76) (SA10)
└── reporting           # AuditLog, ReportDefinition, ReportExecution, ReportSnapshot (SA11)
```

---

## 2. MÔ HÌNH TRUY CẬP DỮ LIỆU LAI & TƯƠNG THÍCH `REVOKE DML` (`CORR-04`)

1. **Tách Biệt Hai Tài Khoản CSDL (Dual Database Login — BẮT BUỘC):**
   * `spring.flyway.user = flyway_migrator`: Chỉ dùng lúc ứng dụng khởi động để thực thi DDL/Migration.
   * `spring.datasource.username = retailcore_app`: Tài khoản runtime bị `REVOKE INSERT, UPDATE, DELETE` trên `SalesOrder`, `OrderAddress`, `Payment`, `Account`, `Customer`, `Employee` và cấm `UPDATE, DELETE` trên `AuditLog`, chỉ có quyền `EXECUTE` trên các Boundary Stored Procedures. **Tuyệt đối không kết nối ứng dụng bằng `sa`.**
2. **Spring Data JPA / Hibernate (An toàn trước Dirty Checking):**
   * Dùng cho CRUD các bảng danh mục không khóa quyền DML và truy vấn đọc (Read Models).
   * **Quy tắc sống còn:** Mọi class `@Entity` ánh xạ tới bảng đã bị thu hồi DML (`SalesOrder`, `OrderAddress`, `Payment`, `Account`, `Customer`, `Employee`, `AuditLog`) **bắt buộc phải gắn `@org.hibernate.annotations.Immutable`** và dùng Read-Only Repository (không kế thừa `save`/`delete`), đồng thời gắn `@Transactional(readOnly = true)` cho các hàm đọc để ngăn Hibernate tự động flush lệnh `UPDATE` gây lỗi `SQL Error 229`.
   * Thiết lập `spring.jpa.open-in-view=false`; sử dụng DTO Projections / `@EntityGraph` chống lỗi N+1.
   * Không trộn thao tác ghi `EntityManager` và lời gọi `JdbcClient` lên cùng thực thể (như `ShoppingCart`) trong một phiên nếu chưa `entityManager.clear()`.
3. **Spring `JdbcClient` / Boundary Stored Procedures:**
   * Gọi nguyên tử một thủ tục biên cho mỗi giao dịch phức tạp:
     * Checkout All-or-Nothing: `usp_Checkout_CreateOrder` (gộp kiểm tra Idempotency, tạo Đơn, Địa chỉ, Phân bổ, Giữ kho và Payment trong 1 giao dịch nguyên tử).
     * Đăng ký Tài khoản TPT: `usp_Identity_RegisterAccount`.
     * Ghi sổ cái biến động kho & WAC: `usp_PostInventoryTransaction`.
     * Tiếp nhận Webhook cổng thanh toán: `usp_ReceivePaymentWebhook`.
     * Giám định RMA & Hoàn tiền lũy tiến $\Delta F$: `usp_AssessReturnReceiptLine`.

---

## 3. QUẢN LÝ SCHEMA MIGRATION THỐNG NHẤT VỚI FLYWAY

* Nguồn sự thật duy nhất của các script T-SQL thực thi là `backend/src/main/resources/db/migration/` (thư mục `database/` ở gốc dự án chỉ dùng chứa script khởi tạo instance/login ban đầu cho Docker và tài liệu kiểm thử tham khảo, tránh nhân đôi bản sao DDL).
* Phân loại: `V<Major>.<Minor>__<Description>.sql` cho bảng/ràng buộc/index và `R__<ProcedureOrViewName>.sql` cho Stored Procedures / Views. Tuyệt đối thiết lập `spring.jpa.hibernate.ddl-auto=validate` (hoặc `none`).

---

## 4. NGUYÊN TẮC BẢO MẬT, IDEMPOTENCY & TIỀN TỆ

1. **Zero-Trust Pricing, Tax & Shipping:** Backend/SP tự truy xuất `SellingPrice` (giá bán lẻ B2C đã bao gồm VAT), `VatRate` (bóc tách `TaxAmount` nội bộ), khuyến mãi 3 tầng (`PRODUCT`, `ORDER`, `SHIPPING` Freeship) và tính phí vận chuyển từ `WeightGrams` / kích thước `ProductVariant`.
2. **Connection Pool Safety:** Tuyệt đối không mở `@Transactional` giữ DB Connection/Lock khi đang gọi HTTP sang cổng thanh toán hoặc hãng vận chuyển.
3. **Atomic Idempotency Lock (`IdempotencyRecord` E74):**
   * Với `POST /api/orders` và `POST /api/payments`, thực hiện `INSERT` nguyên tử bản ghi `IN_PROGRESS` vào `IdempotencyRecord(AccountID, OperationType, IdempotencyKey, RequestHash)` trước:
     * Nếu vi phạm Unique Key và bản ghi cũ đang `IN_PROGRESS`: trả `409 Conflict` (đang xử lý).
     * Nếu bản ghi cũ `SUCCEEDED` và khớp `RequestHash`: trả lại ngay `ResponseSnapshot` đã lưu; nếu khác `RequestHash`: trả `409 Conflict`.
4. **Xử lý Tiền tệ VNĐ:** Số tiền giao dịch dùng `BigDecimal` `scale = 0`; đơn giá và WAC dùng `scale = 4`.
5. **Anti-IDOR & JWT Security:** Luôn lọc theo `CustomerID` của phiên xác thực hiện hành; không trả `PasswordHash` hoặc OTP trong bất kỳ DTO nào.
