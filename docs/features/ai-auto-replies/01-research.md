# AI Auto Replies — Research

**Epic:** https://app.shortcut.com/contentstudio-team/epic/94174
**Date:** 2026-02-25

---

## Feature Summary

Rule-based AI automation on top of ContentStudio's Social Inbox. Users define rules with keyword triggers, intent instructions, account targeting, and reply type (AI-generated or manual hardcoded). On incoming message: keyword match → AI classifies intent → AI drafts or sends reply. AI also reviews rules on save and suggests improvement suggestions.

### Supported Networks

| Inbox Type | Networks |
|-----------|---------|
| Post Comments | Facebook, Instagram, LinkedIn, YouTube |
| Private Chats (DMs) | Facebook, Instagram |
| Reviews | Google Business Profile (GMB) |

---

## Key Shortcut Documents (Epic 94174)

| Doc ID | Title | Author |
|--------|-------|--------|
| `698aba64` | PRD: Inbox Auto-Replies | Azhar (Feb 2026) |
| `695e5fa5` | Inbox Auto-Replies — Functional Workflow | Azhar |
| `698ed2db` | AI Auto Replies — Create Rule Modal (Final UX Spec) | Azhar |
| `698ed494` | AI Auto Replies — Intent Filter Prompts & Test Cases | Azhar |
| `698ed757` | AI Auto Replies — Pre-Save Validation Prompt & UX | Azhar |
| `688c6988` | Reply with AI — Technical Documentation | — |
| `699b5dd6` | Pricing & Limits Strategy v2 | — |

---

## Competitor Research (2025–2026)

### Social Media Management Tools — AI bundled into plans

| Tool | Plans | AI Inbox / Auto-Reply Feature | Pricing Model |
|------|-------|------------------------------|--------------|
| **Hootsuite** | From $99/mo | AI Smart Replies (Advanced/Enterprise only) — AI suggests contextual replies for agents to approve or send. Full AI chatbot on Enterprise. | Bundled — no per-reply charge. AI unavailable without expensive tier upgrade. |
| **Sprout Social** | From $249/seat/mo | AI-assisted inbox suggestions, bot builder on Advanced. Automated chatbot for DMs. | Bundled — no per-reply charge. Effectively enterprise-only pricing. |
| **Agorapulse** | From $79/user/mo | AI Reply Suggestions (April 2025). Automated inbox assistant on Advanced. | Bundled — no per-reply charge. Feature availability varies by tier. |

### Dedicated AI Chat / Support Tools

| Tool | Plan | AI Auto-Reply | Price | Usage Limit |
|------|------|--------------|-------|-------------|
| **Tidio Lyro** | Lyro AI Agent | AI conversation agent for website + social chat | $39–$149/mo | 50–1,000 conversations/mo. A conversation = full multi-message session (many back-and-forths = 1 conversation). |
| **Intercom Fin** | Any seat plan + Fin | Autonomous AI agent resolves support tickets end-to-end | $0.99/resolution + $29–132/seat/mo | Pure pay-per-use; no monthly cap. Very expensive at scale. |
| **Freshdesk Freddy AI** | Growth+ plans | AI bot handles sessions, hands off to human agents | $100 per 1,000 sessions | 500 free sessions included. Session = unique conversation in 24-hr window. |
| **ManyChat** | Pro + AI addon | AI intention recognition, AI reply steps in chatbot flows | $15/mo Pro (contact-based) + $29/mo AI addon | Unlimited AI steps on Pro+AI. Contact-based cap on free tier. |
| **Crisp** | Essentials / Plus | AI resolutions built into platform | €95/mo (Essentials) / €295/mo (Plus) | 50 AI uses/mo on Essentials; unlimited AI resolutions on Plus. |

### Key Takeaways

- SMM tools (Hootsuite, Sprout, Agorapulse) bundle AI into plans that cost $79–$249+/user/mo — AI is inaccessible without a large plan investment
- Dedicated AI tools (Tidio, Intercom, Freshdesk) price on **conversations / sessions / resolutions** — not tokens or raw API usage
- ManyChat is contact-based, not reply-based — different use case (chatbot flows vs. inbox auto-reply)
- Crisp Plus at €295/mo for unlimited AI resolutions is the only flat "unlimited" model at accessible pricing — but it's a full support platform, not an SMM tool
- **Gap:** No SMMT competitor offers an affordable AI auto-reply addon with transparent "AI replies sent" pricing — ContentStudio can own this position at $19/mo
