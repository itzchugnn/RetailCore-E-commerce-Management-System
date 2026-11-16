# 08. NON-FUNCTIONAL REQUIREMENTS – YÊU CẦU PHI CHỨC NĂNG (BASELINE v1.0)

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

Các yêu cầu phi chức năng dưới đây được định nghĩa dưới dạng các tiêu chí cụ thể, có thể đo lường và kiểm thử được. Các ngưỡng số liệu là mục tiêu thiết kế (Design Targets) trên quy mô dữ liệu giả định của dự án.

---

## 8.1. Performance (`NFR-PF`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-PF01** | $95\%$ truy vấn đọc thông thường có thời gian phản hồi dưới $500\text{ms}$ trong môi trường kiểm thử mục tiêu. |
| **NFR-PF02** | $95\%$ API đọc thông thường có thời gian phản hồi dưới $1\text{ giây}$, không tính thời gian xử lý của hệ thống ngoài. |
| **NFR-PF03** | API tìm kiếm sản phẩm phải hỗ trợ phân trang phía Server (Server-side Pagination). |
| **NFR-PF04** | Các truy vấn báo cáo không được gây ảnh hưởng nghiêm trọng (Lock Blocking) đến giao dịch bán hàng. |
| **NFR-PF05** | Phải có kế hoạch đánh giá Execution Plan và tối ưu Index cho các truy vấn trọng tâm. |

---

## 8.2. Security (`NFR-SE`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-SE01** | Mật khẩu phải được băm bằng thuật toán phù hợp như BCrypt hoặc Argon2. |
| **NFR-SE02** | API phải xác thực người dùng (Authentication). |
| **NFR-SE03** | Phải phân quyền theo vai trò và quyền hạn (RBAC Authorization). |
| **NFR-SE04** | Không được truy cập dữ liệu đơn hàng của khách hàng khác (Chống lỗ hổng IDOR / Broken Object Level Authorization). |
| **NFR-SE05** | Các truy vấn Database phải sử dụng cơ chế tham số hóa (Parameterized Queries) để chống SQL Injection. |
| **NFR-SE06** | Thông tin nhạy cảm không được ghi vào log một cách không kiểm soát. |
| **NFR-SE07** | Các API quan trọng phải được bảo vệ trước request giả mạo và lặp. |

---

## 8.3. Data Integrity (`NFR-DI`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-DI01** | Dữ liệu phải đảm bảo toàn vẹn tham chiếu (Referential Integrity). |
| **NFR-DI02** | Không cho phép tạo đơn hàng không hợp lệ. |
| **NFR-DI03** | Các giao dịch ảnh hưởng nhiều bảng phải đảm bảo tính nguyên tử (Atomicity). |
| **NFR-DI04** | Không được phát sinh tồn kho âm do truy cập đồng thời. |
| **NFR-DI05** | Các giao dịch quan trọng phải có khả năng truy vết (Auditability & Traceability). |
| **NFR-DI06** | Không được mất thông tin giao dịch lịch sử do thay đổi dữ liệu danh mục hiện tại. |

---

## 8.4. Concurrency (`NFR-CC`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-CC01** | Hỗ trợ nhiều khách hàng truy cập và đặt hàng đồng thời. |
| **NFR-CC02** | Việc giữ hàng phải được kiểm soát an toàn trong môi trường concurrent transaction. |
| **NFR-CC03** | Không được vượt giới hạn lượt sử dụng voucher khi nhiều người dùng áp dụng đồng thời. |
| **NFR-CC04** | Request tạo đơn và callback thanh toán phải có cơ chế chống xử lý trùng (Idempotency). |
| **NFR-CC05** | Phải kiểm thử deadlock và các tình huống tranh chấp dữ liệu. |

---

## 8.5. Reliability & Recovery (`NFR-RL`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-RL01** | Lỗi xảy ra trong transaction phải được xử lý bằng rollback phù hợp. |
| **NFR-RL02** | Có chiến lược Full Backup định kỳ kết hợp Differential / Log Backup. |
| **NFR-RL03** | Có khả năng phục hồi Database từ bản backup. |
| **NFR-RL04** | Phải kiểm thử quy trình Restore thực tế. |
| **NFR-RL05** | Hệ thống cần có cơ chế ghi log và xử lý lỗi tập trung. |

---

## 8.6. Maintainability & Usability (`NFR-MA` & `NFR-US`)

| ID | Yêu cầu |
| :--- | :--- |
| **NFR-MA01** | Backend phải được tổ chức theo module và phân lớp rõ ràng. |
| **NFR-MA02** | Database phải có quy tắc đặt tên (Naming Conventions) nhất quán cho bảng, cột, khóa, ràng buộc, index và thủ tục. |
| **NFR-MA03** | Các thay đổi schema phải được quản lý bằng công cụ migration (ví dụ: Flyway). |
| **NFR-MA04** | API phải có tài liệu mô tả (OpenAPI / Swagger). |
| **NFR-US01** | Giao diện phải responsive cho cả Desktop và Mobile. |
| **NFR-US02** | Người dùng phải nhận được thông báo rõ ràng khi thao tác thất bại. |
| **NFR-US03** | Các chức năng trên giao diện phải hiển thị theo quyền hạn phù hợp của người dùng. |