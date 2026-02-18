# Stories: Brand Knowledge Setup UX Improvement

---

## Story 1: [FE] Improve brand knowledge setup form to clearly communicate optional fields

### Description:

The brand knowledge setup form currently shows four input fields — Website URL, Brand Info, Upload Files, and Analyze Previous Posts — all at equal visual weight. The form requires at least one to be filled, but nothing in the UI communicates this. Users landing on the form see four fields that all look mandatory, which feels overwhelming and creates unnecessary friction.

This story restructures the form's visual hierarchy so users immediately understand:
- Website URL is the primary, recommended input
- The other three fields are optional enrichment
- Users without a website can still proceed using any other source

**Single file to modify:** `src/modules/publisher/ai-content-library/components/editors/BrandKnowledgeSetupEditor.vue`

This fix applies to both entry points automatically — AI Library (`AIContentMain.vue`) and Settings → Brand (`BrandSettings.vue`) both render this shared component.

**Changes:**
1. Add a `"Recommended"` badge next to the Website URL label
2. Add helper text below the website input field
3. Wrap the three remaining fields (Brand Info, Upload Files, Social Accounts) in a `Collapsible` component from `@contentstudio/ui` — closed by default
4. Add `(Optional)` to the label of each field inside the collapsible
5. Update the validation error message copy
6. Import `Collapsible` from `@contentstudio/ui` (already used elsewhere in the codebase)

---

### Workflow:

**User with a website:**
1. User opens AI Library or navigates to Settings → Brand
2. They see the setup form with a single prominent field: **Website URL** (labelled "Recommended")
3. They paste their URL and click **"Build Brand Knowledge"**
4. Done — the collapsible sections below never distract them

**User without a website:**
1. User opens the setup form
2. They see the Website URL field with helper text below: _"Don't have a website? You can provide brand information using any of the sources below — one is enough to get started."_
3. They click **"Add more sources (optional)"** — the collapsible opens, revealing Brand Info, Upload Files, and Analyze Previous Posts
4. They fill in one or more of the optional fields
5. They click **"Build Brand Knowledge"**

---

### Acceptance criteria:

- [ ] Website URL field remains at the top of the form, visually distinct
- [ ] A `"Recommended"` badge (pill) appears next to the Website URL label
- [ ] Helper text appears below the Website URL input: _"Don't have a website? You can provide brand information using any of the sources below — one is enough to get started."_
- [ ] Brand Info, Upload Files, and Analyze Previous Posts are wrapped in a `Collapsible` component from the design library (`@contentstudio/ui`)
- [ ] The collapsible trigger label reads: **"Add more sources (optional)"**
- [ ] The collapsible is **closed by default** when the form first loads
- [ ] The collapsible opens/closes on click of the trigger
- [ ] Inside the collapsible, each field label includes `(Optional)`: **"Brand Info (Optional)"**, **"Upload Files (Optional)"**, **"Analyze Previous Posts (Optional)"**
- [ ] All existing tooltips on the three optional fields are preserved
- [ ] Validation logic is unchanged — at least one of the four fields must have a value before submitting
- [ ] The validation error message reads: _"Please add at least one source — a website URL, brand info, uploaded file, or social account."_
- [ ] If the user submits with no website and the collapsible is closed, the collapsible auto-expands to reveal the optional fields alongside the error message
- [ ] The fix renders correctly in both AI Library and Settings → Brand (same component, both entry points)
- [ ] No hardcoded colors — uses `text-primary-cs-500`, `bg-primary-cs-50` for the "Recommended" badge

---

### Mock-ups:

N/A — use design library `Collapsible` component. Reference its usage in `src/modules/setting/components/workspace/team/AddTeamMember.vue` for the import and props pattern.

---

### UI Copy

**Website URL section:**

| Element | Copy |
|---|---|
| Label | Website URL |
| "Recommended" badge (next to label) | Recommended |
| Placeholder | https://yourwebsite.com |
| Helper text below input | Don't have a website? You can provide brand information using any of the sources below — one is enough to get started. |
| Tooltip (existing, preserve) | _(keep existing translation key)_ |

**Collapsible trigger:**

| Element | Copy |
|---|---|
| Trigger label (closed state) | Add more sources (optional) |
| Trigger label (open state) | Add more sources (optional) _(same — no need to change on open)_ |

**Fields inside collapsible:**

| Field | Label | Tooltip |
|---|---|---|
| Brand Info | Brand Info (Optional) | _(keep existing tooltip)_ |
| Upload Files | Upload Files (Optional) | _(keep existing tooltip)_ |
| Analyze Previous Posts | Analyze Previous Posts (Optional) | _(keep existing tooltip)_ |

**Validation error:**

| Scenario | Message |
|---|---|
| Submitted with nothing filled | Please add at least one source — a website URL, brand info, uploaded file, or social account. |

---

### Impact on existing data:

None — purely a visual restructuring. The form data model (`setupInfo.website`, `setupInfo.text`, `setupInfo.uploads`, `setupInfo.social`) and submission logic are unchanged.

---

### Impact on other products:

- Settings → Brand and AI Library both benefit from the fix automatically (shared component)
- Mobile apps: Not affected — this is a web-only AI feature
- Chrome extension: Not affected

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (collapsible and form layout should work on smaller screen widths)
- [ ] Multilingual support (new strings — "Recommended" badge, helper text, updated validation message — must use i18n translation keys)
- [ ] UI theming support (design library `Collapsible` used; "Recommended" badge uses `text-primary-cs-500` / `bg-primary-cs-50`, no hardcoded colors)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
