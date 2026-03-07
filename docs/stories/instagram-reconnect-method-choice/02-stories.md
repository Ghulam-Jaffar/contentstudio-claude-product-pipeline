# Stories: Instagram Reconnect Method Choice

---

## Story 1: [FE] Show connection method choice when reconnecting Instagram and display "Connected via" indicator

### Description:
As a user managing my social accounts, I want to choose whether to reconnect my Instagram account via Facebook or via Instagram directly (same choice I had during initial connection), so that I'm not locked into the original connection method and I can see how each account was connected.

Currently, when reconnecting an Instagram account, the system auto-redirects to whichever method was originally used (based on the `connected_via_instagram` flag). The user never gets to choose. Additionally, the "Connected Via" field in the details modal only shows "EasyConnect" or "ContentStudio" -- it doesn't show whether the Instagram account was connected via Facebook or via Instagram directly.

---

### Workflow:

1. User navigates to Settings > Social Accounts
2. User sees their Instagram accounts listed in the table. Each Instagram account row now shows a small label indicating the connection method: "via Facebook" or "via Instagram"
3. User clicks the "Reconnect" button on an Instagram account (from the table row actions, the actions dropdown, or the details modal)
4. Instead of being auto-redirected, user sees a small popover/dropdown with two options:
   - "Reconnect via Facebook" (with Facebook icon)
   - "Reconnect via Instagram" (with Instagram icon)
5. The currently used method is indicated with a subtle highlight or "(current)" label
6. User selects their preferred method
7. User is redirected to the appropriate OAuth flow (Facebook or Instagram Login)
8. After successful reconnection, the "Connected via" indicator updates to reflect the method just used
9. User opens the account details modal and sees the updated "Connected Via" field showing "Facebook" or "Instagram" (instead of just "ContentStudio")

---

### Acceptance criteria:

- [ ] Instagram accounts in the social accounts table show a "via Facebook" or "via Instagram" badge/label next to the account name or in a visible column
- [ ] Clicking "Reconnect" on an Instagram account (table row button) shows a popover with two options: "Reconnect via Facebook" and "Reconnect via Instagram"
- [ ] Clicking "Reconnect" on an Instagram account (details modal) shows the same popover with two options
- [ ] Clicking "Reconnect" on an Instagram account (actions dropdown in `SocialAccountsDatatable.vue`) shows the same popover with two options
- [ ] The currently used connection method is visually indicated in the popover (e.g., "(current)" suffix or a checkmark)
- [ ] Selecting "Reconnect via Facebook" triggers the Facebook OAuth flow (`instagram` type) with the correct `connector_id`
- [ ] Selecting "Reconnect via Instagram" triggers the Instagram Login flow (`instagram_login` type) with the correct `connector_id`
- [ ] After successful reconnection via a different method, the "Connected via" indicator in the table updates without a page refresh (data refetch is acceptable)
- [ ] The "Connected Via" field in the `SocialAccountDetailsModal` shows "Facebook" or "Instagram" for Instagram accounts, instead of just "ContentStudio"
- [ ] The `connectedVia` mapping in `useSocialAccounts.js` (line ~142) is updated to show "Facebook" or "Instagram" for Instagram accounts based on the `connected_via_instagram` flag
- [ ] EasyConnect accounts continue to show "EasyConnect" as the connection source (existing behavior preserved)
- [ ] Non-Instagram accounts are unaffected -- their reconnect flow remains unchanged (auto-redirect)
- [ ] The popover choice UI works in both the v2 (`SocialAccountsDatatable.vue`, `SocialAccountDetailsModal.vue`) and v1 (`AccountListing.vue`) social account views
- [ ] Bulk reconnect for Instagram accounts continues to use the per-account `connected_via_instagram` flag for each account (no choice UI needed for bulk -- existing behavior is acceptable)

---

### UI Copy:

**Reconnect popover (shown when clicking Reconnect on an Instagram account):**
- Popover items:
  - Icon: Facebook icon | Label: "Reconnect via Facebook" | Suffix on current method: "(current)"
  - Icon: Instagram icon | Label: "Reconnect via Instagram" | Suffix on current method: "(current)"

**Connection method badge in the account listing table:**
- For Instagram connected via Facebook: "via Facebook" (small gray text or subtle badge near the account name)
- For Instagram connected via Instagram directly: "via Instagram"

**"Connected Via" field in the details modal (`SocialAccountDetailsModal.vue`):**
- For Instagram accounts connected via Facebook: "Facebook"
- For Instagram accounts connected via Instagram directly: "Instagram"
- For EasyConnect accounts: "EasyConnect" (unchanged)
- For other accounts: "ContentStudio" (unchanged)

**Tooltip on the connection method badge (table listing):**
- For "via Facebook": "This account was connected using your Facebook Page. You can change this when reconnecting."
- For "via Instagram": "This account was connected directly through Instagram. You can change this when reconnecting."

---

### Mock-ups:

N/A

---

### Impact on existing data:

- No data model changes. The `connected_via_instagram` boolean flag already exists on Instagram social integration documents and is already updated correctly during reconnection by both `InstagramLoginConnector` (sets `true`) and `InstagramController` (sets `false`).
- The frontend reads `connected_via_instagram` from the API response already (`useSocialAccounts.js` line 94 and 171). Only the display mapping needs to change.

---

### Impact on other products:

- **Mobile apps**: No impact. Social account management/reconnection is web-only.
- **Chrome extension**: No impact. Does not handle social account connections.
- **White-label**: No impact. Uses the same social accounts page. The popover should use `@contentstudio/ui` components which respect white-label theming.

---

### Dependencies:

None. This is a self-contained frontend change. The backend already stores and updates `connected_via_instagram` correctly.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness tested (frontend only)
- [ ] Multilingual support verified (frontend + backend, translations available or fallback handled)
- [ ] UI theming supported (default + white-label, design library components are being used)
- [ ] White-label domains impact reviewed
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
