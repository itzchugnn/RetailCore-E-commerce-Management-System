---
name: retailcore-physical-optimization
description: >-
  Use when designing physical database storage, table partitioning, advanced indexing, filegroups, and performance tuning for SQL Server 2022 in RetailCore (Milestone 4).
  Guides capacity sizing, clustered columnstore indexes, filtered indexes, execution plan optimization, and HA/DR strategy.
---

# RetailCore Physical Database Design & Optimization Skill

Bộ quy chuẩn hướng dẫn thiết kế CSDL mức Vật lý, phân vùng bảng, tối ưu chỉ mục và kiến trúc lưu trữ cho **Microsoft SQL Server 2022+ Enterprise Edition** trong hệ thống **RetailCore** (Milestone 4).

---

## 1. DỰ PHÓNG DUNG LƯỢNG & SIZING (CAPACITY PLANNING)

Hệ thống được thiết kế đáp ứng tải dự phóng 3 năm:
* **Tải giao dịch:** 1.000.000 đơn hàng/năm, 10.000.000 biến động kho/năm, 50.000.000 dòng audit log/năm.
* **Nguyên tắc tính kích thước trang (8KB Data Page):**
  * Dung lượng hữu dụng mỗi trang: $8.060\text{ bytes}$ (sau khi trừ page header 96 bytes và row offset).
  * Kích thước dòng trung bình $\text{RowSize}$ bao gồm: fixed data, null bitmap, variable-length column offset array.
  * Số dòng trên một trang: $\text{RowsPerPage} = \lfloor 8.060 / (\text{RowSize} + 2) \rfloor$.
  * Dung lượng bảng: $\text{DataPages} = \lceil \text{TotalRows} / \text{RowsPerPage} \rceil \times 8\text{ KB}$.
  * Dự phòng chỉ mục B-Tree (Index Space): Ước tính bằng $25\% - 35\%$ dung lượng Data Space.

---

## 2. CHIẾN LƯỢC PHÂN VÙNG BẢNG (TABLE PARTITIONING SCHEME)

Phân vùng ngang theo thời gian (Horizontal Range Partitioning) bắt buộc áp dụng cho 3 bảng có tốc độ tăng trưởng lớn nhất:
1. `SalesOrder` & `OrderLine` (phân vùng theo `OrderDate` theo quý/năm).
2. `InventoryTransactionLine` (phân vùng theo `CreatedAt` theo tháng/quý).
3. `AuditLog` (phân vùng theo `ActionAt` theo năm).

### Mẫu Partition Function & Scheme:
```sql
-- 1. Partition Function theo Quý
CREATE PARTITION FUNCTION pf_SalesOrder_Quarterly (DATETIMEOFFSET(7))
AS RANGE RIGHT FOR VALUES 
(
    '2026-01-01T00:00:00+07:00',
    '2026-04-01T00:00:00+07:00',
    '2026-07-01T00:00:00+07:00',
    '2026-10-01T00:00:00+07:00',
    '2027-01-01T00:00:00+07:00'
);

-- 2. Partition Scheme ánh xạ vào Filegroups
CREATE PARTITION SCHEME ps_SalesOrder_Quarterly
AS PARTITION pf_SalesOrder_Quarterly
TO (FG_ARCHIVE, FG_2026_Q1, FG_2026_Q2, FG_2026_Q3, FG_2026_Q4, FG_2027_Q1);
```

* **Cơ chế Partition Switching (Archive):** Khi kết thúc chu kỳ lưu trữ hoạt động (12–24 tháng), dữ liệu phân vùng cũ được chuyển tức thì sang bảng đệm lưu trữ lạnh (`ALTER TABLE ... SWITCH PARTITION ... TO ArchiveTable`) mà không gây khóa bảng hay ảnh hưởng đến giao dịch hiện hành.

---

## 3. CHIẾN LƯỢC THIẾT KẾ CHỈ MỤC CHUYÊN SÂU (INDEXING STRATEGY)

1. **Clustered Index:**
   * Luôn gắn với Surrogate Key `BIGINT IDENTITY` đơn điệu tăng dần để chống tràn trang (Page Split) và giảm độ phân mảnh phân tán (Index Fragmentation).
2. **Covering Nonclustered Indexes (`INCLUDE`):**
   * Các truy vấn tra cứu đơn hàng và sản phẩm tần suất cao phải được bao phủ đầy đủ các cột trong `SELECT` để triệt tiêu hoàn toàn chi phí `Key Lookup`.
   ```sql
   CREATE NONCLUSTERED INDEX IX_Order_Customer_Status
   ON dbo.SalesOrder (CustomerID, OrderStatus)
   INCLUDE (OrderNumber, TotalAmount, OrderDate, DeliveredAt);
   ```
3. **Filtered Nonclustered Indexes:**
   * Áp dụng triệt để cho các trạng thái hoạt động chiếm tỷ lệ nhỏ trong CSDL để giảm kích thước B-Tree tới $90\%$:
     * Đơn hàng đang xử lý: `WHERE OrderStatus IN ('CONFIRMED', 'PROCESSING')`.
     * Dòng giữ hàng hiệu lực: `WHERE ReservationLineStatus = 'ACTIVE'`.
     * Sự kiện Webhook chưa đối soát: `WHERE ProcessingStatus = 'RECEIVED'`.
4. **Clustered Columnstore Indexes:**
   * Bắt buộc sử dụng cho các bảng phân tích báo cáo và snapshot chốt số liệu: `DailySalesSummary`, `InventoryDailySnapshot`, `ReportSnapshot`.
   * Tỷ lệ nén dữ liệu từ 5x–10x, tăng tốc độ truy vấn tổng hợp `SUM`, `AVG`, `COUNT` lên gấp 10–50 lần so với B-Tree truyền thống.

---

## 4. QUY HOẠCH FILEGROUPS VẬT LÝ (STORAGE ARCHITECTURE)

Tách biệt các luồng I/O trên đĩa cứng vật lý:
* `PRIMARY`: Chỉ lưu system metadata, catalog tables nhỏ.
* `FG_DATA_HOT`: Ổ cứng NVMe SSD tốc độ cao nhất, lưu trữ các phân vùng dữ liệu đang phát sinh giao dịch thường xuyên (dữ liệu trong năm hiện tại).
* `FG_DATA_COLD`: Ổ cứng SSD tiêu chuẩn, lưu trữ các phân vùng lịch sử cũ.
* `FG_INDEX`: Tách riêng chỉ mục Nonclustered để tránh nghẽn I/O giữa đọc chỉ mục và ghi dữ liệu.
* `LDF (Transaction Log)`: Bắt buộc đặt trên phân vùng đĩa vật lý riêng biệt với cấu hình ghi tuần tự tốc độ cao.

---

## 5. CHIẾN LƯỢC SẴN SÀNG CAO & SAO LƯU PHỤC HỒI (HA/DR)

* **SLA Mục tiêu:** RPO < 5 phút (mất mát dữ liệu tối đa), RTO < 30 phút (thời gian phục hồi hệ thống).
* **High Availability:** Triển khai SQL Server Always On Availability Groups (1 Primary Read-Write Replica, 1 Synchronous Secondary Read-Only Replica phục vụ báo cáo và kiểm toán).
* **Backup Schedule:**
  * Full Backup: Định kỳ hàng tuần vào Chủ nhật lúc 01:00 AM.
  * Differential Backup: Định kỳ hàng ngày lúc 01:00 AM.
  * Transaction Log Backup: Mỗi 10 phút/lần xuyên suốt 24/7.
  * Point-in-Time Recovery: Diễn tập kịch bản phục hồi dữ liệu về thời điểm chính xác trước sự cố.

