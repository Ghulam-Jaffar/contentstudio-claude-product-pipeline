# Research: Real-Time Token Expiry Detection

## Current State

### Scheduled token validation (daily — currently PAUSED)
- `ValidateConnectedAccountsCommand` (`artisan validate:accounts`) — scheduled command that chunks through Facebook and LinkedIn accounts and dispatches `SocialAccountsJob` per account
- **This command is entirely paused** — all code is commented out (lines 52–70)
- `SocialAccountsJob` — validates tokens via Facebook Graph debug_token API and LinkedIn introspect API. Also paused (`return false` at line 51)
- When it was active, it set `validity` to `'expired'` or `'expiring_soon'` on the account model and sent notifications via `LUMOTIVE_EMAILS_API`

### Account validity field
- All social account models (Facebook, Instagram, LinkedIn, Twitter, Pinterest, GMB, etc.) have a `validity` field
- Values: `'valid'`, `'expired'`, `'expiring_soon'`, `'invalid'`
- Repos filter on validity: `SocialRepo`, `FacebookRepo`, `InstagramRepo`, `LinkedinRepo`, `TwitterRepo`, `PinterestRepo` all support `validity` filter

### Token error handling during posting
- **LinkedIn only**: `LinkedinBuilder::checkTokenValidity()` (line 341) validates token before posting, and if expired, tries refresh token. If that fails too, sets `validity` to `'expired'` on the account (line 396). Error message: "Token expired, you will need to reconnect your account to avoid any errors."
- **Other platforms (Facebook, Instagram, Twitter, Pinterest, TikTok, Threads, Bluesky, Tumblr, YouTube, GMB)**: Do **NOT** detect token expiry during posting or update the `validity` field. They return generic error messages when the API call fails.
- The `posting_response` array includes `error: true` and `error_message` with the platform's API error text — which often contains token-related phrases like "Error validating access token", "OAuthException", "invalid_token", "The token used in the request has expired", etc.

### What happens after posting failure
- `PlanPostingJob::performPlanDependingOperation()` updates plan status to `'failed'` or `'published'` with `partially_failed`
- Broadcasts an event for real-time UI update (Published / Failed / Partially Failed notifications)
- **No check is made** on whether the failure was token-related — the error is just stored and the user sees "Failed" with the raw error message

### Frontend token-expired awareness
- `AccountSelectionAside.vue`: Disables account selection when `validity === 'expired'` (via `disabledAccountIds`)
- `SocialAccountsDatatable.vue`: Shows "Token Expired" badge with reconnect prompt in Settings → Social Accounts
- `HeadNotificationSlider.vue`: Shows header notification when accounts have expired tokens
- `CstAlert` component: Has a `type="token-expired"` variant used across Analytics, Automation, and Integration modules
- Composer (`SocialModal.vue`, line 5604): Already checks if account token is valid before allowing selection

## What Needs to Change

### Backend
1. **Detect token-expiry errors from posting responses** — After `SocialPosting::processSocialPosting()` returns, inspect each failed platform's `error_message` for known token-expiry patterns (e.g., "Error validating access token", "OAuthException", "invalid_token", "expired", "unauthorized", "Session has expired", "Invalid access token")
2. **Mark accounts as expired immediately** — When a token-expiry error is detected in the posting response, update the account's `validity` field to `'expired'` in the database
3. **Send real-time notification** — Trigger the existing `account_expired` notification flow so the user is alerted immediately
4. **Prevent further scheduling** — Expired accounts should already be filtered out by the Composer's existing `disabledAccountIds` check, but ensure the posting pipeline also skips accounts with `validity === 'expired'`

### Frontend
5. **No major FE changes needed** — The existing token-expired UI (disabled accounts in Composer, header notifications, reconnect prompts in Settings) will kick in automatically once the backend updates the `validity` field. The real-time notification via broadcast events will surface in the header notification slider.

## Files Involved

### Backend
- `app/Jobs/PlanPostingJob.php:81-135` — `parsePosingResponse()` and `performPlanDependingOperation()` — add token-expiry detection after posting
- `app/Libraries/Publish/Posting/SocialPosting.php` — consider adding detection here or in PlanPostingJob
- `app/Libraries/Publish/Posting/Posting.php:135-145` — `planPostingErrorMessage()` — error message parsing
- `app/Repository/Integrations/Platforms/SocialRepo.php` — update account validity
- `app/Repository/Integrations/Platforms/Social/FacebookRepo.php` — update validity
- `app/Repository/Integrations/Platforms/Social/InstagramRepo.php` — update validity
- `app/Repository/Integrations/Platforms/Social/LinkedinRepo.php` — already has this for LinkedIn
- `app/Repository/Integrations/Platforms/Social/TwitterRepo.php` — update validity
- `app/Repository/Integrations/Platforms/Social/PinterestRepo.php` — update validity
- `app/Jobs/Integrations/Validate/SocialAccountsJob.php:261-332` — existing notification flow (reuse `notify()` pattern)
