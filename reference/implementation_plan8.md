# Image Translation Feature Implementation (Tesseract OCR & Text Overlay)

This plan outlines the steps to add an advanced image upload and translation feature using Tesseract OCR. Based on your request, the application will not only extract the text but also render the translated text directly over the original image, with a toggle to switch between the original and translated views.

## User Action Required
> [!IMPORTANT]
> **Install Tesseract OCR**: Since we are using Tesseract, you **must** install the Tesseract software on your Windows machine for this feature to work.
> 1. Download the installer from: [UB-Mannheim Tesseract OCR](https://github.com/UB-Mannheim/tesseract/wiki) (Tải file `.exe` bản mới nhất 64-bit).
> 2. Run the installer. **Important**: During installation, under "Additional language data (download)", make sure to check "Vietnamese" if you want to scan Vietnamese text.
> 3. After installation, make sure the installation path (usually `C:\Program Files\Tesseract-OCR`) is added to your system's `PATH` environment variable. (Hoặc tôi sẽ chỉ định đường dẫn này trong code Python).

## Proposed Architecture for Image Overlay

To achieve the "replace text in image" effect like Google Translate:
1. **Python Inference**: Instead of just returning a single string, the Python backend will use `pytesseract.image_to_data` to detect the bounding boxes (tọa độ x, y, chiều rộng, chiều cao) of every line of text in the image. It will translate each line individually and return a list of text blocks with their coordinates.
2. **React Frontend**: The frontend will display the uploaded image. We will position `div` elements (containing the translated text) absolutely over the image using the coordinates returned by the backend. These `div`s will have a background color to cover the original text.
3. **Toggle Original**: A simple switch "Hiện bản gốc" (Show original) will hide/show these translated text overlays.

## Proposed Changes

### Frontend (React/TypeScript)
- **[MODIFY]** `frontend/src/App.tsx`: 
  - Add an "Image" vs "Text" tab at the top left of the translation area.
  - Create a drag-and-drop / upload zone for the image.
  - Implement the image viewer with absolute positioning for text overlays.
  - Add the "Hiện bản gốc" (Show original) toggle switch.
- **[MODIFY]** `frontend/src/App.css`:
  - Add styling for the image upload area, the toggle switch, and the overlay text blocks (e.g., solid background, centered text).

### Java Backend (Spring Boot)
- **[NEW]** `backend-java/src/main/java/com/example/translate/dto/OcrResponse.java` and `OcrBlock.java`: DTOs to represent the text blocks and coordinates.
- **[MODIFY]** `backend-java/src/main/java/com/example/translate/controller/TranslateController.java`:
  - Add `POST /api/ocr` endpoint that consumes `multipart/form-data`.
- **[MODIFY]** `backend-java/src/main/java/com/example/translate/service/TranslateService.java`:
  - Forward the multipart file to Python's `/internal/ocr` and return the structured JSON containing text blocks.

### Python Inference (FastAPI)
- **[MODIFY]** `python-inference/requirements.txt`:
  - Add `pytesseract` and `Pillow` (and `python-multipart` for handling file uploads in FastAPI).
- **[MODIFY]** `python-inference/app/main.py`:
  - Add `POST /internal/ocr` endpoint.
  - Group Tesseract's `image_to_data` output by line.
  - Translate each line using the existing `translate` function.
  - Return the bounding boxes and translated text.

## Verification Plan
1. Install Tesseract locally and restart the backend.
2. Open the UI, switch to the "Image" tab.
3. Upload an image containing Vietnamese text.
4. Verify that the loading state is shown.
5. Verify that the image is displayed with translated text covering the original text.
6. Toggle "Hiện bản gốc" to ensure the original image is visible underneath.
