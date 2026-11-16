# 02. STAKEHOLDER REGISTER – DANH SÁCH CÁC BÊN LIÊN QUAN

* **Milestone:** M1 – Business Analysis
* **Phiên bản tài liệu:** 1.0 – Baseline

---

## 2.1. Danh sách Stakeholder

| ID | Stakeholder | Vai trò | Mối quan tâm chính |
| :--- | :--- | :--- | :--- |
| **ST-01** | Business Owner | Chủ doanh nghiệp | Doanh thu, chi phí, hiệu quả kinh doanh |
| **ST-02** | System Administrator | Quản trị hệ thống | Tài khoản, phân quyền, bảo mật |
| **ST-03** | Marketing Staff | Nhân viên marketing | Khuyến mãi, voucher, hiệu quả chương trình |
| **ST-04** | Sales Staff | Nhân viên bán hàng | Đơn hàng, khách hàng, xử lý yêu cầu |
| **ST-05** | Warehouse Staff | Nhân viên kho | Nhập, xuất, tồn kho |
| **ST-06** | Customer | Khách hàng | Tìm kiếm, đặt hàng, thanh toán, đổi trả |
| **ST-07** | Supplier | Nhà cung cấp | Đơn mua hàng, giao hàng, công nợ |
| **ST-08** | Shipping Provider | Đơn vị vận chuyển | Nhận và giao hàng |
| **ST-09** | Payment Provider | Đơn vị thanh toán | Xử lý giao dịch thanh toán và hoàn tiền |
| **ST-10** | Development Team | Nhóm phát triển | Kiến trúc, dữ liệu, hiệu năng, bảo trì |

---

## 2.2. Stakeholder Interest Matrix (Ma trận Mối quan tâm & Ảnh hưởng)

| Stakeholder | Thông tin cần theo dõi | Mức độ ảnh hưởng |
| :--- | :--- | :--- |
| **Business Owner** | Doanh thu, lợi nhuận, tồn kho, báo cáo | Rất cao |
| **Administrator** | Người dùng, vai trò, nhật ký hoạt động | Rất cao |
| **Marketing** | Khuyến mãi, voucher, lượt sử dụng | Cao |
| **Sales Staff** | Đơn hàng, khách hàng, trạng thái thanh toán | Cao |
| **Warehouse Staff** | Số lượng tồn, phiếu nhập, phiếu xuất | Cao |
| **Customer** | Giá, tồn kho, đơn hàng, vận chuyển | Cao |
| **Supplier** | Đơn đặt hàng, số lượng giao | Trung bình |
| **Shipping Provider** | Địa chỉ, kiện hàng, trạng thái giao | Trung bình |
| **Payment Provider** | Mã giao dịch, số tiền, trạng thái | Trung bình |
| **Development Team** | Toàn bộ mô hình dữ liệu và quy trình | Rất cao |

> **Lưu ý kiến trúc:** `Supplier`, `Shipping Provider` và `Payment Provider` là các hệ thống/tác nhân bên ngoài (External Actors / Supporting Systems). Các tác nhân này không nhất thiết phải có tài khoản đăng nhập trực tiếp vào hệ thống RetailCore mà tương tác qua quy trình nghiệp vụ hoặc tích hợp API/Webhook.