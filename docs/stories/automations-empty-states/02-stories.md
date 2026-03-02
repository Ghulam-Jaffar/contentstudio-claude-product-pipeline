# Stories: Automations Empty States

---

## Story 1: [FE] Add empty state UI for RSS Automation listing

### Description:

The RSS Automation listing page (`RSSAutomationListing.vue`) currently shows only a generic "No data found" label when a user has no RSS automations. This should be replaced with a proper empty state that communicates what the feature does and guides the user to create their first automation.

The `DataTable` component supports a named `empty-state` slot (`<slot name="empty-state">`) that can be filled with custom content. Use this slot in `RSSAutomationListing.vue` to render the new empty state.

**Key file:** `src/modules/automation/components/rss/listing/RSSAutomationListing.vue`
**DataTable slot:** `<template v-slot:empty-state>` — replaces the default `FileText` icon + text

**New empty state UI copy:**
- **Illustration:** Use the design reference illustration (orange/RSS-themed icon consistent with the `bg-cstu-warning-50` header color)
- **Headline:** `"No RSS automations yet"`
- **Subtext:** `"Automatically share content from any RSS or Atom feed to your social accounts. Set it up once and let ContentStudio do the posting for you."`
- **CTA button:** `"Create Your First Automation"` — primary button, navigates to `save-rss-automation` route (same as the existing "New Campaign" filter button)

**i18n keys to add** (all 7 locales: en, de, fr, es, it, el, zh):
```
automation.rss_automation.empty_state.headline = "No RSS automations yet"
automation.rss_automation.empty_state.subtext   = "Automatically share content from any RSS or Atom feed to your social accounts. Set it up once and let ContentStudio do the posting for you."
automation.rss_automation.empty_state.cta       = "Create Your First Automation"
```

---

### Workflow:

1. User navigates to **Automate → RSS Feed** in the sidebar
2. User has no RSS automations set up yet (or all have been deleted)
3. Instead of an empty table with a faint "No data found" label, the user sees a centered empty state in the table body:
   - An illustration related to RSS/feed automation
   - Headline: "No RSS automations yet"
   - Subtext: "Automatically share content from any RSS or Atom feed to your social accounts. Set it up once and let ContentStudio do the posting for you."
   - A primary CTA button: "Create Your First Automation"
4. User clicks "Create Your First Automation" and is taken to the RSS automation creation wizard

---

### Acceptance criteria:

- [ ] When the RSS automation listing has zero items and is not loading, the custom empty state is shown (not the generic "No data found" text)
- [ ] Empty state shows an illustration, headline, subtext, and CTA button as specified
- [ ] CTA button "Create Your First Automation" navigates the user to the RSS automation creation page (`save-rss-automation` route)
- [ ] Empty state is not shown during the loading state (spinner shows while data is being fetched)
- [ ] If a user searches and gets no results, the search-specific empty text still shows (existing behavior for the search empty state — this new empty state is for zero automations only, i.e. when no search filter is active)
- [ ] All copy uses i18n keys — no hardcoded English strings
- [ ] New i18n keys are added to all 7 locale files (en, de, fr, es, it, el, zh)
- [ ] Empty state layout is centered within the table body area and visually consistent with the page

---

### Mock-ups:

Design reference: https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a019ff-93b1-4624-b232-7756188a3311/image.png

See also design story: [Design] Automations empty state UI update with user-centric info and guide

---

### Impact on existing data:

None. This is a pure UI change — no schema or data changes required.

---

### Impact on other products:

The RSS automation listing is web-only. No impact on mobile apps or Chrome extension.

---

### Dependencies:

Depends on: **[Design] Automations empty state UI update with user-centric info and guide**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only — ensure empty state layout looks correct on smaller viewports)
- [ ] Multilingual support — new i18n keys must be added to all 7 locale files
- [ ] UI theming support — use `text-primary-cs-500`, `bg-primary-cs-50` for any primary-colored elements; no hardcoded hex colors
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web only — N/A for mobile apps and Chrome extension)

---
---

## Story 2: [FE] Add empty state UI for Bulk Schedule (CSV) listing

### Description:

The Bulk Schedule listing page (`CsvProcessListing.vue`) currently shows only "No data found" when there are no CSV bulk schedule uploads. This should be replaced with a proper empty state that explains the feature and gives users a clear next step.

The `DataTable` component supports a named `empty-state` slot. Use this slot in `CsvProcessListing.vue` to render the new empty state, replacing the current `empty-title` + `empty-message` props.

**Key file:** `src/modules/automation/components/csv/listing/CsvProcessListing.vue`
**DataTable slot:** `<template v-slot:empty-state>` — overrides default icon + text

**New empty state UI copy:**
- **Illustration:** Use the design reference illustration (blue-themed, consistent with the `bg-cstu-primary-50` header color)
- **Headline:** `"No bulk schedules yet"`
- **Subtext:** `"Upload a CSV file to schedule up to 500 posts at once across all your connected social accounts. Perfect for planned campaigns and content batches."`
- **CTA button:** `"Upload Your First CSV"` — primary button, navigates to `save-bulk-csv-automation` route (same as the existing "Bulk Schedule" button in the filter bar)

**i18n keys to add** (all 7 locales):
```
automation.csv_bulk_schedule.listing.empty_state.headline = "No bulk schedules yet"
automation.csv_bulk_schedule.listing.empty_state.subtext  = "Upload a CSV file to schedule up to 500 posts at once across all your connected social accounts. Perfect for planned campaigns and content batches."
automation.csv_bulk_schedule.listing.empty_state.cta      = "Upload Your First CSV"
```

---

### Workflow:

1. User navigates to **Automate → Bulk Scheduling** in the sidebar
2. User has no bulk schedule uploads yet
3. Instead of an empty table with "No data found", the user sees a centered empty state:
   - An illustration related to bulk scheduling/CSV
   - Headline: "No bulk schedules yet"
   - Subtext: "Upload a CSV file to schedule up to 500 posts at once across all your connected social accounts. Perfect for planned campaigns and content batches."
   - A primary CTA button: "Upload Your First CSV"
4. User clicks "Upload Your First CSV" and is taken to the bulk schedule creation wizard

---

### Acceptance criteria:

- [ ] When the CSV listing has zero items and is not loading, the custom empty state is shown
- [ ] Empty state shows illustration, headline, subtext, and CTA as specified
- [ ] CTA button "Upload Your First CSV" navigates to the bulk schedule creation page (`save-bulk-csv-automation` route)
- [ ] Empty state is not shown during the loading state
- [ ] If a user searches and gets no results, the existing behavior (empty table with default text) still applies; the new empty state is for the zero-items baseline state only
- [ ] All copy uses i18n keys — no hardcoded strings
- [ ] New i18n keys added to all 7 locale files
- [ ] Empty state layout is centered and consistent with the page design

---

### Mock-ups:

Design reference: https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a01a0c-1646-449d-aaf3-71ed416e5f65/image.png

See also design story: [Design] Automations empty state UI update with user-centric info and guide

---

### Impact on existing data:

None. Pure UI change.

---

### Impact on other products:

Web-only feature. No impact on mobile apps or Chrome extension.

---

### Dependencies:

Depends on: **[Design] Automations empty state UI update with user-centric info and guide**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — ensure empty state looks correct on smaller viewports
- [ ] Multilingual support — new i18n keys must be added to all 7 locale files
- [ ] UI theming support — use theme-aware color classes; no hardcoded hex values
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web only — N/A for mobile apps and Chrome extension)

---
---

## Story 3: [FE] Add empty state UI for Evergreen/Recycle Automation listing

### Description:

The Evergreen (Recycle) Automation listing page (`EvergreenAutomationListing.vue`) uses a legacy HTML `<table>` and currently shows a plain centered text cell when no automations exist:

```html
<td v-else colspan="7" class="text-center">
  {{ $t('automation.evergreen_automation.listing.empty_states.no_automations') }}
</td>
```

This needs to be replaced with a proper empty state: illustration, headline, subtext, and a CTA button to create the first evergreen automation.

**Key file:** `src/modules/automation/components/evergreen/listing/EvergreenAutomationListing.vue`
**Change:** Replace the `<tr v-else>` block (lines 569–579) with a full-width empty state row containing the new UI

**New empty state UI copy:**
- **Illustration:** Use the design reference illustration (green-themed, consistent with the `bg-[#EDFFE6]` header color)
- **Headline:** `"No evergreen automations yet"`
- **Subtext:** `"Recycle your best-performing content by setting up a post queue that runs on repeat. Your posts get reshared automatically so your profiles stay active without extra effort."`
- **CTA button:** `"Create Your First Automation"` — primary button, navigates to `save-evergreen-automation-accounts` route

**i18n keys to add** (all 7 locales):
```
automation.evergreen_automation.listing.empty_states.headline = "No evergreen automations yet"
automation.evergreen_automation.listing.empty_states.subtext  = "Recycle your best-performing content by setting up a post queue that runs on repeat. Your posts get reshared automatically so your profiles stay active without extra effort."
automation.evergreen_automation.listing.empty_states.cta      = "Create Your First Automation"
```

Note: The existing `no_automations` and `no_search_results` keys remain — the `no_automations` key can be replaced by the new `headline` key, or the old key can be kept as a fallback (team's preference).

---

### Workflow:

1. User navigates to **Automate → Evergreen** in the sidebar
2. User has no evergreen automations set up
3. Instead of a plain text cell inside an otherwise empty table, the user sees a full-width empty state centered below the table header:
   - An illustration (evergreen/recycle themed)
   - Headline: "No evergreen automations yet"
   - Subtext: "Recycle your best-performing content by setting up a post queue that runs on repeat. Your posts get reshared automatically so your profiles stay active without extra effort."
   - A primary CTA button: "Create Your First Automation"
4. User clicks "Create Your First Automation" and is taken to the evergreen automation creation flow

---

### Acceptance criteria:

- [ ] When the evergreen listing has zero items and is not loading, the custom empty state is shown in place of the plain text
- [ ] Empty state shows illustration, headline, subtext, and CTA as specified
- [ ] CTA button "Create Your First Automation" navigates to the evergreen automation creation page (`save-evergreen-automation-accounts` route)
- [ ] When the user has searched and there are no matching results, the search-specific message still shows ("No results found for your search query, please try again.")
- [ ] The empty state does not appear while the loading spinner is active
- [ ] All copy uses i18n keys — no hardcoded strings
- [ ] New i18n keys added to all 7 locale files
- [ ] Empty state is visually consistent with the RSS and Bulk Schedule empty states

---

### Mock-ups:

Design reference: https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a01a19-1abc-46dc-bad2-afe05a3c0270/image.png

See also design story: [Design] Automations empty state UI update with user-centric info and guide

---

### Impact on existing data:

None. Pure UI change.

---

### Impact on other products:

Web-only feature. No impact on mobile apps or Chrome extension.

---

### Dependencies:

Depends on: **[Design] Automations empty state UI update with user-centric info and guide**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — ensure empty state looks correct on smaller viewports
- [ ] Multilingual support — new i18n keys must be added to all 7 locale files
- [ ] UI theming support — use `text-primary-cs-500`, `bg-primary-cs-50` for primary-colored elements; no hardcoded hex values
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web only — N/A for mobile apps and Chrome extension)
