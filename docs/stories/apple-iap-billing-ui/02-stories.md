# Stories — Apple IAP Billing UI

---

## [FE] Show iOS subscriber info section on web billing page instead of invoices

### Description:

Users who purchased ContentStudio via Apple in-app purchase (IAP) on the iOS app and then log in to the web see a confusing billing page: the invoice table is empty (no Paddle invoices exist for Apple subscribers), and the right-side action buttons are already disabled by the backend. The page gives no explanation of why.

This story adds an informational UI section to the billing page that:
1. **Hides the invoice/billing history section** for Apple IAP users (no Paddle invoices exist for them)
2. **Shows a dedicated iOS subscriber info card** in its place, explaining their situation and prompting them to upgrade to a web plan for full access and better pricing

**How to detect Apple IAP users:**
- Check `getPlan.apple_iap === true` from the Vuex store (already populated by the backend)

**Files to modify:**
- `src/modules/setting/components/billing/EnrolledPlanView.vue`
  - Add `isAppleIapUser` computed based on `getPlan.apple_iap`
  - Wrap the invoice section (lines ~445–561) with `v-if="!isAppleIapUser"`
  - Add the new iOS info section with `v-if="isAppleIapUser"` in the same location
  - The "Upgrade Subscription" button in the plan header (line ~96) should also be hidden for Apple IAP users (`v-if="isPlanUpgradeable && !isAppleIapUser"`) since the new info section already has the upgrade CTA — avoids duplicate CTAs

---

**UI Copy — iOS Subscriber Info Card:**

**Card icon:** Apple logo or phone icon (use existing SVG from assets if available)

**Headline:**
> You're subscribed via the iOS app

**Body paragraph:**
> Because Apple charges a 30% fee on all in-app purchases, your iOS plan is priced higher than our equivalent web plans. On your current iOS plan, some features are not available — including Inbox and Competitor Analytics.

**Feature gap list** (shown as a short bullet list with lock or info icons):
- Social Inbox — not available on iOS plans
- Competitor Analytics — not available on iOS plans
- Add-on management — not available on iOS plans

**Upgrade prompt paragraph:**
> Upgrade to a web plan to unlock all features at a lower price. You can choose from our Standard, Advanced, or Agency plans.

**Primary CTA button:** `Explore Web Plans`
- On click → calls `showUpgradeModal()` (the existing upgrade plan modal, already used elsewhere in this component)

**Manage Apple subscription note** (smaller text below the CTA, subdued gray):
> To cancel your subscription, you'll need to do it directly from your iPhone — ContentStudio cannot cancel Apple subscriptions on your behalf. Go to **Settings → [Your Name] → Subscriptions** on your iPhone, find ContentStudio, and tap **Cancel Subscription**.
> [Learn how to cancel →](#) *(links to ContentStudio help doc — URL to be provided by product/support team)*

---

### Workflow:

1. User subscribes to ContentStudio via the iOS app (Apple IAP)
2. User logs in to ContentStudio on the web and navigates to **Settings → Billing**
3. Instead of the empty invoice table, they see the iOS subscriber info card:
   - Headline and explanation of iOS pricing vs web pricing
   - List of features not available on their current plan
   - "Explore Web Plans" CTA
   - Note on how to manage their Apple subscription
4. User clicks **"Explore Web Plans"**
5. The upgrade plan modal opens, showing available web plans (Standard, Advanced, Agency)
6. User selects a plan and completes the upgrade through the existing checkout flow
7. For users **without** `apple_iap`, the billing page works exactly as before — invoices table shows, no info card

---

### Acceptance criteria:

- [ ] Users with `getPlan.apple_iap === true` do **not** see the invoice/billing history section on the billing page
- [ ] Users with `getPlan.apple_iap === true` see the iOS subscriber info card in the left column where the invoices section was
- [ ] The info card displays: headline "You're subscribed via the iOS app", explanation paragraph mentioning Apple's 30% fee, feature gap list (Inbox, Competitor Analytics, Add-on management), upgrade prompt, and "Explore Web Plans" button
- [ ] Clicking "Explore Web Plans" opens the upgrade plan modal (same as the existing `showUpgradeModal()` call)
- [ ] The cancellation note below the CTA reads: "To cancel your subscription, you'll need to do it directly from your iPhone — ContentStudio cannot cancel Apple subscriptions on your behalf. Go to Settings → [Your Name] → Subscriptions on your iPhone, find ContentStudio, and tap Cancel Subscription."
- [ ] A "Learn how to cancel →" link is shown below the cancellation note, linking to the ContentStudio help doc (URL to be provided before release)
- [ ] The "Upgrade Subscription" button in the plan header is hidden for Apple IAP users (the info card's CTA serves this purpose instead)
- [ ] Users without `apple_iap` see the billing page exactly as before — invoices table is shown, no info card appears
- [ ] The info card uses theme-aware Tailwind classes (`text-primary-cs-500`, `bg-primary-cs-50`, etc.) — no hardcoded color values
- [ ] The info card is consistent with the existing billing page visual style

---

### Mock-ups:

N/A — style the card consistently with existing billing page sections (same heading style as `settings.billing.enrolled_plan.headings.*`, use `CstBanner` or a simple card with the existing `subscription-plan-widget` CSS class pattern if applicable).

---

### Impact on existing data:

None. Read-only UI change — reads `getPlan.apple_iap` from the existing store state.

---

### Impact on other products:

- **iOS/Android:** Not applicable — this change is on the web billing page only.
- **Chrome extension:** Not affected.
- **White-label:** The info card uses theme-aware color classes, so it respects white-label themes. Note: Apple IAP users are unlikely to be white-label customers, but the styling should be correct regardless.

---

### Dependencies:

The backend must expose `apple_iap: true` in the `POST /billing/plan` API response (`getPlan` in the Vuex store) for Apple IAP subscribers. The user confirmed this is already handled by the backend.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
