# Research: Manage Social Account Access for Team Members

## Current State

### Access to Account Permissions Data in Frontend
- Platform-level permissions are **already in Vuex state** as `member.permissions[platform]` — an array of allowed account IDs (e.g., `member.permissions.facebook = ['page_id_1', 'page_id_2']`)
- This data lives in `activeWorkspace.members` in Vuex, loaded during workspace initialization
- **No new GET endpoint needed** — the FE can determine pre-existing access by checking if the account's ID is in `member.permissions[platform.toLowerCase()]`
- The `account_id` identifier used in `POST /api/workspaces/{workspace_id}/team/social-account-access` (sc-111605) is the same ID stored in the permissions array — the FE should use the same field from `account.platformData` to build pre-checks (field to confirm when sc-111605 is implemented)

### 3-Dots Dropdown (`AccountActionDropdown.vue`)
- `src/modules/integration/components/platforms/social_v2/components/AccountActionDropdown.vue`
- Uses `<Dropdown placement="bottom-end">` + `<DropdownItem>` from `@contentstudio/ui`
- Current actions: Account Details, Queue Schedule, Connected Devices (conditional), Shuffle Posts, Default Location (conditional), Delete Account (conditional)
- Emits events to parent (`SocialAccountsDatatable.vue`) for most actions; location modal handled inline via composable
- No existing "Manage Access" item

### Bulk Actions (`DataTableFilters.vue`)
- `src/modules/integration/components/platforms/social_v2/components/DataTableFilters.vue`
- Bulk dropdown appears when `selectedItemsCount > 1`
- Current bulk options: Delete, Reconnect (conditional)
- Emits `bulk-action` event to parent

### Parent Table (`SocialAccountsDatatable.vue`)
- `src/modules/integration/components/platforms/social_v2/components/SocialAccountsDatatable.vue`
- Handles bulk action events, routes to `handleBulkDelete` or `handleBulkReconnect` from composable
- Manages `selectedItems` ref (array of account IDs) and `selectedAccounts` (full account objects)
- Mounts `<AccountActionDropdown>` per row and listens to its emitted events

### Team Member Roles
- Roles: `super_admin`, `admin`, `collaborator`, `approver`
- The "Manage Access" modal (from sc-111606) should only show when at least one `collaborator` or `approver` exists in the workspace
- Determining this: filter `activeWorkspace.members` where `status === 'joined'` and `role === 'collaborator' || role === 'approver'`
- `useWorkspaceMembers.js` (`src/composables/useWorkspaceMembers.js`) — composable to read/filter workspace members

### Existing Modal (sc-111606 — not yet built)
- `GrantSocialAccessModal.vue` — planned at `src/modules/integration/components/dialogs/GrantSocialAccessModal.vue`
- Currently does NOT exist (sc-111606 is in the backlog)
- The new stories here extend it to support a "manage" mode (pre-populated checkboxes, PUT endpoint instead of POST)

### Existing API Endpoints (from sc-111605)
- `POST /api/workspaces/{workspace_id}/team/social-account-access` — additive only (grant, never revoke), accepts `platform + account_id + member_ids[]`
- **New endpoint needed:** `PUT /api/workspaces/{workspace_id}/team/social-account-access` — reconcile access (grant new, revoke removed), accepts same payload but replaces the current access state

## What Needs to Change

### Backend
- New `PUT /api/workspaces/{workspace_id}/team/social-account-access` endpoint
  - Same payload structure as POST: `{ platform, account_id, member_ids[] }`
  - For each `collaborator`/`approver` in the workspace: if in `member_ids` → ensure access exists; if NOT in `member_ids` → remove access if present
  - Admins/super_admins are not in the list and not affected
  - Returns `{ status: true }`
  - Add `grantSocialAccountAccessUpdate()` method to `TeamController.php` + corresponding repo method to `WorkspaceTeamRepo.php`

### Frontend — Single Account "Manage Access"
- Add "Manage Access" item to `AccountActionDropdown.vue`
  - Disabled (with tooltip) when workspace has 0 collaborators/approvers
  - Emits `manage-access` event with the account object
- Handle `manage-access` event in `SocialAccountsDatatable.vue`
  - Pass account + pre-computed access state (`memberIdsWithAccess`) to modal
- Extend `GrantSocialAccessModal.vue` (sc-111606) to support a `mode` prop: `'grant'` (original new-account flow) vs `'manage'` (edit access flow)
  - In `'manage'` mode: pre-populate checkboxes based on who already has access
  - In `'manage'` mode: "Grant Access" button → "Save Changes"
  - In `'manage'` mode: calls `PUT` endpoint instead of `POST`
  - Different modal copy for manage mode (see story)

### Frontend — Bulk "Manage Access" (Decision Needed — see below)

## Bulk Action Options

Two approaches:

**Option A — Skip bulk for now**
- Single-account manage access covers the primary use case
- Bulk can be added later once the single-account flow is stable
- Cleaner scope, ship faster

**Option B — Additive-only bulk grant**
- Add "Manage Access" to the `DataTableFilters.vue` bulk dropdown
- Modal opens with all members **unchecked** (no pre-check since state differs per account)
- Header copy explains this is an additive action: "Select members to grant access to all [N] selected accounts"
- Saving calls `POST` endpoint once per selected account (reuses sc-111605's endpoint)
- Never revokes existing access — only adds
- Separate from single-account "Manage Access" semantically: more like "Bulk Grant"

## Files Involved

| File | Change |
|---|---|
| `app/Http/Controllers/Settings/Team/TeamController.php` | Add `grantSocialAccountAccessUpdate()` for PUT |
| `app/Repository/Settings/WorkspaceTeamRepo.php` | Add reconcile method (grant + revoke per member) |
| `routes/api.php` | Register PUT route |
| `src/modules/integration/components/platforms/social_v2/components/AccountActionDropdown.vue` | Add "Manage Access" item |
| `src/modules/integration/components/platforms/social_v2/components/SocialAccountsDatatable.vue` | Handle `manage-access` event |
| `src/modules/integration/components/dialogs/GrantSocialAccessModal.vue` | Extend with `mode` prop for manage mode |
| `src/modules/integration/components/platforms/social_v2/components/DataTableFilters.vue` | Add bulk "Manage Access" (if Option B chosen) |
| `src/composables/useWorkspaceMembers.js` | Possibly extend with a helper to check per-account access |
