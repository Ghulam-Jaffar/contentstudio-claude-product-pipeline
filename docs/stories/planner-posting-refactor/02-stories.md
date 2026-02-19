# Stories — Planner Posting API Refactor & Test Coverage

---

## [BE] Refactor Planner posting pipeline — strategy pattern and interface compliance

### Description:

The Planner posting pipeline has accumulated significant technical debt. The `Strategy\Planner\Posting` dispatcher uses a raw `switch/case` to instantiate platform classes, three major platform posting classes (`FacebookPosting`, `TwitterPosting`, `InstagramPosting`) do not implement `PostingInterface`, and `SocialPosting` relies heavily on static method calls that make the pipeline untestable in isolation. This story refactors the architecture to be consistent, extensible, and testable.

**What needs to be done:**

1. **`app/Strategy/Planner/Posting.php`** — Replace the `switch/case` dispatcher with a factory/registry pattern (e.g., a static map of `type => class`) so adding a new platform no longer requires editing the dispatcher directly

2. **`app/Strategy/Planner/FacebookPosting.php`**, **`TwitterPosting.php`**, **`InstagramPosting.php`** — Implement `PostingInterface` (`initializePosting()`, `parsePostMentions()`, `performPosting()`, `postingResponse()`) to match the contract already followed by TikTok, GMB, Threads, YouTube, and Bluesky

3. **`app/Libraries/Publish/Posting/SocialPosting.php`** — Reduce static coupling where it blocks testability; inject dependencies via constructor or method parameters rather than resolving globally inside static methods

4. **`app/Jobs/PlanPostingJob.php`** — Review and apply constructor property promotion and explicit type hints per Laravel 10 / PHP 8.3 conventions; ensure the job delegates cleanly to the refactored posting classes

5. **No behavior changes** — All refactoring must be functionally equivalent. The posting output, logging, and error handling must remain identical.

---

### Workflow:

This is a backend-only refactor with no user-visible changes. From a developer's perspective:

1. Developer adds a new social platform posting class
2. Instead of editing `Strategy\Planner\Posting.php`'s switch block, they register the class in the factory map
3. All platform classes are guaranteed to implement `PostingInterface` — IDE and static analysis can catch missing method implementations at development time
4. Unit tests can mock individual platform classes in isolation without coupling to static state

---

### Acceptance criteria:

- [ ] `Strategy\Planner\Posting.php` uses a factory/registry pattern (map of type → class) instead of a switch/case block
- [ ] `FacebookPosting`, `TwitterPosting`, and `InstagramPosting` implement `PostingInterface`
- [ ] All 8 platform posting classes implement `PostingInterface`
- [ ] `SocialPosting` static method dependencies are reduced to allow mocking in tests
- [ ] `PlanPostingJob` uses PHP 8.3 constructor property promotion and explicit return types
- [ ] A full posting cycle (schedule → dispatch → post → log response) works identically to before the refactor — no behavioral regression
- [ ] `vendor/bin/sail bin pint --dirty` passes with no formatting errors

---

### Mock-ups:

N/A — backend refactor, no UI.

---

### Impact on existing data:

None. Pure code restructuring — no schema, model, or API contract changes.

---

### Impact on other products:

- **Mobile apps / Chrome extension / Frontend:** Not affected — posting is a backend-only job pipeline.
- **White-label:** Not affected.

---

### Dependencies:

None. This story can be started independently.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (backend-only)
- [ ] Multilingual support — N/A (backend-only, no user-facing strings)
- [ ] UI theming support — N/A (backend-only)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

## [BE] Achieve ≥90% test coverage for the Planner posting pipeline

### Description:

The Planner posting pipeline currently has near-zero automated test coverage — the only existing test is a legacy no-op in `tests/Old/Planner/Unit/Posting/PostingTest.php`. This story adds comprehensive Pest tests across the full posting pipeline targeting ≥90% code coverage.

**Scope of tests to write:**

1. **`tests/Unit/Planner/Posting/PostingTest.php`** — Unit tests for `App\Libraries\Publish\Posting\Posting`:
   - Routes to `BlogPosting` when `blog_selection` is present
   - Routes to `SocialPosting` when no `blog_selection`
   - `savePostingDetails()` persists each response via `PostingRepository`
   - `planPostingStatus()` returns `published` / `failed` / `partially_failed` correctly

2. **`tests/Unit/Planner/Posting/SocialPostingTest.php`** — Unit tests for `SocialPosting`:
   - Trial plan limit reached → returns failed response without calling platforms
   - Hashtag and Replug selection applied before dispatching
   - Each platform's `performPosting()` is called with the correct plan payload
   - Exception during platform posting is caught and logged correctly

3. **`tests/Unit/Planner/Posting/BlogPostingTest.php`** — Unit tests for `BlogPosting`:
   - Correct platform builder is invoked based on blog type
   - Error handling when blog post fails

4. **`tests/Feature/Planner/PlanPostingJobTest.php`** — Feature tests for `PlanPostingJob`:
   - Plan status updated to `processing` before posting begins
   - Successful posting sets plan status to `published`
   - Failed posting sets plan status to `failed`
   - Exception is caught, logged, and does not crash the job
   - Only runs on `attempts() == 1` (no double-processing)

5. **`tests/Unit/Planner/Posting/Strategy/`** — Unit tests for platform strategy classes:
   - Each platform class implements `PostingInterface`
   - `initializePosting()`, `performPosting()`, `postingResponse()` return expected shapes
   - Error/edge cases per platform (e.g., missing account credentials, API timeout mock)

6. **Retire old test:** Move or delete `tests/Old/Planner/Unit/Posting/PostingTest.php` — the test is non-functional and misleading.

**Coverage target:** ≥90% line coverage across all files in `app/Libraries/Publish/Posting/`, `app/Strategy/Planner/`, and `app/Jobs/PlanPostingJob.php`.

---

### Workflow:

Backend-only — no user-visible changes. A developer running `vendor/bin/sail artisan test --compact` against the posting pipeline files should see all tests green and coverage ≥90%.

---

### Acceptance criteria:

- [ ] Pest tests exist in `tests/Unit/Planner/Posting/` covering `Posting`, `SocialPosting`, and `BlogPosting`
- [ ] Pest feature tests exist in `tests/Feature/Planner/` covering `PlanPostingJob` happy path, failure path, and exception handling
- [ ] Pest unit tests exist for each platform strategy class in `app/Strategy/Planner/`
- [ ] All new tests pass with `vendor/bin/sail artisan test --compact`
- [ ] Code coverage for `app/Libraries/Publish/Posting/`, `app/Strategy/Planner/`, and `app/Jobs/PlanPostingJob.php` is ≥90% line coverage
- [ ] The legacy test in `tests/Old/Planner/Unit/Posting/PostingTest.php` is retired (moved or deleted)
- [ ] No existing passing tests are broken

---

### Mock-ups:

N/A — backend only.

---

### Impact on existing data:

None. Test-only additions.

---

### Impact on other products:

Not affected.

---

### Dependencies:

Should be worked on after (or in parallel with) **[BE] Refactor Planner posting pipeline — strategy pattern and interface compliance**, as the refactored code is easier to test and the test structure should reflect the refactored architecture.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (backend-only)
- [ ] Multilingual support — N/A (backend-only)
- [ ] UI theming support — N/A (backend-only)
- [ ] White-label domains impact review — N/A
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
