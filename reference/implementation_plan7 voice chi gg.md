# Kế Hoạch Triển Khai Tính Năng TTS Bằng gTTS (Cách 2)

Chúng ta sẽ thực hiện chuyển từ Web Speech API sang dùng API của Google Dịch (thông qua thư viện `gTTS` trên Backend Python). Hệ thống sẽ chạy theo luồng: React -> Java Backend -> Python Backend (gTTS tạo file MP3) -> Trả về chuỗi Base64 cho React phát.

## Đề Xuất Thay Đổi (Proposed Changes)

### 1. Python Backend (`python-inference`)
- **[MODIFY] `requirements.txt`**: Thêm thư viện `gTTS`.
- **[MODIFY] `app/models/schemas.py`**: Thêm class `TtsRequest` (text, lang) và `TtsResponse` (audioBase64).
- **[MODIFY] `app/main.py`**: Tạo endpoint `POST /internal/tts`. Hàm này sẽ gọi thư viện `gTTS`, xuất âm thanh dưới dạng `BytesIO`, mã hóa thành Base64 và trả về JSON.

### 2. Java Backend (`backend-java`)
- **[NEW] `TtsRequest.java`** & **`TtsResponse.java`**: Tạo 2 file DTO mới trong package `com.example.translate.dto` để nhận yêu cầu và phản hồi.
- **[MODIFY] `TranslateService.java`**: Thêm hàm `getTts(TtsRequest req)` để đẩy request từ Java sang Python (`/internal/tts`).
- **[MODIFY] `TranslateController.java`**: Mở thêm endpoint `POST /api/tts` để Frontend gọi vào.

### 3. Frontend React (`frontend/src/App.tsx`)
- **[MODIFY] `App.tsx`**: Sửa lại hàm `handleSpeak`. Thay vì dùng `window.speechSynthesis.speak()`, chúng ta sẽ `fetch('/api/tts')`. 
- Khi nhận được phản hồi chứa `audioBase64`, React sẽ biến thành chuỗi `data:audio/mp3;base64,...` và dùng đối tượng `new Audio()` để phát âm thanh.
- Giữ nguyên giao diện cái loa và hiệu ứng xanh lúc đang phát âm.

## User Review Required

> [!WARNING]
> Lưu ý nhỏ: Gọi qua API của Google Dịch (gTTS) sẽ có một chút độ trễ (khoảng 0.5s - 1s) so với lúc gọi trực tiếp trên trình duyệt, vì hệ thống phải kết nối mạng, tải file mp3 về rồi mới phát.

Bạn hãy xác nhận nếu đồng ý với kế hoạch này để tôi bắt đầu viết code cho toàn bộ luồng từ Backend đến Frontend nhé!
