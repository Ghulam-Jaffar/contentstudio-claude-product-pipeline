# Research: Shared Folder Team Permission

## Current State

### Team Permissions System (Backend)
- **Model:** `WorkspaceTeam` (`app/Models/Settings/WorkspaceTeam.php`) stores a `permissions` JSON object per team member
- **Default permissions** (line 36-57): boolean flags like `addBlog`, `addSocial`, `addSource`, `addTopic`, `viewTeam`, `postsReview`, `rescheduleQueue`, plus per-platform account arrays
- **Permission checks:** `app/Libraries/Permission/PermissionHelper.php` — switch-case that reads `$this->member->permissions['keyName']` for each action
- **API:** `TeamController::addTeamMember()` and `TeamController::updateTeamMember()` in `app/Http/Controllers/Settings/Team/TeamController.php` — both accept a `permissions` object from the request and persist it

### Team Permissions UI (Frontend)
- **Component:** `src/modules/setting/components/workspace/team/AddTeamMember.vue`
- Collaborator permissions rendered as a grid of `<Checkbox>` components (lines 580-623), each bound to `getTeam.permissions.<key>`
- Approver permissions have a separate section with info icon tooltips (lines 626-653)
- Localization keys in `src/locales/en/settings.json` under `settings.workspace.team.permissions.collaborator.*`

### Global (Shared) Folder in Media Library
- **Backend model:** Media library folders have an `is_global` boolean field. The global folder is created per super-admin and shared across all workspaces under that account.
- **Backend repo:** `app/Repository/Storage/MediaLibraryFoldersRepo.php` — folder queries use `is_global` + `super_admin_id` to include the shared folder in listings. Global root folders cannot be deleted.
- **Frontend folder component:** `src/modules/publish/components/media-library/components/Folder.vue`
  - Already has a `globalFolderAccess` check (line 302): `const globalFolderAccess = canAccess('media_library_shared_folder')`
  - This is a **billing/plan-level** feature gate via `useFeatures()` — NOT a team-member permission
  - If `globalFolderAccess.allowed` is false: folder is shown but disabled (opacity-70, pointer-events-none, lock icon, tooltip with upgrade message)

### What's Missing
There is **no team-member-level permission** for shared folder access. Currently:
- If the plan allows `media_library_shared_folder` → all team members see it
- If the plan doesn't → nobody sees it (billing gate)

The user wants a **per-member permission toggle** so workspace admins can control which team members can access the shared folder, independent of the billing plan.

## What Needs to Change

### Backend
- Add `accessSharedFolder` (default: `true`) to `WorkspaceTeam.$attributes['permissions']`
- Add a case in `PermissionHelper` to check `access_shared_folder` permission
- In `MediaLibraryFolderController` and/or `MediaLibraryFoldersRepo`: when listing folders, check if the current user has `accessSharedFolder` permission — if not, exclude `is_global` folders from the response
- Ensure `addTeamMember` and `updateTeamMember` in `TeamController` pass through the new permission

### Frontend
- Add `accessSharedFolder` checkbox + info tooltip in `AddTeamMember.vue` for collaborator and approver roles
- Add localization keys for the permission label and tooltip
- The existing `Folder.vue` global folder gating already handles the disabled/hidden state — the change is that the **backend won't return** the shared folder if permission is off, so the folder simply won't appear in the list (no FE filtering needed)

## Files Involved

### Backend
- `app/Models/Settings/WorkspaceTeam.php` — add default permission
- `app/Libraries/Permission/PermissionHelper.php` — add permission check case
- `app/Repository/Storage/MediaLibraryFoldersRepo.php` — filter out global folders for unauthorized members
- `app/Http/Controllers/Settings/Team/TeamController.php` — passthrough (already handles permissions object generically)

### Frontend
- `src/modules/setting/components/workspace/team/AddTeamMember.vue` — add checkbox + tooltip
- `src/locales/en/settings.json` — add label + tooltip strings
- `src/locales/{de,el,es,fr,it,zh}/settings.json` — add translation keys (English fallback)
