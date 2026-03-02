# Research: X (Twitter) API Mention Restriction Handling

## Current State

X (Twitter) API v2 has removed the ability for self-serve tier apps to programmatically `@mention` users in posts. Any post sent via the API that contains `@username` text may be rejected or the mention will not tag the user. ContentStudio uses `POST /2/tweets` (v2 API) for all Twitter publishing.

### How mentions work today (web composer)

- **Mention detection:** `src/utils/mentions.js` and `src/composables/useMentions.js` use `/@[A-Za-z]*/g` regex to match `@mentions` in text.
- **Internal mention format:** When a user tags someone in the composer, it's stored as `{(@username)[@display_name]}` in the message string.
- **Backend parsing:** `app/Libraries/Helper.php::parsePostMentions()` (line 623) resolves the format per platform. For Twitter (line 639-650), it converts `{(@username)[@display]}` → `@username` (where ID contains `@`). This creates a literal `@username` string in the tweet payload.
- **Posting:** `app/Libraries/Integrations/Platforms/Social/TwitterPlatform.php` — `textPost()`, `imagePost()`, `videoPost()` (lines 484–516, 380–482, 293–361) all pass the resolved message as `['text' => $posting_details['message']]` to `$twitter->post('tweets', $post_payload)`.
- **Threaded tweets:** Same `parsePostMentions` runs per thread at line 205.
- **Error handling:** `postingResponse()` (line 580) catches X API error responses and propagates `error_message`. Under the new restriction, X would return an error (likely 403 or similar) for posts containing unsolicited @mentions — causing the post to fail.

### How warnings work (web composer footer)

- `SocialModal.vue::socialPostWarnings()` (line 2635) — computed property that pushes warning objects into an array. Already has patterns for Twitter-specific warnings (e.g., `twitter_no_repeat`, `twitter_long_posts_no_preview`).
- Warnings are combined in `getFooterData()` (line 2873) and passed to `MainComposerFooter.vue` as `footerData.warnings`.
- `MainComposerFooter.vue` renders warnings with a `TriangleAlert` icon (lines 37–60). **No code changes to the footer component itself are needed** — just adding a new warning entry in `socialPostWarnings`.

### Mobile (iOS & Android)

- **iOS:** `ComposerViewController.swift` + `SocialContentCommonTab.swift` — mentions stored as `[String]?` array; Twitter account tab exists. Mention UI was recently implemented.
- **Android:** `ComposerActivity.java` — has `isTwitterSelected` boolean, `MentionModel`, `MentionTagsAdapter` imports — mention UI recently implemented.

---

## What Needs to Change

### Backend
- **`app/Libraries/Helper.php::parsePostMentions()`** — Change the `twitter` case: instead of resolving `{(@username)[@display]}` → `@username`, resolve to just the plain display name (without `@` prefix), e.g. `→ display_name`. This strips the API mention without breaking the post text flow. The post will publish successfully; the user is just not tagged.
- **Threaded tweets:** Same fix applies automatically since threaded tweets go through the same `parsePostMentions` call.

### Frontend (Web)
- **`src/modules/composer_v2/views/SocialModal.vue::socialPostWarnings()`** — Add a new warning block: detect if Twitter is selected AND the message (common or Twitter-specific) contains `@` mentions (using existing regex). Show contextual warning:
  - Twitter-only selected: "X no longer allows user tagging — @mentions won't link to profiles, but your post will still go out."
  - Twitter + other platforms: "Your @mentions will work on [platforms], but X no longer allows user tagging via the API."
- **`src/locales/en/composer.json`** — Add i18n key `composer.warnings.twitter_mention_restriction` (+ 6 other locales).
- No changes to `MainComposerFooter.vue` — existing warning render pipeline handles it.

### iOS
- In the iOS composer, when X (Twitter) account is selected and the user types `@` in the caption: show an inline warning (below composer text area or in a notice area) that X tagging is not supported via API. Same behaviour: post still goes through, just no tagging.

### Android
- In the Android composer (ComposerActivity.java), when `isTwitterSelected == true` and caption contains `@`: show a warning snackbar or inline notice. Post still proceeds, just @mention not resolved as a tag on X.

---

## Files Involved

| File | Change |
|---|---|
| `contentstudio-backend/app/Libraries/Helper.php` | Change `parsePostMentions()` twitter case: resolve to display name (no `@`) |
| `contentstudio-frontend/src/modules/composer_v2/views/SocialModal.vue` | Add Twitter mention warning in `socialPostWarnings()` |
| `contentstudio-frontend/src/locales/en/composer.json` (+ 6 other locales) | Add i18n key for warning copy |
| `contentstudio-ios-v2/ContentStudio/Controllers/Nav Menu VCs/Composer/ComposerViewController.swift` | Add inline warning when Twitter selected + @mention detected |
| `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/ComposerActivity/ComposerActivity.java` | Add warning notice when isTwitterSelected + @mention detected |
