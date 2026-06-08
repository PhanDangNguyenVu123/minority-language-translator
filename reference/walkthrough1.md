# Tối Ưu Hóa RAM Cho Ứng Dụng Dịch Máy (Lazy Loading)

Chúng ta đã hoàn thành việc nâng cấp hệ thống backend để có thể hỗ trợ nhiều ngôn ngữ dịch thuật hơn mà không làm treo máy tính của bạn (hay máy chỉ dùng Card Onboard Intel Iris).

## Những Thay Đổi Chính

1. **Sử Dụng Lớp Quản Lý Mô Hình (ModelManager)**
   Thay vì tải (load) tất cả các mô hình lên RAM ngay khi hệ thống vừa khởi động, hiện tại `translator.py` chỉ định nghĩa sẵn danh sách các mô hình hiện có.
   
2. **Kỹ thuật Tải Theo Yêu Cầu (Lazy Loading) & Dọn Rác (Garbage Collection)**
   Hệ thống hiện tại sẽ kiểm tra xem người dùng đang muốn dịch từ tiếng nào sang tiếng nào:
   - Nếu bạn chọn hướng `Việt -> Ba Na`: Backend sẽ xóa mọi mô hình cũ đang có trong RAM. Sau đó, nó tự động gọi lệnh `gc.collect()` ép Python dọn dẹp RAM ngay lập tức, rồi mới tải (load) mô hình `Việt -> Ba Na` vào bộ nhớ.
   - Nếu bạn chuyển sang `Ba Na -> Việt`: Kịch bản dọn dẹp lặp lại y hệt để đảm bảo tại một thời điểm, chỉ có **duy nhất 1 mô hình nằm trong RAM**.

3. **Hỗ trợ tương thích ngược GPU**
   Dù hiện tại bạn đang dùng Intel Iris (chỉ có RAM thường chia sẻ hoặc VRAM rất nhỏ), code vẫn được chèn thêm bộ bắt lỗi an toàn (`torch.cuda.is_available()`). Nghĩa là sau này nếu bạn copy source code này mang đi báo cáo trên một máy tính mạnh hơn có card đồ họa NVIDIA rời, code vẫn sẽ chạy trơn tru và tự động dọn dẹp cả VRAM GPU.

## Cách Kiểm Tra
1. Hãy tắt app đi và bật lại từ đầu (Java & Python). 
2. Mở Task Manager lên. 
3. Thử dịch một câu "Việt -> Ba Na". Bạn sẽ thấy RAM tăng nhẹ (để load 1 mô hình). 
4. Chuyển sang dịch "Ba Na -> Việt", bạn sẽ thấy lúc đầu dịch mất vài giây để nạp model mới, nhưng dung lượng RAM trên Task Manager sẽ vẫn ổn định, không bị cộng dồn lên như trước.

## Các Bước Tiếp Theo (Thêm Ngôn Ngữ)
Bây giờ cấu trúc của app đã sẵn sàng cho bất kỳ ngôn ngữ nào. Bạn có thể huấn luyện tiếp tiếng Ê-đê hoặc Gia Rai, sau đó chỉ cần vào thư mục `models`, bỏ model mới vào, và thêm 1 dòng khai báo đường dẫn trong class `ModelManager` của file `translator.py` là xong!
