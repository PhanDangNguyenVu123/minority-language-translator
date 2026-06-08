# Đánh giá `train-model` theo quy trình Dự án Học máy (lớp học)

**Đối chiếu với 6 bước:** Thu thập dữ liệu → EDA & trực quan → Chuẩn bị dữ liệu → Chọn & huấn luyện mô hình → Tinh chỉnh siêu tham số → Triển khai & bảo trì

**Phạm vi đánh giá:**

| Thành phần | Vị trí |
|------------|--------|
| Notebook Ba Na → Việt | `train-model/translate_app_banatovn.ipynb` |
| Notebook Ê-đê → Việt | `train-model/translate_app_edetovn.ipynb` |
| Tiền xử lý Ba Na | `python-inference/prepare_colab_data.py` |
| Tiền xử lý Ê-đê | `python-inference/prepare_ede_dataset.py` |
| Hướng dẫn train | `Colab_Training_Guide.md` |

**Lưu ý:** Trong repo chỉ có notebook cho **2 hướng ngược** (bna→vi, ede→vi). Hai hướng **vi→bna** và **vi→ede** được mô tả trong `Colab_Training_Guide.md` nhưng **không có file notebook tương ứng** trong `train-model/`.

---

## Bảng tổng hợp nhanh

| Bước (lớp học) | Mức độ đạt | Điểm (10) |
|----------------|------------|-----------|
| 1.2.1 Thu thập dữ liệu | Khá | 7 |
| 1.2.2 EDA & trực quan hóa | **Thiếu** | 2 |
| 1.2.3 Chuẩn bị dữ liệu | Khá | 6.5 |
| 1.2.4 Chọn & huấn luyện mô hình | Khá–Tốt | 7 |
| 1.2.5 Tinh chỉnh siêu tham số | Yếu–Khá | 5 |
| 1.2.6 Triển khai & bảo trì | Khá (ở tầng app) | 7 |

**Điểm trung bình pipeline train:** **~5.8/10** theo rubric đầy đủ 6 bước lớp học.

**Điểm thực tế cho đồ án:** **~7.5/10** — đủ train được model và gắn vào hệ thống; thiếu chủ yếu ở EDA, baseline, tuning có hệ thống.

---

## 1.2.1. Thu thập dữ liệu

### Yêu cầu lớp học

- Nguồn: Kaggle, HuggingFace, UCI, API, Web Scraping, tự thu thập
- Pháp lý: bản quyền, GDPR, đạo đức dữ liệu
- Đánh giá sơ bộ: chất lượng, độ tin cậy, số thuộc tính, số mẫu

### Thực tế trong dự án

| Nguồn | Ngôn ngữ | Quy mô | Ghi chú |
|-------|----------|--------|---------|
| `dataset/dataset-Bahnaric/` | Ba Na | 13.250 cặp câu + 13.030 từ điển | Train 9.275 / Valid 1.988 / Test 1.987 |
| HuggingFace (parquet) | Ê-đê | ~2.000 valid + ~2.000 test trong repo | `train_data.json` Ê-đê **không có** trong git |
| `colab_dataset_bahnaric/` | Ba Na | Chỉ valid + test JSON | Thiếu `train_data.json` trong repo |

**Đã làm:**

- Có corpus song ngữ thật, đủ train/valid/test cho Ba Na
- Script `prepare_ede_dataset.py` đọc parquet HuggingFace → JSON cho Colab
- Notebook unzip `colab_dataset.zip` / `colab_dataset_ede.zip` trước khi train

**Chưa làm / thiếu:**

- Không ghi rõ **nguồn gốc**, **giấy phép**, **điều kiện sử dụng** trong notebook hay báo cáo
- Không có mục **đánh giá chất lượng sơ bộ** (tỷ lệ rỗng, độ dài câu, trùng lặp) trong `train-model`
- `prepare_colab_data.py` trỏ sai path (`../dataset/dictionary/` thay vì `dataset/dataset-Bahnaric/`)
- `prepare_ede_dataset.py` hardcode path máy cá nhân `E:\devpro\...`

### Đánh giá: **7/10**

Đủ dữ liệu để fine-tune, nhưng thiếu tài liệu hóa nguồn và kiểm tra chất lượng ban đầu theo đúng slide lớp.

---

## 1.2.2. Khám phá và trực quan hóa (EDA)

### Yêu cầu lớp học

- EDA: mean, median, độ lệch chuẩn
- Trực quan: histogram, scatter, heatmap, box plot
- Insight: pattern, phát hiện bất thường thủ công

### Thực tế trong dự án

**Trong `train-model/`:** **Không có** ô code hoặc notebook nào cho EDA.

- Không thống kê độ dài câu (vi / ba / ede)
- Không histogram phân phối độ dài token
- Không kiểm tra cặp lệch hàng (misalignment) giữa file `.vi` và `.ba`
- Không visualize tần suất từ OOV / từ hiếm
- Không phân tích từ điển (coverage corpus)

**Hệ quả:** Khó giải thích trước hội đồng *tại sao* chọn `max_length=256`, *tại sao* corpus đủ hay chưa, có outlier không.

### Đánh giá: **2/10**

Đây là **khoảng trống lớn nhất** so với quy trình lớp học. Nên bổ sung ít nhất 1 notebook hoặc vài cell EDA trước khi train.

**Gợi ý tối thiểu (1–2 trang báo cáo):**

```python
# Ví dụ cell EDA nên có
import matplotlib.pyplot as plt
lengths_vi = [len(s.split()) for s in train_data["vi"]]
plt.hist(lengths_vi, bins=50)
plt.title("Phân phối độ dài câu tiếng Việt (train)")
```

---

## 1.2.3. Chuẩn bị dữ liệu

### Yêu cầu lớp học

- Làm sạch: missing values
- Feature: One-hot / Label encoding (categorical → numeric)
- Scaling: Min-Max / Standardization (và giải thích vì sao cần)
- Data augmentation
- Pipeline tự động (Scikit-Learn)

### Thực tế trong dự án

| Kỹ thuật | NMT / dự án của bạn | Đánh giá |
|----------|---------------------|----------|
| Làm sạch missing | `prepare_ede_dataset.py` bỏ dòng `pd.isna`; Ba Na chỉ lấy dòng `vi and ba` non-empty | Cơ bản, chưa đủ |
| Encoding categorical | Tokenizer BARTpho (subword) — **đúng cho NLP**, không dùng One-hot | Phù hợp domain |
| Feature scaling | Không áp dụng (không cần cho deep learning text) | Chấp nhận được — nên **giải thích** trong báo cáo |
| Augmentation | **Không có** (back-translation, synonym, noise…) | Thiếu cho low-resource |
| Pipeline tự động | `datasets.map(preprocess_function, batched=True)` | Có pipeline tokenize |

**Notebook:**

- `max_length = 256`, truncation — hợp lý
- Ba Na notebook: `preprocess_function_bana_to_vi` — input `ba`, target `vi`
- Ê-đê notebook: input `ede`, target `vi` (tên hàm vẫn `preprocess_function_bana_to_vi` — **copy-paste**, nên đổi tên)

**Chưa dùng tập test khi train:**

- Chỉ load `train_data.json` + `valid_data.json`
- `test_data.json` **không** xuất hiện trong notebook → vi phạm tinh thần “giữ test cho đánh giá cuối” (mục 1.2.5)

### Đánh giá: **6.5/10**

Tokenization đúng hướng NMT; thiếu làm sạch có hệ thống, augmentation, và tách biệt rõ test set trong quy trình train.

---

## 1.2.4. Chọn và huấn luyện mô hình

### Yêu cầu lớp học

- Baseline để so sánh
- Short-list thuật toán (Linear, SVM, Trees…)
- Phân tích lỗi (error analysis)

### Thực tế trong dự án

| Hạng mục | Thực tế |
|----------|---------|
| **Mô hình chọn** | `vinai/bartpho-word` + `Seq2SeqTrainer` — lựa chọn **hợp lý** cho Việt ↔ ngôn ngữ ít tài nguyên |
| **Baseline** | **Không có** (không so với: không fine-tune, Google Translate, hoặc chỉ từ điển) |
| **Short-list** | Không thử LSTM, mBART, NLLB — chấp nhận được cho đồ án nếu **giải thích** lý do chọn BARTpho |
| **Frozen encoder** | Có — giảm overfit, train nhanh hơn |
| **Huấn luyện** | `trainer.train()`, eval mỗi epoch, `predict_with_generate=True` |
| **Error analysis** | **Không có** trong notebook (không in câu dịch sai mẫu) |

**Siêu tham số cố định (cả 2 notebook):**

| Tham số | Giá trị |
|---------|---------|
| `learning_rate` | 2e-5 |
| `batch_size` | 16 |
| `num_train_epochs` | **21** (notebook) vs **10** (`Colab_Training_Guide.md`) — **không thống nhất** |
| `weight_decay` | 0.01 |
| `fp16` | True |

### Đánh giá: **7/10**

Có train end-to-end trên GPU Colab, cấu hình Trainer chuẩn. Thiếu baseline và error analysis theo đúng slide lớp.

---

## 1.2.5. Tinh chỉnh mô hình, tối ưu tham số

### Yêu cầu lớp học

- Hyperparameter tuning: Grid Search / Randomized Search
- Ensemble mô hình tốt nhất
- **Final eval trên Test Set**

### Thực tế trong dự án

| Hạng mục | Thực tế |
|----------|---------|
| Grid / Random Search | **Không** — chỉ 1 bộ hyperparameter cố định |
| Ensemble | **Không** |
| Validation khi train | Có — `eval_strategy="epoch"` trên **valid** |
| Metric | Trainer mặc định (loss); **không tính BLEU/chrF** trong notebook |
| Test set | **Không** dùng sau train — `test_data.json` bị bỏ qua |
| Early stopping | **Không** cấu hình `load_best_model_at_end` / `metric_for_best_model` |

**Đặt tên model sau train:**

| Notebook | Lưu thành | Inference cần |
|----------|-----------|---------------|
| `translate_app_banatovn.ipynb` | `best-vi-model` | `best-banatovn-model` |
| `translate_app_edetovn.ipynb` | `best-edetovn-model` | `best-edetovn-model` ✅ |

→ Ba Na notebook **lệch tên** so với `python-inference/models/` (phải đổi tên thủ công).

### Đánh giá: **5/10**

Có fine-tune thực sự nhưng chưa đạt chuẩn “tinh chỉnh có hệ thống” và “đánh giá cuối trên test” như slide.

**Gợi ý nhanh:**

```python
# Sau trainer.train() — đánh giá BLEU trên test
import evaluate
bleu = evaluate.load("bleu")
# generate trên test_data.json, tính BLEU
```

---

## 1.2.6. Triển khai, theo dõi, bảo trì

### Yêu cầu lớp học

- Lưu model: Pickle / Joblib
- Deploy: REST API (Flask/FastAPI), Docker
- Monitoring: data drift, concept drift
- Chiến lược retrain

### Thực tế trong dự án

| Hạng mục | `train-model` | Toàn dự án |
|----------|---------------|------------|
| Lưu model | `trainer.save_model()` + zip tải về | ✅ HuggingFace format (tốt hơn pickle cho Transformer) |
| Deploy API | Không trong notebook | ✅ FastAPI `/internal/translate` + Spring proxy |
| Docker | Không | Không |
| Monitoring drift | Không | Không |
| Retrain | Không mô tả | Crowdsourcing feedback có thể dùng sau này |

Notebook kết thúc ở bước zip model — **đúng phạm vi train**. Phần deploy nằm ở `python-inference/` và được nối tốt qua `ModelManager`.

### Đánh giá: **7/10** (xét cả pipeline đồ án)

Train folder chỉ cover bước export model; triển khai inference **đã có** ở tầng khác.

---

## Đánh giá từng file cụ thể

### `translate_app_banatovn.ipynb`

| Tiêu chí | Nhận xét |
|----------|----------|
| Cấu trúc | 6 bước markdown rõ (cài đặt → unzip → load → preprocess → train → save) |
| Hướng dịch | **Ba Na → Việt** (không phải Việt → Ba Na dù tên file gợi ý ngược) |
| Epoch | 21 (cao hơn guide 10 epoch) |
| Thiếu | EDA, baseline, BLEU, test eval, hyperparameter search |

### `translate_app_edetovn.ipynb`

| Tiêu chí | Nhận xét |
|----------|----------|
| Cấu trúc | 6 code cells, **không có** markdown giải thích (kém hơn notebook Ba Na) |
| Hướng dịch | **Ê-đê → Việt** |
| Tên hàm preprocess | `preprocess_function_bana_to_vi` — nên đổi thành `preprocess_ede_to_vi` |
| Lưu model | `best-edetovn-model` — khớp inference ✅ |

### `prepare_colab_data.py` + `prepare_ede_dataset.py`

| File | Vấn đề |
|------|--------|
| `prepare_colab_data.py` | Path sai; import `Segmenter` không dùng; output `colab_dataset` vs repo `colab_dataset_bahnaric` |
| `prepare_ede_dataset.py` | Path cứng; phụ thuộc pandas + parquet ngoài repo |

### `Colab_Training_Guide.md`

- Mô tả đủ pipeline Việt → Ba Na và Ba Na → Việt
- Epoch mẫu 10, notebook thực tế 21 — **cần đồng bộ**
- Tên model `best-bana-model` vs inference `best-vntobana-model` — **cần đồng bộ**

---

## Checklist đối chiếu slide lớp

| Yêu cầu slide | Ba Na nb | Ê-đê nb | Script/Data |
|---------------|----------|---------|-------------|
| Nguồn dữ liệu rõ ràng | ⚠️ | ⚠️ | ⚠️ |
| Đánh giá sơ bộ chất lượng | ❌ | ❌ | ❌ |
| EDA thống kê | ❌ | ❌ | ❌ |
| Trực quan hóa | ❌ | ❌ | ❌ |
| Làm sạch missing | ⚠️ | ✅ (ede) | ⚠️ (ba) |
| Pipeline tiền xử lý | ✅ tokenize | ✅ tokenize | ⚠️ path |
| Data augmentation | ❌ | ❌ | ❌ |
| Baseline | ❌ | ❌ | — |
| So sánh thuật toán | ❌ | ❌ | — |
| Train + valid | ✅ | ✅ | — |
| Hyperparameter tuning | ❌ | ❌ | — |
| Ensemble | ❌ | ❌ | — |
| **Eval trên test** | ❌ | ❌ | có file test |
| Lưu model | ✅ | ✅ | — |
| Deploy (API) | — | — | ✅ (ngoài folder) |

**Chú thích:** ✅ đạt | ⚠️ một phần | ❌ chưa có

---

## Kết luận

### Điểm mạnh (phù hợp đồ án)

1. Dùng **pre-trained BARTpho** + **fine-tune Seq2Seq** — đúng hướng NMT low-resource.
2. Notebook **chạy được trên Colab** (pip, unzip, train, save, zip).
3. **Frozen encoder** — có lý do kỹ thuật.
4. Model export **tích hợp được** vào `python-inference` (có `ModelManager`).
5. Corpus Ba Na **đủ lớn** (~13k cặp) cho fine-tune cấp đồ án.

### Điểm yếu so với quy trình lớp

1. **Không có EDA / visualization** — khó bảo vệ phần “hiểu dữ liệu”.
2. **Không baseline, không BLEU, không test set cuối**.
3. **Không hyperparameter tuning** có hệ thống.
4. **Thiếu 2/4 notebook** (vi→bna, vi→ede) trong `train-model/`.
5. Script tiền xử lý **lỗi path**, tài liệu nguồn dữ liệu **chưa đầy đủ**.

### Việc nên làm trước bảo vệ (ưu tiên)

1. Thêm **5–10 cell EDA** (độ dài câu, thống kê mẫu, vài biểu đồ).
2. Chạy **BLEU trên `test_data.json`** sau train, điền vào báo cáo Chương 7.
3. Thêm **baseline** 1 dòng: “BARTpho không fine-tune” hoặc “chỉ copy input”.
4. Đồng bộ **tên model**, **số epoch**, **path** giữa guide / notebook / inference.
5. Bổ sung notebook hoặc ghi chú rõ **vi→bna** và **vi→ede** đã train ở đâu.

---

*Tài liệu này đối chiếu trực tiếp với nội dung slide “1.2. Dự án Học Máy” (6 mục con) và mã nguồn thực tế trong repository.*
