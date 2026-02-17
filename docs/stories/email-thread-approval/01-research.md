# Group Approval Emails into Threads — Research & Rationale

## The Problem

When a post goes through ContentStudio's approval workflow, multiple emails are sent:
- "A post was sent to you for review" (pending)
- "Your post was approved" / "Your post was rejected"
- If revised and resubmitted, the cycle repeats

Currently, **every one of these emails appears as a separate item** in the user's inbox. For agencies managing dozens of posts daily, this creates a flood of disconnected emails. Users have to manually search for related emails about the same post, making it easy to lose track of approval conversations.

## Why This Happens

Email clients (Gmail, Outlook, Apple Mail) group emails into conversation threads using **RFC 2822 headers**:
- **`Message-ID`** — Unique identifier for each email
- **`In-Reply-To`** — Points to the `Message-ID` of the parent email
- **`References`** — Chain of all `Message-ID`s in the conversation thread

ContentStudio's approval emails currently set **none of these headers**. The only header set is `Reply-To`. Without threading headers, email clients have no way to know that "Post approved" relates to the earlier "Post sent for review" email.

## Current Email Architecture

| Component | Implementation | File |
|---|---|---|
| Approval notification logic | `ApprovalNotification.php` | `app/Notifications/Approval/ApprovalNotification.php` |
| Email dispatch | `Helper::sendMail()` via Postmark (Laravel Mail) | `app/Libraries/Helper.php` (lines 174-223) |
| Email job queue | `NotificationEmailsJob` on `NotificationsJob` queue | `app/Jobs/Emails/NotificationEmailsJob.php` |
| Email template engine | `EmailTemplate.php` | `app/Notifications/EmailTemplate.php` |
| Blade template | `request_approval.blade.php` | `resources/views/emails/notifications/post/request_approval.blade.php` |
| Approval observer (trigger) | `ApprovalObserver` on Plan model `saved()` event | `app/Observers/Publish/ApprovalObserver.php` |
| External approver emails | `ExternalApprovalBuilder` with token-based links | `app/Builders/Approval/ExternalApprovalBuilder.php` |
| Notification storage | MongoDB `notifications_approval` collection | `app/Models/Notification/ApprovalNotification.php` |

## The Solution: RFC 2822 Threading Headers

Add invisible email headers so clients auto-thread approval emails per plan. **No changes to email content, templates, subject lines, or visual appearance.**

### How It Works

For every approval email about a given plan, we set:

```
Message-ID: <plan-{planId}-{type}-{timestamp}@contentstudio.io>
In-Reply-To: <plan-{planId}-thread@contentstudio.io>
References: <plan-{planId}-thread@contentstudio.io>
X-Entity-Ref-ID: plan-{planId}
```

- **`Message-ID`** — Unique per email (includes plan ID + notification type + timestamp)
- **`In-Reply-To`** + **`References`** — Both point to a deterministic "thread anchor" ID derived from the plan ID. This is the same for ALL emails about the same plan, which tells email clients they belong to one thread.
- **`X-Entity-Ref-ID`** — Prevents Gmail from de-duplicating emails with similar content (Gmail sometimes collapses similar emails without this).

### Why Thread by Plan ID?

- **Simple and deterministic** — The `plan_id` is available everywhere in the approval flow (observer, notification, external builder). No need to store or look up thread IDs.
- **Correct grouping** — All emails about the same post (pending → approved → rejected → resubmitted → approved) land in one thread. This is exactly what users expect — the conversation is about the post, not about a specific approver.
- **Works for both internal and external approvers** — Both `ApprovalNotification` and `ExternalApprovalBuilder` have access to `plan_id`, so both can generate the same thread anchor.

### Why NOT Thread by Plan + Approver?

- More complex with minimal benefit — users want to see the full approval history for a post in one place
- Would require tracking per-approver thread IDs
- Breaks the mental model of "one post = one conversation"

### What Changes (and What Doesn't)

| Aspect | Changes? | Detail |
|---|---|---|
| Email headers | ✅ Yes | New `Message-ID`, `In-Reply-To`, `References`, `X-Entity-Ref-ID` headers added |
| Email content/body | ❌ No | Zero changes to email HTML, text, or layout |
| Email subject lines | ❌ No | Existing subject lines stay exactly the same |
| Email templates (Blade) | ❌ No | No template modifications |
| `Helper::sendMail()` | ✅ Yes | Add `$headers` parameter to accept custom headers |
| `NotificationEmailsJob` | ✅ Yes | Pass headers through to `Helper::sendMail()` |
| `ApprovalNotification` | ✅ Yes | Compute and attach threading headers |
| `ExternalApprovalBuilder` | ✅ Yes | Compute and attach threading headers |
| Frontend | ❌ No | No frontend changes needed |
| Notification preferences | ❌ No | No changes to who receives emails or when |
| White-label email config | ❌ No | Headers are independent of from/reply-to/branding |

## Files Involved

### Modified
- `app/Libraries/Helper.php` — Add `$headers` parameter to `sendMail()`
- `app/Jobs/Emails/NotificationEmailsJob.php` — Forward headers to `sendMail()`
- `app/Notifications/Approval/ApprovalNotification.php` — Compute threading headers using `plan_id`
- `app/Builders/Approval/ExternalApprovalBuilder.php` — Compute threading headers using `plan_id`

### New
- `app/Helpers/ApprovalThreadingHelper.php` (or similar) — Small helper to generate threading headers from `plan_id` + `type`
