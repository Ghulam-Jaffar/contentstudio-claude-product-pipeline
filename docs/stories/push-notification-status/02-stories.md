# Push Notification Status & Handling Improvements — Stories

Epic: #93708 — Push notification status & handling improvements - web/mobile

---

## Status names used across all stories

| Status | Meaning |
|---|---|
| **Notification Sent** | Push notification was sent to user's device; post is awaiting their confirmation |
| **Notification Declined** | User explicitly said they did not publish this post; acts like a draft — editable and reschedulable |

These are distinct from:
- **Failed** — post failed during automated publishing (system error)
- **Rejected** — a reviewer rejected the post in the approval workflow

---

## Story 1 (UPDATE #109136): [BE] Add "Notification Sent" and "Notification Declined" post statuses

### Description:

ContentStudio's push notification publishing flow sends a push to the user's device when it's time to post. Currently there is no way to represent the in-between state (notification sent, waiting for user to confirm) or the declined state (user said they didn't end up posting). This story adds two new post statuses to the system.

**New statuses:**
- `notification_sent` — assigned automatically by the system when a push notification is dispatched for a post. Replaces the existing approach of marking it as scheduled/pending until confirmed.
- `notification_declined` — assigned when the user explicitly says they did not publish the post (via mobile app confirmation flow or web manual resolution). Posts in this status behave like drafts: they remain editable and can be rescheduled. They are excluded from analytics and publishing reports.

**Existing statuses not affected:**
- `failed` remains for system-level publishing errors
- `rejected` remains for approval workflow rejections

**Required changes:**
- Add `notification_sent` and `notification_declined` to the post status enum (backend model)
- Add status transitions:
  - `scheduled` → `notification_sent` (when push is dispatched)
  - `notification_sent` → `published` (user confirms they published)
  - `notification_sent` → `notification_declined` (user says they didn't publish)
  - `notification_declined` → `scheduled` (user reschedules the post)
  - `notification_declined` → `draft` (user saves as draft)
- Include both statuses in all API responses that return post status
- Add both statuses to the post filter/query layer so planner and analytics can filter by them
- Exclude `notification_sent` and `notification_declined` posts from analytics and publishing reports

### Workflow:

1. User schedules a post for an account that requires push notification publishing (e.g., personal Instagram account)
2. At the scheduled time, ContentStudio dispatches the push notification to the user's device
3. System automatically sets post status to **Notification Sent**
4. User either confirms they published (→ Published) or declines (→ Notification Declined) via mobile app or web app
5. If the user reschedules a Notification Declined post, status resets to Scheduled

### Acceptance criteria:

- [ ] `notification_sent` and `notification_declined` are valid values in the post status enum
- [ ] Post status transitions to `notification_sent` when push notification is dispatched
- [ ] Post status transitions from `notification_sent` to `published` when user confirms publishing
- [ ] Post status transitions from `notification_sent` to `notification_declined` when user declines
- [ ] Post status transitions from `notification_declined` to `scheduled` when user reschedules
- [ ] Both statuses are returned in all API responses that include post status fields
- [ ] Both statuses are supported in post list/filter query params (e.g., `?status=notification_sent`)
- [ ] Posts with `notification_sent` or `notification_declined` status are excluded from analytics aggregations
- [ ] Posts with `notification_declined` status retain all original post data (caption, media, account) and remain editable
- [ ] No existing status values (`draft`, `scheduled`, `published`, `failed`, `rejected`) are affected
- [ ] Status transitions are validated server-side (e.g., cannot go directly from `published` to `notification_declined`)

### Mock-ups:

N/A — backend-only story.

### Impact on existing data:

- No migration required for historical posts
- Existing push-published posts that were marked as `published` remain `published`
- New status values only apply to posts created/scheduled after this change is deployed

### Impact on other products:

- Web app must map and display both new statuses (covered in **[FE] Allow manual resolution of push notification posts from web app**)
- iOS app must recognize and display both new statuses (covered in iOS stories)
- Android app must recognize and display both new statuses (covered in Android stories)

### Dependencies:

- None — this is the foundational story; all other stories in this epic depend on it

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, backend only
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used) — N/A, backend only
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 2 (UPDATE #109138): [BE] Create per-account posts for push notification publishing

### Description:

When a user schedules a post for multiple accounts and one or more of those accounts requires push notification publishing, the current system groups them together. This story changes the post creation logic so that any account requiring push notification publishing always generates a **standalone post** — separate from other accounts in the same scheduling action.

This is necessary because push notification confirmation is per-account (e.g., the user may publish to Instagram but not to Facebook), and having them grouped would make it impossible to track confirmation status independently.

**Required changes in post creation/scheduling service:**
- On post creation/scheduling, evaluate the publishing method per account
- If an account requires push notification publishing, create a separate post document for it
- Direct API accounts continue to use the existing grouping behavior
- Ensure the scheduled time is preserved per account in the split
- The split must happen at scheduling time, not at push dispatch time

### Workflow:

1. User creates a post in the Composer and selects multiple accounts — some are direct API accounts, some require push notification
2. User sets a schedule time and clicks Publish/Schedule
3. System evaluates each account's publishing method
4. For accounts requiring push notification: one separate post is created per account
5. For direct API accounts: existing grouping behavior is preserved
6. User sees the posts as separate items in the Planner

### Acceptance criteria:

- [ ] Any account requiring push notification publishing always results in a separate, standalone post
- [ ] Multiple push notification accounts in the same scheduling action result in multiple separate posts (one per account)
- [ ] Direct API accounts are not split unless otherwise required by existing logic
- [ ] No single post contains a mix of push notification and direct API accounts
- [ ] Scheduled time is correctly assigned to each split post
- [ ] Deleting one split post does not affect the others
- [ ] The split logic is applied to newly created/scheduled posts only — no backfill of historical posts
- [ ] Planner correctly renders each push notification post as a separate item

### Mock-ups:

N/A — backend-only story.

### Impact on existing data:

- Applies only to newly created/scheduled posts after deployment
- Historical posts are not affected

### Impact on other products:

- iOS app will receive one push payload per account (no change to push payload structure needed, just one per account)
- Android app will receive one push payload per account (same)

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, backend only
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled) — N/A, backend only
- [ ] UI theming supported (default + white-label, design library components are being used) — N/A, backend only
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 3 (UPDATE #109139 → iOS): [iOS] Allow users to decline push notification posts from iOS app

### Description:

When a user receives a push notification from ContentStudio to publish a post, they should be able to indicate that they did not end up publishing it. This story adds a "I didn't post this" action to the iOS push notification handling screen.

On the iOS push notification screen (the screen shown when the user taps a ContentStudio push notification), a **Decline** button is added alongside the existing "Post Now" action. Tapping Decline:
- Shows a confirmation prompt
- On confirmation, sends a PATCH request to update the post status to `notification_declined`
- The post remains visible in the planner with status **Notification Declined**; the user can edit and reschedule it at any time

### Workflow:

1. User receives a ContentStudio push notification: "Time to post — [Post preview text]"
2. User taps the notification and the ContentStudio iOS app opens to the push notification action screen
3. User sees the post preview, account name, and two CTAs: **Post Now** and **Didn't Post**
4. User taps **Didn't Post**
5. A confirmation prompt appears: "Mark this post as not published? You'll be able to edit and reschedule it from the Planner."
6. User taps **Confirm**
7. Post status updates to **Notification Declined** immediately
8. User sees a success feedback: "Marked as not published. You can reschedule it from the Planner."
9. The push notification action screen is dismissed

### Acceptance criteria:

- [ ] "Didn't Post" CTA is visible on the push notification action screen on iOS
- [ ] Tapping "Didn't Post" shows a confirmation prompt before taking action
- [ ] Confirmation prompt text: "Mark this post as not published? You'll be able to edit and reschedule it from the Planner."
- [ ] Confirmation prompt CTAs: **Confirm** (primary) and **Cancel** (secondary)
- [ ] On Confirm: PATCH request sent to update post status to `notification_declined`
- [ ] On success: post status updates to Notification Declined immediately in the app
- [ ] On success: success feedback shown ("Marked as not published. You can reschedule it from the Planner.")
- [ ] On Cancel: no action taken, user stays on push notification action screen
- [ ] If network request fails: post status remains unchanged; user sees error toast: "Something went wrong. Please try again."
- [ ] Decline action is only available while post is in **Notification Sent** status — not after it's already confirmed or declined
- [ ] Declined post status syncs to web planner on next refresh or in real time if websocket is active
- [ ] Decline action is recorded server-side (audit log / event)

### Mock-ups:

- iOS push notification action screen with "Post Now" and "Didn't Post" CTAs
- Confirmation prompt sheet
  (Figma link to be added)

### Impact on existing data:

No impact on historical posts.

### Impact on other products:

- Web planner reflects **Notification Declined** status after sync
- Covered separately in **[FE] Allow manual resolution of push notification posts from web app**

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native iOS
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 4 (NEW): [Android] Allow users to decline push notification posts from Android app

### Description:

Same scope as **[iOS] Allow users to decline push notification posts from iOS app**, implemented for Android.

When a user receives a ContentStudio push notification on Android and they did not end up publishing the post, they should be able to mark it as not published from the ContentStudio Android app.

On the push notification action screen in the Android app, a **Didn't Post** button is added. Tapping it:
- Shows a confirmation dialog
- On confirmation, sends a PATCH request to update the post status to `notification_declined`
- The post remains editable and reschedulable from the Planner

### Workflow:

1. User receives a ContentStudio push notification on Android: "Time to post — [Post preview text]"
2. User taps the notification; ContentStudio Android app opens to the push notification action screen
3. User sees the post preview, account name, and two CTAs: **Post Now** and **Didn't Post**
4. User taps **Didn't Post**
5. A confirmation dialog appears: "Mark this post as not published? You'll be able to edit and reschedule it from the Planner."
6. User taps **Confirm**
7. Post status updates to **Notification Declined** immediately
8. User sees a success snackbar: "Marked as not published. You can reschedule it from the Planner."
9. The push notification action screen is dismissed

### Acceptance criteria:

- [ ] "Didn't Post" CTA is visible on the push notification action screen on Android
- [ ] Tapping "Didn't Post" shows a confirmation dialog before taking action
- [ ] Confirmation dialog text: "Mark this post as not published? You'll be able to edit and reschedule it from the Planner."
- [ ] Confirmation dialog CTAs: **Confirm** (primary) and **Cancel** (secondary)
- [ ] On Confirm: PATCH request sent to update post status to `notification_declined`
- [ ] On success: post status updates to Notification Declined immediately in the app
- [ ] On success: snackbar shown: "Marked as not published. You can reschedule it from the Planner."
- [ ] On Cancel: no action taken, user stays on push notification action screen
- [ ] If network request fails: post status remains unchanged; user sees error snackbar: "Something went wrong. Please try again."
- [ ] Decline action is only available while post is in **Notification Sent** status
- [ ] Declined post status syncs to web planner on next refresh or in real time if websocket is active
- [ ] Decline action is recorded server-side (audit log / event)

### Mock-ups:

- Android push notification action screen with "Post Now" and "Didn't Post" CTAs
- Confirmation dialog
  (Figma link to be added)

### Impact on existing data:

No impact on historical posts.

### Impact on other products:

- Web planner reflects **Notification Declined** status after sync

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native Android
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 5 (UPDATE #109140 → iOS): [iOS] Confirm push notification publishing outcome on iOS

### Description:

After a user taps **Post Now** on the iOS push notification action screen and opens Instagram or Facebook to post, ContentStudio has no way to know whether they actually completed the post. This story adds a confirmation flow triggered when the user returns to the ContentStudio app after being redirected to an external app.

When the ContentStudio iOS app returns to the foreground from Instagram or Facebook (app lifecycle: applicationDidBecomeActive / sceneDidBecomeActive), if the current session has an unresolved push notification post, a confirmation modal is presented asking whether the user published the post.

### Workflow:

1. User taps **Post Now** on the push notification action screen
2. ContentStudio opens Instagram or Facebook (external app)
3. User returns to ContentStudio (via app switcher, back button, or direct tap)
4. ContentStudio detects foreground return after an external app session
5. A confirmation modal appears:
   - **Title:** "Did you publish this post?"
   - **Subtext:** Shows the post account name and preview text (first ~100 chars of caption)
   - **CTA 1 (primary):** "Yes, I Published It"
   - **CTA 2 (secondary):** "No, I Didn't Post It"
6. User taps **"Yes, I Published It"** → post status updates to **Published** → success feedback shown → modal dismissed
7. User taps **"No, I Didn't Post It"** → post status updates to **Notification Declined** → user sees: "No problem. You can edit and reschedule this post from the Planner." → modal dismissed
8. If user dismisses modal without tapping either CTA: post remains in **Notification Sent** status; modal is shown again next time the app is opened with an unresolved push notification post

### Acceptance criteria:

- [ ] Confirmation modal appears when ContentStudio iOS app returns to foreground after an external app session, if a push notification post is unresolved (status: Notification Sent)
- [ ] Modal shows account name and a preview of the post caption
- [ ] Modal title: "Did you publish this post?"
- [ ] CTA 1: "Yes, I Published It" — updates status to Published
- [ ] CTA 2: "No, I Didn't Post It" — updates status to Notification Declined
- [ ] On "Yes": success feedback shown: "Great! Your post is now marked as published."
- [ ] On "No": feedback shown: "No problem. You can edit and reschedule this post from the Planner."
- [ ] If user dismisses modal (swipe down or tap outside): post remains Notification Sent; no API call made
- [ ] Modal is shown again on next app open if the post is still unresolved
- [ ] Modal does NOT appear again if post is already resolved (Published or Notification Declined)
- [ ] If network request fails on either action: status remains unchanged; error shown: "Something went wrong. Please try again."
- [ ] Status sync to web planner happens in real time (websocket) or on next planner refresh
- [ ] Modal only appears once per session, not repeatedly if the user returns from external apps multiple times in the same session without an unresolved post

### Mock-ups:

- iOS confirmation modal (full design including account avatar + caption preview)
  (Figma link to be added)

### Impact on existing data:

No impact on historical posts.

### Impact on other products:

- Web planner reflects status changes from iOS confirmations

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**
- **[iOS] Allow users to decline push notification posts from iOS app**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native iOS
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 6 (NEW): [Android] Confirm push notification publishing outcome on Android

### Description:

Same scope as **[iOS] Confirm push notification publishing outcome on iOS**, implemented for Android.

When the ContentStudio Android app resumes (onResume lifecycle) after a push notification post session directed the user to Instagram or Facebook, a confirmation dialog is shown asking whether the user published the post.

### Workflow:

1. User taps **Post Now** on the Android push notification action screen
2. ContentStudio opens Instagram or Facebook
3. User returns to ContentStudio app (via back button, app switcher, or task manager)
4. ContentStudio detects Activity resume after an external app session
5. A confirmation dialog appears:
   - **Title:** "Did you publish this post?"
   - **Subtext:** Shows the post account name and preview text (first ~100 chars of caption)
   - **Button 1 (positive):** "Yes, I Published It"
   - **Button 2 (negative):** "No, I Didn't Post It"
6. User taps **"Yes, I Published It"** → status → Published → success snackbar → dialog dismissed
7. User taps **"No, I Didn't Post It"** → status → Notification Declined → snackbar: "No problem. You can edit and reschedule this post from the Planner." → dialog dismissed
8. If user dismisses dialog (back button or tap outside): post remains Notification Sent; modal shown again on next app open

### Acceptance criteria:

- [ ] Confirmation dialog appears when ContentStudio Android app resumes after an external app session, if a push notification post is unresolved (status: Notification Sent)
- [ ] Dialog shows account name and a preview of the post caption
- [ ] Dialog title: "Did you publish this post?"
- [ ] Button 1 (positive): "Yes, I Published It" — updates status to Published
- [ ] Button 2 (negative): "No, I Didn't Post It" — updates status to Notification Declined
- [ ] On "Yes": snackbar: "Great! Your post is now marked as published."
- [ ] On "No": snackbar: "No problem. You can edit and reschedule this post from the Planner."
- [ ] If user dismisses dialog (back button / tap outside): post remains Notification Sent; no API call made
- [ ] Dialog is shown again on next app open if the post is still unresolved
- [ ] Dialog does NOT appear if post is already resolved
- [ ] If network request fails: status unchanged; error snackbar: "Something went wrong. Please try again."
- [ ] Status syncs to web planner in real time or on next refresh

### Mock-ups:

- Android confirmation dialog (account avatar + caption preview)
  (Figma link to be added)

### Impact on existing data:

No impact on historical posts.

### Impact on other products:

- Web planner reflects status changes from Android confirmations

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**
- **[Android] Allow users to decline push notification posts from Android app**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories) — N/A, native Android
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 7 (UPDATE #109141): [FE] Allow manual resolution of push notification posts from web app

### Description:

Users who manage their ContentStudio posts primarily from the web app need a way to manually update the status of a push notification post. This story adds manual resolution controls to the post preview/details view in the Planner for any post in **Notification Sent** status.

The user sees two action buttons on the post: **"I Published This"** and **"I Didn't Post This"**. These are only shown for posts with status **Notification Sent**.

**UI copy and component spec:**

**Status badge:**
- Label: `Notification Sent`
- Color: amber/warning — `bg-amber-100 text-amber-700` (this is a neutral/waiting state, not success or error)
- Tooltip on hover: "This post was sent to your device as a push notification. Let us know if you published it so ContentStudio can track it correctly."

**New status badge (Notification Declined):**
- Label: `Notification Declined`
- Color: gray — `bg-gray-100 text-gray-600` (neutral — post was not published, no error)
- Tooltip on hover: "You indicated this post wasn't published. You can edit and reschedule it from here."

**Resolution action buttons (shown on Notification Sent posts only):**
- Button 1 (primary): `I Published This`
  - Tooltip: "Mark this post as published. Use this if you already posted it manually on Instagram or Facebook."
- Button 2 (secondary/outlined): `I Didn't Post This`
  - Tooltip: "Mark this post as not published. You'll be able to edit and reschedule it."

**Confirmation modal — "I Published This":**
- Title: "Mark as published?"
- Body: "This will mark the post for [Account Name] as published. This can't be undone."
- CTA (primary): "Yes, Mark as Published"
- CTA (secondary): "Cancel"

**Confirmation modal — "I Didn't Post This":**
- Title: "Mark as not published?"
- Body: "This will mark the post for [Account Name] as not published. You'll still be able to edit and reschedule it from the Planner."
- CTA (primary): "Confirm"
- CTA (secondary): "Cancel"

**Success toasts:**
- After "I Published This": `Post marked as published.`
- After "I Didn't Post This": `Post marked as not published. You can edit or reschedule it anytime.`

**Error state:**
- On API failure: `Something went wrong. Please try again.` (inline error below buttons, not a toast)

**Notification Declined post — available actions:**
- Edit button (opens Composer with post data pre-filled)
- Reschedule button (opens date/time picker)
- Delete button

**Planner filter additions:**
- Add **Notification Sent** and **Notification Declined** to the status filter dropdown in the Planner list view
- Filter label for Notification Sent: `Notification Sent`
- Filter label for Notification Declined: `Notification Declined`

### Workflow:

1. User opens the Planner and sees a post with status badge **Notification Sent** (amber)
2. User clicks the post to open the post preview/detail panel
3. User sees the post details plus two buttons: **I Published This** and **I Didn't Post This**
4. User clicks **I Published This**
5. Confirmation modal appears: "Mark as published?"
6. User clicks **Yes, Mark as Published**
7. Post status badge updates to **Published** instantly (optimistic update)
8. Success toast appears: "Post marked as published."

Alternate flow — user clicks **I Didn't Post This**:
5. Confirmation modal appears: "Mark as not published?"
6. User clicks **Confirm**
7. Post status badge updates to **Notification Declined** (gray) instantly
8. Toast appears: "Post marked as not published. You can edit or reschedule it anytime."
9. Resolution buttons are replaced by Edit / Reschedule / Delete actions

### Acceptance criteria:

- [ ] Posts with **Notification Sent** status show an amber `Notification Sent` badge in the Planner list view
- [ ] Posts with **Notification Declined** status show a gray `Notification Declined` badge in the Planner list view
- [ ] "I Published This" and "I Didn't Post This" buttons appear on the post detail/preview panel **only** for posts in Notification Sent status
- [ ] Hovering either button shows the correct tooltip (see Description for copy)
- [ ] Clicking "I Published This" opens the "Mark as published?" confirmation modal
- [ ] Clicking "I Didn't Post This" opens the "Mark as not published?" confirmation modal
- [ ] Modal copy matches the spec in the Description exactly
- [ ] Confirming "I Published This" updates post status to Published (optimistic update + API PATCH)
- [ ] Confirming "I Didn't Post This" updates post status to Notification Declined (optimistic update + API PATCH)
- [ ] Success toast appears after each confirmed action (see copy in Description)
- [ ] If API call fails: optimistic update is reverted; inline error shown below buttons: "Something went wrong. Please try again."
- [ ] After status changes to Notification Declined: resolution buttons are replaced with Edit / Reschedule / Delete actions
- [ ] Notification Declined posts retain all original post data in editable form
- [ ] Planner status filter dropdown includes **Notification Sent** and **Notification Declined** options
- [ ] Filtering by Notification Sent shows only posts with that status
- [ ] Filtering by Notification Declined shows only posts with that status
- [ ] Status badge tooltip text matches spec for both new statuses
- [ ] All color classes use theme-aware values — `bg-amber-100 text-amber-700` for Notification Sent; `bg-gray-100 text-gray-600` for Notification Declined (no hardcoded hex or blue-* classes)
- [ ] Resolution flow is not available on mobile web (defer to native mobile apps for mobile confirmation)

### Mock-ups:

- Planner list view showing Notification Sent amber badge
- Post detail panel with resolution buttons and tooltips
- "Mark as published?" confirmation modal
- "Mark as not published?" confirmation modal
- Notification Declined post with Edit/Reschedule/Delete actions
- Planner filter dropdown with new status options
  (Figma link to be added)

### Impact on existing data:

No impact on existing posts. New status badges and filter options are additive.

### Impact on other products:

- Status changes made on web sync to iOS and Android apps
- White-label customers will see the same status badges; badge colors use semantic (non-primary) classes so white-label theming does not affect them

### Dependencies:

- **[BE] Add "Notification Sent" and "Notification Declined" post statuses**

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only, N/A for backend-only stories)
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
