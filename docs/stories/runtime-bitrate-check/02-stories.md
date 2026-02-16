# Stories: Runtime Bitrate Check During Post Creation

---

## [BE] Extract video bitrate metadata and add platform-specific bitrate validation config

### Description:

Update the video processing pipeline to extract and store bitrate metadata so the frontend can validate videos against platform-specific bitrate limits before publishing.

**1. Enhance bitrate extraction in `Media::fetchMediaThumbnail()`** (`app/Libraries/Media.php`, lines 338–403):

The transcode API (`env('TRANSCODE_API') . 'v2/transcode'`) currently returns `video_codec`, `frame_rate`, `audio_codec`, `audio_hertz`, `duration`, `width`, `height` — but **not bitrate**. After the transcode response is parsed (line 351), extract and store bitrate:

- If the transcode API already returns `bitrate` (or `video_bitrate`) in its response, capture it alongside the other metadata fields at lines 375–378.
- If the transcode API does **not** return bitrate, coordinate with the transcode server team to add it. The transcode server uses FFmpeg/FFprobe internally — bitrate is available via `ffprobe -v quiet -print_format json -show_format` → `format.bit_rate`.
- Store as `$details['bitrate']` in **kbps** (kilobits per second). If the transcode API returns bits per second, divide by 1000.
- Fallback: if bitrate is unavailable from the transcode response, calculate an approximate bitrate from file size and duration: `bitrate_kbps = (file_size_bytes * 8) / (duration_seconds * 1000)`. This is an overall bitrate (video + audio) which is sufficient for validation purposes.

**2. Store bitrate in media metadata:**

In the same `fetchMediaThumbnail()` method, add the `bitrate` field to the `$details` array so it gets persisted to MongoDB via `MediaRepository::saveMedia()`. No schema migration needed — MongoDB is schemaless.

**3. Add bitrate to the media upload API response:**

Ensure the `POST /api/media_library/assets/process-uploaded-gcs-media` endpoint (`MediaLibraryAssetsController`) returns `bitrate` in its response payload alongside existing fields (`duration`, `width`, `height`, `video_codec`, etc.), so the frontend has it immediately after upload processing completes.

**4. Add platform-specific bitrate limits to `config/socialAccountsValidationConfig.php`:**

Add `bitrate_max` (in kbps) to the `video` section of each platform that enforces or recommends a maximum bitrate:

| Platform | `bitrate_max` (kbps) | `bitrate_max_error` |
|----------|----------------------|---------------------|
| LinkedIn | 30000 | "Video bitrate exceeds LinkedIn's 30 Mbps limit. Please compress the video or use a lower quality setting." |
| Facebook | 25000 | "Video bitrate exceeds 25 Mbps. Facebook may reject or heavily re-encode this video. Consider compressing it." |
| Instagram | 25000 | "Video bitrate exceeds 25 Mbps. Instagram may reject or heavily re-encode this video. Consider compressing it." |
| Twitter/X | 25000 | "Video bitrate exceeds 25 Mbps. X (Twitter) may reject or heavily re-encode this video. Consider compressing it." |
| TikTok | 25000 | "Video bitrate exceeds 25 Mbps. TikTok may reject or heavily re-encode this video. Consider compressing it." |
| Threads | 25000 | "Video bitrate exceeds 25 Mbps. Threads may reject or heavily re-encode this video. Consider compressing it." |
| Bluesky | 25000 | "Video bitrate exceeds 25 Mbps. Bluesky may reject this video. Consider compressing it." |
| Pinterest | 25000 | "Video bitrate exceeds 25 Mbps. Pinterest may reject or heavily re-encode this video. Consider compressing it." |
| YouTube | — | No limit needed — YouTube handles very high bitrates |
| Tumblr | 25000 | "Video bitrate exceeds 25 Mbps. Tumblr may reject this video. Consider compressing it." |

LinkedIn has an official 30 Mbps hard limit. Other platforms don't enforce a hard cap but videos above ~25 Mbps are excessively large and likely to be aggressively re-encoded or rejected, so we flag them as a warning-level threshold.

---

### Workflow:

1. User uploads a video in the Composer
2. The system processes the video and extracts metadata including bitrate
3. The bitrate value is saved with the media record and returned to the frontend
4. The frontend uses the bitrate value along with platform-specific limits to flag incompatible videos before publishing

---

### Acceptance criteria:

- [ ] `Media::fetchMediaThumbnail()` extracts and stores `bitrate` (in kbps) from the transcode API response
- [ ] If the transcode API does not return bitrate, the system calculates an approximate bitrate from `(file_size_bytes * 8) / (duration_seconds * 1000)`
- [ ] Bitrate is persisted in the MongoDB `media` document alongside existing video metadata
- [ ] The `process-uploaded-gcs-media` endpoint returns `bitrate` in its response payload
- [ ] `socialAccountsValidationConfig.php` includes `bitrate_max` and `bitrate_max_error` for LinkedIn (30000 kbps), Facebook, Instagram, Twitter/X, TikTok, Threads, Bluesky, Pinterest, and Tumblr (25000 kbps each)
- [ ] YouTube has no bitrate limit configured (it accepts very high bitrates)
- [ ] Existing video uploads without bitrate data continue to work — `bitrate` is nullable/optional
- [ ] The bitrate calculation fallback handles edge cases: zero duration, missing file size

---

### Mock-ups:

N/A — backend only. No UI changes in this story.

---

### Impact on existing data:

No schema migration required — MongoDB is schemaless. Existing media documents without `bitrate` continue to function normally. New uploads will have the `bitrate` field populated. There is no backfill needed for existing media.

---

### Impact on other products:

- **Mobile apps:** No impact — mobile apps don't perform video validation against platform configs. They will simply ignore the new `bitrate` field if present in API responses.
- **Chrome extension:** No impact.
- **White-label:** No impact — validation config is shared across all white-label instances.
- **Transcode server:** May need a minor update to include `bitrate` in its response if it doesn't already. This is a separate service — coordinate with the infra team.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

## [FE] Display video bitrate validation errors in Composer

### Description:

Add real-time bitrate validation to the Composer so users see a clear error when their uploaded video exceeds a platform's bitrate limit — before they schedule or publish. This leverages the existing video validation error system in `EditorMediaBox.vue`.

**1. Add bitrate limits to frontend platform config:**

In `src/modules/integration/config/api-utils.js`, add `bitrate_max` (in kbps) to each platform's video validation section, matching the backend config values:
- LinkedIn: `30000`
- Facebook, Instagram, Twitter/X, TikTok, Threads, Bluesky, Pinterest, Tumblr: `25000`
- YouTube: no limit

**2. Store bitrate from upload response:**

In `src/composables/useComposerMediaUpload.js`, when the `process-uploaded-gcs-media` response is received, ensure the `bitrate` field is stored in the media details object in Vuex (`getSocialSharingMediaDetails`), alongside existing fields like `duration_seconds`, `width`, `height`, `video_codec`.

**3. Add bitrate validation in `getVideoErrors()`:**

In `src/modules/composer_v2/components/EditorBox/EditorMediaBox.vue` (lines 1097–1372), add a bitrate check to the existing `getVideoErrors()` method. For each selected platform, if the platform config has `bitrate_max` and the video's `bitrate` exceeds it, add an error entry to the errors array.

**4. Error display:**

The existing error badge system on video thumbnails (lines 62–95) handles display automatically — a red warning badge appears on the thumbnail, and clicking it shows the error details. No new UI components needed.

**5. UI copy for error messages:**

For each platform, the error message follows this pattern:

| Platform | Error message |
|----------|---------------|
| LinkedIn | "Video bitrate is too high for LinkedIn (max 30 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Facebook | "Video bitrate is too high for Facebook (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Instagram | "Video bitrate is too high for Instagram (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Twitter/X | "Video bitrate is too high for X (Twitter) (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| TikTok | "Video bitrate is too high for TikTok (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Threads | "Video bitrate is too high for Threads (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Bluesky | "Video bitrate is too high for Bluesky (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Pinterest | "Video bitrate is too high for Pinterest (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |
| Tumblr | "Video bitrate is too high for Tumblr (max 25 Mbps). Compress the video or export at a lower quality setting to reduce bitrate." |

**Tooltip on the error badge** (existing pattern — shows when hovering the red warning icon on the video thumbnail):
> "This video's bitrate ({X} Mbps) exceeds the limit for {platform}. High-bitrate videos may be rejected or lose quality when the platform re-encodes them. Try exporting your video at a lower bitrate (e.g., 8–10 Mbps for 1080p)."

---

### Workflow:

1. User opens the Composer and creates a new post
2. User selects multiple social accounts (e.g., LinkedIn, Instagram, Facebook)
3. User uploads a high-bitrate video (e.g., 35 Mbps 4K ProRes export)
4. After the video finishes processing, a red warning badge appears on the video thumbnail
5. User clicks the warning badge and sees: "Video bitrate is too high for LinkedIn (max 30 Mbps). Compress the video or export at a lower quality setting to reduce bitrate."
6. User also sees a tooltip explaining: "This video's bitrate (35 Mbps) exceeds the limit for LinkedIn. High-bitrate videos may be rejected or lose quality when the platform re-encodes them. Try exporting your video at a lower bitrate (e.g., 8–10 Mbps for 1080p)."
7. User removes the video, compresses it externally, and re-uploads a lower bitrate version
8. The warning badge disappears — the video now passes all platform checks
9. User clicks "Schedule" or "Publish Now" and the post is created successfully

---

### Acceptance criteria:

- [ ] When a video's bitrate exceeds the platform-specific limit, a red warning badge appears on the video thumbnail in the Composer
- [ ] Clicking the warning badge shows the platform-specific error message (e.g., "Video bitrate is too high for LinkedIn (max 30 Mbps)...")
- [ ] Hovering the warning badge shows the tooltip with the actual bitrate value and recommendation
- [ ] LinkedIn videos above 30 Mbps show the bitrate error
- [ ] Facebook, Instagram, Twitter/X, TikTok, Threads, Bluesky, Pinterest, and Tumblr videos above 25 Mbps show the bitrate error
- [ ] YouTube videos do not trigger a bitrate error regardless of bitrate
- [ ] If a video targets multiple platforms, the error shows for each platform that the bitrate exceeds
- [ ] Videos with bitrate within limits show no bitrate-related error
- [ ] If the backend does not return a `bitrate` value (e.g., older media), no bitrate validation error is shown (graceful fallback)
- [ ] The bitrate error appears alongside existing video errors (size, duration, aspect ratio) — it does not replace them
- [ ] Single-platform posts (only one platform selected) show bitrate errors correctly when applicable

---

### Mock-ups:

N/A — uses existing video error badge and tooltip pattern in the Composer. No new UI components or layouts needed.

---

### Impact on existing data:

No data changes. Existing videos in the media library without `bitrate` metadata are not affected — bitrate validation is skipped if the field is absent.

---

### Impact on other products:

- **Mobile apps:** No impact — mobile Composer does not perform bitrate validation.
- **Chrome extension:** No impact.
- **White-label:** No impact — uses theme-aware styles (existing error badge pattern).

---

### Dependencies:

Depends on: **[BE] Extract video bitrate metadata and add platform-specific bitrate validation config** — the frontend needs the `bitrate` field in the media upload response and the platform config values.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
