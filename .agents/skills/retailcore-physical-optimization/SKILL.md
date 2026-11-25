---
name: retailcore-physical-optimization
description: >-
  Use when designing physical database storage, table partitioning, advanced indexing, filegroups, and performance tuning for SQL Server 2022 in RetailCore (Milestone 4).
  Guides capacity sizing, partition-aligned indexing, filtered indexes, execution plan optimization, and HA/DR strategy.
---

# RetailCore Physical Database Design & SQL Server 2022 Optimization Skill

Bộ quy chuẩn thiết kế vật lý, quy hoạch lưu trữ, phân vùng bảng hợp chuẩn Engine và tối ưu hiệu năng cho **Microsoft SQL Server 2022+** trong hệ thống **RetailCore** (Milestone 4).

---

## 1. QUY TRÌNH DỰ PHÓNG DUNG LƯỢNG BẮT BUỘC (8KB PAGE CAPACITY SIZING)

Tuyệt đối không nhân thô `RowSize * RowCount / 10^6`. Phải tính toán theo cấu trúc trang dữ liệu **8KB Data Page** (`8,060 bytes` khả dụng) của SQL Server:
1. $\text{AvgRowSize} = \text{FixedBytes} + \text{NullBitmap} \left(2 + \lfloor (\text{Cols}+7)/8 \rfloor\right) + \text{VarOverhead} \left(2 + 2 \times \text{VarCols}\right) + \text{AvgVarDataBytes} + 4\text{B Header}$.
2. $\text{RowsPerPage} = \lfloor 8060 / (\text{AvgRowSize} + 2\text{B SlotArray}) \rfloor$ (có tính hệ số `FILLFACTOR`).
3. $\text{DataPages} = \lceil \text{TotalRows} / \text{RowsPerPage} \rceil \implies \text{DataSpace (MiB)} = (\text{DataPages} \times 8) / 1024$.
4. $\text{IndexSpace (MiB)}$: Tính riêng cho từng Nonclustered Index dựa trên độ rộng khóa Index + Clustered Key Locator + `INCLUDE` columns.

---

## 2. CHIẾN LƯỢC PHÂN VÙNG BẢNG HỢP CHUẨN SQL SERVER (PARTITION-ALIGNED LAW)

> [!IMPORTANT]
> **Luật Bắt Buộc của SQL Server (Chống lỗi Msg 1908, Msg 7733, Msg 4947):**
> 1. Mọi `PRIMARY KEY` và `UNIQUE` index trên bảng phân vùng **bắt buộc phải chứa cột phân vùng (Partition Key)** trong tập cột khóa.
> 2. Lệnh `ALTER TABLE ... SWITCH PARTITION` chỉ hợp lệ khi 100% Index của bảng đều là **Partition-Aligned** (nằm trên cùng Partition Scheme, không đặt sang `FG_INDEX` tĩnh) và **không có khóa ngoại (FK) đang bật từ bảng con trỏ vào**.

Do đó, ở quy mô mục tiêu `1.000.000 đơn/năm` và `10.000.000 dòng sổ cái kho/năm`, RetailCore phân loại chiến lược lưu trữ thực tiễn như sau:

1. **Nhóm Bảng Sổ cái & Nhật ký Khổng lồ (`InventoryTransactionLine` E16, `AuditLog` E65, `PaymentWebhookLog` E71) — ÁP DỤNG RANGE PARTITIONING:**
   * Không có bảng con khác trỏ FK vào dòng chi tiết log $\implies$ Hỗ trợ `SWITCH PARTITION` sang bảng lưu trữ lạnh (`Archive`) tức thì!
   * Đưa cột thời gian vào Composite Clustered PK để đạt 100% Partition-Aligned:
     * `PK_InventoryTransactionLine PRIMARY KEY CLUSTERED (TransactionLineID, CreatedAt)` trên `ps_Ledger_Quarterly(CreatedAt)` (bổ sung cột `CreatedAt` đồng bộ với Header).
     * `PK_AuditLog PRIMARY KEY CLUSTERED (AuditLogID, ActionAt)` trên `ps_Audit_Yearly(ActionAt)`.
     * `PK_PaymentWebhookLog PRIMARY KEY CLUSTERED (WebhookLogID, ReceivedAt)` trên `ps_Webhook_Quarterly(ReceivedAt)`.
2. **Nhóm Bảng Giao dịch Có Nhiều Bảng Con Tham Chiếu (`SalesOrder` E40, `OrderLine` E41):**
   * Giữ trên Filegroup tốc độ cao `FG_OLTP_HOT` với `PK_SalesOrder CLUSTERED (OrderID)` đơn cột (`BIGINT IDENTITY`), bảo toàn `UQ_SalesOrder_OrderNumber(OrderNumber)` toàn cục và giữ nguyên toàn bộ Foreign Keys từ `OrderLine`, `OrderAddress`, `Payment`, `Shipment`, `ReturnRequest` mà không làm phình kích thước FK liên bảng (~3 triệu dòng/3 năm chỉ chiếm ~1.2 GB trên B-Tree).

---

## 3. CHIẾN LƯỢC CHỈ MỤC CHUYÊN SÂU & CHỐNG KHÓA (CONCURRENCY & INDEXING)

1. **Read Committed Snapshot Isolation (RCSI):**
   * Bắt buộc bật `ALTER DATABASE RetailCoreDB SET READ_COMMITTED_SNAPSHOT ON;` để các truy vấn đọc (`SELECT`) sử dụng Row Versioning trong `tempdb`, không bị chặn bởi khóa ghi giao dịch đặt hàng.
   * Với thao tác giữ tồn kho/duyệt hoàn tiền nhạy cảm, sử dụng khóa tường minh `WITH (UPDLOCK, ROWLOCK, HOLDLOCK)` theo đúng thứ tự `ORDER BY VariantID ASC, WarehouseID ASC`.
2. **Covering & Filtered Nonclustered Indexes:**
   * Bao phủ các truy vấn đọc tần suất cao bằng `INCLUDE (...)` để triệt tiêu `Key Lookup`.
   * Dùng Filtered Index cho tập trạng thái hoạt động nhỏ (`WHERE OrderStatus IN ('PENDING_PAYMENT', 'CONFIRMED', 'PROCESSING')`, `WHERE ReservationLineStatus = 'ACTIVE'`, `WHERE ProcessingStatus = 'RECEIVED'`).
3. **Nonclustered Columnstore / Indexed Views cho Báo cáo (`SA11`):**
   * Trên các bảng giao dịch lớn khi cần phân tích BI thời gian thực, sử dụng **Nonclustered Columnstore Index (NCCI)** có lọc hoặc lưu trữ kết quả chốt kỳ cố định qua `ReportSnapshot` (E69) (không khai báo Clustered Columnstore Index lên các bảng metadata ít dòng dưới ngưỡng nén 1.048.576 dòng/rowgroup).

---

## 4. QUY HOẠCH FILEGROUPS VẬT LÝ (STORAGE ARCHITECTURE)

* `PRIMARY`: Chỉ lưu system catalog metadata.
* `FG_MASTER`: Các bảng danh mục đọc nhiều (`SA01`, `Warehouse`, `Carrier`, `Role`).
* `FG_OLTP_HOT`: NVMe SSD tốc độ cao cho các bảng giao dịch đang hoạt động (`SalesOrder`, `OrderLine`, `InventoryBalance`, `Payment`).
* `FG_INDEX`: Dành cho Nonclustered Indexes của các **bảng không phân vùng** (`FG_MASTER`, `FG_OLTP_HOT`); riêng các bảng phân vùng phải đặt NCI ngay trên Partition Scheme tương ứng để giữ tính Aligned.
* `FG_LEDGER_Q*` / `FG_ARCHIVE`: Các filegroup theo kỳ của Partition Scheme cho sổ cái kho và nhật ký kiểm toán.
* `LDF (Transaction Log)`: Đặt trên ổ đĩa ghi tuần tự độc lập.

---

## 5. CHIẾN LƯỢC SẴN SÀNG CAO & SAO LƯU PHỤC HỒI (HA/DR)

* **SLA Mục tiêu:** **RPO < 5 phút**, **RTO < 30 phút**.
* **Phân tầng Môi trường:**
  * *Dev / CI / Demo (Docker):* Sử dụng `SQL Server 2022 Developer Edition` (đầy đủ 100% tính năng Partitioning, Columnstore, RCSI hợp lệ bản quyền phát triển).
  * *Production Enterprise Target:* Triển khai cụm `SQL Server Enterprise Always On Availability Groups` (1 Primary Read-Write, 1 Synchronous Secondary Read-Only gánh tải báo cáo `SA11`).
* **Lịch Sao lưu Nhất quán với RPO < 5 phút:**
  * Full Backup: Chủ nhật hàng tuần lúc 01:00 AM.
  * Differential Backup: Hàng ngày lúc 01:00 AM.
  * **Transaction Log Backup: Định kỳ mỗi 3–5 phút/lần xuyên suốt 24/7** (bảo đảm RPO thực tế luôn $< 5$ phút ngay cả khi phục hồi từ bản sao lưu).
