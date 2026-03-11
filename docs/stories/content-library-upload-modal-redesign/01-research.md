# Research: Content Library Upload Modal Redesign

## Current State

The "Media Library" upload modal (`UploadMediaModal.vue`) is a global modal triggered from 22+ locations across the app via `EventBus.$emit('show-media-library-modal')`. It's mounted in `Home.vue` (app root) and used by Composer, AI Content Library, Automation, Discovery, Planner, and the Media Library page itself.

### Modal structure:
- **`UploadMediaModal.vue`** — outer modal shell (CstuModal, xl size, no header/footer)
- **`SideTabs.vue`** — vertical tab sidebar using `@contentstudio/ui` `Tabs` component
  - Title: "Add Media" (i18n key: `publisher.media_tabs.side_tabs.add_media`)
  - Tabs: Uploads, Media Library (conditional), Direct Link, Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google
  - Tab indices shift based on `type !== 'library'` — fragile numeric index system
- **Tab content components:**
  - `UploadFilesTab` — drag-and-drop upload
  - `MediaLibraryTab` — browse existing media (with sidebar, filters, folders)
  - `DirectUploadFileTab` — paste URL to import
  - `FetchMediaUrlTab` — Pinterest, Facebook, Flickr (URL input → fetch)
  - `SearchMediaTab` — Pixabay, Giphy (search input → results grid)
  - `DropBoxMediaTab` — Dropbox integration
  - `GoogleDriveAuth` — Google Drive integration

### Trigger points (22+ files, 56 occurrences):
- Composer v2: SocialModal, EditorBox, EditorCarouselBox, LiteEditorBox, MediaSelection, EditorMultiThreadsBox, EditorThreadedTweetsBox
- AI Content Library: CustomGenerateForm, UploadsTab
- Publisher: Options (social posting)
- Automation: EvergreenMain, BulkUploadAutomationSave
- Discovery: DiscoveryNavigationLayout
- Dashboard: ChatInput
- Media Library page: Header, MediaLibraryMain
- Video Clips: StepVideoImport
- Composable: `useFileWidget.js` (shared utility)

### Current sidebar UI issues:
- Uses `Tabs` with `variant="subtle"` — basic styling, no collapsibility
- Hard-coded numeric tab indices that shift based on context (`type !== 'library'`)
- No visual grouping or separation between upload sources vs. external sources
- Sidebar title is "Add Media" — needs update to "Content Library"

## What Needs to Change

1. **Rename**: All "Media Library" references → "Content Library" in the modal and all CTA trigger points
2. **Sidebar redesign**: Replace current `Tabs` sidebar with proper list items from the design system, make it collapsible
3. **Uploads tab**: Match the updated Content Library page layout (grid view, filters, team member filter dropdown) — shared UI with story sc-113550
4. **External source tabs** (Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google, Direct Link): Update the basic input+CTA UI per new design
5. **Modal title**: Update from "Add Media" to "Content Library" or per design

## Files Involved

### Core modal files (primary changes):
- `src/modules/publish/components/media-library/components/SideTabs.vue` — sidebar redesign + collapsibility
- `src/modules/publish/components/media-library/components/UploadMediaModal.vue` — modal shell updates
- `src/modules/publish/components/media-library/components/MediaTabs/MediaLibraryTab.vue` — uploads section layout update
- `src/modules/publish/components/media-library/components/FetchMediaUrlTab.vue` — external source UI update (Pinterest, Facebook, Flickr)
- `src/modules/publish/components/media-library/components/SearchMediaTab.vue` — external source UI update (Pixabay, Giphy)
- `src/modules/publish/components/media-library/components/DropBoxMediaTab.vue` — external source UI update
- `src/modules/publish/components/media-library/components/GoogleDriveAuth.vue` — external source UI update
- `src/modules/publish/components/media-library/components/DirectUploadFileTab.vue` — external source UI update

### i18n updates:
- `src/locales/en/publisher.json` (and 6 other locale files) — rename "Media Library" → "Content Library", "Add Media" → "Content Library"

### Trigger point updates (rename CTA labels):
- All 22+ files that emit `show-media-library-modal` — CTA labels need i18n key updates where visible to user
- `src/assets/sass/app/modules/publish/_media-library.scss` — CSS class references (if renaming)
- `src/assets/sass/app/components/modals/_modal_layout.scss` — `.media-library-modal-header` class

## Related Story
- [FE] Unified Content Library — rename and restructure Media Library into Content Library (sc-113550) — the Content Library page restructure. The modal's "Uploads" view should share the same layout/grid/filter UI.
