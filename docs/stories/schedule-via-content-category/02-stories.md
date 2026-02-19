# Stories — Schedule via Content Category

## [FE] Add "Schedule via Content Category" dropdown to AI tools surfaces

---

### Description:

ContentStudio users often generate content in AI tools (AI Studio chat, Video Clips, AI Library) and want to immediately slot it into their content calendar without going through the full Composer. Today they can only "Add to Composer" (which opens the full editor) or "Save as Draft" (which creates an unscheduled draft). There is no way to instantly schedule AI-generated content to a Content Category queue.

This story introduces a **reusable `ContentCategoryDropdown` component** — a lightweight popup panel that displays the workspace's content categories with colored bullets, a search field, and appropriate empty/loading states. When a user selects a category, their content is automatically created as a post and scheduled to that category's next available time slot.

**This component is designed for reuse across all AI tool surfaces:**
- AI Studio chat (`BotChatTemplate.vue`) — for both text and video/image AI responses
- Video Clips modal (`ClipPreviewModal.vue`, `ClipCard.vue`, `ClipsCardView.vue`, `ClipsListView.vue`)
- Upcoming Bulk Scheduling modal
- Any future AI tool surface that produces schedulable content

**Scheduling flow (two steps using existing APIs):**
1. Create a draft post: `POST /planner/api/planner/createDraftPost` with content (caption + video/image media)
2. Schedule to the selected category's next slot: `POST /settings/categories/slots/next_slot` with `{category_id, workspace_id, plan_id}` (plan_id from step 1)

**Key files:**
- New component: `src/components/ContentCategoryDropdown.vue` (or `src/modules/AI-tools/components/ContentCategoryDropdown.vue`)
- `src/composables/useAIChat.js` — add `scheduleToContentCategory(content, categoryId)` method
- `src/modules/AI-tools/video-clips/components/StepVideoResults.vue` — add `handleScheduleCategoryReel(reel, categoryId)`
- `src/modules/AI-tools/BotChatTemplate.vue` — add trigger button + dropdown integration
- `src/modules/AI-tools/video-clips/components/ClipPreviewModal.vue` — add action button
- `src/modules/AI-tools/video-clips/components/ClipCard.vue`, `ClipsCardView.vue`, `ClipsListView.vue` — forward new emit
- Content categories loaded via existing `getContentCategoryList` Vuex getter + `fetchContentCategories` action

---

### Workflow:

**Video Clips surface:**

1. User is in the AI Studio Video Clips tool and has generated a set of video clips.
2. User hovers over a clip card and sees the actions dropdown (or opens the clip preview modal).
3. User sees a new "Schedule via Content Category" option alongside the existing "Add to Composer" and "Save as Draft" options.
4. User clicks "Schedule via Content Category" — a dropdown panel opens anchored to the button.
5. The dropdown shows the workspace's content categories as a scrollable list. Each category has a colored bullet and its name.
6. User types in the search field to filter by category name (e.g., typing "social" filters to matching categories).
7. User clicks a category — the dropdown closes, a loading spinner appears on the trigger button.
8. After ~1–2 seconds, a success toast appears: **"Video scheduled to [Category Name] — added to your next available slot."**
9. If an error occurs, an error toast appears: **"Couldn't schedule the video. Please try again."**

**AI Studio Chat surface:**

1. User is in AI Studio Chat and an AI response has generated content (text caption, image, or video).
2. Under the AI response, user sees the existing action buttons ("Add to Composer", "Save as Draft") and a new "Schedule via Content Category" button.
3. User clicks "Schedule via Content Category" — the `ContentCategoryDropdown` panel opens.
4. Flow from step 5 above continues identically.

**No categories exist (empty state):**

1. User clicks "Schedule via Content Category".
2. The dropdown opens and shows the empty state.
3. User sees a "+" icon in the dropdown header — clicking it opens a confirmation dialog.
4. The confirmation dialog tells the user they will be redirected to the Content Categories settings page.
5. User clicks "Go to Settings" — they are navigated to the Content Categories settings page (Settings → Content Categories).

---

### Acceptance criteria:

**Dropdown trigger:**
- [ ] "Schedule via Content Category" action is visible on all target surfaces: clip card dropdown menu, clip preview modal action buttons, AI Chat response action buttons
- [ ] Trigger button shows a tooltip: "Schedule this content to a content category queue. Pick a category and it will be added to the next available time slot."
- [ ] Clicking the trigger opens the `ContentCategoryDropdown` panel; clicking again (or clicking outside) closes it
- [ ] Only one dropdown is open at a time — opening a new one closes any open one

**Dropdown panel — populated state:**
- [ ] Dropdown header shows the title **"Content Categories"**
- [ ] Header includes a `?` (question mark) icon that links to the ContentStudio help doc for Content Categories (opens in a new tab)
- [ ] Header includes a `+` (plus) icon that triggers the "redirect to settings" confirmation dialog
- [ ] A search input is shown below the header with placeholder text: **"Search categories..."**
- [ ] Categories are listed below the search field, each with a colored bullet matching the category's assigned color and the category name
- [ ] Category names longer than ~28 characters are truncated with an ellipsis and show the full name in a tooltip on hover
- [ ] Search filters the list in real-time as the user types (client-side filter on category name)
- [ ] If search returns no matches, a short inline message is shown: **"No categories match your search."**
- [ ] The list scrolls if there are more categories than fit in the panel

**Dropdown panel — empty state (no categories configured):**
- [ ] When the workspace has zero content categories, the dropdown shows the empty state instead of the list
- [ ] Empty state includes an illustration, the headline **"No content categories yet"**, the subtext **"Create content categories to organize your posts into themed queues with time slots — like 'Motivational Mondays' or 'Product Updates'."**, and a CTA button: **"Create a Category"**
- [ ] Clicking "Create a Category" opens the confirmation dialog (same as clicking the `+` icon)

**Redirect confirmation dialog:**
- [ ] Dialog title: **"You'll be redirected to Settings"**
- [ ] Dialog body: **"To create or manage content categories, you'll need to go to the Content Categories settings page. Any unsaved changes here will remain."**
- [ ] Primary CTA: **"Go to Settings"** — navigates to the Content Categories page (Settings → Content Categories) in the same tab
- [ ] Secondary CTA: **"Cancel"** — closes the dialog and returns to the dropdown

**Scheduling flow:**
- [ ] Selecting a category immediately closes the dropdown and shows a loading state on the trigger button
- [ ] The content (video clip or AI-generated text/image) is created as a draft post via `POST /planner/api/planner/createDraftPost`
- [ ] The draft is then scheduled to the selected category's next available slot via `POST /settings/categories/slots/next_slot`
- [ ] On success, a toast notification is shown: **"Scheduled to [Category Name] — added to the next available slot."**
- [ ] On failure (API error at either step), a toast notification is shown: **"Couldn't schedule the content. Please try again."**
- [ ] Loading state is cleared after success or failure (button returns to normal state)
- [ ] While loading, the trigger button is disabled and shows a spinner icon

**Component reusability:**
- [ ] The `ContentCategoryDropdown` component accepts the content payload as a prop and emits a `schedule` event with `{ categoryId, categoryName }` — the parent handles the API call
- [ ] The component is self-contained: it fetches/uses the category list internally via the Vuex store

---

### UI Copy Reference

#### Trigger button tooltip
> "Schedule this content to a content category queue. Pick a category and it will be added to the next available time slot."

#### Dropdown panel
| Element | Copy |
|---|---|
| Panel title | Content Categories |
| `?` icon tooltip | "Learn more about content categories" |
| `+` icon tooltip | "Create a new content category" |
| Search placeholder | Search categories... |
| No search results | No categories match your search. |

#### Empty state
| Element | Copy |
|---|---|
| Headline | No content categories yet |
| Subtext | Create content categories to organize your posts into themed queues with time slots — like "Motivational Mondays" or "Product Updates". |
| CTA | Create a Category |

#### Redirect confirmation dialog
| Element | Copy |
|---|---|
| Title | You'll be redirected to Settings |
| Body | To create or manage content categories, you'll need to go to the Content Categories settings page. Any unsaved changes here will remain. |
| Primary CTA | Go to Settings |
| Secondary CTA | Cancel |

#### Toast notifications
| State | Copy |
|---|---|
| Success | Scheduled to [Category Name] — added to the next available slot. |
| Error | Couldn't schedule the content. Please try again. |

---

### Mock-ups:

Screenshots provided by the user (see story attachment or Figma):
- **Populated dropdown** — Templates-style panel with title, `+` icon, search bar, `?` icon, and a list of categories with colored bullets
- **Category list** — colored bullets (purple, red, green, dark, etc.) + category name
- **Empty state** — illustration + headline + subtext + CTA

Colored bullets use the category's `color` field value (e.g., `color_1`, `color_2`, etc.) — existing CSS classes in the design system already define these colors (used in the Content Categories settings page).

---

### Impact on existing data:

None. This feature uses existing plan and category data structures. Creating a post via `createDraftPost` creates a standard plan document in MongoDB. Scheduling via `nextAvailableSlotCategoryURL` sets the existing `content_category_id` and `execution_time` fields on the plan. No schema changes required.

---

### Impact on other products:

- **Mobile apps:** Not applicable — this feature is in AI tools, which are web-only. No mobile stories needed.
- **Chrome extension:** No impact.
- **White-label:** The `ContentCategoryDropdown` uses theme-aware Tailwind classes (`text-primary-cs-500`, `bg-primary-cs-50`, `border-primary-cs-200`) to respect white-label color theming. Category bullet colors come from existing category CSS classes and are not affected by white-label theming.

---

### Dependencies:

None. Existing APIs (`createDraftPost`, `nextAvailableSlotCategoryURL`, `fetchCategoriesURL`) are already in production.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, AI tools are web-only
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
