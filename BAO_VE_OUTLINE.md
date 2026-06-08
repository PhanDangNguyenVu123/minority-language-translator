# Outline Slide Bảo Vệ Đồ Án

**Đề tài:** Hệ thống dịch thuật máy cho ngôn ngữ dân tộc thiểu số (Ba Na, Ê-đê)

**Thời lượng gợi ý:** 15–20 phút trình bày + 5–10 phút Q&A

---

## Slide 1 — Trang bìa

- Tên đề tài: *Hệ thống dịch thuật máy cho ngôn ngữ dân tộc thiểu số (Ba Na, Ê-đê)*
- Sinh viên thực hiện, GVHD, khoa/lớp, năm học

---

## Slide 2 — Mục lục

1. Đặt vấn đề
2. Mục tiêu & phạm vi
3. Cơ sở lý thuyết
4. Kiến trúc hệ thống
5. Pipeline ML
6. Demo tính năng
7. Kết quả & đánh giá
8. Hạn chế & hướng phát triển
9. Kết luận

---

## Phần 1 — Đặt vấn đề (2 slide)

### Slide 3 — Bối cảnh

- Việt Nam có 54 dân tộc, nhiều ngôn ngữ thiểu số ít tài nguyên số
- Google Translate / các công cụ phổ biến **chưa hỗ trợ** Ba Na, Ê-đê
- Nhu cầu: bảo tồn ngôn ngữ, hỗ trợ giao tiếp, giáo dục, nghiên cứu

### Slide 4 — Vấn đề cần giải quyết

- Dữ liệu song ngữ **hạn chế** → khó train model chất lượng cao
- Từ ghép dài, ít dấu cách (đặc biệt Ba Na)
- Cần hệ thống **end-to-end**: không chỉ model, mà còn UI, lưu trữ, phản hồi người dùng

---

## Phần 2 — Mục tiêu & phạm vi (1 slide)

### Slide 5 — Mục tiêu

**Mục tiêu chung:** Xây dựng ứng dụng web dịch Việt ↔ Ba Na, Việt ↔ Ê-đê dùng NMT fine-tuned.

**Mục tiêu cụ thể:**

- Fine-tune BARTpho trên corpus song ngữ
- Kết hợp từ điển + segmentation cho Ba Na
- Xây dựng giao diện kiểu Google Dịch
- Hỗ trợ OCR, dịch tài liệu, TTS, lịch sử, crowdsourcing

**Phạm vi:** 4 hướng dịch (`vi↔bna`, `vi↔ede`), chạy local, đồ án học thuật.

---

## Phần 3 — Cơ sở lý thuyết (2–3 slide)

### Slide 6 — Neural Machine Translation (NMT)

- Seq2Seq, Encoder–Decoder
- BARTpho (`vinai/bartpho-word`) — pre-trained cho tiếng Việt
- Fine-tuning trên parallel corpus

### Slide 7 — Xử lý ngôn ngữ Ba Na

- **Underthesea** — tokenization tiếng Việt
- **Từ điển song ngữ** — tra trực tiếp từ khớp
- **Segmentation + NMT theo chunk** — xử lý từ ghép / OOV

Sơ đồ gợi ý:

```text
Input → Segment → Dictionary lookup (ANCHOR) | NMT (CHUNK) → Output
```

### Slide 8 — Công nghệ sử dụng

| Layer | Công nghệ |
|-------|-----------|
| Frontend | React, TypeScript, Vite |
| Backend | Spring Boot, JWT, MySQL |
| Inference | FastAPI, Transformers, PyTorch |
| OCR | Tesseract |
| Training | Google Colab, BARTpho |

---

## Phần 4 — Kiến trúc hệ thống (2 slide)

### Slide 9 — Sơ đồ tổng quan

```text
Browser → Spring Boot :8080 → FastAPI :8001 → BARTpho models
                ↓
             MySQL (users, history, feedback)
```

Giải thích vai trò từng tầng:

- **React:** UI, guest mode, bàn phím ảo
- **Spring Boot:** auth, proxy, lưu DB
- **FastAPI:** inference ML, OCR, TTS

### Slide 10 — Luồng xử lý dịch văn bản

1. User nhập text → debounce 600ms
2. `POST /api/translate` (Java)
3. Forward `POST /internal/translate` (Python)
4. Pipeline dịch → trả JSON
5. Lưu lịch sử (nếu đã đăng nhập)

---

## Phần 5 — Pipeline ML (2–3 slide)

> Phần quan trọng nhất — nên dành thời gian trình bày kỹ.

### Slide 11 — Dữ liệu huấn luyện

- Corpus Ba Na: `dataset/dataset-Bahnaric/` (train/valid/test)
- Corpus Ê-đê: `python-inference/colab_dataset_ede/`
- Từ điển song ngữ Ba Na: `dict.vi`, `dict.ba`
- Tiền xử lý → JSON cho Colab

### Slide 12 — Huấn luyện model

- Base model: `vinai/bartpho-word`
- 4 model: `vi→bna`, `bna→vi`, `vi→ede`, `ede→vi`
- Colab GPU T4, `Seq2SeqTrainer`
- Notebook: `train-model/`
- Hướng dẫn: `Colab_Training_Guide.md`

Thư mục model sau khi deploy:

| Thư mục | Hướng dịch |
|---------|------------|
| `best-vntobana-model/` | Việt → Ba Na |
| `best-banatovn-model/` | Ba Na → Việt |
| `best-vntoede-model/` | Việt → Ê-đê |
| `best-edetovn-model/` | Ê-đê → Việt |

### Slide 13 — Pipeline inference Ba Na

```text
"Câu tiếng Việt"
    → underthesea tokenize
    → ghép từ theo từ điển (ANCHOR)
    → NMT cho phần còn lại (CHUNK)
    → ghép kết quả
```

So sánh ngắn:

- **Ba Na:** dictionary + segmentation + NMT theo chunk
- **Ê-đê:** NMT trực tiếp trên toàn câu (chưa có từ điển)

---

## Phần 6 — Tính năng & Demo (2 slide)

### Slide 14 — Bảng tính năng

| Tính năng | Mô tả ngắn |
|-----------|------------|
| Dịch text | Auto-translate, 4 hướng |
| OCR ảnh | Tesseract + dịch overlay |
| Dịch file | `.txt`, `.docx`, `.pdf` |
| TTS | gTTS + chuyển phonetic |
| Auth | JWT, lịch sử, yêu thích |
| Feedback | Đề xuất sửa, vote |
| Admin | Dashboard thống kê |

### Slide 15 — Kịch bản demo (live)

1. Dịch câu Việt → Ba Na (minh hoạ segmentation)
2. Đổi hướng Ba Na → Việt
3. Dịch Việt → Ê-đê
4. Upload ảnh OCR hoặc file PDF
5. Đăng nhập → xem lịch sử → gửi feedback

**Lưu ý khi demo:**

- Chạy Python (`:8001`) và Java (`:8080`) trước khi vào phòng
- Chuẩn bị sẵn câu mẫu, ảnh, file PDF
- Kiểm tra model đã tải đủ trong `python-inference/models/`

---

## Phần 7 — Kết quả & đánh giá (2 slide)

### Slide 16 — Kết quả đạt được

- Hệ thống web chạy end-to-end
- 4 model NMT fine-tuned, deploy local
- Pipeline Ba Na kết hợp từ điển + NMT
- Đầy đủ tính năng phụ: OCR, doc, TTS, auth, crowdsourcing

### Slide 17 — Đánh giá chất lượng dịch

Nội dung gợi ý (tuỳ dữ liệu thực tế):

- Ví dụ input/output trên tập test
- BLEU score (nếu đã tính)
- So sánh có/không từ điển (Ba Na)
- 5–10 cặp câu mẫu + nhận xét định tính

---

## Phần 8 — Hạn chế & hướng phát triển (1 slide)

### Slide 18

**Hạn chế:**

- Corpus nhỏ → chất lượng dịch còn hạn chế
- Ê-đê chưa có từ điển, chỉ NMT
- TTS chưa phải giọng bản địa
- Chạy local, chưa container hoá (Docker)
- Khmer trên UI chưa có model thật

**Hướng phát triển:**

- Mở rộng corpus, thêm ngôn ngữ dân tộc khác
- Đánh giá BLEU / human evaluation có hệ thống
- Deploy cloud, cung cấp API công khai
- TTS giọng bản địa

---

## Phần 9 — Kết luận (1 slide)

### Slide 19 — Kết luận

- Đã xây dựng hệ thống dịch thuật cho ngôn ngữ thiểu số
- Fine-tune BARTpho + pipeline segmentation cho Ba Na
- Ứng dụng web đầy đủ tính năng, phục vụ mục tiêu bảo tồn & hỗ trợ giao tiếp
- **Cảm ơn & Q&A**

---

## Phụ lục — Câu hỏi thường gặp (dự phòng Q&A)

| Câu hỏi có thể | Gợi ý trả lời |
|----------------|---------------|
| Tại sao chọn BARTpho? | Pre-trained tiếng Việt, phù hợp low-resource NMT |
| Tại sao cần từ điển + NMT? | Corpus nhỏ, từ ghép dài, giảm OOV |
| BLEU bao nhiêu? | Nêu số nếu có, hoặc đánh giá định tính trên tập test |
| Tại sao tách Spring Boot và Python? | Python cho ML/inference, Java cho auth/DB/API gateway ổn định |
| Dữ liệu lấy từ đâu? | Corpus Ba Na trong `dataset/dataset-Bahnaric/`, Ê-đê trong `colab_dataset_ede/` |
| Bảo mật JWT? | Stateless token, phân quyền ADMIN, config dev cho môi trường local |
| ModelManager làm gì? | Lazy-load model, unload khi đổi hướng dịch để tiết kiệm RAM |
| Crowdsourcing hoạt động thế nào? | User đề xuất sửa bản dịch → vote up/down → admin xem thống kê |

---

## Checklist trước ngày bảo vệ

- [ ] README và demo path đã kiểm tra trên máy demo
- [ ] 4 model đã tải đủ vào `python-inference/models/`
- [ ] MySQL đang chạy, database `translate_app` sẵn sàng
- [ ] Python inference + Spring Boot chạy ổn định
- [ ] Slide có sơ đồ kiến trúc và pipeline ML
- [ ] Có 5–10 cặp câu demo chất lượng tốt
- [ ] Đã thử demo OCR / document ít nhất 1 lần
- [ ] Tài khoản demo (user + admin) đã tạo sẵn
