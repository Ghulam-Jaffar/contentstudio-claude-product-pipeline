# Social Listening — Workflow Design

**Feature:** Social Listening
**Pipeline Step:** 2 of 5 — Workflow Design
**Date:** 2026-02-23

---

## 1. Feature Placement

### Navigation
Social Listening is a **top-level module** in the ContentStudio left sidebar, positioned between "Analytics" and "Inbox":

```
[Left Sidebar]
├── Dashboard
├── Publish
├── Planner
├── Analytics
├── Listening          ← NEW
├── Inbox
├── Discovery
└── Settings
```

### Entry Points
1. **Left sidebar "Listening" icon** — primary entry point
2. **Dashboard widget** (V2) — a "Listening Digest" card on the main dashboard showing latest mention volume + sentiment at a glance
3. **Alert notification** — clicking a spike alert notification takes the user directly to the relevant topic's Mentions feed
4. **Inbox conversation** — if a mention is responded to via Inbox, a "Source: Listening" tag links back to the listening topic (V2)

### URL Structure
```
/[workspace-slug]/listening                        → Topic list / empty state
/[workspace-slug]/listening/[topic-id]/overview    → Topic dashboard — Overview tab
/[workspace-slug]/listening/[topic-id]/mentions    → Topic dashboard — Mentions Feed tab
/[workspace-slug]/listening/[topic-id]/insights    → Topic dashboard — Insights tab (V1.5)
/[workspace-slug]/listening/[topic-id]/settings    → Topic dashboard — Settings tab
```

---

## 2. User Flow — Happy Path

### Flow A: First-Time Setup (Creating a Listening Topic)

1. User clicks **"Listening"** in the sidebar
2. Empty state is shown: "You're not monitoring anything yet. Create your first topic to start tracking mentions." → **"Create Topic"** button
3. User clicks "Create Topic" → a **3-step wizard modal** opens

**Wizard Step 1 — Keywords**
- Field: "Topic name" (e.g., "My Brand", "Competitor: Buffer") — required
- Field: "Primary keyword" — the main brand name, product name, or keyword to track — required
- Field: "Additional keywords" (comma-separated) — optional related terms, misspellings, campaign hashtags
- Field: "Exclude keywords" — words that, if present, should filter out the mention (e.g., exclude "buffer zone" when tracking "Buffer")
- **AI Assist button**: "Describe in plain English what you want to monitor" → AI generates the keyword list

**Wizard Step 2 — Sources**
- Platform toggles (default: all ON): X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Web/News/Blogs
- Language filter (default: All Languages; searchable dropdown to restrict to specific languages)

**Wizard Step 3 — Alerts**
- Toggle: "Notify me when mentions spike" (default: ON)
- Threshold selector: "Alert me when mentions are [25% / 50% / 100% / custom] above my 7-day average"
- Delivery: Email + in-app (both enabled by default; user can toggle either off)

4. User clicks **"Start Listening"**
5. Topic is created and user lands on the **topic Overview dashboard**
6. A banner shows: "We're pulling in your mentions now. Initial data will appear within a few minutes."
7. Within minutes, the first mentions begin populating the feed and charts update in near-real-time

---

### Flow B: Returning User — Reviewing Mentions

1. User clicks "Listening" in sidebar → sees the **Topic List** (all active listening topics for this workspace)
2. Each topic card shows: topic name, mention count (last 7 days), sentiment breakdown (mini donut), last mention timestamp, alert status
3. User clicks a topic → lands on **Overview tab**
4. User scans the **volume chart** and **sentiment charts** for anomalies or trends
5. User clicks the **"Mentions" tab** to see the raw feed
6. User filters by **"Negative" sentiment** to review complaints
7. User sees a concerning negative mention — clicks **"Reply"** on the mention card
8. ContentStudio's Inbox composer opens with the mention pre-loaded — user types a response and sends
9. The mention is marked as "Replied" in the listening feed

---

### Flow C: Responding to a Spike Alert

1. User receives an **email alert**: "⚠️ Spike Alert — 'ContentStudio' has 180% more mentions than your 7-day average. Here are 3 samples: [...]"
2. User clicks "View Mentions" link in the email
3. Browser opens to the topic's **Mentions feed**, auto-filtered to the last 24 hours
4. User scans the feed, identifies the cause (e.g., a viral tweet mentioning their product)
5. User responds to key mentions via the **Reply** flow (Flow B, step 6–9)
6. User switches to **Overview tab** to monitor whether the spike is subsiding

---

### Flow D: Competitor Tracking

1. User creates a **new topic** with the competitor's brand name as the primary keyword (e.g., Topic name: "Competitor: Buffer", keyword: "Buffer")
2. The competitor topic populates identically to a brand topic
3. User navigates to their own brand topic → clicks **"Compare"** button (or a "Competitors" tab)
4. **Side-by-side view**: own brand mention volume + sentiment vs. competitor — Share of Voice pie chart at the top
5. User can add up to 3 competitor topics to the comparison (plan-dependent limit)

---

## 3. Alternative Flows / Edge Cases

### Empty States
- **No topics created yet:** "You're not monitoring anything yet. Create your first topic." + "Create Topic" CTA
- **Topic just created, no data yet:** "We're collecting your first mentions. Check back in a few minutes." + animated loading indicator
- **Topic has zero mentions in selected date range:** "No mentions found for this period. Try expanding the date range or checking your keywords."
- **Platform token expired / disconnected:** Banner: "Your [Platform] connection expired. Reconnect to resume monitoring." + "Reconnect" link to Settings → Integrations

### Error States
- **Topic creation fails:** "Something went wrong. Your topic wasn't saved. Please try again." — topic wizard stays open
- **Alert delivery fails:** Logged silently; retry queued; user is not shown an error for delivery failures (they'll see mentions when they log in)
- **API rate limit hit for a platform:** Mention collection pauses for that platform; banner shown on topic dashboard: "X/Twitter data may be delayed due to API rate limits. We'll resume shortly."
- **Too many topics for current plan:** "You've reached your topic limit ([N] topics on your current plan). Upgrade to track more brands." — "Upgrade" CTA

### Keyword Conflicts / Noise
- If a keyword generates more than [threshold] mentions per day, a warning is shown: "This keyword is very broad and may generate a high volume of mentions. Consider adding exclusion keywords to narrow your results." — inline suggestion with common exclusions
- Users can edit exclusion keywords at any time from the Settings tab

---

## 4. Key Design Decisions

### Decision 1: Separate Module vs. Embedded in Discovery

**Option A — Separate top-level "Listening" module** *(Recommended)*
Keep Social Listening as its own sidebar item, separate from the existing Discovery module.

**Option B — Extend the existing Discovery module**
Add listening functionality as a new tab inside Discovery.

**Recommendation: Option A.** Social Listening is strategically distinct from Discovery (which is about finding curated content to reshare, not about monitoring brand mentions). Bundling them would create confusion and undermine the "Listening" narrative in marketing. Separate nav position signals that this is a first-class feature, not a Discovery sub-feature. Long-term, it becomes one of ContentStudio's core product pillars alongside Publish, Analytics, and Inbox.

---

### Decision 2: Topic Limit Per Plan

**Option A — 3 / 10 / Unlimited** *(Recommended)*
- Starter: 3 topics
- Growth/Pro: 10 topics
- Agency/Scale: Unlimited

**Option B — Per-mention volume cap (not per-topic)**
Unlimited topics but cap total mentions processed/stored per month (e.g., 5,000 mentions/mo on Starter).

**Recommendation: Option A — per-topic limit.** Per-topic limits are simpler to understand and communicate. Per-mention caps create unpredictable costs for agencies and are widely disliked (see: Metricool's $25/day/hashtag approach). Topics are the natural unit users think in ("I want to track 3 brands").

---

### Decision 3: Sentiment Analysis — External API vs. Internal Model

**Option A — External NLP API (e.g., Google Natural Language API, OpenAI)** *(Recommended for V1)*
Fast to ship; high accuracy including sarcasm/emoji; pay per mention processed.

**Option B — Internal ML model**
Full control; no ongoing API costs; requires data science effort to train and maintain.

**Recommendation: Option A for V1.** Speed to market matters. An external API is accurate immediately, requires no training data, and costs a fraction of the engineering effort. Lock in an abstraction layer (`SentimentClassifierService`) so the underlying provider can be swapped later.

---

## 5. Integration with Existing ContentStudio Features

### Inbox Integration
- **Reply from Listening:** Clicking "Reply" on any mention in the listening feed opens the Inbox composer with the mention pre-loaded (author handle, original text, platform). The reply is tracked in Inbox as a normal conversation — engagement history, assigned agent, status (open/closed), all the Inbox features apply.
- This is the most important integration: closes the Discover → Respond loop without tool switching.

### Composer / Planner Integration (V2)
- **Create content from trend:** A "Create Post" button on the Overview tab (or on a trending hashtag) opens the Composer with the keyword/hashtag pre-filled in the caption. Converts listening insights directly into content creation.

### Analytics Integration (V2)
- **Earned mention reporting:** Social Listening data (mention volume, sentiment, SOV) surfaces as an "Earned Media" section in the Analytics module, alongside owned channel performance. This gives users a full picture of brand performance: published content (owned) + earned conversation (listening).

### Discovery Integration (None in V1)
- The Discovery module continues to operate independently for content curation. In V2, there may be a "Topics from Listening" feature that auto-suggests Discovery topics based on what's trending in listening data.

### Workspace / Multi-Account Structure
- Topics are workspace-scoped. Each workspace (client for agencies) has its own independent listening topics, mention history, and alert settings.
- Agencies can monitor a different brand per workspace without data leaking between clients.

---

## 6. Scope Recommendation

### V1 (Launch)
Everything a user needs to justify paying for the feature and see immediate value:
- Topic/Query Builder with guided 3-step wizard + AI keyword assist
- Sentiment classification (Positive / Neutral / Negative) via external NLP API
- Mention Feed with filters: platform, sentiment, date range, source type (social vs. web)
- Overview dashboard: mention volume chart, sentiment donut + trend chart, platform breakdown
- Spike/volume alerts (email + in-app, configurable threshold)
- Basic competitor comparison: up to 3 competitor topics tracked, side-by-side SOV chart
- CSV export of mention data; PDF summary report export
- Reply from Listening → Inbox integration
- Platform coverage: X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Web/News/Blogs

### V1.5 (Next Quarter)
Features that significantly increase retention and word-of-mouth:
- AI Trend Summary — weekly narrative digest ("340 mentions this week, 72% positive…")
- Top Voices / Influencer Panel — top 5 accounts by reach driving the conversation
- Saved mention collections (bookmarking for reporting or sharing)

### V2 (Future Roadmap)
Strategic differentiators for the agency/enterprise tier:
- Share of Voice as a formal metric with historical tracking (not just current-period snapshot)
- Create Post from Listening → Composer integration
- Earned Media analytics tab in Analytics module
- Emotion clustering beyond basic sentiment (frustrated, excited, confused)
- Image/logo mention detection (long-term, technically complex)
- Historical data import for accounts with existing listening context
