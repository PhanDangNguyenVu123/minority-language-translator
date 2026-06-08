# TỔNG KẾT LÝ THUYẾT VÀ QUY TRÌNH HUẤN LUYỆN AI (DỊCH MÁY NMT)

Tài liệu này được biên soạn để tổng hợp toàn bộ kiến thức, công nghệ và quy trình từng bước đã thực hiện để huấn luyện AI cho dự án Dịch thuật (Việt ↔ Ba Na, Việt ↔ Ê-đê). Bạn có thể dùng tài liệu này làm đề cương trả lời các câu hỏi phản biện từ thầy cô.

---

## PHẦN 1: TRẢ LỜI NHANH CÁC CÂU HỎI TRỌNG TÂM TỪ GIÁO VIÊN

### 1. "Em train model như thế nào?"
**Trả lời:** 
Dự án của em sử dụng phương pháp **Fine-tuning (Huấn luyện tinh chỉnh)** (hoặc Transfer Learning) trên một mô hình học sâu có sẵn (Pre-trained model), thay vì phải huấn luyện từ con số không (from scratch). Cụ thể, em sử dụng mô hình **BARTpho** – một mô hình Seq2Seq (Sequence-to-Sequence) chuyên biệt cho tiếng Việt. Quá trình train của em tập trung vào việc cho mô hình học các cặp câu song ngữ (Việt-Ba Na, Việt-Ê đê) để điều chỉnh các trọng số mạng nơ-ron nhằm thực hiện tác vụ dịch thuật (Neural Machine Translation - NMT). Để tối ưu tài nguyên, em còn áp dụng kĩ thuật "đóng băng" (Freeze) một số lớp (layer) của Encoder và chỉ tập trung huấn luyện để mô hình dịch nhanh và chính xác hơn.

### 2. "Dữ liệu train từ đâu?"
**Trả lời:** 
Dữ liệu của em là tập dữ liệu song ngữ (Parallel Corpus) đã được thu thập và làm sạch, bao gồm các cặp câu tiếng Việt và tiếng Ba Na / Ê-đê. Dữ liệu này được chia ra làm 3 tệp chính định dạng JSON:
- `train_data.json`: Dữ liệu dùng để huấn luyện chính.
- `valid_data.json`: Dữ liệu dùng để đánh giá mô hình sau mỗi chu kỳ (epoch) để tránh học vẹt (overfitting).
- `test_data.json`: Dữ liệu dùng để kiểm thử.

### 3. "Em train bằng cách nào? (Công cụ & Nơi train)"
**Trả lời:**
- **Môi trường:** Do quá trình huấn luyện AI yêu cầu sức mạnh xử lý đồ họa (GPU) lớn mà máy tính cá nhân không đáp ứng đủ, em đã sử dụng nền tảng đám mây **Google Colab**. Nền tảng này cung cấp GPU (như NVIDIA T4) hoàn toàn miễn phí.
- **Công nghệ & Framework:** Quá trình huấn luyện được viết bằng **Python** trên Jupyter Notebook của Colab. Em sử dụng bộ thư viện **HuggingFace (`transformers`, `datasets`)** – đây là tiêu chuẩn của ngành AI hiện tại cho các bài toán xử lý ngôn ngữ tự nhiên (NLP). 
- Sau khi quá trình train trên Colab hoàn tất, em tải mô hình tốt nhất (best-model) về máy tính cá nhân, và dùng framework **FastAPI** (Python) để chạy mô hình thực tế, giao tiếp với hệ thống web Java Spring Boot qua API.

---

## PHẦN 2: CHI TIẾT CÔNG NGHỆ SỬ DỤNG

1. **Mô hình AI Core (Pre-trained Model):** `vinai/bartpho-word` (BARTpho)
   - *Lý do chọn:* BARTpho là mô hình ngôn ngữ lớn dựa trên kiến trúc BART, được tối ưu riêng cho tiếng Việt. Việc dùng nó làm nền tảng giúp AI đã có sẵn "kiến thức ngữ pháp tiếng Việt", chỉ cần học thêm từ vựng/cấu trúc tiếng Ba Na/Ê-đê là có thể dịch được.
2. **Framework Machine Learning:**
   - **PyTorch:** Framework tính toán tensor bên dưới.
   - **HuggingFace Transformers:** Cung cấp API cấp cao (`Seq2SeqTrainer`, `AutoModelForSeq2SeqLM`) giúp quá trình train dễ dàng, không cần tự code tay từng vòng lặp huấn luyện.
   - **HuggingFace Datasets:** Xử lý và nạp tập dữ liệu (JSON) cực kì tối ưu về mặt RAM.
3. **Công cụ xử lý ngôn ngữ:**
   - `underthesea` & `sentencepiece`: Thư viện dùng để Tách từ (Word Segmentation) giúp mô hình hiểu các âm tiết và từ ngữ tiếng Việt/tiếng dân tộc tốt hơn trước khi số hóa.
4. **Hệ thống Triển khai (Inference System):**
   - **Python FastAPI:** Đóng vai trò làm Server AI. Khi người dùng bấm nút dịch, Server Java Spring Boot sẽ gọi API sang FastAPI, FastAPI nạp model, chạy tính toán (inference) và trả kết quả về.

---

## PHẦN 3: CÁC BƯỚC THỰC HIỆN TRAIN AI (STEP-BY-STEP)

Nếu thầy cô hỏi "Em đã thực hiện những bước nào để ra được con AI này?", bạn sẽ trình bày quy trình 6 bước sau:

### Bước 1: Chuẩn bị & Tiền xử lý dữ liệu (Data Preparation)
- Thu thập các câu song ngữ, cấu trúc chúng thành các đối tượng JSON với format: `{"vi": "Xin chào", "ba": "..."}`.
- Đóng gói dữ liệu thành file `.zip` và tải lên Google Colab.

### Bước 2: Khởi tạo môi trường (Environment Setup)
- Bật Google Colab, chuyển cấu hình Runtime sang **GPU**.
- Cài đặt các thư viện cần thiết bằng lệnh: `!pip install transformers datasets sentencepiece accelerate underthesea`

### Bước 3: Tải Model và Tokenizer
- Load mô hình gốc: `AutoModelForSeq2SeqLM.from_pretrained("vinai/bartpho-word")`
- Load Tokenizer: Công cụ để biến chữ (text) thành số (vector) để AI hiểu được.
- *Kỹ thuật tối ưu:* Cấu hình Freeze Encoder (đóng băng bộ mã hoá) `param.requires_grad = False` để tăng tốc độ train và giữ nguyên các biểu diễn tốt của tiếng Việt.

### Bước 4: Mã hóa dữ liệu (Tokenization & Mapping)
- Cắt gọt độ dài câu (Truncation): Giới hạn chiều dài câu ở mức `max_length = 256` token để tránh tràn bộ nhớ GPU.
- Map toàn bộ tập dữ liệu (cả nguồn và đích) qua Tokenizer để chuyển thành dạng `input_ids` (mảng các con số). 

### Bước 5: Cấu hình và Bắt đầu Huấn luyện (Training)
- Khởi tạo bộ thu thập dữ liệu (`DataCollatorForSeq2Seq`) để tạo các batch (nhóm dữ liệu).
- Thiết lập `Seq2SeqTrainingArguments`:
  - `learning_rate = 2e-5`: Tốc độ học (không quá nhanh để tránh bỏ lỡ điểm tối ưu, không quá chậm).
  - `num_train_epochs = 10` (hoặc 21): Số vòng lặp toàn bộ tập dữ liệu.
  - `fp16 = True`: Áp dụng Mixed Precision (tính toán dấu phẩy động 16-bit) giúp GPU train nhanh gấp đôi mà không suy giảm độ chính xác.
- Gọi hàm `trainer.train()` để tiến trình học bắt đầu. AI sẽ dần dần điều chỉnh sai số (loss function) từ cao xuống thấp thông qua dữ liệu Valid.

### Bước 6: Đánh giá, Lưu mô hình và Tích hợp
- Sau khi chạy đủ số epoch, gọi `trainer.save_model()` để xuất ra thư mục `best-model`.
- Nén mô hình đó lại, tải về máy tính cá nhân.
- Copy vào dự án ở đường dẫn `python-inference/models/`. Cập nhật mã nguồn `translator.py` trong FastAPI để load folder này thay vì đoạn code giả (placeholder). Quá trình đào tạo kết thúc và sẵn sàng cho End-User sử dụng.
