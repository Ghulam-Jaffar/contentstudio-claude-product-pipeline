# Research: Rename Google My Business → Google Business Profile

## Current State

Google rebranded "Google My Business" to **Google Business Profile** (GBP) in late 2021. ContentStudio still uses the old name across all products. Internal code identifiers (`gmb`, `google_my_business`, `gmb_options`, etc.) are technically correct as API/platform type keys and **will not change** — only user-facing display strings are in scope.

### Scale by codebase

| Codebase | User-facing label files | Notes |
|---|---|---|
| Frontend (Vue) | ~35 locale string entries + ~7 component label files | i18n in 7 languages |
| Backend (PHP) | ~10 user-facing validation error messages | Shown to users via API responses |
| iOS (Swift) | ~3 hardcoded user-facing strings | Button titles, alert messages |
| Android (XML) | ~8 string resources across 10 language files | `strings.xml`, `composer_strings.xml`, `inbox_strings.xml` |
| Chrome Extension | ~10 label occurrences in constants + components | Mirrors frontend patterns |

---

## What Needs to Change

### Rule: Display names only
- `"Google My Business"` → `"Google Business Profile"`
- `"GMB"` → `"GBP"` in user-facing contexts (badges, labels, error messages, tooltips)
- Account connection area tooltip: `"Google Business Profile (GBP), formerly known as Google My Business (GMB)"`

### What does NOT change
- Internal identifiers: `gmb`, `google_my_business`, `gmb_options`, `isGMBReqMet`, etc.
- File names: `GmbOptions.vue`, `gmb.js`, `GmbHelper.php`, `GoogleMyBusinessRepo.php`, etc.
- Database values, API field names, route names, locale file keys

---

## Files Involved

### Frontend — `contentstudio-frontend/src/`

**Locale files (English — primary, others auto-propagate):**
- `locales/en/common.json` — `"gmb": "GMB"` → `"GBP"`, message strings
- `locales/en/composer.json` — GMB Options error messages, `"google_my_business": "Google My Business"` → `"Google Business Profile"`
- `locales/en/inbox.json` — `"gmb": "GMB"`, `"google_my_business": "GMB"` → `"GBP"`
- `locales/en/integration.json` — `"gmb": "Choose Google My Business Locations"` → `"Choose Google Business Profile Locations"`
- `locales/en/messages.json` — GMB posting error messages
- `locales/en/planner.json` — GMB references in API limitation warnings, analytics unavailable message
- `locales/en/settings.json` — Multiple `"Google My Business"` label strings + file size/dimension errors
- `locales/en/publish.json` — `"gmb_post": "GMB Post (720×720)"` → `"GBP Post (720×720)"`
- `locales/en/automation.json` — `"GMB:"` in platform list string
- All 6 other language locale files (de, fr, es, it, el, + others) — same keys

**Component label files:**
- `modules/common/lib/integrations.js` — `case 'gmb': return 'Google My Business'` display mapping → `"Google Business Profile"`
- `modules/common/composables/useHelper.js:205` — `return 'Google My Business'` → `"Google Business Profile"`
- `modules/publish/components/posting/social/GmbOptions.vue:231` — `"Select Google My Business Options"` → `"Select Google Business Profile Options"`
- `modules/publisher/ai-content-library/composables/useAIPostGeneration.js:118` — `label: 'Google My Business'` → `"Google Business Profile"`
- `modules/setting/components/workspace/team/platformConfigs.js:75` — `name: 'Google My Business'` → `"Google Business Profile"`
- `composables/usePublishValidation.js:741` — `'Google my business'` → `"Google Business Profile"`
- `modules/integration/components/platforms/social/AccountListing.vue:209` — `'Google My Business'` → `"Google Business Profile"`
- `components/addons/InboxAddOnModal.vue:5` — description text containing "Google My Business"

**Account connection tooltip (new):**
- `modules/integration/components/platforms/social/AccountListing.vue` — add tooltip: `"Google Business Profile (GBP), formerly known as Google My Business (GMB)"`

### Backend — `contentstudio-backend/app/`

- `Http/Requests/Api/V1/PostStoreRequest.php:740,746` — `"Google My Business allows only 1 image..."` / `"Google My Business does not support..."` → `"Google Business Profile ..."`
- `Rules/SocialMedia/GmbOptionsRule.php:81,90–95,117` — user-facing validation messages like `"The GMB options contain invalid values."` → `"GBP"`
- `Libraries/Inbox/HelperClasses/GmbHelper.php:94` — `"Please re-connect your GMB account"` → `"GBP account"`
- `Jobs/PlanRemovePostingJob.php:104` & `UpdatePlanPostingJob.php:113` — `$platform_type = 'GMB'` (used in user-visible output) → `'GBP'`

### iOS — `contentstudio-ios-v2/ContentStudio/`

- `Controllers/Nav Menu VCs/Composer/ComposerViewController.swift:1439` — alert text `"Please Set The GMB Settings Before Posting."` → `"Please Set The GBP Settings Before Posting."`
- `Controllers/Nav Menu VCs/Composer/ComposerViewController.swift:2180` — `gmbSettingsButton.setTitle("GMB Settings", ...)` → `"GBP Settings"`

### Android — `contentstudio-android-v2/app/src/main/res/`

- `values/strings.xml` — `"Set GMB Settings"` → `"Set GBP Settings"`, `"GMB Settings"` × 2 → `"GBP Settings"`
- `values/composer_strings.xml` — `"GMB Options"` → `"GBP Options"` in error messages, section header
- `values/inbox_strings.xml` — GMB references
- All language variants (de, fr, es, it, el, pt, nl, etc.) — same string keys

### Chrome Extension — `contentstudio-chrome-extension-v2/src/`

- `services/constants.js` — GMB platform display name
- `services/common-attributes.js` — GMB label constant
- `tab/components/share/sections/GmbOptions.vue` — "Select Google My Business Options" header
- `tab/components/common/IndividualChannelTooltip.vue` — platform name display
- `tab/components/common/IndividualChannelDropdown.vue` — platform label
- `store/modules/social/gmb.js` — any user-facing label strings (state display values only)

---

## Story Split → Epic Required (5 stories)

1. `[FE] Rename Google My Business to Google Business Profile across web app`
2. `[BE] Update Google My Business to Google Business Profile in user-facing error messages`
3. `[iOS] Rename Google My Business to Google Business Profile in iOS app`
4. `[Android] Rename Google My Business to Google Business Profile in Android app`
5. `[Chrome Extension] Rename Google My Business to Google Business Profile in Chrome extension`

→ 5 stories → requires a dedicated epic.
