# Research: Planner Share Link — Sort & View Persistence

## Current State

**The share link page lives in one component:**
- `contentstudio-frontend/src/modules/planner_v2/views/SharePlans.vue` (1903 lines)
- Standalone public view — no auth required

**Current view state:**
- `state.currentView` defaults to `'list'` (hardcoded in `reactive({...})`)
- `switchView(view)` resets `plans.data = []`, `plans.page = 1`, then calls `fetchPlans()`
- localStorage is already used in this file (`localStorage.getItem('external_email')` at line ~708)
- There is no persistence of the chosen view — every page load resets to list

**Current sort state:**
- `fetchPlans(page, options)` sends: `token`, `page`, `limit` (10 for list, 500 for calendar), optional `calendar_date`
- **No sort params sent** — sort is hardcoded server-side
- Backend route: `POST /shareLink/fetchPlans` → `ShareLinkController::getShareLinkPlans()`
  - File: `contentstudio-backend/app/Http/Controllers/Planner/ShareLinkController.php` (line 289)
  - Calls `PlansRepository::getV2PlansByDate()` (line 2139) or `PlansRepository::getV2PlansByIds()` (line 2124)
  - Both methods: `.orderBy('execution_time.date')` — **ascending (oldest first), hardcoded**
  - File: `contentstudio-backend/app/Repository/Publish/Planner/PlansRepository.php`

**Pagination:** Infinite scroll — 10 posts per page for list view. Sort must be server-side (client-side sort would only sort the current page of 10 results).

**No existing sort dropdown on the SharePlans page.** PlannerHeader has a sort dropdown for the internal planner view, but it has many options not relevant to external client review.

## What Needs to Change

**Backend (`ShareLinkController` + `PlansRepository`):**
- Accept `sort_field` and `sort_direction` from request payload in `getShareLinkPlans()`
- Pass them to `PlansRepository::getV2PlansByDate()` and `PlansRepository::getV2PlansByIds()`
- Both repository methods apply the `orderBy` using the provided sort params (with safe defaults)
- Allowed sort fields: `execution_time.date` (scheduled), `created_at` (created)
- Allowed directions: `asc`, `desc`
- Default: `execution_time.date` / `asc` (preserves current behavior if params not sent)

**Frontend (`SharePlans.vue`):**
- On mount: read `share_link_view` and `share_link_sort` from localStorage; apply to `state.currentView` and sort state
- On view switch: write `share_link_view` to localStorage
- On sort change: write `share_link_sort` to localStorage, reset plans and re-fetch
- Add sort dropdown (list view only — hidden when in calendar view) using `CstDropdown` / `CstDropdownItem` from `@contentstudio/ui`
- Include chosen sort in the `fetchPlans` payload as `sort_field` + `sort_direction`

## Recommended Sort Options (4 options)

| Label | sort_field | sort_direction |
|---|---|---|
| Scheduled: Newest first | `execution_time.date` | `desc` |
| Scheduled: Oldest first | `execution_time.date` | `asc` |
| Created: Newest first | `created_at` | `desc` |
| Created: Oldest first | `created_at` | `asc` |

"Updated" skipped — clients reviewing posts don't need to sort by last-edited time.

**Default sort** (when no localStorage preference): `Scheduled: Newest first` — this directly solves the customer's complaint about oldest posts appearing first.

## Files Involved

**Backend:**
- `app/Http/Controllers/Planner/ShareLinkController.php` — extract and validate sort params, pass to repository
- `app/Repository/Publish/Planner/PlansRepository.php` — accept and apply sort to `getV2PlansByDate()` and `getV2PlansByIds()`

**Frontend:**
- `src/modules/planner_v2/views/SharePlans.vue` — only file to modify
