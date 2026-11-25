---
name: retailcore-concurrency-testing
description: >-
  Use when planning, scripting, and executing concurrency stress tests, ACID verification, and automated testing for RetailCore (Milestone 7).
  Covers Flash Sale anti-overselling, deadlock detection, Testcontainers MSSQL integration testing, and k6 / JMeter performance suites.
---

# RetailCore Concurrency Stress Testing & Quality Assurance Skill

Bộ quy chuẩn kiểm thử chịu tải đồng thời, toàn vẹn giao dịch ACID và tự động hóa kiểm thử trong hệ thống **RetailCore** (Milestone 7).

---

## 1. MƯỜI KỊCH BẢN KIỂM THỬ CỐT LÕI (CORE TEST SCENARIOS)

1. **Chống Bán Âm Kho Flash Sale (Anti-Overselling):**
   * *Điều kiện:* 1 SKU chỉ còn đúng 5 sản phẩm khả dụng (`AvailableQty = 5`).
   * *Kịch bản:* 200 tài khoản khách hàng độc lập (200 VUs k6, mỗi VU 1 JWT riêng) cùng gửi yêu cầu Checkout mua 1 đơn vị SKU này tại cùng 1 giây.
   * *Kết quả chấp nhận:* Đúng 5 đơn hàng thành công (`201 Created`), 195 yêu cầu bị từ chối hết hàng (`409 Conflict` / `422 Unprocessable Entity`). Tuyệt đối $\text{OnHandQty} \ge 0$, $\text{AvailableQty} = 0$, **0 Deadlock**.
2. **Tranh Chấp Voucher Lượt Cuối (Voucher Quota Contention):**
   * *Điều kiện:* Voucher còn đúng 1 lượt dùng (`MaxUsage - (UsedCount + PendingCount) = 1`).
   * *Kịch bản:* 100 khách hàng độc lập cùng áp mã voucher và đặt đơn đồng thời.
   * *Kết quả chấp nhận:* Duy nhất 1 đơn khóa được lượt voucher; 99 yêu cầu nhận lỗi hết lượt.
3. **Chống Tạo Đơn Lặp (Atomic Idempotency Verification — `E74`):**
   * *Kịch bản:* Cùng 1 khách hàng gửi 2 request `POST /api/orders` giống hệt nhau (cùng `Idempotency-Key` và `RequestHash`) cách nhau `50ms`.
   * *Kết quả chấp nhận:* Chỉ sinh đúng 1 `SalesOrder`, chỉ trừ `ReservedQty` 1 lần; request song song nhận `409 Conflict` (nếu đang xử lý) hoặc nhận lại đúng `ResponseSnapshot` (khi đã hoàn tất).
4. **Đứt Gãy Kết Nối Cổng Thanh Toán & Chống Replay Webhook (`E71`):**
   * *Kịch bản:* Gửi cùng 1 sự kiện Webhook (`ProviderEventID`) lặp lại 5 lần đồng thời.
   * *Kết quả chấp nhận:* `UX_PaymentWebhookLog_ProviderEvent` đảm bảo chỉ xử lý nghiệp vụ đúng 1 lần; đơn chuyển `CONFIRMED`, không cộng tiền lặp.
5. **Giải Phóng Tồn Kho Khi Hết Hạn / Hủy Đơn (`BD-04`):**
   * *Kết quả chấp nhận:* Toàn bộ `ReservedQty` hoàn trả về `AvailableQty`; `OnHandQty` giữ nguyên không đổi.
6. **Bảo Toàn Tiền Lẻ Hoàn Trả Nhiều Đợt (`CORR-03` LRM & $\Delta F$):**
   * *Kịch bản:* Dòng đơn 3 sản phẩm có `LineNetAmount = 100.001đ` ($B = 33.333, R = 2$). Khách trả lẻ 3 đợt, mỗi đợt 1 sản phẩm (bao gồm cả trường hợp 2 KCS bấm duyệt đồng thời).
   * *Kết quả chấp nhận:* Khóa `UPDLOCK` trên `OrderLine` bảo đảm tiền hoàn 3 đợt lần lượt là `33.334đ`, `33.334đ`, `33.333đ` ($\sum = 100.001\text{đ}$ chính xác tuyệt đối).
7. **Đa Kho – Split Shipment & Đối Soát COD 3 Lớp (`BD-03`, `CORR-07`):**
   * *Kết quả chấp nhận:* Phân bổ đúng kho, thu tiền và đối soát độc lập từng kiện qua `CODCollection` $\to$ `CODRemittance` $\to$ `CODReconciliation`.
8. **Tính Nguyên Tử Khi Nhập Kho & Giá Vốn WAC (`CORR-08`):**
   * *Kết quả chấp nhận:* Lỗi ở dòng thứ 9 làm Rollback 100% cả 10 dòng; WAC chỉ tính trên `AcceptedQty` của tồn kho `SELLABLE`.
9. **Kiểm Chứng Cưỡng Chế `REVOKE DML` & Phân Quyền RBAC (`CORR-04`):**
   * *Kết quả chấp nhận:* Mọi lệnh `INSERT/UPDATE/DELETE` trực tiếp từ tài khoản `retailcore_app` lên `SalesOrder`/`Payment`/`AuditLog` đều bị SQL Server chặn đứng (`SQL Error 229`); khách hàng gọi API Admin nhận `403 Forbidden`.
10. **Bảo Toàn Bất Biến Lịch Sử Giá, Thuế & JSON Snapshot (`CORR-09`):**
    * *Kết quả chấp nhận:* Thay đổi giá/tên/thuộc tính trong Catalog không làm đổi `OrderLine` của các đơn cũ.

---

## 2. MẪU KỊCH BẢN K6 ĐA NGƯỜI DÙNG (MULTI-CUSTOMER POOL)

```javascript
import http from 'k6/http';
import { check } from 'k6';
import { SharedArray } from 'k6/data';

// Bắt buộc nạp danh sách 200 tài khoản Customer độc lập (không dùng chung 1 JWT!)
const users = new SharedArray('customers', () => JSON.parse(open('./seed_customers_200.json')));

export const options = {
  scenarios: {
    flash_sale_spike: {
      executor: 'per-vu-iterations',
      vus: 200,
      iterations: 1,
      maxDuration: '30s',
    },
  },
};

export default function () {
  const user = users[__VU - 1];
  const payload = JSON.stringify({
    cartId: user.activeCartId,
    shippingAddressId: user.defaultAddressId,
    paymentMethodCode: 'VNPAY',
  });

  const res = http.post('http://localhost:8080/api/orders', payload, {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${user.jwt}`,
      'Idempotency-Key': `IDEMP-${user.accountId}-${__ITER}`,
    },
  });

  check(res, {
    'status is 201 (Success) or 409/422 (Out of Stock)': (r) =>
      r.status === 201 || r.status === 409 || r.status === 422,
  });
}
```

---

## 3. INTEGRATION TESTING VỚI TESTCONTAINERS & DUAL DB USERS

* **Tuyệt đối không dùng H2 in-memory.** Sử dụng `MSSQLServerContainer` (`mcr.microsoft.com/mssql/server:2022-latest` với `MSSQL_PID=Developer`).
* Khởi tạo database với script tạo 2 login: `flyway_migrator` cho `spring.flyway.*` và `retailcore_app` (đã thu hồi quyền DML trực tiếp theo `CORR-04`) cho `spring.datasource.*` để kiểm chứng thực chất quyền hạn CSDL và Hibernate `@Immutable`.
