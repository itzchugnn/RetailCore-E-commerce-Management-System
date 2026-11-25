---
name: retailcore-frontend-standards
description: >-
  Use when developing, structuring, and maintaining the React + TypeScript frontend web applications for RetailCore (Milestone 6).
  Guides Customer Storefront, Admin Dashboard, state management with TanStack Query and Zustand, secure auth, and UX standards.
---

# RetailCore Frontend Standards & React Architecture Skill

Bộ quy chuẩn phát triển giao diện Web bằng **React, TypeScript, Tailwind CSS** trong hệ thống **RetailCore** (Milestone 6).

---

## 1. KIẾN TRÚC HAI PHÂN HỆ GIAO DIỆN (SINGLE SPA ROUTE-SPLIT ARCHITECTURE)

Đóng gói trong một dự án Vite + React + TypeScript duy nhất tách nhánh theo Route & Lazy-loaded Code Splitting:
1. **Customer Storefront (`/*` — Cổng Khách hàng B2C):**
   * Trang chủ, Danh sách sản phẩm (Server-side Faceted Filter theo Danh mục `Category`, Thương hiệu `Brand`, Thuộc tính động `AttributeValue`).
   * Trang chi tiết sản phẩm: Hỗ trợ cả Sản phẩm có biến thể (`VARIANT_DEFINING` Selector theo Size/Màu) lẫn Sản phẩm đơn giản (1 SKU mặc định rỗng thuộc tính theo `PC-Q03`), hiển thị thông số mô tả (`DESCRIPTIVE`), giá bán nguyên đồng VNĐ và trạng thái kho.
   * Giỏ hàng, Checkout đa bước (kèm đồng hồ đếm ngược giữ hàng `ExpiresAt`: 15 phút chuẩn / 5 phút cho đơn Flash Sale, tùy chọn nhập thông tin xuất hóa đơn GTGT B2B), theo dõi đơn hàng đa kiện (Split Shipment Timeline) và yêu cầu đổi trả theo từng kiện giao.
2. **Admin Back-Office (`/admin/*` — Cổng Quản trị Vận hành):**
   * Quản lý danh mục `Brand`, `Category`, `Product`, `ProductVariant` và cấu hình thuộc tính động.
   * Quản lý kho, tra cứu số dư theo 3 phẩm chất (`SELLABLE`, `QUARANTINED`, `DAMAGED`) + giá vốn WAC.
   * Quản lý PO, phiếu nhập kho, phân bổ Split Shipment, đối soát COD 3 lớp và thẩm định KCS `ReturnLineAssessment`.

---

## 2. TECH STACK, STATE MANAGEMENT & BẢO MẬT PHIÊN

* **Core:** React 18/19, TypeScript (`strict: true`, cấm `any`), Tailwind CSS, shadcn/ui (Radix UI).
* **Server State:** `@tanstack/react-query` (Query Key Factory, caching, invalidation).
* **Client UI State:** `zustand` (chỉ lưu trạng thái giao diện như Drawer, Modal, bộ lọc tạm).
* **Bảo mật Xác thực (Chống XSS Token Theft):** Lưu Access Token ngắn hạn trong bộ nhớ RAM (memory closure) và Refresh Token trong `HttpOnly; Secure; SameSite` Cookie; **tuyệt đối không lưu JWT lâu dài trong `localStorage`**.
* **Idempotency UX khi Checkout:** Sinh UUID `Idempotency-Key` cho mỗi ý định đặt hàng/thanh toán; giữ nguyên Key khi bấm thử lại do rớt mạng và chỉ sinh Key mới khi người dùng thay đổi giỏ hàng, địa chỉ hoặc mã voucher.
* **Form & Validation:** `react-hook-form` + `zod` đồng bộ với thông báo lỗi chi tiết từ Backend.

---

## 3. TIÊU CHUẨN TRẢI NGHIỆM NGƯỜI DÙNG (UX CRITERIA)

1. **Ba Trạng Thái UI Bắt Buộc:** Mọi màn hình dữ liệu phải có `Skeleton` (Loading), `Empty State` có CTA, và `Error State` có nút Retry.
2. **Server-Side Processing:** Phân trang, sắp xếp, tìm kiếm (Debounce `300ms–500ms`) và Faceted Filter thực thi hoàn toàn phía Server.
3. **Hiển Thị Đầy Đủ Máy Trạng Thái Đơn Hàng & Split Shipment:**
   * Hiển thị rõ các trạng thái phân tách: `PENDING_PAYMENT` $\to$ `CONFIRMED` $\to$ `PROCESSING` $\to$ `PARTIALLY_SHIPPED` / `SHIPPED` $\to$ `PARTIALLY_DELIVERED` / `DELIVERED` (và `PARTIALLY_CANCELLED` / `CANCELLED`), kèm tiến trình riêng của từng kiện `Shipment`.
4. **Confirmation Dialogs:** Bắt buộc xác nhận trước các thao tác nhạy cảm (Hủy đơn, Khóa danh mục, Phê duyệt kiểm kê, Duyệt giám định hoàn tiền).
