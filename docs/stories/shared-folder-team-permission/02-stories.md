# Stories: Shared Folder Team Permission

---

## [BE] Add shared folder access permission to team member roles

### Description:

Add a new `accessSharedFolder` boolean permission to the team member permissions system. When this permission is `false` for a team member, the media library folder listing API must exclude all `is_global` folders from the response. This ensures that team members without this permission never see the shared folder in their media library.

**Key files:**
- `app/Models/Settings/WorkspaceTeam.php` — add `"accessSharedFolder" => true` to the default `$attributes['permissions']` array
- `app/Libraries/Permission/PermissionHelper.php` — add a new case `"access_shared_folder"` that checks `$this->member->permissions['accessSharedFolder']`
- `app/Repository/Storage/MediaLibraryFoldersRepo.php` — in the query builder (specifically the `find_root_folders_by_workspace_only_structured` case and similar listing cases), when the current user is not the workspace super-admin, check if the team member has `accessSharedFolder` permission. If `false`, add `->where('is_global', '!=', true)` to exclude global folders from the result.
- `app/Http/Controllers/Settings/Team/TeamController.php` — no changes needed; `addTeamMember()` and `updateTeamMember()` already accept and persist the full `permissions` object from the request.

**Behavior rules:**
- Workspace owner (super-admin) always has access to the shared folder — this permission only applies to invited team members (collaborators, approvers, admins).
- Default value is `true` — existing team members retain access. Only newly toggled-off members lose access.
- When `accessSharedFolder` is `false`, the API must not return `is_global` folders at all (not just hide them — don't send them).

---

### Workflow:

1. Workspace owner navigates to Settings → Manage Team & Roles
2. Owner adds a new team member or edits an existing one
3. Owner sets the "Access to shared folder" permission to off
4. Owner saves the team member settings
5. The team member logs in and opens the Media Library
6. The team member does not see the shared folder in their folder list — it is completely absent
7. If the owner later turns the permission back on and saves, the team member sees the shared folder again on their next visit to Media Library

---

### Acceptance criteria:

- [ ] `WorkspaceTeam` model has `accessSharedFolder` defaulting to `true` in the `$attributes['permissions']` array
- [ ] `PermissionHelper` includes a case for `"access_shared_folder"` that returns the value of `permissions['accessSharedFolder']`
- [ ] Media library folder listing API excludes `is_global` folders when the requesting team member has `accessSharedFolder` set to `false`
- [ ] Workspace super-admin always sees the shared folder regardless of any permission setting
- [ ] Existing team members (who don't have the `accessSharedFolder` key in their permissions document) default to having access (backward-compatible)
- [ ] `addTeamMember` and `updateTeamMember` endpoints correctly persist the `accessSharedFolder` value when passed in the request
- [ ] When `accessSharedFolder` is `false`, subfolder and asset endpoints for global folders also return appropriate forbidden/empty responses

---

### Mock-ups:

N/A — backend only.

---

### Impact on existing data:

- Existing `workspace_team` documents in MongoDB do not have the `accessSharedFolder` field. The code must treat a missing field as `true` (default access granted) for backward compatibility. No data migration needed — the `$attributes` default handles new records, and code should use `$permissions['accessSharedFolder'] ?? true` for existing records.

---

### Impact on other products:

- **Mobile apps:** No impact — mobile apps do not have a media library shared folder feature.
- **Chrome extension:** No impact — Chrome extension does not use the media library.
- **White-label:** No impact — permission is per team member, not per domain.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## [FE] Add shared folder access permission toggle in team member settings

### Description:

Add a new "Access to shared folder" checkbox with an info tooltip to the team member permissions section in `AddTeamMember.vue`. This permission controls whether the team member can see and interact with the shared (global) folder in the Media Library. The permission should appear for collaborator, approver, and admin roles, and default to on.

Since the backend (see **[BE] Add shared folder access permission to team member roles**) will exclude global folders from the API response when the permission is off, no changes are needed in the media library `Folder.vue` component — the shared folder simply won't be in the data.

**Key files:**
- `src/modules/setting/components/workspace/team/AddTeamMember.vue` — add the checkbox + tooltip
- `src/locales/en/settings.json` — add label and tooltip strings
- `src/locales/{de,el,es,fr,it,zh}/settings.json` — add English fallback keys for other locales

---

### Workflow:

1. User (workspace owner or admin) navigates to Settings → Manage Team & Roles
2. User clicks "Add Member" or edits an existing team member
3. In the Permissions section, user sees a new checkbox: "Access to shared folder" (checked by default)
4. User hovers over the info icon (`ℹ`) next to the checkbox and sees a tooltip explaining what this permission does
5. User unchecks the "Access to shared folder" checkbox
6. User saves the team member settings
7. The affected team member no longer sees the shared folder in their Media Library

---

### Acceptance criteria:

- [ ] A new checkbox labeled "Access to shared folder" appears in the Permissions section for collaborator role
- [ ] A new checkbox labeled "Access to shared folder" appears in the Permissions section for approver role
- [ ] A new checkbox labeled "Access to shared folder" appears in the Permissions section for admin role
- [ ] The checkbox is checked (on) by default for new team members
- [ ] An info icon (`ℹ`) appears next to the checkbox label
- [ ] Hovering over the info icon shows the tooltip text
- [ ] The checkbox value is saved as `accessSharedFolder` in the team member's `permissions` object when the form is submitted
- [ ] When editing an existing team member who already has this permission set, the checkbox reflects the current value
- [ ] For existing team members without this field in their data, the checkbox defaults to checked (on)

---

### UI Copy:

**Checkbox label:**
> Access to shared folder

**Info icon (`ℹ`) tooltip:**
> Allow this team member to view and use the shared folder in the Media Library. The shared folder is a common space where all permitted team members can upload, organize, and access media files together. If turned off, the shared folder won't appear in this member's Media Library at all.

**Localization keys to add in `settings.json`:**

```json
{
  "settings": {
    "workspace": {
      "team": {
        "permissions": {
          "collaborator": {
            "access_shared_folder": "Access to shared folder"
          },
          "approver": {
            "access_shared_folder": "Access to shared folder"
          },
          "admin": {
            "access_shared_folder": "Access to shared folder"
          }
        },
        "tooltips": {
          "access_shared_folder": "Allow this team member to view and use the shared folder in the Media Library. The shared folder is a common space where all permitted team members can upload, organize, and access media files together. If turned off, the shared folder won't appear in this member's Media Library at all."
        }
      }
    }
  }
}
```

**Placement:** The checkbox should appear in the existing permissions grid in `AddTeamMember.vue`:
- For **collaborator** role (lines 580-623): add after the existing "Change FB Group Publish Method" checkbox. Use the same `grid grid-cols-2 gap-3` layout. The checkbox + info icon should span 1 column, matching the pattern used by approver permissions (checkbox + `<i>` info icon).
- For **approver** role (lines 626-653): add after the existing "Can create posts & send for approval" row. Use the same `flex items-center gap-2` pattern with info icon.
- For **admin** role: add in the admin permissions section. Admins currently have limited visible permissions (e.g., `hasBillingAccess`); add this checkbox following the same pattern.

---

### Mock-ups:

N/A — follows existing checkbox + tooltip pattern already used in the permissions section (e.g., approver's "Can edit posts" with `fa-info-circle` icon).

---

### Impact on existing data:

No data migration needed on the frontend. The checkbox reads `getTeam.permissions.accessSharedFolder`. For existing team members where this key is absent, the v-model binding will be `undefined` which is falsy — however, the backend defaults it to `true`, so on the next save the correct value will be persisted. To handle the display edge case, initialize the value to `true` if undefined when loading team member data.

---

### Impact on other products:

- **Mobile apps:** No impact — team member settings are managed via web only.
- **Chrome extension:** No impact.
- **White-label:** The checkbox uses standard `<Checkbox>` component from the design library — no hardcoded colors.

---

### Dependencies:

Depends on: **[BE] Add shared folder access permission to team member roles** — the backend must accept and persist the `accessSharedFolder` permission, and filter folder listings accordingly.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
