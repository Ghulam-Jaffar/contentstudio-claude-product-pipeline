# Composer v2 Refactor — Workflow Design

---

## 1. Feature Placement

This is an **internal refactoring** — no new user-facing features or UI changes. The Composer modal continues to live in the same places:
- **Entry points:** Composer button (header), Planner edit, Publisher routes, EventBus triggers
- **Route:** `/composer`, `/planner/plan/:planId/edit`
- **Async import:** from `Home.vue`, `planner.js`, `publisher.js`

The refactoring is invisible to users — same UI, same behavior, same API calls. The difference is entirely in code architecture.

---

## 2. Refactoring Flow — Phased Approach

This refactor must be phased to avoid a big-bang rewrite. Each phase is independently shippable and testable.

### Phase 0: Backend Validation Migration (Prerequisite — sc-110785)

Already scoped in the existing story. Must complete first:
1. Backend implements centralized validation for all post creation sources
2. Backend returns structured errors (`{ field, message, error_code }`)
3. Frontend reduces validation to UX-only hints (field highlighting, character counter display)
4. All scattered frontend validation logic identified and tagged for removal

### Phase 1: State Layer Foundation

Replace the 4-way state chaos (Vuex + component refs + EventBus + route params) with a clean architecture:

1. **Create a Pinia `composerStore`** — single source of truth for composer state:
   - Post data (`sharingDetails` unified schema)
   - Account selection state
   - Scheduling state
   - UI state (active tab, sidebar visibility, modal visibility)
   - Validation errors (from backend responses)
   - Loading/processing flags

2. **Create domain composables** that read/write to the store:
   - `useComposerPost()` — post content CRUD, platform-specific details
   - `useAccountSelection()` — account list, selection, filtering
   - `usePostSchedule()` — date/time, repeat, queue
   - `useComposerMedia()` — media upload, gallery, resize
   - `useComposerValidation()` — error state from backend, UX hints
   - `useComposerUI()` — sidebar/modal/tab visibility
   - `usePlatformMetadata()` — icons, names, colors, config (extracted from useComposerHelper)
   - `useMobileDetection()` — viewport tracking (extracted from useComposerHelper)

3. **Replace EventBus** with store actions + watchers:
   - Audit all 48 EventBus files
   - Replace `$emit/$on` with store mutations or typed custom events via `mitt` (lightweight typed event emitter)
   - Remove EventBus imports one file at a time

### Phase 2: SocialModal Decomposition

Reduce SocialModal from 7,110 lines to ~500 lines (thin orchestrator):

1. **SocialModal becomes a layout shell:**
   - Renders the modal wrapper (`CstuModal`)
   - Initializes the Pinia store on open, resets on close
   - Provides store instance via `provide()`
   - Renders 3 slots: left sidebar, main content, right sidebar
   - No business logic — just mount/unmount lifecycle

2. **Extract AccountSelectionPanel** (from AccountSelectionAside):
   - Self-contained component that reads/writes `useAccountSelection()`
   - No props from parent — uses `inject()` to get store
   - Can be used independently (e.g., in automation, bulk scheduling)

3. **Extract ComposerMain** (from MainComposer):
   - Thin wrapper for editor area + tabs
   - Uses `inject()` — no 80+ props
   - Renders editor, platform options, footer

4. **Extract ComposerSidebarPanel** (from ComposerSidebar/ActionsAside):
   - Self-contained — reads/writes store
   - Campaign, labels, comments, tasks, AI assistant

5. **Modal components remain separate** but use `inject()` instead of props:
   - AiCaptionModal, TemplateModal, ApprovalModal, etc.
   - Each reads what it needs from store

### Phase 3: Editor Decomposition

Break EditorBox (2,873 lines) into independent, reusable units:

1. **EditorTextBox** — Pure rich text editing:
   - CstTextArea wrapper
   - Character count display
   - Mention suggestions
   - No media, no AI, no platform options
   - Reusable anywhere text input is needed

2. **EditorMediaManager** — Media attachment management:
   - Gallery display, upload triggers, drag-drop zone
   - Carousel/multi-image management
   - Uses `useComposerMedia()` composable
   - Reusable for any media attachment context

3. **EditorAIActions** — AI button toolbar:
   - Caption generation, improve text, hashtag suggestions
   - Image generation trigger
   - Uses AI composable
   - Reusable for any AI-enhanced text input

4. **EditorToolbar** — Options bar:
   - UTM, hashtag groups, Replug, emoji picker
   - Uses `useComposerPost()` for state

5. **EditorLinkPreview** — Link card preview:
   - Extracts and displays link metadata
   - Uses existing `useComposerLinkPreview()`

### Phase 4: Schedule & Platform Options Decomposition

1. **PostSchedulePicker** — Standalone scheduling component:
   - Date/time selection, repeat configuration
   - Uses `usePostSchedule()` composable
   - Can be used independently (e.g., in automation scheduling)

2. **Platform option components stay separate** but simplified:
   - Each reads platform state from store via `inject()`
   - No props from MainComposer — self-sufficient
   - InstagramOptions, FacebookOptions, YouTubeOptions, etc.

### Phase 5: Composable Cleanup & Finalization

1. **Delete `useComposerHelper`** (1,576 lines) — replaced by focused composables in Phase 1
2. **Migrate remaining EventBus usage** to store/mitt
3. **Remove all prop drilling** — components use inject/store
4. **Update entry points** (Home.vue, planner routes, publisher routes)
5. **Update tests** — component tests can now instantiate with just store, no 80+ prop mocks

---

## 3. Alternative Flows & Edge Cases

| Scenario | Handling |
|---|---|
| Edit existing post (from planner) | Store initialized from API response instead of blank state. Same component tree. |
| Template application | Store action `applyTemplate()` populates post state. Components react via store watchers. |
| Approval workflow | Store tracks approval state. ApprovalModal reads/writes via store. |
| Multi-thread posts (Twitter, Bluesky) | Store supports thread array. EditorThreadedTweetsBox reads from store index. |
| External template open (EventBus) | Replaced by `mitt` typed event → store action. |
| Mobile responsive | `useMobileDetection()` provides responsive state. Layout shell adapts. No duplicate component rendering. |

---

## 4. Key Design Decisions

### Decision 1: Pinia Store vs Provide/Inject Only

**Options:**
- A) **Pinia store** — global-ish store for composer state, accessed via `useComposerStore()`
- B) **Provide/Inject only** — SocialModal provides reactive state, children inject
- C) **Hybrid** — Pinia for persistent state (drafts, account cache), provide/inject for ephemeral modal state

**Recommendation: Option C (Hybrid)**
- Pinia for state that persists across modal open/close (draft auto-save, account selection cache, user preferences)
- Provide/inject for ephemeral state scoped to the modal lifecycle (current post being edited, active tab, validation errors)
- This keeps the store lean while avoiding prop drilling. Components use `inject('composerContext')` for modal state and `useComposerStore()` for persistent state.

### Decision 2: EventBus Replacement

**Options:**
- A) **Remove entirely** — all communication through store
- B) **Replace with `mitt`** — typed, lightweight event emitter
- C) **Replace with Vue's built-in `emits` + provide/inject**

**Recommendation: Option B (mitt) for cross-module events + Option C for parent-child**
- Parent-child communication: use Vue `emits` (standard pattern)
- Cross-module communication (e.g., planner → composer): use `mitt` with a typed event map
- Within composer: prefer store actions over events

### Decision 3: Refactor Strategy

**Options:**
- A) **Big-bang rewrite** — rebuild from scratch alongside old code, swap when ready
- B) **Strangler fig** — wrap old components, replace one at a time
- C) **Inside-out** — start with state layer, then decompose top-down

**Recommendation: Option C (Inside-out)**
- Phase 1 (state layer) can be built alongside existing code
- Once state layer works, components can be migrated one at a time
- Each phase is independently shippable — no "big switch" moment
- Old and new components can coexist during migration

---

## 5. Integration with Existing Features

| Feature | Integration |
|---|---|
| **Planner** | Opens composer via route. Store initialized from plan data. No changes to planner code needed after refactor. |
| **Automation / AI Scheduling** | Uses same API. Backend validation (sc-110785) ensures consistency. Frontend components become reusable for automation UI. |
| **Zapier / Integrations** | No impact — they hit API directly. Backend validation covers them. |
| **Media Library** | UploadMediaModal becomes a standalone component. `useComposerMedia()` provides clean interface. |
| **AI Features** | AiCaptionModal, ImageGeneratorModal become standalone. Can be reused in other contexts. |
| **Approval Workflow** | ApprovalModal reads/writes store. Decoupled from SocialModal lifecycle. |
| **Chrome Extension** | Uses composer API. No frontend component changes needed. |
| **Mobile Apps** | Not impacted — mobile has its own composer. |

---

## 6. Scope: v1 (This Epic) vs v2 (Future)

### v1 — This Epic (5 Phases)
- ✅ Phase 0: Backend validation migration (sc-110785 — existing story)
- ✅ Phase 1: State layer (Pinia store + domain composables + EventBus replacement)
- ✅ Phase 2: SocialModal decomposition (layout shell + independent panels)
- ✅ Phase 3: Editor decomposition (5 independent components)
- ✅ Phase 4: Schedule & platform options decomposition
- ✅ Phase 5: Composable cleanup & finalization

### v2 — Future
- ⏳ Post preview system rebuild (17 preview components have their own debt)
- ⏳ TypeScript migration for composer module
- ⏳ Unit test coverage to 80%+ (currently likely <10%)
- ⏳ Performance profiling and optimization (virtual scrolling for long thread editors)
- ⏳ Design system alignment (replace custom CSS with design library components)
