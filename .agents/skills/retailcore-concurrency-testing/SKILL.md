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

Hệ thống bắt buộc phải vượt qua bộ 10 kịch bản kiểm thử nghiêm ngặt trước khi nghiệm thu:

1. **Chống Bán Âm Kho Flash Sale (Anti-Overselling):**
   * *Điều kiện:* 1 SKU chỉ còn đúng 5 sản phẩm khả dụng (`AvailableQty = 5`).
   * *Kịch bản:* 200 Virtual Users (k6) cùng gửi yêu cầu Checkout mua SKU này tại cùng 1 giây.
   * *Kết quả chấp nhận:* Đúng 5 giao dịch thành công (hoặc tổng số lượng thành công đúng bằng 5). 195 giao dịch bị từ chối với thông báo hết hàng. Tuyệt đối $\text{OnHandQty} \ge 0$ và $\text{AvailableQty} = 0$. Không xảy ra Deadlock.
2. **Tranh Chấp Voucher Lượt Cuối (Voucher Quota Contention):**
   * *Điều kiện:* Voucher còn đúng 1 lượt dùng (`UsageLimit - UsedCount = 1`).
   * *Kịch bản:* 100 khách hàng cùng nhập mã voucher và bấm xác nhận đơn hàng đồng thời.
   * *Kết quả chấp nhận:* Duy nhất 1 đơn hàng được hưởng giảm giá. Các đơn còn lại báo lỗi voucher hết lượt. `UsedCount` sau cùng đúng bằng `UsageLimit`.
3. **Chống Tạo Đơn Lặp (Idempotency Verification):**
   * *Kịch bản:* Client gửi 2 request `POST /api/orders` giống hệt nhau (cùng `Idempotency-Key` và payload hash) cách nhau 50ms.
   * *Kết quả chấp nhận:* Hệ thống chỉ sinh duy nhất 1 đơn hàng trong CSDL, chỉ giữ tồn kho 1 lần; request thứ 2 nhận lại kết quả tức thời từ cache.
4. **Đứt Gãy Kết Nối Cổng Thanh Toán (Payment Recovery):**
   * *Kịch bản:* Client thanh toán thành công tại Gateway nhưng kết nối mạng về server bị timeout; Webhook gửi tới chậm sau 10 phút.
   * *Kết quả chấp nhận:* Trạng thái đơn được chuyển sang `CONFIRMED` chính xác sau khi Webhook được thẩm định chữ ký hợp lệ; không trừ kho lặp lại.
5. **Giải Phóng Tồn Kho Khi Hết Hạn / Hủy Đơn:**
   * *Kịch bản:* Hủy đơn hàng ở trạng thái `PENDING_PAYMENT` hoặc để đơn quá 15 phút.
   * *Kết quả chấp nhận:* Toàn bộ `ReservedQty` được hoàn lại cho `AvailableQty`; `OnHandQty` vật lý không bị biến động.
6. **Bảo Toàn Tiền Lẻ Hoàn Trả Từng Món (LRM & F(q) Integrity):**
   * *Kịch bản:* Đơn hàng 3 sản phẩm tổng giá trị sau giảm là 100.001đ VNĐ. Khách trả lần lượt từng sản phẩm qua 3 đợt.
   * *Kết quả chấp nhận:* Tiền hoàn 3 đợt lần lượt là 33.334đ, 33.334đ, và 33.333đ. Tổng tiền hoàn đúng 100.001đ, không sai lệch 1 đồng.
7. **Đa Kho – Split Shipment:**
   * *Kịch bản:* Đơn hàng gồm 2 sản phẩm phân bổ từ 2 kho khác nhau.
   * *Kết quả chấp nhận:* Hệ thống sinh 2 dòng `OrderWarehouseAllocation` và 2 phiếu `StockReservationLine` tại 2 kho độc lập, không trừ nhầm kho.
8. **Tính Nguyên Tử Khi Nhập Kho (PO Atomic Rollback):**
   * *Kịch bản:* Tiếp nhận phiếu nhập kho gồm 10 SKU, cố tình gây lỗi dữ liệu hoặc vi phạm ràng buộc ở dòng thứ 9.
   * *Kết quả chấp nhận:* Toàn bộ 10 dòng bị Rollback $100\%$, không có SKU nào bị cộng nhầm vào `InventoryBalance`.
9. **Bảo Mật Phân Quyền RBAC:**
   * *Kịch bản:* Khách hàng hoặc tài khoản nhân viên kho gửi token truy cập endpoint Admin `/api/admin/reports`.
   * *Kết quả chấp nhận:* Hệ thống chặn ngay lập tức và trả HTTP `403 Forbidden`.
10. **Bảo Toàn Bất Biến Lịch Sử Giá (Snapshot Integrity):**
    * *Kịch bản:* Admin sửa giá bán biến thể từ 500.000đ lên 700.000đ trong Catalog sau khi đơn hàng đã đặt.
    * *Kết quả chấp nhận:* `OrderLine.UnitPrice` của các đơn cũ giữ nguyên 500.000đ; báo cáo doanh thu lịch sử không thay đổi.

---

## 2. CÔNG CỤ & KỊCH BẢN KIỂM THỬ TẢI VỚI K6

Kịch bản mẫu kiểm thử Concurrency Checkout với k6:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  scenarios: {
    flash_sale_spike: {
      executor: 'per-vu-iterations',
      vus: 100,
      iterations: 1,
      maxDuration: '30s',
    },
  },
};

export default function () {
  const url = 'http://localhost:8080/api/orders';
  const payload = JSON.stringify({
    variantId: 101, // Flash Sale SKU
    quantity: 1,
    shippingAddressId: 1,
  });

  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer ' + __ENV.TEST_JWT_TOKEN,
      'Idempotency-Key': 'IDEMP-' + __VU + '-' + Date.now(),
    },
  };

  const res = http.post(url, payload, params);
  check(res, {
    'status is 201 or 400 (Out of stock)': (r) => r.status === 201 || r.status === 400,
  });
}
```

---

## 3. INTEGRATION TESTING VỚI TESTCONTAINERS & SQL SERVER THẬT

* **Nguyên tắc vàng:** Tuyệt đối không dùng H2 Database in-memory để kiểm thử tầng Repository/Service của RetailCore, vì H2 không hỗ trợ cú pháp T-SQL, Table Partitioning, Filtered Indexes và Stored Procedures phức tạp.
* **Sử dụng Testcontainers:**
  ```java
  @Testcontainers
  @SpringBootTest
  public class OrderConcurrencyIntegrationTest {
      @Container
      static MSSQLServerContainer<?> mssql = new MSSQLServerContainer<>("mcr.microsoft.com/mssql/server:2022-latest")
          .acceptLicense();
  
      @DynamicPropertySource
      static void properties(DynamicPropertyRegistry registry) {
          registry.add("spring.datasource.url", mssql::getJdbcUrl);
          registry.add("spring.datasource.username", mssql::getUsername);
          registry.add("spring.datasource.password", mssql::getPassword);
      }
  }
  ```

