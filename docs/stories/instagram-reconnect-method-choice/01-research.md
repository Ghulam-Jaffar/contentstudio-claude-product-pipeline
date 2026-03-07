# Research: Instagram Reconnect Method Choice

## Current State

### How connection works today
When a user connects an Instagram account for the first time, they see two options:
1. **Connect via Facebook** (uses Facebook OAuth, sets `connected_via_instagram = false`)
2. **Connect via Instagram directly** (uses Instagram Login API, sets `connected_via_instagram = true`)

### How reconnection works today
When a user reconnects an Instagram account (from the table row button, the details modal, or the actions dropdown), the system **auto-redirects** to whichever method was originally used, based on the `connected_via_instagram` flag:
- `connected_via_instagram === true` -> redirects to `instagram_login` (direct Instagram)
- `connected_via_instagram === false` -> redirects to `instagram` (via Facebook)

The user is **never shown a choice** during reconnection -- it just auto-redirects.

### Where "Connected Via" is displayed today
- **Details modal** (`SocialAccountDetailsModal.vue:154`): Shows a "Connected Via" field but it only displays "EasyConnect" or "ContentStudio" -- **not** whether it was Facebook or Instagram direct.
- **Account listing table**: No visible indicator of connection method (FB vs Instagram direct).
- **Actions dropdown** (`SocialAccountsDatatable.vue:265-281`): Shows a single "Reconnect" button with no method info.

### Key backend field
- `connected_via_instagram` (boolean) stored on the Instagram social integration document in MongoDB
- Set to `true` in `InstagramLoginConnector.php:268` (direct Instagram)
- Set to `false` in `InstagramController.php:414` (via Facebook)
- Read in `InstagramHelper.php:31` via `isConnectedViaInstagram()`

## What Needs to Change

### Backend
- No new API endpoints needed. The `connected_via_instagram` flag is already returned in the API response and used by the frontend.
- When a user reconnects via a *different* method than original, the `connected_via_instagram` flag must be updated accordingly:
  - `InstagramLoginConnector::createAccountData()` already sets `connected_via_instagram = true`
  - `InstagramController` already sets `connected_via_instagram = false`
  - `InstagramRepo::saveIGBusinessProfile()` handles upsert -- this should already update the flag on reconnection. **Needs verification** that the flag is included in the update fields.

### Frontend
1. **Reconnect flow (table + modal + dropdown)**: Instead of auto-redirecting, show a small choice UI (e.g., a dropdown or mini-modal) letting the user pick "Reconnect via Facebook" or "Reconnect via Instagram" -- same as the initial connection experience.
2. **"Connected Via" display**: Update the label in the details modal, account listing table, and reconnect dropdown to show the actual connection method: "Facebook" or "Instagram" (not just "EasyConnect" / "ContentStudio").
3. **Dynamic update**: After reconnecting with a different method, the "Connected Via" display should reflect the new method.

## Files Involved

### Backend
- `app/Strategy/Integrations/InstagramLoginConnector.php` -- direct Instagram connector (sets `connected_via_instagram = true`)
- `app/Http/Controllers/Integrations/Platforms/Social/InstagramController.php` -- FB-based connector (sets `connected_via_instagram = false`)
- `app/Repository/Integrations/Platforms/Social/InstagramRepo.php` -- `saveIGBusinessProfile()` upsert logic
- `app/Helpers/Integrations/InstagramHelper.php` -- `isConnectedViaInstagram()` helper

### Frontend (v2 social accounts -- current/active)
- `src/modules/integration/components/platforms/social_v2/composables/useSocialAccounts.js` -- `handleReconnectAccount()` at line 692, `connectedVia` mapping at line 142
- `src/modules/integration/components/platforms/social_v2/components/SocialAccountsDatatable.vue` -- reconnect button in actions column (line 265-281)
- `src/modules/integration/components/platforms/social_v2/components/SocialAccountDetailsModal.vue` -- "Connected Via" display (line 154), reconnect handler (line 709)

### Frontend (v1 social accounts -- legacy, still active)
- `src/modules/integration/components/platforms/social/AccountListing.vue` -- `onReconnectClick()` at line 1232, `connectAccount()` at line 840
- `src/modules/integration/components/sections/IntegrationSection.vue` -- reconnect button (line 197-213)
