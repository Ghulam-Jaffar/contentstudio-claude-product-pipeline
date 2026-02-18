# Stories: Social Account Access Modal for New Connections

---

## Story 1: [BE] Grant social account access to team members when a new account is connected

### Description:

When a workspace owner or admin connects a new social account, collaborators and approvers in the workspace do not automatically get access to it. This story adds a backend endpoint that the frontend calls to grant (or skip granting) access to a specific newly connected social account for a selected set of workspace team members.

**How access works today:**
- Team member permissions are stored in `workspace_team.permissions` per platform (e.g., `facebook: ['page_id_1', 'page_id_2']`)
- An empty array means no access to that platform's accounts
- When a new member is added, `TeamMemberPermissionsJob` populates their permissions with all accounts that existed at that point
- When a new social account is connected AFTER members exist, nothing updates their permissions — the new account is silently inaccessible

**New endpoint:**

`POST /api/workspaces/{workspace_id}/team/social-account-access`

Request payload:
```json
{
  "platform": "facebook",
  "account_id": "page_id_123",
  "member_ids": ["workspace_team_id_1", "workspace_team_id_2"]
}
```

Behavior:
- For each `member_id` in the array: look up the `workspace_team` record, push `account_id` into their `permissions[platform]` array (skip if already present to avoid duplicates)
- Members NOT in `member_ids` remain unchanged — no access is added or removed for them
- If `member_ids` is an empty array, the call is valid — it means the caller intentionally granted access to no one (all members skip)
- Authorization: caller must be `super_admin` or `admin` in the workspace

**Where this is called from:**
After a social account is successfully connected in `SaveSocialAccounts.vue`, the frontend shows a modal listing collaborators/approvers. This endpoint is called with the member IDs the owner selected.

**Key files:**
- `app/Http/Controllers/Settings/Team/TeamController.php` — add new `grantSocialAccountAccess()` method
- `app/Repository/Settings/WorkspaceTeamRepo.php` — add repo method to push account_id into `permissions[platform]` for multiple members
- Route registration in `routes/api.php`

---

### Workflow:

1. Owner/admin connects a new social account via the standard OAuth flow
2. They select which pages/channels to connect in `SaveSocialAccounts.vue`
3. Accounts are saved successfully on the backend
4. Frontend detects collaborators/approvers exist → shows `GrantSocialAccessModal`
5. Owner selects which members get access and clicks "Grant Access" (or skips)
6. Frontend calls `POST /api/workspaces/{workspace_id}/team/social-account-access` with the platform, account_id, and selected member_ids
7. Backend pushes the account_id into each selected member's `permissions[platform]` array
8. Selected members can now see and use the new account in the Composer

---

### Acceptance criteria:

- [ ] `POST /api/workspaces/{workspace_id}/team/social-account-access` exists and accepts `platform`, `account_id`, and `member_ids[]`
- [ ] For each member in `member_ids`, the `account_id` is pushed into their `workspace_team.permissions[platform]` array
- [ ] If `account_id` is already present in a member's platform array, it is not duplicated
- [ ] Members not in `member_ids` are unaffected — their permissions are not modified
- [ ] If `member_ids` is empty, the endpoint returns `{ status: true }` with no permission changes
- [ ] Callers who are not `super_admin` or `admin` in the workspace receive a 403 response
- [ ] Invalid `workspace_id`, `platform`, or `account_id` returns a 422 with a clear error message
- [ ] Endpoint works for all supported platforms: `facebook`, `instagram`, `twitter`, `linkedin`, `pinterest`, `gmb`, `tiktok`, `youtube`, `tumblr_blogs`, `tumblr_profiles`, `medium`, `wordpress`

---

### Mock-ups:

N/A — backend-only story.

---

### Impact on existing data:

- Modifies `workspace_team.permissions[platform]` arrays for selected members
- Only additive — never removes existing access

---

### Impact on other products:

- Mobile apps: Members whose permissions are updated will see the newly accessible account in the mobile Composer on next load. No mobile code changes needed — they already read from the same permissions.
- Chrome extension: Not affected.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (backend-only story)
- [ ] Multilingual support — N/A (backend-only story, no user-facing strings)
- [ ] UI theming support — N/A (backend-only story)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [FE] Show team access modal after connecting a new social account

### Description:

When a workspace owner or admin successfully connects a new social account, collaborators and approvers in the workspace silently lose out — they get no access to the new account unless someone manually goes to team settings and updates each member individually. There's no prompt, no discovery, no workflow for this.

This story adds a modal that appears immediately after a new social account is saved. It lists all collaborators and approvers in the workspace and lets the owner/admin choose who gets access to the new account in one step — without leaving the current page.

If the owner dismisses or skips the modal, a confirmation popup appears explaining the impact before the decision is finalized.

**What is a "new" account vs. a reconnect:**
- Show modal only when a social account is connected for the first time (new account)
- Do NOT show modal when an existing account is reconnected (token refresh). Detect this by comparing newly saved accounts from the API response against accounts already in Vuex state before the save.

**When NOT to show the modal:**
- Workspace has no collaborators or approvers (only owner/admins) — skip silently
- Reconnect flow (account was already connected) — skip silently

**New component:** `GrantSocialAccessModal.vue`
Location: `src/modules/integration/components/dialogs/GrantSocialAccessModal.vue`

**Component to modify:** `src/modules/integration/components/dialogs/SaveSocialAccounts.vue`
After `processSavePlatforms` succeeds, detect new accounts, check for collaborators/approvers in Vuex state, and trigger the new modal.

---

### Workflow:

1. Owner/admin connects a social account (e.g., a new Facebook Page) via Settings → Social Accounts
2. They see the `SaveSocialAccounts` modal, select their pages, and click "Save"
3. Accounts are saved. The system detects: (a) at least one NEW account was added, and (b) the workspace has at least one collaborator or approver
4. A new modal appears: **"Who should have access to [Account Name]?"**
5. The modal lists all workspace collaborators and approvers — each row shows avatar, full name, and role badge ("Collaborator" or "Approver")
6. A "Select All" toggle appears at the top of the list
7. Owner checks the boxes for the people they want to give access to
8. Owner clicks **"Grant Access"** — the selected members get access. The modal closes. A success toast appears: _"Access granted to [N] team member(s)."_
9. **If owner clicks "Skip" instead** — a confirmation popup appears before closing:
   - Owner reads the impact copy and either clicks **"Yes, Skip"** (closes modal, no access granted) or **"Go Back"** (returns to the access modal)
10. Newly granted members can now see and post to the new account from the Composer

---

### Acceptance criteria:

- [ ] After a new social account is saved successfully, the `GrantSocialAccessModal` is shown if the workspace has at least one collaborator or approver
- [ ] Modal does NOT appear if the workspace has only owners/admins (no collaborators or approvers)
- [ ] Modal does NOT appear for reconnect flows — only for truly new accounts
- [ ] Modal title shows: **"Who should have access to [Account Name]?"** — where [Account Name] is the connected account's display name (e.g., "Acme Corp Page")
- [ ] When multiple accounts are connected at once, the modal covers them all — the description explains this (see copy below)
- [ ] Each list row shows: avatar (with fallback to initials), full name, role badge
- [ ] "Select All" checkbox at the top selects all members; unchecks all when all are selected
- [ ] "Grant Access" button is enabled even if 0 members are selected (grants access to nobody — same as skip)
- [ ] Clicking "Grant Access" calls `POST /api/workspaces/{workspace_id}/team/social-account-access` with the selected member IDs
- [ ] On API success, modal closes and a toast shows: **"Access granted to [N] team member(s)."** (if N = 0: **"No access granted. You can update this anytime from team settings."**)
- [ ] On API error, toast shows: **"Something went wrong. Please try again or manage access from team settings."** — modal stays open
- [ ] "Skip" button triggers a confirmation popup before closing
- [ ] Confirmation popup shows the correct copy (see UI Copy below)
- [ ] Clicking "Yes, Skip" in the confirmation popup closes the modal without any API call
- [ ] Clicking "Go Back" in the confirmation popup returns the user to the grant access modal (popup closes, modal stays open)
- [ ] List is scrollable if there are many members — does not overflow the modal
- [ ] Modal uses design library components throughout (`CstSimpleCheckbox` or equivalent, `CstConfirmationPopup` or equivalent)
- [ ] Uses `text-primary-cs-500`, `bg-primary-cs-50` — no hardcoded colors
- [ ] Loading state: "Grant Access" button shows spinner while API call is in progress

---

### Mock-ups:

N/A — use the design library throughout. Refer to existing `SaveSocialAccounts.vue` and `SocialPlatformAccounts.vue` for layout patterns.

---

### UI Copy

**Modal:**

| Element | Copy |
|---|---|
| Title | Who should have access to [Account Name]? |
| Subtitle (single account) | Choose which team members can see and post to this account from the Composer. Members not selected here won't see it at all — you can always update this from team settings. |
| Subtitle (multiple accounts connected) | You connected [N] new accounts. Choose which team members can see and post to them from the Composer. Members not selected here won't see any of the new accounts. You can always update this from team settings. |
| "Select All" label | Select all |
| Role badge — Collaborator | Collaborator |
| Role badge — Approver | Approver |
| Primary CTA | Grant Access |
| Secondary CTA | Skip |

**Member list empty state** (no collaborators or approvers — modal should not show, but as a safeguard):

> _"No collaborators or approvers in this workspace."_

**Success toasts:**

| Scenario | Toast copy |
|---|---|
| N members selected (N ≥ 1) | Access granted to [N] team member(s). |
| 0 members selected (clicked "Grant Access" with none checked) | No access granted. You can update this anytime from team settings. |
| API error | Something went wrong. Please try again or manage access from team settings. |

**Skip confirmation popup:**

| Element | Copy |
|---|---|
| Title | Skip access setup? |
| Body | If you skip, none of your collaborators or approvers will be able to see or post to **[Account Name]** from the Composer. You can grant access later from **Settings → Team**, but they'll be blocked from using this account until you do. |
| Confirm button (destructive) | Yes, Skip |
| Cancel button | Go Back |

*(If multiple accounts: replace "[Account Name]" with "these new accounts")*

**Tooltips:**

| Element | Tooltip |
|---|---|
| Role badge "Collaborator" | Can create and schedule posts, but needs an approver to publish. |
| Role badge "Approver" | Can review and approve posts created by collaborators. |
| Info icon (ℹ) next to modal title | Only collaborators and approvers are listed here. Admins already have access to all accounts automatically. |

---

### Impact on existing data:

None — this story is purely additive UI. The actual permission data is written by the backend story (**[BE] Grant social account access to team members when a new account is connected**).

---

### Impact on other products:

- Mobile apps: No change needed. Members whose permissions are updated via this flow will see the newly accessible account on mobile on next load.
- Chrome extension: Not affected.
- White-label: Modal uses CSS variable colors and design library components — compatible with white-label theming.

---

### Dependencies:

Depends on: **[BE] Grant social account access to team members when a new account is connected**

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (modal should display correctly on smaller screens)
- [ ] Multilingual support (all copy strings should be wrapped in i18n translation keys)
- [ ] UI theming support (design library components used throughout, no hardcoded colors)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
