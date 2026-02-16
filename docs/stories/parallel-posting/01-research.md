# Research: Parallel Posting for Multi-Platform Publishing

## Current State

### Publishing Flow (Sequential)
When a post is scheduled/published from the Composer, the flow is:

1. **`PlanPostingJob`** (`app/Jobs/PlanPostingJob.php`) is dispatched as a queued job
2. It calls `Posting::processPlanPosting($plan)` (`app/Libraries/Publish/Posting/Posting.php:11`) which delegates to `SocialPosting::processSocialPosting($plan)`
3. **`SocialPosting::processSocialPosting()`** (`app/Libraries/Publish/Posting/SocialPosting.php:23-68`) calls each platform **sequentially** in a fixed order:
   - `FacebookPlatform::performPosting()` → `InstagramPlatform::performPosting()` → `TwitterPlatform::performPosting()` → `LinkedinBuilder` → `PinterestPlatform::performPosting()` → Tumblr → GMB → YouTube → TikTok → Threads → Bluesky
   - Each platform call blocks until it completes before the next one starts
   - Results are merged into a single `$posting_response` array via `array_merge()`

4. Back in `PlanPostingJob::parsePosingResponse()`, posting details are saved and `performPlanDependingOperation()` determines the final status:
   - `Posting::planPostingStatus($posting_response)` — returns `"published"` if at least one response has a `link`, otherwise `"failed"`
   - `Posting::partialPlanPostingStatus($posting_response)` — returns `true` if some succeeded and some failed (the `partially_failed` flag)
   - Final plan status is set to `"published"` or `"failed"`, with a `partially_failed` boolean flag

### Status Model
- **Plans model** (`app/Models/Publish/Planner/V2/Plans.php`) has a `partially_failed` field in its `$fillable` array
- The plan `status` field takes values: `scheduled`, `processing`, `published`, `failed`, `draft`, `rejected`, `under_review`
- "Partially Failed" is not a separate status value — it's `status: "published"` + `partially_failed: true`
- The broadcast event in `PlanPostingJob::broadcastEvent()` (line 553-591) sets the notification title to `'Partially Failed'` when `partially_published_status === true`

### Planner Display (Frontend)
- The `partially_failed` flag is already used in multiple planner components (`DataRow.vue`, `CalendarItemPost.vue`, `AccountDetailItem.vue`, etc.) to show a "Partially Failed" visual indicator
- The status rendering in `Plans.php:200-215` maps statuses to icons and tooltip text

## What Needs to Change

### Backend — `SocialPosting::processSocialPosting()`
This is the **core change**. Currently lines 46-64 call each platform sequentially. This must be refactored to:
- Build an array of platform posting callables/closures (one per selected platform)
- Execute all of them in parallel (using PHP `pcntl_fork`, `parallel`, Laravel's `Bus::batch()`, or async HTTP via `Http::pool()` / `GuzzleHttp\Promise`)
- Collect all responses after all complete
- Merge into the same `$posting_response` format the rest of the system expects

### No Changes Needed
- **Status logic:** The existing `planPostingStatus()` and `partialPlanPostingStatus()` methods already handle mixed success/failure responses correctly. The `partially_failed` field already exists on the Plans model. No status-related changes are needed.
- **Frontend / Planner:** The Planner already displays Published, Failed, and Partially Failed states. No FE changes are required.
- **`PlanPostingJob`:** The job already calls `processPlanPosting()` and handles the response array — it doesn't care whether the platform calls were sequential or parallel.

## Files Involved

### Backend (changes required)
- `app/Libraries/Publish/Posting/SocialPosting.php` — refactor `processSocialPosting()` to run platform postings in parallel

### Backend (for reference, no changes)
- `app/Jobs/PlanPostingJob.php` — dispatches and handles posting response (unchanged)
- `app/Libraries/Publish/Posting/Posting.php` — status determination logic (unchanged)
- `app/Models/Publish/Planner/V2/Plans.php` — already has `partially_failed` field (unchanged)
