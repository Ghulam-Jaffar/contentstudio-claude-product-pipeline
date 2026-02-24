# GMB → GBP Rename: Stories

## Epic

**Title:** Rename Google My Business to Google Business Profile across all products
**Objective:** Q1-2026
**Sprint:** 09 March – 20 March 2026

**Description:**
Google rebranded "Google My Business" to **Google Business Profile (GBP)** in late 2021. ContentStudio still uses the old name throughout — in labels, tooltips, error messages, alerts, and UI copy across the web app, backend API responses, iOS app, Android app, and Chrome extension.

This epic updates all user-facing display strings to the correct name. Internal code identifiers (`gmb`, `google_my_business`, `gmb_options`, file names, class names, API field names, database values) are **not changed** — only what users see on screen.

---

## Story 1: [FE] Rename Google My Business to Google Business Profile across web app

### Description:

Update all user-facing display strings in the ContentStudio web app that still say "Google My Business" or the abbreviation "GMB" to "Google Business Profile" and "GBP" respectively. Internal code identifiers (variable names, locale key names, API field names) are not changed.

**Scope of changes — display/label strings only:**

**`src/modules/common/lib/integrations.js`**
- `case 'gmb': return 'Google My Business'` → `return 'Google Business Profile'`

**`src/modules/common/composables/useHelper.js` (line ~205)**
- `return 'Google My Business'` → `return 'Google Business Profile'`

**`src/composables/usePublishValidation.js` (line ~741)**
- `platformName === 'Gmb' ? 'Google my business'` → `'Google Business Profile'`

**`src/modules/publish/components/posting/social/GmbOptions.vue` (line ~231)**
- `<p>Select Google My Business Options</p>` → `<p>Select Google Business Profile Options</p>`

**`src/modules/publisher/ai-content-library/composables/useAIPostGeneration.js` (line ~118)**
- `label: 'Google My Business'` → `label: 'Google Business Profile'`

**`src/modules/setting/components/workspace/team/platformConfigs.js` (line ~75)**
- `name: 'Google My Business'` → `name: 'Google Business Profile'`

**`src/modules/integration/components/platforms/social/AccountListing.vue` (line ~209)**
- Display name `'Google My Business'` → `'Google Business Profile'`
- Add a tooltip to the Google Business Profile account entry in the connection list: *"Google Business Profile (GBP), formerly known as Google My Business (GMB)"*

**`src/components/addons/InboxAddOnModal.vue` (line ~5)**
- Description text: replace `Google My Business` → `Google Business Profile`

**Locale files — English (primary; all other 6 languages get same key updates in their files):**

`src/locales/en/common.json`:
- `"gmb": "GMB"` → `"GBP"`
- `"message_box_empty_gmb": "Message box is empty, please add something to post on GMB."` → `"...to post on GBP."`
- `"video_platforms_only": "...GMB..."` → `"...GBP..."`

`src/locales/en/composer.json`:
- `"gmb_button_link_empty": "Button link cannot be empty in GMB Options"` → `"...GBP Options"`
- `"gmb_button_link_invalid": "Button link is not valid in GMB Options"` → `"...GBP Options"`
- `"gmb_title_empty": "Title cannot be empty in GMB Options."` → `"...GBP Options."`
- `"gmb_date_empty": "Date cannot be empty in GMB Options."` → `"...GBP Options."`
- `"google_my_business": "Google My Business"` → `"Google Business Profile"`
- `"message": "Incase of posting video to GMB..."` → `"...GBP..."`

`src/locales/en/inbox.json`:
- `"gmb": "GMB"` → `"GBP"`
- `"google_my_business": "GMB"` → `"GBP"`
- `"The review was deleted on GMB..."` → `"...GBP..."`

`src/locales/en/integration.json`:
- `"gmb": "Choose Google My Business Locations"` → `"Choose Google Business Profile Locations"`

`src/locales/en/messages.json`:
- `"gmb_null_title_error": "Title is required for GMB posting."` → `"...GBP posting."`
- `"gmb_null_start_date_error": "Start date is required for GMB posting."` → `"...GBP posting."`
- `"gmb_null_end_date_error": "End date is required for GMB posting."` → `"...GBP posting."`

`src/locales/en/planner.json`:
- All `GMB` references in `api_limitations`, `warning_note`, `gmb_video` deletion warning strings → `GBP`
- `"unsupported_gmb": "Post analytics for Google My Business are not yet available..."` → `"Google Business Profile"`

`src/locales/en/publish.json`:
- `"gmb_post": "GMB Post (720×720)"` → `"GBP Post (720×720)"`

`src/locales/en/settings.json`:
- `"gmb": "Google My Business"` (tab label) → `"Google Business Profile"`
- `"gmb_locations": "No Google My Business location found."` → `"No Google Business Profile location found."`
- `"gmb": "Select Google My Business locations"` → `"Select Google Business Profile locations"`
- `"gmb": "No Google My Business location found."` (duplicate) → same update
- `"gmb": "GMB"` (manage limits tab) → `"GBP"`
- All `"Min. file size for Google My Business..."` / `"Max. file size for Google My Business..."` / `"Dimension for Google My Business..."` / `"GIF file is not allowed for Google My Business."` / `"Google My Business doesn't allow publishing more than 1 image..."` error strings → replace `"Google My Business"` with `"Google Business Profile"` throughout
- `"Max. file size for GMB video..."` → `"GBP video"`

`src/locales/en/automation.json`:
- `"other_platforms": "X (Twitter), Pinterest, Threads, Bluesky, Tumblr, GMB:"` → `"...GBP:"`

**Apply the same value changes to all other language locale files** (de, fr, es, it, el, and any others) for the same locale keys.

**Account connection tooltip copy:**
- Location: Google Business Profile row in the social accounts connection list (`AccountListing.vue`)
- Tooltip text: `"Google Business Profile (GBP), formerly known as Google My Business (GMB)"`
- Show on hover of the platform name/logo, using the standard `ℹ` icon pattern

---

### Workflow:

1. User opens any area of ContentStudio that references Google Business Profile (Composer account selector, Planner filter, Settings → Social Accounts, Analytics, Inbox filter, etc.)
2. All labels, section headers, error messages, and tooltips previously reading "Google My Business" or "GMB" now read "Google Business Profile" or "GBP"
3. User connects a Google Business Profile account in Settings → Social Accounts → sees the platform name "Google Business Profile" with a tooltip: "Google Business Profile (GBP), formerly known as Google My Business (GMB)"
4. User posts to GBP and sees "GBP" in post status, composer section header, and any validation error messages

---

### Acceptance criteria:

- [ ] All instances of "Google My Business" in the web app UI are replaced with "Google Business Profile"
- [ ] All instances of "GMB" in user-visible labels, badges, error messages, and tooltips are replaced with "GBP"
- [ ] The account connection list shows "Google Business Profile" as the platform name
- [ ] A tooltip `ℹ` is present next to "Google Business Profile" in the account connection area with text: "Google Business Profile (GBP), formerly known as Google My Business (GMB)"
- [ ] Composer section headers show "Google Business Profile Options" (not "Google My Business Options")
- [ ] Validation error messages reference "GBP Options" (e.g., "Button link cannot be empty in GBP Options")
- [ ] Post-size/dimension validation errors reference "Google Business Profile" (e.g., "Min. file size for Google Business Profile is 10KB")
- [ ] Planner filter dropdown shows "GBP" (not "GMB") for this platform's filter label
- [ ] Analytics unavailability message references "Google Business Profile"
- [ ] Settings → Manage Limits tab shows "GBP" label
- [ ] Inbox filter and platform labels show "GBP"
- [ ] Content Categories settings show "Google Business Profile"
- [ ] Automation CSV bulk upload shows "GBP" in platform list
- [ ] All 7 supported UI languages reflect the updated values for the same locale keys
- [ ] No internal code identifiers are changed (locale key names, variable names, API field names remain `gmb`, `google_my_business`, etc.)
- [ ] All color classes use theme-aware values — no hardcoded hex or blue-* classes added

---

### Mock-ups:

- Account connection list with updated platform name and tooltip
- Composer GBP section header
  (Figma link to be added)

---

### Impact on existing data:

No data changes. Locale values and display string changes only. No API payloads, database fields, or user data are affected.

---

### Impact on other products:

- Chrome extension carries its own copy of these labels — covered in **[Chrome Extension] Rename Google My Business to Google Business Profile in Chrome extension**
- iOS and Android carry native string resources — covered in respective stories

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories)
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 2: [BE] Update Google My Business to Google Business Profile in user-facing error messages

### Description:

Several backend validation rules and error messages that are returned to users via API responses still use "Google My Business" or "GMB". This story updates those user-facing strings to "Google Business Profile" and "GBP".

**Scope: user-visible strings only.** Class names, method names, variable names, API field names (`gmb_options`, `google_my_business` platform type identifier), and database values are not changed.

**Files and specific changes:**

**`app/Http/Requests/Api/V1/PostStoreRequest.php` (lines ~740, ~746)**
- `'Google My Business allows only 1 image per post. Multiple images are not supported.'`
  → `'Google Business Profile allows only 1 image per post. Multiple images are not supported.'`
- `'Google My Business does not support mixing images and video. Please use either an image or video.'`
  → `'Google Business Profile does not support mixing images and video. Please use either an image or video.'`

**`app/Rules/SocialMedia/GmbOptionsRule.php` (lines ~81, ~90–95, ~117)**
- `'The GMB options contain invalid values.'` → `'The GBP options contain invalid values.'`
- `'GMB topic_type must be one of: STANDARD, EVENT, OFFER'` → `'GBP topic_type must be one of: STANDARD, EVENT, OFFER'`
- `'GMB start_date must be a valid date'` → `'GBP start_date must be a valid date'`
- `'GMB end_date must be a valid date and after or equal to start_date'` → `'GBP end_date ...'`
- `'GMB redeem_online_url must be a valid URL'` → `'GBP redeem_online_url must be a valid URL'`
- `'GMB cta_link must be a valid URL'` → `'GBP cta_link must be a valid URL'`
- `'GMB action_type must be one of: ...'` → `'GBP action_type must be one of: ...'`
- `'GMB topic_type must be one of: ...'` (duplicate in line ~117) → `'GBP ...'`

**`app/Libraries/Inbox/HelperClasses/GmbHelper.php` (line ~94)**
- `"Please re-connect your GMB account"` → `"Please re-connect your Google Business Profile account"`

**`app/Jobs/PlanRemovePostingJob.php` (line ~104)**
- `$platform_type = 'GMB'` (used in user-visible output/logging) → `'GBP'`

**`app/Jobs/UpdatePlanPostingJob.php` (line ~113)**
- `$platform_type = 'GMB'` → `'GBP'`

---

### Workflow:

1. User attempts to post to Google Business Profile with an invalid media combination (e.g., 2 images)
2. API returns validation error: "Google Business Profile allows only 1 image per post."
3. User sees the correct brand name in the error, not the outdated "Google My Business"

---

### Acceptance criteria:

- [ ] API validation error for multiple GBP images reads "Google Business Profile allows only 1 image per post..."
- [ ] API validation error for mixed media reads "Google Business Profile does not support mixing images and video..."
- [ ] GBP options validation errors (`GmbOptionsRule`) reference "GBP" (e.g., "The GBP options contain invalid values.")
- [ ] Field-level validation messages in `GmbOptionsRule` reference "GBP" instead of "GMB"
- [ ] Inbox re-connect error reads "Please re-connect your Google Business Profile account"
- [ ] No class names, method names, variable names, API field names, or database values are changed
- [ ] All existing validation tests continue to pass

---

### Mock-ups:

N/A — backend only.

---

### Impact on existing data:

No data changes. String literals in PHP files only.

---

### Impact on other products:

Error messages returned by these endpoints are displayed in the web app, Chrome extension, and mobile apps. The FE/mobile/Chrome Extension stories update the locally-stored display strings; this story updates the server-side messages that are returned when server-side validation fails.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, backend only
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used) — N/A, backend only
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 3: [iOS] Rename Google My Business to Google Business Profile in iOS app

### Description:

The ContentStudio iOS app has hardcoded user-facing strings that still reference "GMB" instead of "Google Business Profile" / "GBP". This story updates those display strings.

**Scope: hardcoded UI strings only.** Internal Swift identifiers (`gmbOptions`, `isGMBSettingOnDisplay`, `GMBOptions`, `gmbSelectionView`, `GMBSelectionView`, etc.) and API field names are not changed.

**Files and specific changes:**

**`ContentStudio/Controllers/Nav Menu VCs/Composer/ComposerViewController.swift`**

Line ~1439 — alert title:
- `"Please Set The GMB Settings Before Posting."` → `"Please Set The GBP Settings Before Posting."`

Line ~2180 — button title:
- `gmbSettingsButton.setTitle("GMB Settings", for: .normal)` → `gmbSettingsButton.setTitle("GBP Settings", for: .normal)`

---

### Workflow:

1. User selects a Google Business Profile account in the iOS Composer
2. The settings button label reads "GBP Settings" (not "GMB Settings")
3. If user tries to post without completing GBP settings, the alert reads "Please Set The GBP Settings Before Posting."

---

### Acceptance criteria:

- [ ] The "GBP Settings" button label is shown in the Composer when a Google Business Profile account is selected (was "GMB Settings")
- [ ] The pre-posting alert reads "Please Set The GBP Settings Before Posting." (was "GMB Settings")
- [ ] No internal Swift identifiers are changed (variable names, class names, model property names)
- [ ] No API field names or network request structures are changed
- [ ] Existing Composer flow for Google Business Profile accounts works as before

---

### Mock-ups:

- Composer screen with "GBP Settings" button
- Pre-posting alert with updated text
  (Figma link to be added)

---

### Impact on existing data:

No data changes. UI string constants only.

---

### Impact on other products:

None. iOS-only change.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native iOS
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 4: [Android] Rename Google My Business to Google Business Profile in Android app

### Description:

The ContentStudio Android app has string resources referencing "GMB" / "Google My Business" across multiple layout and string XML files, including localized language variants. This story updates all user-facing Android string values.

**Scope: Android string resource values only (`res/values/*.xml`).** String resource keys (e.g., `name="set_gmb_settings"`), Kotlin identifiers, layout file names, and API field names are not changed — only the string values that users see.

**Files and specific changes:**

**`app/src/main/res/values/strings.xml`**
- `<string name="set_gmb_settings">Set GMB Settings</string>` → `Set GBP Settings`
- `<string name="gmb_settings">GMB Settings</string>` → `GBP Settings`
- `<string name="gmb_setting">GMB Settings</string>` → `GBP Settings`

**`app/src/main/res/values/composer_strings.xml`**
- `"Button link cannot be empty in GMB Options."` → `"...GBP Options."`
- `"Button link is not valid in GMB Options"` → `"...GBP Options"`
- `"Title cannot be empty in GMB Options."` → `"...GBP Options."`
- Any other "GMB" label in this file → "GBP"
- Section comment header `<!-- GMB Button Labels -->` / `<!-- GMB Settings -->` — update section display text if surfaced in UI

**`app/src/main/res/values/inbox_strings.xml`**
- Any "GMB" user-facing string values → "GBP"

**All language variant folders** (`values-de/`, `values-fr/`, `values-es/`, `values-it/`, `values-el/`, `values-pt/`, `values-nl/`, and any others):
- Apply the same value substitutions for the same string resource keys in each language file

---

### Workflow:

1. User selects a Google Business Profile account in the Android Composer
2. The "GBP Settings" button is shown (was "GMB Settings")
3. Validation errors and section headers reference "GBP Options" (not "GMB Options")
4. Inbox platform filter labels show "GBP" for Google Business Profile accounts

---

### Acceptance criteria:

- [ ] "Set GBP Settings" button/label shown in Composer when GBP account is selected (was "Set GMB Settings")
- [ ] "GBP Settings" section header visible in Composer GBP settings view (was "GMB Settings")
- [ ] Validation error messages reference "GBP Options" (e.g., "Button link cannot be empty in GBP Options.")
- [ ] Inbox filter/label shows "GBP" for Google Business Profile accounts
- [ ] All translated language variants (`values-de/`, `values-fr/`, `values-es/`, etc.) updated for the same string keys
- [ ] No string resource key names are changed (only values)
- [ ] No Kotlin identifiers, layout file names, or API field names are changed
- [ ] Build compiles without errors after changes

---

### Mock-ups:

- Android Composer with "GBP Settings" label
  (Figma link to be added)

---

### Impact on existing data:

No data changes. Android string resource values only.

---

### Impact on other products:

None. Android-only change.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native Android
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 5: [Chrome Extension] Rename Google My Business to Google Business Profile in Chrome extension

### Description:

The ContentStudio Chrome extension carries its own copy of platform display strings and labels, separate from the web app. All user-facing references to "Google My Business" and "GMB" need to be updated to "Google Business Profile" and "GBP".

**Scope: display string values only.** Internal identifiers (`gmb`, `google_my_business`, store file names like `gmb.js`, component file names like `GmbOptions.vue`) are not changed.

**Files and specific changes:**

**`src/services/constants.js`**
- Platform display name for GMB: `'Google My Business'` → `'Google Business Profile'`

**`src/services/common-attributes.js`**
- Any display label value for GMB platform: `'Google My Business'` / `'GMB'` → `'Google Business Profile'` / `'GBP'`

**`src/tab/components/share/sections/GmbOptions.vue`**
- Section header: `"Select Google My Business Options"` → `"Select Google Business Profile Options"`

**`src/tab/components/common/IndividualChannelTooltip.vue`**
- Platform name display for GMB: `'Google My Business'` → `'Google Business Profile'`

**`src/tab/components/common/IndividualChannelDropdown.vue`**
- Platform label for GMB: `'Google My Business'` / `'GMB'` → `'Google Business Profile'` / `'GBP'`

**`src/tab/components/share/CommonBox.vue`** and **`IndividualBox.vue`**
- Any user-facing label referencing "Google My Business" or "GMB" → update to "Google Business Profile" / "GBP"

**`src/tab/App.vue`**
- Any display string referencing "Google My Business" or "GMB" → update

**`src/tab/components/social/ListAccounts.vue`** and **`SocialAccounts.vue`**
- Platform display name for GMB accounts → `'Google Business Profile'`

**`src/store/modules/social/gmb.js`**
- Any state properties used as display labels (e.g., `platformName: 'Google My Business'`) → `'Google Business Profile'`

**Account connection tooltip (mirror the web app):**
- In the account selector/dropdown where GBP accounts are shown, add or update tooltip: `"Google Business Profile (GBP), formerly known as Google My Business (GMB)"`

---

### Workflow:

1. User opens the ContentStudio Chrome extension
2. In the account selector, Google Business Profile accounts are labeled "Google Business Profile" (not "Google My Business")
3. GBP-specific options section header reads "Select Google Business Profile Options"
4. Any validation error messages reference "GBP Options"

---

### Acceptance criteria:

- [ ] Platform display name in account selector shows "Google Business Profile" (not "Google My Business")
- [ ] GBP options section header reads "Select Google Business Profile Options"
- [ ] Channel tooltip and dropdown label show "Google Business Profile" / "GBP"
- [ ] Tooltip on GBP account entry reads: "Google Business Profile (GBP), formerly known as Google My Business (GMB)"
- [ ] No internal identifiers are changed (file names, store keys, component names, API field names)
- [ ] Extension builds and loads without errors after changes

---

### Mock-ups:

- Chrome extension account selector showing "Google Business Profile"
- GBP options section with updated header
  (Figma link to be added)

---

### Impact on existing data:

No data changes. Display string values only.

---

### Impact on other products:

None. Chrome extension-only change.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, Chrome extension
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
