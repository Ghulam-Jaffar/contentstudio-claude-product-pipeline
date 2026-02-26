# AI Auto Replies — Pricing Strategy (v3)

**Shortcut Doc:** https://app.shortcut.com/contentstudio-team/write/IkRvYyI6I3V1aWQgIjY5OWI1ZGQ2LWQ0YjItNGQwNS1hODA1LTg0Njk2ZDI2MDUyYSI=
**Doc ID:** `699b5dd6-d4b2-4d05-a805-84696d26052a`
**Last Updated:** 2026-02-26

---

## Key Decisions

| Decision | Value |
|----------|-------|
| Feature name | AI Auto Replies |
| Top-up unit name | AI Inbox Replies (not "credits" — concrete and manageable) |
| Eligible plans | Advanced, Agency Unlimited (Standard has no Inbox) |
| Unlock fee | $19/workspace/mo (monthly) / $15/workspace/mo (annual) |
| Core formula | **1 social account = 20 AI Inbox Replies/month** |
| Top-up increment | 100 AI Inbox Replies = $60/year ($5/mo) |
| Free trial | 50 AI Inbox Replies on first activation (one-time) |

---

## The Formula

> **1 social account = 20 AI Inbox Replies/month**

One formula drives everything: base allotments, per-account scaling for Agency Unlimited, and top-up sizing. No separate logic per plan.

---

## Plans & Base Allotments

| Plan | Price | Social Accounts (base) | AI Inbox Replies/mo |
|------|-------|------------------------|---------------------|
| Standard | $19/mo | 5 | — (no Inbox) |
| Advanced | $49/mo | 10 | **200** |
| Agency Unlimited | $99/mo+ | 25 | **500** |

---

## Agency Unlimited Per-Account Scaling

Each connected social account adds to the AI Inbox Replies allotment at the same formula rate:

**+20 AI Inbox Replies per connected social account per month**

| Total Accounts | AI Inbox Replies/mo |
|---------------|---------------------|
| 25 (base) | 500 |
| 50 | 1,000 |
| 100 | 2,000 |
| 200 | 4,000 |
| 500 | 10,000 |

Recalculates on account add/remove, effective next billing cycle.

---

## Top-Up Increment

**100 AI Inbox Replies = $60/year ($5/mo)**

Follows ContentStudio's existing addon increment pattern ($60/year per increment). Multiple increments stack. Deduct after base allotment is exhausted.

---

## What Counts / What's Free

| Action | Counts as 1 AI Inbox Reply? |
|--------|----------------------------|
| AI drafts a reply (Review & Send mode) | ✅ Yes |
| AI sends a reply automatically (Auto-Send mode) | ✅ Yes |
| AI intent classification | ❌ Free |
| AI rule review & suggestions | ❌ Free |
| Manual/hardcoded reply rules | ❌ Free |
| Rule management, log viewing | ❌ Free |

---

## Access Model

```
Plan (Advanced / Agency Unlimited)
    → Unlock AI Auto Replies ($19/mo)
        → Base AI Inbox Replies allotment (social accounts × 20)
            → Top up if needed (100 replies / $5/mo)
```

---

## Profitability

AI cost per reply (Haiku-level): ~$0.002–$0.005

| Scenario | Revenue | AI Cost | Margin |
|----------|---------|---------|--------|
| Advanced, 200 replies used | $19/mo | ~$0.60 | ~97% |
| Agency Unlimited base, 500 replies used | $19/mo | ~$1.50 | ~92% |
| Agency Unlimited, 100 accounts, 2,000 replies | $19/mo | ~$6.00 | ~68% |
| Advanced + 1 top-up (300 replies) | $24/mo | ~$0.90 | ~96% |
| Agency Unlimited 100 accts + 5 top-ups (2,500 replies) | $44/mo | ~$7.50 | ~83% |

---

## Version History

- **v1:** Token/credit-based model (1 credit = classification, 5 credits = generation, 1 = review). Scrapped — too complex.
- **v2:** Flat 1 AI reply = 1 deduction. Correct direction but wrong plan names and arbitrary allotments.
- **v3:** Formula-driven. One rate (20/account) drives all allotments uniformly. Plan names corrected (Standard / Advanced / Agency Unlimited). Top-up aligned to existing $60/year addon pricing.
