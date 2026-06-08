# Kế hoạch Triển khai Trang Admin (Dashboard & Analytics)

Tạo một bảng điều khiển (Dashboard) dành riêng cho Quản trị viên (Admin) để quản lý người dùng, phân tích dữ liệu dịch thuật và xem phản hồi của cộng đồng.

## Open Questions

> [!WARNING]
> Mình cần bạn xác nhận một số chi tiết trước khi code:
> 1. Hiện tại Frontend chưa cài đặt thư viện quản lý Router (chuyển trang) như `react-router-dom`. Để giữ cho app nhẹ và không làm vỡ cấu trúc hiện tại, mình sẽ làm Trang Admin dạng "Chuyển Tab" (Bấm nút Admin Panel trên Header thì ẩn giao diện Dịch thuật đi và hiện Giao diện Admin). Bạn có đồng ý cách này không?
> 2. Mình sẽ cài thêm thư viện `recharts` cho Frontend để vẽ biểu đồ thống kê rất đẹp mắt.
> 3. Tạm thời mình sẽ set cho User đầu tiên trong hệ thống (hoặc User do bạn chỉ định) có quyền `ROLE_ADMIN` để bạn dễ test nhé. 

## Proposed Changes

---

### Backend - Database & Security

Sửa đổi CSDL và phân quyền để hỗ trợ Role Admin.

#### [MODIFY] [User.java](file:///e:/devpro/JavaSpringBoot/translate-app/backend-java/src/main/java/com/example/translate/entity/User.java)
- Thêm thuộc tính `private String role;` (mặc định là `ROLE_USER`).
- Tạo cơ chế cấp quyền `ROLE_ADMIN` cho tài khoản quản trị.

#### [MODIFY] [UserDetailsImpl.java](file:///e:/devpro/JavaSpringBoot/translate-app/backend-java/src/main/java/com/example/translate/security/UserDetailsImpl.java)
- Ánh xạ trường `role` của Entity vào mảng `GrantedAuthority` của Spring Security để nhận diện quyền hạn khi gọi API.

---

### Backend - Admin APIs

Tạo Controller mới chuyên biệt cho Admin lấy dữ liệu thống kê.

#### [NEW] [AdminController.java](file:///e:/devpro/JavaSpringBoot/translate-app/backend-java/src/main/java/com/example/translate/controller/AdminController.java)
- Viết API `/api/admin/stats`: Trả về tổng số User, tổng số bản dịch, tổng số lượng Feedback (Vote tốt/xấu).
- Viết API `/api/admin/recent-feedback`: Trả về danh sách các câu dịch bị Downvote hoặc có Edit mới nhất để Admin theo dõi.
- Đặt `@PreAuthorize("hasRole('ADMIN')")` để chặn tuyệt đối người dùng thường gọi API này.

---

### Frontend - React Dashboard

Cài đặt thư viện và thiết kế giao diện trang Quản trị.

#### [NEW] [AdminDashboard.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/components/AdminDashboard.tsx)
- Viết Component hiển thị 3 phần:
  - **Overview Cards**: Các thẻ thống kê số lượng tổng (Users, Translations...).
  - **Charts**: Vẽ biểu đồ dạng cột/tròn thể hiện chất lượng dịch thuật dựa trên Vote (Upvote vs Downvote).
  - **Data Table**: Bảng danh sách các phản hồi (Edit/Vote) gần nhất của người dùng.

#### [MODIFY] [App.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/App.tsx)
- Thêm state `const [view, setView] = useState<'TRANSLATE' | 'ADMIN'>('TRANSLATE')`.
- Nếu User đăng nhập và có quyền `role === 'ROLE_ADMIN'`, hiển thị thêm nút "👑 Admin Panel" trên thanh Header.
- Khi bấm nút này, chuyển từ giao diện Dịch thuật sang gọi `AdminDashboard.tsx`.

#### [MODIFY] [AuthContext.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/context/AuthContext.tsx)
- Cập nhật Interface `User` để lưu thêm thông tin `role` lấy từ API Login về, giúp UI biết ai là Admin.

## Verification Plan

### Automated Tests
- Chạy lệnh `npm install recharts` để cài thư viện vẽ biểu đồ.

### Manual Verification
- Bạn (hoặc mình) sẽ đổi role của tài khoản bạn đang dùng trong Database thành `ROLE_ADMIN`.
- Đăng nhập lại -> Nút "Admin Panel" phải hiện ra.
- Truy cập Admin Panel -> Dữ liệu thống kê phải hiển thị chính xác dưới dạng biểu đồ.
- Thử dùng tài khoản thường đăng nhập -> Nút Admin Panel bị ẩn, và không thể gọi lén API của Admin.
