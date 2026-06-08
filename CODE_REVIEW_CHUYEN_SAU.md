# CODE REVIEW CHUYÊN SÂU

**Dự án:** translate-app — Hệ thống dịch thuật ngôn ngữ dân tộc thiểu số

**Phạm vi:** Toàn bộ mã nguồn `frontend/`, `backend-java/`, `python-inference/`

**Phương pháp:** Static code review — đọc mã nguồn, phân tích luồng dữ liệu, kiểm tra bảo mật, đánh giá maintainability

**Ngày review:** 31/05/2026

---

## Mục lục

1. [Tóm tắt điều hành](#1-tóm-tắt-điều-hành)
2. [Phạm vi và phương pháp](#2-phạm vi-và-phương-pháp)
3. [Kiến trúc và luồng dữ liệu](#3-kiến-trúc-và-luồng-dữ-liệu)
4. [Review Backend Java](#4-review-backend-java)
5. [Review Frontend React](#5-review-frontend-react)
6. [Review Python Inference](#6-review-python-inference)
7. [Audit bảo mật](#7-audit-bảo-mật)
8. [Database và JPA](#8-database-và-jpa)
9. [Issue Register](#9-issue-register)
10. [Code smells và technical debt](#10-code-smells-và-technical-debt)
11. [Khuyến nghị refactor](#11-khuyến-nghị-refactor)
12. [Ma trận ưu tiên sửa lỗi](#12-ma-trận-ưu-tiên-sửa-lỗi)

---

## 1. Tóm tắt điều hành

### 1.1. Kết luận chung

Codebase **đạt chất lượng tốt cho đồ án môn học**, với kiến trúc 3 tầng rõ ràng và pipeline ML có chiều sâu. Backend Java tuân thủ layering convention; Python inference có thiết kế hợp lý cho low-resource NMT. Frontend hoạt động đầy đủ tính năng nhưng mang technical debt do monolithic structure.

### 1.2. Thống kê nhanh

| Metric | Giá trị |
|--------|---------|
| File Java nguồn | ~47 |
| Dòng `App.tsx` | ~1.584 |
| Module Python inference | 7 file chính |
| API endpoints (Java public) | 15 |
| Issue tìm thấy | 17 (2 High, 8 Medium, 7 Low) |
| Test coverage | ~0% (không có test tự động) |

### 1.3. Điểm mạnh kỹ thuật

- Separation of concerns giữa UI / API gateway / ML inference
- `ModelManager` lazy-load/unload model — thiết kế tốt cho RAM hạn chế
- Pipeline Ba Na kết hợp rule-based + neural — phù hợp low-resource
- JWT stateless, BCrypt password, soft-delete history
- Error forwarding từ Python qua `UpstreamServiceException`

### 1.4. Rủi ro chính

- **IDOR** trên favorites và feedback (không kiểm tra ownership của `historyId`)
- **TTS bug** — frontend luôn gửi `lang: 'vi'`
- **UI/backend mismatch** — Khmer hiển thị nhưng không có model
- **Config dev-only** — JWT secret hardcode, MySQL root rỗng
- **Không có test** — regression risk cao khi sửa code

---

## 2. Phạm vi và phương pháp

### 2.1. Phạm vi

| Module | Đường dẫn | Đã review |
|--------|-----------|-----------|
| Frontend | `frontend/src/` | App.tsx, AuthContext, 3 components, VirtualKeyboard |
| Backend | `backend-java/src/main/java/` | 8 controllers, security, service, 5 entities |
| Inference | `python-inference/app/` | main.py, translator, segmenter, ocr, document, tts, examples |
| Config | `application.properties`, `requirements.txt`, `database.sql` | Có |
| Training | `Colab_Training_Guide.md`, `prepare_*.py` | Có |

### 2.2. Tiêu chí đánh giá

- **Correctness** — logic đúng, edge cases
- **Security** — auth, authorization, input validation, secrets
- **Maintainability** — modularity, naming, duplication
- **Performance** — N+1 queries, model memory, debounce
- **Reliability** — error handling, fallbacks
- **Consistency** — naming, API contract, schema drift

### 2.3. Thang mức độ nghiêm trọng

| Mức | Ý nghĩa |
|-----|---------|
| **Critical** | Lỗ hổng bảo mật nghiêm trọng, data loss, crash production |
| **High** | Bug chức năng rõ ràng, ảnh hưởng UX/demo |
| **Medium** | Rủi ro bảo mật dev, maintainability, inconsistency |
| **Low** | Code smell, documentation, minor edge case |
| **Info** | Ghi chú, không cần sửa ngay |

---

## 3. Kiến trúc và luồng dữ liệu

### 3.1. Topology

```text
┌─────────────────────────────────────────────────────────────┐
│  Browser (React SPA)                                        │
│  - AuthContext (localStorage JWT)                           │
│  - App.tsx: translate, OCR, doc, history, feedback          │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP /api/*
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Spring Boot :8080                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │ Controllers │→ │ Repositories │→ │ MySQL (translate_app)│ │
│  └──────┬──────┘  └──────────────┘  └─────────────────────┘ │
│         │ TranslateService (RestTemplate)                   │
│  ┌──────┴──────┐  AuthTokenFilter → JwtUtils               │
│  │ static/     │  WebSecurityConfig                         │
│  └─────────────┘                                            │
└──────────────────────────┬──────────────────────────────────┘
                           │ POST /internal/*
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  FastAPI :8001                                              │
│  translator.py ← segmenter.py + ModelManager + BARTpho      │
│  ocr.py, document.py, examples.py, tts_phonetics.py         │
└─────────────────────────────────────────────────────────────┘
```

### 3.2. Luồng dịch văn bản (chi tiết)

```text
1. User gõ text → handleInputChange (debounce 600ms)
2. doTranslate() → POST /api/translate
3. TranslateController → TranslateService.translate()
4. RestTemplate → POST http://localhost:8001/internal/translate
5. translator.translate(text, sourceLang, targetLang)
   ├── vi→bna: segmenter.segment() → ANCHOR|CHUNK → NMT
   ├── bna→vi: segmenter.segment_bana() → ANCHOR|CHUNK → NMT
   ├── vi→ede: _translate_chunk(clean, 'vi-ede')
   └── ede→vi: _translate_chunk(clean, 'ede-vi')
6. Response JSON → UI render outputText
7. Nếu logged-in: POST /api/history (dedup by user+langs+texts)
```

### 3.3. Luồng xác thực

```text
Register: POST /api/auth/register → BCrypt hash → User (ROLE_USER default)
Login:    POST /api/auth/login → AuthenticationManager → JwtUtils.generateJwtToken()
Request:  Authorization: Bearer <token>
          → AuthTokenFilter.doFilterInternal()
          → JwtUtils.validateJwtToken() + getUserNameFromJwtToken()
          → UserDetailsServiceImpl.loadUserByUsername()
          → SecurityContextHolder.setAuthentication()
Protected: @PreAuthorize("hasAuthority('ROLE_ADMIN')") on AdminController
```

### 3.4. Đánh giá kiến trúc

| Khía cạnh | Đánh giá | Chi tiết |
|-----------|----------|----------|
| Coupling | Tốt | Java không import Python; contract JSON |
| Cohesion | Khá | Backend tách lớp tốt; frontend gom quá nhiều |
| Scalability | Trung bình | Python single-process; ModelManager 1 model/lúc |
| Deployability | Trung bình | 2 process + MySQL; không container hoá |
| Observability | Yếu | Không structured logging, metrics, health check |

---

## 4. Review Backend Java

### 4.1. Controllers

#### `TranslateController.java`

**Vai trò:** Proxy public endpoints sang Python.

**Đánh giá:**
- ✅ Thin controller — delegate sang `TranslateService`
- ✅ Validation qua DTO (`@NotBlank`, `@Size(max=5000)`)
- ⚠️ Tất cả endpoint dịch thuật public — không rate limit (chấp nhận được cho đồ án)

#### `AuthController.java`

**Vai trò:** Login/register.

**Đánh giá:**
- ✅ BCrypt qua `PasswordEncoder`
- ✅ Trả JWT + user info trong `JwtResponse`
- ⚠️ Register không validate email format ngoài `@Email` annotation
- ⚠️ Không có endpoint refresh token
- ⚠️ Không có cơ chế promote user lên ADMIN qua API

#### `HistoryController.java`

**Vai trò:** CRUD lịch sử dịch (JWT required).

**Đánh giá:**
- ✅ Soft-delete (`isDeleted = true`) thay vì xóa cứng
- ✅ Dedup khi POST — kiểm tra user + langs + texts trùng
- ✅ DELETE `/{id}` kiểm tra ownership (`history.getUser().getId()`)
- ✅ DELETE all chỉ xóa của user hiện tại

#### `FavoriteController.java`

**Vai trò:** Toggle favorite theo `historyId`.

**Đánh giá:**
- ✅ Toggle logic (add/remove) hoạt động
- ❌ **IDOR:** `findById(historyId)` không kiểm tra `history.user.id == currentUser.id`
- ⚠️ Query tất cả favorites rồi filter in-memory (line 51-52) — O(n) không cần thiết

```47:48:backend-java/src/main/java/com/example/translate/controller/FavoriteController.java
        TranslationHistory history = historyRepository.findById(historyId)
                .orElseThrow(() -> new RuntimeException("History not found"));
```

**Khuyến nghị:** Thêm `if (!history.getUser().getId().equals(user.getId())) throw Forbidden`.

#### `FeedbackController.java`

**Vai trò:** Submit edit suggestion và vote.

**Đánh giá:**
- ✅ Vote dedup — kiểm tra `findByUserIdAndHistoryId` trước khi save
- ❌ **IDOR:** Edit và vote không kiểm tra ownership của `historyId`
- ⚠️ `RuntimeException` thay vì custom exception → 500 thay vì 404

```47:48:backend-java/src/main/java/com/example/translate/controller/FeedbackController.java
        TranslationHistory history = historyRepository.findById(editRequest.getHistoryId())
                .orElseThrow(() -> new RuntimeException("Error: History not found."));
```

#### `SyncController.java`

**Vai trò:** Merge guest localStorage data vào server khi login.

**Đánh giá:**
- ✅ Logic merge history hợp lý
- ⚠️ Favorite sync có thể vi phạm `UNIQUE(user_id, history_id)` nếu duplicate
- ⚠️ Không transaction wrapper — partial sync nếu lỗi giữa chừng

#### `AdminController.java`

**Vai trò:** Stats và recent feedback.

**Đánh giá:**
- ✅ `@PreAuthorize("hasAuthority('ROLE_ADMIN')")` ở class level
- ⚠️ `historyRepository.count()` đếm cả bản ghi soft-deleted
- ⚠️ EAGER fetch history trong votes/edits — N+1 potential với list lớn

### 4.2. Security layer

#### `WebSecurityConfig.java`

```62:69:backend-java/src/main/java/com/example/translate/security/WebSecurityConfig.java
                .authorizeHttpRequests(auth ->
                        auth.requestMatchers("/api/auth/**").permitAll()
                                .requestMatchers("/api/translate/**").permitAll()
                                .requestMatchers("/api/tts/**").permitAll()
                                .requestMatchers("/api/ocr/**").permitAll()
                                .requestMatchers("/api/document/**").permitAll()
                                .requestMatchers("/api/examples/**").permitAll()
                                .anyRequest().authenticated()
                );
```

**Đánh giá:**
- ✅ Stateless session, CSRF disabled (phù hợp JWT API)
- ✅ CORS whitelist localhost dev ports
- ⚠️ Static files (`/**`) không explicit trong config — Spring mặc định permit static
- ⚠️ `@CrossOrigin(origins = "*")` trên controllers trùng lặp với CORS config

#### `AuthTokenFilter.java`

- Parse `Bearer ` prefix
- Invalid token → log error, **continue filter chain** (request anonymous)
- Protected routes sẽ trả 401 qua `AuthEntryPointJwt` — behavior đúng
- ⚠️ Không phân biệt expired vs malformed token trong response

#### `JwtUtils.java`

- Algorithm: HS256
- Secret: hardcode trong `application.properties` line 19
- Expiry: 24 giờ (86400000 ms)
- ❌ Secret trong repo — rủi ro nếu deploy public

### 4.3. Service layer

#### `TranslateService.java`

**Đánh giá:**
- ✅ Forward error body + status từ Python
- ✅ Catch `RestClientException` → 502 "Python service unavailable"
- ⚠️ Không có timeout config trên `RestTemplate` — có thể hang nếu Python chậm
- ⚠️ Không retry logic

### 4.4. Entity layer

| Entity | Fetch type | Ghi chú |
|--------|------------|---------|
| `TranslationHistory.user` | LAZY | OK |
| `Favorite.history` | **EAGER** | Có thể gây over-fetch |
| `TranslationEdit.history` | **EAGER** | Tương tự |
| `TranslationVote.history` | **EAGER** | Tương tự |

**Unique constraints:**
- `favorites`: `(user_id, history_id)`
- `translation_votes`: `(user_id, history_id)`

---

## 5. Review Frontend React

### 5.1. Cấu trúc component

```text
main.tsx
└── AuthProvider (AuthContext.tsx)
    └── App.tsx (~1.584 dòng)
        ├── AuthModal.tsx
        ├── FeedbackModal.tsx
        ├── AdminDashboard.tsx
        └── VirtualKeyboard.tsx
```

### 5.2. State management

`App.tsx` quản lý **~25 useState hooks** — không dùng Redux/Zustand/Context ngoài auth.

| Nhóm state | Variables | Vấn đề |
|------------|-----------|--------|
| Translation | sourceLang, targetLang, inputText, outputText, loading, error | OK |
| History/Favorites | history, favorites | Dual source: localStorage vs API |
| OCR | imageFile, ocrBlocks, imageScale... | 6+ states riêng |
| Document | docFile, docResult, isDocProcessing | OK |
| Auth/Feedback | user, feedbackConfig, votedHistoryIds | OK |

**Dual data source pattern:**
- Guest: `localStorage` keys `translate_history`, `translate_favorites`
- Logged-in: fetch từ `/api/history`, `/api/favorites`
- Login trigger: `syncGuestData()` trong `AuthModal.tsx`

**Đánh giá:** Pattern hoạt động nhưng phức tạp, dễ desync nếu API fail sau login.

### 5.3. Hàm quan trọng

#### `doTranslate()` (~line 582-668)

- POST `/api/translate`
- Set outputText từ response
- Nếu authenticated: POST `/api/history`, lưu `currentHistoryId` cho feedback
- Error handling: setError state

**Vấn đề:** Không cancel in-flight request khi user gõ tiếp (race condition nhẹ với debounce).

#### `handleInputChange()` (~line 696-709)

- Debounce 600ms qua `setTimeout` + `useRef`
- Gọi `doTranslate` + `fetchExamples`

**Đánh giá:** ✅ Pattern debounce đúng.

#### `handleSpeak()` (~line 768-824)

```792:796:frontend/src/App.tsx
      const resp = await fetch('/api/tts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text, lang: 'vi' }),
      });
```

**❌ Bug High:** Luôn gửi `lang: 'vi'`. Python `tts_phonetics.py` chỉ chạy khi `lang in ["bana", "ede"]`. Kết quả: TTS output Ba Na/Ê-đê không qua phonetic mapping.

**Fix:** Gửi `lang: targetLang === 'bna' ? 'bana' : targetLang === 'ede' ? 'ede' : 'vi'`.

#### `toggleFavorite()` (~line 826-855)

- Guest: mutate localStorage
- Logged-in: POST `/api/favorites` với `historyId`
- ⚠️ Guest favorite dùng local object, logged-in dùng DB id — structure khác nhau

### 5.4. Type safety

```typescript
type LangCode = 'vi' | 'bna' | 'ede' | 'km'
```

- `km` (Khmer) trong type nhưng backend không hỗ trợ
- Không có `'en'` trong type nhưng UI có thể hiển thị English ở popular dictionaries section

### 5.5. AuthContext.tsx

- Persist `UserInfo` vào `localStorage` key `translate_user`
- ⚠️ Token không có expiry check phía client — user có thể dùng token hết hạn đến khi API trả 401
- ✅ Logout clear localStorage

### 5.6. AdminDashboard.tsx

- Fetch `/api/admin/stats` và `/api/admin/recent-feedback`
- Recharts PieChart cho vote distribution
- ✅ Tách component riêng — pattern tốt so với App.tsx monolith

### 5.7. VirtualKeyboard.tsx

- Layout riêng cho `km`, `bna`/`ede`, default
- Custom implementation — `react-simple-keyboard` trong package.json **không được sử dụng** (dead dependency)

---

## 6. Review Python Inference

### 6.1. `translator.py`

#### ModelManager

```21:33:python-inference/app/inference/translator.py
class ModelManager:
    def __init__(self):
        self.current_direction = None
        self.model = None
        self.tokenizer = None
        
        self.model_dirs = {
            'vi-bana': ... "best-vntobana-model",
            'bana-vi': ... "best-banatovn-model",
            'vi-ede':  ... "best-vntoede-model",
            'ede-vi':  ... "best-edetovn-model",
        }
```

**Đánh giá:**
- ✅ Singleton pattern, lazy load
- ✅ `_clear_memory()` — del model + gc.collect() + cuda.empty_cache()
- ✅ Cache hit khi cùng direction
- ⚠️ Inference trên CPU mặc định — không explicit `.to(device)`
- ⚠️ `model.generate(**inputs)` không set `num_beams`, `early_stopping` — quality/speed tradeoff mặc định

#### Pipeline Ba Na

**Segmentation (`segmenter.py`):**
- Load 13.030 cặp từ điển
- `segment()`: underthesea `word_tokenize` → classify ANCHOR vs CHUNK
- `segment_bana()`: greedy longest-match phrase (track `max_ba_phrase_len`)

**Đánh giá:**
- ✅ Thiết kế hợp lý cho low-resource
- ⚠️ ANCHOR chỉ check lowercase — có thể miss proper nouns
- ⚠️ Không xử lý multi-word Vietnamese compounds ngoài underthesea

#### Fallback

```174:175:python-inference/app/inference/translator.py
    return f"[{normalized_target or 'target'}-demo] {clean}"
```

Unsupported directions (km, en) → demo placeholder. Frontend vẫn hiển thị Khmer.

### 6.2. `ocr.py`

```8:11:python-inference/app/inference/ocr.py
tesseract_cmd_path = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
if os.path.exists(tesseract_cmd_path):
    pytesseract.pytesseract.tesseract_cmd = tesseract_cmd_path
```

- ❌ Hardcode Windows path — fail trên Linux/macOS nếu không trong PATH
- ✅ Group OCR words theo `(block_num, par_num, line_num)`
- ✅ Mỗi line → `translate()` riêng → bounding box overlay
- ⚠️ Lang map: bna/ede dùng `vie` — approximation, không optimal

### 6.3. `document.py`

- Hỗ trợ `.txt`, `.docx`, `.pdf`
- Batch ~2000 chars trước khi translate
- ⚠️ `translated_paragraphs.extend(t_text.split('\n'))` — có thể misalign paragraph count nếu NMT thay đổi số dòng

### 6.4. `examples.py`

- Load JSON từ `colab_dataset_bahnaric/` và `colab_dataset_ede/`
- Substring search, limit 5 results
- ⚠️ `train_data.json` không có trong repo — examples chỉ từ valid/test split

### 6.5. `tts_phonetics.py`

- Mapping ký tự Ba Na/Ê-đê → phonetic Vietnamese approximation
- Chỉ activate khi `lang in ["bana", "ede"]`
- Frontend bug (issue #1) khiến module này **không được dùng** trong TTS thực tế

### 6.6. `main.py`

- FastAPI với Pydantic validation
- Auto docs tại `/docs`
- ⚠️ Không có `/health` endpoint
- ⚠️ Exception handler generic `HTTPException(500, str(e))` — leak internal error message

### 6.7. Data prep scripts

| Script | Vấn đề |
|--------|--------|
| `prepare_colab_data.py` | Path `../dataset/dictionary/` — sai, thực tế `../dataset/dataset-Bahnaric/dictionary/` |
| `prepare_ede_dataset.py` | Hardcode `E:\devpro\JavaSpringBoot\...` — machine-specific |

---

## 7. Audit bảo mật

### 7.1. Authentication

| Kiểm tra | Kết quả |
|----------|---------|
| Password hashing | ✅ BCrypt |
| JWT algorithm | ✅ HS256 |
| Token expiry | ✅ 24h |
| Secret management | ❌ Hardcode in repo |
| Refresh token | ❌ Không có |
| Brute force protection | ❌ Không rate limit login |

### 7.2. Authorization

| Endpoint | Kiểm tra ownership | Kết quả |
|----------|-------------------|---------|
| GET/DELETE `/api/history` | ✅ Filter by user | OK |
| POST `/api/favorites` | ❌ Không check history owner | **IDOR** |
| POST `/api/feedback/edit` | ❌ Không check history owner | **IDOR** |
| POST `/api/feedback/vote` | ❌ Không check history owner | **IDOR** |
| GET `/api/admin/*` | ✅ ROLE_ADMIN | OK |

**Kịch bản IDOR:**
1. User A dịch câu → history id = 42
2. User B gửi `POST /api/favorites { historyId: 42 }` → thành công
3. User B vote/edit trên history của User A → thành công

**Mức độ:** Medium (đồ án local, không public internet — nhưng cần fix nếu demo multi-user).

### 7.3. Input validation

| Input | Validation | Ghi chú |
|-------|------------|---------|
| Translate text | `@Size(max=5000)` Java + Pydantic | OK |
| File upload OCR/doc | Không check file size/type strict | ⚠️ Potential DoS với file lớn |
| Username/email | `@NotBlank`, `@Email` | OK |
| SQL injection | JPA parameterized | OK |

### 7.4. CORS

- Config: whitelist `localhost:5173`, `localhost:3000`
- Controllers: `@CrossOrigin(origins = "*")` — **mâu thuẫn**, permissive hơn config

### 7.5. Sensitive data

| Data | Exposure |
|------|----------|
| JWT secret | `application.properties` in repo |
| User password | BCrypt hash in DB — OK |
| Translation text | Public API, no encryption at rest concern for thesis |

---

## 8. Database và JPA

### 8.1. Schema drift

| Column | `database.sql` | JPA Entity | Action needed |
|--------|----------------|------------|---------------|
| `users.role` | ❌ Missing | ✅ `User.role` VARCHAR(20) | Update SQL |
| `translation_history.is_deleted` | ❌ Missing | ✅ `TranslationHistory.isDeleted` | Update SQL |
| `vote_type` | ENUM | STRING enum | Minor |

**Runtime source of truth:** Hibernate `ddl-auto=update` — chạy Spring Boot sẽ tự thêm column.

### 8.2. ER relationships

```text
users (1) ──→ (N) translation_history
users (1) ──→ (N) favorites ──→ (1) translation_history
users (1) ──→ (N) translation_edits ──→ (1) translation_history
users (1) ──→ (N) translation_votes ──→ (1) translation_history
```

**Thiết kế tốt:** Favorites/edits/votes reference `history_id` thay vì duplicate text — normalized.

### 8.3. Index recommendations (chưa có)

- `translation_history(user_id, is_deleted, created_at)` — query history list
- `translation_votes(history_id)` — admin stats

---

## 9. Issue Register

| ID | Severity | Module | Location | Mô tả | Khuyến nghị |
|----|----------|--------|----------|-------|-------------|
| CR-01 | **High** | Frontend | `App.tsx:795` | TTS luôn gửi `lang: 'vi'` | Truyền đúng targetLang mapped |
| CR-02 | **High** | Frontend/Python | `App.tsx:16` / `translator.py:175` | Khmer trong UI, backend placeholder | Ẩn hoặc disable `km` |
| CR-03 | Medium | Security | `FavoriteController.java:47` | IDOR — favorite history của user khác | Check ownership |
| CR-04 | Medium | Security | `FeedbackController.java:47,67` | IDOR — edit/vote history của user khác | Check ownership |
| CR-05 | Medium | Security | `application.properties:19` | JWT secret hardcode | Env variable |
| CR-06 | Medium | Backend | `AdminController.java` | count() gồm soft-deleted | Filter `isDeleted=false` |
| CR-07 | Medium | Python | `ocr.py:9` | Tesseract path Windows-only | Detect OS hoặc env var |
| CR-08 | Medium | Python | `prepare_colab_data.py:35` | Sai đường dẫn dataset | Fix path |
| CR-09 | Medium | Backend | `SyncController.java` | Favorite sync duplicate | Check before insert |
| CR-10 | Low | Frontend | `App.tsx` | Monolith 1.584 dòng | Tách hooks/components |
| CR-11 | Low | Frontend | `package.json` | `react-simple-keyboard` unused | Remove dependency |
| CR-12 | Low | Backend | Controllers | `@CrossOrigin("*")` redundant | Remove annotation |
| CR-13 | Low | Backend | `TranslateService` | No RestTemplate timeout | Set connect/read timeout |
| CR-14 | Low | Python | `document.py:54` | Paragraph misalignment | Map by index not split |
| CR-15 | Low | Docs | Colab guide vs README | Model name mismatch | Standardize names |
| CR-16 | Low | Database | `database.sql` | Schema drift | Sync with entities |
| CR-17 | Info | Testing | — | Zero automated tests | Add basic pytest/JUnit |

---

## 10. Code smells và technical debt

### 10.1. God Component — `App.tsx`

**Smell:** Single component chứa UI rendering, API calls, localStorage logic, speech recognition, OCR state, document processing.

**Impact:** Khó test, khó review PR, khó onboard developer mới.

**Effort to fix:** Medium (2-3 ngày tách hooks).

### 10.2. RuntimeException thay vì domain exceptions

**Locations:** `FavoriteController`, `FeedbackController`, `SyncController`

**Smell:** `throw new RuntimeException("History not found")` → HTTP 500 thay vì 404.

### 10.3. In-memory filter thay vì query

**Location:** `FavoriteController.java:51-52`

```java
List<Favorite> userFavs = favoriteRepository.findByUserIdOrderByCreatedAtDesc(user.getId());
Favorite existing = userFavs.stream().filter(f -> f.getHistory().getId().equals(historyId))...
```

**Smell:** Load all favorites rồi filter. Nên có `findByUserIdAndHistoryId()`.

### 10.4. Magic strings

- Language codes: `"vi"`, `"bna"`, `"ede"`, `"bana"`, `"ban"` scattered across Java/Python/TS
- Role: `"ROLE_USER"`, `"ROLE_ADMIN"` — OK cho Spring convention

### 10.5. Dead code / unused dependencies

- `react-simple-keyboard` in package.json, not imported
- Commented SVG icons in `App.tsx` (lines ~85-90)

---

## 11. Khuyến nghị refactor

### 11.1. Frontend decomposition plan

```text
App.tsx (thin shell)
├── hooks/
│   ├── useTranslate.ts      → doTranslate, debounce, examples
│   ├── useHistory.ts        → history CRUD, guest/localStorage
│   ├── useFavorites.ts      → toggle favorite
│   ├── useOcr.ts            → image upload, blocks, scale
│   └── useDocument.ts       → doc upload, result
├── components/
│   ├── TranslatePanel.tsx   → input/output panes
│   ├── LanguageSelector.tsx
│   ├── HistorySidebar.tsx
│   ├── OcrViewer.tsx
│   └── DocumentViewer.tsx
```

### 11.2. Backend security fixes

```java
// Pattern cho mọi controller dùng historyId:
TranslationHistory history = historyRepository.findById(historyId)
    .orElseThrow(() -> new NotFoundException("History not found"));
if (!history.getUser().getId().equals(currentUser.getId())) {
    throw new ForbiddenException("Not authorized");
}
```

### 11.3. Python improvements

- `TESSERACT_CMD` env variable
- `/health` endpoint check model loaded
- Explicit `device = "cuda" if torch.cuda.is_available() else "cpu"`

### 11.4. Testing minimum viable

| Layer | Test | Priority |
|-------|------|----------|
| Python | `pytest` cho `translate()`, `segmenter.segment()` | High |
| Java | `@WebMvcTest` cho `/api/translate` proxy mock | Medium |
| Frontend | Vitest cho `doTranslate` mock fetch | Low |

---

## 12. Ma trận ưu tiên sửa lỗi

```text
Impact ↑
  │  CR-01 TTS bug    CR-03,04 IDOR
  │  CR-02 Khmer UI
  │
  │  CR-07 OCR path   CR-09 Sync dup
  │  CR-08 data path
  │
  │  CR-10 monolith   CR-17 no tests
  │  CR-11 dead dep
  └────────────────────────────────→ Effort
       Low              Medium           High
```

### Trước bảo vệ (Must fix / Must explain)

1. **CR-01** — Sửa TTS lang hoặc giải thích trong slide
2. **CR-02** — Ẩn Khmer hoặc ghi rõ "demo UI"
3. Demo path ổn định — model, MySQL, 2 services

### Sau bảo vệ (Nice to have)

4. CR-03, CR-04 — IDOR fixes
5. CR-10 — Frontend refactor
6. CR-17 — Basic tests

---

## Phụ lục A — File index

| File | LOC ước lượng | Complexity |
|------|---------------|------------|
| `frontend/src/App.tsx` | ~1.584 | Very High |
| `frontend/src/App.css` | ~1.571 | N/A (styles) |
| `backend-java/.../TranslateService.java` | ~174 | Low |
| `backend-java/.../WebSecurityConfig.java` | ~90 | Medium |
| `python-inference/app/inference/translator.py` | ~177 | Medium |
| `python-inference/app/inference/segmenter.py` | ~168 | Medium |
| `python-inference/app/main.py` | ~89 | Low |

## Phụ lục B — API contract reference

Xem chi tiết tại `README.md` mục **API chính** hoặc `BAO_CAO_DO_AN_DAY_DU.md` Chương 4.

---

*Báo cáo code review này bổ sung cho `BAO_CAO_REVIEW.md` (review cấp hệ thống) với phân tích cấp file/hàm và issue register có thể action.*
