# Kế hoạch Triển khai: Chức năng Dịch Tài liệu (Document Translation)

Chức năng Dịch tài liệu sẽ cho phép người dùng upload các file tài liệu (như `.txt`, `.docx`, `.pdf`), trích xuất văn bản từ đó, thực hiện dịch và hiển thị kết quả.

## Cơ chế hoạt động (Giống với dịch ảnh)
Đúng như bạn đã dự đoán, luồng hoạt động sẽ rất giống với chức năng dịch hình ảnh:
1. **Frontend (React)**: Thêm 1 tab "Tài liệu" cùng với "Văn bản" và "Hình ảnh". Cung cấp giao diện Drag & Drop cho phép upload file.
2. **Backend (Spring Boot)**: Nhận file (Multipart form data) và forward (chuyển tiếp) sang Python backend.
3. **Backend (Python)**: Sử dụng các thư viện đọc tài liệu (ví dụ: `python-docx` cho Word, `PyMuPDF` cho PDF) để trích xuất chữ. Sau đó dịch đoạn text đã trích xuất, và trả về kết quả (Original Text & Translated Text).

## Câu hỏi Mở (Cần bạn xác nhận)

> [!IMPORTANT]
> **Về cách hiển thị kết quả dịch tài liệu**:
> Vì tài liệu có thể rất dài, chúng ta có 2 cách phổ biến để xử lý sau khi dịch xong:
> **Cách 1**: Sau khi dịch xong, **đổ toàn bộ văn bản gốc và văn bản dịch** vào 2 ô nhập/xuất chữ ở tab "Văn bản" thông thường để bạn dễ đọc và chỉnh sửa.
> **Cách 2**: Vẫn ở tab "Tài liệu", hiển thị nội dung gốc và dịch song song hoặc theo luồng trên dưới, kèm theo một nút **"Tải xuống bản dịch"** (Download file `.txt` chứa nội dung đã dịch).
> 
> *Bạn muốn làm theo Cách 1 hay Cách 2? (Hoặc kết hợp cả hai).*

## Các bước thay đổi chi tiết

### 1. Python Inference (`python-inference/`)
- Thêm thư viện `python-docx` và `PyMuPDF` vào `requirements.txt` để hỗ trợ đọc file Word và PDF.
- Tạo file `app/inference/document.py` để xử lý việc đọc file (dựa vào file extension) và trích xuất text.
- Thêm API endpoint `POST /internal/document` vào `main.py` để nhận UploadFile, trích xuất text, chia nhỏ đoạn text nếu cần, gọi hàm `translate`, và trả về chuỗi văn bản đã dịch.

### 2. Spring Boot Backend (`backend-java/`)
- Cập nhật `TranslateController` thêm endpoint `POST /api/document`.
- Cập nhật `TranslateService` thêm hàm chuyển tiếp file sang `/internal/document` của Python (tương tự như API `/api/ocr`).

### 3. React Frontend (`frontend/`)
- Thêm một tuỳ chọn `document` vào biến state `translateMode`.
- Thêm UI tab **Tài liệu** (Icon Document) cạnh tab Hình ảnh.
- Trong giao diện Tài liệu:
  - Vùng kéo thả file (hỗ trợ `.txt, .docx, .pdf`).
  - Gọi request chứa file (`FormData`) lên `/api/document`.
  - Hiển thị loading.
  - Xử lý và hiển thị kết quả trả về.

## Kế hoạch Kiểm tra (Verification Plan)
- Tạo một file `.txt` và một file `.docx` mẫu chứa văn bản tiếng Việt.
- Upload thử từng file qua giao diện mới để đảm bảo hệ thống đọc được chữ và dịch thành công sang ngôn ngữ đích.
- Xác nhận giao diện kéo thả hoạt động giống tab Hình ảnh và phản hồi đúng đắn.
