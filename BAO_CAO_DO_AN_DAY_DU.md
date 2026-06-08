# BÁO CÁO ĐỒ ÁN

---

**TRƯỜNG ĐẠI HỌC [TÊN TRƯỜNG]**

**KHOA [TÊN KHOA]**

---

# ĐỒ ÁN TỐT NGHIỆP / ĐỒ ÁN MÔN HỌC

## HỆ THỐNG DỊCH THUẬT MÁY CHO NGÔN NGỮ DÂN TỘC THIỂU SỐ VIỆT NAM

### (Tiếng Việt ↔ Ba Na, Tiếng Việt ↔ Ê-đê)

---

**Sinh viên thực hiện:** [Họ và tên]

**Mã sinh viên:** [MSSV]

**Lớp:** [Lớp]

**Giảng viên hướng dẫn:** [Họ tên GVHD]

**Môn học:** DACN1 — Đồ án chuyên ngành

**Thời gian:** Học kỳ 6 — Năm học 2025–2026

---

<div style="page-break-after: always;"></div>

## LỜI CẢM ƠN

Em xin chân thành cảm ơn [Tên GVHD] đã tận tình hướng dẫn, chỉ bảo trong suốt quá trình thực hiện đồ án. Em cũng xin cảm ơn các thầy cô trong Khoa [Tên Khoa] đã truyền đạt kiến thức nền tảng về trí tuệ nhân tạo, xử lý ngôn ngữ tự nhiên và phát triển phần mềm — những kiến thức quan trọng giúp em hoàn thành đề tài này.

Em xin cảm ơn cộng đồng nghiên cứu ngôn ngữ học và các nguồn dữ liệu song ngữ Ba Na, Ê-đê đã công bố, tạo điều kiện cho việc huấn luyện mô hình dịch máy.

Do thời gian và kinh nghiệm còn hạn chế, đồ án không tránh khỏi những thiếu sót. Em rất mong nhận được ý kiến đóng góp từ quý thầy cô.

**Sinh viên thực hiện**

*[Họ và tên]*

---

## MỤC LỤC

1. [Chương 1: Tổng quan](#chương-1-tổng-quan)
2. [Chương 2: Cơ sở lý thuyết](#chương-2-cơ-sở-lý-thuyết)
3. [Chương 3: Phân tích yêu cầu](#chương-3-phân-tích-yêu-cầu)
4. [Chương 4: Thiết kế hệ thống](#chương-4-thiết-kế-hệ-thống)
5. [Chương 5: Dữ liệu và huấn luyện mô hình](#chương-5-dữ-liệu-và-huấn-luyện-mô-hình)
6. [Chương 6: Triển khai hệ thống](#chương-6-triển-khai-hệ-thống)
7. [Chương 7: Kết quả và đánh giá](#chương-7-kết-quả-và-đánh-giá)
8. [Chương 8: Kết luận và hướng phát triển](#chương-8-kết-luận-và-hướng-phát-triển)

**Phụ lục**

- [Phụ lục A: Hướng dẫn cài đặt](#phụ-lục-a-hướng-dẫn-cài-đặt)
- [Phụ lục B: Danh mục API](#phụ-lục-b-danh-mục-api)
- [Phụ lục C: Sơ đồ cơ sở dữ liệu](#phụ-lục-c-sơ-đồ-cơ-sở-dữ-liệu)
- [Tài liệu tham khảo](#tài-liệu-tham-khảo)

---

## DANH MỤC HÌNH VẼ

| Hình | Mô tả |
|------|-------|
| Hình 4.1 | Kiến trúc tổng quan hệ thống 3 tầng |
| Hình 4.2 | Luồng xử lý dịch văn bản |
| Hình 4.3 | Pipeline dịch Ba Na (segmentation + NMT) |
| Hình 4.4 | Sơ đồ ER cơ sở dữ liệu |
| Hình 5.1 | Quy trình huấn luyện trên Google Colab |
| Hình 6.1 | Giao diện ứng dụng (màn hình dịch chính) |

> *Ghi chú: Chèn screenshot thực tế khi hoàn thiện báo cáo Word/PDF.*

---

## DANH MỤC BẢNG

| Bảng | Mô tả |
|------|-------|
| Bảng 3.1 | Yêu cầu chức năng |
| Bảng 3.2 | Yêu cầu phi chức năng |
| Bảng 5.1 | Thống kê dữ liệu huấn luyện |
| Bảng 5.2 | Siêu tham số huấn luyện BARTpho |
| Bảng 6.1 | Công nghệ sử dụng |
| Bảng 7.1 | Ví dụ kết quả dịch Ba Na |
| Bảng 7.2 | Ví dụ kết quả dịch Ê-đê |

---

# CHƯƠNG 1: TỔNG QUAN

## 1.1. Đặt vấn đề

Việt Nam là quốc gia đa dân tộc với 54 dân tộc anh em, mỗi dân tộc có ngôn ngữ và văn hoá riêng. Trong bối cảnh hội nhập và chuyển đổi số, nhu cầu giao tiếp giữa tiếng Việt và các ngôn ngữ dân tộc thiểu số ngày càng tăng — phục vụ giáo dục, y tế, hành chính và bảo tồn di sản văn hoá phi vật thể.

Tuy nhiên, các công cụ dịch thuật phổ biến như Google Translate chưa hỗ trợ hầu hết ngôn ngữ dân tộc Việt Nam, bao gồm **tiếng Ba Na** (Bahnaric, Tây Nguyên) và **tiếng Ê-đê** (Rhade, Tây Nguyên). Nguyên nhân chính:

- **Dữ liệu song ngữ hạn chế** so với các cặp ngôn ngữ phổ biến (Anh–Việt, Trung–Việt).
- **Đặc thù ngôn ngữ:** từ ghép dài, ít ranh giới từ rõ ràng, bảng chữ cái Latin có dấu phụ.
- **Thiếu công cụ NLP** chuyên biệt cho các ngôn ngữ này.

Do đó, việc xây dựng hệ thống dịch thuật máy (Machine Translation — MT) cho các ngôn ngữ dân tộc thiểu số là cần thiết và có ý nghĩa thực tiễn.

## 1.2. Mục tiêu đồ án

### 1.2.1. Mục tiêu chung

Xây dựng ứng dụng web dịch thuật giữa **Tiếng Việt** và **Ba Na**, **Ê-đê** sử dụng mô hình Neural Machine Translation (NMT) fine-tuned, kết hợp từ điển song ngữ cho pipeline Ba Na.

### 1.2.2. Mục tiêu cụ thể

1. Thu thập, tiền xử lý corpus song ngữ Ba Na và Ê-đê.
2. Fine-tune mô hình BARTpho (`vinai/bartpho-word`) trên Google Colab cho 4 hướng dịch.
3. Xây dựng pipeline inference kết hợp segmentation, từ điển và NMT.
4. Phát triển giao diện web kiểu Google Dịch với các tính năng OCR, dịch tài liệu, TTS.
5. Xây dựng backend quản lý người dùng, lịch sử dịch và cơ chế crowdsourcing phản hồi bản dịch.

## 1.3. Phạm vi đồ án

### 1.3.1. Phạm vi chức năng

- Dịch văn bản: `vi ↔ bna`, `vi ↔ ede` (4 hướng).
- OCR ảnh, dịch tài liệu (.txt, .docx, .pdf).
- Text-to-Speech, bàn phím ảo, nhập giọng nói.
- Đăng ký/đăng nhập, lịch sử, yêu thích, feedback, admin dashboard.

### 1.3.2. Phạm vi kỹ thuật

- Huấn luyện trên Google Colab (GPU T4), inference trên CPU local.
- Triển khai local: Spring Boot + FastAPI + MySQL.
- Không bao gồm: deploy cloud, mobile app, dịch real-time streaming.

### 1.3.3. Phạm vi ngoài

- Khmer (`km`) hiển thị trên UI nhưng chưa có model NMT.
- Không đánh giá BLEU chính thức trên benchmark quốc tế.

## 1.4. Phương pháp nghiên cứu

1. **Nghiên cứu lý thuyết:** NMT, BARTpho, xử lý ngôn ngữ low-resource.
2. **Phân tích và thiết kế:** UML, sơ đồ kiến trúc, thiết kế CSDL.
3. **Phát triển và tích hợp:** Agile incremental — frontend, backend, ML pipeline.
4. **Thử nghiệm:** Demo end-to-end, đánh giá định tính trên tập test.

## 1.5. Cấu trúc báo cáo

Báo cáo gồm 8 chương: Tổng quan → Lý thuyết → Yêu cầu → Thiết kế → Dữ liệu & Train → Triển khai → Kết quả → Kết luận, kèm phụ lục hướng dẫn cài đặt và API.

---

# CHƯƠNG 2: CƠ SỞ LÝ THUYẾT

## 2.1. Dịch máy (Machine Translation)

### 2.1.1. Phát triển qua các thế hệ

| Thế hệ | Phương pháp | Đặc điểm |
|--------|-------------|----------|
| 1 | Rule-based | Từ điển + quy tắc ngữ pháp |
| 2 | Statistical MT (SMT) | Mô hình xác suất, phrase table |
| 3 | Neural MT (NMT) | Deep learning, end-to-end |

Đồ án tập trung vào **NMT** — phương pháp state-of-the-art cho hầu hết cặp ngôn ngữ.

### 2.1.2. Kiến trúc Seq2Seq

Mô hình Sequence-to-Sequence gồm:
- **Encoder:** Mã hoá câu nguồn thành vector ngữ cảnh.
- **Decoder:** Sinh câu đích từng token dựa trên ngữ cảnh.

Kiến trúc **Transformer** (Vaswani et al., 2017) thay thế RNN bằng cơ chế **Self-Attention**, xử lý song song tốt hơn và nắm bắt phụ thuộc xa.

### 2.1.3. BART và BARTpho

**BART** (Lewis et al., 2020) là mô hình denoising autoencoder pre-trained, phù hợp fine-tune cho generation tasks bao gồm dịch máy.

**BARTpho** (`vinai/bartpho-word`, Nguyen & Tuan Nguyen, 2020) là BART pre-trained trên corpus tiếng Việt lớn (~20GB), tokenize theo từ (word-level). Đây là lựa chọn phù hợp vì:
- Tiếng Việt là ngôn ngữ nguồn/đích chính trong các cặp dịch.
- Pre-training giúp giảm yêu cầu dữ liệu fine-tune — quan trọng với low-resource languages.

## 2.2. Xử lý ngôn ngữ tự nhiên cho ngôn ngữ low-resource

### 2.2.1. Thách thức

- Corpus song ngữ nhỏ (vài nghìn đến vài chục nghìn câu).
- Out-of-Vocabulary (OOV) cao.
- Từ ghép dài, không có word segmenter chuẩn cho Ba Na/Ê-đê.

### 2.2.2. Chiến lược bổ trợ

1. **Transfer learning:** Fine-tune từ BARTpho pre-trained.
2. **Frozen encoder:** Giảm tham số cần train, tránh overfit.
3. **Dictionary-augmented MT:** Tra từ điển song ngữ trước, NMT cho phần còn lại.
4. **Word segmentation:** Dùng `underthesea` cho tiếng Việt, greedy longest-match cho Ba Na.

## 2.3. Công nghệ web và hệ thống

### 2.3.1. Kiến trúc 3 tầng (Three-tier)

- **Presentation tier:** React SPA — giao diện người dùng.
- **Application tier:** Spring Boot — business logic, auth, proxy.
- **Data/ML tier:** FastAPI + MySQL — inference và lưu trữ.

### 2.3.2. RESTful API

Giao tiếp giữa các tầng qua HTTP JSON REST, stateless, dễ mở rộng và debug.

### 2.3.3. JWT Authentication

JSON Web Token (RFC 7519) — xác thực stateless, phù hợp SPA không dùng session cookie.

---

# CHƯƠNG 3: PHÂN TÍCH YÊU CẦU

## 3.1. Tác nhân (Actors)

| Tác nhân | Mô tả |
|----------|-------|
| Khách (Guest) | Dùng dịch không đăng nhập, lịch sử lưu localStorage |
| Người dùng (User) | Đăng ký/đăng nhập, lịch sử trên server, feedback |
| Quản trị viên (Admin) | Xem thống kê, feedback gần đây |

## 3.2. Yêu cầu chức năng

**Bảng 3.1 — Yêu cầu chức năng**

| ID | Yêu cầu | Mô tả | Trạng thái |
|----|---------|-------|------------|
| F01 | Dịch văn bản | Dịch real-time khi gõ, 4 hướng vi↔bna, vi↔ede | ✅ |
| F02 | Tra ví dụ | Hiển thị câu mẫu từ corpus | ✅ |
| F03 | OCR ảnh | Nhận dạng chữ + dịch overlay | ✅ |
| F04 | Dịch tài liệu | Upload .txt, .docx, .pdf | ✅ |
| F05 | Text-to-Speech | Đọc kết quả dịch | ✅ |
| F06 | Bàn phím ảo | Nhập chữ Ba Na, Ê-đê | ✅ |
| F07 | Nhập giọng nói | Web Speech API (tiếng Việt) | ✅ |
| F08 | Chia sẻ link | URL với tham số ngôn ngữ + text | ✅ |
| F09 | Đăng ký/Đăng nhập | JWT auth | ✅ |
| F10 | Lịch sử dịch | Lưu, xem, xoá (soft-delete) | ✅ |
| F11 | Yêu thích | Đánh dấu câu dịch quan trọng | ✅ |
| F12 | Đồng bộ guest | Merge localStorage khi login | ✅ |
| F13 | Đề xuất sửa | Crowdsourcing bản dịch | ✅ |
| F14 | Bình chọn | Upvote/downvote bản dịch | ✅ |
| F15 | Admin dashboard | Thống kê hệ thống | ✅ |

## 3.3. Yêu cầu phi chức năng

**Bảng 3.2 — Yêu cầu phi chức năng**

| ID | Yêu cầu | Tiêu chí |
|----|---------|----------|
| NF01 | Hiệu năng | Dịch câu ≤ 3 giây trên CPU (câu ngắn) |
| NF02 | Khả dụng | Giao diện responsive, trực quan |
| NF03 | Bảo mật | Mật khẩu BCrypt, JWT 24h |
| NF04 | Mở rộng | Thêm ngôn ngữ mới qua model + config |
| NF05 | Tương thích | Chrome/Edge, Windows 10+ |
| NF06 | Dễ cài đặt | Hướng dẫn README, 3 bước chạy |

## 3.4. Use Case chính

### UC-01: Dịch văn bản

**Tác nhân:** Guest hoặc User

**Luồng chính:**
1. Người dùng chọn ngôn ngữ nguồn và đích.
2. Nhập văn bản vào ô bên trái.
3. Hệ thống tự động gọi API dịch (debounce 600ms).
4. Hiển thị kết quả bên phải.
5. Nếu đã đăng nhập, lưu vào lịch sử.

**Luồng thay thế:**
- 3a. Model chưa tải → hiển thị placeholder `[Chunk: ...]`.
- 3b. Ngôn ngữ không hỗ trợ → hiển thị `[target-demo] text`.

### UC-02: Đăng nhập và đồng bộ dữ liệu khách

**Tác nhân:** Guest → User

**Luồng:**
1. Guest chọn Đăng nhập.
2. Nhập username/password.
3. Hệ thống trả JWT.
4. Gọi `/api/sync-guest-data` merge lịch sử/yêu thích từ localStorage.
5. Chuyển sang dùng API server.

---

# CHƯƠNG 4: THIẾT KẾ HỆ THỐNG

## 4.1. Kiến trúc tổng quan

**Hình 4.1 — Kiến trúc hệ thống 3 tầng**

```text
┌──────────────────────────────────────────────────────────────┐
│                    PRESENTATION TIER                          │
│  React 19 + TypeScript + Vite                                │
│  ┌────────────┐ ┌────────────┐ ┌─────────────────────────┐  │
│  │ Translate  │ │ OCR/Doc    │ │ Auth, History, Admin    │  │
│  │ Panel      │ │ Viewer     │ │ Dashboard               │  │
│  └────────────┘ └────────────┘ └─────────────────────────┘  │
└────────────────────────────┬─────────────────────────────────┘
                             │ REST /api/*
                             ▼
┌──────────────────────────────────────────────────────────────┐
│                    APPLICATION TIER                           │
│  Spring Boot 3.3 — Port 8080                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │ Controllers │→ │ Services     │→ │ JPA Repositories   │  │
│  └─────────────┘  └──────────────┘  └─────────┬──────────┘  │
│  ┌─────────────┐  ┌──────────────┐              │             │
│  │ JWT Security│  │ RestTemplate │              ▼             │
│  └─────────────┘  │ Proxy        │         MySQL 8           │
│                   └──────┬───────┘                            │
└──────────────────────────┼───────────────────────────────────┘
                           │ /internal/*
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                    ML / INFERENCE TIER                          │
│  FastAPI — Port 8001                                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌───────────┐ │
│  │ translator │ │ ocr        │ │ document   │ │ tts       │ │
│  │ + segmenter│ │ (Tesseract)│ │ (docx/pdf) │ │ (gTTS)    │ │
│  └─────┬──────┘ └────────────┘ └────────────┘ └───────────┘ │
│        ▼                                                      │
│  models/ — BARTpho fine-tuned (4 models)                     │
└──────────────────────────────────────────────────────────────┘
```

## 4.2. Luồng xử lý dịch văn bản

**Hình 4.2 — Sequence diagram dịch văn bản**

```text
User          React           Spring Boot       FastAPI         Model
 │              │                  │                │              │
 │── type ─────→│                  │                │              │
 │              │── debounce 600ms │                │              │
 │              │── POST /api/translate ────────────→│              │
 │              │                  │── POST /internal/translate ──→│
 │              │                  │                │── segment ──→│
 │              │                  │                │── NMT ──────→│
 │              │                  │                │←─ translated ─│
 │              │                  │←── JSON ────────│              │
 │              │←── JSON ─────────│                │              │
 │←─ display ───│                  │                │              │
 │              │── POST /api/history (if logged in)│              │
```

## 4.3. Pipeline dịch Ba Na

**Hình 4.3 — Pipeline segmentation + dictionary + NMT**

```text
                    ┌─────────────────────┐
                    │  Input (Tiếng Việt) │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ underthesea         │
                    │ word_tokenize       │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Ghép từ theo        │
                    │ từ điển (longest)   │
                    └──────────┬──────────┘
                               ▼
              ┌────────────────┴────────────────┐
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │ ANCHOR          │              │ CHUNK           │
    │ Tra từ điển     │              │ BARTpho NMT     │
    │ vi → ba trực tiếp│              │ fine-tuned      │
    └────────┬────────┘              └────────┬────────┘
              │                                 │
              └────────────────┬────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Ghép kết quả        │
                    │ Output (Ba Na)      │
                    └─────────────────────┘
```

**Pipeline Ê-đê:** Bỏ qua bước segmentation và từ điển — NMT trực tiếp trên toàn câu.

## 4.4. Thiết kế cơ sở dữ liệu

**Hình 4.4 — Sơ đồ ER**

```text
users ──────────────────────────────────────────────
  │ id (PK)                                          │
  │ username, email, password, role, created_at    │
  │                                                  │
  ├──→ translation_history                           │
  │      id (PK), user_id (FK), source_lang,         │
  │      target_lang, original_text, translated_text,│
  │      is_deleted, created_at                      │
  │                                                  │
  ├──→ favorites                                     │
  │      id (PK), user_id (FK), history_id (FK),     │
  │      note, created_at                            │
  │      UNIQUE(user_id, history_id)               │
  │                                                  │
  ├──→ translation_edits                             │
  │      id (PK), user_id (FK), history_id (FK),     │
  │      suggested_translation, created_at           │
  │                                                  │
  └──→ translation_votes                             │
         id (PK), user_id (FK), history_id (FK),     │
         vote_type (UPVOTE/DOWNVOTE), created_at     │
         UNIQUE(user_id, history_id)                 │
```

**Thiết kế normalized:** Favorites, edits, votes tham chiếu `history_id` thay vì lưu trùng nội dung dịch — tránh data redundancy.

## 4.5. Thiết kế bảo mật

| Thành phần | Thiết kế |
|------------|----------|
| Authentication | JWT Bearer token, HS256, expiry 24h |
| Password | BCrypt hash trước khi lưu DB |
| Authorization | Public: dịch/OCR/TTS; Protected: history/favorites/feedback; Admin: stats |
| Session | Stateless — không dùng server session |
| CORS | Whitelist localhost dev ports |

---

# CHƯƠNG 5: DỮ LIỆU VÀ HUẤN LUYỆN MÔ HÌNH

## 5.1. Nguồn dữ liệu

### 5.1.1. Corpus Ba Na

**Bảng 5.1 — Thống kê dữ liệu huấn luyện**

| Tập | Số cặp câu | File |
|-----|------------|------|
| Train | 9.275 | `train.vi`, `train.ba` |
| Valid | 1.988 | `valid.vi`, `valid.ba` |
| Test | 1.987 | `test.vi`, `test.ba` |
| **Tổng** | **13.250** | |
| Từ điển song ngữ | 13.030 cặp | `dict.vi`, `dict.ba` |

Vị trí: `dataset/dataset-Bahnaric/`

### 5.1.2. Corpus Ê-đê

| Tập | Số cặp (ước lượng) | File |
|-----|---------------------|------|
| Valid | ~1.000 | `colab_dataset_ede/valid_data.json` |
| Test | ~1.000 | `colab_dataset_ede/test_data.json` |

Dữ liệu Ê-đê nhỏ hơn Ba Na và chưa có từ điển song ngữ bổ trợ.

### 5.1.3. Tiền xử lý

Script `prepare_colab_data.py` chuyển corpus raw sang JSON format:

```json
[
  { "vi": "Xin chào", "ba": "Hơi nơi" },
  ...
]
```

Output: `python-inference/colab_dataset_bahnaric/` và `colab_dataset_ede/`.

## 5.2. Mô hình và siêu tham số

### 5.2.1. Base model

- **Model:** `vinai/bartpho-word`
- **Kiến trúc:** BART (Encoder-Decoder Transformer)
- **Tokenizer:** Word-level, SentencePiece

### 5.2.2. Chiến lược fine-tune

- **Frozen encoder:** `requires_grad = False` cho encoder — giảm overfit trên corpus nhỏ.
- **Train decoder + cross-attention:** Chỉ cập nhật phần sinh văn bản.

**Bảng 5.2 — Siêu tham số huấn luyện**

| Tham số | Giá trị |
|---------|---------|
| `max_length` | 256 |
| `learning_rate` | 2e-5 |
| `batch_size` (train/eval) | 16 |
| `weight_decay` | 0.01 |
| `num_train_epochs` | 10 |
| `eval_strategy` | epoch |
| `fp16` | True |
| `predict_with_generate` | True |
| `save_total_limit` | 3 |
| Optimizer | AdamW (via Trainer) |
| GPU | Google Colab T4 |

### 5.2.3. Quy trình huấn luyện

**Hình 5.1 — Quy trình train trên Colab**

```text
1. Zip colab_dataset (train/valid/test JSON)
2. Upload lên Google Colab
3. Chọn runtime GPU T4
4. Cài transformers, datasets, underthesea
5. Load vinai/bartpho-word, freeze encoder
6. Tokenize parallel corpus
7. Seq2SeqTrainer.train() — 10 epochs
8. Save model → zip → download
9. Giải nén vào python-inference/models/
```

### 5.2.4. Bốn model triển khai

| Thư mục | Hướng dịch | Input → Target |
|---------|------------|----------------|
| `best-vntobana-model` | Việt → Ba Na | vi → ba |
| `best-banatovn-model` | Ba Na → Việt | ba → vi |
| `best-vntoede-model` | Việt → Ê-đê | vi → ede |
| `best-edetovn-model` | Ê-đê → Việt | ede → vi |

---

# CHƯƠNG 6: TRIỂN KHAI HỆ THỐNG

## 6.1. Công nghệ sử dụng

**Bảng 6.1 — Stack công nghệ**

| Layer | Công nghệ | Phiên bản |
|-------|-----------|-----------|
| Frontend | React, TypeScript, Vite | 19, 5.9, 8 |
| UI chart | Recharts | 3.8 |
| Backend | Spring Boot, Java | 3.3.3, 17 |
| Security | Spring Security, JWT (jjwt) | — |
| ORM | Spring Data JPA, Hibernate | — |
| Database | MySQL | 8.x |
| ML inference | FastAPI, Uvicorn | 0.135, 0.42 |
| Deep learning | PyTorch, Transformers | ≥2.0, ≥4.40 |
| NLP | underthesea | ≥6.8 |
| OCR | Tesseract, pytesseract | — |
| TTS | gTTS | 2.5 |
| Documents | python-docx, PyMuPDF | — |

## 6.2. Triển khai Backend Java

### 6.2.1. Cấu trúc package

```text
com.example.translate
├── controller/     (8 controllers)
├── service/        (TranslateService)
├── repository/     (5 repositories)
├── entity/         (5 entities + VoteType enum)
├── dto/            (request/response objects)
├── security/       (JWT filter, config, UserDetails)
├── config/         (RestTemplate bean)
└── exception/      (RestExceptionHandler, ApiError)
```

### 6.2.2. API Gateway pattern

`TranslateService` dùng `RestTemplate` proxy tất cả ML endpoints sang Python:

```java
public TranslateResponse translate(TranslateRequest req) {
    String url = pythonBaseUrl + "/internal/translate";
    // POST JSON → parse response
    // Catch HttpStatusCodeException → UpstreamServiceException
}
```

Lợi ích: Frontend chỉ gọi một origin (`:8080`), Python có thể thay đổi nội bộ mà không ảnh hưởng contract.

### 6.2.3. Phục vụ static UI

Sau `npm run build`, script `copyToBackend.mjs` copy `frontend/dist/` sang `backend-java/src/main/resources/static/`. Spring Boot phục vụ SPA tại `http://localhost:8080/`.

## 6.3. Triển khai Python Inference

### 6.3.1. ModelManager

Quản lý vòng đời model:
- Lazy-load khi cần dịch hướng mới.
- Unload model cũ (del + gc.collect()) trước khi load model mới.
- Tránh OOM khi chạy 4 model trên máy RAM hạn chế.

### 6.3.2. Inference flow

```python
def translate(text, source_lang, target_lang):
    if vi → bna:
        segments = segmenter.segment(text)
        for each segment:
            if ANCHOR: dict lookup
            if CHUNK:  _translate_chunk(text, 'vi-bana')
        return join(results)
    elif vi → ede:
        return _translate_chunk(text, 'vi-ede')
    # ... các hướng khác
```

### 6.3.3. OCR pipeline

1. Tesseract `image_to_data()` → words + bounding boxes.
2. Group words theo `(block_num, par_num, line_num)`.
3. Mỗi line → `translate()` → block `{originalText, translatedText, x, y, w, h}`.
4. Frontend overlay bản dịch lên ảnh gốc.

## 6.4. Triển khai Frontend

### 6.4.1. Component chính

| Component | Chức năng |
|-----------|-----------|
| `App.tsx` | Layout chính, state, API calls |
| `AuthContext.tsx` | JWT persistence localStorage |
| `AuthModal.tsx` | Login/register + guest sync |
| `FeedbackModal.tsx` | Edit suggestion / vote |
| `AdminDashboard.tsx` | Stats + charts |
| `VirtualKeyboard.tsx` | Custom keyboard Ba Na/Ê-đê/Khmer |

### 6.4.2. Dual-mode data

- **Guest:** `localStorage` keys `translate_history`, `translate_favorites`
- **Authenticated:** REST API `/api/history`, `/api/favorites`
- **Sync:** On login → `POST /api/sync-guest-data` merge data

## 6.5. Cấu hình và chạy hệ thống

```powershell
# Terminal 1 — Python
cd python-inference
.\.venv\Scripts\uvicorn app.main:app --host 127.0.0.1 --port 8001

# Terminal 2 — Java
cd backend-java
mvn spring-boot:run -DskipTests

# Browser
http://127.0.0.1:8080/
```

---

# CHƯƠNG 7: KẾT QUẢ VÀ ĐÁNH GIÁ

## 7.1. Kết quả đạt được

### 7.1.1. Hệ thống

- ✅ Ứng dụng web chạy end-to-end trên local.
- ✅ 4 model NMT fine-tuned triển khai thành công.
- ✅ Pipeline Ba Na kết hợp từ điển + segmentation + NMT hoạt động.
- ✅ Đầy đủ tính năng: OCR, document, TTS, auth, crowdsourcing, admin.

### 7.1.2. Tính năng đã demo

| Tính năng | Kết quả |
|-----------|---------|
| Dịch Việt → Ba Na | Hoạt động, kết hợp dict + NMT |
| Dịch Ba Na → Việt | Hoạt động |
| Dịch Việt → Ê-đê | Hoạt động (NMT trực tiếp) |
| Dịch Ê-đê → Việt | Hoạt động |
| OCR ảnh | Hoạt động (Tesseract + overlay) |
| Dịch PDF/DOCX | Hoạt động |
| TTS | Hoạt động (gTTS tiếng Việt) |
| Đăng nhập + lịch sử | Hoạt động |
| Feedback + Admin | Hoạt động |

## 7.2. Đánh giá chất lượng dịch (định tính)

> *Bổ sung cặp câu thực tế từ tập test khi hoàn thiện báo cáo.*

**Bảng 7.1 — Ví dụ dịch Ba Na (Việt → Ba Na)**

| STT | Tiếng Việt (Input) | Ba Na (Output) | Nhận xét |
|-----|---------------------|----------------|----------|
| 1 | Xin chào | [Điền kết quả] | |
| 2 | Cảm ơn bạn | [Điền kết quả] | |
| 3 | Tôi yêu gia đình | [Điền kết quả] | |
| 4 | Hôm nay trời đẹp | [Điền kết quả] | |
| 5 | [Câu dài hơn] | [Điền kết quả] | Test segmentation |

**Bảng 7.2 — Ví dụ dịch Ê-đê (Việt → Ê-đê)**

| STT | Tiếng Việt (Input) | Ê-đê (Output) | Nhận xét |
|-----|---------------------|---------------|----------|
| 1 | Xin chào | [Điền kết quả] | |
| 2 | Cảm ơn | [Điền kết quả] | |
| 3 | [Câu mẫu] | [Điền kết quả] | |

### 7.2.1. Nhận xét chất lượng

**Điểm mạnh:**
- Câu ngắn, từ phổ biến có trong từ điển → dịch chính xác nhờ dictionary lookup.
- Pipeline segmentation giúp xử lý câu dài hơn so với NMT thuần.

**Hạn chế:**
- Câu phức tạp, ngữ cảnh rộng → NMT có thể sai.
- Corpus nhỏ → model chưa generalize tốt cho OOV.
- Ê-đê không có từ điển bổ trợ → phụ thuộc hoàn toàn vào NMT.

### 7.2.2. So sánh pipeline Ba Na: có/không từ điển

| Phương pháp | Câu có từ trong dict | Câu OOV |
|-------------|----------------------|---------|
| NMT thuần | Có thể sai từ phổ biến | Sai nhiều |
| Dict + NMT (đồ án) | Chính xác (lookup) | NMT xử lý phần còn lại |

## 7.3. Đánh giá hệ thống

| Tiêu chí | Đánh giá |
|----------|----------|
| Kiến trúc | Tốt — 3 tầng rõ ràng |
| Tính năng | Rất tốt — vượt mức dịch câu đơn giản |
| ML pipeline | Tốt — có chiều sâu cho low-resource |
| UI/UX | Khá — gần Google Translate |
| Bảo mật | Chấp nhận được (dev) |
| Tài liệu | Khá — README, training guide, outline |
| Test | Yếu — chưa có test tự động |

## 7.4. So sánh với mục tiêu ban đầu

| Mục tiêu | Hoàn thành |
|----------|------------|
| Fine-tune BARTpho 4 hướng | ✅ |
| Pipeline dict + segmentation Ba Na | ✅ |
| Giao diện web Google Translate-like | ✅ |
| OCR, document, TTS | ✅ |
| Auth, history, crowdsourcing | ✅ |
| BLEU evaluation formal | ❌ (hướng phát triển) |
| Deploy cloud | ❌ (ngoài phạm vi) |

---

# CHƯƠNG 8: KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN

## 8.1. Kết luận

Đồ án đã xây dựng thành công **hệ thống dịch thuật máy cho ngôn ngữ dân tộc thiểu số Việt Nam**, hỗ trợ dịch giữa Tiếng Việt và Ba Na, Ê-đê. Hệ thống sử dụng mô hình BARTpho fine-tuned kết hợp từ điển song ngữ và word segmentation cho pipeline Ba Na — giải pháp phù hợp với đặc thù low-resource NLP.

Về mặt hệ thống, đồ án triển khai kiến trúc 3 tầng (React — Spring Boot — FastAPI) với đầy đủ tính năng: dịch văn bản, OCR, dịch tài liệu, TTS, quản lý người dùng, lịch sử, crowdsourcing và admin dashboard. Ứng dụng chạy end-to-end trên môi trường local.

Đồ án đáp ứng mục tiêu môn học DACN1, thể hiện khả năng áp dụng kiến thức AI/NLP vào bài toán thực tiễn có ý nghĩa xã hội — hỗ trợ bảo tồn và phát triển ngôn ngữ dân tộc thiểu số.

## 8.2. Hạn chế

1. **Corpus nhỏ** — 13.250 cặp Ba Na, ~2.000 cặp Ê-đê; chất lượng dịch còn hạn chế với câu phức tạp.
2. **Không có đánh giá định lượng (BLEU)** — chỉ đánh giá định tính.
3. **Ê-đê chưa có từ điển bổ trợ** — chỉ dùng NMT trực tiếp.
4. **TTS chưa phải giọng bản địa** — dùng gTTS tiếng Việt + phonetic mapping.
5. **Chạy local** — chưa container hoá, chưa deploy cloud.
6. **Không có test tự động** — regression risk khi sửa code.
7. **UI Khmer** hiển thị nhưng chưa có model.

## 8.3. Hướng phát triển

1. **Mở rộng corpus** — thu thập thêm dữ liệu song ngữ, data augmentation.
2. **Đánh giá BLEU/chrF** — metric chuẩn trên tập test.
3. **Thêm ngôn ngữ** — Jarai, H'Mông, Khmer (có model thật).
4. **TTS giọng bản địa** — record native speaker hoặc fine-tune TTS model.
5. **Từ điển Ê-đê** — xây dựng dictionary pipeline tương tự Ba Na.
6. **Deploy cloud** — Docker Compose, CI/CD, API public.
7. **Mobile app** — React Native hoặc PWA.
8. **Human-in-the-loop** — dùng feedback crowdsourcing để cải thiện model.

---

# PHỤ LỤC A: HƯỚNG DẪN CÀI ĐẶT

Xem chi tiết tại `README.md`. Tóm tắt:

1. Clone repository
2. Tải 4 model từ Google Drive → `python-inference/models/`
3. `pip install -r requirements.txt` (Python 3.10+)
4. `npm install && npm run build` (frontend)
5. Tạo MySQL database `translate_app`
6. Chạy Python `:8001` → Java `:8080` → mở browser

---

# PHỤ LỤC B: DANH MỤC API

| Method | Endpoint | Auth | Mô tả |
|--------|----------|------|-------|
| POST | `/api/translate` | No | Dịch văn bản |
| POST | `/api/examples` | No | Ví dụ từ corpus |
| POST | `/api/tts` | No | Text-to-speech |
| POST | `/api/ocr` | No | OCR + dịch ảnh |
| POST | `/api/document` | No | Dịch tài liệu |
| POST | `/api/auth/register` | No | Đăng ký |
| POST | `/api/auth/login` | No | Đăng nhập |
| GET | `/api/history` | JWT | Lấy lịch sử |
| POST | `/api/history` | JWT | Lưu lịch sử |
| DELETE | `/api/history/{id}` | JWT | Xoá 1 mục |
| DELETE | `/api/history` | JWT | Xoá tất cả |
| GET | `/api/favorites` | JWT | Lấy yêu thích |
| POST | `/api/favorites` | JWT | Toggle yêu thích |
| POST | `/api/sync-guest-data` | JWT | Đồng bộ guest |
| POST | `/api/feedback/edit` | JWT | Đề xuất sửa |
| POST | `/api/feedback/vote` | JWT | Bình chọn |
| GET | `/api/admin/stats` | ADMIN | Thống kê |
| GET | `/api/admin/recent-feedback` | ADMIN | Feedback gần đây |

---

# PHỤ LỤC C: SƠ ĐỒ CƠ SỞ DỮ LIỆU

Xem `database.sql` và Chương 4.4. Lưu ý: JPA entity có thêm cột `users.role` và `translation_history.is_deleted` so với file SQL gốc.

---

# TÀI LIỆU THAM KHẢO

1. Vaswani, A., et al. (2017). *Attention Is All You Need.* NeurIPS.
2. Lewis, M., et al. (2020). *BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension.* ACL.
3. Nguyen, D. Q., & Nguyen A. Tuan. (2020). *PhoBERT: Pre-trained language models for Vietnamese.* Findings of EMNLP.
4. VinAI Research. *BARTpho: Pre-trained seq2seq models for Vietnamese.* Hugging Face Model Hub.
5. Luong, M.-T., & Manning, C. D. (2015). *Neural Machine Translation Systems for Low-resource Languages.* ICML Workshop.
6. Spring Boot Documentation. https://spring.io/projects/spring-boot
7. Hugging Face Transformers Documentation. https://huggingface.co/docs/transformers
8. FastAPI Documentation. https://fastapi.tiangolo.com
9. React Documentation. https://react.dev
10. Underthesea — Vietnamese NLP toolkit. https://github.com/undertheseanlp/underthesea

---

**Tài liệu liên quan trong repository:**

| File | Nội dung |
|------|----------|
| `README.md` | Hướng dẫn cài đặt và chạy |
| `CODE_REVIEW_CHUYEN_SAU.md` | Code review chuyên sâu |
| `BAO_CAO_REVIEW.md` | Review cấp hệ thống |
| `BAO_VE_OUTLINE.md` | Outline slide bảo vệ |
| `Colab_Training_Guide.md` | Hướng dẫn train Colab |

---

*Báo cáo này là bản markdown — chuyển sang Word/PDF để nộp chính thức. Điền thông tin cá nhân, chèn screenshot và bảng kết quả dịch thực tế trước khi nộp.*
