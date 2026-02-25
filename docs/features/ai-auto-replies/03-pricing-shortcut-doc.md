# AI Auto Replies — Pricing & Limits Strategy (v2)

## Feature Summary

AI Auto Replies is a rule-based automation layer on top of ContentStudio's Social Inbox. Users define rules with keyword triggers, intent instructions, account targeting, and a reply type (AI-generated or hardcoded manual). When a message arrives, the AI classifies whether the rule's intent truly applies, then drafts or sends a reply automatically. After saving a rule, AI reviews it and suggests improvements to triggers and intent instructions.

This is a **paid addon** — users must purchase the addon to unlock the feature. Usage is metered on **AI Replies** (not tokens or credits).

---

## What Changed from the Previous Draft

The v1 draft used a granular credit system (1 credit for classification, 5 credits for generation, 1 credit for rule review). This was scrapped in favour of how the industry actually prices it:

- **Before:** 3 separate credit types, variable deductions per operation, hard to explain to users
- **After:** 1 AI Reply = 1 deduction, flat and predictable, consistent with how Tidio, Intercom, Freshdesk, and others price it

---

## What Counts as 1 AI Reply

| Action | Counts? |
|--------|---------|
| AI **drafts** a reply (Review & Send mode — you approve before it goes out) | ✅ Yes — 1 AI Reply |
| AI **sends** a reply automatically (Auto-Send mode) | ✅ Yes — 1 AI Reply |
| AI intent classification (deciding if a rule should fire) | ❌ Free |
| AI rule review & improvement suggestions (on save/edit) | ❌ Free |
| Manual/hardcoded reply rules (no AI involved) | ❌ Free |
| Rule management (create, edit, delete, toggle) | ❌ Free |
| Viewing auto-reply logs | ❌ Free |

**Rule of thumb:** If AI put words in a reply box — paid. Everything else — free.

---

## Competitor Benchmarks

### Social Media Management Tools (AI inbox included in plan)

| Tool | Plans | AI Inbox / Auto-Reply | AI Pricing Model |
|------|-------|----------------------|-----------------|
| **Hootsuite** | From $99/mo | AI Smart Replies (Advanced/Enterprise only) — AI suggests contextual replies for agents to approve or send. Full AI chatbot on Enterprise. | Bundled into plan — no per-reply charge. AI features unavailable without upgrading to expensive tiers. |
| **Sprout Social** | From $249/seat/mo | AI-assisted inbox suggestions, bot builder (Advanced plan). Automated chatbot for DMs. | Bundled into plan — no per-reply charge. Extremely high seat cost makes it enterprise-only. |
| **Agorapulse** | From $79/user/mo | AI Reply Suggestions (April 2025 launch) — suggests replies based on past conversations. Automated inbox assistant on Advanced. | Bundled into plan — no per-reply charge. Feature availability varies by plan tier. |

### Dedicated AI Chat / Support Tools

| Tool | Plan | AI Auto-Reply Access | Price | Usage Limit |
|------|------|---------------------|-------|-------------|
| **Tidio Lyro** | Lyro AI Agent | AI conversation agent for website + social chat | $39–$149/mo | 50–1,000 conversations/mo (a conversation = full session with multiple back-and-forth messages) |
| **Intercom Fin** | Any seat plan + Fin | Autonomous AI agent that resolves support tickets | $0.99 per resolution + $29–132/seat/mo | Pay-per-use, no monthly cap |
| **Freshdesk Freddy AI** | Growth+ plans | AI bot that handles sessions; hands off to human agents | $100 per 1,000 sessions | 500 free sessions; session = unique convo in 24-hr window |
| **ManyChat** | Pro + AI addon | AI intention recognition, AI reply steps in flows | $15/mo (Pro, contact-based) + $29/mo AI addon | Unlimited AI steps on Pro+AI — but contact-based cap on free tier |
| **Crisp** | Essentials ($95/mo) / Plus ($295/mo) | AI-powered resolutions built into platform | Flat plan fee | 50 AI uses/mo on Essentials; **unlimited AI resolutions** on Plus |

### Key Takeaways

- **SMM tools** (Hootsuite, Sprout, Agorapulse) bundle AI into expensive plans — no per-reply pricing, but the floor price is prohibitive ($79–$249+/user/mo)
- **Dedicated AI tools** (Tidio, Intercom, Freshdesk) price on **conversations/sessions/resolutions** — not raw token or API usage
- **ManyChat** is affordable but contact-based, not reply-based — different use case (chatbot flows)
- **Crisp Plus** at $295/mo for unlimited AI resolutions is the only "unlimited" flat model at accessible pricing — but it's a full customer support platform, not an SMM tool
- **No SMMT competitor** offers a clean, affordable AI auto-reply addon with per-plan allotments — this is a real gap ContentStudio can fill at an accessible price point with a transparent "AI replies sent" metric

---

## Addon Name

**AI Auto Replies**

Consistent with ContentStudio's existing naming pattern (AI Text Credits, AI Image Credits, AI Video Credits). Self-explanatory, maps cleanly to marketing copy.

---

## Two-Layer Access Model

Both layers must be active for AI features to work:

```
Layer 1: Addon purchased (workspace-level unlock)
    ↓
Layer 2: AI Reply limit available (consumed per AI-drafted/sent reply)
```

If **Layer 1 is missing** → entire Auto Replies feature is locked (no rules can be created).
If **Layer 2 is exhausted** → addon stays unlocked, but AI reply rules pause (manual rules continue).

---

## Addon Pricing

|  | Monthly | Annual |
|--|---------|--------|
| **AI Auto Replies addon** | **$19/workspace/mo** | **$15/workspace/mo** |

- Available on **Advanced and Agency plans only** (Standard has no Inbox)
- Per-workspace purchase — agencies with multiple workspaces pay per workspace
- Unlocks: rule creation, AI intent classification, AI reply drafting/sending, AI rule review & suggestions

---

## AI Reply Limits by Plan

Limits are per workspace, per billing month. Resets on billing date.

| Plan | Included AI Replies/mo | Capacity Illustration |
|------|----------------------|-----------------------|
| **Advanced** ($49/mo) | **300** | ~300 AI-drafted or auto-sent replies/mo across all accounts |
| **Agency** ($99/mo+) | **750** | ~750 replies across workspaces |
| **Agency Unlimited** | **2,500 base** | + per-account bonus (see below) |

### Agency Unlimited — Per Social Account Bonus

Agency Unlimited users get additional AI replies based on the number of social accounts actively connected to their workspace:

**+10 AI Replies per connected social account per month**

| Connected Accounts | Base | Bonus | Total AI Replies/mo |
|--------------------|------|-------|---------------------|
| 50 accounts | 2,500 | +500 | **3,000** |
| 100 accounts | 2,500 | +1,000 | **3,500** |
| 200 accounts | 2,500 | +2,000 | **4,500** |
| 500 accounts | 2,500 | +5,000 | **7,500** |

This rewards agencies that run ContentStudio at scale and directly ties AI usage allotment to the scope of their operation. It mirrors ContentStudio's existing "per social account" pricing model for other addons.

---

## Buying More AI Replies

Top-up packs are workspace-scoped and auto-renew monthly:

| Pack | Monthly Price | Annual Price | Per-Reply Cost |
|------|-------------|--------------|---------------|
| **250 AI Replies** | $7/mo | $6/mo | ~$0.028/reply |
| **750 AI Replies** | $17/mo | $14/mo | ~$0.023/reply (~18% savings) |
| **2,500 AI Replies** | $45/mo | $36/mo | ~$0.018/reply (~36% savings) |

- Multiple packs can be stacked on a single workspace
- Packs renew alongside the main subscription
- Packs deduct after the plan's included replies are exhausted

---

## Profitability Check

Estimated AI generation cost per reply (using Claude Haiku or equivalent): **~$0.002–$0.005/reply** (intent classification + reply generation, combined).

| Scenario | Revenue | AI Cost | Gross Margin |
|----------|---------|---------|-------------|
| Addon only, Advanced (300 replies used) | $19 | ~$1.00 | ~$18 (95%) |
| Addon only, Agency (750 replies used) | $19 | ~$2.50 | ~$16.50 (87%) |
| Addon + 750-reply pack (1,500 replies used) | $36 | ~$5.00 | ~$31 (86%) |
| Agency Unlimited, 200 accounts (4,500 replies used) | $19 | ~$15 | ~$4 (21%) |
| Agency Unlimited + 2,500-reply pack (7,000 replies used) | $64 | ~$23 | ~$41 (64%) |

**Notes:**
- Agency Unlimited with heavy AI usage and no top-ups is the one edge case with thin margin — solved by the per-account bonus design (heavy users tend to buy top-ups)
- If 30%+ of Agency Unlimited users exhaust their limit and buy even one top-up pack, overall blended margin is strong
- Review & suggestions being free has near-zero cost impact (one lightweight API call per rule save)

---

## Exhaustion Behavior

When AI Replies hit 0 mid-month:

1. **AI rules pause** — keyword triggers still match, but AI classification is skipped; the rule does not fire (prevents unreviewed replies going out)
2. **Manual rules continue** — hardcoded reply rules are unaffected
3. **In-app banner** — shown in Inbox and Auto-Replies settings: *"You've used all your AI Replies for this month. [Buy more] or wait for reset on [date]."*
4. **Email notifications** — sent to workspace owner at 80% usage and again at 0%
5. **No automatic top-up** — user must manually purchase or wait for monthly reset

### What Stays Functional at 0 AI Replies

- Rule management (create, edit, delete, enable/disable) — no AI replies consumed
- Viewing auto-reply logs and history
- Keyword matching logic (evaluated, just not acted on)
- Manual hardcoded reply rules (reply sends; no AI involved)
- AI rule review & suggestions when saving a rule (free operation)

---

## Billing & Reset Rules

- Limits reset on the **workspace billing date** each month — no rollover
- Top-up packs renew monthly alongside the main subscription
- Downgrading plan: limit caps to new plan's allotment at next reset
- Cancelling addon: active AI rules pause immediately; rule data retained for 90 days
- **Free trial:** 25 AI Replies on first addon activation (one-time, non-renewable) — enough to test ~25 AI-drafted replies

---

## Rollout Checklist

- [ ] Addon upsell shown only on Advanced and Agency plan settings — not visible to Standard users
- [ ] Agency Unlimited per-account bonus (+10/account) computed from active connected accounts, updated daily
- [ ] Enterprise: custom AI reply ceiling negotiated per contract
- [ ] Top-up packs available from workspace billing page and from the inline "AI Replies exhausted" banner
- [ ] First-time activation: 25 free AI Replies (one-time, non-renewable)
- [ ] Usage counter visible in Inbox sidebar and Auto-Replies settings: *"218 / 300 AI Replies used this month"*

---

## Summary

| | Advanced | Agency | Agency Unlimited (100 accts) |
|-|----------|--------|------------------------------|
| Addon price | $19/mo | $19/mo | $19/mo |
| Included AI Replies/mo | 300 | 750 | 3,500 |
| Est. AI replies/mo at full use | ~300 | ~750 | ~3,500 |
| Top-up (smallest pack) | $7/250 replies | $7/250 replies | $7/250 replies |
