# Stories: Manage Social Account Access for Team Members

---

## Story 1: [BE] Add endpoint to update team member access for a social account

### Description:

The existing `POST /api/workspaces/{workspace_id}/team/social-account-access` (sc-111605) is additive only — it grants access to selected members but never removes it. That's appropriate for the "new account connected" flow, but not for an edit/manage flow where the owner needs to both add and revoke access.

This story adds a `PUT` endpoint with reconcile semantics: the caller passes the complete list of member IDs who should have access, and the backend reconciles — adding access to members who don't have it yet, and revoking it from members who have it but are no longer in the list.

**New endpoint:**

`PUT /api/workspaces/{workspace_id}/team/social-account-access`

Request payload:
```json
{
  "platform": "facebook",
  "account_id": "page_id_123",
  "member_ids": ["workspace_team_id_1", "workspace_team_id_2"]
}
```

Behavior:
- Fetch all `workspace_team` records for this workspace where `role` is `collaborator` or `approver` (exclude `super_admin` and `admin` — they always have access and are never managed through this flow)
- For each collaborator/approver:
  - If their `_id` is in `member_ids` → push `account_id` into their `permissions[platform]` if not already present
  - If their `_id` is NOT in `member_ids` → pull `account_id` from their `permissions[platform]` if present
- Authorization: caller must be `super_admin` or `admin` in the workspace — 403 otherwise
- If `member_ids` is an empty array: revoke access from all collaborators/approvers for this account (none get access)
- Returns `{ status: true }` on success

**Key files:**
- `app/Http/Controllers/Settings/Team/TeamController.php` — add `updateSocialAccountAccess()` method
- `app/Repository/Settings/WorkspaceTeamRepo.php` — add `reconcileSocialAccountAccess(platform, account_id, member_ids)` method that iterates all collaborators/approvers in the workspace and applies the grant/revoke logic
- Route registration in `routes/api.php` as `PUT`

---

### Workflow:

1. Owner/admin opens Settings → Social Accounts and clicks the 3-dots menu on an account
2. They click "Manage Access" in the dropdown
3. The frontend displays the current access state (pre-populated from Vuex) and lets the owner adjust it
4. Owner clicks "Save Changes"
5. Frontend calls `PUT /api/workspaces/{workspace_id}/team/social-account-access` with the full list of member IDs who should have access
6. Backend reconciles: adds access for newly selected members, removes access for deselected members
7. Selected members can see and use the account; deselected members lose access

---

### Acceptance criteria:

- [ ] `PUT /api/workspaces/{workspace_id}/team/social-account-access` exists and accepts `platform`, `account_id`, and `member_ids[]`
- [ ] Members in `member_ids` who do not yet have access get `account_id` added to their `permissions[platform]` array
- [ ] Members NOT in `member_ids` who currently have access get `account_id` removed from their `permissions[platform]` array
- [ ] Admins and super_admins are excluded from reconciliation — their permissions are never modified
- [ ] If a member in `member_ids` already has access, no duplicate is written
- [ ] If a member not in `member_ids` doesn't have access, no change is made
- [ ] If `member_ids` is empty, all collaborators/approvers in the workspace have access revoked for this account
- [ ] Callers who are not `super_admin` or `admin` receive a 403 response
- [ ] Invalid `workspace_id`, `platform`, or `account_id` returns a 422 with a clear error message
- [ ] Endpoint works for all supported platforms: `facebook`, `instagram`, `twitter`, `linkedin`, `pinterest`, `gmb`, `tiktok`, `youtube`, `tumblr_blogs`, `tumblr_profiles`, `medium`, `wordpress`
- [ ] The existing `POST` endpoint behavior is unaffected — it remains additive-only

---

### Mock-ups:

N/A — backend-only story.

---

### Impact on existing data:

- Modifies `workspace_team.permissions[platform]` arrays for collaborators/approvers in the workspace
- Both additive (grant) and subtractive (revoke) operations — members not in the payload lose access to the specified account

---

### Impact on other products:

- Mobile apps: Members whose permissions are updated will see the change reflected in the mobile Composer on next load. No mobile code changes needed.
- Chrome extension: Not affected.

---

### Dependencies:

Depends on: **[BE] Grant social account access to team members when a new account is connected**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (backend-only story)
- [ ] Multilingual support — N/A (backend-only story, no user-facing strings)
- [ ] UI theming support — N/A (backend-only story)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [FE] Add "Manage Access" option to social account row actions

### Description:

Owners and admins currently have no way to review or change which team members can access a specific social account after it's been connected. The only workaround is navigating to Settings → Team and editing each member individually.

This story adds a "Manage Access" item to the 3-dots dropdown menu on each social account row. When clicked, it opens the team access modal (`GrantSocialAccessModal.vue` from **[FE] Show team access modal after connecting a new social account**) in a new "manage" mode — pre-populated with the current access state for that account, letting the owner add or remove access in one step.

**Changes to `AccountActionDropdown.vue`:**
- Add a "Manage Access" `<DropdownItem>` below the existing actions and above Delete
- Gate visibility: always shown (not conditionally hidden)
- Gate interactivity: disabled when the workspace has no collaborators or approvers (same condition as the access modal trigger in sc-111606). Show a tooltip on hover when disabled.
- Emit `manage-access` event with the full `account` object when clicked

**Changes to `SocialAccountsDatatable.vue`:**
- Listen to `@manage-access` from `<AccountActionDropdown>`
- On receive: compute `memberIdsWithAccess` — the list of member IDs in `activeWorkspace.members` (filtered to collaborators/approvers) who have the account's ID in their `permissions[account.platform.toLowerCase()]` array
- Open `GrantSocialAccessModal` with `mode="manage"`, `account`, and `memberIdsWithAccess`

**Changes to `GrantSocialAccessModal.vue` (extend with `mode` prop):**
- Add `mode` prop: `'grant'` (existing) or `'manage'` (new)
- In `'manage'` mode:
  - Pre-populate checkboxes: members whose IDs are in `memberIdsWithAccess` start checked; others start unchecked
  - Change primary CTA label from "Grant Access" to "Save Changes"
  - Replace "Skip" button with "Cancel" (closes modal, no confirmation popup needed — user is editing, not skipping setup)
  - On save: call `PUT /api/workspaces/{workspace_id}/team/social-account-access` instead of `POST`
  - Show different modal title and subtitle (see UI Copy below)
  - Show different success toast (see UI Copy below)

**Account ID for permission check:**
Use the same field that the `POST` endpoint stores in `permissions[platform]` — confirm with the backend implementer of sc-111605 which field from `account.platformData` is used as `account_id`.

---

### Workflow:

1. Owner/admin navigates to Settings → Social Accounts
2. They click the 3-dots (⋮) icon on a social account row
3. The dropdown shows "Manage Access" as an option
   - If the workspace has no collaborators or approvers: the item appears grayed out and shows a tooltip explaining why
4. Owner clicks "Manage Access"
5. A modal opens titled "Manage access to [Account Name]"
6. The modal lists all workspace collaborators and approvers — each row shows avatar, full name, and role badge
7. Members who already have access appear with their checkbox checked; members without access appear unchecked
8. Owner adjusts the checkboxes — checking grants access, unchecking removes it
9. Owner clicks "Save Changes"
10. Access is updated: newly checked members gain access; newly unchecked members lose access
11. A success toast confirms: _"Access settings updated."_
12. If owner clicks "Cancel" instead, the modal closes with no changes made

---

### Acceptance criteria:

- [ ] "Manage Access" item appears in the `AccountActionDropdown` for every social account row
- [ ] The item is disabled (non-clickable, visually grayed out) when the workspace has no collaborators or approvers
- [ ] A tooltip is shown on hover when the item is disabled (see UI Copy below)
- [ ] The item is clickable and opens the `GrantSocialAccessModal` when at least one collaborator or approver exists
- [ ] Modal opens in "manage" mode: title reads "Manage access to [Account Name]"
- [ ] Members who currently have access to this account appear with their checkboxes pre-checked
- [ ] Members who do not have access appear with their checkboxes unchecked
- [ ] The primary CTA button reads "Save Changes"
- [ ] The secondary button reads "Cancel" (no skip confirmation popup)
- [ ] Clicking "Cancel" closes the modal without making any API call
- [ ] Clicking "Save Changes" calls `PUT /api/workspaces/{workspace_id}/team/social-account-access` with the full list of checked member IDs
- [ ] On API success, modal closes and toast shows: _"Access settings updated."_
- [ ] On API error, toast shows the error copy (see UI Copy) and modal stays open
- [ ] "Save Changes" button shows a spinner while the API call is in progress
- [ ] "Select All" checkbox at the top works the same as in grant mode
- [ ] The existing grant-mode behavior (used on new account connection from sc-111606) is unaffected

---

### Mock-ups:

N/A — extend existing `GrantSocialAccessModal.vue` layout. Use the same design library components (`CstSimpleCheckbox` or equivalent, avatars, role badges).

---

### UI Copy

**"Manage Access" dropdown item label:** `Manage Access`

**Dropdown item tooltip (disabled state — no collaborators/approvers):**
> "No collaborators or approvers in this workspace. Admins already have access to all accounts."

**Modal (manage mode):**

| Element | Copy |
|---|---|
| Title | Manage access to [Account Name] |
| Subtitle | Control which team members can see and post to this account from the Composer. Checked members have access; unchecked members don't. Changes take effect immediately. |
| "Select All" label | Select all |
| Role badge — Collaborator | Collaborator |
| Role badge — Approver | Approver |
| Primary CTA | Save Changes |
| Secondary CTA | Cancel |

**Tooltips on role badges (same as grant mode):**

| Element | Tooltip |
|---|---|
| Role badge "Collaborator" | Can create and schedule posts, but needs an approver to publish. |
| Role badge "Approver" | Can review and approve posts created by collaborators. |
| Info icon (ℹ) next to modal title | Only collaborators and approvers are listed here. Admins already have access to all accounts automatically. |

**Success toast:**

| Scenario | Toast copy |
|---|---|
| Save successful | Access settings updated. |

**Error toast:**

| Scenario | Toast copy |
|---|---|
| API error | Something went wrong. Please try again or manage access from team settings. |

---

### Impact on existing data:

None from the frontend. The backend story (**[BE] Add endpoint to update team member access for a social account**) writes the permission changes.

---

### Impact on other products:

- Mobile apps: Members whose access is updated via this flow will see the change reflected in the mobile Composer on next load. No mobile code changes.
- Chrome extension: Not affected.
- White-label: Modal uses `text-primary-cs-500`, `bg-primary-cs-50`, and design library components — fully compatible with white-label theming.

---

### Dependencies:

Depends on: **[FE] Show team access modal after connecting a new social account**
Depends on: **[BE] Add endpoint to update team member access for a social account**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (modal should display correctly on smaller screens)
- [ ] Multilingual support (all copy strings should be wrapped in i18n translation keys)
- [ ] UI theming support (design library components used throughout, no hardcoded colors)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
