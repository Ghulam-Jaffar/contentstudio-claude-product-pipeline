# Research: Automations Empty States

## Current State

All three automation listing pages currently show bare-minimum empty states when no automations exist:

### RSS Automation
- **File:** `contentstudio-frontend/src/modules/automation/components/rss/listing/RSSAutomationListing.vue`
- Uses `<DataTable>` component with `:empty-title="$t('automation.rss_automation.table.empty_title')"` and `empty-message=""`
- Current locale key: `"empty_title": "No data found"` — no description, no illustration, no CTA
- `DataTable` supports an `empty-state` named slot (`<slot name="empty-state">`) that can be overridden with custom content

### Bulk Schedule (CSV)
- **File:** `contentstudio-frontend/src/modules/automation/components/csv/listing/CsvProcessListing.vue`
- Uses `<DataTable>` with `:empty-title="$t('automation.csv_bulk_schedule.listing.table.empty_title')"` and `:empty-message="$t('automation.csv_bulk_schedule.listing.table.empty_message')"`
- Current locale keys: `"empty_title": "No data found"`, `"empty_message": ""`
- Same `DataTable` empty-state slot available

### Evergreen / Recycle Automation
- **File:** `contentstudio-frontend/src/modules/automation/components/evergreen/listing/EvergreenAutomationListing.vue`
- Uses a legacy `<table>` (not DataTable) with a plain `<tr v-else>` row:
  ```html
  <td v-else colspan="7" class="text-center">
    {{ $t('automation.evergreen_automation.listing.empty_states.no_automations') }}
  </td>
  ```
- Current locale: `"no_automations": "You have not created any automation yet."`
- No illustration, no CTA, no description

### Locale File
- **File:** `contentstudio-frontend/src/locales/en/automation.json` (plus 6 other language files)
- All 7 locales need new keys added

### DataTable Component
- **File:** `contentstudio-frontend/src/modules/common/components/DataTable.vue`
- Default empty state renders a `FileText` icon + `emptyTitle` + `emptyMessage` (lines 147–153)
- Can be overridden by filling the `empty-state` slot

## What Needs to Change

- **RSS:** Override DataTable's `empty-state` slot in `RSSAutomationListing.vue` with illustration, headline, subtext, and "New Campaign" CTA button
- **Bulk Schedule:** Override DataTable's `empty-state` slot in `CsvProcessListing.vue` with illustration, headline, subtext, and "Bulk Schedule" CTA button
- **Evergreen:** Replace the plain `<td v-else>` empty row in `EvergreenAutomationListing.vue` with a proper empty state block (illustration, headline, subtext, CTA)
- All three: add i18n keys (headline, subtext) to all 7 locale files

## Design References

- **RSS:** https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a019ff-93b1-4624-b232-7756188a3311/image.png
- **Bulk Schedule:** https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a01a0c-1646-449d-aaf3-71ed416e5f65/image.png
- **Evergreen:** https://media.app.shortcut.com/api/attachments/files/clubhouse-assets/5e0c5625-83f1-4c4f-b9a3-ac79e02e1f07/69a01a19-1abc-46dc-bad2-afe05a3c0270/image.png
- **Design story:** https://app.shortcut.com/contentstudio-team/story/111725/automations-empty-state-ui-update-with-user-centric-info-and-guide-design

## Files Involved

- `src/modules/automation/components/rss/listing/RSSAutomationListing.vue`
- `src/modules/automation/components/csv/listing/CsvProcessListing.vue`
- `src/modules/automation/components/evergreen/listing/EvergreenAutomationListing.vue`
- `src/locales/en/automation.json` (+ de, fr, es, it, el, zh)
