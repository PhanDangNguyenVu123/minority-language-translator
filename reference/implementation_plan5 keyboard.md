# Virtual Keyboard Integration for Ethnic Languages

Add a virtual keyboard to the translation app to support typing in Bana, Ede, and Khmer, which require special characters not easily accessible on standard keyboards.

## User Review Required

> [!IMPORTANT]
> The backend currently does not support Khmer translation. Adding Khmer to the frontend will provide the keyboard support, but the translation results will be placeholders until the backend models are updated.

## Proposed Changes

### Frontend Component

#### [NEW] [VirtualKeyboard.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/VirtualKeyboard.tsx)
- Create a reusable virtual keyboard component using `react-simple-keyboard`.
- Define custom layouts:
    - **Bana/Ede**: Latin layout with an additional row for special characters like `ƀ, č, ñ, ł, ă, â, ê, ô, ơ, ư, ơ̆, ư̆`.
    - **Khmer**: A standard Khmer script layout.
- Support switching between layouts based on the selected source language.
- Handle input synchronization with the main text area.

#### [MODIFY] [App.tsx](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/App.tsx)
- Add Khmer (`km`) to `LangCode` and `LANG_META`.
- Add state for `showKeyboard`.
- Integrate `VirtualKeyboard` component.
- Add a toggle button (keyboard icon) in the input pane.
- Update `handleInputChange` to work with the virtual keyboard's output.

#### [MODIFY] [App.css](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/App.css)
- Style the virtual keyboard for a premium "Glassmorphism" look that matches the app's theme.
- Ensure responsive design (it should look good on mobile too).
- Add styles for the keyboard toggle button.

#### [NEW] [khmer.png](file:///e:/devpro/JavaSpringBoot/translate-app/frontend/src/assets/khmer.png)
- Generate a Khmer flag icon for the language selector.

## Verification Plan

### Automated Tests
- Manual testing of character input in each language.
- Verify that clicking keys on the virtual keyboard correctly updates the `inputText`.
- Verify that the keyboard layout changes automatically when the source language is changed.

### Manual Verification
- Test on different screen sizes.
- Verify that special characters for Bana and Ede are correctly inserted.
- Verify Khmer script input.
