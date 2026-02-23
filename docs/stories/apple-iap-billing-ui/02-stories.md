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
  - The "Upgrade Subscription" button in the plan header (line ~96) should also be hidden for Apple IAP users (`v-if="isPlanUpgradeable && !isAppleIapUser"`) since the new info section already has the "Upgrade Plan" CTA — avoids duplicate CTAs

---

**UI Copy — iOS Subscriber Info Card:**

**Card icon:** Apple logo or phone icon (use existing SVG from assets if available)

**Headline:**
> You're subscribed via the iOS app

**Body paragraph:**
> You're currently subscribed to ContentStudio through an Apple In-App Purchase. Because of this, your plan is priced higher than our equivalent web plans. Your current plan also comes with limited features and options — explore our web plans to see everything that's available.

**Manage subscription note** (shown just below the body paragraph, subdued gray, normal readable text size — not small):
> All plan changes, upgrades, and cancellations for your Apple subscription are managed through your iPhone. Go to **iPhone Settings → [Your Name] → Subscriptions**, find ContentStudio, and manage your plan from there.

**Upgrade prompt paragraph:**
> Want a better price or more features? You can upgrade to a web plan directly from here. However, to avoid being charged twice, you'll need to **cancel your current Apple subscription from your iPhone first** before completing the upgrade.

**Steps before upgrading** (numbered list, clearly visible, not small text):
1. On your iPhone, go to **Settings → [Your Name] → Subscriptions**
2. Find **ContentStudio** and tap **Cancel Subscription**
3. Once cancelled, come back here and click **Upgrade Plan** below
4. Choose a web plan and complete the upgrade

**Important note** (inline info banner or subdued callout):
> ContentStudio cannot cancel your Apple subscription on your behalf. This is required by Apple — only you can manage and cancel an Apple In-App Purchase from your device.

**Primary CTA button:** `Upgrade Plan`
- On click → calls `showUpgradeModal()` (the existing upgrade plan modal, already used elsewhere in this component)

---

### Workflow:

1. User subscribes to ContentStudio via the iOS app (Apple IAP)
2. User logs in to ContentStudio on the web and navigates to **Settings → Billing**
3. Instead of the empty invoice table, they see the iOS subscriber info card:
   - Headline: "You're subscribed via the iOS app"
   - Body explaining their plan is priced higher and has limited features compared to web plans
   - A note that plan changes, upgrades, and cancellations are managed through iPhone Settings
   - An upgrade prompt explaining they can switch to a web plan, but must cancel their Apple subscription first to avoid dual charges
   - A numbered step-by-step guide on how to cancel their Apple IAP before upgrading
   - An inline callout clarifying ContentStudio cannot cancel Apple subscriptions on their behalf
   - An **"Upgrade Plan"** button
4. User reads the steps, goes to their iPhone, and cancels their Apple subscription via **iPhone Settings → [Your Name] → Subscriptions**
5. User returns to the web billing page and clicks **"Upgrade Plan"**
6. The upgrade plan modal opens, showing available web plans (Standard, Advanced, Agency)
7. User selects a plan and completes the upgrade through the existing checkout flow
8. For users **without** `apple_iap`, the billing page works exactly as before — invoices table shows, no info card

---

### Acceptance criteria:

- [ ] Users with `getPlan.apple_iap === true` do **not** see the invoice/billing history section on the billing page
- [ ] Users with `getPlan.apple_iap === true` see the iOS subscriber info card in the left column where the invoices section was
- [ ] The info card displays the headline: "You're subscribed via the iOS app"
- [ ] The body paragraph reads: "You're currently subscribed to ContentStudio through an Apple In-App Purchase. Because of this, your plan is priced higher than our equivalent web plans. Your current plan also comes with limited features and options — explore our web plans to see everything that's available."
- [ ] A subscription management note is shown below the body paragraph (in subdued gray, at a legible text size — not small): "All plan changes, upgrades, and cancellations for your Apple subscription are managed through your iPhone. Go to iPhone Settings → [Your Name] → Subscriptions, find ContentStudio, and manage your plan from there."
- [ ] The upgrade prompt reads: "Want a better price or more features? You can upgrade to a web plan directly from here. However, to avoid being charged twice, you'll need to cancel your current Apple subscription from your iPhone first before completing the upgrade."
- [ ] The numbered step-by-step list is shown clearly (not in small text): Step 1: On your iPhone, go to Settings → [Your Name] → Subscriptions / Step 2: Find ContentStudio and tap Cancel Subscription / Step 3: Once cancelled, come back here and click Upgrade Plan below / Step 4: Choose a web plan and complete the upgrade
- [ ] An inline callout or banner reads: "ContentStudio cannot cancel your Apple subscription on your behalf. This is required by Apple — only you can manage and cancel an Apple In-App Purchase from your device."
- [ ] The primary CTA button is labelled **"Upgrade Plan"** and on click opens the upgrade plan modal (`showUpgradeModal()`)
- [ ] The "Upgrade Subscription" button in the plan header is hidden for Apple IAP users (the info card's CTA serves this purpose instead)
- [ ] Users without `apple_iap` see the billing page exactly as before — invoices table is shown, no info card appears
- [ ] The info card uses theme-aware Tailwind classes (`text-primary-cs-500`, `bg-primary-cs-50`, etc.) — no hardcoded color values
- [ ] All text in the card is rendered at a legible size; the cancellation steps and management note are NOT rendered in a small/muted font that makes them easy to overlook
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
