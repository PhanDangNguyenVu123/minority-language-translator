# Ứng dụng Dịch thuật Ngôn ngữ Dân tộc Thiểu số

Ứng dụng web dịch thuật giao diện kiểu Google Dịch, hỗ trợ dịch giữa **Tiếng Việt** và các ngôn ngữ dân tộc **Ba Na**, **Ê-đê** bằng mô hình NMT (BARTpho) fine-tuned kết hợp từ điển song ngữ.

**Đồ án:** DACN1 — Hệ thống dịch thuật máy cho ngôn ngữ thiểu số Việt Nam.

## Tính năng

- **Dịch văn bản** — auto-translate khi gõ, hỗ trợ 4 hướng: `vi ↔ bna`, `vi ↔ ede`
- **Ví dụ từ điển** — tra cứu câu mẫu từ corpus song ngữ
- **OCR ảnh** — nhận dạng chữ trên ảnh và dịch theo từng khối văn bản
- **Dịch tài liệu** — hỗ trợ `.txt`, `.docx`, `.pdf`
- **Text-to-Speech** — đọc kết quả dịch (gTTS + chuyển phonetic cho Ba Na/Ê-đê)
- **Bàn phím ảo** — nhập chữ Ba Na, Ê-đê trên máy tính
- **Nhập giọng nói** — Web Speech API (tiếng Việt)
- **Chia sẻ link** — URL kèm ngôn ngữ và nội dung dịch
- **Tài khoản & lịch sử** — đăng ký/đăng nhập JWT, lưu lịch sử và yêu thích trên MySQL
- **Chế độ khách** — lịch sử/yêu thích lưu tạm trong trình duyệt, đồng bộ khi đăng nhập
- **Crowdsourcing** — đề xuất chỉnh sửa bản dịch, bình chọn up/down
- **Admin dashboard** — thống kê và xem feedback gần đây (role `ADMIN`)

## Kiến trúc hệ thống

```text
Browser (React SPA)
    │  /api/*
    ▼
Spring Boot :8080          ← JWT auth, MySQL, proxy API
    │  http://localhost:8001/internal/*
    ▼
FastAPI (python-inference) ← NMT, OCR, TTS, documents
    └── models/            ← BARTpho fine-tuned (tải từ Google Drive)
```

### Pipeline dịch Ba Na

1. **Segmentation** — tách câu bằng `underthesea`, ghép từ theo từ điển song ngữ
2. **Dictionary lookup** — tra trực tiếp các từ khớp trong từ điển
3. **NMT** — dịch các đoạn còn lại bằng BARTpho fine-tuned

Pipeline **Ê-đê** dùng NMT trực tiếp trên toàn câu (chưa có từ điển song ngữ).

## Cấu trúc thư mục

```text
translate-app/
├── frontend/              # React + TypeScript + Vite
├── backend-java/          # Spring Boot API + phục vụ static UI
├── python-inference/      # FastAPI inference service
│   ├── app/inference/     # translator, ocr, document, tts...
│   ├── colab_dataset_*/   # JSON corpus cho examples & training
│   └── models/            # Model weights (gitignored, tải thủ công)
├── dataset/               # Parallel corpus & từ điển Ba Na
├── train-model/           # Notebook Colab huấn luyện
├── database.sql           # Schema MySQL tham khảo
├── Colab_Training_Guide.md
└── README.md
```

## Yêu cầu hệ thống

| Thành phần | Phiên bản |
|------------|-----------|
| Python | 3.10+ |
| Java | 17+ |
| Maven | 3.8+ |
| Node.js | 18+ |
| MySQL | 8.x |
| Tesseract OCR | (tuỳ chọn, cho tính năng OCR ảnh) |

## Cài đặt

### 1. Clone repository

```powershell
git clone https://github.com/quangtuane2/translate-app.git
cd translate-app
```

### 2. Tải model AI

Tải các model từ [Google Drive](https://drive.google.com/drive/folders/1M5o-T0alc5zGt8IGBH5am4ew7Ej5kxlO?usp=sharing) và giải nén vào `python-inference/models/` với đúng tên thư mục:

| Thư mục | Hướng dịch |
|---------|------------|
| `best-vntobana-model/` | Việt → Ba Na |
| `best-banatovn-model/` | Ba Na → Việt |
| `best-vntoede-model/` | Việt → Ê-đê |
| `best-edetovn-model/` | Ê-đê → Việt |

> Nếu thiếu model, hệ thống vẫn chạy nhưng trả về placeholder `[Chunk: ...]` cho phần NMT.

### 3. Cài Python dependencies

```powershell
cd python-inference
python -m venv .venv
.\.venv\Scripts\activate          # Windows
# source .venv/bin/activate         # Linux/macOS
pip install -r requirements.txt
```

> Lần đầu chạy, `transformers` sẽ tải tokenizer `vinai/bartpho-word` từ Hugging Face.

### 4. Cài Node.js dependencies & build UI

```powershell
cd ../frontend
npm install
npm run build
```

Script `postbuild` tự copy `frontend/dist` sang `backend-java/src/main/resources/static/`.

### 5. Cấu hình MySQL

Tạo database (Spring Boot có thể tự tạo nếu user có quyền):

```sql
CREATE DATABASE translate_app;
```

Cấu hình trong `backend-java/src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/translate_app?createDatabaseIfNotExist=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=
```

Schema tham khảo: `database.sql`. JPA dùng `ddl-auto=update` nên bảng tự tạo khi chạy lần đầu.

### 6. Build Java (tuỳ chọn)

```powershell
cd ../backend-java
mvn install -DskipTests
```

## Chạy ứng dụng

Cần chạy **2 dịch vụ** song song:

### Bước 1 — Python inference (port 8001)

```powershell
cd python-inference
.\.venv\Scripts\uvicorn app.main:app --host 127.0.0.1 --port 8001
```

FastAPI docs: http://127.0.0.1:8001/docs

### Bước 2 — Spring Boot (port 8080)

```powershell
cd backend-java
mvn spring-boot:run -DskipTests
```

Mở trình duyệt: **http://127.0.0.1:8080/**

### Dev mode (chỉ frontend)

```powershell
cd frontend
npm run dev
```

Vite dev server (http://localhost:5173) proxy `/api` sang Spring Boot `:8080`.

## API chính

### Dịch thuật (public, không cần auth)

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/api/translate` | Dịch văn bản |
| POST | `/api/examples` | Lấy ví dụ từ corpus |
| POST | `/api/tts` | Text-to-speech |
| POST | `/api/ocr` | OCR + dịch ảnh (multipart) |
| POST | `/api/document` | Dịch tài liệu (multipart) |

### Xác thực

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/api/auth/register` | Đăng ký |
| POST | `/api/auth/login` | Đăng nhập (trả JWT) |

### Cần JWT (header `Authorization: Bearer <token>`)

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET/POST/DELETE | `/api/history` | Lịch sử dịch |
| GET/POST | `/api/favorites` | Yêu thích |
| POST | `/api/sync-guest-data` | Đồng bộ dữ liệu khách |
| POST | `/api/feedback/edit` | Đề xuất sửa bản dịch |
| POST | `/api/feedback/vote` | Bình chọn bản dịch |
| GET | `/api/admin/stats` | Thống kê (ADMIN) |
| GET | `/api/admin/recent-feedback` | Feedback gần đây (ADMIN) |

### Request dịch văn bản

```json
{
  "text": "Xin chào",
  "sourceLang": "vi",
  "targetLang": "bna"
}
```

### Response

```json
{
  "translatedText": "Hơi nơi",
  "sourceLang": "vi",
  "targetLang": "bna"
}
```

### Mã ngôn ngữ hỗ trợ NMT

| Mã | Ngôn ngữ |
|----|----------|
| `vi` | Tiếng Việt |
| `bna` | Tiếng Ba Na |
| `ede` | Tiếng Ê-đê |

## Huấn luyện model

- Hướng dẫn chi tiết: [`Colab_Training_Guide.md`](Colab_Training_Guide.md)
- Notebook mẫu: `train-model/` (chạy trên Google Colab với GPU T4)
- Base model: [`vinai/bartpho-word`](https://huggingface.co/vinai/bartpho-word)
- Dữ liệu Ba Na: `dataset/dataset-Bahnaric/`
- Dữ liệu JSON cho Colab: `python-inference/colab_dataset_bahnaric/`, `colab_dataset_ede/`

Sau khi train, đổi tên thư mục model cho khớp với bảng ở mục **Tải model AI** ở trên.

## Hạn chế & hướng phát triển

- Chưa hỗ trợ NMT cho Khmer (`km`) — hiện chỉ hiển thị trên UI
- TTS dùng gTTS tiếng Việt + phonetic mapping, chưa phải giọng bản địa
- OCR phụ thuộc Tesseract (cần cài riêng trên Windows)
- Chạy local, chưa container hoá (Docker)
- Chưa có bộ test tự động

## Tài liệu liên quan

| File | Nội dung |
|------|----------|
| `train-model/DANH_GIA_THEO_QUY_TRINH_ML.md` | Đánh giá notebook train theo 6 bước ML (lớp học) |
| `BAO_CAO_DO_AN_DAY_DU.md` | Báo cáo đồ án đầy đủ (8 chương) |
| `CODE_REVIEW_CHUYEN_SAU.md` | Code review chuyên sâu |
| `BAO_CAO_REVIEW.md` | Báo cáo review dự án |
| `BAO_VE_OUTLINE.md` | Outline slide bảo vệ đồ án |
| `Colab_Training_Guide.md` | Huấn luyện BARTpho trên Colab |
| `database.sql` | Schema MySQL |
| `python-inference/app/inference/translator.py` | Logic dịch NMT + segmentation |
