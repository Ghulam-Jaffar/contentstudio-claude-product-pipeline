# Composer v2 Refactor — Epic & Stories

---

## Epic

**Title:** Composer v2 Module Refactor

**Description:**
Refactor the ContentStudio Composer v2 module from a monolithic, tightly-coupled system (~51,000 lines, 119 files) into a modular, maintainable architecture with independent, reusable components.

The current composer has severe architectural debt: a 7,110-line god component (SocialModal.vue), 80+ props drilled through MainComposer, 50+ through EditorBox, a 1,576-line utility composable, 48 files using EventBus anti-pattern, and scattered validation logic. No component can be reused independently outside the module.

This epic delivers a clean state layer (Pinia + provide/inject), decomposes god components into focused units, eliminates prop drilling, replaces EventBus with typed events, and moves validations to the backend. Zero UI changes — same behavior, new internals. Prerequisite: sc-110785 (backend validation migration).

---

## Stories

---

### Story 1: [FE] Create Pinia composer store and domain composables

### Description:

Create the foundational state layer that replaces prop drilling and EventBus usage across the composer module. This is the most critical story — everything else builds on it.

**Create Pinia store** at `src/modules/composer_v2/store/composerStore.js`:

The store is the single source of truth for all composer state. Define the state schema:

```javascript
state: () => ({
  // Post data
  post: {
    commonDetails: { message: '', image: [], video: null, url: '', mentions: [], location: null },
    platformDetails: {}, // keyed by platform slug
    threadedPosts: [],    // for Twitter/Bluesky threads
    carouselItems: [],    // for carousel posts
    firstComment: { message: '', image: [] },
  },
  // Account selection
  accounts: {
    selected: [],         // array of account objects
    contentCategory: null,
    contentCategorySlot: null,
  },
  // Scheduling
  schedule: {
    publishTimeOption: 'now',  // now | schedule | queue | repeat
    scheduledAt: null,
    repeatConfig: null,
    queueStatus: false,
  },
  // UI state
  ui: {
    activeTab: 'common',
    showAccountSidebar: false,
    showActionsSidebar: false,
    activeModal: null,      // string name of open modal, or null
    isMobile: false,
    previewVisible: false,
  },
  // Validation (from backend responses)
  validation: {
    errors: [],            // array of { field, message, error_code }
    warnings: [],
  },
  // Meta
  meta: {
    planId: null,          // null for new post, string for editing
    isTemplateMode: false,
    isPublishedPost: false,
    approval: null,
    processingLoader: false,
    lastUpdated: null,
  },
})
```

Define actions for:
- `initializeForNewPost()` — reset to blank state
- `initializeForEdit(planId)` — fetch plan data from API, hydrate state
- `initializeFromTemplate(templateData)` — populate from template
- `resetStore()` — clean reset on modal close
- `updatePlatformDetails(platform, data)` — update platform-specific post data
- `setValidationErrors(errors)` — populate from backend response
- `clearValidationErrors()` — clear on user edit

**Create 8 domain composables** in `src/modules/composer_v2/composables/`:

1. **`useComposerPost.js`** — Post content read/write. Returns `{ commonDetails, platformDetails, updateCommonMessage, updatePlatformField, addThread, removeThread }`. Reads from store.
2. **`useAccountSelection.js`** — Account list, selection state, filtering. Returns `{ selectedAccounts, selectAccount, deselectAccount, filteredAccounts, searchQuery, selectedPlatforms }`. Reads from store + Vuex integrations.
3. **`usePostSchedule.js`** — Scheduling state. Returns `{ publishTimeOption, scheduledAt, repeatConfig, setSchedule, setQueue }`. Reads from store.
4. **`useComposerMedia.js`** — Media upload, gallery, resize coordination. Returns `{ images, videos, uploadMedia, removeMedia, isUploading }`. Wraps existing `useComposerMediaUpload`.
5. **`useComposerValidation.js`** — Validation error state from backend. Returns `{ errors, warnings, getFieldError, hasErrors, clearErrors }`. Reads from store.
6. **`useComposerUI.js`** — Sidebar/modal/tab visibility. Returns `{ activeTab, setActiveTab, showModal, hideModal, activeModal, toggleSidebar }`. Reads from store.
7. **`usePlatformMetadata.js`** — Platform icons, names, colors, config. Returns `{ getPlatformIcon, getPlatformName, getPlatformColor, platformConfig }`. Extracted from `useComposerHelper`.
8. **`useMobileDetection.js`** — Viewport tracking. Returns `{ isMobile, isTablet }`. Extracted from `useComposerHelper`.

Each composable:
- Is <300 lines
- Has a single responsibility
- Does NOT access Vuex directly (goes through Pinia store or receives injected dependencies)
- Has no lifecycle hooks (pure reactive logic)
- Returns a clean API of refs and functions

**Wire the store into SocialModal** (initial integration — SocialModal still large at this point):
- SocialModal creates the store instance on mount: `const store = useComposerStore()`
- SocialModal provides it: `provide('composerStore', store)`
- Begin using store for one data flow as proof of concept (e.g., `ui.activeTab`)
- Old prop-based flow remains for everything else (gradual migration)

---

### Workflow:

1. User opens the Composer modal — behavior is identical to before
2. User creates a new post — same UI, same flow, same result
3. User edits an existing post from the Planner — same behavior
4. User applies a template — same behavior
5. User closes the modal — state resets cleanly
6. No visible difference to the user — all changes are internal architecture

---

### Acceptance criteria:

- [ ] Pinia `composerStore` created with the full state schema (post, accounts, schedule, ui, validation, meta)
- [ ] Store actions work: `initializeForNewPost()`, `initializeForEdit()`, `initializeFromTemplate()`, `resetStore()`
- [ ] 8 domain composables created, each <300 lines with single responsibility
- [ ] `useComposerPost` reads/writes post state from store
- [ ] `useAccountSelection` reads/writes account selection from store
- [ ] `usePostSchedule` reads/writes scheduling state from store
- [ ] `useComposerMedia` wraps media upload logic with clean API
- [ ] `useComposerValidation` reads validation errors from store
- [ ] `useComposerUI` manages UI state (tabs, sidebars, modals)
- [ ] `usePlatformMetadata` provides platform icons/names/colors without Vuex
- [ ] `useMobileDetection` provides responsive state without lifecycle hooks in composable
- [ ] SocialModal creates store on mount and provides it via `provide()`
- [ ] At least one data flow migrated to store as proof of concept
- [ ] All existing E2E tests pass — zero regressions
- [ ] No composable accesses Vuex store directly

---

### Mock-ups:

N/A — zero UI changes, internal architecture only

---

### Impact on existing data:

- No database or API changes
- Pinia store is client-side only — no persisted data changes
- Existing Vuex store remains functional during migration

---

### Impact on other products:

- **Mobile apps:** No impact — mobile has its own composer
- **Chrome extension:** No impact — uses API directly
- **White-label:** No impact — no visual changes

---

### Dependencies:

- None — this is the foundation story, no prerequisites besides existing codebase

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, no UI changes
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no UI changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 2: [FE] Replace EventBus with typed events and store actions in composer module

### Description:

Replace all 48 EventBus usages in the composer_v2 module with store actions and `mitt` typed events. This eliminates the anti-pattern of untyped global events that cause memory leaks, silent failures, and untraceable data flow.

**Install `mitt`** as a project dependency (lightweight typed event emitter, ~200 bytes gzipped).

**Create typed event bus** at `src/modules/composer_v2/lib/composerEvents.js`:
```javascript
import mitt from 'mitt'
// Typed event map — documents every event
const composerEmitter = mitt()
export { composerEmitter }
// Event types: 'template:apply', 'audio:select', 'media:uploaded', etc.
```

**Migration strategy — file by file:**

1. Audit all 48 files using EventBus in `composer_v2/`
2. Categorize each usage:
   - **Store action** — events that modify composer state (e.g., template apply, account change) → replace with store action
   - **Typed mitt event** — events for cross-module communication (e.g., planner → composer open) → replace with `composerEmitter`
   - **Vue emits** — events between parent-child components → replace with standard Vue `emits`
3. Migrate one file at a time, verify E2E tests pass after each
4. Remove `EventBus` import from each migrated file
5. After all 48 files migrated, verify zero `EventBus` references remain in `composer_v2/`

**Key replacements:**

| Current EventBus Event | Replacement |
|---|---|
| `handle-external-template-open` | `composerEmitter.emit('template:apply', data)` |
| `audio-track-selected` | Store action `store.updatePlatformDetails('audio', { trackId })` |
| `modal-opened` / `modal-closed` | Store action `store.ui.activeModal = name` / `= null` |
| `post-updated` | Store watcher (reactive — no event needed) |
| Parent → child events | Vue `emits` + `defineEmits()` |

**Cleanup:**
- Remove `import { EventBus } from '@common/lib/event-bus'` from all composer_v2 files
- Ensure cleanup handlers (`mitt.off()`) are registered in `onUnmounted` for every listener
- No memory leak risk — mitt automatically cleans up with `emitter.all.clear()`

---

### Workflow:

1. User opens the Composer from Planner by clicking "Edit" on a post — works the same as before
2. User applies a saved template from the template panel — works the same
3. User selects an audio track from the media library — works the same
4. User opens/closes modals within the composer — works the same
5. All cross-module triggers (planner → composer, header → composer) continue working

---

### Acceptance criteria:

- [ ] `mitt` installed as a project dependency
- [ ] Typed event bus created at `src/modules/composer_v2/lib/composerEvents.js`
- [ ] All 48 EventBus usages in composer_v2 replaced (zero `EventBus` imports remaining)
- [ ] State-modifying events replaced with Pinia store actions
- [ ] Cross-module events replaced with `composerEmitter` (mitt)
- [ ] Parent-child events replaced with Vue `emits`
- [ ] All `mitt` listeners cleaned up in `onUnmounted` (no memory leaks)
- [ ] Cross-module communication works: planner → composer open, header → composer open
- [ ] Template application via external trigger still works
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — zero UI changes

---

### Impact on existing data:

- No data changes — event system is client-side only
- New `mitt` dependency added (~200 bytes gzipped)

---

### Impact on other products:

- **Planner module:** Uses EventBus to open composer — needs update to use `composerEmitter` or a shared mitt instance
- **Publisher module:** Same — EventBus trigger needs migration
- **Mobile apps:** No impact
- **Chrome extension:** No impact

---

### Dependencies:

- Depends on: **[FE] Create Pinia composer store and domain composables** — store must exist for state-modifying event replacements

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, no UI changes
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no UI changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 3: [FE] Decompose SocialModal into thin layout shell with independent panels

### Description:

Reduce SocialModal.vue from 7,110 lines to <500 lines by extracting it into a thin layout shell that delegates all business logic to child panels and the Pinia store.

**Refactored SocialModal.vue** (~400-500 lines) becomes:
- Modal wrapper (`CstuModal`) with open/close lifecycle
- Store initialization on open (`composerStore.initializeForNewPost()` or `initializeForEdit(planId)`)
- Store reset on close (`composerStore.resetStore()`)
- `provide('composerStore', store)` for all children
- Layout grid: left sidebar | main content | right sidebar
- Responsive: drawer wrappers for mobile
- No business logic — just mount/unmount orchestration

**Extract `AccountSelectionPanel.vue`** (from AccountSelectionAside.vue, 1,935 lines):
- Self-contained component
- Uses `inject('composerStore')` + `useAccountSelection()` composable
- Zero props from parent — reads account data from store
- Manages its own search/filter UI state locally
- Emits nothing to parent — writes to store directly
- **Reusable:** Can be embedded in automation, bulk scheduling, or any context that provides the store

**Extract `ComposerMainArea.vue`** (from MainComposer.vue, 4,072 lines — first pass):
- Replaces MainComposer as the central content area
- Uses `inject('composerStore')` — zero props from SocialModal
- Renders: tab navigation, active editor variant, platform options, footer
- Still large at this point (~2,000 lines) — further decomposition in later stories
- The goal here is to eliminate the 80+ prop chain from SocialModal → MainComposer

**Extract `ComposerSidebarPanel.vue`** (from ComposerSidebar/ActionsAside.vue, 788 lines):
- Self-contained right sidebar
- Uses `inject('composerStore')` — zero props
- Renders: campaigns, labels, comments, tasks, AI assistant, members, activities
- **Reusable:** sidebar sections can be independently imported

**Modal components updated:**
- All modals (AiCaptionModal, TemplateModal, ApprovalModal, etc.) switch from props to `inject('composerStore')`
- Each modal reads what it needs from store, writes back to store
- SocialModal no longer manages modal state — `useComposerUI()` handles it

---

### Workflow:

1. User clicks "Create Post" in the header — composer modal opens with left sidebar (accounts), center (editor), right sidebar (actions)
2. User selects social accounts in the left sidebar — selections update in the center area instantly
3. User writes content in the editor — content state is managed by the store
4. User opens the AI Caption modal — modal reads current content from store, writes generated caption back
5. User opens the right sidebar to add labels/campaigns — sidebar reads/writes store independently
6. User schedules and publishes — same flow, same result
7. Everything looks and works identically to before

---

### Acceptance criteria:

- [ ] SocialModal.vue is <500 lines (from 7,110)
- [ ] SocialModal contains only: modal wrapper, store lifecycle, provide, layout grid, responsive drawers
- [ ] SocialModal has zero business logic — no validation, no data fetching, no platform-specific code
- [ ] AccountSelectionPanel works with zero props — uses inject/composable only
- [ ] AccountSelectionPanel is reusable: can render in any context that provides the store
- [ ] ComposerMainArea works with zero props from SocialModal — uses inject only
- [ ] ComposerSidebarPanel works with zero props — uses inject only
- [ ] All modals use `inject('composerStore')` instead of props
- [ ] Account selection, content editing, scheduling, publishing all work identically
- [ ] Edit mode (from planner) works — store hydrates from plan data
- [ ] Template mode works — store hydrates from template data
- [ ] Mobile responsive layout works — drawers for account selection and actions sidebar
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — zero UI changes

---

### Impact on existing data:

- No data changes
- Component file structure changes (new files, SocialModal significantly smaller)

---

### Impact on other products:

- **Planner:** Edit post flow unchanged — same route, same component, store handles initialization
- **Mobile apps:** No impact
- **Chrome extension:** No impact

---

### Dependencies:

- Depends on: **[FE] Create Pinia composer store and domain composables** — store must exist
- Depends on: **[FE] Replace EventBus with typed events and store actions in composer module** — EventBus must be gone before restructuring components

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A for visual changes, but mobile drawer layout must still work
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no visual changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 4: [FE] Decompose EditorBox into independent reusable components

### Description:

Break EditorBox.vue (2,873 lines, 50+ props) into 5 independent components that can each be used standalone outside the composer.

**Extract `EditorTextBox.vue`** — Pure rich text editing:
- Wraps `CstTextArea` component
- Accepts: `v-model` for text content, `maxLength` for character limit, `platform` for mention config
- Provides: text input, character count display, mention suggestions, paste handling
- Does NOT include: media, AI buttons, platform options, link previews
- Uses `useComposerPost()` composable for platform-aware message access
- **Reusable:** Can embed this anywhere text input with character counting is needed (automation, templates, bulk editor)

**Extract `EditorMediaManager.vue`** — Media attachment management:
- Refactored from EditorMediaBox.vue (1,967 lines → target <800 lines)
- Manages: media gallery, upload triggers, drag-drop zone, carousel/multi-image
- Uses `useComposerMedia()` composable
- Accepts: minimal config props (allowed media types, max count)
- **Reusable:** Can embed for any media attachment context

**Extract `EditorAIActions.vue`** — AI action toolbar:
- AI caption generation button
- Improve text button
- Hashtag suggestion button
- Image generation trigger
- Uses its own composable for AI API calls
- Reads current text from store to pass to AI
- Writes AI results back to store
- **Reusable:** Can embed in any text editing context that has AI features

**Extract `EditorToolbar.vue`** — Options bar:
- UTM parameter dropdown
- Hashtag group selector
- Replug link shortener
- Emoji picker
- Uses `useComposerPost()` to read/write UTM and hashtag state
- **Reusable:** Toolbar items can be selectively rendered

**Extract `EditorLinkPreview.vue`** — Link card preview:
- Detects URLs in text content
- Fetches and displays link metadata (title, description, image)
- Uses existing `useComposerLinkPreview()` composable
- Reads URL from store, writes preview data back
- **Reusable:** Can show link previews anywhere

**Refactored EditorBox.vue** (~300-400 lines) becomes a composition wrapper:
```vue
<template>
  <div class="editor-box">
    <EditorToolbar />
    <EditorTextBox v-model="message" :max-length="charLimit" :platform="activePlatform" />
    <EditorAIActions />
    <EditorMediaManager :allowed-types="allowedMediaTypes" :max-count="maxMediaCount" />
    <EditorLinkPreview />
  </div>
</template>
```

Platform-specific editor variants (EditorFirstCommentBox, EditorThreadedTweetsBox, EditorCarouselBox, EditorBlueskyBox) are updated to use the extracted components instead of duplicating editor logic.

---

### Workflow:

1. User opens the Composer and starts typing in the editor — text input works identically
2. User sees the character count update as they type — same behavior
3. User clicks the AI caption button — AI generates a caption and inserts it, same as before
4. User drags an image into the editor — media uploads and appears in gallery, same as before
5. User pastes a URL — link preview card appears, same as before
6. User adds UTM parameters from the toolbar — same behavior
7. User switches to Twitter thread mode — threaded editor uses the same EditorTextBox per tweet

---

### Acceptance criteria:

- [ ] EditorTextBox works standalone with just `v-model`, `maxLength`, `platform` props
- [ ] EditorTextBox provides character counting, mention suggestions, paste handling
- [ ] EditorMediaManager works with `useComposerMedia()` composable — no props from EditorBox
- [ ] EditorMediaManager is <800 lines (from 1,967 in EditorMediaBox)
- [ ] EditorAIActions reads current text from store and writes AI results back
- [ ] EditorToolbar provides UTM, hashtags, Replug, emoji — reads/writes store
- [ ] EditorLinkPreview detects URLs and displays link metadata
- [ ] Refactored EditorBox.vue is <400 lines (from 2,873)
- [ ] EditorBox composes the 5 extracted components correctly
- [ ] Platform-specific variants (threaded, carousel, first comment) use extracted components
- [ ] No component receives >15 props
- [ ] All text editing, media upload, AI generation, link preview functionality works identically
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — zero UI changes

---

### Impact on existing data:

- No data changes — component structure only

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** No impact — no visual changes

---

### Dependencies:

- Depends on: **[FE] Create Pinia composer store and domain composables** — composables must exist
- Depends on: **[FE] Decompose SocialModal into thin layout shell with independent panels** — ComposerMainArea must use inject pattern

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A for visual changes, but editor must remain responsive
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no visual changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 5: [FE] Extract PostSchedulePicker and platform options as independent components

### Description:

Extract the scheduling UI and platform option components to be self-sufficient via the Pinia store, eliminating their dependence on props from MainComposer.

**Extract `PostSchedulePicker.vue`** (from PostingSchedule.vue, 2,090 lines → target <800 lines):
- Self-contained scheduling component
- Uses `usePostSchedule()` composable — reads/writes to store
- Accepts minimal config: `allowedOptions` (which scheduling modes to show)
- Manages: date/time picker, repeat settings, queue toggle, timezone
- Does NOT receive scheduling state as props — reads from store
- **Reusable:** Can embed in automation scheduling, bulk scheduling, or any context that provides the store

**Simplify platform option components:**

Each platform option component (InstagramOptions 1,111 lines, FacebookOptions 663 lines, YouTubeOptions, GmbOptions, TikTokOptions, etc.) is updated to:
- Use `inject('composerStore')` instead of receiving 10-20 props from MainComposer
- Read platform-specific state from `store.post.platformDetails[platform]`
- Write changes back to store via `store.updatePlatformDetails(platform, data)`
- Manage their own local UI state (dropdowns, toggles) internally
- No emits to parent — all state flows through store

**Refactored ComposerMainArea** (from Story 3) is further reduced:
- No longer passes scheduling props down to PostSchedulePicker
- No longer passes platform-specific props to option components
- Becomes a layout component: renders tabs, active editor, platform options, schedule, footer
- Target: <800 lines (from ~2,000 after Story 3)

**MainComposerFooter simplification:**
- Currently 703 lines, receives many props
- Updated to use `inject('composerStore')` for all state
- Action buttons (Publish, Schedule, Draft, Approve) read state from store
- Submission calls store actions which handle API calls
- Target: <400 lines

---

### Workflow:

1. User toggles between scheduling options (Now, Schedule, Queue, Repeat) — same behavior
2. User picks a date and time for scheduled posts — same date/time picker behavior
3. User configures repeat settings — same repeat options
4. User expands Instagram options to set first comment, collaborators — same behavior
5. User sets YouTube visibility, title, tags — same behavior
6. User clicks Publish/Schedule — same submission flow and result

---

### Acceptance criteria:

- [ ] PostSchedulePicker uses `usePostSchedule()` composable — zero scheduling props from parent
- [ ] PostSchedulePicker is <800 lines (from 2,090)
- [ ] PostSchedulePicker is reusable: works in any context that provides the store
- [ ] InstagramOptions uses `inject('composerStore')` — zero props from parent
- [ ] FacebookOptions uses `inject('composerStore')` — zero props from parent
- [ ] All other platform options (YouTube, GMB, TikTok, Threads, Bluesky) use inject — zero props
- [ ] ComposerMainArea is <800 lines
- [ ] MainComposerFooter uses inject — zero props from parent, <400 lines
- [ ] Scheduling (now, schedule, queue, repeat) works identically
- [ ] All platform-specific options work identically
- [ ] Publish, Schedule, Draft, Approve actions work identically
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — zero UI changes

---

### Impact on existing data:

- No data changes — component structure only

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** No impact

---

### Dependencies:

- Depends on: **[FE] Decompose SocialModal into thin layout shell with independent panels** — ComposerMainArea must exist
- Depends on: **[FE] Create Pinia composer store and domain composables** — store and schedule composable must exist

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A for visual changes, but schedule picker must remain responsive
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no visual changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 6: [FE] Remove frontend validation logic and wire backend validation errors

### Description:

After the backend validation migration (sc-110785) is complete, remove scattered frontend validation logic from the composer module and wire the composer to display backend validation errors using the `useComposerValidation()` composable.

**Prerequisite:** sc-110785 must be deployed — backend must return structured validation errors for all post creation/scheduling requests.

**Remove scattered frontend validation:**
- EditorBox: Remove character count validation logic (keep character counter display as UX hint only)
- EditorCarouselBox: Remove URL validation logic
- CustomThumbnailModal: Remove file size validation logic
- MainComposerFooter: Remove pre-submit validation orchestration
- Platform option components: Remove platform-specific rule enforcement
- Remove all local `validations: { ... }` reactive objects from components

**Wire backend errors:**
- On post submission (publish/schedule/draft), if backend returns 422 with structured errors:
  - Store action `setValidationErrors(response.errors)` populates store
  - `useComposerValidation()` composable makes errors available
  - Components use `getFieldError('fieldName')` to display inline errors
- Error display pattern:
  - Field-level errors: shown inline next to the relevant field (red border + error text below)
  - Non-field errors: shown as toast notification
  - Errors clear when user modifies the relevant field

**Keep UX-only hints (NOT validation, just helpers):**
- Character counter display (shows current/max but doesn't block submission)
- Required field visual indicators (asterisk, subtle border)
- File type indicators in upload dropzone

---

### Workflow:

1. User opens the Composer and writes a post
2. User sees character count updating as they type (UX hint — same as before)
3. User attempts to publish a post with too-long text for Twitter
4. Backend returns a validation error: `{ field: "twitter_message", message: "Message exceeds 280 characters", error_code: "char_limit_exceeded" }`
5. User sees an inline error below the Twitter editor tab highlighting the issue
6. User shortens the text and clicks Publish again — this time it succeeds
7. If a non-field error occurs (e.g., scheduling conflict), user sees a toast notification

---

### Acceptance criteria:

- [ ] All scattered frontend validation logic removed from composer components
- [ ] No local `validations: {}` objects remain in composer components
- [ ] Backend 422 responses populate `composerStore.validation.errors` via store action
- [ ] `useComposerValidation().getFieldError(field)` returns error message for a given field
- [ ] Field-level errors display inline next to the relevant component
- [ ] Non-field errors display as toast notifications
- [ ] Errors clear when user modifies the relevant field
- [ ] Character counter continues to display as UX hint (not blocking validation)
- [ ] Required field visual indicators remain (asterisk, border)
- [ ] Post submission without validation errors works identically
- [ ] All post creation sources (composer, template apply, edit) handle errors consistently
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — error display follows existing inline error patterns used elsewhere in the app

---

### Impact on existing data:

- No data changes — validation logic is client-side
- Backend validation (sc-110785) must be deployed first

---

### Impact on other products:

- **Mobile apps:** No impact — mobile has its own validation
- **Chrome extension:** No impact
- **White-label:** No impact — no visual design changes

---

### Dependencies:

- Depends on: **[BE] Move post creation validations to backend for all creation sources (sc-110785)** — backend must return structured errors
- Depends on: **[FE] Create Pinia composer store and domain composables** — `useComposerValidation()` composable must exist

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A for visual changes
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no visual changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 7: [FE] Cleanup legacy code — delete useComposerHelper, update entry points, finalize migration

### Description:

Final cleanup story to complete the refactor. Delete the legacy utility composable, remove all remaining prop drilling, update entry points, and verify the entire module is clean.

**Delete `useComposerHelper.js`** (1,576 lines):
- All functionality has been extracted into focused composables in earlier stories
- Verify zero imports of `useComposerHelper` remain in any file
- Delete the file

**Remove remaining prop drilling:**
- Audit all components in `composer_v2/` for props that can be replaced with inject/store
- Target: no component receives >15 props
- Remove unused prop definitions from all components
- Remove computed properties that just pass props through

**Update entry points:**
- `src/views/Home.vue` — verify async import still works with refactored SocialModal
- `src/modules/planner_v2/config/routes/planner.js` — verify route component works
- `src/modules/publisher/config/routes/publisher.js` — verify route component works
- Verify edit mode (planId route param → store initialization) works from all entry points

**Clean up old composable wrappers:**
- Review global composables (`useComposerPost`, `useComposerHelpers`, `useComposerMediaUpload`, `useComposerLinkPreview` in `src/composables/`)
- Either migrate their logic into the new domain composables or create thin wrappers
- Remove duplicated logic

**Final verification checklist:**
- `grep -r "EventBus" composer_v2/` returns 0 results
- `grep -r "useComposerHelper" composer_v2/` returns 0 results (except the deleted file itself)
- No component in `composer_v2/` has >15 props
- SocialModal.vue is <500 lines
- All composables in `composer_v2/composables/` are <300 lines
- Full E2E test suite passes

---

### Workflow:

1. User opens Composer from the header "Create Post" button — works identically
2. User opens Composer from Planner by editing a post — works identically
3. User opens Composer from Publisher routes — works identically
4. User completes a full post creation flow (select accounts, write content, add media, schedule, publish) — works identically
5. User completes an edit flow (open existing post, modify, save) — works identically
6. User applies a template — works identically
7. User sends for approval — works identically

---

### Acceptance criteria:

- [ ] `useComposerHelper.js` deleted — file no longer exists
- [ ] Zero imports of `useComposerHelper` in the codebase
- [ ] Zero `EventBus` imports in `composer_v2/`
- [ ] No component in `composer_v2/` has >15 props
- [ ] SocialModal.vue is <500 lines
- [ ] All composables in `composer_v2/composables/` are <300 lines each
- [ ] Composer opens correctly from Home.vue header button
- [ ] Composer opens correctly from Planner edit route
- [ ] Composer opens correctly from Publisher routes
- [ ] New post creation flow works end-to-end
- [ ] Edit post flow works end-to-end
- [ ] Template application flow works
- [ ] Approval workflow works
- [ ] Draft saving works
- [ ] Mobile responsive layout works (drawers, sidebars)
- [ ] All existing E2E tests pass — zero regressions

---

### Mock-ups:

N/A — zero UI changes

---

### Impact on existing data:

- No data changes
- File deletions: `useComposerHelper.js` removed
- File modifications: all composer components cleaned up

---

### Impact on other products:

- **Planner:** Verify edit post route still works after cleanup
- **Publisher:** Verify publisher routes still work
- **Mobile apps:** No impact
- **Chrome extension:** No impact

---

### Dependencies:

- Depends on: **[FE] Create Pinia composer store and domain composables**
- Depends on: **[FE] Replace EventBus with typed events and store actions in composer module**
- Depends on: **[FE] Decompose SocialModal into thin layout shell with independent panels**
- Depends on: **[FE] Decompose EditorBox into independent reusable components**
- Depends on: **[FE] Extract PostSchedulePicker and platform options as independent components**
- Depends on: **[FE] Remove frontend validation logic and wire backend validation errors**
- This is the final story — all other stories must complete first

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — verify mobile drawer layout still works
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, no visual changes
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
