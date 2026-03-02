# Stories: AI-Powered Variations in Evergreen Automation

Epic: https://app.shortcut.com/contentstudio-team/epic/112900
Iteration: 09 March - 20 March 2026 (ID: 111909)

---

## Story 1: [BE] Add AI variation generation API and background job for Evergreen automation

### Description:
As a ContentStudio user running an Evergreen automation, I want the system to generate AI caption variations for my posts in the background, so that I get multiple variations without blocking my browser session.

---

### Workflow:

1. User triggers AI variation generation from the Compose Posts step (single post or bulk) via the frontend
2. Frontend calls the relevant endpoint with: automation ID, post index/indices, variation count, image generation flag, custom instructions (optional)
3. Backend validates AI text credit balance — if 0, returns an error; no job is dispatched
4. If image generation is requested, backend validates AI image credit balance; if 0 image credits, proceeds with text-only (does not block the full request)
5. Backend dispatches a background async job for each post to be processed
6. Job processes each post: if image-only (no caption), analyses the image and generates a caption first, saves it to Variation 1; then generates N caption variations
7. Job deducts AI text credits after each variation is successfully generated (not upfront)
8. If image generation is on, job generates one AI image per variation and deducts 1 AI image credit per image
9. On completion, job appends generated variations to the automation draft and emits a completion event (websocket push or polling endpoint)
10. If credits run out mid-generation, job saves completed variations, skips the rest, and includes partial completion metadata in the event payload

---

### Acceptance criteria:

- [ ] `POST /triggerVariationGeneration` accepts: `automation_id`, `post_index`, `variation_count` (1–10), `generate_images` (bool), `custom_instructions` (optional string)
- [ ] `POST /triggerBulkVariationGeneration` accepts the same fields without `post_index` (applies to all posts); supports parallel processing where infrastructure allows
- [ ] Both endpoints validate AI text credits before dispatching — return 402 with a translatable error key if 0 credits available
- [ ] If `generate_images: true` and image credits = 0, generation proceeds text-only; response indicates image generation was skipped
- [ ] For image-only posts, the job calls the image-to-caption AI service before generating variations; the generated caption is stored in the post's Variation 1 and saved to the automation draft
- [ ] Variations are appended to the automation draft — existing variations are never replaced or removed
- [ ] AI text credits deducted after each variation is successfully generated (~60 words per credit unit, based on actual generated length)
- [ ] AI image credits deducted after each image variation is successfully generated (1 credit per image)
- [ ] If credits run out mid-generation, job saves all completed variations and records `variations_generated` vs. `variations_requested` in the completion payload
- [ ] On job completion, the automation draft is updated and a completion event is emitted; event payload includes: `post_index` (or all post indices for bulk), `variations_generated`, `variations_requested`, `credits_used`
- [ ] Each generated variation includes `ai_generated: true` flag in its data structure
- [ ] Each variation also includes `ai_edited: false`; backend updates this to `true` when the user edits the variation's caption text via the existing edit endpoint
- [ ] New API endpoint URLs added to `src/config/api-utils.js` on the frontend side
- [ ] All API error responses use translatable message keys, not hardcoded English strings

---

### Mock-ups:

N/A — backend story

---

### Impact on existing data:

- New fields on variation objects: `ai_generated` (boolean), `ai_edited` (boolean)
- No changes to existing variation structure, draft save logic, or automation schema beyond these two new fields

---

### Impact on other products:

- Web only — automation wizard is not available on mobile apps
- No Chrome extension impact
- Credit deduction integrates with the existing AI credit system; no new billing tables required

---

### Dependencies:

- Image-to-caption AI service (same service used in Composer)
- Existing background job infrastructure
- Existing AI credit tracking and deduction system
- Existing `saveDraftEvergreenAutomation` mechanism

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend story
- [ ] Multilingual support — all API error messages use translatable keys, not hardcoded strings
- [ ] UI theming support — N/A, backend story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---

## Story 2: [FE] Add AI generate buttons, variation badges, and loading states to Compose Posts step

### Description:
As a user composing posts in the Evergreen automation wizard, I want to see AI generation controls directly on the post cards and receive clear feedback while generation runs, so I can trigger and monitor AI variations without leaving the step or losing my work.

---

### Workflow:

1. User opens Step 2 (Compose Posts) of the Evergreen automation wizard
2. User sees a "✨ Generate variations for all" button above the post list, right-aligned
3. Each post card header shows (left to right): post name, variation count badge, "+ Add variation" button (existing), "✨ Generate with AI" button (new), delete icon (existing)
4. User clicks "✨ Generate with AI" on a post → Generate Variations modal opens (see: **[FE] Build Generate Variations modal with per-post AI generation and credit state handling**)
5. User clicks "✨ Generate variations for all" → Bulk Generate modal opens (see: **[FE] Build Bulk Generate Variations modal for all posts in Evergreen automation**)
6. Once generation is triggered, a loading card appears at the bottom of the relevant post card during generation; removed when complete
7. Generated variations appear as regular variation cards with an "AI" badge; variation count badge updates
8. If user edits a variation's caption text, the AI badge is removed from that variation automatically
9. If any generation is in progress and the user tries to navigate away (Previous, Next, sidebar), a non-blocking banner appears at the top of the page; user can navigate freely

---

### Acceptance criteria:

- [ ] "✨ Generate variations for all" button appears above the post list, right-aligned, always visible on Step 2
- [ ] Each post card header contains: post name, variation count badge, "+ Add variation" (existing, unchanged), "✨ Generate with AI" (new), delete icon (existing, unchanged)
- [ ] Variation count badge label: "{N} Variation" (singular) or "{N} Variations" (plural); updates live as variations are added or removed
- [ ] Variation count badge styled as a pill using `bg-primary-cs-500` and white text
- [ ] "✨ Generate with AI" button shown on all posts regardless of whether they have a caption or only an image
- [ ] Clicking "✨ Generate with AI" opens the per-post Generate Variations modal
- [ ] Clicking "✨ Generate variations for all" opens the Bulk Generate Variations modal
- [ ] Loading card appears at the bottom of the relevant post card when generation is triggered; spinner icon visible
- [ ] Loading card text for image-only posts: "Analysing image & generating caption first, then creating {N} variation(s)…"
- [ ] Loading card text for posts with captions: "Generating {N} variation(s)…"
- [ ] Loading card styling: dashed border (`border-primary-cs-200`), `bg-primary-cs-50` background, no solid border
- [ ] Loading card is removed when generation completes; new variation cards appear in its place
- [ ] AI-generated variations render with identical card styling to manual variations, plus a small "AI" pill badge (primary-cs gradient) in the top-right of the card header, positioned before edit/delete icons
- [ ] AI badge is removed automatically when the user edits the variation's caption text; it is not manually removable by the user
- [ ] Non-blocking navigation warning banner appears at the top of the page when any generation job is in progress and the user attempts to navigate away
- [ ] Banner copy: "Variations are still being generated. You can leave — your results will be saved automatically and will be ready when you return."
- [ ] Banner does not block navigation — user can proceed freely after it appears
- [ ] Banner auto-dismisses when all in-progress generation jobs complete or when the user navigates away
- [ ] All new strings use i18n keys in the `automation.evergreen_automation` namespace, with translations added for all 7 locales (en, fr, de, es, it, el, zh)
- [ ] No hardcoded hex values — all colors via `primary-cs-*` and `gray-*` Tailwind classes

---

### Mock-ups:

N/A — no designs provided; implementation follows spec in PRD.

---

### Impact on existing data:

- Reads new `ai_generated` and `ai_edited` fields on variation objects (added by backend) to determine whether to show the AI badge
- On variation edit, sends an update to mark `ai_edited: true` on the backend — this removes the AI badge on the frontend

---

### Impact on other products:

- Web only — Evergreen automation wizard is not available on mobile apps; no Chrome extension impact

---

### Dependencies:

- **[BE] Add AI variation generation API and background job for Evergreen automation** — must be complete before frontend can wire up API calls and receive completion events
- **[FE] Build Generate Variations modal with per-post AI generation and credit state handling**
- **[FE] Build Bulk Generate Variations modal for all posts in Evergreen automation**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — Evergreen wizard is desktop-only; N/A
- [ ] Multilingual support — all new strings added to all 7 locale files
- [ ] UI theming support — all colors via `primary-cs-*` CSS variable-backed classes; no hardcoded hex
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---

## Story 3: [FE] Build Generate Variations modal with per-post AI generation and credit state handling

### Description:
As a user composing posts in the Evergreen automation wizard, I want a modal to configure and trigger AI variation generation for a single post, so I can set the variation count, optionally generate images, and give the AI custom guidance before it runs.

---

### Workflow:

1. User clicks "✨ Generate with AI" on a post card — modal opens
2. Modal title: "Generate Variations"; subtitle: "Post {N}" (e.g., "Post 2")
3. **Source field** (read-only chip, auto-detected from post content):
   - Image-only post → chip: "🖼 Image (caption will be generated first)" + hint: "This post has no caption yet. AI will analyse the image, generate a caption, then create variations from it."
   - Caption only → chip: "💬 Post caption" + hint: "AI will use the existing post caption as the base for all variations."
   - Caption + link → chip: "🔗 Link content + post caption" + hint: "AI will use both the linked article and the existing caption as context for generating variations."
4. **How many variations?** — stepper (− / number input / +), min 1, max 10, default 3; hint below: "Max 10 per post. Each variation consumes AI text credits."
5. **Generate images** toggle row (off by default):
   - Off: label "Generate images"; sub-text "Uses 1 AI image credit per variation. You have {N} AI image credits remaining."
   - On: label "Generate images ✓"; row background changes to a light primary-cs tint
   - Image credits = 0: toggle disabled at 50% opacity; sub-text: "You've used all your AI image credits. [Upgrade your plan] to get more." (upgrade link styled in warning color)
6. **Custom instructions** textarea (optional): placeholder "e.g. Keep under 280 chars, use 2 hashtags, match brand tone…"
7. Footer: "Cancel" (secondary) | "✨ Generate" (AI gradient)
8. User clicks "✨ Generate" → modal closes; generation begins; loading card appears on post card

**Zero text credits state** (when user has 0 AI text credits):
- Count stepper section hidden
- Image toggle section hidden
- Amber alert shown: "You've run out of AI text credits. Variations can't be generated right now. [Upgrade your plan] to get more credits."
- "✨ Generate" button disabled (opacity 45%, not clickable)
- Upgrade link opens billing/upgrade flow

---

### Acceptance criteria:

- [ ] Modal opens when user clicks "✨ Generate with AI" on any post card
- [ ] Modal title: "Generate Variations"; subtitle reflects the correct post number (e.g., "Post 2")
- [ ] Source chip is auto-detected from the post's content type and is read-only; correct chip label and hint text shown for each post type (image-only, caption-only, caption+link)
- [ ] Variation count stepper: default 3, minimum 1, maximum 10; clamped at boundaries; cannot submit values outside range
- [ ] Hint below stepper: "Max 10 per post. Each variation consumes AI text credits."
- [ ] Image toggle is off by default; sub-text displays the actual remaining AI image credit count
- [ ] Toggling images on changes the row background to a light `bg-primary-cs-50` tint
- [ ] When image credits = 0: toggle is disabled at 50% opacity; sub-text shows "You've used all your AI image credits. [Upgrade your plan] to get more." with warning-color upgrade link
- [ ] Custom instructions field is optional; accepts free-form text; placeholder matches spec
- [ ] When text credits = 0: stepper and image toggle sections are hidden; amber alert shown with upgrade link; Generate button disabled at 45% opacity and not clickable
- [ ] When text credits > 0: all controls are active; Generate button is enabled
- [ ] Modal closes on: "Cancel" button, X icon, clicking outside the modal overlay — none of these trigger generation
- [ ] Clicking "✨ Generate" submits the configuration and closes the modal; loading card appears on the post card
- [ ] Toast on full success: "{N} variation(s) generated successfully." — green, 4 seconds
- [ ] Toast on partial credits (some skipped): "Only {X} of {N} variations generated — you ran out of AI text credits mid-way. Upgrade your plan to get more." — red, 7 seconds
- [ ] Toast on zero output (none generated, credits ran out): "Not enough AI text credits to generate any variations. Please upgrade your plan." — red, 7 seconds
- [ ] All strings use i18n keys; translations added for all 7 locales

---

### Mock-ups:

N/A — no designs provided; implementation follows spec in PRD.

---

### Impact on existing data:

None — modal only configures and triggers generation; variation data is stored by the backend job.

---

### Impact on other products:

Web only — not applicable to mobile apps or Chrome extension.

---

### Dependencies:

- **[BE] Add AI variation generation API and background job for Evergreen automation**
- **[FE] Add AI generate buttons, variation badges, and loading states to Compose Posts step** — renders the "✨ Generate with AI" button that triggers this modal

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — Evergreen wizard is desktop-only; N/A
- [ ] Multilingual support — all strings in all 7 locale files
- [ ] UI theming support — all colors via `primary-cs-*` CSS variable classes; warning color via token, not hex; no hardcoded hex values
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---

## Story 4: [FE] Build Bulk Generate Variations modal for all posts in Evergreen automation

### Description:
As a user composing posts in the Evergreen automation wizard, I want to generate AI caption variations across all posts at once from a single modal, so I can fully populate my automation's variation library in one action instead of going post by post.

---

### Workflow:

1. User clicks "✨ Generate variations for all" above the post list
2. "Generate Variations for All Posts" modal opens
3. Modal title: "Generate Variations for All Posts"; subtitle: "Settings apply to every post in this automation"
4. **Warning notice** (amber/yellow alert, always shown): dynamically lists which posts already have variations (new ones will be added on top, not replace existing) and which posts are image-only (caption will be generated first)
   - Example: "Posts 2 & 3 already have variations. New ones will be added on top — existing variations won't be replaced. Post 1 has no caption. AI will generate a caption from its image first, then create variations."
5. **Variations per post** stepper (same as per-post modal: 1–10, default 3); below stepper: dynamic label "{N} posts × {V} = {total} variations" (updates live as count changes); hint: "Each variation consumes AI text credits."
6. **Generate images** toggle (same behaviour as per-post modal)
7. **Custom instructions** textarea (same as per-post modal)
8. Footer: "Cancel" (secondary) | "✨ Generate for All" (AI gradient)
9. User clicks "✨ Generate for All" → modal closes; each post shows its own loading card; posts are processed (parallel where infrastructure allows)
10. When all posts finish: single completion toast shown

**Zero text credits state**:
- Stepper, dynamic label, and image toggle hidden
- Amber alert: "You've run out of AI text credits. [Upgrade your plan] to generate variations."
- "✨ Generate for All" button disabled at 45% opacity

---

### Acceptance criteria:

- [ ] Modal opens when user clicks "✨ Generate variations for all"
- [ ] Modal title: "Generate Variations for All Posts"; subtitle: "Settings apply to every post in this automation"
- [ ] Warning notice always shown; dynamically lists posts that already have variations and image-only posts based on the current automation's state
- [ ] Variations per post stepper: default 3, min 1, max 10
- [ ] Dynamic label "{N} posts × {V} = {total} variations" updates in real time as the stepper changes
- [ ] Hint: "Each variation consumes AI text credits."
- [ ] Image toggle behaviour and disabled state (0 image credits) identical to per-post modal
- [ ] Custom instructions textarea identical to per-post modal
- [ ] When text credits = 0: stepper, dynamic label, and image toggle hidden; amber alert shown; Generate button disabled at 45% opacity
- [ ] Modal closes on: "Cancel", X icon, clicking outside overlay — no generation triggered
- [ ] Clicking "✨ Generate for All" calls the bulk generation endpoint and closes the modal
- [ ] Each post card shows its own loading card while its variations are being generated
- [ ] Toast on full success: "{Total} variations generated across all posts." — green, 4 seconds
- [ ] Toast on partial credits: "{X} variations generated across all posts. {Y} couldn't be created — AI text credits ran out mid-way. Upgrade to get more." — orange, 8 seconds
- [ ] Toast on zero output: "No variations could be generated — you've run out of AI text credits. Please upgrade your plan." — red, 7 seconds
- [ ] All strings use i18n keys; translations added for all 7 locales

---

### Mock-ups:

N/A — no designs provided; implementation follows spec in PRD.

---

### Impact on existing data:

None — modal triggers bulk generation; all variation data is stored by the backend jobs.

---

### Impact on other products:

Web only — not applicable to mobile apps or Chrome extension.

---

### Dependencies:

- **[BE] Add AI variation generation API and background job for Evergreen automation**
- **[FE] Add AI generate buttons, variation badges, and loading states to Compose Posts step** — renders the "✨ Generate variations for all" button that triggers this modal

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — Evergreen wizard is desktop-only; N/A
- [ ] Multilingual support — all strings in all 7 locale files
- [ ] UI theming support — all colors via `primary-cs-*` CSS variable classes; no hardcoded hex
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
