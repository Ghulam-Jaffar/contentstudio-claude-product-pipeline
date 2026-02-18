# Research: Social Account Access Modal for New Connections

## Current State

### Permission Model
- Team member access to social accounts is stored in `workspace_team.permissions` (MongoDB collection: `workspace_team`)
- Each platform has an array of account identifiers (e.g., `facebook: ['page_id_1', 'page_id_2']`, `instagram: ['ig_id_1']`)
- An empty array means the member has NO access to any accounts on that platform
- Roles: `super_admin` (Owner), `admin`, `approver`, `collaborator`
- Owners and Admins have full access regardless of permission arrays. Only `approver` and `collaborator` roles are restricted.

### What Happens When a Team Member Is Added
- `TeamMemberPermissionsJob` fires (`app/Jobs/Settings/TeamMemberPermissionsJob.php`)
- This job queries ALL existing social accounts in the workspace and populates the new member's permissions with every account automatically
- So new members get access to all accounts that existed at the time they were added

### The Gap
- When a **new social account** is connected AFTER team members exist, nothing updates their permissions
- The new account is silently inaccessible to all collaborators/approvers
- There is no UI or mechanism to grant access to existing members when a new account is connected

### Social Account Connection Flow (Frontend)
- OAuth redirect flow → platform-specific controllers → `SaveSocialAccounts.vue` modal shown for account selection
- File: `src/modules/integration/components/dialogs/SaveSocialAccounts.vue`
- After success, the component emits `EventBus.$emit('refetchSocialPlatforms')` and shows a success toast
- **Hook point**: in the success handler after `processSavePlatforms` dispatch succeeds and `response.data.status === true`

### Existing Team Permission UI
- `src/modules/setting/components/workspace/team/SocialPlatformAccounts.vue` — collapsible per-platform account list with checkboxes, used in the "Edit Team Member" drawer
- `src/modules/setting/components/workspace/team/AddTeamMember.vue` — the add/edit team member drawer (uses `SocialPlatformAccounts.vue`)

### Existing Permission Update API
- `PUT /team/update` → `TeamController::updateTeamMember()` — updates full permissions object for a team member
- The full `permissions` object (including all platform arrays) must be passed

### Existing Account Save Dialog
- `src/modules/integration/components/dialogs/SaveSocialAccounts.vue` — uses `CstCardCheckbox` (design library) for account selection UI, has "Select All / Select None" pattern

## What Needs to Change

### Backend
1. **New endpoint**: `POST /api/workspaces/{workspace_id}/team/social-account-access`
   - Accepts: `platform` (e.g., 'facebook'), `account_id` (platform identifier), `member_ids[]` (array of `workspace_team._id` values)
   - For each member in `member_ids[]`: pushes `account_id` into their `permissions[platform]` array (avoiding duplicates)
   - Members NOT in `member_ids[]` remain unchanged (no access granted)
   - Must validate caller is owner or admin of the workspace
2. **Return list of workspace collaborators/approvers** (or use an existing endpoint — the frontend already has workspace members in Vuex state, so may not need a new BE endpoint for the list)

### Frontend
1. **New `GrantSocialAccessModal.vue` component** — modal shown after a new social account is successfully connected
   - Lists all workspace collaborators and approvers with avatar, name, role badge, checkbox
   - Select All / None toggle
   - "Grant Access" (primary) + "Skip" (secondary) CTAs
2. **Trigger logic in `SaveSocialAccounts.vue`** — after success, check if workspace has any collaborators/approvers. If yes, show the modal with the newly connected account info
3. **Reconnect detection** — only show modal for truly NEW accounts (not reconnections). The `processSavePlatforms` response includes the saved accounts — check if any are new (not previously in the accounts list)

## Files Involved

**Backend:**
- `app/Http/Controllers/Settings/Team/TeamController.php` — add new endpoint method
- `app/Repository/Settings/WorkspaceTeamRepo.php` — add repo method to update single account access for multiple members
- `routes/api.php` — register new route

**Frontend:**
- `src/modules/integration/components/dialogs/SaveSocialAccounts.vue` — add post-save trigger logic
- `src/modules/integration/components/dialogs/GrantSocialAccessModal.vue` — new modal component
- `src/modules/setting/components/workspace/team/AddTeamMember.vue` — reference for team member display pattern

## Edge Cases

1. **No collaborators/approvers in workspace** → skip silently, no modal
2. **User dismisses/skips modal** → new account has no member access (acceptable — owner/admin can still manage via team settings)
3. **Reconnect flow** (existing account token refresh) → do NOT show modal; account permissions are preserved
4. **Multiple accounts connected at once** (e.g., multiple Facebook pages) → show modal for the batch, one access decision covers all newly connected accounts in that session
5. **50+ team members** → modal list is scrollable
6. **White-label** → modal inherits workspace theme, uses CSS variable colors (`text-primary-cs-500` etc.)
7. **Admin connecting via External Cloud Connect link** → already handled separately (external flow); this modal is only for the internal OAuth flow
