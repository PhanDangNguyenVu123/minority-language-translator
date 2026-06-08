# Kế hoạch Triển khai Admin Dashboard

- `[x]` 1. Cập nhật Backend (Database & Security)
  - `[x]` Thêm trường `role` vào Entity `User.java`.
  - `[x]` Cập nhật `UserDetailsImpl.java` để ánh xạ quyền `ROLE_ADMIN`.
- `[x]` 2. Tạo API Quản trị viên (Admin API)
  - `[x]` Viết `AdminController.java`.
  - `[x]` Tạo API `/api/admin/stats` thống kê số lượng User, Translation, Feedback.
  - `[x]` Tạo API `/api/admin/recent-feedback` hiển thị danh sách phản hồi mới nhất.
  - `[x]` Bảo vệ API bằng `@PreAuthorize("hasRole('ADMIN')")`.
- `[x]` 3. Xây dựng Giao diện React
  - `[x]` Cài đặt thư viện `recharts`.
  - `[x]` Tạo component `AdminDashboard.tsx` hiển thị thẻ thống kê và biểu đồ.
  - `[x]` Cập nhật `AuthContext.tsx` để nhận biết `role` của User hiện tại.
  - `[x]` Thêm state `view` vào `App.tsx` để chuyển đổi giữa Dịch thuật và Admin Panel.
- `[/]` 4. Kiểm tra
  - `[ ]` Đăng nhập bằng tài khoản Admin để xác minh Dashboard.
  - `[ ]` Xác minh tài khoản User thường không thể truy cập Admin Panel.
