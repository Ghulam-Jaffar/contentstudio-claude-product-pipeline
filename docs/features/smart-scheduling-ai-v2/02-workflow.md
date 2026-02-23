# Workflow: Smart Scheduling via AI V2

**Feature:** Smart Scheduling via AI V2 — Guided Conversational Workflow
**Epic:** [Smart scheduling guided workflow](https://app.shortcut.com/contentstudio-team/epic/109981)

---

## Overview

Smart Scheduling via AI V2 transforms the AI Studio scheduling experience from a prompt-triggered black box into a guided, step-by-step conversation. Instead of silently executing a command and hoping for the best, the AI now clarifies intent first, generates content progressively, and only acts after explicit user confirmation.

Users start by choosing one of **four entry paths** that match how they actually work — whether they're starting from zero, have content ready, or need to schedule in bulk. The AI then walks them through a structured flow: generate → review → select accounts → set time → confirm.

**All generated posts default to Draft. Nothing goes live without the user clicking "Confirm."**

---

## The Four Entry Paths

When a user opens Smart Scheduling, they see a choice of four starting points:

| Path | Who it's for | Starting trigger |
|---|---|---|
| **Start from Scratch** | User has a topic but no content yet | "I want to create posts from scratch" |
| **I Have Captions** | User has written copy and wants to schedule it | "I have captions ready" |
| **I Have Media** | User has photos/videos and needs captions written | "I want to create posts from my media" |
| **Plan in Bulk** | User needs 10+ posts across a date range | "I want to bulk plan posts" |

Users can also skip the cards and type freely — the AI detects scheduling intent from natural language.

---

## Path A — Start from Scratch

> User has a topic in mind but no existing content.

### Initial AI Response
> *"Great, let's build your posts from scratch. A few quick questions first:*
> *What type of content do you want? Caption only, caption with image, or caption with video?*"

### Workflow

```
User selects path: "Start from Scratch"
        │
        ▼
AI asks: What type of content?
  → Caption only
  → Caption + Image
  → Caption + Video
        │
        ▼
AI asks: How many posts? And what's the topic or keywords?
        │
        ▼
AI confirms plan:
  "I'll generate 7 caption-only posts about [topic]. Starting now..."
        │
        ▼
AI streams posts progressively — each post appears as a card:
  ┌─────────────────────────────────────┐
  │ #1  [Draft]                         │
  │ Caption text here...                │
  │ Accounts: not selected yet          │
  │ Scheduled: Draft                    │
  │ [Edit] [Remove] [Override time]     │
  └─────────────────────────────────────┘
        │
        ▼
AI asks: "Here are your 7 posts. Ready to schedule, save as drafts, or edit one?"
```

### Options at this point
- **"Schedule these"** → moves to Account Selection
- **"Save as drafts"** → posts saved to Draft, conversation ends
- **"Edit #3"** → opens post 3 in the Composer
- **"Remove #5"** → removes that post from the list
- **"Start over"** → resets the flow entirely

---

## Path B — I Have Captions

> User has already written their copy and wants to schedule it.

### Initial AI Response
> *"Perfect — paste your captions below. You can paste one or multiple. I'll take it from there."*

### Workflow

```
User selects path: "I Have Captions"
        │
        ▼
User pastes captions into chat
        │
        ▼
AI detects: single caption or multiple?
        │
        ├─ Single caption
        │       │
        │       ▼
        │  AI asks: "Schedule as-is, or add images/video first?"
        │    → "As-is" → moves to Account Selection
        │    → "Add images" → prompts media upload
        │    → "Add video" → prompts media upload
        │
        └─ Multiple captions
                │
                ▼
           AI creates draft post cards (one per caption)
                │
                ▼
           AI asks: "I've created [N] draft posts from your captions.
                     Want to add images to any of them, or schedule now?"
                  → "Schedule now" → Account Selection
                  → "Add images to #2" → prompts media for that post
                  → "Edit #1" → opens in Composer
```

### Options at this point
- Add media to specific posts before scheduling
- Schedule all at once
- Save as drafts

---

## Path C — I Have Media

> User has photos or videos and wants captions generated for them.

### Initial AI Response
> *"Nice — upload your media below and I'll write the captions for you."*

### Workflow

```
User selects path: "I Have Media"
        │
        ▼
User uploads image(s) or video(s) via media upload
        │
        ▼
AI generates caption(s) for the uploaded media
        │
        ▼
AI shows post cards (one per media item):
  ┌─────────────────────────────────────┐
  │ #1  [Draft]                         │
  │ AI-generated caption...             │
  │ 📷 [uploaded image thumbnail]       │
  │ Accounts: not selected yet          │
  │ [Edit] [Remove] [Override time]     │
  └─────────────────────────────────────┘
        │
        ▼
AI asks: "Here are your posts with captions. Want to schedule these?"
```

### Options at this point
- **"Schedule these"** → moves to Account Selection
- **"Regenerate caption for #2"** → AI rewrites that caption
- **"Edit #1"** → opens in Composer
- **"Save as drafts"** → saves without scheduling

---

## Path D — Plan in Bulk

> User needs a large batch of posts across a date range.

### Initial AI Response
> *"Let's plan in bulk. What are you creating — bulk captions, bulk image posts, or bulk video posts?"*

### Workflow

```
User selects path: "Plan in Bulk"
        │
        ▼
AI asks: Bulk captions, bulk image posts, or bulk video posts?
        │
        ├─ Bulk captions
        │       │
        │       ▼
        │  AI asks: How many? What topic? What date range?
        │       │
        │       ▼
        │  AI generates all posts in batch, streams cards progressively
        │       │
        │       ▼
        │  Moves to Account Selection →
        │
        ├─ Bulk image posts
        │       │
        │       ▼
        │  AI says: "For bulk image posts, the Bulk Composer
        │            is the best tool for this."
        │  CTA: [Open Bulk Composer →]
        │       │
        │       ▼
        │  User completes bulk upload in Bulk Composer
        │       │
        │       ▼
        │  AI resumes scheduling when user returns
        │
        └─ Bulk video posts
                │
                ▼
           Same as bulk image: routes to Bulk Composer
```

### Options at this point (bulk captions)
- Adjust post count or date range before generation
- Edit individual posts after generation
- Schedule all, or save as drafts

---

## Shared Final Flow (All Paths)

Once the user has confirmed they want to schedule, all four paths converge into the same three-step closing flow.

### Step 1 — Account Selection

```
AI triggers account selection block in the chat:

┌──────────────────────────────────────────────────────┐
│  Choose accounts to publish to                       │
│  Type a number (e.g. "1, 3") or account name        │
│                                                      │
│  #  Platform  Account Name       Username    Status  │
│  1  IG        Brand Instagram    @brand_ig   Active  │
│  2  FB        Brand Facebook     Brand Page  Active  │
│  3  LI        Company LinkedIn   Company     Active  │
│  4  TW        Brand Twitter      @brand_tw   Active  │
└──────────────────────────────────────────────────────┘

User types: "1, 3"  OR  "brand instagram"
```

AI confirms: *"Got it — posting to Brand Instagram and Company LinkedIn. Apply to all [N] posts?"*

**Options:**
- **"Yes, all posts"** → same accounts applied to every post
- **"Different for each"** → AI asks account selection per post
- **"Back"** → returns to post review

---

### Step 2 — Time Selection

```
AI asks: "When do you want to schedule these?
          You can give me specific times, a date range,
          or I can distribute them automatically."
```

**Options:**
- **Specific times** → user provides dates/times per post (or one time for all)
- **Date range** → AI distributes evenly at 10:00 AM workspace time
- **"Schedule now"** → posts go out immediately

---

### Step 3 — Review & Confirm

```
AI shows the Review Panel:

┌──────────────────────────────────────────────────────┐
│  Ready to schedule?                                  │
│  Review your posts below. Once you confirm,          │
│  they'll be added to your queue.                     │
│                                                      │
│  🏢 Workspace: Brand Co.  👤 Accounts: 2  📄 7 posts │
│                                                      │
│  #  Caption Preview          Accounts  Time    Status│
│  1  "Here's how we help..."  IG, LI    Feb 25  Sched │
│  2  "Behind the scenes..."   IG, LI    Feb 26  Sched │
│  3  "Our team this week..."  IG, LI    Feb 27  Sched │
│  ...                                                 │
│                                                      │
│  [    Schedule All Posts    ]                        │
│  [       Make Changes       ]                        │
└──────────────────────────────────────────────────────┘
```

**On "Schedule All Posts":**

Confirmation dialog appears:
> *"Schedule 7 posts? This will add 7 scheduled posts to your Planner. You can still edit or delete them there."*
> — **Yes, Schedule All** / Cancel

**On confirmation:**
> *"Done! 7 posts have been scheduled and added to your Planner. ✓*
> *[View in Planner →]*"

**On "Make Changes":**
> Review panel collapses → chat returns to active conversation for adjustments

---

## Escape Hatches (Any Point in the Flow)

The AI never locks the user into a workflow state. At any point:

| User says... | What happens |
|---|---|
| "Start over" | Full flow resets, entry cards reappear |
| "Cancel" | Posts remain as drafts, scheduling cancelled |
| "Go back" | Returns to the previous step |
| Unrelated question | AI pauses the scheduling flow and answers, then offers to resume |
| Off-topic message | AI responds and asks: "Want to continue scheduling?" |

---

## Edge Cases

| Scenario | Behavior |
|---|---|
| No social accounts connected | Account Selection step shows "No accounts connected" + Connect CTA |
| User provides topic but count = 0 | AI re-prompts: "How many posts would you like to create?" |
| Empty topic input | AI re-prompts: "What topic or keywords should I write about?" |
| Generation fails for one post | Error card shown for that slot with "Try Again" option |
| Scheduling partially fails | Success/failure breakdown shown; retry option for failed posts |
| User returns mid-flow | AI resumes from the last saved workflow step (24h session TTL) |
