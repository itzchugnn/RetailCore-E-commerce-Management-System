# M2.05 - CONCEPTUAL DESIGN DECISIONS & ARCHITECTURAL PRINCIPLES

* **Milestone:** M2 – Conceptual Database Design
* **Phiên bản:** 1.0
* **Trạng thái:** Approved Decisions
* **Kế thừa trực tiếp:** BD-01 đến BD-07 (M1)

---

## 1. Ba Quyết định Kỹ thuật Trọng tâm Bước vào M2 (Không thể Vi phạm)

### A. `OrderAllocation` là thực thể ứng viên cốt lõi cho Split Shipment
* **Vấn đề thực tế:** Một đơn hàng `ORD001` gồm Áo khoác (Kho Quận 1) và Bàn làm việc (Kho Thủ Đức). Hoặc 1 dòng `OrderLine` đặt 5 cái nhưng Kho A chỉ có 3 và Kho B có 2.
* **Quyết định thiết kế:** Bắt buộc có lớp phân bổ trung gian `OrderAllocation`.
* **Cơ chế liên kết:** `OrderAllocation` là cầu nối chuẩn xác kết nối giữa:
  * Yêu cầu thương mại của khách hàng (`OrderLine`).
  * Thực tế kho cung ứng (`Warehouse`).
  * Cam kết giữ hàng vật lý (`StockReservation`).
  * Kiện hàng thực tế xuất đi (`ShipmentLine`).
* Nếu thiếu `OrderAllocation`, mô hình sẽ bị vướng vào bế tắc khi cố gắng nối `OrderLine` trực tiếp với `Warehouse` hoặc `ShipmentLine`.

### B. Tách bạch 4 khái niệm Tồn kho & Vận hành (Không được gộp)
Tuyệt đối không được gộp 4 khái niệm sau thành một thực thể duy nhất:
1. **`OrderAllocation`**: Đơn hàng được **phân bổ** bao nhiêu hàng từ kho nào (Cam kết kế hoạch).
2. **`StockReservation`**: Lượng hàng đang được **giữ tạm thời tại kho** trước khi xuất kho (Khóa logic tồn kho có hạn `ExpiresAt`).
3. **`Shipment`**: Một **lần giao hàng** vật lý từ một kho cụ thể (Vận đơn, đối tác vận chuyển, tracking).
4. **`StockMovement`**: **Sự kiện ghi sổ kế toán kho** làm thay đổi tồn kho vật lý (`OnHandQty`) bất biến.

### C. Giao hàng thất bại KHÔNG đồng nghĩa hàng đã quay về kho
* **Nguyên tắc cốt lõi:** Khi shipper báo giao thất bại, hàng vẫn đang nằm trên thùng xe hoặc tại bưu cục chuyển phát. Hàng **chưa hề** quay lại giá kệ của kho.
* **Vòng đời 5 bước của tồn kho:**
  1. **Tạo đơn hàng:** `OnHandQty` giữ nguyên; `ReservedQty` tăng; $\text{AvailableQty} = \text{OnHandQty} - \text{ReservedQty}$.
  2. **Xuất kho bàn giao vận chuyển (`SHIPMENT_OUT`):** `OnHandQty` giảm; `ReservedQty` giảm. Ghi nhật ký `StockMovement` loại `SHIPMENT_OUT`. Lượt reservation tại kho được tiêu thụ/kết thúc. Hàng rời khỏi kho được theo dõi qua `Shipment`, không nằm trong `OnHandQty` và không nằm trong `ReservedQty`.
  3. **Giao hàng thất bại:** Cập nhật trạng thái `Shipment` thành `DELIVERY_FAILED`. **Chưa tăng tồn kho.**
  4. **Kho xác nhận nhận hàng (`DELIVERY_RETURN_IN`):** Nhân viên kho kiểm tra kiện hàng hoàn về, ký biên bản tiếp nhận `DeliveryReturnReceipt`. Hệ thống ghi `StockMovement` loại `DELIVERY_RETURN_IN` và tăng `OnHandQty` theo số lượng thực tế nhận lại.
  5. **Giao lại hoặc Hủy đơn:**
     * *Giao lại:* Tạo `OrderAllocation` và `StockReservation` mới theo tồn kho hiện hành trước khi xuất lần 2.
     * *Hủy đơn:* Kích hoạt quy trình `Refund` và hoàn tất đơn hàng.

---

## 2. Giải quyết 10 Câu hỏi Thiết kế Trọng yếu (M2-Q01 đến M2-Q10)

| ID | Câu hỏi thiết kế | Kết luận & Quyết định kỹ thuật |
| :---: | :--- | :--- |
| **M2-Q01** | `ProductVariant` có phải là thực thể độc lập không? | **Có.** `ProductVariant` (SKU) có surrogate key riêng (`VariantID`), định danh duy nhất toàn hệ thống bằng `SKU`. Giá bán, tồn kho, giữ kho và dòng đơn hàng đều gắn với `ProductVariant`, không gắn với `Product`. |
| **M2-Q02** | `ProductAttribute` có cần tách khỏi `Attribute` không? | **Cần tách.** `Attribute` là từ điển thuộc tính toàn cục (Color, Size). `ProductAttribute` xác định sản phẩm cụ thể áp dụng thuộc tính nào (ngăn chặn gán Size cho Đồ gia dụng). |
| **M2-Q03** | `OrderAllocation` có cần tồn tại độc lập không? | **Bắt buộc có.** Đóng vai trò thực thể trung gian phân bổ chi tiết số lượng của 1 `OrderLine` cho từng kho vật lý cụ thể (phục vụ Split Shipment). |
| **M2-Q04** | `StockReservation` liên hệ với `OrderLine` hay `OrderAllocation`? | **Liên hệ với `OrderAllocation`** (hoặc cặp `OrderLineID + WarehouseID`). Tồn kho vật lý luôn gắn với một kho cụ thể, do đó lượt giữ hàng phải gắn đích danh vào kho đã phân bổ. |
| **M2-Q05** | `ShipmentLine` liên hệ với `OrderLine` hay `OrderAllocation`? | **Liên hệ với `OrderAllocation`.** Mỗi `Shipment` xuất phát từ 1 kho và sẽ tiêu thụ trực tiếp lượng hàng đã phân bổ và giữ tại kho đó. |
| **M2-Q06** | `StockMovement` có cần tham chiếu chứng từ nguồn không? | **Bắt buộc có.** Cần `ReferenceType` (`GOODS_RECEIPT`, `SHIPMENT_OUT`, `TRANSFER`, `STOCK_ADJUST`, `DELIVERY_RETURN_IN`) và `ReferenceID` để đối soát kiểm toán kho bất biến. |
| **M2-Q07** | `Promotion` có cần chuyên biệt hóa theo tầng không? | **Có.** Áp dụng mô hình kế thừa (Generalization): Bảng cha `Promotion` (thời gian, tầng khuyến mãi); Bảng con `Voucher` (mã code, hạn mức dùng, cờ `IsStackable`). |
| **M2-Q08** | `ReturnRequest` có liên hệ với `Shipment`? | **Không gắn cứng.** Giao thất bại được xử lý tự động qua `Shipment` $\rightarrow$ `DeliveryReturnReceipt`. Còn `ReturnRequest` phục vụ khiếu nại của khách sau khi đã nhận hàng thành công (`OrderLine`). |
| **M2-Q09** | `CustomerAddress` có cần snapshot trong `Order`? | **Bắt buộc snapshot 100%.** Địa chỉ giao hàng tại thời điểm đặt phải được sao chép nguyên văn vào `Order` để tránh việc khách đổi sổ địa chỉ làm sai lệch lịch sử đơn cũ. |
| **M2-Q10** | `Payment` và `Order` có quan hệ 1–1 hay 1–N? | **Quan hệ 1–N.** Một đơn hàng có thể có nhiều lần thử thanh toán (`PaymentAttempt` thất bại/thành công) và nhiều giao dịch hoàn tiền (`Refund`). |

