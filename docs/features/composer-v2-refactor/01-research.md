# Composer v2 Refactor — Research

## Executive Summary

The Composer v2 module (`src/modules/composer_v2/`) is a **monolithic, tightly-coupled system with severe architectural debt**. It totals ~51,000 lines across 119 files with a 7,110-line god component (`SocialModal.vue`), 80+ prop drilling chains, composable spaghetti, scattered validation, and EventBus anti-patterns. No component can be reused independently outside the module.

An existing backend story exists for migrating validations: **[Move post creation validations to backend for all creation sources — sc-110785](https://app.shortcut.com/contentstudio-team/story/110785)** (currently in epic 107951, state: queued).

---

## Current Architecture Problems

### 1. PROP DRILLING (Severity: CRITICAL)

**MainComposer receives 80+ props** from SocialModal. **EditorBox receives 50+ props** from MainComposer. Props are drilled 5 levels deep:

```
SocialModal (root) → MainComposer (80+ props) → EditorBox (50+ props) → EditorMediaBox (20+) → Platform-specific (10+)
```

Anti-patterns found:
- Entire config objects drilled as-is (`sharingDetails`, `toolbar`, `linkedinOptions`)
- Function props instead of events/provide (`applyHashtag()`, `approverModalValidation()`, `clearState()`)
- Duplicate flag props per platform (`facebookGroupSelected`, `facebookPageSelected`, `instagramShareToStory`, etc.)
- Separate error arrays per platform (`gmbErrors`, `youtubeErrors`, `fbCarouselErrors`, `socialErrors`)

### 2. GOD COMPONENTS (Severity: CRITICAL)

| Component | Lines | Responsibilities |
|---|---|---|
| SocialModal.vue | 7,110 | 21+ — orchestration, account selection, sidebar toggling, modal management, data fetching, template management, approval workflow, validation, event bus bridging, file uploads, scheduling, link previews, AI captions, image resizing, character counting |
| MainComposer.vue | 4,072 | 15+ — editor tabs, platform options, scheduling, footer actions, preview panel, template/label/campaign attachments |
| EditorBox.vue | 2,873 | 8+ — rich text, AI buttons, media attachment, paste/drag-drop, link previews, media uploads, platform options |
| PostingSchedule.vue | 2,090 | Date/time picking, repeat settings, plan updates, timezone handling |
| AccountSelectionAside.vue | 1,935 | Search/filter, checkbox selection, content categories, platform grouping |

### 3. COMPOSABLE SPAGHETTI (Severity: HIGH)

9 composables, dominated by `useComposerHelper` (1,576 lines) which is a utility dump:
- Mobile viewport detection
- Platform icon/image imports
- Date formatting
- Draft/template state refs
- Modal control (Nitro modals)
- API utils and store dispatches
- 10+ unrelated responsibilities in one file

Other issues:
- Composables access Vuex store directly (no abstraction)
- Lifecycle hooks inside composables (violates best practices)
- Reference counting for event listeners (band-aid for cleanup issues)

### 4. STATE MANAGEMENT CHAOS (Severity: HIGH)

State lives in 4 different places simultaneously:
- **Vuex store** — accounts, workspace, user, permissions
- **Component local state** — scattered `ref()`/`reactive()` in each component
- **EventBus** — 48 files use `EventBus.$emit/$on` for cross-component communication
- **Route params** — plan ID for editing

Post state is a deeply nested `sharingDetails` object with platform-specific sub-objects passed through entire prop chains. No single schema definition.

### 5. VALIDATION SCATTERED (Severity: HIGH)

Validations are spread across multiple components with no shared schema:
- EditorBox: character count validation
- EditorCarouselBox: URL validation
- CustomThumbnailModal: file size validation
- Platform-specific rules hardcoded in components
- `socialIntegrationsConfigurations` from integration module provides limits
- No validation middleware — validation happens on submit, errors scattered

### 6. NO COMPONENT REUSABILITY (Severity: HIGH)

| Component | Reusable? | Why Not |
|---|---|---|
| EditorBox | ❌ | Needs 50+ props, coupled to sharingDetails shape |
| PostingSchedule | ❌ | Needs specific prop shapes, coupled to MainComposer |
| AccountSelectionAside | ❌ | Hardcoded to modal workflow, 1,935 lines of coupled logic |
| Platform previews | ❌ | Rely on specific data shapes from parent chain |

### 7. EVENT BUS ANTI-PATTERN (Severity: MEDIUM)

48 files use `EventBus.$emit/$on`:
- No type safety
- Silent failures on misnamed events
- Memory leak risk
- No centralized event registry
- Cannot trace data flow

Limited `provide/inject` usage exists (only for `root`, `socialModal`, `isTemplateMode`).

---

## Component Tree (Simplified)

```
SocialModal (7,110 lines)
├── AccountSelectionAside (1,935) — left sidebar
├── MainComposer (4,072) — main content
│   ├── EditorBox (2,873) — text + media
│   │   ├── CstTextArea — rich text editor
│   │   ├── EditorOptions — UTM, hashtag, replug
│   │   ├── EditorMediaBox (1,967) — media gallery
│   │   ├── EditorFirstCommentBox
│   │   ├── EditorThreadedTweetsBox
│   │   ├── EditorCarouselBox (714)
│   │   └── EditorBlueskyBox, LinkedInCarouselBox
│   ├── Platform Options (Instagram 1,111 / Facebook 663 / YouTube / GMB / TikTok / etc.)
│   ├── PostingSchedule (2,090)
│   ├── MainComposerFooter (703) — action buttons
│   └── PostPreview (17 platform previews)
├── ComposerSidebar (788) — right sidebar
│   ├── AssistantMain (750) — AI + media search
│   ├── Comments (985)
│   ├── Tasks (575)
│   └── Labels, Members, Activities
└── 13+ Modals (AI Caption 1,050 / Template / Approval / etc.)
```

---

## Existing Backend Story (sc-110785)

**Title:** Move post creation validations to backend for all creation sources
**Epic:** 107951 | **State:** Queued

**Key acceptance criteria already defined:**
- All frontend validations implemented in backend as source of truth
- Backend validations for ALL entry points: composer, AI smart scheduling, Zapier, integrations
- Centralized validation logic (no duplication)
- Structured error responses (field, message, error_code)
- Frontend reduced to UX-only checks (required field highlighting)
- Backward compatible with existing integrations

This is a prerequisite for the frontend refactor — once validations move to backend, the frontend can remove scattered validation logic and simplify components significantly.

---

## Refactoring Best Practices (Vue 3)

Based on industry research:

1. **Use Provide/Inject for component-tree-local state** — 20x faster than Pinia for ref changes, perfect for modal/form scoped data. No more 80+ prop chains.
2. **Reserve Pinia for truly global state** (user auth, workspace settings). Overkill for form data.
3. **Composables for reusable logic** (validation, formatting, API calls) — NOT for state management.
4. **Split by feature domain** — `useFormValidation()`, `useMediaManagement()`, `useScheduling()` — not by layer.
5. **Keep composables pure** — No DOM manipulation, no lifecycle hooks, return clean state + action functions.
6. **Break template into sub-components only where reusable** — Avoid component instance overhead for non-reusable splits.

---

## Files Involved

### Top 10 Largest (Will Be Refactored)
1. `views/SocialModal.vue` — 7,110 lines
2. `components/MainComposer.vue` — 4,072 lines
3. `components/EditorBox.vue` — 2,873 lines
4. `components/PostingSchedule.vue` — 2,090 lines
5. `components/EditorMediaBox.vue` — 1,967 lines
6. `components/AccountSelectionAside.vue` — 1,935 lines
7. `components/InstagramOptions.vue` — 1,111 lines
8. `components/AiCaptionModal.vue` — 1,050 lines
9. `components/Comments.vue` — 985 lines
10. `composables/useComposerHelper.js` — 1,576 lines

### All Composables (Will Be Restructured)
- `useComposerHelper.js` — 1,576 lines (utility dump → split into 5+ focused composables)
- `useImageResize.js` — 923 lines
- `useImageGeneration.js` — 910 lines
- `usePostSchedulePlans.js` — 415 lines
- `useTemplates.js` — 194 lines
- `useDeviceManagement.js` — 188 lines
- `useTimeRecommendation.js` — 160 lines
- `usePublicationFolder.js` — 120 lines
- `useFetchMembers.js` — 65 lines

### Entry Points (Will Need Updated Imports)
- `src/views/Home.vue` — async imports SocialModal
- `src/modules/planner_v2/config/routes/planner.js` — route component
- `src/modules/publisher/config/routes/publisher.js` — route component
