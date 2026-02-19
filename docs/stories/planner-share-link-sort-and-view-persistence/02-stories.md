# Stories: Planner Share Link — Sort & View Persistence

---

## Story 1: [BE] Add sort parameter support to the planner share link fetch endpoint

### Description:

The `POST /shareLink/fetchPlans` endpoint currently returns plans sorted by scheduled date ascending (oldest first) with no way for the caller to change it. This story adds server-side sort support so the frontend can request plans in any supported order.

**Files to modify:**

1. `app/Http/Controllers/Planner/ShareLinkController.php` — method `getShareLinkPlans()` (line 289)
2. `app/Repository/Publish/Planner/PlansRepository.php` — methods `getV2PlansByDate()` (line 2139) and `getV2PlansByIds()` (line 2124)

**Changes:**

1. In `getShareLinkPlans()`: extract `sort_field` and `sort_direction` from the request payload. Validate against the allowed allowlist:
   - Allowed `sort_field` values: `execution_time.date`, `created_at`
   - Allowed `sort_direction` values: `asc`, `desc`
   - If not provided or invalid: default to `execution_time.date` / `asc` (preserves current behavior)
2. Pass the validated sort params to both `PlansRepository::getV2PlansByDate()` and `PlansRepository::getV2PlansByIds()`
3. In both repository methods: replace the hardcoded `.orderBy('execution_time.date')` with `.orderBy($sortField, $sortDirection)` using the passed params

**No new endpoint, no schema changes, no breaking changes** — the sort params are optional; omitting them falls back to current behavior.

---

### Workflow:

1. External client opens a share link
2. Their browser sends `POST /shareLink/fetchPlans` with `{ token, page, limit, sort_field: "execution_time.date", sort_direction: "desc" }`
3. The backend validates the sort params, queries the database with the requested sort order, and returns plans newest-scheduled-first
4. If `sort_field` or `sort_direction` are missing or invalid, the backend silently defaults to `execution_time.date` / `asc` and returns results as before

---

### Acceptance criteria:

- [ ] `POST /shareLink/fetchPlans` accepts optional `sort_field` and `sort_direction` in the request body
- [ ] When `sort_field: "execution_time.date"` and `sort_direction: "desc"` are sent, plans are returned newest-scheduled-first
- [ ] When `sort_field: "execution_time.date"` and `sort_direction: "asc"` are sent, plans are returned oldest-scheduled-first
- [ ] When `sort_field: "created_at"` and `sort_direction: "desc"` are sent, plans are returned newest-created-first
- [ ] When `sort_field: "created_at"` and `sort_direction: "asc"` are sent, plans are returned oldest-created-first
- [ ] When `sort_field` or `sort_direction` are omitted, response is identical to current behavior (sorted by `execution_time.date` ascending)
- [ ] Sending an unsupported `sort_field` or `sort_direction` value falls back to the default (no 400 error)
- [ ] Sort applies correctly for both share modes: date-range (future content) and plan-IDs (specific plans)
- [ ] Existing share link functionality (token validation, password protection, social filtering, notes) is unaffected

---

### Mock-ups:

N/A — backend-only change.

---

### Impact on existing data:

None. No schema changes. The `sort_field` and `sort_direction` params are optional — existing clients that don't send them will receive unchanged behavior.

---

### Impact on other products:

- Mobile apps: Not affected — mobile does not use the shareable planner link view
- Chrome extension: Not affected
- Analytics share links: Not affected — this is the planner share link endpoint only

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support — N/A, no user-facing strings in this story
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 2: [FE] Add sort dropdown and persist view/sort preferences on the planner share link page

### Description:

The shared planner link page (`SharePlans.vue`) currently resets to list view and shows posts oldest-first every time the page is loaded. External clients (typically agency customers' clients) have no control over sort order, and their last-used view preference isn't saved. This story adds:

1. **View persistence** — the client's last-used view (list or calendar) is saved in localStorage and restored on page load
2. **Sort dropdown** — a simplified sort control in the list view header, using existing `CstDropdown` / `CstDropdownItem` components from `@contentstudio/ui`
3. **Sort persistence** — the chosen sort option is saved in localStorage and restored on page load

**Single file to modify:** `src/modules/planner_v2/views/SharePlans.vue`

**Changes:**

1. On mount: read `share_link_view` from localStorage; if set, initialize `state.currentView` to that value (list or calendar)
2. On mount: read `share_link_sort` from localStorage; if set, initialize sort state from it; otherwise default to `{ field: 'execution_time.date', direction: 'desc' }` (Scheduled: Newest first)
3. In `switchView(view)`: write `share_link_view` to localStorage after updating `state.currentView`
4. Add a `sort` reactive state: `{ field: 'execution_time.date', direction: 'desc' }`
5. Add a `changeSortOption(option)` handler: updates sort state, writes `share_link_sort` to localStorage, resets `plans.data = []` and `plans.page = 1`, then calls `fetchPlans()`
6. In `fetchPlans()`: include `sort_field` and `sort_direction` from sort state in the API payload
7. Add a sort `CstDropdown` in the controls row — visible only when `state.currentView === 'list'`; positioned inline with the existing view switcher (list/calendar toggle)

**Depends on:** [BE] Add sort parameter support to the planner share link fetch endpoint

---

### Workflow:

**First visit — client with a website link (typical case):**
1. Client opens the share link URL
2. Page loads in list view (default, unchanged), posts sorted **Scheduled: Newest first** (new default — most recent scheduled content appears at top)
3. Client sees a sort dropdown in the controls row labelled with the active sort: **"Scheduled: Newest first"**
4. Client browses; their view and sort choice are automatically remembered for their next visit

**Returning client — saved preferences applied:**
1. Client reopens the same share link URL
2. Page loads in whatever view they last used (list or calendar), with the sort they last chose applied instantly
3. No extra steps — it just works as they left it

**Client changing sort:**
1. Client clicks the sort dropdown
2. They see 4 options: Scheduled: Newest first, Scheduled: Oldest first, Created: Newest first, Created: Oldest first
3. They click one — the plan list reloads with the new sort order
4. The dropdown label updates to show the selected option
5. Their choice is saved — it'll be applied on their next visit

---

### Acceptance criteria:

- [ ] On first page load (no localStorage), `state.currentView` defaults to `'list'` (unchanged)
- [ ] On first page load (no localStorage), sort defaults to `Scheduled: Newest first` (`execution_time.date` desc)
- [ ] On subsequent page loads, `share_link_view` from localStorage is applied to the initial view (list or calendar)
- [ ] On subsequent page loads, `share_link_sort` from localStorage is applied to the initial sort before the first `fetchPlans()` call
- [ ] When the user switches between list and calendar view, `share_link_view` is written to localStorage
- [ ] `fetchPlans()` includes `sort_field` and `sort_direction` in every POST payload
- [ ] The sort dropdown is visible only when `state.currentView === 'list'`; it is hidden in calendar view
- [ ] The sort dropdown shows the currently active sort option as its label
- [ ] Selecting a sort option resets `plans.data` and `plans.page`, then re-fetches with the new sort
- [ ] Selecting a sort option writes the new value to `share_link_sort` in localStorage
- [ ] The sort dropdown contains exactly 4 options in this order: Scheduled: Newest first, Scheduled: Oldest first, Created: Newest first, Created: Oldest first
- [ ] The active (currently selected) sort option is visually indicated (checkmark or highlight) in the dropdown
- [ ] Infinite scroll continues working correctly after a sort change (loads subsequent pages in the same sort order)
- [ ] No regressions: existing filter dropdown, view switcher, bulk actions, password gate, comment/approval flows all work as before
- [ ] No hardcoded colors — sort dropdown uses design library components styled via `text-primary-cs-500` / `bg-primary-cs-50` for active state

---

### Mock-ups:

N/A — uses existing `CstDropdown` / `CstDropdownItem` design library components, same pattern as other dropdowns in the planner. The sort dropdown sits inline with the view switcher (list/calendar toggle) in the controls row at the top of the page.

---

### UI Copy

**Sort dropdown trigger (default / no selection saved):**

| State | Label |
|---|---|
| Default (no preference) | Scheduled: Newest first |
| After selecting option | _(shows the selected option label)_ |

**Sort dropdown options:**

| Option label | Behavior |
|---|---|
| Scheduled: Newest first | Posts sorted by scheduled date, most recent first |
| Scheduled: Oldest first | Posts sorted by scheduled date, oldest first |
| Created: Newest first | Posts sorted by when they were created, most recent first |
| Created: Oldest first | Posts sorted by when they were created, oldest first |

**No tooltip needed** on the sort dropdown — the labels are self-explanatory for client reviewers.

---

### Impact on existing data:

None. localStorage keys (`share_link_view`, `share_link_sort`) are new — no existing data is overwritten. The default sort change (from oldest-first to newest-first) only affects new page loads; it does not alter any stored data.

---

### Impact on other products:

- Mobile apps: Not affected — the share link page is web-only
- Chrome extension: Not affected
- Settings → Brand / AI Library: Not affected — unrelated component

---

### Dependencies:

Depends on: **[BE] Add sort parameter support to the planner share link fetch endpoint**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (sort dropdown and controls row should work on smaller screen widths — external clients may open share links on mobile)
- [ ] Multilingual support — N/A, no new translatable strings; sort labels are English-only UI controls on a public page without i18n
- [ ] UI theming support (design library `CstDropdown` used; active sort state uses `text-primary-cs-500` / `bg-primary-cs-50`, no hardcoded colors)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
