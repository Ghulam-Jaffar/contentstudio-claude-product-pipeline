# Research — Schedule via Content Category Dropdown

## Current State

Users can "Add to Composer" or "Save as Draft" from two AI tools surfaces:
- **Video Clips modal** (`ClipPreviewModal.vue`, `ClipCard.vue`, `ClipsCardView.vue`, `ClipsListView.vue`, `StepVideoResults.vue`)
- **AI Chat action buttons** (`BotChatTemplate.vue`)

There is no way to directly schedule content to a Content Category slot from these surfaces without going through the full Composer.

### Existing APIs (no new BE endpoints needed)

| Endpoint | Purpose |
|---|---|
| `POST /planner/api/planner/createDraftPost` | Creates a draft plan (`{workspace_id, caption, images?, videos?}`) → returns the created plan |
| `POST /settings/categories/slots/next_slot` | Schedules a plan to next available slot (`{category_id, workspace_id, plan_id}`) |
| `POST /settings/api/categories/fetch` | Fetches all content categories for the workspace |

### Key files — content categories

- `src/modules/setting/store/states/content-categories.js`
  - `getContentCategoryList` getter → `state.categories_list` (array of `{_id, name, color, slots_count}`)
  - `fetchContentCategories` action → `POST /settings/api/categories/fetch`
  - `nextAvailableSlot(categoryId)` Vuex action — reads `plan_id` from `getPublishSelection.plan_id` (Composer context only; **not usable directly** from AI tools)

### Key files — surfaces to modify

| File | Current Actions |
|---|---|
| `ClipPreviewModal.vue:109-130` | "Add to Composer" (`$emit('add-to-composer', reel)`), "Save as Draft" (`$emit('save-draft', reel)`) |
| `ClipCard.vue:123,178` | `$emit('save-draft')`, `$emit('add-to-composer')` via dropdown menu |
| `ClipsCardView.vue:14,44` | Forwards `save-draft`, `add-to-composer` emits from `ClipCard` |
| `ClipsListView.vue:175,229` | Inline dropdown with same actions |
| `StepVideoResults.vue:234-246` | `handleSaveDraftReel(reel)` — calls `createDraftPost` from `useAIChat` |
| `BotChatTemplate.vue:381-405, 734-754` | "Add to Composer" (`handleAddImageToComposer`), "Save as Draft" (`handleAddToDraft`) |

### Scheduling flow (two-step)

The new feature will:
1. Call `POST /planner/api/planner/createDraftPost` → receive created plan with `_id`
2. Call `POST /settings/categories/slots/next_slot` with `{category_id, workspace_id, plan_id}` → schedules the draft to the next available slot in the chosen category

The existing `nextAvailableSlot` Vuex action reads `plan_id` from composer state and cannot be reused. The new composable method will call the API directly.

### `createDraftPost` composable (`useAIChat.js:1264-1337`)

- Accepts `(caption, images, videos)`
- Calls `POST createDraftPostURL`, shows success/error toast
- Used by `StepVideoResults.vue::handleSaveDraftReel` for video clips
- Returns `{ success: true, data }` where `data.data` contains the created plan

### Design reference (from user screenshots)

- Dropdown panel styled like the "Templates" popup: title row with `+` and `?` icons, search input, scrollable list
- List items: colored bullet (matching `category.color` → e.g. `color_1` CSS class) + category name, truncated with tooltip if long
- Empty state: illustration + title + description + CTA to create a category
- On selection: show inline loader → call APIs → show success/error toast

## What Needs to Change

- **New reusable component**: `ContentCategoryDropdown.vue` — the dropdown panel (category list, search, `+`, `?` icons, empty state, loading state)
- **New composable method** in `useAIChat.js` (or a new composable): `scheduleToContentCategory(reel, categoryId)` — handles two-step create-draft → schedule-to-slot flow
- **ClipPreviewModal.vue**: Add a new action button ("Schedule via Content Category") that emits `schedule-category` with the reel; show `ContentCategoryDropdown`
- **ClipCard.vue**: Add new dropdown menu item; emit `schedule-category`
- **ClipsCardView.vue / ClipsListView.vue**: Forward the `schedule-category` emit up to `StepVideoResults`
- **StepVideoResults.vue**: Add `handleScheduleCategoryReel(reel, categoryId)` handler using the new composable method
- **BotChatTemplate.vue**: Add "Schedule via Content Category" button alongside "Add to Composer" and "Save as Draft" for both image/video and text responses; handle the two-step scheduling inline

## Files Involved

```
src/modules/AI-tools/video-clips/components/ClipPreviewModal.vue
src/modules/AI-tools/video-clips/components/ClipCard.vue
src/modules/AI-tools/video-clips/components/ClipsCardView.vue
src/modules/AI-tools/video-clips/components/ClipsListView.vue
src/modules/AI-tools/video-clips/components/StepVideoResults.vue
src/modules/AI-tools/BotChatTemplate.vue
src/composables/useAIChat.js
src/components/ContentCategoryDropdown.vue  (new)
```

No backend changes required — existing endpoints cover the full flow.
