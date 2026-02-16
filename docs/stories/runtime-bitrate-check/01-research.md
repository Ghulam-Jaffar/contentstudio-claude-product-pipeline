# Research: Runtime Bitrate Check During Post Creation

## Current State

### Video Upload Flow
1. User uploads video in Composer via drag-drop, paste, or file picker
2. Frontend sends file to GCS via signed URL (`POST /api/media_library/assets/generate-signed-url`)
3. Backend processes uploaded media via external transcode API (`env('TRANSCODE_API') . 'v2/transcode'`)
4. Transcode API returns metadata: duration, resolution (width/height), video_codec, frame_rate, audio_codec, audio_hertz, thumbnail, converted video
5. Metadata saved to MongoDB `media` collection
6. Frontend validates video against platform-specific rules and displays errors on the media thumbnail

### What metadata is extracted today
- Duration (HH:MM:SS + seconds)
- Resolution (width, height)
- Video codec (h264, hevc)
- Frame rate
- Audio codec (aac)
- Audio sample rate (hertz)
- **Bitrate: NOT extracted** — transcode API response does not include bitrate

### What is validated today
Frontend (`EditorMediaBox.vue` → `getVideoErrors()`) checks per-platform:
- File size (max, min)
- Duration (max, min)
- Resolution (width, height)
- Aspect ratio (min, max)
- Video codec (h264/hevc for Instagram, Threads, Bluesky)
- Frame rate range (for Instagram, TikTok, Threads, Bluesky)
- Audio codec and sample rate (for Instagram, Threads, Bluesky)
- **Bitrate: NOT validated** — no platform config includes bitrate limits

### Platform bitrate limits (from official docs)
| Platform | Max Bitrate | Notes |
|----------|-------------|-------|
| LinkedIn | 30 Mbps | Highest allowance |
| Facebook | No hard max | Recommends ≤8 Mbps for 1080p |
| Instagram | No hard max | Re-encodes everything; recommends ≤5 Mbps |
| TikTok | No hard max | Re-encodes everything; min 516 kbps for ads |
| Twitter/X | No hard max | Recommends ≤5 Mbps |
| YouTube | No hard max | Accepts very high bitrates |
| Pinterest | No hard max | Recommends ≤5 Mbps |
| Bluesky | No hard max | 50MB file limit effectively caps bitrate |
| Threads | No hard max | Same as Instagram |
| Tumblr | No hard max | 100MB file limit effectively caps it |

Most platforms don't enforce a hard bitrate limit — they re-encode. The main risk is that excessively high bitrate causes: (a) large file sizes that exceed platform limits, (b) upload timeouts, (c) quality loss from aggressive re-encoding.

## What Needs to Change

### Backend
1. **Enhance transcode API response** (or extract locally) to include `bitrate` (video bitrate in kbps/Mbps)
2. **Store `bitrate` in media metadata** — add field to the media details array in `Media::fetchMediaThumbnail()`
3. **Add bitrate limits to `socialAccountsValidationConfig.php`** — for platforms that enforce limits (LinkedIn: 30 Mbps) and recommended maximums for others
4. **Return bitrate in media API response** so frontend can validate

### Frontend
5. **Add bitrate validation in `getVideoErrors()`** — check against platform-specific bitrate limits from config
6. **Display bitrate error message** in the existing error badge/tooltip on video thumbnails
7. **Update platform config** (`socialIntegrationsConfigurations`) with bitrate limits

## Files Involved

### Backend
- `app/Libraries/Media.php:338-403` — `fetchMediaThumbnail()` — add bitrate extraction from transcode response
- `config/socialAccountsValidationConfig.php` — add `bitrate_max` per platform video config
- `app/Http/Controllers/Storage/MediaLibrary/MediaLibraryAssetsController.php` — ensure bitrate is returned in upload response
- Transcode server (external) — may need update to return bitrate in response

### Frontend
- `src/modules/composer_v2/components/EditorBox/EditorMediaBox.vue:1097-1372` — `getVideoErrors()` — add bitrate validation
- `src/modules/integration/config/api-utils.js:128-416` — add bitrate limits to platform configs
- `src/composables/useComposerMediaUpload.js` — ensure bitrate metadata is stored after upload
