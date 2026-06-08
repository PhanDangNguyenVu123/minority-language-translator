# Thêm Tính Năng Đọc Phát Âm Tiếng Việt (Text-to-Speech)

Mục tiêu là thêm một nút bấm hình chiếc loa vào ô nhập liệu (hoặc kết quả dịch) tiếng Việt. Khi người dùng bấm vào, ứng dụng sẽ đọc to văn bản tiếng Việt lên giống như Google Dịch.

## Sự Nhầm Lẫn Giữa "Nhận dạng" và "Phát âm" (Vui lòng đọc kỹ)

> [!WARNING]
> **Code của bạn bạn làm là STT (Nhận diện giọng nói)**
> File mà bạn của bạn thêm vào dự án là dùng để huấn luyện mô hình **nghe** giọng người và chuyển thành **chữ viết**. Nó phục vụ cho nút Micro mà bạn của bạn vừa thêm vào frontend (nói vào mic để nhập chữ). Nó **hoàn toàn không thể** dùng để đọc văn bản thành tiếng.

> [!TIP]
> **Bạn không cần phải Train AI cho việc đọc Tiếng Việt!**
> Để ứng dụng đọc được văn bản Tiếng Việt (Text-to-Speech), cách tối ưu, xịn và chuẩn xác nhất giống hệt Google Dịch là sử dụng trực tiếp **Web Speech API** có sẵn trên các trình duyệt (Chrome, Safari, Edge). Trình duyệt đã có sẵn một "cô gái/chàng trai AI" đọc tiếng Việt vô cùng tự nhiên miễn phí. Việc dùng API này có 2 lợi ích khổng lồ:
> 1. Không tốn bất kỳ tài nguyên server/GPU nào để chạy model.
> 2. Bạn **KHÔNG CẦN phải train model**, tiết kiệm cho bạn hàng tuần lễ học tập và chuẩn bị dữ liệu cực kỳ phức tạp (như VITS hay FastSpeech).

## Đề Xuất Của Tôi (Proposed Changes)

Tôi sẽ lập tức thêm tính năng này ngay trên giao diện web (Frontend) mà không cần bạn phải tốn công train mô hình AI nào cả.

### Frontend (`frontend/src/App.tsx` & `frontend/src/App.css`)
1. **Thêm Icon Loa (Speaker Icon):** Tạo một icon hình chiếc loa bằng SVG.
2. **Nút "Đọc phát âm":** Thêm nút bấm này vào trong khung nhập văn bản/kết quả dịch. Nút này sẽ chỉ hiện ra nếu ngôn ngữ đang được chọn là Tiếng Việt (`vi`).
3. **Logic xử lý (`speechSynthesis`):** Khi bấm vào nút loa, React sẽ lấy text hiện tại và gọi hàm `window.speechSynthesis.speak()` với ngôn ngữ là `vi-VN`.
4. **Hiệu ứng:** Nút loa sẽ có hiệu ứng hoạt hình gợn sóng hoặc đổi màu trong lúc đang phát âm.

### Backend (`python-inference`)
- Không thay đổi gì ở Backend. Việc đọc âm thanh sẽ do thiết bị của người dùng (điện thoại/máy tính) tự xử lý tại chỗ, giúp ứng dụng của bạn phản hồi siêu nhanh (0 độ trễ).

## User Review Required

Bạn có đồng ý với giải pháp sử dụng **Web Speech API** (tính năng đọc có sẵn của trình duyệt) thay vì phải tự tay train một mô hình AI từ đầu không? Hãy cho tôi biết để tôi tiến hành sửa code luôn nhé! Lựa chọn này sẽ giúp app của bạn hoạt động ngay lập tức chỉ sau 5 phút.
