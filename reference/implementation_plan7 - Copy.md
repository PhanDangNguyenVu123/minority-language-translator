# Tích hợp Hệ thống Người dùng, Cơ sở dữ liệu và Tính năng Đánh giá / Chỉnh sửa bản dịch

Kế hoạch này phác thảo cách chúng ta sẽ chuyển đổi ứng dụng từ việc chỉ dùng `localStorage` sang có một hệ thống Cơ sở dữ liệu hoàn chỉnh để quản lý người dùng, đồng bộ hóa dữ liệu (History, Favorites) và bổ sung tính năng Edit (đề xuất bản dịch) / Vote (đánh giá bản dịch).

> [!IMPORTANT]
> **User Review Required**
> Vui lòng xem kỹ cấu trúc Database bên dưới, đặc biệt là phần `translation_edits` và `translation_votes`. Đồng thời, xác nhận loại Database bạn muốn dùng (MySQL hay PostgreSQL). Trong đồ án, MySQL là phổ biến nhất.

## Open Questions

1. **Database Engine:** Bạn muốn sử dụng cơ sở dữ liệu nào cho Backend (MySQL, PostgreSQL, hay H2 in-memory tạm thời)? Mình đề xuất dùng **MySQL** cho chuyên nghiệp.
2. **Xác thực người dùng (Authentication):** Vì là đồ án, chúng ta có thể làm hệ thống JWT Authentication (JSON Web Token) bằng Spring Security. Bạn có đồng ý triển khai JWT không hay chỉ cần Login đơn giản (trả về user object/id)? (Khuyến nghị dùng JWT).
3. **Mật khẩu:** Bảng `users` do bạn phác thảo thiếu trường `password`. Mình đã bổ sung trường này để người dùng có thể đăng nhập. Bạn có muốn thêm các thông tin như `avatar` không?

## Proposed Changes

### 1. Database Schema Update

Chúng ta sẽ có 5 bảng chính trong cơ sở dữ liệu:

```sql
-- 1. Bảng Users
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, -- Bổ sung mật khẩu (sẽ hash bằng BCrypt)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Bảng Lịch sử dịch
CREATE TABLE translation_history (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    source_lang VARCHAR(10) NOT NULL,
    target_lang VARCHAR(10) NOT NULL,
    original_text TEXT NOT NULL,
    translated_text TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 3. Bảng Yêu thích / Sổ tay từ vựng
CREATE TABLE favorites (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    history_id BIGINT,
    source_lang VARCHAR(10) NOT NULL,
    target_lang VARCHAR(10) NOT NULL,
    original_text TEXT NOT NULL,
    translated_text TEXT NOT NULL,
    note VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (history_id) REFERENCES translation_history(id) ON DELETE SET NULL
);

-- 4. Bảng Đề xuất chỉnh sửa (Edits)
CREATE TABLE translation_edits (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    source_lang VARCHAR(10) NOT NULL,
    target_lang VARCHAR(10) NOT NULL,
    original_text TEXT NOT NULL,
    suggested_translation TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 5. Bảng Đánh giá bản dịch (Votes)
CREATE TABLE translation_votes (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    source_lang VARCHAR(10) NOT NULL,
    target_lang VARCHAR(10) NOT NULL,
    original_text TEXT NOT NULL,
    translated_text TEXT NOT NULL,
    vote_type ENUM('UPVOTE', 'DOWNVOTE') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_vote (user_id, source_lang, target_lang, original_text(255)) -- Mỗi user chỉ được vote 1 lần cho 1 câu
);
```

---

### Backend (Java Spring Boot)

Cập nhật `backend-java` để hỗ trợ cơ sở dữ liệu và API RESTful.

#### [NEW] `pom.xml` Dependencies
Thêm các thư viện:
- `spring-boot-starter-data-jpa`
- `mysql-connector-j` (hoặc PostgreSQL)
- `spring-boot-starter-security` và `java-jwt` (cho phần Login bằng Token).

#### [NEW] Entities
Tạo các class Entity tương ứng với 5 bảng trên (`User`, `TranslationHistory`, `Favorite`, `TranslationEdit`, `TranslationVote`).

#### [NEW] Repositories
Tạo các interface JpaRepository để thao tác với DB.

#### [NEW] Services & Controllers
- `AuthController`: Các endpoint `/api/auth/login`, `/api/auth/register`.
- `SyncController`: Endpoint `/api/sync-guest-data` để nhận dữ liệu từ LocalStorage đẩy lên.
- `HistoryController`, `FavoriteController`: Quản lý CRUD.
- `FeedbackController`: Endpoint `/api/feedback/edit` và `/api/feedback/vote`.

---

### Frontend (React)

Cập nhật giao diện và luồng dữ liệu (State Management).

#### [MODIFY] Authentication Context & State
- Xây dựng form Đăng ký / Đăng nhập.
- Quản lý trạng thái đăng nhập (lưu JWT Token vào `localStorage` hoặc Cookie).

#### [MODIFY] Synchronization Logic
- Xử lý sự kiện sau khi đăng nhập thành công:
  1. Gom dữ liệu `history` và `favorites` từ `localStorage`.
  2. Gửi request `POST /api/sync-guest-data`.
  3. Xóa dữ liệu cũ ở `localStorage` và fetch dữ liệu mới từ Database.
- Cập nhật logic các thành phần liên quan để đọc/ghi qua API nếu có Token, nếu không thì dùng `localStorage`.

#### [MODIFY] Translation Box Component
- Thêm nút **Edit** và **Voting Translation** (như ảnh thiết kế) ở góc dưới của ô kết quả dịch.
- Khi người dùng chưa đăng nhập bấm vào Edit/Vote: Hiện Modal yêu cầu "Bạn cần đăng nhập để sử dụng tính năng này".
- Nếu đã đăng nhập:
  - **Edit**: Mở một dialog nhỏ cho phép nhập bản dịch tốt hơn và gửi lên DB.
  - **Voting Translation**: Cho phép đánh giá (Like/Dislike).

## Verification Plan

### Automated Tests
1. Unit test logic đồng bộ dữ liệu: Đảm bảo dữ liệu LocalStorage không bị mất hoặc bị double khi đẩy lên Database.
2. Kiểm tra JWT token generation ở Backend.

### Manual Verification
1. Trải nghiệm luồng **Khách**:
   - Dịch thử, lịch sử lưu vào `localStorage`.
   - Bấm nút Edit -> Hệ thống yêu cầu đăng nhập.
2. Trải nghiệm luồng **Đăng ký / Đăng nhập**:
   - Đăng nhập thành công -> Theo dõi tab Network, đảm bảo lệnh gọi `sync-guest-data` chạy đúng.
   - Refresh trang web, lịch sử vẫn tồn tại (load từ DB).
3. Đánh giá bản dịch:
   - Thử Upvote / Downvote và Đề xuất dịch -> Kiểm tra bảng trong Database có lưu đúng thông tin không.
