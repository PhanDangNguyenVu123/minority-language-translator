# Implement Lazy Loading for Translation Models

Để tối ưu hóa dung lượng RAM và dọn đường cho việc thêm các ngôn ngữ mới sau này, chúng ta sẽ thay đổi cách load mô hình từ "Load tất cả khi khởi động" sang "Chỉ load mô hình khi cần dùng và giải phóng mô hình cũ".

## User Review Required

> [!IMPORTANT]
> - Việc tải mô hình theo yêu cầu (Lazy Loading) sẽ khiến lần dịch đầu tiên sau khi chuyển đổi ngôn ngữ mất thêm vài giây để hệ thống đọc mô hình từ ổ cứng vào RAM.
> - Nếu bạn đang dùng GPU (Card màn hình rời), chúng ta sẽ cần thêm lệnh dọn dẹp bộ nhớ VRAM của GPU (như `torch.cuda.empty_cache()`). Mặc định tôi sẽ viết code hỗ trợ dọn dẹp cả RAM thường và VRAM.
> - Các mô hình khác trong tương lai cũng sẽ được quản lý bằng cơ chế này, giúp bạn có thể tích hợp vô hạn ngôn ngữ mà mức RAM tiêu thụ tối đa vẫn chỉ bằng 1 ngôn ngữ.
> Xin bạn hãy xác nhận xem bạn có đồng ý với phương án này không.

## Proposed Changes

### Python Inference Backend

#### [MODIFY] [translator.py](file:///e:/devpro/JavaSpringBoot/translate-app/python-inference/app/inference/translator.py)
Thay đổi cấu trúc file `translator.py`:
- Bỏ các biến toàn cục load model lúc khởi tạo file (`nmt_bana_model`, `nmt_vi_model`, ...).
- Import thêm thư viện `gc` (Garbage Collector) và `torch` để giải phóng bộ nhớ.
- Tạo một lớp (class) `ModelManager` (hoặc hàm quản lý trạng thái) chuyên đảm nhận việc:
    1. Lưu giữ trạng thái xem mô hình nào đang được load (`current_model_name`).
    2. Nếu có yêu cầu dịch bằng mô hình khác với mô hình hiện tại:
        - Xóa mô hình hiện tại khỏi bộ nhớ (`del current_model`).
        - Gọi `gc.collect()` và `torch.cuda.empty_cache()` để ép Python trả lại RAM cho hệ điều hành.
        - Load mô hình mới từ ổ cứng vào RAM.
- Cập nhật hàm `_translate_chunk` để lấy model và tokenizer từ `ModelManager` trước khi dịch thay vì dùng biến toàn cục trực tiếp.

## Verification Plan

### Automated Tests
- Gửi yêu cầu dịch thử hướng `vi-bana` và theo dõi Task Manager xem RAM có tăng lên mức load 1 mô hình không.
- Gửi tiếp yêu cầu dịch hướng `bana-vi`, theo dõi Task Manager. RAM sẽ nhích lên một chút rồi giảm xuống (do tiến trình dọn dẹp RAM) và không bị nhân đôi.

### Manual Verification
- Chạy lại toàn bộ app (Frontend, Backend Java, FastAPI Python).
- Mở Task Manager / Resource Monitor để theo dõi dung lượng RAM của tiến trình Python.
- Thực hiện dịch liên tục 2 chiều và chắc chắn không bị treo máy hay tràn RAM.
