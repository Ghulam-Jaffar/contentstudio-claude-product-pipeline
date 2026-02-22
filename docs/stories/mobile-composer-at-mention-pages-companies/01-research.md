# Research: Mobile Composer @Mention Pages & Companies (iOS + Android)

## Current State

**The web composer already supports @mentions for pages and companies** across Facebook, Twitter/X, LinkedIn, and Bluesky. The backend API and frontend implementation are mature and production-ready.

### Backend (API exists — no changes needed)

The existing `fetchSocialMentionSuggestions` endpoint already supports all platforms and is platform-agnostic (works for any client, not just web):

- **Route:** `POST /fetchSocialMentionSuggestions` — `HelperController@fetchSocialMentionSuggestions`
  - File: `contentstudio-backend/routes/web.php:242-243`
  - Controller: `contentstudio-backend/app/Http/Controllers/Planner/HelperController.php:947-1015`
- **Payload:** `{ "platforms": ["facebook", "twitter", "linkedin", "bluesky"], "keyword": "search text", "workspace_id": "..." }`
- **Response:** `{ "suggestions": { "facebook": [...], "twitter": [...], "linkedin": [...], "bluesky": [...] } }`
- **Platform-specific search methods:**
  - Facebook: `FacebookPlatform::searchPages($payload)` — returns `name`, `username`, `id`, `verification_status`, `fan_count`, `picture`
  - Twitter/X: `TwitterPlatform::searchUser($keyword)` — returns `name`, `screen_name`, `followers_count`, `verified`, `profile_image_url`
  - LinkedIn: `(new LinkedinBuilder([]))->searchOrganizations($payload)` — returns `name`, `id`, `is_verified`, `picture`
  - Bluesky: `(new BlueskyConnector())->searchProfiles($account, $payload)` — returns `display_name`, `handle`, `did`, `avatar`

Additional endpoints (used in older flows, not needed for mobile):
- `POST /fetchSharingMentionSuggestions` — single-platform variant
- `POST /fetchFacebookMentionSuggestions` — Facebook-specific with direct access token

### Frontend (Web Composer — reference implementation)

- **Composable:** `contentstudio-frontend/src/composables/useComposerHelpers.js:56-78` — `fetchComposerMentionSuggestions(text, platforms, cancelToken)` wraps the API call
- **EditorBox integration:** `contentstudio-frontend/src/modules/composer_v2/components/EditorBox/EditorBox.vue:2195-2293` — detects `@` in editor, debounces, calls API, maps platform results into unified suggestion format `{ name, username, id, tab, verification_status, fan_count, picture, selectable_field }`
- **LiteEditorBox:** `contentstudio-frontend/src/modules/composer_v2/components/EditorBox/LiteEditorBox.vue:317-570` — same mention flow for the lite editor variant
- **Mention utilities:** `contentstudio-frontend/src/utils/mentions.js` — `highlightMentions()`, `extractMentions()`, `getMentionedUserIds()` for processing mentions in text
- **ProfileTaggingModal:** `contentstudio-frontend/src/modules/composer_v2/components/ProfileTaggingModal.vue` — separate Instagram image/video profile tagging (not related to inline @mentions)
- **API URL config:** `contentstudio-frontend/src/modules/publish/config/api-utils.js:54-58`
- **Mention dropdown component:** `contentstudio-frontend/src/modules/publish/components/posting/social/options/Mention.vue` — renders the suggestion dropdown with Facebook/Twitter tabs, profile pics, verification badges, follower counts

### iOS App (No mention support currently)

- **Composer:** `contentstudio-ios-v2/ContentStudio/Controllers/Nav Menu VCs/Composer/ComposerViewController.swift`
  - Uses `UITextView` (`txtViewPost`) for post text input
  - Has `TextWatcher`-like handling for character counting and URL detection
  - Has hashtag support via `HashTagViewProtocol` / `HashTagView`
  - **No @mention detection, no mention API calls, no mention suggestion UI**
- **Models:** `contentstudio-ios-v2/ContentStudio/Modals/Composer/` — has models for Labels, Hashtags, UTM, Link Preview, GMB — no mention model
- **Service layer:** `contentstudio-ios-v2/ContentStudio/HelperClasses/ServiceManager.swift` — manages API calls; no mention-related endpoints configured

### Android App (No mention support currently)

- **Composer:** `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/ComposerActivity/ComposerActivity.java`
  - Uses `EditText` for post text input with `TextWatcher` for character counting
  - Written in Java (not Kotlin — the codebase is primarily Java despite .kt search)
  - Has hashtag support, UTM, labels, link preview
  - **No @mention detection, no mention API calls, no mention suggestion UI**
- **Layouts:** `contentstudio-android-v2/app/src/main/res/layout/activity_composer_new.xml`, `new_composer_view.xml`
- **Strings:** `contentstudio-android-v2/app/src/main/res/values/composer_strings.xml`

## What Needs to Change

### No backend changes needed
The `fetchSocialMentionSuggestions` API is already platform-agnostic and returns all the data the mobile apps need. The mobile apps just need to call it.

### iOS App
- Add `@` character detection in `ComposerViewController`'s text view delegate (likely `textViewDidChange`)
- Create a mention suggestion model (similar to existing `HashTagResponse`)
- Create a mention suggestions dropdown/overlay UI (list with profile pic, name, platform badge)
- Call `POST /fetchSocialMentionSuggestions` via `ServiceManager` with currently selected platforms
- Insert selected mention into the text view with proper formatting
- Cancel in-flight requests when user continues typing (debounce)

### Android App
- Add `@` character detection in `ComposerActivity`'s `TextWatcher.afterTextChanged`
- Create a mention suggestion adapter + RecyclerView/popup
- Call `POST /fetchSocialMentionSuggestions` via the API client
- Insert selected mention into the EditText with proper formatting
- Cancel in-flight requests on new keystrokes (debounce)

## Mobile Context

- **Both apps have NO existing @mention functionality** — this is entirely new
- Both apps already support hashtag suggestion flows, which can serve as a UX/architecture reference
- The API is ready — no backend work required
- Selected social accounts are known in the composer context on both platforms, so the `platforms` array can be derived from selected accounts
- Mention behavior should match web: typing `@` followed by 2+ characters triggers the search, selecting a suggestion inserts the mention

## Files Involved

### iOS
- `contentstudio-ios-v2/ContentStudio/Controllers/Nav Menu VCs/Composer/ComposerViewController.swift` — add @ detection + trigger suggestions
- New: Mention suggestion model (in `Modals/Composer/`)
- New: Mention suggestion dropdown UI
- `contentstudio-ios-v2/ContentStudio/HelperClasses/ServiceManager.swift` — add mention search API call

### Android
- `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/ComposerActivity/ComposerActivity.java` — add @ detection + trigger suggestions
- New: Mention suggestion adapter + layout
- New: Mention suggestion popup/overlay
- `contentstudio-android-v2/app/src/main/res/layout/activity_composer_new.xml` or `new_composer_view.xml` — add mention suggestion container
- `contentstudio-android-v2/app/src/main/res/values/composer_strings.xml` — add mention-related strings
