# Kế hoạch Triển khai Hệ thống Dịch thuật Việt - Ba Na

Dựa trên bài báo nghiên cứu "Revitalizing Bahnaric Language through Neural Machine Translation" và cấu trúc dữ liệu trong thư mục `dataset/`, dưới đây là kế hoạch chi tiết từng bước để biến dự án demo của bạn thành một ứng dụng dịch thuật AI hoàn chỉnh.

Bài báo sử dụng phương pháp **Chunking Translation** (Dịch theo cụm từ) kết hợp giữa từ điển (Dictionary-based) và mô hình học sâu (NMT - Neural Machine Translation) để tối ưu hiệu suất cho ngôn ngữ hiếm (low-resource) như tiếng Ba Na.

## User Review Required
> [!IMPORTANT]
> Phương pháp này yêu cầu huấn luyện (fine-tune) mô hình AI. Quá trình này cần tài nguyên máy tính (đặc biệt là GPU) và thời gian. 
> Bạn có muốn tự huấn luyện (train) mô hình trên máy cục bộ, hay sử dụng các dịch vụ đám mây (như Google Colab) để huấn luyện rồi tải file trọng số (weights) về dự án?

## Open Questions
- Bạn có muốn tích hợp thư viện `VnCoreNLP` (yêu cầu Java chạy ngầm) cho việc tách từ/nhận diện thực thể, hay sử dụng các thư viện Python gọn nhẹ hơn như `PyVi`/`underthesea`?
- Tài nguyên máy tính hiện tại của bạn có GPU để chạy huấn luyện mô hình `BARTpho` không?

---

## Các Bước Triển Khai Đề Xuất

### Bước 1: Xây dựng Module Tiền xử lý & Phân rã (Segmentation Phase)
Mục tiêu là biến một câu tiếng Việt thành một danh sách các **Anchors** (neo) và **Chunks** (cụm từ).
- **Cài đặt thư viện NLP:** Sử dụng `underthesea` hoặc `VnCoreNLP` cho tiếng Việt.
- **Load Dictionary:** Đọc file `dataset/dictionary/dict.vi` và `dict.ba` tạo thành một hash map (từ điển bộ nhớ) để tra cứu nhanh.
- **Logic Phân rã:**
  - Tách câu thành các từ/cụm từ.
  - Từ nào có trong `dict.vi`, tên riêng (NER), số, dấu câu -> Phân loại là **Anchor**.
  - Tên riêng sẽ được chuẩn hóa (bỏ dấu tiếng Việt).
  - Các cụm từ còn lại không có trong từ điển -> Phân loại là **Chunk**.

### Bước 2: Huấn luyện mô hình NMT cho cụm từ (BN-BARTpho Fine-tuning)
Mục tiêu là tạo ra mô hình AI dùng để dịch các **Chunks** sang tiếng Ba Na.
- **Tiền xử lý tập train:** Đọc các file trong `dataset/parallel_corpus/` (`train.vi`, `train.ba`). Áp dụng Segmenter ở Bước 1 để trích xuất ra các cặp `Chunk (Vi) -> Chunk (Ba Na)`.
- **Data Augmentation (Tăng cường dữ liệu):** Theo bài báo, áp dụng các kỹ thuật như Swap (hoán đổi từ), Token Replace (thay từ bằng token ngẫu nhiên) để tăng lượng dữ liệu học.
- **Fine-tuning:**
  - Tải pre-trained model `vinai/bartpho-word` (hoặc syllable) từ HuggingFace.
  - Đóng băng (freeze) Encoder, chỉ huấn luyện Decoder dựa trên tập dữ liệu Chunks đã tạo.
  - Đánh giá trên tập `valid` và lưu lại trọng số mô hình tốt nhất vào thư mục `python-inference/models/`.

### Bước 3: Tích hợp vào Python FastAPI (Mapping Phase)
Đưa mô hình đã huấn luyện vào mã nguồn chạy thực tế.
#### [MODIFY] `python-inference/app/inference/translator.py`
- Thay thế đoạn code `[BaNa-demo]` hiện tại.
- Lắp ráp pipeline hoàn chỉnh:
  1. Nhận câu tiếng Việt đầu vào.
  2. Gọi **Segmenter (Bước 1)** để tách thành Anchors và Chunks.
  3. **Mapping Anchors:** Dịch Anchors trực tiếp bằng từ điển hoặc bỏ dấu tiếng Việt với tên riêng.
  4. **NMT Chunks:** Gọi mô hình **BN-BARTpho (Bước 2)** để dịch từng Chunk.
  5. Nối (Concatenate) các bản dịch lại thành một câu tiếng Ba Na hoàn chỉnh và trả về kết quả.

### Bước 4: Tối ưu hoá Backend Java và Frontend (Tuỳ chọn)
- Đảm bảo Backend proxy đúng API.
- Cập nhật UI/UX ở Frontend để người dùng có thể nhập liệu dễ dàng hơn, hiển thị rõ quá trình dịch (nếu cần).

---

## Verification Plan
1. **Kiểm tra Module Tách từ (Segmenter):** Chạy thử nghiệm một số câu tiếng Việt mẫu, in ra mảng Anchors và Chunks để đảm bảo thuật toán hoạt động đúng như sơ đồ trong bài báo.
2. **Kiểm tra kết quả Dịch nghiệm:** Sử dụng tập `test.vi` và so sánh kết quả sinh ra với `test.ba` bằng độ đo BLEU score.
3. **Thử nghiệm E2E:** Mở giao diện Frontend, nhập một câu tiếng Việt và nhận lại câu tiếng Ba Na chính xác thay vì câu demo.
