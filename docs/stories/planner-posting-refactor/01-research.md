# Research — Planner Posting API Refactor & Test Coverage

## Current State

The posting pipeline is split across two parallel namespaces and has near-zero test coverage.

### Posting Pipeline Architecture

**Entry point:** `app/Jobs/PlanPostingJob.php`
- Dispatches to `App\Libraries\Publish\Posting\Posting::processPlanPosting($plan)`

**Orchestration layer** (`app/Libraries/Publish/Posting/`):
- `Posting.php` — routes to `BlogPosting` or `SocialPosting` based on plan type
- `SocialPosting.php` — calls each platform sequentially via static methods (FacebookPlatform, InstagramPlatform, TwitterPlatform, LinkedinBuilder, PinterestPlatform, etc.)
- `BlogPosting.php` — handles blog/CMS posting

**Strategy/platform layer** (`app/Strategy/Planner/`):
- `Posting.php` — switch-case dispatcher: instantiates platform-specific posting classes by string type
- `PostingInterface.php` — defines `initializePosting()`, `parsePostMentions()`, `performPosting()`, `postingResponse()`
- Platform classes: `FacebookPosting`, `TwitterPosting`, `InstagramPosting`, `GmbPosting`, `YoutubePosting`, `TikTokPosting`, `ThreadsPosting`, `BlueskyPosting`
- **Problem:** Only `TikTokPosting`, `GmbPosting`, `ThreadsPosting`, `YoutubePosting`, `BlueskyPosting` implement `PostingInterface`. `FacebookPosting`, `TwitterPosting`, `InstagramPosting` do not.
- **Problem:** The `Strategy\Planner\Posting` dispatcher is a raw switch-case with no factory abstraction — adding a new platform requires editing the dispatcher directly.

**Related jobs:**
- `PlanPostingJob.php` — main posting job
- `RetryPostingJob.php` — retry logic
- `UpdatePlanPostingJob.php` — update flow
- `PlanRemovePostingJob.php` — removal flow

### Test Coverage

Near zero for the posting pipeline:
- `tests/Old/Planner/Unit/Posting/PostingTest.php` — 1 test, in `tests/Old/`, essentially a no-op (just checks Redis and asserts `true`)
- No tests in `tests/Unit/` or `tests/Feature/` for any posting class
- No Pest tests for `SocialPosting`, `BlogPosting`, `PlanPostingJob`, `RetryPostingJob`, or any platform posting strategy

## What Needs to Change

**Story 1 — Refactor:**
- Make all platform posting classes implement `PostingInterface` (`FacebookPosting`, `TwitterPosting`, `InstagramPosting`)
- Replace the switch-case in `Strategy\Planner\Posting.php` with a proper factory/registry pattern
- Reduce static method usage in `SocialPosting.php` to enable testability (dependency injection)
- Ensure `PlanPostingJob` uses constructor injection where possible

**Story 2 — Test Coverage:**
- Write Pest feature/unit tests for the full posting pipeline targeting ≥90% code coverage
- Cover: `Posting`, `SocialPosting`, `BlogPosting`, `PlanPostingJob`, `RetryPostingJob`, and key platform strategy classes
- Move/retire the old test in `tests/Old/`

## Files Involved

| File | Change |
|---|---|
| `app/Strategy/Planner/Posting.php` | Replace switch-case with factory pattern |
| `app/Strategy/Planner/FacebookPosting.php` | Implement `PostingInterface` |
| `app/Strategy/Planner/TwitterPosting.php` | Implement `PostingInterface` |
| `app/Strategy/Planner/InstagramPosting.php` | Implement `PostingInterface` |
| `app/Libraries/Publish/Posting/SocialPosting.php` | Reduce static coupling for testability |
| `app/Jobs/PlanPostingJob.php` | Constructor injection improvements |
| `tests/Unit/Planner/Posting/` (new) | Pest unit tests |
| `tests/Feature/Planner/Posting/` (new) | Pest feature tests |

## Story Split

2 `[BE]` stories → create a dedicated epic.
