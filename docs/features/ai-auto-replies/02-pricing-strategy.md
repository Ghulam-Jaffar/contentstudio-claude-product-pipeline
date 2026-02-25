# AI Auto Replies — Pricing & Limits Strategy (v2)

**Shortcut Doc:** https://app.shortcut.com/contentstudio-team/write/IkRvYyI6I3V1aWQgIjY5OWI1ZGQ2LWQ0YjItNGQwNS1hODA1LTg0Njk2ZDI2MDUyYSI=
**Doc ID:** `699b5dd6-d4b2-4d05-a805-84696d26052a`
**Last Updated:** 2026-02-25

---

## What Changed from v1

v1 used a granular credit system (1 credit for intent classification, 5 credits for AI reply generation, 1 credit for rule review). This was scrapped because:

- Too complex to explain to users
- No competitor prices at the API-operation level
- Industry standard is conversations / sessions / resolutions — one atomic unit per interaction

**v2 model: 1 AI Reply = 1 deduction.** Everything else is free.

---

## Access Model

Two layers must both be active for AI features to function:

```
Layer 1: Addon purchased (workspace-level unlock)
    ↓
Layer 2: AI Reply limit available (consumed per AI-drafted/sent reply)
```

- If Layer 1 missing → feature fully locked (no rules can be created)
- If Layer 2 exhausted → addon stays unlocked, but AI reply rules pause; manual rules continue

---

## What Counts as 1 AI Reply

| Action | Counts? |
|--------|---------|
| AI **drafts** a reply (Review & Send mode — you approve first) | ✅ Yes |
| AI **sends** a reply automatically (Auto-Send mode) | ✅ Yes |
| AI intent classification (deciding if rule should fire) | ❌ Free |
| AI rule review & improvement suggestions (on save/edit) | ❌ Free |
| Manual/hardcoded reply rules (no AI) | ❌ Free |
| Rule management (create, edit, delete, toggle on/off) | ❌ Free |
| Viewing auto-reply logs | ❌ Free |

**Rule of thumb:** If AI put words in a reply — paid. Everything else — free.

---

## Addon Pricing

| | Monthly | Annual |
|-|---------|--------|
| **AI Auto Replies addon** | **$19/workspace/mo** | **$15/workspace/mo** |

- Available on **Advanced and Agency plans only** (Standard has no Inbox)
- Per-workspace — agencies with multiple workspaces pay per workspace
- Unlocks: rule creation, AI intent classification, AI reply drafting/sending, AI rule review & suggestions

---

## AI Reply Limits by Plan

Limits are per workspace, per billing month. Reset on billing date. No rollover.

| Plan | Included AI Replies/mo |
|------|------------------------|
| **Advanced** ($49/mo) | **300** |
| **Agency** ($99/mo+) | **750** |
| **Agency Unlimited** | **2,500 base + 10 per connected social account** |

### Agency Unlimited — Per Social Account Bonus

+10 AI Replies per connected social account per month (active accounts only).

| Connected Accounts | Base | Bonus | Total |
|-------------------|------|-------|-------|
| 50 accounts | 2,500 | +500 | **3,000** |
| 100 accounts | 2,500 | +1,000 | **3,500** |
| 200 accounts | 2,500 | +2,000 | **4,500** |
| 500 accounts | 2,500 | +5,000 | **7,500** |

Rationale: ties AI reply allotment directly to the scale of their ContentStudio operation; rewards agencies running more accounts; mirrors ContentStudio's existing per-account addon pricing model.

---

## Top-Up Packs

Auto-renew monthly. Multiple packs can be stacked. Deduct after plan's included replies are exhausted.

| Pack | Monthly Price | Annual Price | Per-Reply Cost |
|------|-------------|--------------|----------------|
| **250 AI Replies** | $7/mo | $6/mo | ~$0.028/reply |
| **750 AI Replies** | $17/mo | $14/mo | ~$0.023/reply (~18% savings) |
| **2,500 AI Replies** | $45/mo | $36/mo | ~$0.018/reply (~36% savings) |

---

## Free Trial

**25 AI Replies** on first addon activation — one-time, non-renewable.
Enough to test ~25 AI-drafted or auto-sent replies before committing.

---

## Exhaustion Behavior

When AI Replies hit 0 mid-month:

1. AI rules pause — keyword triggers still match, but AI classification is skipped; the rule does not fire
2. Manual/hardcoded rules continue unaffected
3. In-app banner in Inbox and Auto-Replies settings: *"You've used all your AI Replies for this month. [Buy more] or wait for reset on [date]."*
4. Email notification to workspace owner at 80% usage and again at 0%
5. No automatic top-up — user must manually purchase or wait for reset

**What stays on at 0 AI Replies:**
- Rule management (create, edit, delete, toggle) — no replies consumed
- Viewing auto-reply logs
- Keyword matching (evaluated, just not acted on)
- Manual hardcoded reply rules — reply sends with no AI
- AI rule review & suggestions on save — free operation

---

## Profitability Analysis

Estimated AI cost per reply (Claude Haiku or equivalent, classification + generation combined): **~$0.002–$0.005/reply**.

| Scenario | Revenue | AI Cost | Gross Margin |
|----------|---------|---------|-------------|
| Addon only, Advanced (300 replies used) | $19 | ~$1.00 | ~$18 (~95%) |
| Addon only, Agency (750 replies used) | $19 | ~$2.50 | ~$16.50 (~87%) |
| Addon + 750-reply pack (1,500 replies) | $36 | ~$5.00 | ~$31 (~86%) |
| Agency Unlimited, 200 accounts (4,500 replies) | $19 | ~$15 | ~$4 (~21%) |
| Agency Unlimited + 2,500-reply pack (7,000 replies) | $64 | ~$23 | ~$41 (~64%) |

**Notes:**
- Agency Unlimited with heavy use and no top-ups is the thin-margin edge case — solved by design (heavy users buy top-ups)
- Review & suggestions being free has near-zero cost impact (one lightweight call per rule save, not per incoming message)
- If 30%+ of Agency Unlimited users exhaust their limit and buy one top-up pack, blended margin across the tier is strong

---

## Billing Rules

- Limits reset on **workspace billing date** — no rollover
- Top-up packs renew monthly alongside main subscription
- Plan downgrade mid-cycle: limit caps to new plan's allotment at next reset
- Addon cancellation: active AI rules pause immediately; rule data retained 90 days
- Usage counter visible in Inbox sidebar and Auto-Replies settings: *"218 / 300 AI Replies used this month"*

---

## Rollout Checklist

- [ ] Addon upsell shown only on Advanced and Agency plan settings — not visible to Standard users
- [ ] Agency Unlimited per-account bonus computed from active connected accounts, updated daily
- [ ] Enterprise: custom AI reply ceiling negotiated per contract
- [ ] Top-up packs available from workspace billing page and from inline "AI Replies exhausted" banner
- [ ] First-time activation: 25 free AI Replies (one-time, non-renewable)
- [ ] Usage counter in Inbox sidebar and Auto-Replies settings

---

## Summary Table

| | Advanced | Agency | Agency Unlimited (100 accts) |
|-|----------|--------|------------------------------|
| Addon price | $19/mo | $19/mo | $19/mo |
| Included AI Replies/mo | 300 | 750 | 3,500 |
| Top-up (smallest pack) | $7/250 | $7/250 | $7/250 |
