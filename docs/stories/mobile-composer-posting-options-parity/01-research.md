# Research: Mobile Composer Posting Options Parity (TikTok + All Platforms)

## Overview

Two related requests:
1. **TikTok sync** — Bring all TikTok settings from the web composer to iOS & Android (video/carousel, privacy, AI-generated/branded content, publishing method, scheduling)
2. **General platform sync** — Bring all web composer posting options to iOS & Android for Facebook, Instagram, LinkedIn, and others (post types, toggles, validations, scheduling parity)

---

## Current State

### Web Composer (FE — the reference)

**File:** `contentstudio-frontend/src/modules/composer_v2/components/ChannelOptions/`

#### TikTok (`TiktokOptions.vue`)
Full feature set on web:
| Field | Values |
|---|---|
| Publishing Method | `direct` / `notification` |
| Post Type | `video` / `carousel` |
| Carousel Title | Text, max 90 chars (carousel only) |
| Privacy Level | `PUBLIC_TO_EVERYONE` / `MUTUAL_FOLLOW_FRIENDS` / `SELF_ONLY` |
| Allow Comments | Toggle (default: on for carousel) |
| Allow Duet | Toggle (video only) |
| Allow Stitch | Toggle (video only) |
| Auto Add Music | Toggle (carousel only) |
| Disclose Branded Content | Toggle (`disclose_commercial_content`) |
| → Your Brand | Checkbox (`brand_organic_toggle`) |
| → Third Party Brand | Checkbox (`brand_content_toggle`) |
| AI-Generated Content | Toggle (`is_aigc`) |

#### Facebook (`FacebookOptions.vue`)
| Field | Values |
|---|---|
| Post Type | `feed` / `reel` / `story` |
| Video Title | Text, max 100 chars (video posts) |
| Share to Story | Toggle (for feed + reel posts) |

#### Instagram (`InstagramOptions.vue`)
| Field | Values |
|---|---|
| Publishing Method | `API` / `Mobile` (if devices linked) |
| Post Type | `feed` / `reel` / `feed+reel` / `story` / `carousel` |
| Share to Story | Toggle (feed/reel/carousel) |
| Device Selection | Multi-select (mobile publishing) |

#### LinkedIn
| Field | Values |
|---|---|
| Post Type | Regular / Poll / Document Carousel |
| Poll | Question, up to 4 options, duration |
| Carousel | Document upload + title |

#### YouTube (`YoutubeOptions.vue`)
| Field | Values |
|---|---|
| Post Type | `video` / `shorts` |
| Title | Text, max 100 chars (required) |
| Privacy | `public` / `private` / `unlisted` |
| Category | 14 categories |
| Playlist | Linked to account |
| License | `youtube` / `creative_commons` |
| Tags | Multi-tag input |
| Embeddable | Toggle |
| Notify Subscribers | Toggle |
| Made for Kids | Toggle |

---

### Backend (Laravel API)

**Key files:**
- `contentstudio-backend/app/Strategy/Planner/TikTokPosting.php` — TikTok publish strategy
- `contentstudio-backend/app/Rules/SocialMedia/TiktokOptionsRule.php` — TikTok validation
- `contentstudio-backend/config/socialPost.php` — Platform config values
- `contentstudio-backend/app/Strategy/Planner/Posting.php` — Strategy dispatcher

Mobile apps use the **same API endpoints** as web (`POST /processSocialShare`). No separate mobile endpoints. The backend fully supports all TikTok fields (`post_type`, `publishing_method`, `privacy_level`, `disable_comment`, `disable_duet`, `disable_stitch`, `auto_add_music`, `brand_organic_toggle`, `brand_content_toggle`, `is_aigc`, `disclose_commercial_content`). No backend changes needed — the API already accepts all fields.

---

### iOS App — Current State

**Key files:**
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/View/ComposerVC.swift`
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/ViewModel/ComposerViewModel.swift`
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/View/AccountList/PostSettingVC.swift` (stub/placeholder)

**TikTok:** ✅ Platform supported (listed in `Constant.swift` line 181; video validation exists in `ComposerViewController.swift`). ❌ Zero TikTok-specific *options* — `PostSettingVC.swift` is a 40-line stub with no implemented controls.

**Facebook:** ❌ No post type selection, no share-to-story toggle.

**Instagram:** Partial — basic image/video detection for app handoff (`PreProcessingIGView.swift`), but no post type picker, no API vs mobile publishing selection.

**LinkedIn:** ❌ No LinkedIn-specific options.

**YouTube:** ❌ No YouTube-specific options in composer settings.

---

### Android App — Current State

**Key files:**
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/ComposerSettingsActivity.java` — Per-platform settings tabs
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/BottomSheetDialog/Data/SettingModel.java` — Options model
- `contentstudio-android-v2/app/src/main/res/layout/activity_compose_settings.xml` — Layout

**TikTok:** Partial — has privacy level (3 options) + comment/duet/stitch toggles. Missing:
- Publishing method (direct vs notification)
- Post type (video vs carousel)
- Carousel title
- Auto-add music
- Branded content disclosure (`disclose_commercial_content`, `brand_organic_toggle`, `brand_content_toggle`)
- AI-generated content flag (`is_aigc`)

**Facebook:** Partial — `fbPostType` (feed/reel) + `fbVideo` title. Missing: share-to-story toggle, story post type.

**Instagram:** Partial — `instaPostType` (feed/reel/feed_reel/story). Missing: API vs mobile publishing method, device selection.

**LinkedIn:** ❌ Zero platform-specific options implemented.

**YouTube:** Good — has post type, title, privacy, category, license, playlist, isForKids, isNotifySubscribers, isEmbeddable. Missing: tags.

---

## What Needs to Change

### TikTok (iOS — from scratch)
- Add TikTok as a supported platform in the iOS composer
- Build TikTok options screen: publishing method, post type, carousel title, privacy, comment/duet/stitch toggles, auto-add music, branded content disclosure (your brand + third-party), AI-generated content toggle
- Pass all fields in the API payload to `tiktok_options`

### TikTok (Android — complete existing implementation)
- Add publishing method selector (direct vs notification)
- Add post type selector (video vs carousel)
- Add carousel title input (90 char max, shown only when carousel selected)
- Add auto-add music toggle (carousel only)
- Add branded content section: "Disclose Branded Content" toggle → "Your Brand" + "Third Party Brand" sub-checkboxes
- Add AI-generated content toggle (`is_aigc`)
- Map all new fields to `SettingModel` and include in API payload

### General Platform Options (iOS — from scratch for most)
- **Facebook:** Add post type picker (Feed/Reel/Story) + share-to-story toggle + video title field
- **Instagram:** Add post type picker (Feed/Reel/Story/Carousel) + publishing method (API vs Mobile)
- **LinkedIn:** Add LinkedIn options section (basic post type — regular text, poll creation, document carousel)
- **YouTube:** Add YouTube options: post type (Video/Shorts), title, privacy, category, playlist, license, toggles

### General Platform Options (Android — fill gaps)
- **Facebook:** Add share-to-story toggle + story as a post type option
- **Instagram:** Add API vs Mobile publishing method selector + device selection
- **LinkedIn:** Build LinkedIn options section from scratch (matches what iOS needs)

---

## Files Involved

**Frontend (reference only — no changes):**
- `contentstudio-frontend/src/modules/composer_v2/components/ChannelOptions/TiktokOptions.vue`
- `contentstudio-frontend/src/modules/composer_v2/components/ChannelOptions/FacebookOptions.vue`
- `contentstudio-frontend/src/modules/composer_v2/components/ChannelOptions/InstagramOptions.vue`
- `contentstudio-frontend/src/modules/composer_v2/components/ChannelOptions/YoutubeOptions.vue`
- `contentstudio-frontend/src/modules/composer_v2/views/composerInitialState.js`

**Backend (no changes needed — API already supports all fields):**
- `contentstudio-backend/app/Strategy/Planner/TikTokPosting.php`
- `contentstudio-backend/app/Rules/SocialMedia/TiktokOptionsRule.php`

**iOS (changes needed):**
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/View/AccountList/PostSettingVC.swift` (expand from stub)
- `contentstudio-ios-v2/ContentStudio/Controllers/Composer/ViewModel/ComposerViewModel.swift`
- New: TikTok options view, per-platform settings views

**Android (changes needed):**
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/ComposerSettingsActivity.java`
- `contentstudio-android-v2/app/.../ComposerActivity/Settings/BottomSheetDialog/Data/SettingModel.java`
- `contentstudio-android-v2/app/src/main/res/layout/activity_compose_settings.xml`

---

## Story Split

Given the scope, this warrants a **dedicated epic** (user requested). Stories:

1. **[iOS] Add TikTok posting options to iOS composer** — Full TikTok options from scratch (publishing method, post type, carousel title, privacy, toggles, branded content, AIGC)
2. **[Android] Complete TikTok posting options in Android composer** — Fill the 6 missing fields (publishing method, post type, carousel title, auto-music, branded content, AIGC)
3. **[iOS] Add platform-specific posting options to iOS composer** — Facebook (post type, share-to-story), Instagram (post type, publishing method), LinkedIn (basic options), YouTube (options)
4. **[Android] Add missing platform-specific posting options to Android composer** — Facebook (share-to-story, story type), Instagram (publishing method, device selection), LinkedIn (from scratch)

4 stories total — fits within the `/story` pipeline limit.
