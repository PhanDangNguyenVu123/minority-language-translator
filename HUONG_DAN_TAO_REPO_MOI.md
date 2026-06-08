# Hướng dẫn tạo Git repository mới

Dự án hiện đang trỏ remote cũ: `https://github.com/quangtuane2/translate-app.git`

Chọn **một** trong hai cách dưới đây.

---

## Cách 1 — Repo mới trên GitHub, giữ lịch sử commit (khuyến nghị)

Phù hợp khi bạn muốn tách sang tài khoản/repo mới nhưng vẫn giữ commit cũ.

### Bước 1: Tạo repo trống trên GitHub

1. Vào https://github.com/new
2. **Repository name:** ví dụ `minority-language-translator` ho hoặc `translate-app`
3. **Public** hoặc **Private** tuỳ ý
4. **Không** tick "Add a README" / ".gitignore" / "license" (repo phải trống)
5. Bấm **Create repository**

### Bước 2: Đổi remote và push

```powershell
cd d:\E\DAIHOC\hk6\DACN1\translate-app

# Xem remote hiện tại
git remote -v

# Gỡ remote cũ
git remote remove origin

# Thêm remote mới (thay USERNAME và TEN-REPO)
git remote add origin https://github.com/USERNAME/TEN-REPO.git

# Push toàn bộ lịch sử
git push -u origin main
```

Nếu repo mới đã có commit (README mặc định), dùng:

```powershell
git pull origin main --allow-unrelated-histories
# Giải quyết conflict nếu có, rồi:
git push -u origin main
```

---

## Cách 2 — Bắt đầu lại từ đầu (không giữ lịch sử)

Phù hợp khi muốn repo “sạch”, một commit initial duy nhất.

```powershell
cd d:\E\DAIHOC\hk6\DACN1\translate-app

# Xóa git cũ (chỉ metadata, không xóa code)
Remove-Item -Recurse -Force .git

# Khởi tạo repo mới
git init
git branch -M main

# Stage và commit
git add .
git status   # kiểm tra không có .zip, .pdf, models/, node_modules/, .venv/
git commit -m "Initial commit: minority language translation system"

# Sau khi tạo repo trống trên GitHub:
git remote add origin https://github.com/USERNAME/TEN-REPO.git
git push -u origin main
```

---

## Trước khi push — checklist

- [ ] Model AI **không** commit (đã ignore: `python-inference/models/*/`)
- [ ] File `.zip`, `.pdf` **không** commit (đã ignore)
- [ ] `node_modules/`, `.venv/`, `target/` **không** commit
- [ ] JWT secret trong `application.properties` — cân nhắc đổi trước khi public repo
- [ ] Notebook `train-model/*.ipynb` khá nặng — vẫn push được nhưng clone sẽ chậm

Tải model sau khi clone: link Google Drive trong `README.md`.

---

## Commit các thay đổi mới (README, báo cáo, v.v.)

Sau khi đã cấu hình remote mới:

```powershell
git add README.md python-inference/requirements.txt
git add BAO_CAO_DO_AN_DAY_DU.md BAO_CAO_REVIEW.md BAO_VE_OUTLINE.md
git add CODE_REVIEW_CHUYEN_SAU.md train-model/DANH_GIA_THEO_QUY_TRINH_ML.md
git add reference/ HUONG_DAN_TAO_REPO_MOI.md .gitignore

git commit -m "docs: update README, reports, and gitignore for new repository"
git push
```

---

## Xử lý lỗi thường gặp

| Lỗi | Cách xử lý |
|-----|------------|
| `remote origin already exists` | `git remote remove origin` rồi add lại |
| `Permission denied` | Đăng nhập GitHub: `git credential` hoặc dùng Personal Access Token |
| `Large files` bị reject | Kiểm tra `.gitignore`, không add file `.zip` / model |
| Push bị reject (non-fast-forward) | Repo GitHub có sẵn commit → dùng `--allow-unrelated-histories` hoặc repo trống |

---

## SSH (tuỳ chọn)

Thay HTTPS bằng SSH:

```powershell
git remote set-url origin git@github.com:USERNAME/TEN-REPO.git
```
