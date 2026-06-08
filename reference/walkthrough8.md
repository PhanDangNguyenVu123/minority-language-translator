# Báo cáo Cập nhật Trang Quản trị (Admin Dashboard)

Mình đã hoàn thành việc xây dựng **Admin Dashboard** theo phương pháp đơn giản nhất (chuyển đổi giao diện trực tiếp trên React State mà không làm phức tạp hóa dự án với React Router).

## Các thay đổi chính

1. **Cơ sở dữ liệu (Backend):**
   - Đã thêm cột `role` vào bảng `users` (mặc định là `ROLE_USER`).
   - Cập nhật cơ chế Authentication (JWT) để trả về quyền hạn của User.
   - Thêm bộ API thống kê (`/api/admin/stats` và `/api/admin/recent-feedback`).
   - Tất cả API Admin đều được bảo mật tuyệt đối với annotation `@PreAuthorize("hasAuthority('ROLE_ADMIN')")`.

2. **Giao diện (Frontend):**
   - Tích hợp biểu đồ thống kê bằng thư viện `recharts`.
   - Tạo nút **"👑 Admin Panel"** trên thanh công cụ (chỉ hiện ra khi user là Admin).
   - Khi bấm vào nút này, giao diện Dịch thuật sẽ được ẩn đi và thay bằng bảng điều khiển của Admin.
   - Admin có thể theo dõi tỷ lệ Upvote/Downvote dưới dạng biểu đồ tròn.
   - Admin cũng xem được các phản hồi Downvote hoặc các đề xuất dịch mới nhất dạng bảng biểu.

## Hướng dẫn Test chức năng

Để test chức năng này, bạn cần biến tài khoản của bạn thành Admin. Do mình không có mật khẩu root của MySQL trên máy bạn, bạn hãy làm theo các bước sau:

1. **Khởi động lại Backend:**
   Hãy chắc chắn bạn đã Restart lại Spring Boot để JPA tự động thêm cột `role` vào MySQL.

2. **Cập nhật Role trong MySQL:**
   Mở phần mềm quản lý MySQL (DBeaver, MySQL Workbench, XAMPP...) chạy câu lệnh sau:
   ```sql
   UPDATE users SET role = 'ROLE_ADMIN' WHERE username = 'tên_đăng_nhập_của_bạn';
   ```
   *(Lưu ý: Thay `tên_đăng_nhập_của_bạn` bằng tài khoản bạn đang dùng).*

3. **Chạy Frontend và Đăng nhập:**
   - Mở lại giao diện Web và bấm **Đăng nhập** lại (để JWT cập nhật role mới).
   - Nút "👑 Admin Panel" sẽ xuất hiện trên thanh công cụ.
   - Nhấp vào nút đó, bạn sẽ thấy biểu đồ thống kê và danh sách phản hồi!

---

> [!TIP]
> Bạn có thể tạo thêm một tài khoản mới tinh (tự động là `ROLE_USER`) để kiểm chứng rằng người dùng bình thường sẽ không thấy nút Admin, và cũng không thể gọi API của Admin.
