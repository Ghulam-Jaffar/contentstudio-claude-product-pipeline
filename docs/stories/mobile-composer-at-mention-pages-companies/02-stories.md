# Stories: Mobile Composer @Mention Pages & Companies

## Epic

**Title:** Mobile Composer: @Mention Pages & Companies (iOS + Android)

**Description:**
Add @mention support to the mobile post composer on both iOS and Android. When a user types "@" followed by 2 or more characters while composing a post, the app should display a searchable suggestions dropdown showing matching pages and profiles from the social platforms the user has selected (Facebook, Twitter/X, LinkedIn, Bluesky). Selecting a suggestion inserts the mention into the post text.

The backend API (`POST /fetchSocialMentionSuggestions`) already exists and powers the web composer's @mention feature — no backend changes are required. This epic covers the native mobile implementation on both platforms to match the web experience.

---

## Story 1: [iOS] Add @mention suggestions to the post composer

### Description:

Implement @mention support in the iOS post composer so users can tag pages and company profiles while creating posts. When the user types `@` followed by 2+ characters, a suggestions overlay should appear below the cursor area showing matching pages/profiles from the user's selected social platforms.

The existing backend API `POST /fetchSocialMentionSuggestions` (called in `contentstudio-backend/app/Http/Controllers/Planner/HelperController.php:947`) already returns mention suggestions for Facebook, Twitter/X, LinkedIn, and Bluesky. The iOS app needs to call this endpoint and display the results.

**Key files to modify:**
- `contentstudio-ios-v2/ContentStudio/Controllers/Nav Menu VCs/Composer/ComposerViewController.swift` — add `@` detection in text view delegate, trigger suggestion search
- `contentstudio-ios-v2/ContentStudio/HelperClasses/ServiceManager.swift` — add `fetchSocialMentionSuggestions` API method

**New files to create:**
- Mention suggestion model (in `Modals/Composer/`) — to parse the API response
- Mention suggestion overlay view — dropdown UI with profile picture, name, platform badge

**Reference implementation (web):** `contentstudio-frontend/src/modules/composer_v2/components/EditorBox/EditorBox.vue:2195-2293` and `contentstudio-frontend/src/composables/useComposerHelpers.js:56-78`

---

### Workflow:

1. User opens the Composer and selects one or more social accounts (e.g., a Facebook page and a LinkedIn company page)
2. User starts typing their post in the text area
3. User types `@` followed by at least 2 characters (e.g., `@Cont`)
4. A suggestions overlay appears below the text area showing matching pages and profiles from the selected platforms
5. Each suggestion shows: a profile picture (or placeholder), the page/profile name, and a small platform icon (Facebook, Twitter/X, LinkedIn, or Bluesky)
6. User scrolls through the suggestions if there are more than fit on screen
7. User taps on a suggestion (e.g., "ContentStudio" from Facebook)
8. The mention is inserted into the post text, replacing the `@Cont` typed text with the full mention (e.g., `@ContentStudio`)
9. The suggestions overlay closes
10. User can type `@` again anywhere in the post to mention another page/profile
11. If no suggestions match, the overlay shows "No results found for '@Cont'" and closes automatically when the user continues typing without `@`

---

### Acceptance criteria:

- [ ] Typing `@` followed by 2 or more characters in the composer text view triggers a mention search
- [ ] Typing `@` followed by only 1 character or `@` alone does not trigger a search
- [ ] The search calls `POST /fetchSocialMentionSuggestions` with the typed keyword and the currently selected platforms
- [ ] Only the platforms of the user's currently selected social accounts are passed in the `platforms` array
- [ ] Suggestions display: profile picture (or gray placeholder), page/profile name, and platform icon
- [ ] Facebook suggestions show the page name as the primary text
- [ ] Twitter/X suggestions show the `@username` as the primary text
- [ ] LinkedIn suggestions show the organization name as the primary text
- [ ] Bluesky suggestions show the `@handle` as the primary text
- [ ] Tapping a suggestion inserts the mention into the text at the cursor position, replacing the typed `@keyword`
- [ ] After inserting a mention, the cursor is placed immediately after the inserted mention text
- [ ] The suggestions overlay closes after a selection is made
- [ ] If the user deletes the `@` character or moves the cursor away, the suggestions overlay closes
- [ ] Typing additional characters after `@` refines the search results (each keystroke triggers a new API call)
- [ ] Previous in-flight API requests are cancelled when a new keystroke occurs (debounce/cancel pattern)
- [ ] If the API returns no results, the overlay shows "No results found"
- [ ] The overlay does not block the user from continuing to type — it appears as a non-modal overlay
- [ ] The mention search works correctly when mentioning at the beginning, middle, or end of the post text
- [ ] Multiple mentions can be inserted in the same post

---

### Mock-ups:

N/A — follow the web composer's @mention dropdown pattern adapted to native iOS conventions. Use a floating overlay/popup anchored below the text area.

---

### Impact on existing data:

None. The mention text is stored as plain text in the post body, same as the web composer. No new data models or schema changes required.

---

### Impact on other products:

- **Web app:** None — the web already has this feature
- **Android app:** Separate story covers Android implementation
- **Chrome extension:** No impact
- **White-label:** No impact — mentions are platform-native behavior

---

### Dependencies:

None — the backend API already exists. This story can be implemented independently.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [Android] Add @mention suggestions to the post composer

### Description:

Implement @mention support in the Android post composer so users can tag pages and company profiles while creating posts. When the user types `@` followed by 2+ characters, a suggestions popup should appear below the text input showing matching pages/profiles from the user's selected social platforms.

The existing backend API `POST /fetchSocialMentionSuggestions` (called in `contentstudio-backend/app/Http/Controllers/Planner/HelperController.php:947`) already returns mention suggestions for Facebook, Twitter/X, LinkedIn, and Bluesky. The Android app needs to call this endpoint and display the results.

**Key files to modify:**
- `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/ComposerActivity/ComposerActivity.java` — add `@` detection in `TextWatcher.afterTextChanged`, trigger suggestion search
- `contentstudio-android-v2/app/src/main/res/values/composer_strings.xml` — add mention-related strings

**New files to create:**
- Mention suggestion model class — to parse the API response
- Mention suggestion adapter (RecyclerView adapter) — for the suggestion list items
- Mention suggestion popup layout XML — item layout with profile picture, name, platform badge
- `contentstudio-android-v2/app/src/main/res/layout/item_mention_suggestion.xml` — suggestion list item layout

**Reference implementation (web):** `contentstudio-frontend/src/modules/composer_v2/components/EditorBox/EditorBox.vue:2195-2293` and `contentstudio-frontend/src/composables/useComposerHelpers.js:56-78`

---

### Workflow:

1. User opens the Composer and selects one or more social accounts (e.g., a Facebook page and a LinkedIn company page)
2. User starts typing their post in the text input field
3. User types `@` followed by at least 2 characters (e.g., `@Cont`)
4. A suggestions popup appears below the text input showing matching pages and profiles from the selected platforms
5. Each suggestion shows: a profile picture (or placeholder), the page/profile name, and a small platform icon (Facebook, Twitter/X, LinkedIn, or Bluesky)
6. User scrolls through the suggestions if there are more than fit on screen
7. User taps on a suggestion (e.g., "ContentStudio" from Facebook)
8. The mention is inserted into the post text, replacing the `@Cont` typed text with the full mention (e.g., `@ContentStudio`)
9. The suggestions popup closes
10. User can type `@` again anywhere in the post to mention another page/profile
11. If no suggestions match, the popup shows "No results found for '@Cont'" and closes automatically when the user continues typing without `@`

---

### Acceptance criteria:

- [ ] Typing `@` followed by 2 or more characters in the composer EditText triggers a mention search
- [ ] Typing `@` followed by only 1 character or `@` alone does not trigger a search
- [ ] The search calls `POST /fetchSocialMentionSuggestions` with the typed keyword and the currently selected platforms
- [ ] Only the platforms of the user's currently selected social accounts are passed in the `platforms` array
- [ ] Suggestions display: profile picture (or gray placeholder), page/profile name, and platform icon
- [ ] Facebook suggestions show the page name as the primary text
- [ ] Twitter/X suggestions show the `@username` as the primary text
- [ ] LinkedIn suggestions show the organization name as the primary text
- [ ] Bluesky suggestions show the `@handle` as the primary text
- [ ] Tapping a suggestion inserts the mention into the text at the cursor position, replacing the typed `@keyword`
- [ ] After inserting a mention, the cursor is placed immediately after the inserted mention text
- [ ] The suggestions popup closes after a selection is made
- [ ] If the user deletes the `@` character or moves the cursor away, the suggestions popup closes
- [ ] Typing additional characters after `@` refines the search results (each keystroke triggers a new API call)
- [ ] Previous in-flight API requests are cancelled when a new keystroke occurs (debounce/cancel pattern)
- [ ] If the API returns no results, the popup shows "No results found"
- [ ] The popup does not block the user from continuing to type — it appears as a non-modal popup
- [ ] The mention search works correctly when mentioning at the beginning, middle, or end of the post text
- [ ] Multiple mentions can be inserted in the same post

---

### Mock-ups:

N/A — follow the web composer's @mention dropdown pattern adapted to native Android conventions. Use a `PopupWindow` or similar anchored below the text input.

---

### Impact on existing data:

None. The mention text is stored as plain text in the post body, same as the web composer. No new data models or schema changes required.

---

### Impact on other products:

- **Web app:** None — the web already has this feature
- **iOS app:** Separate story covers iOS implementation
- **Chrome extension:** No impact
- **White-label:** No impact — mentions are platform-native behavior

---

### Dependencies:

None — the backend API already exists. This story can be implemented independently of the iOS story.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, native mobile app with its own theming
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
