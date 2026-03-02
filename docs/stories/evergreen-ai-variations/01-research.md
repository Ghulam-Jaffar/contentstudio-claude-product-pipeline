# Research: AI-Powered Variations — Evergreen Automation (Step 2)

## Current State

The Evergreen Automation wizard has 3 steps:
1. Campaign & Accounts — `EvergreenAccountSelection.vue`
2. **Compose Posts** — `EvergreenFinalization.vue` ← target step
3. Schedule & Finalize — `EvergreenOptimization.vue`

**Orchestrator:** `EvergreenMain.vue` — handles step navigation and draft saves
**State management:** Vuex store at `src/modules/automation/store/recipes/evergreen.js`

### Current Variation Architecture

Posts are stored as nested arrays: `posts[postIndex][variationIndex]`

Current variation operations in `EvergreenFinalization.vue`:
- **Add variation**: copies first variation as template → opens `AddEvergreenPost` modal → pushes to `posts[index]`
- **Edit variation**: opens modal pre-filled with existing data → updates `posts[postIndex][variationIndex]`
- **Auto-generate from URL**: `addURLVariation()` → opens `URLVariationModal` → calls `fetchVariationUrl` action → user confirms preview → `addVariationFromPreview()` pushes selected
- **Remove variation**: `posts[postIndex].splice(variationIndex, 1)`

The existing "Auto generate variations" is URL-based content extraction (not AI caption generation from the post's own content). The new feature is a different, AI-native flow.

### UI/Modal System
- **Modals**: `$cstuModal.show('id')` / `$cstuModal.hide('id')` using `CstuModal` component
- **Toasts**: `dispatch('toastNotification', { message, type: 'success' | 'error' | 'warn' })`
- **Component library**: `@contentstudio/ui` — Button, ActionIcon, CstuModal, etc.
- **Button variants**: `color="secondary"` for secondary, `color="primary"` for primary; the AI gradient style needs to follow whatever `@contentstudio/ui` provides (check existing AI button usage in composer)

### Credit System (existing pattern)
- Getters: `getCreditUsedLimit`, `getCreditSubscribeLimit` (from AI tools module)
- Upgrade link: `handleIncreaseLimitsClick(ADDON_KEYS.AI_TEXT_CREDITS)` opens billing modal
- The automation wizard does NOT currently use AI credits — this is new integration

### Code Standards (from `contentstudio-frontend/CLAUDE.md`)
- **New components**: must use `<script setup lang="ts">` (Composition API)
- **Existing components** (EvergreenFinalization.vue): Options API — add new code in same style, or extract into composable
- **No hardcoded hex colors** — use CSS variables / Tailwind theme classes
- **i18n required**: all strings via `$t('automation.evergreen_automation...')` in all 7 locales
- **API URLs**: defined in `src/config/api-utils.js`
- **HTTP**: via `proxy` from `@common/lib/http-common`

---

## What Needs to Change

### Backend
- New API endpoint to trigger AI variation generation (single post + bulk)
- Validate AI text credits before generating; validate AI image credits if image toggle on
- Async background job: generate caption (if image-only post), then N variations
- Deduct credits after generation (only for successfully completed variations)
- Auto-save completed variations to the automation draft when job finishes
- Expose job status for frontend polling or push via websocket

### Frontend
- **New AI generate button** on each post card header: `✨ Generate with AI` (alongside existing `+ Add variation`)
- **New top-level bulk button**: `✨ Generate variations for all` (above post list, right-aligned)
- **AI badge** on variations generated via AI (no state machine — variations are pushed directly in normal card styling, same on-hover Edit/Delete as existing)
- **Variation count badge**: live-updating pill showing `{N} Variation(s)` on each post card header
- **Loading card**: dashed border, spinner, appended during generation, removed when done
- **New per-post modal**: "Generate Variations" — source detection, count stepper, image toggle, custom instructions, credit state handling
- **New bulk modal**: "Generate Variations for All Posts" — multiplier label, same controls, warning notice
- **Navigation warning banner**: non-blocking, appears if user tries to navigate away during generation
- **Credit state integration**: pull `getCreditUsedLimit`/`getCreditSubscribeLimit` into the automation context; show upgrade prompts

---

## Spec Adjustments vs. Submitted Spec

| Spec item | Adjustment needed |
|---|---|
| `#F8F7FF` / `#C7D2FE` for AI variation card | Use `bg-primary-cs-50`, `border-primary-cs-200` (CSS variable-backed) |
| `#FAFBFD` / `#E2E8F0` for original variation card | Use `bg-gray-50`, `border-gray-200` |
| "AI gradient (indigo/purple)" button style | Use `@contentstudio/ui` Button with the AI variant (check composer for exact prop) |
| "orange" upgrade link in image credits exhausted state | Use `text-[rgb(var(--cstu-warning-500))]` or the warning color token |
| All UI string literals | All must be i18n keys — copy in spec is the source of truth for translations |
| `warn` toast type | Confirmed valid: `type: 'warn'` works in the toast system |
| `+Add variation` button (out of scope per spec §17) | Don't remove existing button — it stays; the new AI button is additive |

---

## UX Consistency Observations

The existing `URLVariationModal` shows a preview of generated variations in a list where users check which to add. The new AI modal is different — it's a settings modal (configure then fire), not a preview modal. This is consistent with how AI Caption modal works in the Composer. ✓

The spec's "pending → keep/discard" flow is a new UX pattern for evergreen (existing has no pending state — you add from preview in bulk). This is intentional and an improvement. ✓

---

## Files Involved

### Frontend
| File | Change type |
|---|---|
| `src/modules/automation/components/evergreen/create/EvergreenFinalization.vue` | Modify — add AI generate buttons, variation card states, variation count badge, navigation banner |
| `src/modules/automation/components/evergreen/create/GenerateVariationsModal.vue` | **New** — per-post AI generate modal |
| `src/modules/automation/components/evergreen/create/BulkGenerateVariationsModal.vue` | **New** — bulk generate modal |
| `src/modules/automation/store/recipes/evergreen.js` | Modify — add AI generation state, actions, mutations |
| `src/config/api-utils.js` | Add new AI variation generation endpoint URL |
| `src/locales/en/automation.json` (+ 6 other locales) | Add all new i18n keys |

### Backend
| Area | Change |
|---|---|
| New route | `POST /triggerVariationGeneration` (single post) + `POST /triggerBulkVariationGeneration` |
| Credit validation | Validate text + image credits before dispatch |
| Async job | Background worker: analyse image → generate caption → generate N variations |
| Credit deduction | After generation, deduct per completed variation |
| Draft auto-save | On job completion, push variations to wizard draft and trigger websocket/polling event |

---

## Story Count (revised — simplified flow)

With no pending/keep/discard state machine, the story split is:

1. `[BE]` AI variation generation — trigger API (single + bulk) + async background job + credit validation/deduction + draft auto-save on completion
2. `[FE]` Compose Posts step — AI generate buttons, variation count badge, AI badge on generated variations, loading card, navigation warning banner
3. `[FE]` Generate Variations modal (per post) with credit state handling
4. `[FE]` Bulk Generate Variations for All Posts modal + sequential processing

**4 stories** → fits within `/story` count, but user has requested a dedicated epic → using `/feature` pipeline for epic creation.
