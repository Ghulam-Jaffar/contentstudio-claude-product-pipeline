# Research: Workspace Locked State — Mobile (iOS & Android)

## Background

When a user's trial ends (or there's a payment issue), a workspace gets flagged with `has_payment_issue = true` on the backend. The goal is to surface this locked state clearly in the mobile apps, prevent users from switching into a locked workspace, and show a proper action when their currently active workspace is locked.

## Backend (no changes needed)

- `WorkspaceRepo::fetchUserWorkspaces()` already includes `has_payment_issue` in every workspace object returned by `GET /workspaces`
  - File: `app/Repository/Settings/WorkspaceRepo.php`
- Workspace locking is already set by the billing system when trial ends or payment fails
- The API field to use: `workspace.has_payment_issue` (boolean)
- No backend changes required — the data is already flowing

## Android (partial implementation exists)

**What's already done:**
- `Workspace.java` (`app/src/main/java/com/muneeb/lumotive/Workspace/Workspace.java`): has `isLocked` field with `getLocked()` / `setLocked()` ✅
- `WorkspaceActivity.java` reads `has_payment_issue` from the API response and calls `workspaceDataSource.setLocked()` ✅
- `AdapterWorkspace.java`: shows a "workspace_locked" label (`tvPaused`) in the workspace list when `getLocked() == true` ✅
- `AdapterWorkspace.java`: when user taps a locked workspace in the list → shows `LockedProfileDialogue` (a basic dialog from `R.layout.dialogue_locked_profile`) — blocks the switch ✅

**What's missing:**
- When the **currently active/default workspace** is locked, `WorkspaceActivity.java` only shows a `tvPausedMain` text view in the header area — not prominent, no CTA, no instruction to switch
- There is no popup/dialog when the user lands on the workspace screen with their active workspace already locked
- The `LockedProfileDialogue` used for the list (when switching) has no "Switch workspace" CTA button — it's just a static layout

**Files to modify:**
- `app/src/main/java/com/muneeb/lumotive/Workspace/WorkspaceActivity.java` — show a proper locked-workspace alert/bottom sheet when active workspace is locked
- `app/src/main/java/com/muneeb/lumotive/Util/LockedProfileDialogue.java` — add a "Switch workspace" action

## iOS (nothing implemented)

**What exists:**
- `WorkspaceViewController.swift` — handles workspace list and switching via `setDefaultWorkspace(wsId:)` (`ContentStudio/Controllers/Workspace/WorkspaceViewController.swift`)
- `Workspace.swift` — the workspace model (`ContentStudio/Modals/Workspace/Workspace.swift`)
- `ServiceManager.swift` (`ContentStudio/HelperClasses/ServiceManager.swift`) — API calls
- `Constant.swift` (`ContentStudio/HelperClasses/Constant.swift`) — stores `uActiveWorkspaceDetails: Workspace?`
- `Alert.swift` extension (`ContentStudio/Extensions/Alert.swift`) — existing alert utilities

**What's missing (everything):**
- `Workspace.swift`: no `isLocked` field
- `WorkspaceViewController.swift`: doesn't read `has_payment_issue` when parsing workspace API response
- No "Locked" indicator on workspace rows in the list
- No protection against tapping a locked workspace (currently any tap triggers `setDefaultWorkspace`)
- No alert/popup when the active workspace is locked

**Files to modify:**
- `ContentStudio/Modals/Workspace/Workspace.swift` — add `isLocked: Bool` field
- `ContentStudio/Controllers/Workspace/WorkspaceViewController.swift` — read `has_payment_issue`, show locked badge in list rows, block switching, show alert if active workspace is locked

## What Needs to Change

**iOS:**
1. Add `isLocked: Bool` to `Workspace.swift` (and update its `init`)
2. In `WorkspaceViewController.fetchWorkSpaces()`: read `has_payment_issue` from `workspaceInfoDic` and pass to `Workspace(...)` init
3. In `tableView(_:cellForRowAt:)`: show a "Locked" badge on locked workspace rows
4. In `tableView(_:didSelectRowAt:)`: if the selected workspace `isLocked`, show an alert instead of calling `setDefaultWorkspace`
5. In `viewDidLoad()` or `fetchWorkSpaces()` completion: if active workspace `isLocked`, show an alert with a "Switch Workspace" option

**Android:**
1. In `WorkspaceActivity.setUpDefaultWS()`: when `workspaceListItem.getLocked() == true`, show a proper bottom sheet / dialog (not just `tvPausedMain`) with workspace name, "Your workspace is locked" message, and CTA to scroll the list and switch
2. In `LockedProfileDialogue` (used when tapping a locked workspace in list): add a dismiss button and brief message
3. Optional: `tvPausedMain` can remain as a persistent banner but a one-time popup on first load is also needed

## Files Involved

**iOS:**
- `contentstudio-ios-v2/ContentStudio/Modals/Workspace/Workspace.swift`
- `contentstudio-ios-v2/ContentStudio/Controllers/Workspace/WorkspaceViewController.swift`

**Android:**
- `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/Workspace/WorkspaceActivity.java`
- `contentstudio-android-v2/app/src/main/java/com/muneeb/lumotive/Util/LockedProfileDialogue.java`
