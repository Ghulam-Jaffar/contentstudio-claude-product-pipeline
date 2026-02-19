# Stories: Workspace Locked State — Mobile (iOS & Android)

---

## Story 1: [iOS] Show locked workspace state and block switching to locked workspaces

### Description

When a workspace owner's trial ends or has a billing issue, the workspace is flagged `has_payment_issue: true` with a `super_admin_state` value indicating the reason. The iOS app currently ignores both fields entirely — the `Workspace` model has no locked state, the workspace list shows no indication of a locked workspace, and nothing prevents a user from switching into one.

This story adds full locked-workspace handling to the iOS app.

**Backend API (no changes needed):** `GET /workspaces` already returns `has_payment_issue` (boolean) and `super_admin_state` (string) on every workspace object inside `workspace.workspace`.

**Files to modify:**
1. `ContentStudio/Modals/Workspace/Workspace.swift` — add `isLocked: Bool` and `lockReason: String?` fields
2. `ContentStudio/Controllers/Workspace/WorkspaceViewController.swift` — read both fields when parsing API response; show locked badge in list rows; block switching; show alert when active workspace is locked

**Changes:**

**`Workspace.swift`:**
- Add `let isLocked: Bool` (default `false`)
- Add `let lockReason: String?` (maps to `super_admin_state`)
- Update `init(...)` to accept both new params

**`WorkspaceViewController.swift`:**

1. **Parsing** — in `fetchWorkSpaces()`, when reading `workspaceInfoDic`:
   - Read `has_payment_issue` → pass as `isLocked`
   - Read `super_admin_state` → pass as `lockReason`

2. **Locked badge in list** — in `tableView(_:cellForRowAt:)`:
   - If `workspaceDetails.isLocked == true`, show a "Locked" badge label on the cell (red background, white text)
   - If not locked, hide the badge

3. **Block switching** — in `tableView(_:didSelectRowAt:)`:
   - If the tapped workspace `isLocked`, do **not** call `setDefaultWorkspace`
   - Instead, present an alert using the existing `showAlert(title:message:)` extension from `Alert.swift`
   - Alert message is derived from `lockReason` (see UI Copy section)

4. **Active workspace locked alert** — in the completion block of `fetchWorkSpaces()`:
   - After loading the workspace list, check if `Constant.uActiveWorkspaceDetails?.isLocked == true`
   - If so, present a non-dismissible alert with the lock reason message and a "Switch Workspace" button
   - "Switch Workspace" scrolls the table view to the first non-locked workspace in the list and dismisses the alert

---

### Workflow

**User whose active workspace is locked opens the app:**
1. User taps the workspace icon or navigates to the Workspaces screen
2. The workspace list loads — any locked workspace shows a red "Locked" badge on its row
3. If the currently active workspace is locked, an alert automatically appears explaining why and offering a "Switch Workspace" button
4. User taps "Switch Workspace" — the alert dismisses and the list scrolls to the first available (non-locked) workspace they can switch to

**User tries to tap a locked workspace row:**
1. User taps a workspace row that has the "Locked" badge
2. An alert appears explaining why it is locked (e.g., trial ended, payment issue)
3. The alert has a single "OK" button — switching does not happen

---

### Acceptance Criteria

- [ ] `Workspace.swift` has `isLocked: Bool` and `lockReason: String?` fields
- [ ] `fetchWorkSpaces()` reads `has_payment_issue` and `super_admin_state` from the API response and populates the new fields
- [ ] Locked workspaces show a visible "Locked" badge on their row in the workspace list
- [ ] Non-locked workspaces show no badge
- [ ] Tapping a locked workspace row does NOT call `setDefaultWorkspace` — switching is blocked
- [ ] Tapping a locked workspace row shows an alert with a reason-specific message (see UI Copy)
- [ ] On page load, if the currently active workspace is locked, an alert appears automatically with the reason and a "Switch Workspace" button
- [ ] Tapping "Switch Workspace" dismisses the alert
- [ ] The active workspace locked alert shows the workspace name so the user knows which workspace is affected
- [ ] All existing workspace switching behavior for non-locked workspaces is unaffected

---

### Mock-ups

N/A — use the existing `showAlert(title:message:)` UIViewController extension from `ContentStudio/Extensions/Alert.swift` for alerts. Use a `UILabel` with a red background pill for the "Locked" badge on the cell.

---

### UI Copy

**Locked badge (on row):**
`Locked`

**Alert: active workspace is locked (shown automatically on load)**

| Lock reason (`super_admin_state`) | Alert title | Alert message |
|---|---|---|
| `trial_finished` | Your trial has ended | Your free trial for **{workspace name}** has ended. Please upgrade to a paid plan to keep using this workspace. |
| `canceled` / `cancelled` | Subscription cancelled | The subscription for **{workspace name}** has been cancelled. The workspace owner needs to renew their plan. |
| `past_due` | Payment overdue | There's a payment issue with **{workspace name}**. The workspace owner needs to update their payment details. |
| `paused` | Subscription paused | The subscription for **{workspace name}** is currently paused. The workspace owner needs to resume it. |
| `deactivated` / `disabled` / `deleted` | Workspace unavailable | **{workspace name}** is no longer active. Please contact support if you think this is a mistake. |
| _(any other state)_ | Workspace locked | **{workspace name}** is currently locked. Please contact support for help. |

- Primary CTA: **Switch Workspace**
- No secondary/cancel button — user must act

**Alert: user taps a locked workspace row**

Same message table as above, but based on the tapped workspace's `lockReason`.
- Single button: **OK**

---

### Impact on Existing Data

None — `isLocked` and `lockReason` are derived from existing API fields. No local storage or user data is changed.

---

### Impact on Other Products

- Web app: Not affected — this is iOS-only
- Android: Separate story covers Android
- Chrome extension: Not affected

---

### Dependencies

None.

---

### Global Quality & Compliance

- [ ] Mobile responsiveness — N/A (this is the iOS app itself)
- [ ] Multilingual support (alert copy must use localized strings, not hardcoded English)
- [ ] UI theming support (locked badge uses system red, not a hardcoded hex color)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 2: [Android] Improve locked workspace UX — reason-specific messaging and active workspace popup

### Description

The Android app has partial locked-workspace support: the `Workspace` model has an `isLocked` field, and `WorkspaceActivity.java` reads `has_payment_issue` from the API. The workspace list shows a "workspace_locked" label badge per row, and tapping a locked workspace shows a `LockedProfileDialogue`. However:

1. The `LockedProfileDialogue` has no dismiss button, no workspace name, and no explanation of why it's locked — it's a static layout only
2. When the **currently active/default workspace** is locked, `WorkspaceActivity.java` only shows a small `tvPausedMain` text view — there is no popup or CTA directing the user to switch
3. Neither the badge nor the dialogue uses `super_admin_state` to explain the reason for locking

This story upgrades the locked workspace UX on Android.

**Backend API (no changes needed):** `GET /workspaces` already returns `has_payment_issue` (boolean) and `super_admin_state` (string) on every workspace object inside `workspace.workspace`. The `super_admin_state` field is already included in `WorkspaceRepo::fetchUserWorkspaces()`.

**Files to modify:**
1. `app/src/main/java/com/muneeb/lumotive/Workspace/Workspace.java` — add `lockReason` (String) field with getter/setter
2. `app/src/main/java/com/muneeb/lumotive/Workspace/WorkspaceActivity.java` — read `super_admin_state`, pass to workspace model; show a bottom sheet / alert dialog when active workspace is locked
3. `app/src/main/java/com/muneeb/lumotive/Util/LockedProfileDialogue.java` — update to accept workspace name + lock reason, show human-readable message and a dismiss button
4. `app/src/main/java/com/muneeb/lumotive/Workspace/AdapterWorkspace.java` — pass lock reason when constructing `LockedProfileDialogue`

**Changes:**

**`Workspace.java`:**
- Add `private String lockReason;` with `getLockReason()` / `setLockReason(String)`

**`WorkspaceActivity.java`:**
- In `fetchAllWorkspaces()` → when reading `workspaceInfoDic`: also read `super_admin_state` and call `workspaceDataSource.setLockReason(...)`
- In `setUpDefaultWS()`: if `workspaceListItem.getLocked() == true`, instead of (or in addition to) showing `tvPausedMain`, show an `AlertDialog` with:
  - Workspace name
  - Reason-specific message derived from `getLockReason()`
  - A "Switch Workspace" button that scrolls `workspaceRecycler` to the first non-locked workspace

**`LockedProfileDialogue.java`:**
- Accept `workspaceName: String` and `lockReason: String` in constructor
- Display the workspace name and a reason-specific message in `R.layout.dialogue_locked_profile`
- Add a dismiss button to the layout

**`AdapterWorkspace.java`:**
- When constructing `LockedProfileDialogue` on row click, pass `workspaceListItem.getName()` and `workspaceListItem.getLockReason()`

---

### Workflow

**User whose active workspace is locked opens WorkspaceActivity:**
1. User opens the workspace switcher
2. The workspace list loads — locked workspaces show the existing "workspace_locked" badge
3. An `AlertDialog` automatically appears explaining why the active workspace is locked and naming the workspace
4. User taps "Switch Workspace" — the dialog dismisses and the list scrolls to the first non-locked workspace they can switch to

**User taps a locked workspace row in the list:**
1. User taps a workspace row with the "workspace_locked" badge
2. `LockedProfileDialogue` opens showing the workspace name and a reason-specific message explaining why it's locked
3. User taps "OK" / "Dismiss" — dialog closes, no workspace switch happens

---

### Acceptance Criteria

- [ ] `Workspace.java` has a `lockReason` (String) field
- [ ] `WorkspaceActivity.fetchAllWorkspaces()` reads `super_admin_state` from the API response and calls `setLockReason()` on each workspace
- [ ] When the active/default workspace is locked, an `AlertDialog` appears automatically on `WorkspaceActivity` load with the workspace name and a reason-specific message
- [ ] The `AlertDialog` for the active locked workspace has a "Switch Workspace" button; tapping it dismisses the dialog
- [ ] The existing `tvPausedMain` label remains as a persistent indicator (the dialog is shown once per load, not blocking all interactions)
- [ ] `LockedProfileDialogue` displays the workspace name and a reason-specific message (not a generic string)
- [ ] `LockedProfileDialogue` has a dismiss button ("OK")
- [ ] `AdapterWorkspace` passes the workspace name and lock reason to `LockedProfileDialogue`
- [ ] Tapping a locked workspace row still shows the dialogue and does NOT trigger a workspace switch
- [ ] Non-locked workspaces are unaffected — switching still works normally

---

### Mock-ups

N/A — use Android's native `AlertDialog.Builder` for the active workspace popup, and update `R.layout.dialogue_locked_profile` to add a `TextView` for the message and a dismiss `Button`. Reference the existing `ProgressFragment` pattern for dialog construction.

---

### UI Copy

**Active workspace locked AlertDialog (shown automatically):**

| Lock reason (`super_admin_state`) | Title | Message |
|---|---|---|
| `trial_finished` | Free trial ended | Your free trial for **{workspace name}** has ended. Upgrade to a paid plan to continue using this workspace. |
| `canceled` / `cancelled` | Subscription cancelled | The subscription for **{workspace name}** has been cancelled. The workspace owner needs to renew their plan to unlock it. |
| `past_due` | Payment overdue | There's a payment issue with **{workspace name}**. The workspace owner needs to update their payment details. |
| `paused` | Subscription paused | The subscription for **{workspace name}** is paused. The workspace owner needs to resume it. |
| `deactivated` / `disabled` / `deleted` | Workspace unavailable | **{workspace name}** is no longer active. Contact support if you think this is a mistake. |
| _(any other)_ | Workspace locked | **{workspace name}** is currently locked. Please contact support for help. |

- Primary button: **Switch Workspace**

**LockedProfileDialogue (when tapping a locked workspace in list):**

Same message table as above, based on the tapped workspace's `lockReason`.
- Button: **OK**

---

### Impact on Existing Data

None — `lockReason` is derived from `super_admin_state`, which is already in the API response. No local storage or user data is changed.

---

### Impact on Other Products

- Web app: Not affected — this is Android-only
- iOS: Separate story covers iOS
- Chrome extension: Not affected

---

### Dependencies

None.

---

### Global Quality & Compliance

- [ ] Mobile responsiveness — N/A (this is the Android app itself)
- [ ] Multilingual support (all copy must use string resources, not hardcoded English)
- [ ] UI theming support (dialog uses system `AlertDialog` theme, no hardcoded colors)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
