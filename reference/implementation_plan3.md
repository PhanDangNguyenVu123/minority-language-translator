# Kế hoạch Triển khai Dịch ngược (Ba Na ➡️ Việt)

Đúng như bạn nhận xét, dự án hiện tại mới chỉ được thiết kế cấu trúc cho chiều **Việt ➡️ Ba Na**. Để thực hiện chiều ngược lại một cách hoàn chỉnh theo mô hình lai (Dictionary + AI), chúng ta cần thiết kế lại một số module vì bản chất ngôn ngữ Ba Na và tiếng Việt là khác nhau (Ví dụ: Thư viện `underthesea` chỉ hỗ trợ tách từ tiếng Việt, không hỗ trợ tiếng Ba Na).

Dưới đây là kế hoạch chi tiết để xây dựng chiều dịch ngược **Ba Na ➡️ Việt**.

## User Review Required
> [!IMPORTANT]
> - Việc dịch ngược sẽ yêu cầu bạn phải **lên Google Colab huấn luyện thêm một mô hình AI thứ hai** (chiều từ Ba Na sang Việt). Bạn sẽ tốn thêm khoảng 1-2 tiếng để train và sẽ thu được file `best-vi-model`.
> - Việc tải 2 mô hình AI cùng lúc (mỗi mô hình nặng khoảng 2GB) vào RAM có thể làm nặng máy tính của bạn khi chạy Inference cục bộ. 

## Các Quyết định Kỹ thuật (Đã Chốt)
- **Quản lý RAM:** Do máy tính có 16GB RAM (dư dả), hệ thống sẽ **Nạp song song cả 2 mô hình (Dual Model Load)** ngay khi khởi động. Điều này giúp tốc độ dịch tức thì và mượt mà cho cả 2 chiều mà không bị khựng lại.

---

## Các Bước Triển Khai Đề Xuất

### Bước 1: Nâng cấp `segmenter.py` để hỗ trợ tiếng Ba Na
- **Vấn đề:** Không có thư viện `underthesea` cho tiếng Ba Na.
- **Giải pháp:** Xây dựng một hàm tách từ cơ bản bằng dấu cách (Whitespace Tokenizer) kết hợp với thuật toán **Greedy Matching (Khớp chuỗi tham lam)** để quét các cụm từ (Anchors) có 2-3 chữ trong từ điển tiếng Ba Na (`dict.ba`).
- Nạp thêm từ điển chiều đảo ngược `dict_ba_to_vi` vào bộ nhớ.

### Bước 2: Soạn kịch bản Huấn luyện Mô hình chiều Ba Na ➡️ Việt
- Thay vì đảo ngược toàn bộ dữ liệu JSON, chúng ta chỉ cần cập nhật một chút kịch bản (Notebook) trên Google Colab.
- Sửa lại code trong file Hướng dẫn ở Bước 4 (Tokenization) để AI hiểu `inputs` là tiếng Ba Na và `targets` là tiếng Việt:
  ```python
  inputs = [ex for ex in examples["ba"]]
  targets = [ex for ex in examples["vi"]]
  ```
- Kết quả thu được sẽ là một bộ trọng số mới, lưu vào thư mục `python-inference/models/best-vi-model`.

### Bước 3: Cập nhật `translator.py` cho Dịch vụ 2 chiều
#### [MODIFY] `python-inference/app/inference/translator.py`
- Bổ sung cấu trúc kiểm tra:
  - NẾU `source_lang == "vi"` VÀ `target_lang == "bna"` ➡️ Gọi luồng cũ (Dùng `segmenter` tiếng Việt + mô hình `best-bana-model`).
- NẾU `source_lang == "bna"` VÀ `target_lang == "vi"` ➡️ Gọi luồng mới (Dùng `segmenter` tiếng Ba Na + mô hình `best-vi-model`).
- Cấu hình Load song song 2 models ngay khi khởi động để tối ưu tốc độ dịch.

---

## Verification Plan
1. **Kiểm tra Module Tách từ (Segmenter Ba Na):** Đưa một câu tiếng Ba Na có cụm từ dài vào Segmenter để xem thuật toán Greedy Matching có bắt đúng các cụm từ trong từ điển hay không.
2. **Kiểm tra Inference giả lập:** Khi chưa có mô hình `best-vi-model`, tiến hành nhập tiếng Ba Na trên giao diện web để xác nhận các từ khóa (Anchor) được dịch ngược ra tiếng Việt chuẩn xác bằng Từ điển, các chữ còn lại biến thành `[Chunk: ...]`.
3. **Thử nghiệm E2E sau khi Train:** Chờ bạn lên Colab tải file weights về, chạy thử nghiệm một câu hoàn chỉnh và đo kết quả thực tế trên giao diện.
