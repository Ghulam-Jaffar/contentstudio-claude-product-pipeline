# Research: Brand Knowledge Setup UX Improvement

## Current State

**The form lives in one shared component used in two places:**
- `src/modules/publisher/ai-content-library/components/editors/BrandKnowledgeSetupEditor.vue` — the single source of truth
- Rendered in **AI Library** via `src/modules/publisher/ai-content-library/views/AIContentMain.vue`
- Rendered in **Settings → Brand** via `src/modules/setting/components/BrandSettings.vue`

**The 4 fields in the form (all visually equal weight):**
1. **Website URL** — `TextInput` for `setupInfo.website.url`
2. **Brand Info** — `Textarea` for `setupInfo.text.content` (1000 char max)
3. **Upload Files** — `UploadsTab` component for `setupInfo.uploads.files`
4. **Analyze Previous Posts** — `SocialAccountsFilterDropdown` for `setupInfo.social`

**Validation (already correct in code — NOT correct in visual presentation):**
```js
// At least one of these must be filled — code is already right
if (!hasWebsite && !hasText && !hasFiles && !hasSocialAccounts) {
  validationError.value = 'Please fill at least one...'
}
```
The problem is purely visual: all 4 fields look equally mandatory in the UI. Nothing tells the user "website is primary, the rest are optional enrichment."

**Design library Collapsible:** Already used in this codebase (`AddTeamMember.vue`, etc.). Imported from `@contentstudio/ui`.

## Recommended UX Approach

**Keep the form as-is structurally. Change only the visual hierarchy and presentation:**

1. **Website URL** stays prominent at the top — add a `"Recommended"` badge next to the label
2. **Add helper text** below the website field:
   > _"Don't have a website? You can provide brand information using any of the sources below — one is enough to get started."_
3. **Wrap the remaining 3 fields** in a `Collapsible` component from the design library
   - **Trigger label:** `"Add more sources (optional)"`
   - **Default state:** Closed — users who just want to enter a website go straight to "Build Brand Knowledge"
   - Each of the 3 fields inside gets `(Optional)` in its label
4. **Validation stays identical** — at least one of the four fields (including website) must be filled. Error message updated to reflect the new UX.

**This UX handles both user types:**
- User has a website → type URL → Build → done (3 fields never distract them)
- User has no website → sees helper text → opens collapsible → fills in brand info/files/social → Build

## What Needs to Change

- `BrandKnowledgeSetupEditor.vue`:
  - Add "Recommended" badge to Website URL label
  - Add helper text below website field
  - Wrap Brand Info + Upload Files + Analyze Previous Posts in a `Collapsible` from `@contentstudio/ui`
  - Add `(Optional)` to the 3 labels inside the collapsible
  - Update validation error message copy
  - Import `Collapsible` from `@contentstudio/ui`

No backend changes. No other files need to change (both Settings and AI Library render the same component).

## Files Involved

- `src/modules/publisher/ai-content-library/components/editors/BrandKnowledgeSetupEditor.vue` — only file to modify
