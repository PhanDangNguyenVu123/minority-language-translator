# Virtual Keyboard Integration Walkthrough

I have successfully integrated a virtual keyboard into the Minority Language Translator app. This feature allows users to type special characters for Bana, Ê-đê, and Khmer script directly within the application.

## Key Changes

### 1. New Virtual Keyboard Component
-   **File**: [VirtualKeyboard.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/VirtualKeyboard.tsx)
-   **Functionality**: 
    -   Uses `react-simple-keyboard` with custom layouts.
    -   **Bana/Ê-đê**: Includes a dedicated row for special Latin characters: `ƀ, č, ñ, ł, ă, â, ê, ô, ơ, ư, ơ̆, ư̆`.
    -   **Khmer**: Implements a standard Khmer script layout (NiDA-based).
    -   **Sync**: Real-time synchronization with the main text input area.

### 2. Language Support Expansion
-   **Khmer (KM)**: Added as a new supported language in the frontend UI.
-   **Flag**: Generated a high-quality Khmer flag icon [khmer.png](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/assets/khmer.png).

### 3. UI Integration in App.tsx
-   Added a keyboard toggle button next to the character count in the input pane.
-   The keyboard automatically switches layouts based on the selected source language.
-   Integrated the component seamlessly into the existing "Glassmorphism" design.

### 4. Premium Styling
-   Custom CSS in [App.css](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/App.css) for a sleek, translucent look.
-   Smooth slide-down animations and hover effects for a premium feel.

## Verification
-   [x] **Bana/Ê-đê Layout**: Verified that special characters are correctly inserted.
-   [x] **Khmer Layout**: Verified that Khmer script is typed correctly.
-   [x] **Input Synchronization**: Verified that typing on either the physical or virtual keyboard keeps the state consistent.
-   [x] **Responsive Design**: Keyboard container adjusts to fit the container width.

> [!NOTE]
> While the keyboard support for Khmer is now active, remember that the backend currently uses a placeholder for Khmer translation until the specific model is integrated.
