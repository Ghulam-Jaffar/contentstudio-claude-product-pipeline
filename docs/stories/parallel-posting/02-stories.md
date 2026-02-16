# Stories: Parallel Posting for Multi-Platform Publishing

---

## [BE] Refactor social posting to execute platform API calls in parallel

### Description:

Refactor `SocialPosting::processSocialPosting()` in `app/Libraries/Publish/Posting/SocialPosting.php` to execute all platform API calls concurrently instead of sequentially. Currently, lines 46-64 call each platform one after another (Facebook → Instagram → Twitter → LinkedIn → Pinterest → Tumblr → GMB → YouTube → TikTok → Threads → Bluesky), each blocking until complete before the next begins. For posts targeting multiple platforms, this means total publish time = sum of all individual platform response times.

**What to change:**

In `SocialPosting::processSocialPosting()`, replace the sequential `array_merge()` chain with a parallel execution strategy:

1. Build an array of closures — one per platform — that each return their posting response array. Only include platforms that have selected accounts in `$plan['account_selection']`.
2. Execute all closures concurrently using an appropriate parallel mechanism (e.g., Laravel's `Http::pool()` if platform calls use HTTP, `Bus::batch()` with separate jobs per platform, PHP Fibers, or `GuzzleHttp\Promise\Utils::all()`). Choose the approach that best fits the existing platform posting implementations.
3. Wait for all to complete (with a reasonable per-platform timeout to prevent indefinite hangs).
4. Collect and merge all results into the same `$posting_response` array format that `PlanPostingJob::parsePosingResponse()` expects.

**Important constraints:**
- The merged `$posting_response` array must remain in the exact same format — each entry has `type`, `plan_id`, `workspace_id`, `platform_type`, `platform_id`, `platform`, `link`, `posted_id`, `error`, `error_message`, etc. Downstream consumers (`Posting::savePostingDetails()`, `Posting::planPostingStatus()`, `Posting::partialPlanPostingStatus()`, `PlanPostingJob::performPlanDependingOperation()`) must not be affected.
- If a single platform times out or throws an exception, it must not kill the other parallel calls. Each platform should handle its own errors independently and return an error response entry.
- The `$plan` and `$workspace` objects are read-only during posting — no platform should mutate them.
- Hashtag and Replug preprocessing (`applyHashtagSelection`, `applyReplugSelection`) must still happen before all platform calls, as currently.

**Key files:**
- `app/Libraries/Publish/Posting/SocialPosting.php` — primary change: refactor `processSocialPosting()` method
- Individual platform classes (`FacebookPlatform`, `InstagramPlatform`, `TwitterPlatform`, `LinkedinBuilder`, `PinterestPlatform`, and `Posting` builder for Tumblr/GMB/YouTube/TikTok/Threads/Bluesky) — no changes to their interfaces, but verify they are stateless and safe for concurrent execution

---

### Workflow:

1. User opens the Composer and creates a post
2. User selects multiple social platforms (e.g., Facebook, Instagram, LinkedIn, Twitter)
3. User clicks "Publish Now" or schedules the post
4. The system begins publishing to all selected platforms simultaneously instead of one after another
5. The system waits for all platforms to respond
6. If all succeed → the post shows as "Published" in the Planner
7. If some succeed and some fail → the post shows as "Partially Failed" in the Planner, with per-platform error details visible in the post detail view
8. If all fail → the post shows as "Failed" in the Planner
9. The total publish time is now approximately equal to the slowest single platform, not the sum of all platforms

---

### Acceptance criteria:

- [ ] When a post targets multiple platforms, all platform API calls execute concurrently (not sequentially)
- [ ] Total publish time for a multi-platform post is approximately equal to the slowest single platform response time, not the sum of all
- [ ] The `$posting_response` array format is identical to the current format — no downstream consumers need changes
- [ ] If one platform fails or times out, other platforms still complete independently and return their results
- [ ] `Posting::planPostingStatus()` correctly returns `"published"` when at least one platform succeeds
- [ ] `Posting::partialPlanPostingStatus()` correctly returns `true` when some platforms succeed and some fail
- [ ] The plan's `partially_failed` field is correctly set in the database after mixed success/failure
- [ ] Single-platform posts (only one platform selected) continue to work identically to current behavior
- [ ] Blog posting flow (`BlogPosting::processBlogPosting()`) is not affected — only social posting is parallelized
- [ ] Hashtag and Replug preprocessing still runs before all platform calls begin
- [ ] Existing notifications (Published / Partially Failed / Failed to Publish) fire correctly with accurate platform counts
- [ ] No race conditions or shared state mutations between concurrent platform calls
- [ ] Per-platform timeout prevents a single slow platform from blocking the entire post indefinitely

---

### Mock-ups:

N/A — backend only. No UI changes required; the Planner already displays Published, Failed, and Partially Failed states.

---

### Impact on existing data:

No schema changes. The `$posting_response` array format, the plan `status` field values (`published`, `failed`), and the `partially_failed` boolean flag all remain unchanged. Existing published/failed/partially-failed posts in the database are not affected.

---

### Impact on other products:

- **Mobile apps:** No impact — mobile apps display post statuses from the same plan data, and the status values/format are unchanged.
- **Chrome extension:** No impact.
- **White-label:** No impact.
- **Automation posts (Evergreen, RSS, Bulk CSV):** These also dispatch `PlanPostingJob` and flow through the same `SocialPosting::processSocialPosting()` method, so they will automatically benefit from parallel posting as well.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
