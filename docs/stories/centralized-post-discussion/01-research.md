# Centralized Post Discussion — Research

## Current State

### Comment Resolution System (Backend)
- **`Comment` model** (`app/Models/Publish/Planner/Comment.php`) — MongoDB `comment` collection
  - Fields: `is_resolved` (default `false`), `resolved_by`, `resolved_at`
- **`CommentResolutionService`** (`app/Services/Comments/CommentResolutionService.php`) — Full resolve/unresolve system
  - `toggleResolve()` — toggle single comment
  - `toggleExternalCommentResolve()` — toggle external comment (stored in `plan.external_comments` array)
  - `resolveAllForPlan()` / `unresolveAllForPlan()` — bulk resolve/unresolve for a plan
- **`PlannerViewService::getCommentCount()`** (lines 524–568) — already excludes resolved comments from counts using MongoDB aggregation (`is_resolved !== true`)

### Custom Views System
- **`PlannerSavedViews` model** (`app/Models/Publish/Planner/PlannerSavedViews.php`) — MongoDB `planner_saved_views` collection
  - Fillable: `name`, `workspace_id`, `user_id`, `types`, `statuses`, `members`, `created_by`, `approval_requested_by`, `approval_assigned_to`, `labels`, `campaigns`, `content_categories`, `platform_selections`, `platform_accounts`, `view_type`, `visibility`, `sort_order`, `date_range`, `calendar_settings`, `order`
  - Does NOT currently have a `comment_status` field
- **Migration pattern** (`database/migrations/2026_01_21_100000_create_approval_custom_views.php`) — creates "My Pending Approvals" / "My Approval Requests" views per joined workspace team member, chunked by 500

### Frontend
- **`FilterSidebar.vue`** (`contentstudio-frontend/src/modules/planner_v2/components/FilterSidebar.vue`)
  - Current filter sections: Status, Assigned To, Created By, Content Type, Labels, Campaigns, Content Categories
  - **No comment/discussion filter exists yet**
- **`useCustomViews.js`** (`contentstudio-frontend/src/modules/publisher/composables/useCustomViews.js`)
  - Full CRUD for custom views: list, create, delete, toggle default, reorder
  - `transformViewToFilter()` maps saved view fields to filter format
  - `navigateToCustomView()` applies view type route + filters via EventBus

### Plans Repository
- **`PlansRepository::fetchPlans()`** (`app/Repository/Publish/Planner/PlansRepository.php`, line 164)
  - **No comment-based filtering exists yet** — needs new filter parameter

## What Needs to Change

### Backend
1. **Add `comment_status` filter to `PlansRepository::fetchPlans()`**
   - When `comment_status = 'unresolved'`: return plans that have at least one comment with `is_resolved !== true` (or null)
   - When `comment_status = 'resolved'`: return plans where ALL comments have `is_resolved === true`
   - Must consider both regular comments (in `comment` collection) and external comments (in `plan.external_comments` array)
2. **Add `comment_status` field to `PlannerSavedViews` model** — add to `$fillable` and `$casts`
3. **Create migration** to create "Post Discussions" custom view for every joined workspace team member
   - Follow the approval custom views migration pattern (chunk by 500, check for existing, logging)
   - View defaults: `view_type: 'Feed'`, `comment_status: 'unresolved'`, `visibility: 'Only me'`
4. **Wire `comment_status` filter** through the API controller/service layer so it reaches `fetchPlans()`

### Frontend
1. **Add "Post Comments" filter section** to `FilterSidebar.vue`
   - Two options: "Resolved" and "Unresolved"
   - Behaves like other filter sections (checkbox-style selection)
2. **Support `comment_status` in custom views** — update `useCustomViews.js` `transformViewToFilter()` and `CreateCustomView.vue`
3. **The new "Post Discussions" custom view** will appear automatically in the sidebar (migrated from backend)

## Files Involved

### Backend
- `app/Repository/Publish/Planner/PlansRepository.php` — add comment_status filter logic
- `app/Models/Publish/Planner/PlannerSavedViews.php` — add `comment_status` to fillable/casts
- `app/Services/PlannerViewService.php` — wire comment_status filter parameter
- `app/Http/Controllers/` (planner controller) — accept comment_status from request
- `database/migrations/xxxx_create_post_discussions_custom_view.php` — new migration

### Frontend
- `src/modules/planner_v2/components/FilterSidebar.vue` — new "Post Comments" filter section
- `src/modules/publisher/composables/useCustomViews.js` — support comment_status in view↔filter transform
- Custom view creation components — support comment_status field
