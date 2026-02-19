# Stories — SAML Addon Modal (FE)

---

## [FE] Show SAML/SSO addon upgrade modal in SSO settings

### Description:

The SAML/SSO settings page (`Settings → SSO`) is currently accessible to all users regardless of their subscription plan. SAML SSO is a premium add-on priced at **$150/month**. When a user's workspace does not have the SAML SSO add-on, the SSO settings page should display an upgrade modal overlay — matching the same pattern used by the White Label settings page (`WhiteLabelMain.vue` + `WhiteLabelUpgradeModal.vue`) — prompting the user to purchase the add-on via Paddle checkout.

**What needs to be built:**

1. **New component: `src/modules/setting/components/sso/SamlUpgradeModal.vue`**
   - Mirrors the structure of `WhiteLabelUpgradeModal.vue`
   - Title, description paragraph, 5-item feature checklist, $150/month pricing card (monthly only — no annual toggle), "Purchase Now" CTA button, optional explainer video embed
   - Uses `usePaddle` composable to open Paddle checkout for the SAML SSO add-on product ID
   - Uses `usePermissions` to check `can_see_subscription` — hides the CTA and shows "Contact your workspace admin to purchase this add-on." for users without billing permission
   - Uses `CstButton` from `@contentstudio/ui`

2. **Modify `src/modules/setting/components/DomainVerification.vue`:**
   - Import `useFeatures` and call `canAccess('saml_sso_addon')`
   - Add the modal overlay template block when the user doesn't have access (same fixed+backdrop pattern as `WhiteLabelMain.vue` lines 199–209)

3. **Modify `src/modules/billing/constants/featureList.js`:**
   - Add `'saml_sso_addon'` to the `ALL_FEATURES` array

4. **Modify `src/modules/billing/constants/featureMessages.js`:**
   - Add message entry for `saml_sso_addon`

5. **Modify `src/common/constants/pricing.js`:**
   - Add Paddle product ID(s) for the SAML SSO add-on (monthly plan) — coordinate with backend team for the actual Paddle product ID

---

**UI Copy — SamlUpgradeModal.vue:**

**Modal title:**
> Unlock SAML Single Sign-On

**Description paragraph:**
> Give your team a seamless, secure login experience. With SAML SSO, your employees sign in to ContentStudio using your company's existing identity provider — like Okta, Azure AD, or Google Workspace — no separate passwords to manage.

**Feature checklist (5 items, each with a green tick icon):**
1. Connect any SAML 2.0 identity provider (Okta, Azure AD, Google Workspace, OneLogin, and more)
2. Enforce company-wide login policies from one central place
3. One login for everything — no password fatigue for your team
4. Instant deprovisioning — remove an employee's access in seconds
5. Full audit trail — see who logged in and when

**Pricing card:**
- Label: `Monthly`
- Price: `$150 / month`
- Subtext: `Billed monthly. Cancel anytime.`
- (No annual toggle — monthly only)

**Primary CTA button:** `Purchase Now`

**For users without `can_see_subscription` permission** (instead of the CTA):
> Only workspace admins can manage billing. Contact your admin to purchase the SAML SSO add-on.

**Backdrop:** Semi-transparent dark overlay (`bg-[#595c5f40]`) covering the SSO page behind the modal, preventing interaction with the underlying page.

---

### Workflow:

1. User navigates to **Settings → SSO** (the `sso-domains` route, rendered by `DomainVerification.vue`)
2. If the user's workspace **does not** have the `saml_sso_addon` subscription feature:
   a. The SSO setup wizard renders in the background (non-interactive, covered by the backdrop)
   b. A centered upgrade modal appears on top with the title, description, feature checklist, pricing, and "Purchase Now" button
3. User reads the modal and clicks **"Purchase Now"**
4. Paddle checkout opens for the SAML SSO add-on ($150/month)
5. User completes the payment flow
6. On success, the page refreshes / the subscription features reload, the modal disappears, and the user can now access the full SSO setup wizard
7. If the user **does have** the `saml_sso_addon` feature, the page loads normally — no modal shown, the SSO wizard is fully accessible

---

### Acceptance criteria:

- [ ] Users whose workspace does **not** have `saml_sso_addon` see the upgrade modal overlay when visiting Settings → SSO
- [ ] Users whose workspace **does** have `saml_sso_addon` see the normal SSO setup wizard with no modal
- [ ] The modal displays: title "Unlock SAML Single Sign-On", description paragraph, 5-item feature checklist with tick icons, $150/month pricing card with "Billed monthly. Cancel anytime." subtext, and "Purchase Now" CTA
- [ ] Clicking "Purchase Now" opens Paddle checkout for the SAML SSO add-on (monthly)
- [ ] The modal is centered on screen with a semi-transparent dark backdrop (`bg-[#595c5f40]`) covering the SSO page behind it — the backdrop prevents interaction with the underlying content
- [ ] Users without `can_see_subscription` permission see the modal without the "Purchase Now" button; instead they see: "Only workspace admins can manage billing. Contact your admin to purchase the SAML SSO add-on."
- [ ] `'saml_sso_addon'` is added to `ALL_FEATURES` in `featureList.js` so `canAccess('saml_sso_addon')` returns the correct access state
- [ ] The modal uses theme-aware Tailwind classes (`text-primary-cs-500`, `bg-primary-cs-50`, etc.) — no hardcoded color values
- [ ] The component uses `CstButton` from `@contentstudio/ui` for the CTA, consistent with `WhiteLabelUpgradeModal.vue`
- [ ] The modal is responsive and displays correctly on viewports ≥ 768px (SSO is a web-only enterprise feature)

---

### Mock-ups:

N/A — follow the exact layout of `WhiteLabelUpgradeModal.vue` as the reference implementation. Replace White Label content with SAML copy and remove the annual plan toggle (monthly only).

---

### Impact on existing data:

None. This is a UI-only gate. The `saml_sso_addon` feature flag is already controlled server-side via the subscription `features` map. No schema or data changes.

---

### Impact on other products:

- **Mobile apps (iOS/Android):** Not affected. SAML/SSO is a web-only enterprise feature; mobile apps do not have an SSO settings section.
- **Chrome extension:** Not affected.
- **White-label:** The upgrade modal uses theme-aware color classes so it will automatically respect white-label primary color themes.

---

### Dependencies:

1. **`saml_sso_addon` feature flag in subscription system (BE):** The backend must add `saml_sso_addon` as a recognized feature key in the subscription plan features map so that `canAccess('saml_sso_addon')` returns the correct access state per workspace. Without this, all users would either see the modal permanently or never see it.

2. **Paddle product ID for SAML SSO add-on (BE/Billing):** The Paddle product ID for the monthly $150/month SAML SSO add-on must be configured and provided by the backend team. It needs to be added to `src/common/constants/pricing.js` alongside the existing white-label Paddle IDs. The frontend component can be built and merged behind the feature flag, but checkout will not function in production until this is in place.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (SSO settings is a web-only enterprise feature; not available on mobile viewports)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
