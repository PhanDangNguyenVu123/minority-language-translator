# Hướng dẫn sử dụng tính năng Dịch Hình Ảnh

Tuyệt vời! Tôi đã hoàn tất việc viết code và tích hợp thành công tính năng dịch trực tiếp trên hình ảnh theo yêu cầu của bạn, sử dụng Tesseract OCR để nhận diện chữ.

## Những thay đổi đã được thực hiện:
1. **Giao diện người dùng (Frontend)**:
   - Thêm hai tab "Văn bản" và "Hình ảnh" để bạn có thể chuyển đổi qua lại giữa hai chế độ dịch.
   - Khi ở chế độ "Hình ảnh", một khu vực kéo-thả (hoặc click để chọn tệp) sẽ xuất hiện để bạn tải ảnh lên.
   - Khi ảnh được tải lên, ứng dụng sẽ gọi backend, nhận diện chữ, và tạo ra các khối chữ đã dịch **chèn trực tiếp lên đúng vị trí** của chữ gốc trong ảnh.
   - Có thêm công tắc **"Hiện bản gốc"** ở góc trên để ẩn các khối chữ dịch, giúp bạn dễ dàng so sánh bản gốc và bản dịch.

2. **Dịch vụ Backend (Java Spring Boot)**:
   - Đã thêm API `POST /api/ocr` để tiếp nhận ảnh và thông tin ngôn ngữ từ giao diện, sau đó chuyển tiếp sang Python.

3. **Dịch vụ AI/Inference (Python FastAPI)**:
   - Cài đặt thêm các thư viện `pytesseract`, `Pillow`, và `python-multipart` vào môi trường ảo.
   - Tạo endpoint `/internal/ocr` nhận file ảnh, dùng Tesseract (`pytesseract.image_to_data`) quét toàn bộ ảnh, nhóm các chữ cái lại thành từng dòng kèm theo **tọa độ (x, y, width, height)**.
   - Dịch từng dòng và trả về kết quả tọa độ cùng văn bản đã dịch cho Frontend hiển thị.

## Hướng dẫn kiểm tra tính năng

> [!IMPORTANT]
> **Khởi động lại các dịch vụ**
> Do đã cài đặt thêm thư viện Python và thêm class mới trong Java, bạn cần phải:
> 1. Dừng và khởi động lại dịch vụ **Python Inference** (`uvicorn app.main:app`).
> 2. Dừng và khởi động lại dịch vụ **Java Spring Boot Backend**.
> 3. Tải lại trang web (F5).

Sau khi khởi động lại:
1. Nhấp vào tab **Hình ảnh** ở góc trái trên cùng.
2. **Kéo thả** một bức ảnh có chứa tiếng Việt (hoặc tiếng Anh) vào khung tải lên.
3. Chờ một chút để hệ thống phân tích. Kết quả sẽ hiển thị ngay trên hình ảnh.
4. Gạt công tắc **Hiện bản gốc** để so sánh.

Nếu có bất kỳ vấn đề gì về việc nhận diện chữ, hãy đảm bảo rằng bạn đã chọn cài đặt "Vietnamese" lúc cài đặt Tesseract nhé. Chúc bạn trải nghiệm tính năng mới vui vẻ!
