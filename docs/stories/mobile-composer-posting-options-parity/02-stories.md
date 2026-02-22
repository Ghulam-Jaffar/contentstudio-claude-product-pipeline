# Stories: Mobile Composer Posting Options Parity

---

## Story 1: [iOS] Add TikTok posting options to iOS composer

### Description:

The iOS composer supports posting to TikTok (platform selection, video validation) but has no TikTok-specific options screen. `PostSettingVC.swift` — the view controller that should house per-platform settings — is a near-empty stub with no implemented controls.

This story adds a fully functional TikTok options panel to the iOS composer, matching the web composer's `TiktokOptions.vue` feature set. When a user selects a TikTok account and opens the post settings, they should see all TikTok-specific fields: publishing method, post type, carousel title, privacy level, engagement toggles, auto-add music, branded content disclosure, and the AI-generated content flag.

**Files to update:**
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/View/AccountList/PostSettingVC.swift` — implement TikTok options panel
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/ViewModel/ComposerViewModel.swift` — store and pass `tiktok_options` in the API payload
- New model/struct: `TikTokOptions.swift` (or extend existing composer model) with all fields:
  - `publishing_method`: `"direct"` / `"notification"` (default: `"direct"`)
  - `post_type`: `"video"` / `"carousel"` (default: `"video"`)
  - `title`: String, max 90 chars (carousel only)
  - `privacy_level`: `"PUBLIC_TO_EVERYONE"` / `"MUTUAL_FOLLOW_FRIENDS"` / `"SELF_ONLY"`
  - `disable_comment`: Bool
  - `disable_duet`: Bool (video only)
  - `disable_stitch`: Bool (video only)
  - `auto_add_music`: Bool (carousel only)
  - `disclose_commercial_content`: Bool
  - `brand_organic_toggle`: Bool (sub-option under branded content)
  - `brand_content_toggle`: Bool (sub-option under branded content)
  - `is_aigc`: Bool

**UI copy and field specs:**

**Section: Publishing Method**
- Label: "Publishing Method"
- Option 1: "Direct" — subtext: "Post directly to TikTok. All options below are available."
- Option 2: "Notification" — subtext: "ContentStudio sends a notification to your phone. You complete the post in the TikTok app."
- When "Notification" is selected: hide privacy, comments, duet, stitch, music, and branded content sections. Show only post type.

**Section: Post Type**
- Label: "Post Type"
- Option 1: "Video" (default)
- Option 2: "Carousel"
- When "Carousel" selected: show carousel title field; hide Duet and Stitch toggles; show Auto Add Music toggle

**Field: Carousel Title** (carousel only)
- Label: "Carousel Title"
- Placeholder: "e.g. My top 5 tips for better content"
- Helper text: "Required for carousel posts. Max 90 characters."
- Validation: required when post type is carousel; show error "Please enter a carousel title" if empty on submit

**Section: Privacy**
- Label: "Who Can Watch This Video"
- Tooltip: "Choose who can see your TikTok post. 'Everyone' means it's public on TikTok. 'Mutual Friends' means only people who follow each other with you. 'Only Me' keeps it private."
- Options:
  - "Everyone" → `PUBLIC_TO_EVERYONE`
  - "Mutual Friends" → `MUTUAL_FOLLOW_FRIENDS`
  - "Only Me" → `SELF_ONLY`

**Section: Comments, Duet, Stitch** (video only)
- Toggle: "Allow Comments" — default ON — tooltip: "Let viewers leave comments on your TikTok post."
- Toggle: "Allow Duet" — default OFF — tooltip: "Allow other TikTok users to create a split-screen video alongside yours."
- Toggle: "Allow Stitch" — default OFF — tooltip: "Allow other TikTok users to clip and include part of your video in their own posts."

**Section: Auto Add Music** (carousel only)
- Toggle: "Auto Add Music" — default OFF — tooltip: "Automatically add background music to your carousel post on TikTok."

**Section: Branded Content**
- Toggle: "Disclose Branded Content" — default OFF
- Tooltip: "Turn this on if this post promotes a product, service, or brand — whether you're being paid or it's your own brand. TikTok requires disclosure for commercial content."
- When toggled ON, show two checkboxes beneath:
  - Checkbox: "Your Brand" — label: "This post promotes my own brand or product." — `brand_organic_toggle`
  - Checkbox: "Third Party Brand" — label: "This post promotes another brand or product I've been paid to feature." — `brand_content_toggle`
- Info note (shown when either checkbox is checked): "By checking this, you confirm this post will be classified as Branded Content on TikTok and may be subject to additional review."

**Section: AI-Generated Content**
- Toggle: "AI-Generated Content" — default OFF
- Tooltip: "Turn this on if your video or images were created or significantly altered using AI tools (e.g. AI-generated visuals, voiceovers, or effects). TikTok requires disclosure of AIGC."
- Maps to `is_aigc`

---

### Workflow:

1. User opens the ContentStudio iOS app and navigates to the Composer.
2. User selects a TikTok account from their connected accounts.
3. User taps the settings / post options button to open post settings.
4. User sees the TikTok options panel with all available fields.
5. User selects "Publishing Method" — Direct or Notification.
   - If Notification: only post type is shown. All other TikTok options are hidden.
   - If Direct: full options are shown.
6. User selects "Post Type" — Video or Carousel.
   - If Carousel: Carousel Title field appears; Duet and Stitch toggles are hidden; Auto Add Music toggle appears.
   - If Video: Carousel Title is hidden; Duet and Stitch toggles are visible; Auto Add Music is hidden.
7. User sets Privacy (who can watch), toggles Comments/Duet/Stitch as needed.
8. (Optional) User enables "Disclose Branded Content" and checks the relevant sub-options.
9. (Optional) User enables "AI-Generated Content" if applicable.
10. User returns to the main composer, completes their caption and media, and taps Post/Schedule.
11. The app submits the full `tiktok_options` object in the API payload to `POST /processSocialShare`.

---

### Acceptance criteria:

- [ ] When a TikTok account is selected, the post settings screen shows a TikTok options section
- [ ] Publishing method defaults to "Direct"; switching to "Notification" hides all options except post type
- [ ] Post type defaults to "Video"; switching to "Carousel" shows carousel title input and hides duet/stitch toggles
- [ ] Carousel title is required when post type is carousel; shows inline error "Please enter a carousel title" if empty on submit
- [ ] Carousel title enforces a 90-character limit with a visible character counter
- [ ] Privacy level shows three options (Everyone, Mutual Friends, Only Me) with correct API value mapping
- [ ] Allow Comments toggle defaults to ON; Allow Duet and Stitch default to OFF
- [ ] Duet and Stitch toggles are hidden when post type is carousel
- [ ] Auto Add Music toggle appears only when post type is carousel
- [ ] "Disclose Branded Content" toggle reveals "Your Brand" and "Third Party Brand" checkboxes when turned ON
- [ ] "AI-Generated Content" toggle maps to `is_aigc` in the payload
- [ ] All selected TikTok options are included in `tiktok_options` in the API payload sent to the backend
- [ ] When Notification publishing method is selected, `tiktok_options` only includes `publishing_method` and `post_type`

---

### Mock-ups:

N/A — match the layout and field order of the web composer's TikTok options panel (`TiktokOptions.vue`). Refer to the web app at Compose → select TikTok account → settings icon.

---

### Impact on existing data:

No changes to existing data. The `tiktok_options` fields are already accepted and validated by the backend (`TiktokOptionsRule.php`). This is purely a mobile UI addition.

---

### Impact on other products:

- Web composer: no impact
- Android: no impact (separate story)
- Chrome extension: no impact

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, this is a native iOS screen
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

## Story 2: [Android] Complete TikTok posting options in Android composer

### Description:

The Android composer (`ComposerSettingsActivity.java`) already has a partial TikTok options panel: privacy level (3 options) and comment/duet/stitch toggles stored in `SettingModel.java`. However, six fields present in the web composer are completely missing:

1. **Publishing Method** — direct vs notification (not in Android at all)
2. **Post Type** — video vs carousel (not in Android at all)
3. **Carousel Title** — required field for carousel posts (not in Android at all)
4. **Auto Add Music** — carousel-only toggle (not in Android at all)
5. **Branded Content Disclosure** — `disclose_commercial_content`, `brand_organic_toggle`, `brand_content_toggle` (not in Android at all)
6. **AI-Generated Content** — `is_aigc` flag (not in Android at all)

This story adds the missing fields to the existing TikTok card in `ComposerSettingsActivity.java`, updates `SettingModel.java` to hold the new values, and ensures all fields are included in the API payload.

**Files to update:**
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/ComposerSettingsActivity.java` — add UI controls to the existing TikTok card
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/BottomSheetDialog/Data/SettingModel.java` — add new fields: `publishingMethod`, `postType`, `carouselTitle`, `autoAddMusic`, `disclosedCommercialContent`, `brandOrganicToggle`, `brandContentToggle`, `isAigc`
- `contentstudio-android-v2/app/src/main/res/layout/activity_compose_settings.xml` — add UI elements to the TikTok card (lines 274–435)

**Field specs and UI copy:**

**Publishing Method** (add above existing privacy dropdown)
- Label: "Publishing Method"
- Radio group:
  - "Direct" — subtext: "Post directly to TikTok. All options are available."
  - "Notification" — subtext: "ContentStudio notifies your phone. You complete the post in TikTok."
- Default: Direct
- When "Notification" selected: hide privacy, comments, duet, stitch, music, branded content sections

**Post Type** (add below publishing method)
- Label: "Post Type"
- Radio group or segmented control:
  - "Video" (default)
  - "Carousel"
- When "Carousel" selected: show carousel title field; show Auto Add Music; hide Duet and Stitch
- When "Video" selected: hide carousel title and Auto Add Music; show Duet and Stitch

**Carousel Title** (conditional, carousel only)
- Label: "Carousel Title"
- Input field — max 90 characters — show character counter
- Helper text: "Required for carousel posts."
- Validation: required if post type is carousel

**Auto Add Music** (conditional, carousel only)
- Toggle: "Auto Add Music" — default OFF
- Tooltip/subtext: "Automatically add background music to your carousel post on TikTok."

**Branded Content Disclosure** (add after stitch toggle)
- Toggle: "Disclose Branded Content" — default OFF
- Subtext: "Enable if this post promotes a product or brand — paid or your own."
- When ON: show two checkboxes:
  - "Your Brand" — `brand_organic_toggle` — label: "This post promotes my own brand or product"
  - "Third Party Brand" — `brand_content_toggle` — label: "This post promotes a brand I've been paid to feature"

**AI-Generated Content** (add at bottom of TikTok section)
- Toggle: "AI-Generated Content" — default OFF
- Subtext: "Enable if your video or visuals were created or altered with AI tools. Required by TikTok."
- Maps to `is_aigc`

---

### Workflow:

1. User opens the ContentStudio Android app and goes to the Composer.
2. User selects a TikTok account.
3. User taps the settings icon to open `ComposerSettingsActivity`.
4. User sees the TikTok tab with the complete options panel.
5. User selects Publishing Method — Direct or Notification.
   - Notification: collapses privacy, toggles, music, and branded content. Only post type remains visible.
6. User selects Post Type — Video or Carousel.
   - Carousel: carousel title input and Auto Add Music appear; Duet and Stitch are hidden.
   - Video: carousel title and music are hidden; Duet and Stitch are shown.
7. User sets privacy, toggles for comments/duet/stitch as needed.
8. (Optional) User enables Branded Content Disclosure and checks relevant sub-options.
9. (Optional) User enables AI-Generated Content.
10. User saves settings and posts/schedules. All new fields are included in `tiktok_options` in the API payload.

---

### Acceptance criteria:

- [ ] Publishing Method (Direct/Notification) appears at the top of the TikTok settings card, defaulting to Direct
- [ ] Selecting Notification hides privacy, comments, duet, stitch, auto-add music, and branded content fields
- [ ] Post Type (Video/Carousel) selector is present, defaulting to Video
- [ ] Selecting Carousel shows the Carousel Title input and Auto Add Music toggle; hides Duet and Stitch
- [ ] Selecting Video shows Duet and Stitch; hides Carousel Title and Auto Add Music
- [ ] Carousel Title enforces 90-character max with a visible counter; is required when post type is carousel
- [ ] "Disclose Branded Content" toggle reveals "Your Brand" and "Third Party Brand" checkboxes when enabled
- [ ] "AI-Generated Content" toggle maps to `is_aigc` in the payload
- [ ] All new fields are stored in `SettingModel` and included in the `tiktok_options` object sent to the API
- [ ] Existing fields (privacy level, comment/duet/stitch toggles) continue to work correctly

---

### Mock-ups:

N/A — match the web composer's TikTok options layout. Refer to web app at Compose → select TikTok → settings. Integrate into the existing tabbed card UI in `ComposerSettingsActivity`.

---

### Impact on existing data:

No backend or data changes. The API already accepts all fields. `SettingModel` is extended with new fields; existing fields are unchanged.

---

### Impact on other products:

- Web composer: no impact
- iOS: no impact (separate story)
- Chrome extension: no impact

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, native Android screen
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

## Story 3: [iOS] Add platform-specific posting options to iOS composer

### Description:

The iOS composer's `PostSettingVC.swift` is a near-empty stub. Beyond TikTok (covered in **[iOS] Add TikTok posting options to iOS composer**), the web composer also has rich per-platform options for Facebook, Instagram, LinkedIn, and YouTube that are entirely absent from iOS.

This story builds out the platform-specific options panel for these four platforms inside `PostSettingVC.swift`, matching the web composer's feature set.

**Files to update:**
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/View/AccountList/PostSettingVC.swift` — implement per-platform sections for Facebook, Instagram, LinkedIn, YouTube
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/ViewModel/ComposerViewModel.swift` — store and pass platform options in the API payload

**Platform specs:**

---

#### Facebook Options

**Post Type**
- Label: "Post Type"
- Options: "Feed Post", "Reel", "Story"
- Default: "Feed Post"
- Subtext per option:
  - "Feed Post" — "Appears in your Page's main feed."
  - "Reel" — "Short-form video. Available for Pages only."
  - "Story" — "Disappears after 24 hours. Available for Pages only."

**Video Title** (shown when post type is Feed Post or Reel and media contains video)
- Label: "Video Title"
- Placeholder: "e.g. Behind the scenes at our studio"
- Helper text: "Required when posting a video. Max 100 characters."
- Validation: required for video posts; show error "Please enter a video title" if empty on submit

**Share to Story** (shown when post type is Feed Post or Reel)
- Toggle: "Also Share to Story"
- Subtext: "Automatically share this post to your Facebook Story at the same time."
- Default: OFF

---

#### Instagram Options

**Post Type**
- Label: "Post Type"
- Options: "Feed Post", "Reel", "Story", "Carousel"
- Default: "Feed Post"
- Descriptions:
  - "Feed Post" — "Single image or video in your profile grid."
  - "Reel" — "Short video shown in Reels tab and Explore."
  - "Story" — "Disappears after 24 hours."
  - "Carousel" — "Up to 10 images or videos in a swipeable format."

**Share to Story** (shown for Feed Post, Reel, Carousel)
- Toggle: "Also Share to Story"
- Subtext: "Post to your Instagram Story at the same time as your feed post."
- Default: OFF

**Publishing Method**
- Label: "Publishing Method"
- Options:
  - "API" — subtext: "ContentStudio posts directly via the Instagram API."
  - "Mobile" — subtext: "ContentStudio sends a notification to your phone. You complete the post in Instagram." (show only if the user has a mobile device linked)
- Default: "API"

---

#### LinkedIn Options

**Post Type**
- Label: "Post Type"
- Options: "Regular Post", "Document Carousel", "Poll"
- Default: "Regular Post"

**Document Carousel** (shown when post type is Document Carousel)
- File upload field — label: "Upload Document" — accepted types: PDF, PPT, PPTX, DOC, DOCX
- Helper text: "Upload a document to display as a scrollable carousel on LinkedIn."
- Document Title field — label: "Document Title" — placeholder: "e.g. Our 2025 Marketing Report" — required

**Poll** (shown when post type is Poll)
- Label: "Poll Question" — placeholder: "e.g. What's your biggest challenge in 2025?" — required
- Up to 4 poll options — label: "Option 1", "Option 2", etc. — minimum 2 required
- Poll Duration — label: "Poll Duration" — options: "1 day", "3 days", "1 week", "2 weeks"

---

#### YouTube Options

**Post Type**
- Label: "Post Type"
- Options: "Video", "Shorts"
- Default: "Video"

**Video Title**
- Label: "Video Title"
- Placeholder: "e.g. How to schedule posts in ContentStudio"
- Helper text: "Required. Max 100 characters."
- Validation: required; show error "Please enter a video title"

**Privacy Status**
- Label: "Privacy"
- Options: "Public", "Private", "Unlisted"
- Descriptions:
  - "Public" — "Anyone can find and watch this video."
  - "Private" — "Only you can see this video."
  - "Unlisted" — "Anyone with the link can watch, but it won't appear in search."
- Default: "Public"

**Category**
- Label: "Category"
- Dropdown with all standard YouTube categories (e.g. Education, Entertainment, Music, etc.)

**Playlist**
- Label: "Playlist"
- Dropdown of playlists linked to the selected YouTube channel; option "None" if no playlist

**License**
- Label: "License"
- Options: "Standard YouTube License", "Creative Commons"

**Toggles**
- "Made for Kids" — default OFF — subtext: "Enable if this content is specifically made for children. This limits comments and features."
- "Notify Subscribers" — default ON — subtext: "Send a notification to your subscribers when this video is published."
- "Embeddable" — default ON — subtext: "Allow this video to be embedded on other websites."

---

### Workflow:

1. User opens the iOS composer and selects one or more social accounts.
2. User taps the post settings button.
3. The settings screen shows a section for each platform the user has selected accounts for.
4. **Facebook section:** User picks post type (Feed/Reel/Story). If video media is attached, a Video Title field appears. For Feed or Reel, a "Also Share to Story" toggle is shown.
5. **Instagram section:** User picks post type (Feed/Reel/Story/Carousel). Share to Story toggle appears for Feed/Reel/Carousel. Publishing Method selector appears (API/Mobile).
6. **LinkedIn section:** User picks Regular Post, Document Carousel, or Poll. If carousel: document upload + title appear. If poll: question + options + duration appear.
7. **YouTube section:** User fills in video title (required), picks privacy, category, playlist, license, and toggles.
8. User saves settings and returns to the composer to finalise caption and media.
9. The app submits all platform options in the API payload.

---

### Acceptance criteria:

- [ ] Facebook section shows Post Type picker (Feed Post / Reel / Story) when a Facebook account is selected
- [ ] Facebook Video Title field appears when post type is Feed Post or Reel and media contains video; is required on submit
- [ ] Facebook "Also Share to Story" toggle appears for Feed Post and Reel post types; is hidden for Story
- [ ] Instagram section shows Post Type picker (Feed Post / Reel / Story / Carousel) when an Instagram account is selected
- [ ] Instagram "Also Share to Story" toggle appears for Feed Post, Reel, and Carousel; hidden for Story
- [ ] Instagram Publishing Method selector appears (API / Mobile); Mobile option only visible if user has a mobile device linked in ContentStudio
- [ ] LinkedIn section shows Post Type picker (Regular Post / Document Carousel / Poll)
- [ ] Selecting Document Carousel shows document upload and Document Title fields; Title is required
- [ ] Selecting Poll shows question field, minimum 2 and maximum 4 option fields, and duration selector
- [ ] YouTube section shows Video Title (required), Privacy, Category, Playlist, License, and the three toggles
- [ ] YouTube Video Title shows error "Please enter a video title" if left empty on submit
- [ ] All platform option values are included in the correct platform-specific fields in the API payload (e.g. `facebook_post_type`, `instagram_post_type`, `youtube_options`)
- [ ] Platform sections are only shown for platforms that have a connected account selected in the composer

---

### Mock-ups:

N/A — match the web composer's per-platform options panels. Reference: web app Compose → select accounts → settings icon.

---

### Impact on existing data:

No schema or backend changes. All fields are already accepted by the API. Purely a mobile UI addition.

---

### Impact on other products:

- Web composer: no impact
- Android: no impact (separate story)
- Chrome extension: no impact

---

### Dependencies:

- **[iOS] Add TikTok posting options to iOS composer** — both stories implement the same `PostSettingVC.swift` settings screen; coordinate to avoid conflicts if worked in parallel

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, native iOS screen
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

## Story 4: [Android] Add missing platform-specific posting options to Android composer

### Description:

The Android composer (`ComposerSettingsActivity.java`) has a tabbed per-platform settings UI. Some platforms are partially implemented; LinkedIn has no options at all. This story adds the missing fields to bring Android to web parity for Facebook, Instagram, and LinkedIn.

**Current state in Android:**
- **Facebook:** Has `fbPostType` (feed/reel) and `fbVideo` (video title). Missing: "Story" as a post type option, "Share to Story" toggle.
- **Instagram:** Has `instaPostType` (feed/reel/feed_reel/story). Missing: API vs Mobile publishing method selector and device selection.
- **LinkedIn:** Zero platform-specific options in the settings tab.

**YouTube** is largely complete on Android (post type, title, privacy, category, license, playlist, kids/subscribers/embeddable toggles). The only gap is a **Tags** field — not included in this story as it is lower priority. Can be addressed separately.

**Files to update:**
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/ComposerSettingsActivity.java`
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/BottomSheetDialog/Data/SettingModel.java` — add: `fbShareToStory`, `instagramPublishingMethod`, `instagramDeviceId`, LinkedIn fields
- `contentstudio-android-v2/app/src/main/res/layout/activity_compose_settings.xml` — update Facebook card, Instagram card; add LinkedIn card

**Field specs and UI copy:**

---

#### Facebook — Add missing fields

**Add "Story" to Post Type**
- Existing `fbPostType` supports "feed" and "reel" — add "story" as a third option
- Option label: "Story" — subtext: "Disappears after 24 hours. Available for Pages only."

**Share to Story Toggle** (show for Feed and Reel post types)
- Toggle: "Also Share to Story"
- Subtext: "Share this post to your Facebook Story at the same time."
- Default: OFF
- Maps to `facebook_share_to_story: true` in the API payload

---

#### Instagram — Add missing fields

**Publishing Method**
- Label: "Publishing Method"
- Radio group / segmented control:
  - "API" (default) — subtext: "ContentStudio posts directly via the Instagram API."
  - "Mobile" — subtext: "ContentStudio notifies your phone. You complete the post in Instagram." (show only if user has a linked device)
- Maps to `instagram_posting_option` in the API payload

**Device Selection** (show only when "Mobile" is selected)
- Label: "Select Device"
- Dropdown of mobile devices the user has connected in ContentStudio
- Helper text: "Select which phone should receive the notification."

---

#### LinkedIn — Build from scratch

Add a LinkedIn tab/card to the settings screen (same pattern as existing Facebook/Instagram cards).

**Post Type**
- Label: "Post Type"
- Options: "Regular Post" (default), "Document Carousel", "Poll"

**Document Carousel** (shown when Document Carousel is selected)
- File upload — accepted: PDF, PPT, PPTX, DOC, DOCX
- Label: "Upload Document"
- Document Title — label: "Document Title" — placeholder: "e.g. Our 2025 Marketing Playbook" — required
- Helper text: "Your document will appear as a scrollable carousel on LinkedIn."

**Poll** (shown when Poll is selected)
- Poll Question — label: "Poll Question" — placeholder: "e.g. What's your top marketing challenge this year?" — required
- 2–4 answer options — labels: "Option 1", "Option 2", "Option 3", "Option 4" — minimum 2 required
- Duration — label: "Poll Duration" — options: "1 day", "3 days", "1 week", "2 weeks"

---

### Workflow:

1. User opens the ContentStudio Android app and goes to the Composer.
2. User selects Facebook, Instagram, or LinkedIn accounts.
3. User taps the settings icon to open `ComposerSettingsActivity`.
4. **Facebook tab:** User sees Post Type now includes "Story" as an option. For Feed or Reel, a "Also Share to Story" toggle is visible.
5. **Instagram tab:** User sees a Publishing Method selector (API / Mobile). If "Mobile" is chosen, a device dropdown appears.
6. **LinkedIn tab (new):** User sees Post Type options. Selecting Document Carousel shows document upload and title. Selecting Poll shows question, options, and duration.
7. User saves settings and completes composing. All new fields are passed in the API payload.

---

### Acceptance criteria:

- [ ] Facebook Post Type selector now includes "Story" as a valid option alongside Feed and Reel
- [ ] "Also Share to Story" toggle is shown for Facebook Feed and Reel post types; hidden when Story is selected
- [ ] `facebook_share_to_story: true` is included in the API payload when the toggle is ON
- [ ] Instagram Publishing Method selector (API / Mobile) is present; defaults to API
- [ ] Instagram device dropdown appears when "Mobile" is selected and correctly lists the user's connected devices
- [ ] LinkedIn tab/card appears in `ComposerSettingsActivity` when a LinkedIn account is selected
- [ ] LinkedIn Post Type selector shows Regular Post, Document Carousel, and Poll
- [ ] Selecting Document Carousel reveals a document upload field and a required Document Title field
- [ ] Selecting Poll reveals a required question field, 2–4 option fields (minimum 2 enforced), and a duration selector
- [ ] All new fields are stored in `SettingModel` and included in the correct fields in the API payload
- [ ] Existing Android Facebook and Instagram options (post type, video title, instaPostType) continue working correctly

---

### Mock-ups:

N/A — match the web composer's options panels for each platform. Integrate into the existing tab-card layout in `ComposerSettingsActivity`.

---

### Impact on existing data:

No backend or data changes. All fields are already supported by the API. `SettingModel` is extended with new fields; existing fields are unchanged.

---

### Impact on other products:

- Web composer: no impact
- iOS: no impact (separate story)
- Chrome extension: no impact

---

### Dependencies:

- **[Android] Complete TikTok posting options in Android composer** — both stories modify `ComposerSettingsActivity.java` and `SettingModel.java`; coordinate to avoid merge conflicts if worked in parallel

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, native Android screen
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
