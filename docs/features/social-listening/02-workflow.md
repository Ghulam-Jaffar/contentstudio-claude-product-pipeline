# Social Listening — Workflow Design

**Feature:** Social Listening
**Pipeline Step:** 2 of 5 — Workflow Design
**Date:** 2026-02-23
**Last Updated:** 2026-03-02

---

## 1. Feature Placement

### Navigation
Social Listening is a **top-level module** in the ContentStudio left sidebar, positioned between "Analytics" and "Inbox":

```
[Topbar]
├── Dashboard
├── Publish
├── Planner
├── Analytics
├── Listening          ← NEW
├── Inbox
├── Discovery
└── Settings
```

Visible to **all users** regardless of subscription state. Clicking it always works — non-subscribers land on the marketing/upsell page.

### Entry Points
1. **Left sidebar "Listening" icon** — primary entry point for all users
2. **Spike alert email** — clicking "View Mentions" deep-links to the topic's Mentions feed filtered to last 24 hours
3. **In-app spike notification** — clicking the notification navigates to the topic dashboard
4. **Demo topic** — non-subscribers can click "Open Demo Topic" on the landing page to explore a live sample

### URL Structure (Prototype / Production)
```
/features/listening                                  → Landing page (trial/locked/expired-no-topics) OR Topic list (unlocked/expired-with-topics)
/features/listening/demo                             → Server redirect → /features/listening/topic-demo/analytics
/features/listening/new                              → Create Topic Wizard (step 1)
/features/listening/new?step=2                       → Wizard step 2 — Keywords
/features/listening/new?step=3                       → Wizard step 3 — Sources
/features/listening/new?step=4                       → Wizard step 4 — Alerts
/features/listening/new/preview                      → Preview results (non-subscriber post-setup)
/features/listening/[topicId]                        → Redirect → /[topicId]/analytics
/features/listening/[topicId]/analytics              → Analytics tab (Performance + Insights sub-tabs)
/features/listening/[topicId]/mentions               → Mentions feed tab
/features/listening/[topicId]/compare                → Compare tab
/features/listening/[topicId]/reports                → Reports tab
/features/listening/[topicId]/settings               → Settings tab
```

---

## 2. Subscription States

Four states drive different UI experiences. All routing is handled in the root `/features/listening/page.tsx`.

| State | Who | What They See |
|---|---|---|
| **trial** | On ContentStudio trial, no add-on | Landing page with orange alert banner ("Agency Unlimited required") |
| **locked** | Paid plan, no Social Listening add-on | Landing page (no banner) |
| **unlocked** | Active Social Listening add-on | Topic list + full dashboard |
| **expired** | Had add-on, subscription lapsed | If topics exist: churned topic list. If no topics: landing page |

---

## 3. User Flow — Happy Path

### Flow A: Non-Subscriber Exploring the Feature

1. User clicks "Listening" in sidebar → **Landing page** shown
2. User reads the hero, feature pillars, and pricing ($99/mo or $79/mo annual)
3. User clicks **"Open Demo Topic"** → server-redirects to `/topic-demo/analytics` with sample data
4. Persistent demo banner at top: "You're viewing a demo topic with sample data. Add Social Listening to start monitoring real mentions."
5. User explores Analytics, Mentions, Compare tabs — all functional with mock data
6. User clicks **"Add to My Plan"** → taken to billing/upgrade flow

---

### Flow B: First-Time Setup (Creating a Listening Topic)

1. Subscribed user clicks "Listening" → **Topic list** shown (empty state if no topics yet)
2. Empty state: "You're not listening to anything yet. Create your first topic..." → **"+ Create Your First Topic"** button
3. User clicks the button → navigates to `/features/listening/new` — **4-step wizard**

**Wizard Step 1 — Basic Info**
- Topic Name (required, max 80 chars, unique within workspace)
- Description (optional, for team reference)
- Topic Color (8 presets for visual identification)
- Visibility: Only Me / Everyone / Specific Members (member selector)
- Footer: Back | Next →

**Wizard Step 2 — Keywords**
- **Include Keywords** (required, OR logic) — chip input with type selector: Text / Hashtag / Mention
- **Required Keywords** (optional, AND logic) — mention must contain ALL of these
- **Excluded Keywords** (optional, NOT logic) — excludes any mention with these
- Real-time conflict detection: include+exclude conflict, duplicate warning, short keyword warning
- **AI Assist button** ("Get AI keyword suggestions"): textarea for plain-English description → generates keyword suggestions per group

**Wizard Step 3 — Sources**
- Connected Profiles section (workspace's connected social accounts, toggleable)
- Platform toggles: Social (X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Bluesky, Pinterest, Threads) | Web & Media (News Sites, Blogs, Forums, Review Sites, Podcasts)
- Language MultiSelect (default: all; searchable, 170+ options)
- Switch: Hide reshares/reposts/retweets
- Switch: Only collect verified account mentions

**Wizard Step 4 — Alerts**
- Spike alert toggle (ON by default) + threshold: 25% / 50% / 100% / Custom above 7-day average
- Minimum mentions input (e.g., 5 mentions in 6-hour window) to avoid noise on low-volume brands
- Sentiment drop alert toggle (OFF by default) + drop threshold + time window (24h/48h/72h)
- Notification delivery: member selector, email toggle + recipients, in-app toggle

**Footer CTA:**
- Subscribers: "Start Listening →" → creates topic, navigates to `/[topicId]/analytics`
- Non-subscribers: "Preview My Results →" → navigates to `/new/preview` with sample data

4. Topic is created → user lands on the Analytics tab
5. Banner: "We're pulling in your mentions now. Initial data will appear within a few minutes."

---

### Flow C: Non-Subscriber Post-Wizard Preview

1. Non-subscriber completes the wizard → lands on `/features/listening/new/preview`
2. Banner: "Here's a preview of what '[Topic Name]' will look like once you subscribe. Data shown is representative."
3. Preview shows: KPI summary bar, mention volume chart, sentiment donut, 10 sample mention cards (labeled "Sample mention")
4. Reply buttons disabled with tooltip: "Subscribe to reply to mentions"
5. Upgrade CTA with pricing block below the preview

---

### Flow D: Returning User — Reviewing Mentions

1. User clicks "Listening" → **Topic list** showing all active topics with KPI chips
2. User clicks a topic → lands on **Analytics tab** (Performance sub-tab by default)
3. User reviews the **KPI grid** (7 metrics: Mentions, Reach, Impressions, Engagements, Sentiment Score, Net Reach, Unique Authors)
4. User scans **time-series charts** (Mention Volume, Reach, Engagement, Impressions, Sentiment Trend, Sentiment Donut)
5. User clicks the **AI Insights button** (violet Sparkles icon) on any chart → popover shows 2–3 tailored insights
6. User clicks the **"Mentions" tab**
7. Filters by **"Negative" sentiment** using the filter sidebar
8. Finds a concerning mention → clicks **"Reply"**
9. Inbox composer opens pre-loaded with the mention → user sends response
10. Mention is marked "Replied ✓" in the feed

---

### Flow E: Responding to a Spike Alert

1. User receives spike alert email: "⚠ Spike Alert — 'ContentStudio' has 180% more mentions than your 7-day average. Sample mentions: [...]"
2. User clicks **"View Mentions"** → browser deep-links to `/[topicId]/mentions?filter=last24h`
3. User scans the feed, identifies the cause
4. Replies to key mentions via the Reply flow (Flow D, steps 8–10)
5. Switches to **Analytics tab** → monitors whether the spike is subsiding in the Mention Volume chart

---

### Flow F: Competitor Tracking & Share of Voice

1. User creates a **new topic** using a competitor's brand name (e.g., "Competitor: Buffer")
2. Competitor topic populates identically to a brand topic
3. User navigates to the **Compare tab** on their own brand topic
4. Selects the competitor topic(s) to compare (up to 4 additional topics)
5. **Share of Voice donut** shows % split of total mentions across all selected topics
6. **Side-by-side KPI table**: Mentions, Reach, Engagement, Sentiment Score, Positive %, Unique Authors — starred cell marks highest value per row
7. **Volume Over Time** multi-line chart and **Sentiment Score** trend overlaid per topic

---

### Flow G: Exporting a Report

1. User clicks **"Export"** button in the topic dashboard header (available from any tab)
2. **Export Modal** opens with 3 tabs:
   - **Download** — select report type (Performance & Mentions / Compare Topics / Compare Periods), date range, format (PDF or CSV), sections to include
   - **Schedule** — configure recurring delivery: frequency, day, time, recipients, "skip if no new mentions" toggle
   - **Share** — generate a shareable read-only link with expiry
3. User clicks **"Generate & Download"** → PDF or CSV generated
4. Report appears in **Reports tab → Downloaded Reports sub-tab**

---

### Flow H: Previewing a Generated Report

1. User navigates to **Reports tab** → **Downloaded Reports sub-tab**
2. Sees a list of generated reports with status badges (Generating… / Ready / Failed)
3. Clicks the **eye icon** on a "Ready" report
4. **Report Preview Drawer** opens from the right side (full viewport height)
5. Drawer shows a static, non-interactive analytics snapshot:
   - 7 KPI cards (values only)
   - Mention Volume area chart
   - Sentiment Breakdown donut + Platform Breakdown bar chart (2-col)
   - Geographic Distribution + Top Hashtags bar charts (2-col)
6. All charts use `ChartCard isStatic={true}` — no dropdowns or controls, but AI Insights buttons remain visible
7. User clicks "Download" or closes the drawer

---

### Flow I: Churned User (Expired Add-On)

1. Subscribed user's add-on lapses
2. User clicks "Listening" → **Topic list in churned mode** (if they had existing topics)
3. **UpgradeModal auto-opens** (cannot be dismissed by clicking outside)
4. Orange banner at top: "Your Social Listening add-on is no longer active. Your N topics and data are saved — re-subscribe to access them."
5. All topic cards visible but **dimmed at 50% opacity** — clicking any card re-opens the UpgradeModal
6. User clicks "Re-subscribe" → upgrade/billing flow

---

## 4. Alternative Flows / Edge Cases

### Empty States
| Situation | What user sees |
|---|---|
| No topics created | Ear/sound wave icon + "You're not listening to anything yet." + "Create Your First Topic" button |
| Topic just created, no data | Radar animation + "We're listening..." + expected timing info (minutes for social, up to 1 hour for web) |
| No mentions matching filters | Magnifying glass icon + "No mentions found" + "Clear all filters" button + "Edit topic keywords" link |
| No downloaded reports | FileDown icon + "No downloaded reports yet" + instructions to use the Export button |
| No scheduled reports | Calendar icon + "No scheduled reports" + "Create Schedule" button |
| Competitor topic comparison empty | "Compare topics side-by-side" icon + "Add at least one more topic to compare" |

### Error States
| Situation | What user sees |
|---|---|
| Topic creation fails | Wizard stays open; error toast: "Your topic wasn't saved. Please try again." |
| Platform token expired | Amber banner on dashboard: "Your [Platform] connection expired. [Reconnect →]" |
| X/Twitter API rate limit | Blue info banner: "X/Twitter data may be delayed due to API rate limits. We'll resume automatically." |
| Data older than retention window | Overlay: "🔒 This data is older than your history window. Re-subscribe to access saved data." |
| Topic paused | Gray banner: "⏸ This topic is paused — no new mentions are being collected. [Resume]" |
| Report generation failed | "Failed" badge on report row + "Retry" button |

### Keyword Conflicts
- Same keyword in Include AND Exclude → chip highlighted red in both fields; Next button disabled
- Same keyword in Required AND Exclude → same error
- Keyword under 3 characters → warning badge "⚠ Too short"; does not block progression
- Very broad keyword (high estimated volume) → amber banner suggesting Required Keywords to narrow results

---

## 5. Key Design Decisions

### Decision 1: Separate Module vs. Embedded in Discovery

**Option A — Separate top-level "Listening" module** *(Chosen)*
Keep Social Listening as its own sidebar item, separate from the existing Discovery module.

**Option B — Extend the existing Discovery module**
Add listening as a new tab inside Discovery.

**Recommendation: Option A.** Social Listening is strategically distinct from Discovery (content curation). Bundling them would create confusion and undermine the "Listening" narrative. Separate nav position signals a first-class feature. Long-term, Listening becomes one of ContentStudio's core product pillars alongside Publish, Analytics, and Inbox.

---

### Decision 2: Topic Limit Per Plan

**Option A — 3 / 10 / Unlimited** *(Chosen)*
- Starter: 3 topics
- Growth/Pro: 10 topics
- Agency/Scale: Unlimited

**Option B — Per-mention volume cap (not per-topic)**
Unlimited topics but a monthly cap on mentions processed.

**Recommendation: Option A.** Per-topic limits are simpler to understand. Per-mention caps create unpredictable costs for agencies and are widely disliked. Topics are the natural unit users think in.

---

### Decision 3: Sentiment Analysis — External API vs. Internal Model

**Option A — External NLP API (e.g., Google Natural Language, OpenAI)** *(Chosen for V1)*
Fast to ship; high accuracy including sarcasm/emoji; pay per mention processed.

**Option B — Internal ML model**
Full control; no ongoing API costs; requires data science effort and training data.

**Recommendation: Option A for V1.** Speed to market matters. Lock in a `SentimentClassifierService` abstraction so the provider can be swapped later.

---

### Decision 4: Analytics Layout — Single Tab vs. Sub-Tabs

**Option A — Single Analytics tab (all charts in one scrollable page)** *(Original design)*

**Option B — Two sub-tabs: Performance + Insights** *(Chosen)*
- Performance: KPI grid + 5 time-series charts (Mention Volume, Reach, Engagement, Impressions, Sentiment)
- Insights: Network Breakdown Table, Influencers, Word Cloud, Geographic Distribution

**Recommendation: Option B.** The volume of charts is too high for a single scrollable page. Performance charts are high-frequency checks (daily); Insights tables are deeper dives (weekly reporting). Splitting them improves scannability without hiding content.

---

### Decision 5: AI Insights Delivery — Inline vs. On-Demand

**Option A — AI insights rendered inline below each chart** *(Intrusive for power users)*

**Option B — Sparkles (✨) button per widget that opens a popover on click** *(Chosen)*

**Recommendation: Option B.** Popover-on-demand keeps the dashboard clean while making AI insights discoverable. Every chart widget has one consistent Sparkles ActionIcon (violet) in its header. The popover shows 2–3 pre-generated insight bullets specific to that chart's data. This pattern also works cleanly in static report preview mode (isStatic=true — no controls but AI button remains).

---

## 6. Integration with Existing ContentStudio Features

### Inbox Integration (V1 — P0)
- **Reply from Listening:** Clicking "Reply" on any mention opens the Inbox composer with the mention pre-loaded (author handle, original text, platform). The reply flows through the standard Inbox conversation lifecycle.
- Requires: platform account must be connected in the workspace. If not connected, Reply button shows an error with a link to Integrations settings.
- For platforms ContentStudio cannot publish to (Reddit, news sites): Reply button replaced with "View on [Platform] →" external link.

### Composer / Planner Integration (V2)
- **Create content from trend:** A "Create Post" button on a trending hashtag or the Word Cloud opens the Composer with the keyword pre-filled. Converts listening insights directly into content creation.

### Analytics Integration (V2)
- **Earned mention reporting:** Social Listening data (mention volume, sentiment, SOV) surfaces as an "Earned Media" section in the Analytics module — giving users a full picture: owned channel performance + earned conversation alongside each other.

### Workspace / Multi-Account Structure
- Topics are workspace-scoped. Each workspace (each client for agencies) has its own independent listening topics, mention history, and alert settings.
- Competitors can only be compared within the same workspace — no cross-workspace data exposure.

---

## 7. Scope

### V1 (Launch)

Everything needed for a user to justify paying for the feature and see immediate value:

- **Landing page** with full upsell narrative, single flat add-on pricing ($99/mo, $79/mo annual), and demo topic access
- **4-step topic creation wizard** (Basic Info → Keywords → Sources → Alerts) with guided UI and AI keyword assist — no Boolean syntax required
- **Mention Feed** with filters: Platform, Sentiment, Content Type, Message Type, Language, Country, Date Range; sort by Newest / Most Engaged / Most Influential
- **Analytics tab** with two sub-tabs:
  - Performance: 7-card KPI grid + 5 time-series charts (Mention Volume, Reach, Engagement, Impressions, Sentiment Trend + Donut) + Network Breakdown Table
  - Insights: Sentiment by Platform, Geographic Distribution, Influencer/Top Authors panel, Word Cloud
- **AI Insights button** (Sparkles icon) on every analytics widget — opens popover with 2–3 tailored insight bullets
- **Competitor tracking**: Compare tab with Share of Voice donut, side-by-side KPI table, volume and sentiment comparison charts (up to 4 comparison topics)
- **Spike alerts**: configurable threshold + minimum floor, email + in-app delivery, 7-day baseline requirement
- **Reports tab**: Downloaded Reports list + Scheduled Reports list + Report Preview Drawer (static analytics snapshot)
- **Export modal**: 3 report types (Performance & Mentions, Compare Topics, Compare Periods) × 3 tabs (Download PDF/CSV, Schedule recurring delivery, Share link)
- **Settings tab**: keyword editor, platform toggles, alert configuration, danger zone (pause/delete)
- **Reply from Listening → Inbox** integration
- **Platform coverage**: X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Bluesky, Pinterest, Threads + News, Blogs, Forums, Review Sites
- **Plan limits + churned state**: topic limits enforced per plan tier; expired users see churned topic list with UpgradeModal

### V1.5 (Next Quarter)
- **AI Weekly Trend Summary** — auto-generated narrative digest per topic (surfaced as in-app card + optional email)
- **Saved mention collections** — bookmark mentions to named collections per topic for reporting
- **Reply tracking** — "Replied ✓" indicator on mention cards that have been actioned
- **Sentiment feedback loop** — "Mark as incorrect sentiment" button to improve NLP accuracy over time

### V2 (Future Roadmap)
- **Share of Voice historical tracking** — SOV as a time-series metric, not just a current-period snapshot
- **Create Post from Listening** → Composer integration
- **Earned Media tab** in Analytics module — published content (owned) + mentions (earned) in one view
- **Emotion clustering** beyond Positive/Neutral/Negative (frustrated, excited, confused)
- **Image/logo mention detection** — visual listening (technically complex, Brandwatch-tier)
- **Historical data import** for accounts with prior listening context
