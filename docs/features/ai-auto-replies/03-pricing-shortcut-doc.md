# AI Auto Replies — Pricing & Limits Strategy (v3)

## Feature Summary

AI Auto Replies is a rule-based automation layer on top of ContentStudio's Social Inbox. Users define rules with keyword triggers, intent instructions, account targeting, and a reply type (AI-generated or hardcoded manual). When a message arrives, the AI classifies whether the rule's intent truly applies, then drafts or sends a reply automatically. After saving a rule, AI reviews it and suggests improvements to triggers and intent instructions.

---

## Naming

- **Feature name:** AI Auto Replies
- **Purchasable top-up unit:** AI Inbox Replies

"AI Inbox Replies" is used specifically for the quota unit — how many AI-drafted or AI-sent replies are available. The name is concrete (100 AI Inbox Replies = AI will write/send up to 100 replies), avoids the confusion of "credits", and sits naturally alongside AI Text Credits, AI Image Credits, AI Video Credits in ContentStudio's billing UI.

---

## What Changed from Previous Drafts

**v1 → v2:** Replaced a 3-tier credit system (1 credit classification, 5 credits generation, 1 credit review) with a flat "1 AI reply = 1 deduction" model. Review & suggestions made free.

**v2 → v3:**
- Corrected plan names: Standard / Advanced / Agency Unlimited (not Agency Small/Medium/Large)
- Replaced arbitrary per-plan allotments with a **single formula** derived from social accounts
- Separated the unlock fee from the top-up clearly in language and structure
- Renamed top-up unit to "AI Inbox Replies"

---

## What Counts as 1 AI Inbox Reply

| Action | Counts? |
|--------|---------|
| AI **drafts** a reply (Review & Send mode — you approve before it goes out) | ✅ Yes — 1 AI Inbox Reply |
| AI **sends** a reply automatically (Auto-Send mode) | ✅ Yes — 1 AI Inbox Reply |
| AI intent classification (deciding if a rule should fire) | ❌ Free |
| AI rule review & improvement suggestions (on save/edit) | ❌ Free |
| Manual/hardcoded reply rules (no AI involved) | ❌ Free |
| Rule management (create, edit, delete, toggle) | ❌ Free |
| Viewing auto-reply logs | ❌ Free |

**Rule of thumb:** If AI put words in a reply — it counts. Everything else is free.

---

## Competitor Benchmarks

### Social Media Management Tools (AI inbox included in plan)

| Tool | Plans | AI Inbox / Auto-Reply | AI Pricing Model |
|------|-------|----------------------|-----------------|
| **Hootsuite** | From $99/mo | AI Smart Replies (Advanced/Enterprise only) — suggests contextual replies for agents to approve or send. Full AI chatbot on Enterprise. | Bundled into plan — no per-reply charge. AI features unavailable without upgrading to expensive tiers. |
| **Sprout Social** | From $249/seat/mo | AI-assisted inbox suggestions, bot builder on Advanced. Automated chatbot for DMs. | Bundled into plan — no per-reply charge. Effectively enterprise-only pricing. |
| **Agorapulse** | From $79/user/mo | AI Reply Suggestions (April 2025). Automated inbox assistant on Advanced. | Bundled — no per-reply charge. Feature availability varies by tier. |

### Dedicated AI Chat / Support Tools

| Tool | Plan | AI Auto-Reply Access | Price | Usage Limit |
|------|------|---------------------|-------|-------------|
| **Tidio Lyro** | Lyro AI Agent | AI conversation agent for website + social chat | $39–$149/mo | 50–1,000 conversations/mo (conversation = full multi-message session) |
| **Intercom Fin** | Any seat plan + Fin | Autonomous AI agent resolves support tickets | $0.99/resolution + $29–132/seat/mo | Pay-per-use, no cap |
| **Freshdesk Freddy AI** | Growth+ plans | AI bot handles sessions; hands off to human agents | $100 per 1,000 sessions | 500 free sessions; session = unique convo in 24-hr window |
| **ManyChat** | Pro + AI addon | AI intention recognition, AI reply steps in flows | $15/mo (contact-based) + $29/mo AI addon | Unlimited AI steps on Pro+AI |
| **Crisp** | Essentials / Plus | AI-powered resolutions built into platform | €95/mo / €295/mo | 50 AI uses/mo on Essentials; unlimited on Plus |

### Key Takeaways

- SMM tools bundle AI into expensive plans ($79–$249+/user/mo) — no per-reply pricing but the floor cost is prohibitive
- Dedicated AI tools price on conversations/sessions/resolutions — not tokens or raw API usage
- No SMMT competitor offers an affordable AI auto-reply unlock with a transparent per-reply quota — ContentStudio fills this gap at $19/mo

---

## Access Model

**Step 1 — Unlock:** Pay $19/mo to activate AI Auto Replies for the workspace. This unlocks the feature and includes a base allotment of AI Inbox Replies derived from the plan's social account count.

**Step 2 — Top up (optional):** If the base allotment is exceeded, purchase additional AI Inbox Replies in increments from the billing page.

```
Plan subscription (Advanced / Agency Unlimited)
    → Includes Social Inbox access
        → Unlock AI Auto Replies ($19/mo)
            → Base AI Inbox Replies allotment (derived from social accounts)
                → Top up AI Inbox Replies if needed (+$5/mo per 100 replies)
```

---

## Unlock Fee

|  | Monthly | Annual |
|--|---------|--------|
| **AI Auto Replies unlock** | **$19/workspace/mo** | **$15/workspace/mo** |

- Available on **Advanced and Agency Unlimited plans only** (Standard has no Social Inbox)
- Per-workspace — agencies with multiple workspaces pay per workspace
- Unlocks: rule creation, AI intent classification, AI reply drafting/sending, AI rule review & suggestions

---

## The Formula

> **1 social account = 20 AI Inbox Replies/month**

This single rate drives base allotments, per-account scaling, and top-up increments uniformly. No separate logic per plan — everything flows from account count.

---

## Base Allotment by Plan

Allotment is calculated from the plan's included social account count at the time of unlock.

| Plan | Price | Included Social Accounts | Base AI Inbox Replies/mo |
|------|-------|--------------------------|--------------------------|
| **Standard** | $19/mo | 5 accounts | — (no Social Inbox) |
| **Advanced** | $49/mo | 10 accounts | **200 AI Inbox Replies/mo** |
| **Agency Unlimited** | $99/mo+ | 25 accounts (base) | **500 AI Inbox Replies/mo** |

---

## Agency Unlimited — Per Social Account Scaling

Agency Unlimited users add social accounts over time. Each additional connected account increases the AI Inbox Replies allotment by the same rate:

**+20 AI Inbox Replies per connected social account per month**

| Total Connected Accounts | AI Inbox Replies/mo |
|--------------------------|---------------------|
| 25 (base) | 500 |
| 50 | 1,000 |
| 100 | 2,000 |
| 200 | 4,000 |
| 500 | 10,000 |

Allotment is recalculated when accounts are added or removed, taking effect from the next billing cycle.

---

## Top-Up: AI Inbox Replies

When the monthly allotment is exhausted, users can purchase additional AI Inbox Replies from the billing page. Follows ContentStudio's standard addon increment model:

| Increment | Price |
|-----------|-------|
| **100 AI Inbox Replies** | **$60/year ($5/mo)** |

- Multiple increments can be purchased and stacked
- Each increment of 100 replies = 5 accounts' worth of allotment at the standard rate
- Increments renew annually alongside the main subscription
- Top-up deducts after the base allotment is exhausted

**Examples:**

| Extra replies needed | Increments to buy | Annual cost |
|---------------------|-------------------|-------------|
| ~100 | 1 | $60/yr |
| ~200 | 2 | $120/yr |
| ~500 | 5 | $300/yr |

---

## Profitability Check

Estimated AI cost per reply (Claude Haiku-level, classification + generation combined): **~$0.002–$0.005/reply**

| Scenario | Unlock Revenue | AI Cost | Gross Margin |
|----------|---------------|---------|-------------|
| Advanced, full allotment used (200 replies) | $19/mo | ~$0.60 | ~$18.40 (~97%) |
| Agency Unlimited base, full use (500 replies) | $19/mo | ~$1.50 | ~$17.50 (~92%) |
| Agency Unlimited, 100 accounts, full use (2,000 replies) | $19/mo | ~$6.00 | ~$13.00 (~68%) |
| Advanced + 1 top-up increment (300 replies) | $24/mo | ~$0.90 | ~$23.10 (~96%) |
| Agency Unlimited, 100 accounts + 5 increments (2,500 replies) | $44/mo | ~$7.50 | ~$36.50 (~83%) |

At realistic AI generation costs, margins remain strong even at high usage. The $19 unlock covers AI costs many times over at base allotment levels.

---

## Exhaustion Behavior

When AI Inbox Replies hit 0 mid-month:

1. **AI rules pause** — keyword triggers still match, but AI classification is skipped; the rule does not fire
2. **Manual/hardcoded rules continue** unaffected
3. **In-app banner** in Inbox and Auto-Replies settings: *"You've used all your AI Inbox Replies for this month. [Buy more] or wait for reset on [date]."*
4. **Email notifications** to workspace owner at 80% usage and again at 0%
5. **No automatic top-up** — user must manually purchase or wait for reset

### What Stays On at 0 AI Inbox Replies

- Rule management (create, edit, delete, toggle)
- Viewing auto-reply logs
- Keyword matching (evaluated, not acted on)
- Manual hardcoded reply rules (send as normal)
- AI rule review & suggestions on save (free operation)

---

## Billing & Reset Rules

- Allotment resets on **workspace billing date** each month — no rollover
- Top-up increments renew annually alongside main subscription
- Plan downgrade: allotment recalculated from new plan's account count at next reset
- Account removal (Agency Unlimited): allotment recalculated at next billing cycle
- Unlock cancellation: active AI rules pause immediately; rule data retained 90 days
- **Free trial:** 50 AI Inbox Replies on first unlock activation (one-time, non-renewable) — enough to test ~50 AI-drafted replies

---

## Rollout Checklist

- [ ] Unlock only visible on Advanced and Agency Unlimited plan billing — hidden from Standard users
- [ ] Base allotment calculated from connected social account count at unlock time
- [ ] Agency Unlimited per-account scaling recalculates on account add/remove (effective next billing cycle)
- [ ] Top-up increments available from billing page and from inline "AI Inbox Replies exhausted" banner
- [ ] Usage counter in Inbox sidebar and Auto-Replies settings: *"143 / 200 AI Inbox Replies used this month"*
- [ ] Free trial: 50 AI Inbox Replies on first activation (one-time, non-renewable)
- [ ] Enterprise: custom allotment negotiated per contract

---

## Summary

| | Advanced | Agency Unlimited (base) | Agency Unlimited (100 accts) |
|-|----------|------------------------|------------------------------|
| Plan price | $49/mo | $99/mo | $99/mo+ |
| Unlock fee | $19/mo | $19/mo | $19/mo |
| Formula | 10 accts × 20 | 25 accts × 20 | 100 accts × 20 |
| AI Inbox Replies/mo | **200** | **500** | **2,000** |
| Top-up increment | 100 replies / $5/mo | 100 replies / $5/mo | 100 replies / $5/mo |
