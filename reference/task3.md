# Tasks

## Python Backend
- [x] Add `gTTS` to `requirements.txt`
- [x] Add `TtsRequest` and `TtsResponse` to `app/models/schemas.py`
- [x] Implement `POST /internal/tts` in `app/main.py`

## Java Backend
- [x] Create `TtsRequest.java` in `com.example.translate.dto`
- [x] Create `TtsResponse.java` in `com.example.translate.dto`
- [x] Add `getTts` method to `TranslateService.java`
- [x] Add `POST /api/tts` to `TranslateController.java`

## Frontend
- [x] Update `handleSpeak` in `App.tsx` to fetch and play base64 audio from `/api/tts`
