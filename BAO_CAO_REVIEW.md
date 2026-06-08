# BÁO CÁO REVIEW DỰ ÁN

**Tên dự án:** Hệ thống dịch thuật máy cho ngôn ngữ dân tộc thiểu số (Ba Na, Ê-đê)

**Loại dự án:** Đồ án môn học (DACN1)

**Repository:** `translate-app`

**Ngày review:** 31/05/2026

**Phạm vi review:** Toàn bộ mã nguồn, kiến trúc, pipeline ML, tài liệu và khả năng demo end-to-end.

---

## 1. Tóm tắt điều hành

Dự án xây dựng một ứng dụng web dịch thuật giữa **Tiếng Việt** và hai ngôn ngữ dân tộc thiểu số **Ba Na** và **Ê-đê**, sử dụng mô hình **BARTpho** fine-tuned kết hợp từ điển song ngữ và segmentation cho Ba Na.

Hệ thống gồm 3 tầng: React (UI) → Spring Boot (API, auth, MySQL) → FastAPI (inference ML). Ngoài dịch văn bản, dự án còn có OCR ảnh, dịch tài liệu, TTS, quản lý người dùng, lịch sử, crowdsourcing và admin dashboard.

**Đánh giá tổng thể:** Dự án **đạt yêu cầu và vượt mức kỳ vọng** cho một đồ án môn học. Phần ML có chiều sâu thực tế, hệ thống chạy được end-to-end. Cần cải thiện nhẹ về tách module frontend, test tự động và một số chi tiết UI/backend chưa khớp nhau.

---

## 2. Thông tin kỹ thuật

| Hạng mục | Chi tiết |
|----------|----------|
| Frontend | React 19, TypeScript, Vite 8 |
| Backend | Spring Boot 3.3.3, Java 17, JWT, MySQL 8 |
| Inference | FastAPI, Transformers, PyTorch, underthesea |
| Base model | `vinai/bartpho-word` |
| Hướng dịch NMT | `vi↔bna`, `vi↔ede` (4 model) |
| Training | Google Colab, notebook trong `train-model/` |
| Database | MySQL — users, history, favorites, edits, votes |

### Cấu trúc mã nguồn

| Module | Quy mô ước lượng |
|--------|------------------|
| Backend Java | ~47 file nguồn, phân lớp controller/service/repository/entity |
| Frontend | `App.tsx` ~1.580 dòng (component chính), 4 component phụ |
| Python inference | translator, segmenter, ocr, document, tts, examples |
| Dữ liệu | Corpus Ba Na, JSON Colab, từ điển song ngữ |

---

## 3. Kiến trúc hệ thống

### 3.1. Sơ đồ tổng quan

```text
Browser (React SPA)
    │  HTTP /api/*
    ▼
Spring Boot :8080
    ├── JWT Authentication
    ├── MySQL (JPA/Hibernate)
    ├── Static UI (frontend/dist)
    └── RestTemplate proxy
            │
            ▼
FastAPI :8001
    ├── /internal/translate   → NMT + segmentation
    ├── /internal/examples    → Tra corpus
    ├── /internal/ocr         → Tesseract + dịch
    ├── /internal/document    → txt/docx/pdf
    └── /internal/tts         → gTTS + phonetic
            │
            ▼
    python-inference/models/  (BARTpho fine-tuned)
```

### 3.2. Đánh giá kiến trúc

| Tiêu chí | Đánh giá | Nhận xét |
|----------|----------|----------|
| Tách lớp (separation of concerns) | Tốt | UI, business logic, ML inference tách rõ |
| Contract API | Tốt | JSON schema ổn định giữa Java ↔ Python |
| Khả năng mở rộng | Khá | Thêm ngôn ngữ mới chỉ cần thêm model + logic trong `translator.py` |
| Triển khai | Trung bình | Chạy local, 2 process song song, chưa container hoá |
| Bảo mật | Chấp nhận được (dev) | JWT stateless, role ADMIN; secret hardcode cho môi trường dev |

**Kết luận kiến trúc:** Phù hợp đồ án. Lựa chọn Spring Boot làm API gateway và Python cho inference là hợp lý, dễ giải thích trước hội đồng.

---

## 4. Review từng module

### 4.1. Frontend (React)

**Chức năng đã triển khai:**
- Giao diện kiểu Google Dịch (2 panel nhập/xuất)
- Chọn ngôn ngữ, hoán đổi hướng dịch
- Auto-translate (debounce ~600ms)
- Chế độ dịch: text / ảnh / tài liệu
- Bàn phím ảo cho chữ dân tộc
- Nhập giọng nói (Web Speech API)
- TTS, copy, share URL
- Auth modal, lịch sử, yêu thích
- Feedback modal (edit/vote)
- Admin dashboard (recharts)

**Điểm mạnh:**
- UI đầy đủ tính năng, trải nghiệm gần sản phẩm thật
- Guest mode + sync khi đăng nhập — thoughtful UX
- Build tích hợp Spring Boot qua `postbuild` script

**Điểm cần cải thiện:**
- `App.tsx` quá lớn (~1.580 dòng), gom UI + fetch + business logic
- UI hiển thị Khmer (`km`) nhưng backend chưa có model → dễ gây hiểu nhầm
- TTS luôn gửi `lang: 'vi'` dù đang đọc output Ba Na/Ê-đê
- Không có React Router — chấp nhận được cho SPA đơn trang

**Điểm:** 7.5 / 10

---

### 4.2. Backend (Spring Boot)

**Chức năng đã triển khai:**

| Nhóm API | Endpoints |
|----------|-----------|
| Dịch thuật (public) | `/api/translate`, `/examples`, `/tts`, `/ocr`, `/document` |
| Auth | `/api/auth/login`, `/register` |
| User (JWT) | `/api/history`, `/favorites`, `/sync-guest-data` |
| Feedback | `/api/feedback/edit`, `/vote` |
| Admin | `/api/admin/stats`, `/recent-feedback` |

**Điểm mạnh:**
- Phân lớp rõ: Controller → Service → Repository → Entity
- JWT stateless, `@PreAuthorize` cho admin
- Soft-delete lịch sử (`is_deleted`)
- Proxy Python có xử lý lỗi qua `UpstreamServiceException`
- JPA `ddl-auto=update` — setup nhanh cho dev

**Điểm cần cải thiện:**
- JWT secret hardcode trong `application.properties`
- MySQL root/password rỗng — chỉ phù hợp dev local
- `database.sql` lệch entity JPA (thiếu `role`, `is_deleted`)
- Không có unit test / integration test

**Điểm:** 8 / 10

---

### 4.3. Python Inference (FastAPI + ML)

**Pipeline dịch Ba Na (`vi → bna`):**

```text
Input (tiếng Việt)
  → underthesea word_tokenize
  → Segmenter: ghép từ theo từ điển
  → ANCHOR: tra dict trực tiếp
  → CHUNK: gọi BARTpho fine-tuned
  → Ghép kết quả → Output (Ba Na)
```

**Pipeline dịch Ê-đê:** NMT trực tiếp trên toàn câu (không có từ điển).

**ModelManager:**
- Lazy-load model theo hướng dịch
- Unload model cũ khi đổi hướng → tiết kiệm RAM
- 4 thư mục model: `best-vntobana-model`, `best-banatovn-model`, `best-vntoede-model`, `best-edetovn-model`

**Điểm mạnh:**
- Pipeline Ba Na có chiều sâu — không chỉ gọi model thuần
- Kết hợp rule-based (từ điển) + neural (NMT) phù hợp low-resource
- Hỗ trợ thêm OCR, document, TTS, examples
- Fallback graceful khi thiếu model (`[Chunk: ...]`)

**Điểm cần cải thiện:**
- Tesseract hardcode path Windows
- `prepare_colab_data.py` có đường dẫn dataset không khớp runtime
- Chưa có metric đánh giá chất lượng dịch (BLEU) trong repo
- Script test thủ công (`test_translation.py`), không phải pytest

**Điểm:** 8.5 / 10

---

### 4.4. Dữ liệu & Huấn luyện

| Nguồn | Vị trí | Mục đích |
|-------|--------|----------|
| Parallel corpus Ba Na | `dataset/dataset-Bahnaric/parallel_corpus/` | Train/valid/test |
| Từ điển Ba Na | `dataset/dataset-Bahnaric/dictionary/` | Segmentation + lookup |
| JSON Colab Ba Na | `python-inference/colab_dataset_bahnaric/` | Training + examples |
| JSON Colab Ê-đê | `python-inference/colab_dataset_ede/` | Training + examples |
| Notebook train | `train-model/` | Colab GPU |
| Hướng dẫn train | `Colab_Training_Guide.md` | Quy trình fine-tune |

**Điểm mạnh:**
- Có corpus, từ điển, script tiền xử lý, notebook và hướng dẫn train
- Chu trình train → deploy model → inference hoàn chỉnh

**Điểm cần cải thiện:**
- Tên model trong Colab guide (`best-bana-model`) khác inference (`best-vntobana-model`)
- Corpus Ê-đê nhỏ hơn, chưa có từ điển bổ trợ
- Notebook lớn, khó quản lý trên git

**Điểm:** 8 / 10

---

### 4.5. Tài liệu

| Tài liệu | Trạng thái |
|----------|------------|
| `README.md` | Đã cập nhật, mô tả đúng tính năng và hướng dẫn chạy |
| `Colab_Training_Guide.md` | Đầy đủ cho Ba Na, cần đồng bộ tên model |
| `BAO_VE_OUTLINE.md` | Outline slide bảo vệ 19 slide |
| `database.sql` | Tham khảo, lệch một phần so với JPA entity |
| `frontend/README.md` | Template Vite mặc định, chưa cập nhật |

**Điểm:** 7.5 / 10

---

## 5. Bảng chấm điểm tổng hợp

| Tiêu chí | Trọng số | Điểm (10) | Điểm có trọng số |
|----------|----------|-----------|------------------|
| Ý nghĩa & phạm vi đề tài | 10% | 9.5 | 0.95 |
| Kiến trúc hệ thống | 15% | 8.0 | 1.20 |
| Triển khai ML / NMT | 25% | 8.5 | 2.13 |
| Chất lượng code backend | 15% | 8.0 | 1.20 |
| Chất lượng code frontend | 10% | 7.5 | 0.75 |
| Tính năng & trải nghiệm người dùng | 10% | 9.0 | 0.90 |
| Dữ liệu & quy trình train | 10% | 8.0 | 0.80 |
| Tài liệu & khả năng demo | 5% | 7.5 | 0.38 |
| **Tổng** | **100%** | | **8.31 / 10** |

**Xếp loại gợi ý:** **Khá — Giỏi** (phù hợp bảo vệ đồ án)

---

## 6. Điểm mạnh nổi bật

1. **Đề tài có giá trị thực tiễn** — dịch ngôn ngữ dân tộc thiểu số, ít công cụ hỗ trợ trên thị trường.
2. **Pipeline ML có chiều sâu** — kết hợp từ điển + segmentation + NMT cho Ba Na, không dừng ở mức gọi API có sẵn.
3. **Hệ thống end-to-end hoàn chỉnh** — từ train model trên Colab đến deploy inference và giao diện web.
4. **Phạm vi tính năng rộng** — OCR, dịch tài liệu, TTS, auth, crowdsourcing, admin dashboard.
5. **Kiến trúc 3 tầng rõ ràng** — dễ vẽ sơ đồ, dễ giải thích trước hội đồng.
6. **Quản lý bộ nhớ model** — `ModelManager` lazy-load/unload phù hợp môi trường máy cá nhân.

---

## 7. Hạn chế & rủi ro

| # | Hạn chế | Mức độ | Ghi chú |
|---|---------|--------|---------|
| 1 | Corpus nhỏ, chất lượng dịch còn hạn chế | Cao | Đặc thù low-resource, cần nêu trong báo cáo |
| 2 | Không có test tự động | Trung bình | Chấp nhận được cho đồ án, nên nêu hướng phát triển |
| 3 | Frontend monolithic (`App.tsx`) | Trung bình | Không ảnh hưởng demo |
| 4 | UI Khmer chưa có model | Thấp | Nên ẩn hoặc gắn nhãn "Coming soon" |
| 5 | TTS chưa tối ưu cho mọi ngôn ngữ | Thấp | Dùng gTTS + phonetic mapping |
| 6 | Chạy local, chưa Docker/CI | Thấp | Phù hợp phạm vi đồ án |
| 7 | Config bảo mật dev-only | Thấp | JWT secret hardcode, MySQL root rỗng |
| 8 | Schema SQL lệch JPA entity | Thấp | JPA tự tạo bảng khi chạy |

---

## 8. Khuyến nghị

### Ưu tiên cao (trước bảo vệ)

- [ ] Chuẩn bị 5–10 cặp câu demo chất lượng tốt cho cả Ba Na và Ê-đê
- [ ] Kiểm tra 4 model đã tải đủ trong `python-inference/models/`
- [ ] Tạo sẵn tài khoản demo (user + admin)
- [ ] Thử demo OCR và dịch document ít nhất 1 lần trước ngày bảo vệ
- [ ] Slide có sơ đồ kiến trúc và pipeline ML (tham khảo `BAO_VE_OUTLINE.md`)

### Ưu tiên trung bình (nếu còn thời gian)

- [ ] Ẩn hoặc disable Khmer trên UI
- [ ] Sửa TTS truyền đúng `targetLang`
- [ ] Đồng bộ tên model giữa Colab guide và inference
- [ ] Cập nhật `database.sql` khớp JPA entity

### Hướng phát triển dài hạn (nêu trong báo cáo)

- Mở rộng corpus, thêm ngôn ngữ dân tộc khác
- Đánh giá BLEU / human evaluation
- Tách `App.tsx` thành hooks và components nhỏ
- Container hoá (Docker Compose)
- TTS giọng bản địa

---

## 9. Kết luận

Dự án **translate-app** là một đồ án có chất lượng tốt, vượt mức demo đơn giản thường thấy ở các đồ án dịch thuật. Sinh viên đã hoàn thành đầy đủ chu trình: thu thập dữ liệu → huấn luyện model → xây dựng pipeline inference → triển khai hệ thống web với nhiều tính năng phụ trợ.

Phần kỹ thuật ML (segmentation + dictionary + NMT cho Ba Na) là điểm sáng nhất, thể hiện hiểu biết về xử lý ngôn ngữ low-resource. Phần hệ thống (auth, history, feedback, admin) cho thấy tư duy sản phẩm, không chỉ tập trung vào model.

Các hạn chế chủ yếu nằm ở quy mô dữ liệu, thiếu test tự động và vài chi tiết UI chưa khớp backend — đều có thể giải thích hợp lý trong phần "Hạn chế & hướng phát triển" của báo cáo bảo vệ.

**Khuyến nghị:** **Chấp thuận bảo vệ** với điều kiện chuẩn bị demo ổn định và slide trình bày pipeline ML rõ ràng.

---

## Phụ lục — Danh sách file quan trọng

| File | Vai trò |
|------|---------|
| `frontend/src/App.tsx` | Giao diện chính |
| `backend-java/.../TranslateService.java` | Proxy sang Python |
| `backend-java/.../WebSecurityConfig.java` | Cấu hình JWT |
| `python-inference/app/inference/translator.py` | Logic NMT + segmentation |
| `python-inference/app/inference/segmenter.py` | Từ điển + tách từ |
| `python-inference/app/main.py` | FastAPI endpoints |
| `Colab_Training_Guide.md` | Hướng dẫn train |
| `BAO_VE_OUTLINE.md` | Outline slide bảo vệ |
| `README.md` | Hướng dẫn cài đặt & chạy |
