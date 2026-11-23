---
name: retailcore-backend-architecture
description: >-
  Use when developing, structuring, and maintaining the Java Spring Boot 3.x backend REST API, Flyway migrations, and database access layer for RetailCore (Milestone 5).
  Enforces clean layered architecture, hybrid JPA / JdbcClient data access, zero-trust pricing, idempotency, and security standards.
---

# RetailCore Backend Architecture & Spring Boot Engineering Skill

Bộ quy chuẩn phát triển tầng Backend bằng **Java 17/21+** và **Spring Boot 3.x** tích hợp **Microsoft SQL Server** trong hệ thống **RetailCore** (Milestone 5).

---

## 1. CẤU TRÚC PACKAGE PHÂN LỚP ĐỀ XUẤT

Tổ chức mã nguồn theo kiến trúc Domain-Driven Package (`com.retailcore.*`):

```text
com.retailcore
├── common
│   ├── config          # DatabaseConfig, SecurityConfig, WebConfig, OpenApiConfig
│   ├── dto             # ApiResponse<T>, PageResponse<T>, ErrorResponse
│   ├── exception       # BusinessException, ResourceNotFoundException, GlobalExceptionHandler
│   ├── security        # JwtTokenProvider, CustomUserDetails, SecurityUtils
│   └── util            # MoneyUtils, HashUtils, DateTimeUtils
├── catalog             # Product, Category, Variant, Attribute
├── inventory           # Warehouse, InventoryBalance, Allocation, Reservation
├── procurement         # Supplier, PurchaseOrder, GoodsReceipt
├── identity            # Account, Customer, Employee, Role, Permission
├── cart                # ShoppingCart, CartItem
├── sales               # SalesOrder, OrderLine, OrderAddress, OrderStatusHistory
├── shipping            # Carrier, Shipment, ShipmentLine, DeliveryAttempt
├── promotion           # Promotion, Voucher, PromotionUsage, LrmAllocator
├── payment             # Payment, PaymentTransaction, Refund, WebhookLog
├── returns             # ReturnRequest, ReturnReceipt, ReturnLineAssessment
└── reporting           # ReportExecution, ReportSnapshot, SalesSummary
```

---

## 2. MÔ HÌNH TRUY CẬP DỮ LIỆU LAI (HYBRID DATA ACCESS PATTERN)

Để tối ưu giữa tốc độ phát triển và hiệu năng tối đa trên CSDL SQL Server:

1. **Spring Data JPA / Hibernate:**
   * Áp dụng cho: Các thao tác CRUD danh mục master, quản lý quan hệ thực thể, truy vấn đọc thông thường (Read Models) và nghiệp vụ giỏ hàng/người dùng.
   * Quy tắc cấu hình:
     * Luôn sử dụng DTO Projections hoặc `@EntityGraph` để chống triệt để lỗi **N+1 Query**.
     * Thiết lập `spring.jpa.open-in-view=false` để tránh giữ Connection DB ngoài tầng Service.
     * Cấm định nghĩa quan hệ hai chiều `@OneToMany` không kiểm soát trên các bảng giao dịch lớn (`SalesOrder`, `InventoryTransaction`).
2. **Spring `JdbcClient` / Native Stored Procedure Caller:**
   * Áp dụng cho: Các tác vụ nghiệp vụ có yêu cầu khắt khe về tính nguyên tử và tốc độ cao:
     * Checkout tạo đơn hàng: `usp_CreateSalesOrderWithAddress`.
     * Giữ hàng và phân bổ kho: `usp_AllocateAndReserveStock`.
     * Ghi sổ cái biến động kho: `usp_PostInventoryTransaction`.
     * Tiếp nhận Webhook cổng thanh toán: `usp_ReceivePaymentWebhook`.
   * Tránh hoàn toàn tình trạng Entity Tracking ngốn bộ nhớ heap khi xử lý các batch hàng nghìn dòng.

---

## 3. QUY CHUẨN QUẢN LÝ SCHEMA MIGRATION VỚI FLYWAY

Mọi thay đổi CSDL bắt buộc được quản lý qua Flyway Migration trong `src/main/resources/db/migration/`:

* **Phiên bản hóa (Versioned Migrations):** `V<Major>.<Minor>__<Description>.sql`
  * `V1.0__init_core_schema.sql` (Tạo bảng, khóa chính).
  * `V1.1__add_constraints_and_indexes.sql` (Named constraints, Filtered indexes).
  * `V1.2__seed_master_data.sql` (Dữ liệu nền tảng Role, Category, Carrier, PaymentMethod).
* **Di chuyển Lặp lại (Repeatable Migrations):** `R__<ProcedureOrViewName>.sql`
  * Áp dụng cho Stored Procedures, Views, Functions để có thể sửa đổi và tái triển khai theo mã nguồn Git mà không cần sinh file version mới.
* **Nguyên tắc:** Tuyệt đối không bật `spring.jpa.hibernate.ddl-auto=update` trong môi trường Production.

---

## 4. TÁM NGUYÊN TẮC BẢO MẬT & XỬ LÝ DỮ LIỆU BẮT BUỘC

1. **Zero-Trust Pricing:** Tuyệt đối không nhận `UnitPrice` hoặc `TotalAmount` từ Frontend gửi lên. Backend tự động lấy giá niêm yết từ `ProductVariant`, tính toán chiết khấu qua Promotion Engine và tính phí ship server-side.
2. **Connection Pool Safety:** **Tuyệt đối không mở SQL Transaction giữ khóa (Lock) trong thời gian gọi HTTP ra bên ngoài** (như gọi API VNPAY/MoMo).
   * Quy trình đúng: Tạo bản ghi `PaymentTransaction` trạng thái `INITIATED` $\to$ Commit Transaction $\to$ Gọi HTTP ra Gateway $\to$ Nhận phản hồi/Webhook $\to$ Mở Transaction mới cập nhật kết quả.
3. **Idempotency Chống Gửi Lặp:**
   * API Tạo đơn (`POST /api/orders`) và Thanh toán (`POST /api/payments`) bắt buộc nhận header `Idempotency-Key`.
   * Backend kiểm tra bảng đệm: nếu trùng Key và trùng băm SHA-256 payload thì trả lại `ResponseSnapshot` cũ ngay lập tức; nếu trùng Key nhưng khác payload thì báo lỗi `409 Conflict`.
4. **Bảo mật Phân quyền RBAC:** Cấu hình Spring Security với JWT; bảo vệ API theo quyền hạn tối thiểu: `@PreAuthorize("hasAuthority('ORDER_MANAGE')")`.
5. **Chống lỗi IDOR (Insecure Direct Object Reference):** Mọi truy vấn xem/hủy đơn hàng của khách hàng bắt buộc phải kiểm tra điều kiện sở hữu: `WHERE OrderID = :id AND CustomerID = :currentAuthCustomerId`.
6. **Bảo vệ Thông tin Nhạy cảm:** DTO không bao giờ chứa cột `PasswordHash`, mã OTP, hoặc khóa bí mật Webhook.
7. **Xử lý Tiền tệ VNĐ Chính xác:**
   * Tiền giao dịch VNĐ dùng kiểu `BigDecimal` với `scale = 0`.
   * Đơn giá và giá vốn WAC dùng `BigDecimal` với `scale = 4`.
   * Sử dụng thuật toán Largest Remainder Method (LRM) khi phân bổ chiết khấu voucher cấp đơn.
8. **Xử lý Ngoại lệ Tập trung (`@RestControllerAdvice`):** Bắt và ánh xạ mã lỗi SQL Server (ví dụ Custom Error Number của Stored Procedure) sang mã HTTP và thông điệp chuẩn hóa cho người dùng.

