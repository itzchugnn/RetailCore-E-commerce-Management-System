---
name: retailcore-frontend-standards
description: >-
  Use when developing, structuring, and maintaining the React + TypeScript frontend web applications for RetailCore (Milestone 6).
  Guides Customer Storefront, Admin Dashboard, state management with TanStack Query and Zustand, responsive UI, and UX standards.
---

# RetailCore Frontend Standards & React Architecture Skill

Bộ quy chuẩn phát triển giao diện Web bằng **React, TypeScript, Tailwind CSS** trong hệ thống **RetailCore** (Milestone 6).

---

## 1. KIẾN TRÚC HAI PHÂN HỆ GIAO DIỆN (DUAL-PORTAL ARCHITECTURE)

Hệ thống Frontend phục vụ hai đối tượng người dùng độc lập:
1. **Customer Storefront (Cổng Khách hàng B2C):**
   * Trang chủ, Danh sách sản phẩm (Faceted Filter theo Danh mục, Thương hiệu, Thuộc tính động).
   * Trang chi tiết sản phẩm: Trình chọn biến thể linh hoạt (Variant Selector theo Size/Màu), thư viện hình ảnh, hiển thị SKU và giá bán tức thời.
   * Giỏ hàng, Checkout đa bước, Trang theo dõi tiến trình bưu kiện (Order Timeline) và hồ sơ cá nhân.
2. **Admin Dashboard (Cổng Quản trị Vận hành Back-Office):**
   * Bảng điều khiển KPI doanh thu, quản lý danh mục sản phẩm, biến thể SKU.
   * Quản lý kho, tra cứu thẻ kho số dư theo chất lượng (`Sellable`, `Quarantine`, `Damaged`).
   * Quản lý đơn mua hàng NCC, tiếp nhận kho, quản lý đơn bán hàng, phân bổ Split Shipment, đối soát COD và phê duyệt RMA.

---

## 2. QUY CHUẨN TECH STACK & QUẢN LÝ TRẠNG THÁI (STATE MANAGEMENT)

* **Core:** React 18/19, TypeScript (chế độ `strict: true`, cấm sử dụng `any`).
* **Styling & UI Components:** Tailwind CSS, Radix UI / Headless UI (hoặc shadcn/ui components), Lucide Icons.
* **Server State Management:** `@tanstack/react-query`
  * Quản lý toàn bộ dữ liệu từ API: caching, tự động refetching, pagination và optimistic updates.
  * Cấu trúc Query Keys theo Factory Pattern (ví dụ: `productKeys.detail(id)`, `orderKeys.list(filters)`).
* **Client UI State Management:** `zustand`
  * Quản lý trạng thái giao diện cục bộ: đóng/mở Giỏ hàng Drawer, Modal, bộ lọc tạm, phiên đăng nhập JWT.
* **Form & Validation:** `react-hook-form` kết hợp `zod` schema validator.

---

## 3. NGUYÊN TẮC THIẾT KẾ TRẢI NGHIỆM NGƯỜI DÙNG (UX CRITERIA)

1. **Ba Trạng Thái UI Bắt Buộc (The 3 Mandatory UI States):**
   * **Loading State:** Sử dụng Skeleton Screen khớp với layout bảng hoặc lưới sản phẩm, cấm dùng spinner toàn màn hình gây giật giao diện.
   * **Empty State:** Hiển thị hình minh họa trực quan, thông điệp rõ ràng và nút kêu gọi hành động (Call to Action).
   * **Error State:** Hiển thị thông báo lỗi thân thiện, có nút "Thử lại" (Retry), ẩn thông tin kỹ thuật sâu phía sau.
2. **Xử lý Dữ liệu Phía Server (Server-Side Processing):**
   * Phân trang, sắp xếp và lọc danh sách lớn phải thực hiện qua query parameters (`?page=1&size=20&sort=createdAt,desc`).
   * Tìm kiếm từ khóa phải có cơ chế **Debounce (300ms–500ms)** để chống spam API lên Backend.
3. **Bộ Chọn Biến Thể Sản Phẩm Động (Variant Selector UX):**
   * Dựa trên thuộc tính `VARIANT_DEFINING` của sản phẩm.
   * Tự động làm mờ (Grey out) hoặc gạch chéo các tổ hợp thuộc tính đã hết tồn kho khả dụng (`AvailableQty = 0`).
   * Khi khách chọn đủ tổ hợp thuộc tính hợp lệ, lập tức cập nhật mã SKU, giá bán tương ứng và trạng thái kho.
4. **Hộp Thoại Xác Nhận Thao Tác Nguy Hiểm (Confirmation Dialogs):**
   * Bắt buộc có Modal xác nhận trước khi: Hủy đơn hàng, Xóa/Khóa danh mục, Phê duyệt biên bản kiểm kê, Duyệt hoàn tiền.
5. **Hiển Thị Tiến Trình Đơn Hàng (Order Status Timeline):**
   * Thể hiện trực quan các mốc thời gian: Đã đặt $\to$ Đã xác nhận $\to$ Đang chuẩn bị $\to$ Đang giao $\to$ Đã giao thành công.

