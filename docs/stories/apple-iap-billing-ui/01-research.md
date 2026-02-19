# Research — Apple IAP Billing UI (Web)

## Existing Story

Story 106546 "Handle Web Billing UI for Apple IAP Subscribers" — very thin stub with minimal AC. This research is for a full rewrite.

## Current State

**Backend:**
- `apple_iap` flag is set on the `User` model in `AppleNotificationService.php` (line 406) when an Apple IAP subscription is processed
- Apple IAP subscriptions lock automations and disable white_label (handled in `AppleNotificationService.php`)
- The `GET /billing/subscriptions` endpoint (`PlanController::getUserSubscriptions`) does **NOT** currently expose `apple_iap` in its response — FE has no way to know if the user is on Apple IAP
- Migration `2026_01_07_070000_add_apple_standard_monthly_plan.php` confirms Apple IAP plan with `is_apple_iap: true` at the subscription level too

**Frontend — `EnrolledPlanView.vue`:**
- **Invoice section** (lines 445–561): Renders `getPaddleTransactionsHistory` — a Paddle-fetched list. Apple IAP users have no Paddle invoices so this renders empty, but the full table/section is still shown
- **Right-side panel** (line 565+):
  - Limits panel: always shown — OK for Apple IAP (can see usage)
  - "Manage Add-ons" button: `disabled` when `!addonAccess.allowed` — already disabled via BE feature flags for Apple IAP users ✓
  - "Upgrade Subscription" button (`isPlanUpgradeable`): returns `true` for Apple IAP plan slugs (e.g. `standard-monthly`) — currently shown but leads to upgrade flow that shouldn't be the primary CTA here
- **No Apple IAP detection in the FE at all** — no `apple_iap` or `is_apple_iap` reference anywhere in the frontend codebase

## What Needs to Change

**[BE]** — Expose Apple IAP flag in subscriptions API:
- In `PlanController::getUserSubscriptions`, include `is_apple_iap: true` in `mainSubscription` response when `$user->apple_iap === true`
- File: `app/Http/Controllers/Billing/PlanController.php` (around line 256, in the return payload)

**[FE]** — Update `EnrolledPlanView.vue`:
1. Detect `mainSubscription.is_apple_iap` (after BE exposes it)
2. **Hide invoice section** when `isAppleIapUser` — no Paddle invoices exist
3. **Show new Apple IAP info section** in place of invoices — tells user they're on an iOS plan, what features they're missing, and prompts them to upgrade on web via the existing `showUpgradeModal()`
4. **Right-side Upgrade/Manage-Addons CTAs**: already disabled by BE — confirm no FE changes needed there (the "Upgrade Subscription" button from `isPlanUpgradeable` may need to be hidden for Apple IAP users since the info section already has the upgrade CTA)

## Files Involved

| File | Change |
|---|---|
| `app/Http/Controllers/Billing/PlanController.php` | Add `is_apple_iap` to API response (BE) |
| `src/modules/setting/components/billing/EnrolledPlanView.vue` | Detect Apple IAP, hide invoices, show info section (FE) |

## Story Split

2 stories: `[BE]` + `[FE]`
