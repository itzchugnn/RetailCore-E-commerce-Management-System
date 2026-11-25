# 08. NON-FUNCTIONAL REQUIREMENTS – YÊU CẦU PHI CHỨC NĂNG (BASELINE v2.0 – SSOT)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 2.0 – Quantified Engineering Targets
* **Quy mô dữ liệu thiết kế (3 năm):** `50.000 Products`, `200.000 SKUs`, `1.000.000 Đơn hàng/năm` (~`2.740 đơn/ngày` trung bình; đỉnh tải Flash Sale `200+ kết nối đặt đơn đồng thời/giây`), `10.000.000 dòng biến động sổ cái kho/năm`.

---

## 8.1. Performance & Scalability (`NFR-PF`)

| ID | Yêu cầu Định lượng |
| :--- | :--- |
| **NFR-PF01** | $95\%$ truy vấn đọc danh mục và tồn kho SKU trên SQL Server có thời gian phản hồi dưới $100\text{ms}$ (P95 API đọc dưới $500\text{ms}$). |
| **NFR-PF02** | Giao dịch Checkout nguyên tử (`usp_Checkout_CreateOrder`) đạt P95 dưới $800\text{ms}$ ở mức tải đồng thời $200\text{ VUs}$ không tính độ trễ mạng ra cổng thanh toán ngoài. |
| **NFR-PF03** | Mọi danh sách sản phẩm, đơn hàng, thẻ kho bắt buộc phân trang, lọc và sắp xếp phía Server (Server-side Pagination & Faceted Filtering). |
| **NFR-PF04** | Bật `READ_COMMITTED_SNAPSHOT (RCSI)` trên `RetailCoreDB` để truy vấn đọc và báo cáo không gây Lock Blocking lên luồng giao dịch đặt hàng. |
| **NFR-PF05** | Mọi khóa ngoại (Foreign Keys) tham gia JOIN hoặc lọc truy vấn phải có Nonclustered Index hỗ trợ để triệt tiêu Table/Clustered Index Scan ngoài ý muốn. |

---

## 8.2. Security & Access Control (`NFR-SE`)

| ID | Yêu cầu Bảo mật |
| :--- | :--- |
| **NFR-SE01** | Mật khẩu người dùng và mã xác thực OTP phải được băm một chiều bằng BCrypt (cost $\ge 10$) hoặc Argon2id. |
| **NFR-SE02** | Xác thực JWT kết hợp Refresh Token lưu trong `HttpOnly; Secure; SameSite` Cookie để phòng chống đánh cắp phiên qua XSS. |
| **NFR-SE03** | Phân quyền RBAC động (`Role` – `Permission`) kết hợp kiểm tra sở hữu bản ghi theo `CustomerID` chống triệt để lỗ hổng IDOR. |
| **NFR-SE04** | Tách biệt 2 tài khoản CSDL: `flyway_migrator` (chỉ chạy DDL khi khởi động) và `retailcore_app` (runtime bị `REVOKE INSERT, UPDATE, DELETE` trực tiếp trên các bảng chứng từ lõi theo `CORR-04`). |
| **NFR-SE05** | 100% truy vấn SQL và lời gọi Stored Procedure phải tham số hóa (Parameterized Queries) chống SQL Injection. |
| **NFR-SE06** | Tuyệt đối không ghi mật khẩu, OTP, khóa bí mật Webhook hoặc dữ liệu nhạy cảm vào `AuditLog` hay application logs. |

---

## 8.3. Data Integrity & Accounting Accuracy (`NFR-DI`)

| ID | Yêu cầu Toàn vẹn Dữ liệu |
| :--- | :--- |
| **NFR-DI01** | Đảm bảo 100% toàn vẹn tham chiếu bằng Named Foreign Keys (`ON DELETE NO ACTION`), bao gồm Composite FK chống nhiễm chéo thuộc tính động. |
| **NFR-DI02** | Tiền giao dịch VNĐ lưu chính xác kiểu số nguyên `DECIMAL(19,0)`; đơn giá và giá vốn WAC lưu `DECIMAL(19,4)` (tuyệt đối cấm `MONEY`, `FLOAT`). |
| **NFR-DI03** | Tổng chiết khấu phân bổ LRM và tổng tiền hoàn lũy tiến qua nhiều đợt ($\Delta F = F(a+b) - F(a)$) phải khớp tuyệt đối từng đồng VNĐ (`0đ` sai số làm tròn). |
| **NFR-DI04** | Bộ 5 CHECK Constraints trên `InventoryBalance` cưỡng chế tuyệt đối `AvailableQty >= 0` ở cấp Engine CSDL trong mọi kịch bản lỗi ứng dụng. |
| **NFR-DI05** | Bảo toàn bất biến 100% lịch sử chứng từ giao dịch (`No Hard Delete` & `Immutable Snapshot`). |

---

## 8.4. Concurrency & Idempotency (`NFR-CC`)

| ID | Yêu cầu Xử lý Đồng thời |
| :--- | :--- |
| **NFR-CC01** | Đạt **0% Overselling** và **0 Deadlock** khi $200$ khách hàng đồng thời tranh mua những đơn vị tồn kho cuối cùng của cùng một SKU Flash Sale. |
| **NFR-CC02** | Khóa dòng tồn kho theo thứ tự sắp xếp tất định (`VariantID ASC, WarehouseID ASC`) với gợi ý khóa `UPDLOCK, HOLDLOCK`. |
| **NFR-CC03** | Không giữ kết nối/giao dịch SQL Server mở trong khi chờ gọi HTTP ra cổng thanh toán hoặc hãng vận chuyển bên ngoài. |
| **NFR-CC04** | Cưỡng chế Idempotency nguyên tử cho `POST /api/orders` và `POST /api/payments` qua `IdempotencyRecord` (E74) và chống replay callback qua `PaymentWebhookLog` (E71). |

---

## 8.5. Reliability, Availability & Recovery (`NFR-RL`)

| ID | Yêu cầu Độ tin cậy & Phục hồi |
| :--- | :--- |
| **NFR-RL01** | Mọi Boundary Stored Procedure bật `SET XACT_ABORT ON` và quản lý Savepoint an toàn để bảo đảm Rollback nguyên vẹn khi gặp lỗi runtime. |
| **NFR-RL02** | Mục tiêu phục hồi thảm họa CSDL: **RPO < 5 phút** (nhờ lịch Transaction Log Backup mỗi 3–5 phút kết hợp Full/Differential Backup) và **RTO < 30 phút**. |
| **NFR-RL03** | Môi trường phát triển/kiểm thử Docker dùng `SQL Server 2022 Developer Edition`; kiến trúc hướng sản xuất hỗ trợ mở rộng lên `SQL Server Enterprise Always On Availability Groups` (mục tiêu sẵn sàng cao lên tới $99.95\% - 99.99\%$ khi triển khai đa nút). |
| **NFR-RL04** | Tích hợp giám sát thời gian thực (Prometheus + Grafana) đo lường HikariCP pool, độ trễ API, lỗi 5xx và SQL Server Lock/Deadlock metrics. |

---

## 8.6. Maintainability & Usability (`NFR-MA` & `NFR-US`)

| ID | Yêu cầu Bảo trì & Trải nghiệm |
| :--- | :--- |
| **NFR-MA01** | Quản lý 100% phiên bản lược đồ CSDL và Stored Procedures tự động qua Flyway Migration. |
| **NFR-MA02** | Tuân thủ nghiêm ngặt chuẩn đặt tên Named Constraints (`PK_`, `FK_`, `UQ_`, `UX_`, `IX_`, `CK_`, `DF_`) xuyên suốt 79 bảng. |
| **NFR-MA03** | Xuất bản tài liệu đặc tả REST API tự động theo chuẩn OpenAPI 3.0 / Swagger. |
| **NFR-US01** | Giao diện Web hỗ trợ Responsive đầy đủ 3 trạng thái (`Skeleton`, `Empty`, `Error`) trên cả Desktop và Mobile. |
