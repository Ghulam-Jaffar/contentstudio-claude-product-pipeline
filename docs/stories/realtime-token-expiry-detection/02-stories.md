# Stories: Real-Time Token Expiry Detection

---

## [BE] Detect token expiry from posting failures and immediately flag accounts as expired

### Description:

When a scheduled or immediate post fails due to an expired/invalid token, the system should automatically detect this from the platform API error response, mark the account as `validity: 'expired'`, and send a real-time notification — so the user knows immediately instead of discovering it from repeated failed posts.

Currently, the only platform that does this is LinkedIn (`LinkedinBuilder::checkTokenValidity()` at line 341 of `app/Builders/Integrations/Platforms/Social/LinkedinBuilder.php`). All other platforms return generic error messages without updating the account's validity. The scheduled validation system (`ValidateConnectedAccountsCommand` / `SocialAccountsJob`) is paused entirely.

**1. Create a token-expiry detection service:**

Create a method (e.g., `Posting::detectAndHandleTokenExpiry($posting_response)` in `app/Libraries/Publish/Posting/Posting.php`) that inspects each entry in the `$posting_response` array after posting completes. For each entry where `error === true`, check the `error_message` against a list of known token-expiry patterns per platform:

**Known token-expiry error patterns:**
| Platform | Error patterns to match (case-insensitive) |
|----------|---------------------------------------------|
| Facebook | `"Error validating access token"`, `"OAuthException"`, `"Session has expired"`, `"Invalid OAuth access token"` |
| Instagram | `"Error validating access token"`, `"OAuthException"`, `"Invalid OAuth access token"`, `"The user has not authorized application"` |
| Twitter/X | `"Invalid or expired token"`, `"Could not authenticate you"`, `"unauthorized"` (401 status) |
| LinkedIn | Already handled by `LinkedinBuilder::checkTokenValidity()` — skip or keep as-is |
| Pinterest | `"Authorization failed"`, `"Access token is not valid"`, `"invalid_token"` |
| TikTok | `"access_token is invalid"`, `"Token has been revoked"`, `"invalid_access_token"` |
| YouTube/GMB | `"Invalid Credentials"`, `"Token has been expired or revoked"`, `"UNAUTHENTICATED"` |
| Threads | Same as Instagram (uses Facebook Graph API) |
| Bluesky | `"ExpiredToken"`, `"InvalidToken"`, `"Authentication Required"` |
| Tumblr | `"401 Not Authorized"`, `"Invalid OAuth token"` |

Store these patterns in a config file (e.g., `config/tokenExpiryPatterns.php`) so they can be updated without code changes.

**2. Mark account as expired:**

When a token-expiry pattern is matched for a failed posting entry, update the corresponding social account's `validity` field to `'expired'` in the database. Use the `platform_id` and `platform_type` from the posting response to look up the account. Each platform has its own Repo class (`FacebookRepo`, `InstagramRepo`, `TwitterRepo`, `LinkedinRepo`, `PinterestRepo`, `GoogleMyBusinessRepo`, etc.) with an existing `updateItem()` method.

**3. Send notification:**

After marking an account as expired, send the existing `account_expired` notification using the same pattern from `SocialAccountsJob::notify()` (lines 261–332 of `app/Jobs/Integrations/Validate/SocialAccountsJob.php`). This sends a notification via `LUMOTIVE_EMAILS_API . 'notifications/accounts/expired'` to workspace admins and the account owner, which triggers:
- In-app header notification (via `HeadNotificationSlider.vue`)
- Email notification to workspace admins

Extract the notification logic from `SocialAccountsJob` into a reusable service or helper so both the scheduled validation (if re-enabled) and real-time detection can use it.

**4. Integrate into posting pipeline:**

Call `Posting::detectAndHandleTokenExpiry($posting_response)` from `PlanPostingJob::parsePosingResponse()` (line 81 of `app/Jobs/PlanPostingJob.php`), after `Posting::savePostingDetails($posting_response)` and before `performPlanDependingOperation()`. This ensures:
- The posting details are already saved (so the user can see what failed)
- The account is flagged before the next scheduled post attempts to use it
- The notification fires while the user is still likely looking at the result

**5. Skip expired accounts during posting:**

In `SocialPosting::processSocialPosting()` (`app/Libraries/Publish/Posting/SocialPosting.php`), before calling each platform's posting method, check if the account's `validity` is `'expired'`. If so, skip posting and add an error entry to `$posting_response` with message: "This account's token has expired. Please reconnect the account in Settings → Social Accounts." This prevents wasted API calls and faster feedback for posts targeting multiple platforms where one account was just flagged expired by a previous post.

---

### Workflow:

1. User schedules a post targeting Facebook, Instagram, and LinkedIn
2. The scheduled time arrives and the system begins publishing
3. The Facebook API returns an error: "Error validating access token: Session has expired"
4. Instagram and LinkedIn publish successfully
5. The system detects that the Facebook error matches a token-expiry pattern
6. The system immediately marks the Facebook account as `validity: 'expired'`
7. The user receives a notification: "Your Facebook token expired! — [Account Name] token has expired. Account synchronization, scheduled publishing, analytics and notifications have been stopped."
8. The user sees the post as "Partially Failed" in the Planner, with the Facebook error visible
9. In the Composer, the Facebook account now appears disabled with a "Token Expired" indicator
10. The user goes to Settings → Social Accounts, sees the reconnect prompt, and reconnects the Facebook account
11. After reconnection, the account's `validity` is reset to `'valid'` and future posts publish normally

---

### Acceptance criteria:

- [ ] When a post fails on a platform due to a token-expiry error, the system automatically sets that account's `validity` to `'expired'`
- [ ] Token-expiry error patterns are configurable via a config file (`config/tokenExpiryPatterns.php`), not hardcoded in business logic
- [ ] The system correctly detects token-expiry errors for Facebook, Instagram, Twitter/X, Pinterest, TikTok, YouTube, GMB, Threads, Bluesky, and Tumblr
- [ ] LinkedIn's existing token validation in `LinkedinBuilder::checkTokenValidity()` is not affected or duplicated
- [ ] After marking an account as expired, the system sends an `account_expired` notification to workspace admins and the account owner
- [ ] The notification uses the existing `LUMOTIVE_EMAILS_API . 'notifications/accounts/expired'` endpoint with the same payload format as `SocialAccountsJob`
- [ ] Subsequent scheduled posts skip accounts with `validity === 'expired'` and return an error entry: "This account's token has expired. Please reconnect the account in Settings → Social Accounts."
- [ ] If multiple platforms fail with token-expiry errors in the same post, each account is independently flagged as expired
- [ ] Non-token errors (e.g., "Media type not supported", "Rate limit exceeded") do NOT trigger the expiry detection
- [ ] The detection runs after posting details are saved, so the user can still see the original error in the Planner
- [ ] When the user reconnects an account, the `validity` field is reset to `'valid'` (verify this is already the case in the reconnection flow)
- [ ] Single-platform posts that fail due to token expiry also trigger the detection and notification
- [ ] Blog posting flow (`BlogPosting::processBlogPosting()`) is not affected

---

### Mock-ups:

N/A — backend only. No UI changes required. The frontend already handles `validity === 'expired'` with disabled accounts in the Composer, "Token Expired" badges in Settings → Social Accounts, and header notifications.

---

### Impact on existing data:

No schema changes. The `validity` field already exists on all social account models with values `'valid'`, `'expired'`, `'expiring_soon'`, `'invalid'`. This story only changes when and how `validity` gets set to `'expired'` — adding a new trigger (posting failure) alongside the existing trigger (scheduled validation, currently paused).

---

### Impact on other products:

- **Mobile apps:** No impact — mobile apps display account status from the same `validity` field. Once an account is marked expired, mobile will reflect this on next data fetch.
- **Chrome extension:** No impact.
- **White-label:** No impact.
- **Automation posts (Evergreen, RSS, Bulk CSV):** These also flow through `PlanPostingJob` → `SocialPosting::processSocialPosting()`, so they will automatically benefit from real-time token expiry detection as well.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
