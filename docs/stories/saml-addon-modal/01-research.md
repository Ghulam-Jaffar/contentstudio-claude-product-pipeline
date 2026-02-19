# Research — SAML Addon Modal (FE)

## Current State

The SAML/SSO settings page (`Settings → SSO`) is accessible by all users with no subscription gating. It's a 3-step wizard (Add Domain → Verify Domain → Configure IdP).

Key files:
- **Page component:** `src/modules/setting/components/DomainVerification.vue` — no addon check at all currently
- **Route:** `sso-domains` in `src/modules/setting/config/routes/setting.js`
- **Sub-components:** `SsoAddDomain.vue`, `SsoVerifyDomain.vue`, `SsoIdpConfig.vue` (all in `src/modules/setting/components/sso/`)
- **Reference modal:** `src/modules/setting/components/white-label/WhiteLabelUpgradeModal.vue`
- **Reference page:** `src/modules/setting/components/white-label/WhiteLabelMain.vue`
- **Feature flags:** `src/modules/billing/constants/featureList.js` — has `white_label_addon`, no SAML key yet
- **Feature composable:** `src/modules/billing/composables/useFeatures.js` — `canAccess(feature)` returns `{ allowed: bool, error: { type, message } }`

## Reference Pattern (WhiteLabel)

`WhiteLabelMain.vue` uses this pattern when the user doesn't have the addon:
```html
<template v-if="!isWhiteLabelUnlocked">
  <div class="z-100 fixed top-[50%] left-[51%] ...">
    <WhiteLabelUpgradeModal />
  </div>
  <div class="bg-[#595c5f40] ... absolute"></div>  <!-- semi-transparent backdrop -->
</template>
<!-- actual page content always renders behind the overlay -->
```

`WhiteLabelUpgradeModal.vue` structure:
- Title + description paragraph
- Feature checklist (3 bullet items with tick icons)
- Monthly / Annual plan pricing cards (radio toggle)
- "Purchase Now" CTA → opens Paddle checkout via `openCheckout()`
- YouTube embed for explainer video

## What Needs to Change

1. **New component:** `src/modules/setting/components/sso/SamlUpgradeModal.vue`
   - Mirrors `WhiteLabelUpgradeModal.vue` structure
   - Monthly-only pricing ($150/month) — no annual option
   - SAML-specific copy, feature list, and YouTube embed (TBD — placeholder URL)
   - Paddle product IDs for SAML addon (BE will provide; story must call this out)

2. **Modify `DomainVerification.vue`:**
   - Import `useFeatures` and call `canAccess('saml_sso_addon')`
   - Add overlay + `<SamlUpgradeModal />` when not subscribed (same pattern as WhiteLabel)

3. **Modify `src/modules/billing/constants/featureList.js`:**
   - Add `'saml_sso_addon'` to the `ALL_FEATURES` array

4. **Modify `src/modules/billing/constants/featureMessages.js`:**
   - Add message entry for `saml_sso_addon`

## Files Involved

| File | Change |
|---|---|
| `src/modules/setting/components/sso/SamlUpgradeModal.vue` | New component |
| `src/modules/setting/components/DomainVerification.vue` | Add addon check + overlay |
| `src/modules/billing/constants/featureList.js` | Add `saml_sso_addon` feature key |
| `src/modules/billing/constants/featureMessages.js` | Add SAML message |

## Story Split

Single `[FE]` story — no backend changes needed for the modal (BE already controls the feature flag via subscription `features` map). No mobile impact (SSO/SAML is a web-only enterprise feature).
