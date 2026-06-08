# Tổng kết: Hoàn thành tính năng Dịch Tài liệu

Tôi đã triển khai xong tính năng **Dịch Tài liệu** theo đúng giao diện và yêu cầu bạn đưa ra. Dưới đây là các thay đổi đã thực hiện:

## 1. Giao diện (Frontend)
- Đã thêm tab **Tài liệu** bên cạnh "Văn bản" và "Hình ảnh".
- Cung cấp giao diện **Kéo thả hoặc duyệt file** hỗ trợ các định dạng `.txt`, `.docx`, `.pdf`.
- Khi tải file lên thành công, hệ thống sẽ gửi file đến backend để trích xuất chữ và dịch tự động.
- Sau khi dịch xong, giao diện sẽ hiển thị ô trạng thái chứa tên file, dung lượng và 2 nút bấm:
  - **Tải bản dịch xuống**: Cho phép tải nhanh nội dung đã dịch thành một file `.txt` trực tiếp xuống máy.
  - **Mở bản dịch**: Lập tức chuyển sang tab **Văn bản**, đưa toàn bộ nội dung gốc vào ô nguồn và nội dung dịch vào ô kết quả để bạn dễ dàng đối chiếu, chỉnh sửa và nghe phát âm.
- Có nút `X` để xóa file hiện tại và tải file khác.

## 2. API Chuyển tiếp (Java Backend)
- Thêm endpoint `POST /api/document` (dạng `multipart/form-data`) trong `TranslateController` và `TranslateService` để nhận file từ trình duyệt và chuyển tiếp an toàn sang Python backend.
- Tạo DTO `DocumentResponse` để hứng dữ liệu trả về gồm `originalText` và `translatedText`.

## 3. Xử lý Trích xuất & Dịch thuật (Python Backend)
- Cập nhật thư viện `python-docx` và `PyMuPDF` vào `requirements.txt`.
- Tạo module xử lý riêng `app/inference/document.py` với khả năng đọc và bóc tách chữ từ PDF, DOCX, TXT.
- Cài đặt chia nhỏ văn bản (nếu quá dài) và gọi model dịch tuần tự/gộp để tránh quá tải.
- Cung cấp API nội bộ `POST /internal/document` để tương tác với Java backend.

> [!IMPORTANT]
> **Hướng dẫn khởi động lại hệ thống:**
> Do có thêm thư viện mới ở Python, bạn cần đảm bảo chạy lệnh cài đặt trước khi khởi động lại backend Python:
> ```bash
> cd python-inference
> pip install -r requirements.txt
> ```
> Sau đó hãy chạy lại cả Java Backend, Python Inference và Frontend để trải nghiệm tính năng mới nhé!
