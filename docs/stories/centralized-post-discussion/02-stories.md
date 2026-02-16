# Centralized Post Discussion — Stories

---

## Story 1: [BE] Add comment status filter and "Post Discussions" custom view migration

### Description:

Add a new `comment_status` filter to the planner's post-fetching logic so users can filter posts by whether their comments are resolved or unresolved. Also create a migration that provisions a "Post Discussions" custom view for every active workspace team member.

**Comment status filter in `PlansRepository::fetchPlans()`** (`app/Repository/Publish/Planner/PlansRepository.php`):
- Accept a new optional `comment_status` parameter (values: `resolved`, `unresolved`)
- When `comment_status = 'unresolved'`: return only plans that have **at least one** comment where `is_resolved` is `false` or `null`. This must check both:
  - Regular comments in the `comment` collection (matching by `plan_id`)
  - External comments stored in `plan.external_comments` array
- When `comment_status = 'resolved'`: return only plans where **all** comments have `is_resolved === true` (both regular and external). Plans with zero comments should be excluded.
- Use MongoDB aggregation with `$lookup` on the `comment` collection to determine comment resolution status per plan, similar to how `PlannerViewService::getCommentCount()` (lines 524–568) already filters on `is_resolved !== true`.

**Wire the filter through the request/service layer:**
- Accept `comment_status` in the planner API request (the controller that calls `fetchPlans()`)
- Pass it through `PlannerViewService` down to `PlansRepository`

**Update `PlannerSavedViews` model** (`app/Models/Publish/Planner/PlannerSavedViews.php`):
- Add `comment_status` to `$fillable` array
- Add `'comment_status' => 'string'` to `$casts` array

**Create migration: `create_post_discussions_custom_view`**
Follow the exact pattern from `database/migrations/2026_01_21_100000_create_approval_custom_views.php`:
- Query `WorkspaceTeam::where('status', 'joined')`, chunk by 500
- For each member, check if a view named "Post Discussions" already exists for that `user_id` + `workspace_id`
- If not, create a `PlannerSavedViews` record:
  - `name`: `"Post Discussions"`
  - `workspace_id`: member's workspace_id
  - `user_id`: member's user_id
  - `comment_status`: `"unresolved"` (auto-checked filter)
  - `view_type`: `"Feed"`
  - `visibility`: `"Only me"`
  - `default`: `false`
  - `sort_order`: `"Last Created"`
  - All other filter arrays (`types`, `statuses`, `members`, `created_by`, `labels`, `campaigns`, `content_categories`, `platform_selections`, `platform_accounts`): empty arrays `[]`
  - `date_range`: `null`
- Log progress every 1000 members, log final summary (created/skipped/failed counts)
- `down()`: delete all `PlannerSavedViews` where `name = "Post Discussions"`

---

### Workflow:

1. User opens the Content Planner and sees the existing list of posts
2. User applies the "Unresolved" comment filter from the filter sidebar
3. The planner refetches posts, showing only posts that have at least one unresolved comment
4. User switches to the "Resolved" filter and sees only posts where all comments are resolved
5. User clicks the "Post Discussions" custom view in the sidebar (pre-created by migration)
6. The planner loads with Feed view and the unresolved comment filter auto-applied, showing all posts with active discussions

---

### Acceptance criteria:

- [ ] `PlansRepository::fetchPlans()` accepts `comment_status` parameter and correctly filters plans
- [ ] `comment_status = 'unresolved'` returns only plans with at least one comment where `is_resolved` is `false` or `null`
- [ ] `comment_status = 'resolved'` returns only plans where every comment has `is_resolved === true`
- [ ] Both regular comments (`comment` collection) and external comments (`plan.external_comments`) are evaluated
- [ ] Plans with zero comments are excluded from both resolved and unresolved results
- [ ] `PlannerSavedViews` model includes `comment_status` in `$fillable` and `$casts`
- [ ] Migration creates a "Post Discussions" view for every joined workspace team member
- [ ] Migration skips members who already have a "Post Discussions" view (idempotent)
- [ ] Migration processes in chunks of 500 and logs progress every 1000 members
- [ ] Migration `down()` deletes all "Post Discussions" views
- [ ] API endpoint accepts `comment_status` filter parameter and passes it through to the repository

---

### Mock-ups:

N/A — backend only

---

### Impact on existing data:

- New `comment_status` field added to `PlannerSavedViews` model — no impact on existing views (field is optional, defaults to null/unused)
- Migration creates new `planner_saved_views` documents — no modification to existing data
- Filter is additive to `fetchPlans()` — no existing queries are changed when `comment_status` is not provided

---

### Impact on other products:

- **Mobile apps:** Mobile does not currently use planner custom views, so no impact. If mobile adds planner views in the future, they will need to support the `comment_status` field.
- **Chrome extension:** No impact — Chrome extension does not access planner views.
- **White-label:** No impact — this is data/logic only, no branding involved.

---

### Dependencies:

- The comment resolution feature (resolve/unresolve) must be live. The `is_resolved` field on comments and the `CommentResolutionService` must be deployed.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [FE] Add "Post Comments" filter section to planner FilterSidebar

### Description:

Add a new "Post Comments" filter section to the planner's `FilterSidebar.vue` (`contentstudio-frontend/src/modules/planner_v2/components/FilterSidebar.vue`) that lets users filter posts by comment resolution status. Also update the custom views composable to support the new `comment_status` field so the "Post Discussions" custom view (created by the backend migration) works correctly.

**New filter section in `FilterSidebar.vue`:**
- Add a new collapsible section called **"Post Comments"** in the filter sidebar, placed after the existing "Content Categories" section
- The section contains two checkbox options: **"Unresolved"** and **"Resolved"**
- Selecting "Unresolved" sends `comment_status: 'unresolved'` to the API
- Selecting "Resolved" sends `comment_status: 'resolved'` to the API
- Only one can be active at a time (radio-style behavior, not multi-select)
- Clearing the filter removes `comment_status` from the API request

**Update `useCustomViews.js`** (`contentstudio-frontend/src/modules/publisher/composables/useCustomViews.js`):
- Add `comment_status` to `transformViewToFilter()` so that when a saved view has `comment_status: 'unresolved'`, it maps to the filter state correctly
- Add `comment_status` to the filter-to-view mapping when creating/editing custom views
- The "Post Discussions" custom view (migrated from backend) should auto-apply the "Unresolved" filter when selected

**Update custom view creation/editing:**
- In the create/edit custom view flow, add "Post Comments" as a filter option alongside the existing filters (Status, Members, Content Type, Labels, Campaigns, Content Categories, etc.)
- When a user creates a new custom view and selects "Unresolved" or "Resolved" under Post Comments, that `comment_status` value is saved to the view
- When a user edits an existing custom view, the "Post Comments" filter reflects the saved `comment_status` value (if any) and can be changed
- The filter appears in the same style/pattern as other filter options in the custom view form

**UI Copy:**

Section header label: **"Post Comments"**

Filter option labels:
- **"Unresolved"**
- **"Resolved"**

Tooltip for the section header (on `ℹ` icon):
> "Filter posts by their comment status. 'Unresolved' shows posts that have comments still being discussed. 'Resolved' shows posts where all comments have been marked as done."

Empty state when filter returns no results:
> **Headline:** "No posts found"
> **Subtext:** "There are no posts matching the selected comment filter in this view. Try switching between Resolved and Unresolved, or clear the filter to see all posts."

---

### Workflow:

1. User opens the Content Planner and expands the filter sidebar
2. User scrolls to the "Post Comments" section (below Content Categories)
3. User clicks "Unresolved" — the planner refreshes and shows only posts with at least one unresolved comment
4. User sees posts with active discussions, making it easy to find posts that need attention
5. User clicks "Resolved" — the planner refreshes and shows only posts where all comments are resolved
6. User clears the filter to return to the full post list
7. User clicks the "Post Discussions" custom view in the sidebar — the planner switches to Feed view with "Unresolved" auto-selected
8. User can see all posts with active discussions in one centralized view
9. User clicks "Create Custom View" — in the view creation form, the "Post Comments" filter is available alongside Status, Members, Labels, etc.
10. User selects "Unresolved" under Post Comments, names the view, and saves — the new custom view is created with the comment filter baked in
11. User later edits that custom view — the Post Comments filter shows "Unresolved" pre-selected and can be changed or cleared

---

### Acceptance criteria:

- [ ] A new "Post Comments" collapsible section appears in the filter sidebar after "Content Categories"
- [ ] The section contains two options: "Unresolved" and "Resolved"
- [ ] Selecting "Unresolved" filters the planner to show only posts with unresolved comments
- [ ] Selecting "Resolved" filters the planner to show only posts with all-resolved comments
- [ ] Only one option can be active at a time (radio behavior)
- [ ] Clicking the active option deselects it and clears the comment filter
- [ ] The filter state persists when switching between planner view types (Feed, Calendar, etc.)
- [ ] The "Post Discussions" custom view auto-applies the "Unresolved" filter when selected
- [ ] `useCustomViews.js` `transformViewToFilter()` correctly maps `comment_status` from saved views
- [ ] Creating a new custom view shows the "Post Comments" filter option in the view form
- [ ] User can select Unresolved or Resolved in the custom view form and it saves to the view's `comment_status`
- [ ] Editing an existing custom view shows the saved `comment_status` value pre-selected under Post Comments
- [ ] Editing a custom view allows changing or clearing the Post Comments filter
- [ ] Section header has an info tooltip with the specified copy
- [ ] When the filter returns no results, the empty state shows with the specified copy
- [ ] No hardcoded color values — uses `primary-cs-*` theme classes for any primary-colored elements
- [ ] Filter section uses existing component patterns from sibling filter sections in `FilterSidebar.vue`

---

### Mock-ups:

N/A — follow the existing filter section pattern in `FilterSidebar.vue`

---

### Impact on existing data:

- No data changes — this is purely a frontend filter UI addition
- Existing custom views are unaffected (they don't have `comment_status`, so it defaults to null/no filter)

---

### Impact on other products:

- **Mobile apps:** Mobile does not use the planner filter sidebar. No impact.
- **Chrome extension:** No impact.
- **White-label:** Must use theme-aware classes (`primary-cs-*`) for any primary-colored elements. No hardcoded brand colors.

---

### Dependencies:

- Depends on: **[BE] Add comment status filter and "Post Discussions" custom view migration** — the API must accept and process `comment_status` before the frontend can use it.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
