# Kế hoạch Triển khai Hệ thống Dịch thuật Việt - Ba Na

Dựa trên bài báo nghiên cứu "Revitalizing Bahnaric Language through Neural Machine Translation" và phản hồi của bạn, đây là bản thiết kế hệ thống tối ưu cho môi trường không có GPU.

## Các Quyết định Kỹ thuật (Đã Chốt)
- **Tiền xử lý (Segmentation):** Sử dụng thư viện `underthesea` (Python) thay cho `VnCoreNLP` để đảm bảo hệ thống gọn nhẹ, tiêu thụ ít RAM và CPU.
- **Môi trường Huấn luyện (Training):** Sử dụng **Google Colab** (chạy GPU miễn phí đám mây). Mã nguồn tại máy của bạn sẽ chịu trách nhiệm trích xuất dữ liệu, sau đó bạn đem dữ liệu lên Colab để train và chỉ tải file trọng số (`.pt` / `.bin`) về máy.

---

## Các Bước Thực Thi

### Bước 1: Xây dựng Module Tách Cụm Từ (Segmentation Phase)
Thực hiện ngay trên máy tính của bạn (vì CPU xử lý tốt):
- Cài đặt thư viện `underthesea`.
- Viết `segmenter.py` thực hiện đọc từ điển `dataset/dictionary/dict.vi` và `dict.ba`.
- Xây dựng hàm phân loại **Anchor** (từ có trong từ điển, tên riêng, số) và **Chunk** (các cụm từ còn lại cần AI dịch).

### Bước 2: Trích xuất dữ liệu và Huấn luyện trên Colab (BN-BARTpho)
- Viết script `prepare_colab_data.py` để tự động chạy qua toàn bộ tập `train.vi` và `train.ba`, tách ra thành các cụm từ (Chunks).
- Xuất dữ liệu Chunk ra file định dạng JSON/CSV.
- Cung cấp cho bạn một đoạn mã Script / Notebook hoàn chỉnh để bạn copy lên Google Colab, tiến hành Fine-tune mô hình `vinai/bartpho-word`.

### Bước 3: Tích hợp Hệ thống (Inference)
- Viết sẵn luồng chạy chuẩn trong `python-inference/app/inference/translator.py`.
- Khi có request dịch:
  1. Dùng `segmenter.py` tách câu thành Anchors và Chunks.
  2. Dịch Anchors bằng từ điển.
  3. Load file weights mô hình từ Colab về để dịch Chunks (Inference chạy trên CPU hoàn toàn khả thi vì dịch từng cụm ngắn).
  4. Nối lại thành câu hoàn chỉnh và trả về.

### Bước 4: Kiểm tra và Vận hành
- Xác nhận các đầu cuối API giao tiếp mượt mà với Backend Java và Frontend.
